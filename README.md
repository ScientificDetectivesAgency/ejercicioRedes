**NOTA** El examen consta de 3 secciones, las primeras dos secciones las tienen que resolver todos,la sección tres es para quienes reprobaron el primer exámen y puntos extras para los que si lo pasaron. 

Para resolver el exámen debes restaurar el backup que contiene las siguientes tablas:

| Tabla |Descripción |
|     :---:    |     :---:      |
| osm_cdmx | Red de la Ciudad de México (Open Street Maps, 2016) |
| denue_cdmx | Directorio Estadístico Nacional de Unidades Económicas (INEGI, 2016) |
| alcandias | Alcandías de las Ciudad de México |
| ecobici | Puntos de las estaciones de EcoBici |
| museos | Puntos de los museos en la Ciudad de México |


**Sección 1:** Preparación de los datos:

  1. Recortar la tabla **osm_cdmx** y el **denue_cdmx** con el poligono de las alcaldías: Benito Juárez, Alvaro Obregón, Miguel Hidalgo y Cuauhtémoc, y creas las tablas **osm_recorte** y **denue_recorte** (1 Punto). 
  1. Calcular la topología de la red (1 Punto).
  1. Agregar una columan que se llame _closest_node_ a las tablas **denue_recorte**, **museos** y **ecobici**, y realiza un update con el id del nodo más cercano de los vertices de la red (1 Puntos).
  1. Calcula la _longitud_ y el _tiempo_ de recorrido y agregalos como columnas a la red (1 punto)

**NOTA:** No olvides verificar las proyecciones de las capas

**Sección 2:** El gobierno de la Ciudad de México está planeando crear un programa de impulso a las actividades culturales a través del servicio de ecoBici; por lo que necesita elaborar rutas entre diferentes museos y saber cuáles son las estaciones de ecobici que se encuentran a menos de 20 minutos. Además de identificar el tipo de comercios que se pueden entrontrar al rededor de los museos.

  1. Agrega una columna que se llame _indice_ y calcula el tiempo en función del tipo de camino, dando prioridad al cycleway
(1 punto). 
**Pista** Para ello necesitas hacer un _case_
**Nota:** En la tabla osm_recorte columna  _type_id_ representa los id de ls tipos de camino, en la tabla de abajo puedes encontrar lo que significa cada uno.



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

1. Como ejemplo de las rutas del programa utiliza la columna _indice_ como costo y calcula la función pgr_TSP (Problema del Agente Viajero) para encontrar el orden en el que se deben recorrer puntos y trazar la ruta, guarda la tabla con la geometría de la ruta con el nombre **tsp_indice** (3 puntos):

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

  
 
