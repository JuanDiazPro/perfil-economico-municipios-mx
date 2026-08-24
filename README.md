# Perfil económico de los municipios de Morelos por tipo de actividad

## Integrantes

- García Quiroz Gaspar Andrés
- Contreras Cortés César Emilio
- Díaz Barroso Juan José
- Jiménez Ureña Ángel Sebastián
- Teja Carvajal Erick Humberto

## Grupo

9A

## Materia

Extracción de conocimiento de Bases de Datos

## Pregunta de investigación

> ¿Qué municipios del estado de Morelos comparten un perfil económico similar según la concentración de sus unidades económicas dedicadas a la manufactura, el comercio y los servicios?

## Fuente de datos

**DENUE** (Directorio Estadístico Nacional de Unidades Económicas) — INEGI.

- **Corte:** DENUE 2026/06
- **URL de descarga:** https://www.inegi.org.mx/app/descarga/?ti=6
- **Cobertura:** estado de Morelos (36 municipios)
- **Fecha de descarga:** agosto de 2026

La procedencia completa —ruta de descarga, checksums, columnas utilizadas, condiciones de uso y cita— está documentada en [`referencias/fuentes_datos.txt`](referencias/fuentes_datos.txt).

## Descripción breve del dataset

El dataset original contiene **113,066 registros** (uno por unidad económica: cada establecimiento registrado en Morelos) y **42 columnas** en codificación `latin-1`, incluyendo domicilio, contacto, actividad económica (SCIAN), personal ocupado (en rangos), tipo de unidad económica y coordenadas geográficas.

Tras la limpieza se conservan **11 columnas relevantes** y se agregan variables derivadas (sector económico, clave geográfica INEGI). Para el modelo, esos 113,066 registros se agregan a una tabla de **36 filas** —una por municipio— con la composición porcentual de sus unidades económicas por sector y variables estructurales (tamaño promedio de establecimiento, % de comercio semifijo).

## Técnicas usadas

- **Limpieza de datos:** selección de columnas relevantes, verificación de duplicados y nulos, creación de `sector_nombre` a partir del código SCIAN.
- **Agregación e ingeniería de variables:** conversión de un directorio de establecimientos a una tabla municipal; composición sectorial en porcentajes (no conteos absolutos) para evitar que el modelo agrupe por tamaño de economía en vez de por perfil; agrupación de 20 sectores SCIAN en 8 grupos; estimación de tamaño promedio de unidad a partir de los rangos de `per_ocu`.
- **Escalado:** `StandardScaler` sobre las 9 variables del modelo.
- **Modelo no supervisado:** `KMeans`, con selección de `k` mediante el método del codo (inercia) y el coeficiente de `silhouette`.
- **Visualización:** `PCA` a 2 componentes, únicamente para proyectar y comunicar el resultado del agrupamiento (el modelo se ajusta sobre las 9 variables originales, no sobre los componentes).
- **Persistencia del modelo:** guardado con `joblib` del modelo, el escalador, las variables usadas y el reetiquetado de clusters, con una prueba de recarga que verifica la reproducibilidad.

## Principales resultados

Con `k = 4` (silhouette = 0.2074) se identificaron cuatro perfiles económicos municipales:

| Cluster | Municipios | Perfil | Representante |
|---|---|---|---|
| 0 | 1 | Especializado en manufactura (dulces y confitería) | Temoac |
| 1 | 17 | Servicios y cabeceras urbanas, establecimientos más grandes | Jojutla |
| 2 | 11 | Comercio al por mayor y manufactura ligera del oriente | Tetela del Volcán |
| 3 | 7 | Consumo final: comercio al por menor y alojamiento/alimentos | Tlayacapan |

El comercio al por menor es el sector mayoritario en 35 de los 36 municipios (28.8%–56.2%); la única excepción es Temoac, donde lo desplaza la manufactura (45.3%). Lo que distingue a los grupos son diferencias de 1 a 3 puntos porcentuales en el resto de la composición, salvo el propio Temoac, que se separa por una desviación de +34.8 puntos en manufactura. Los detalles completos, gráficas y la interpretación de cada grupo están en el notebook.

## Limitaciones

- El DENUE cuenta establecimientos, no producción, ingresos ni valor agregado.
- `per_ocu` es un rango categórico; el tamaño promedio de unidad se estima con el punto medio de cada rango, no es empleo medido.
- Solo 36 observaciones para 9 variables: el modelo es sensible a la escala y a la inicialización.
- Las variables de composición son composicionales: los ocho grupos de sector suman 100% por municipio. Se excluyó `pct_otros_grupo` del modelo para reducir esa redundancia, pero la dependencia parcial entre las siete restantes permanece.
- La silhouette obtenida (0.2074) indica una separación débil entre grupos.
- Corte temporal único (DENUE 2026/06): es una fotografía, no permite hablar de tendencias.
- No se incorporan población, superficie ni PIB municipal.
- Los resultados son descriptivos, no causales.

El detalle completo está en la sección 16 del notebook.

## Cómo ejecutar el notebook

Requiere **Python 3.12** (con Python 3.14 hay riesgo de que `scikit-learn`/`matplotlib` no tengan wheel disponible).

Las versiones exactas con las que se ejecutó el análisis están fijadas en [`requirements.txt`](requirements.txt) — las principales son `pandas 3.0.5`, `numpy 2.5.2`, `scikit-learn 1.9.0` y `matplotlib 3.11.1`.

```bash
git clone https://github.com/JuanDiazPro/perfil-economico-municipios-mx.git
cd perfil-economico-municipios-mx
py -3.12 -m venv .venv
.venv/Scripts/python -m pip install --upgrade pip
.venv/Scripts/python -m pip install -r requirements.txt
.venv/Scripts/python -m ipykernel install --user --name perfil-morelos --display-name "Python (perfil-morelos)"
```

El dataset original (`data/raw/dataset_original.csv`) ya está incluido en el repositorio, así que no requiere descarga adicional.

Para ejecutar el notebook completo desde la terminal:

```bash
.venv/Scripts/jupyter nbconvert --to notebook --execute --inplace notebook/analisis_datos_publicos.ipynb
```

O abrirlo con Jupyter Lab/Notebook y correrlo con el kernel **Python (perfil-morelos)**:

```bash
.venv/Scripts/jupyter lab
```

## Artículo técnico

El artículo está en [`articulo_tecnico.md`](articulo_tecnico.md) (~1,800 palabras). Aún no se envía a publicación.
