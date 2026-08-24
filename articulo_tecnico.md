# Cuatro Morelos en uno: qué revela el mapa de los negocios del estado

**Autores**

- García Quiroz, Gaspar Andrés - *20233tn107* - *UTEZ*
- Contreras Cortés, César Emilio - *20233tn076* - *UTEZ*
- Díaz Barroso, Juan José - *20233tn067* - *UTEZ*
- Jiménez Ureña, Ángel Sebastián - *20233tn097* - *UTEZ*
- Teja Carvajal, Erick Humberto - *20233tn060* - *UTEZ*

Grupo 9A. Materia: Extracción de conocimiento de Bases de Datos.

---

## Resumen

Morelos tiene 36 municipios y poco más de 113 mil negocios registrados. ¿Cuáles se parecen entre sí por el tipo de actividad al que se dedican? Para responderlo agrupamos los negocios de cada municipio por sector económico, convertimos esos conteos en porcentajes y dejamos que un algoritmo llamado K-medias encontrara las semejanzas por su cuenta. El resultado fueron cuatro perfiles. Uno es un caso único: Temoac, donde casi la mitad de los establecimientos fabrican dulces. Los otros tres agrupan municipios urbanos con oferta de servicios, municipios rurales dedicados al abasto, y municipios volcados al consumo directo. El hallazgo de fondo es que, fuera de Temoac, los municipios de Morelos se parecen mucho más entre sí de lo que uno imaginaría al comparar Cuernavaca con un pueblo de la sierra.

## Introducción

Camine por el centro de cualquier pueblo de Morelos y verá lo mismo: una tienda de abarrotes, una papelería, una tortillería, una estética. Camine por Cuernavaca y también las verá, solo que además hay despachos, hospitales privados y agencias. La pregunta que nos hicimos es si esa impresión de la calle se sostiene con datos, y sobre todo si se puede medir: **¿qué municipios de Morelos comparten un perfil económico parecido según el tipo de negocios que tienen?**

Comparar 36 municipios de a dos es imposible a simple vista: cada uno tiene su propia mezcla de comercios, talleres, restaurantes y consultorios, y esa mezcla son varios números a la vez. Aquí sirve el aprendizaje automático no supervisado, una familia de técnicas que busca patrones sin que nadie le diga de antemano qué buscar. Le entregamos la composición de cada municipio y le pedimos que formara grupos con los que se parecieran; nunca le dijimos cuáles debían quedar juntos.

El objetivo es descriptivo: ordenar el estado en unos pocos perfiles reconocibles que sirvan de punto de partida para decidir dónde ofrecer un servicio o dónde apoyar un giro específico.

## Fuente de datos

Usamos el DENUE, el Directorio Estadístico Nacional de Unidades Económicas del INEGI. Es el padrón oficial de establecimientos del país: cada renglón es un negocio con su ubicación, su giro y un rango de cuántas personas trabajan ahí. Tomamos el corte de junio de 2026, filtrado al estado de Morelos.

Son 113,066 establecimientos y 42 columnas por cada uno. Conviene aclarar qué cuenta el DENUE, porque de eso dependen todas las conclusiones: registra que un negocio **existe** y a qué se dedica, no cuánto vende ni cuánto gana. Una fábrica y un taller familiar valen uno cada uno.

## Preparación de datos

De las 42 columnas conservamos 11. Las demás eran datos de contacto —teléfono, correo, número exterior— que no dicen nada sobre el perfil económico de un municipio y que además concentraban casi todos los datos faltantes, por ser campos opcionales.

El paso clave fue traducir el giro de cada negocio a algo comparable. El DENUE clasifica con el SCIAN, un catálogo de casi mil actividades distintas cuyos dos primeros dígitos identifican el sector. Usamos ese criterio para pasar de mil actividades a 20 sectores, y luego juntamos esos 20 en 8 grupos: comercio al por menor, comercio al por mayor, manufactura, alojamiento y alimentos, servicios personales, salud y educación, servicios especializados y un grupo residual. Agrupamos porque algunos sectores tenían dos o tres establecimientos en todo el estado, y una variable casi vacía no distingue municipios; solo mete ruido.

Después convertimos el directorio en una tabla de 36 renglones, uno por municipio. Aquí está la decisión más importante del trabajo: usamos **porcentajes, no cantidades**. Cuernavaca tiene 25,404 establecimientos y Coatlán del Río tiene 235. Si le damos los conteos al algoritmo, va a agrupar por tamaño y nos va a devolver lo obvio, municipios grandes contra municipios chicos. Con porcentajes preguntamos otra cosa: no cuántos negocios hay, sino de qué tipo son.

Agregamos dos variables más: el tamaño promedio del establecimiento, calculado a partir del rango de personal, que distingue una economía de changarros de una de locales formales; y el porcentaje de puestos semifijos —tianguis y puestos de calle— como indicador aproximado de comercio en vía pública.

## Análisis exploratorio

Antes de modelar hay que mirar. Dos cosas saltaron a la vista.

La primera es que el comercio al por menor domina en casi todos lados. Es el sector mayoritario en 35 de los 36 municipios, en un rango que va del 28.8% al 56.2%. Dicho de otro modo: tener mucho comercio no distingue a nadie. Lo que separa a los municipios de Morelos no es el comercio, sino lo que tienen **además** del comercio.

La segunda es que hay una excepción, y es rotunda. En Temoac el comercio al por menor no manda: lo desplaza la manufactura, con 45.3% de los establecimientos frente a un promedio estatal de 10.5%.

Fuera de ese caso, la composición de todos los municipios se parece bastante. Eso ya anticipaba que los grupos existirían, pero separados por diferencias finas y no por contrastes dramáticos.

## Metodología

Trabajamos con nueve variables por municipio: siete de composición sectorial más el tamaño promedio del establecimiento y el porcentaje de semifijos. Dejamos fuera el grupo residual a propósito: los ocho porcentajes suman 100, así que conociendo siete el octavo queda determinado e incluirlo sería contar dos veces lo mismo.

Antes de agrupar, escalamos: los porcentajes rondan 45 y el tamaño promedio ronda 4, de modo que sin escalar las variables grandes pesarían más solo por estar medidas en otra escala.

El algoritmo es K-medias: coloca un número fijo de centros y asigna cada municipio al más cercano, repitiendo hasta que las asignaciones se estabilizan. Hay que decirle cuántos grupos formar, y ese número no viene con los datos. Probamos de 2 a 8 con dos criterios: uno mide cuánto se compacta la solución al agregar un grupo; el otro, el coeficiente de silueta, mide qué tan bien separados quedan.

Los dos no coincidieron: la silueta apuntaba a tres grupos y la compactación a cuatro. Elegimos cuatro. Con tres, 31 de los 36 municipios caían en el mismo saco, y un resultado que dice "31 municipios se parecen" no responde la pregunta; además la mayor ganancia de toda la serie ocurre justo al pasar de tres a cuatro, así que el cuarto grupo todavía captura estructura real.

También probamos la objeción evidente: excluir a Temoac por ser un dato raro. No mejora. Al sacarlo la separación queda prácticamente igual y el problema se muda de municipio: ahora es Tlalnepantla el que queda solo. Quitarlo no elimina la estructura que lo aisló; solo esconde el caso más informativo del estado.

## Resultados

Los cuatro perfiles quedaron así:

| Grupo | Municipios | Qué lo define | Representante |
|---|---|---|---|
| 1 | 1 | Manufactura: 45.3% contra 10.5% estatal | Temoac |
| 2 | 17 | Más servicios y establecimientos más grandes | Jojutla |
| 3 | 11 | Comercio de abasto, perfil rural | Tetela del Volcán |
| 4 | 7 | Comercio al por menor y alimentos | Tlayacapan |

El representante es el municipio más cercano al centro de su grupo: el ejemplo más típico del perfil.

## Interpretación

**Temoac es un caso de especialización genuina.** De sus 1,551 establecimientos, 592 —el 38% del municipio— se dedican a una sola actividad: elaboración de dulces y productos de confitería, lo que coincide con la especialización históricamente reconocida de la región. Queda solo con cualquier número de grupos que probamos, así que no es un accidente del método.

**El segundo grupo reúne a las economías urbanas y de cabecera.** Concentra las tres ciudades principales y las cabeceras del sur. Tiene más servicios personales, más salud y educación, y establecimientos más grandes: 4.89 personas estimadas por unidad contra 4.43 estatal. También es el grupo con más negocios, unos 5,084 por municipio contra 1,300 a 1,500 de los demás. Es el perfil donde, además del comercio de siempre, hay servicios que necesitan población concentrada para sostenerse.

**El tercero es un perfil de abasto rural.** Su rasgo distintivo es el comercio al por mayor, 3.85% contra 2.68% estatal. Son municipios del oriente que funcionan como intermediarios comerciales, sin la oferta de servicios de las cabeceras.

**El cuarto se vuelca al consumo directo.** Comercio al por menor y alojamiento y alimentos suman 65.7% de sus establecimientos, contra 59.6% del promedio. Aquí hay un matiz importante: el grupo mezcla municipios turísticos como Tepoztlán y Tlayacapan con municipios rurales de fuerte identidad comunitaria como Hueyapan y Xoxocotla. En Tepoztlán el peso de la comida y el hospedaje viene del visitante; en Hueyapan, el del comercio viene del abasto cotidiano. K-medias agrupa por composición, nunca por causa.

## Limitaciones

Reconocer hasta dónde llega el análisis es parte del análisis.

El DENUE cuenta negocios, no economía. No sabemos qué municipio factura más ni cuál está mejor: solo cómo está compuesto su padrón. El personal ocupado viene en rangos, así que el tamaño promedio es una estimación nuestra y no empleo medido. El directorio tampoco captura bien la actividad informal ambulante ni el trabajo desde casa, y ese subregistro probablemente no es parejo entre municipios.

Del lado del método, 36 municipios son muy pocos para nueve variables: quitar uno o cambiar una variable puede reacomodar la solución. La separación entre grupos es débil —coeficiente de silueta de 0.2074—, lo que significa que los grupos existen pero no están nítidamente delimitados, y hay municipios de frontera que podrían haber caído en otro lado. K-medias siempre devuelve el número de grupos que se le pida, existan o no; nunca avisa que los datos no se agrupan bien. Y elegir cuatro en vez de tres fue una decisión nuestra, justificada pero discutible.

Finalmente, es un corte único de junio de 2026: una fotografía, no una película. Y el análisis es descriptivo, no causal: dice qué municipios se parecen, no por qué.

## Conclusiones

Los municipios de Morelos se ordenan en cuatro perfiles: uno definido por la manufactura, uno por los servicios urbanos, uno por el comercio de abasto y uno por el consumo final. Las tres familias que nombraba nuestra pregunta resultaron ser efectivamente los ejes que los separan.

Tres cosas vale la pena subrayar. El comercio al por menor casi no distingue a nadie: está en todas partes. Hay un caso de especialización real y verificable en Temoac. Y fuera de ese caso, las diferencias son de grado y no de naturaleza: apenas uno a tres puntos porcentuales separan a los otros tres grupos. Esa es quizá la conclusión menos esperada, que la estructura económica de Morelos, medida por composición de establecimientos, es más homogénea de lo que uno supondría entre una capital estatal y un municipio rural.

El resultado sirve para focalizar servicios entre empresas, distribuir soluciones según la actividad dominante y como punto de partida para política pública regional. Como continuación, cruzar estos perfiles con la población municipal permitiría distinguir tamaño de intensidad económica, y compararlos contra otro corte del DENUE diría si son estables en el tiempo.

## Referencias

INEGI. (2026). *Directorio Estadístico Nacional de Unidades Económicas (DENUE)*, corte 2026/06, entidad 17 Morelos. Instituto Nacional de Estadística y Geografía, México. https://www.inegi.org.mx/app/descarga/?ti=6

INEGI. (s.f.). *Sistema de Clasificación Industrial de América del Norte (SCIAN)*. Instituto Nacional de Estadística y Geografía, México. https://www.inegi.org.mx/app/scian/

INEGI. (s.f.). *Términos de Libre Uso de la Información del INEGI*. https://www.inegi.org.mx/inegi/terminos.html

Pedregosa, F., et al. (2011). Scikit-learn: Machine Learning in Python. *Journal of Machine Learning Research*, 12, 2825–2830.
