**NOTA** El examen consta de 3 secciones, las primeras dos secciones las tienen que resolver todos,la sección tres es para quienes reprobaron el primer exámen y puntos extras para los que si lo pasaron. 

Para resolver el exámen deber restaurar el backup que contiene las siguientes tablas:

| Tabla |Descripción | Id del Museo |
|     :---:    |     :---:      |     :---:     |
| Inicio | Museo Casa Lenin Trotsky | 966 |
| 00 | Museo Histórico Judío y del Holocausto Tuvie Maizel | 847 |
| 00 | Planetario Joaquín Gallo | 1776 |
| 00 | Museo de la tortura | 1806 |
| 00 | Museo del Pulque y las pulquerías | 2110 |

**Sección 1:** Preparación de los datos
  1. Recortar la red de la cdmx y el DENUE con el poligono de las alcaldías: Benito Juárez, Alvaro Obregón, Miguel Hidalgo y Cuauhtémoc (1 Punto). 
  1. Calcular la topología de la red (1 Punto).
  1. Agregar una columan que se llame closest_node y realiza un update con el id del nodo más cercano de la red a las tablas:      Denue(RECORTADO), museos y ecobici (1 Punto).

NOTA: No olvides verificar las proyecciones de las capas

Sección 2: Supongamos que el gobierno de la Ciudad de México está planeando impulsar un programa de impulso al turismo a través del servicio de ecoBici, por lo que necesita elaborar rutas entre diferentes museos y saber qué servicios asociados al turismo, como cafeterías y restaurantes, se pueden entrontrar al rededor de las estaciones de ecoBici y los museos a 20 y 10 minutos.

1. Como ejemplo de las rutas del programa utiliza la función pgr_TSP (Problema del Agente Viajero) para encontrar el     orden en el que se deben recorrer los siguientes puntos para optimizar los costos: 


| Orden del nodo | Nombre del Museo | Id del Museo |
|     :---:    |     :---:      |     :---:     |
| Inicio | Museo Casa Lenin Trotsky | 966 |
| 00 | Museo Histórico Judío y del Holocausto Tuvie Maizel | 847 |
| 00 | Planetario Joaquín Gallo | 1776 |
| 00 | Museo de la tortura | 1806 |
| 00 | Museo del Pulque y las pulquerías | 2110 |

1. Calcula las áreas de servicio de cada uno de los museo e identifica cuáles son las estaciones de EcoBici que se encuentran a 10 min caminando. Y crea una tabla que se llame **estaciones_10min**
