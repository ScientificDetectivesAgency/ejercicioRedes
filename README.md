
## Problema ##
El gobierno de la Ciudad de México está planeando crear un programa de impulso a las actividades culturales a través del servicio de ecoBici; por lo que necesita elaborar rutas entre diferentes museos y saber cuáles son las estaciones de ecobici que se encuentran a menos de 20 minutos. Además de identificar el tipo de comercios que se pueden entrontrar al rededor de los museos, impulsar la economía de los comerciantes en esa zona.

Para resolver el ejercicio debes restaurar el backup que contiene las siguientes tablas:

| Tabla |Descripción |
|     :---:    |     :---:      |
| osm_cdmx | Red de la Ciudad de México (Open Street Maps, 2016) |
| denue_cdmx | Directorio Estadístico Nacional de Unidades Económicas (INEGI, 2016) |
| alcaldias | Alcaldías de las Ciudad de México |
| ecobici | Puntos de las estaciones de EcoBici |
| museos | Puntos de los museos en la Ciudad de México |

### Parte I: Preparación de los datos ### 

Dado que los museos con los que vamos a trabajar están en las delegaciones: Benito Juárez, Alvaro Obregón, Miguel Hidalgo y Cuauhtémoc. Será necesario hacer un recorte de las tablas **osm_cdmx** y **osm_cdmx** con el polígono de alcaldías. 

```sql
select  a.cvegeo, b.*
into osm_recorte
from alcaldias a
join osm_cdmx b
on st_intersects(a.geom, b.geom)
where a.cve_mun IN ('014','010', '016', '015');
```
__NOTA:__ El Qry anterior corta la red, ahora haz lo mismo para el DENUE

Ya que tenemos la red con la que vamos a trabajar, será necesario calcular la topología de la misma 

```sql
alter table osm_recorte add column source integer;
alter table  osm_recorte add column target integer;

select pgr_createTopology ('osm_recorte', 0.005, 'geom', 'id'); 
```
Ahora como los algoritmos que trabajan con redes no reconocen puntos que no estén asociados a los nodos de la red, vamos a agregar a las tablas **denue_recorte**, **museos** y **ecobici** una coloumna que se llame `closest_node` (este campo debe ser de tipo `bigint`) y le asignaremos el id del nodo más cercano de la tabla de vertices. 

```sql
alter table ecobici add column closest_node bigint;

update ecobici set closest_node = c.closest_node
from  
(select b.id as id_ecobici, (
SELECT a.id
FROM osm_recorte_vertices_pgr As a
ORDER BY b.geom <-> a.the_geom LIMIT 1
)as closest_node
from  ecobici b) as c
where c.id_ecobici = ecobici.id;
```
**NOTA1:** Repite la operación anterior para las todas las tablas de puntos 
**NOTA2:** No olvides verificar las proyecciones de las capas


### Parte II: Costos ### 
Hasta este punto ya hemos preparado los datos para comenzar a trabajar. Sin embargo, es necesario determinar cuales son los valores de costo con los que se va a realizar en análisis, el costo en análisis de redes de transporte implica la dificultad de circular por un camino (arco) comparado con otro y podemos definirlo como fricción. En este caso es necesario hacer que las rutas vayan por calles seguras para circular en bicicleta, en la tabla de abajo podemos identificar la clasificación de Open Street Maps del tipo de vialidad, además de la velocidad máxima. 

|class_id | type_id | name | priority | default_maxspeed|
|  :---:  | :---:   | :---: |     :---:      |    :---:    |   
|201 |       2 | lane              |        1 |               50|
|204 |       2 | opposite          |        1 |               50|
|203 |       2 | opposite_lane     |        1 |               50|
|202 |       2 | track             |        1 |               50|
|120 |       1 | bridleway         |        1 |               50|
|116 |       1 | bus_guideway      |        1 |               50|
|121 |       1 | byway             |        1 |               50|
|118 |       1 | cycleway          |        1 |               50|
|119 |       1 | footway           |        1 |               50|
|111 |       1 | living_street     |        1 |               50|
|101 |       1 | motorway          |        1 |               50|
|103 |       1 | motorway_junction |        1 |               50|
|102 |       1 | motorway_link     |        1 |               50|
|117 |       1 | path              |        1 |               50|
|114 |       1 | pedestrian        |        1 |               50|
|106 |       1 | primary           |        1 |               50|
|107 |       1 | primary_link      |        1 |               50|
|110 |       1 | residential       |        1 |               50|
|100 |       1 | road              |        1 |               50|
|108 |       1 | secondary         |        1 |               50|
|124 |       1 | secondary_link    |        1 |               50|

Para determinar los costos agregaremos tres columnas: `longitud`, `tiempo` e `indice` y calculamos cada una de ellas. Primero calculamos la `longitud` con la función `st_length`: 

```sql
update osm_recorte set longitud = st_length(geom)
```
Teniendo la longituda y la velocidad, ahora despejamos el tiempo y lo convertimos a minutos: 

```sql
update osm_recorte
set tiempo_min = longitud::float/((maxspeed::float)*16.6667) 
```
Ahora para poder considerar el tipo de camino y escalar la velocidad, longitud o tiempo en función de este atributo, vamos a hacer un índice, donde seleccionemos los caminos transitables con mayor facilidad por una bicicleta y vamos a estirar los valores de longitud para todas las demás categorias. Esto es porque si el valor del indice es mayor, entonces el algoritmo no tomará como primera opción ese camino. En tabla de abajo observarás los tipos de vialidad por donde es mejor circular en bicicleta.  

|class_id | type_id | name | priority | default_maxspeed|
|  :---:  | :---:   | :---: |     :---:      |    :---:    |   
|201 |       2 | lane              |        1 |               50|
|204 |       2 | opposite          |        1 |               50|
|203 |       2 | opposite_lane     |        1 |               50|
|202 |       2 | track             |        1 |               50|
|118 |       1 | cycleway          |        1 |               50|


```sql
update osm_recorte set indice =
  case
    when class_id = '201' then longitud*0.5
    when class_id = '204' then longitud*2
    when class_id = '203' then longitud*2
    when class_id = '202' then longitud*2
    else longitud * 10
  end;
```

### Parte III: Agente Viajero ### 
1. Como ejemplo de las rutas del programa utiliza la columna _indice_ como costo y calcula la función pgr_TSP (Problema del Agente Viajero) para encontrar el orden en el que se deben recorrer puntos y trazar la ruta, guarda la tabla con la geometría de la ruta con el nombre **tsp_indice** (3 puntos):

https://github.com/eurekastein/agente

| Orden del nodo | Nombre del Museo | Id del Museo |
|     :---:    |     :---:      |     :---:     |
| Inicio | Museo Casa Lenin Trotsky | 966 |
| 00 | Museo Histórico Judío y del Holocausto Tuvie Maizel | 847 |
| 00 | Planetario Joaquín Gallo | 1776 |
| 00 | Museo de la tortura | 1806 |
| 00 | Museo del Pulque y las pulquerías | 2110 |

  1. Repite lo anterior pero con la columna tiempo como costo y guarda la tabla con la geometría de la ruta con el nombre **tsp_tiempo** (3 puntos)

  1. Calcula las áreas de servicio a 20 minutos de cada uno de los museos de las lista anterior, usa el _indice_ como costo, haz el join con la geometría de los puntos y guarda las tablas con el nombre **as20min_#id_museo#**. Con la siguiente función calcula el poligono que lo delimita (3 Puntos): 

```sql
create table area20min_#id_museo# as 
select * from st_setsrid(
pgr_pointsAsPolygon(
'select node::int4 as id, st_x(st_geometryn(the_geom,1)) as x,  st_y(st_geometryn(the_geom,1)) as y 
from as20min_#id_museo#'),32614)as geom;
```
  1. Ahora  queremos saber cuales son las estaciones de EcoBici y los comercios que se encuentran a 20 minutos, intersecta los 4 polígonos con las tablas **denue_recorte** y **ecobici** y crea las tablas **20min_denue**, **20min_ecobici** (3 puntos) 

**Sección 3:** 

1. Ahora para comprar la diferencia entre áreas de servicio y un buffer, calcula el buffer a 20min cominando con el índice como costo de cada uno de los museos de la lista anterior y realizala intersección anterior, guardalas como **buf20min_denue**, **buf20min_ecobici** (4 puntos)

**Pista:** Despeja de la formula de velocidad usando la longitud y la velocidad promedio caminando

  
 
