Tema 2:
Perfil economico de municipios por tipo de actividad

Pregunta guia:
Que municipios tienen perfiles economicos parecidos?

Fuente sugerida:
INEGI o datos abiertos relacionados con unidades economicas.

Modelo sugerido:
KMeans

Variables posibles:

- numero de unidades economicas;
- sector economico;
- comercio;
- servicios;
- manufactura;
- actividad por municipio.

Producto esperado:
Segmentos de municipios segun su perfil economico.



aca empieza el análisis gente:

El análisis o el notebook intenta encontrar municipios que compartan un perfil económisoc según la concentración de sus unidades económicas, una unidad económica es básicamente una tiendita y así, cosas que son parte de la economía

##Exploración inicial

El dataset tiene 42 columnas inicialmente, la mayoría no sirve
mucho nulo, aquello que tenga nulos es opcional y por ende se elimina


##Limpieza de los datos

Se eliminan las columnas que no tienen mucha importancia, y eso tmb se lleva los nulos
Cada fila tiene un "codigo_act", que es el sector económico que representa (los 2 primeros números), se mapean los datos originales para crear una nueva columna "sector_nombre" que sea en texto


##Construcción de la tabla inicial

Se toma las unidades de un municipio de acuerdo a un porcentaje para evitar que el modelo se vaya por economía "grande", "mediana"", y "chica"
Se agrupan los sectores para evitar el ruido y mejorar el análisis, solo quedan 8:
- Comercios al por mayor
- Comercios al por menor
- Manufactora
- Alojamiento y alimentos
- Salud y educación
- Servicios especializados
- Otros servicios
- Otros grupos

Se transforman otras columnas:
- per_ocu es un rando de personas, por lo que se cambia a una estimado
- Se transforma "TipoUniEco" a "es_semifijo" para que 0 sea lugar fijo y 1 semifijo (tianguis, puestos)
- Se genera una crosstab para generar una tabla con columnas de sector y filas municipios

Se genera otra crosstab pero ahora en lugar de conteo, porcentajes
Se genera otra tabla, ahora si la final, en esta se ponen variables más útiles pal modelo (tamaño promedio, porcentaje de puestos semifijos)
y ya se tiene la tabla de los 36 municipios con los datos útiles


##Análisis exploratorio

aca se inicia finalmente
se genera la primera gráfica que es solo para decir la cantidad de unidades económicas absolutas por municipio, lo que acaba siendo muy desigual
Por eso se trabaja con porcentajes, pues se puede notar que hay demasiada diferencia y se agruparía por tamaños, no por perfiles+

La segunda tabla es una comparación de los tipos de sectores en un municipio, viendo qué porcentaje representan
Se crea una tabla complementaria para ver los municipios que representan el mayor y menor porcentaje de los tres sectores más grandes


## variables y escalado

esta parte es para ver bn qué es lo que el modelo necesita en base a la pregunta inicial
Se agrupan nuevamente sectores para un análisis básico y se 

escalado: Esto es básicamente reducir los valores a un rango de 0 y 1, esto es para que las variables tengan el mismo valor
de no ser así, algunos valores pesarían mucho y otros casi nada

Matriz de correlación:
Básicamente cómo escalan unas variables con otras, 1 es fuerte positiva (una sube la otra tmb, aunque el 1 en sí es para la variable misma), -1 lo contrario, y 0 nada


## Elección del número de grupos