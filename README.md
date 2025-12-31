# 📚 Reporte de Análisis de Catálogo Editorial

## 1. Resumen Ejecutivo

Este proyecto nace con el objetivo de explorar el mercado editorial actual utilizando datos reales obtenidos mediante **Web Scraping**. Logré transformar una base de datos propia de **44,489 títulos** en un panel interactivo diseñado para la toma de decisiones.

Durante el desarrollo, me enfoqué en identificar patrones de precios, el liderazgo de las editoriales y la distribución de género en la autoría. Uno de los mayores aprendizajes fue resolver técnicamente el "problema de empates" en rankings mediante **DAX**, asegurando que los gráficos muestren la información exacta y visualmente limpia.

---

## 🛠️ Tech Stack

* **Extracción:** Python (Web Scraping para recolección de datos).
* **Limpieza y ETL:** Microsoft Excel (Uso de fórmulas anidadas y Power Query).
* **Modelado:** Power BI (Implementación de Esquema de Estrella).
* **Cálculos:** DAX (Data Analysis Expressions).

---

## 2. Metodología: El Ciclo del Dato

Para asegurar que el análisis fuera confiable y profesional, seguí este proceso:

### 2.1 Extracción y Enriquecimiento

* **Web Scraping:** Recolecté títulos, autores, editoriales y precios en tiempo real de una librería líder.
* **Enriquecimiento de Datos:** Realicé una investigación adicional para identificar el **género de los autores**, lo que me permitió crear un perfil de autoría que no existía en la fuente original.

### 2.2 Ingeniería de Limpieza y Normalización (Excel)

Debido a que los datos venían directamente de la web, realicé un trabajo exhaustivo de limpieza para que el modelo funcionara correctamente:

<p align="center">
  <img src="03_Images/BUSCARX.gif" width="650" alt="Evidencia de BUSCARX en Excel">
  <br>
  <em>Uso de la función =BUSCARX() para el enriquecimiento de la identidad de género.</em>
</p>

#### **A. Calidad y Consistencia de Datos**

* **Gestión de Errores:** Eliminé registros con años erróneos (como `9999`) y filas sin **ISBN** o **Precio** para mantener la integridad de los promedios.
* **Depuración:** Filtré productos ajenos al catálogo editorial como "Gift Cards".
* **Corrección de Encoding:** Arreglé errores tipográficos comunes en scraping (ej. `Ã¡` → `á`, `Ã±` → `ñ`) para que los textos se lean correctamente.

#### **B. Normalización Mediante Fórmulas**

Utilicé funciones avanzadas para estandarizar miles de filas de forma automática:

* **Formato de Texto:** `=NOMPROPIO()` para corregir mayúsculas.
* **Extracción de Autores:** Creé una fórmula anidada para extraer solo al autor principal en casos de co-autoría:  
    `=NOMPROPIO(SI.ERROR(IZQUIERDA([@Autor];HALLAR(";";[@Autor])-1);[@Autor]))`
* **Limpieza de Ruidos:** Uso de `=SUSTITUIR(ESPACIOS([@Autor]);"...";"")`.

#### **C. Transformación de Datos**

* **Formatos Numéricos:** Eliminé unidades de medida (`g`) y corregí los formatos de moneda para poder realizar cálculos matemáticos en Power BI.
* **Cruce de Datos:** Usé `=BUSCARX()` para integrar la identidad de género asignada a cada autor:  
    `=BUSCARX(B2;autores!A:A;autores!B:B;"No encontrado";0)`

### 2.3 Análisis Exploratorio (EDA en Excel)

Antes de pasar a Power BI, audité los datos con herramientas nativas de Excel:

* **Tablas Dinámicas:** Para validar que los promedios y recuentos fueran consistentes entre categorías.
* **Gráficos de Comparación:** Para detectar visualmente si alguna editorial o precio presentaba anomalías.
* **Segmentadores (Slicers):** Implementé filtros rápidos para interactuar con los datos y verificar su coherencia.

---

## 🛡️ Garantía de Diseño, Usabilidad y Calidad del Dato

Para asegurar que este proyecto sea una herramienta profesional, robusta y escalable, implementé estándares de calidad en cada etapa del desarrollo:

### 📗 En Microsoft Excel (ETL y UX)

* **Optimización de Interfaz y UX:** * Se implementó la **protección de hojas** con permisos específicos, permitiendo la interacción total con segmentadores pero bloqueando la edición accidental de celdas clave.
    * Se aplicó el bloqueo de **relación de aspecto y posición** en todos los gráficos; esto asegura que los elementos visuales mantengan su estructura fija y profesional, sin deformarse ni desplazarse durante el filtrado de datos.
* **Protección de Datos e Integridad:** Se bloquearon las hojas de "Catálogo" y "Autores" para resguardar la fuente de origen, configurando los permisos de modo que la seguridad de Excel no interfiera con la actualización automatizada del modelo en Power BI.

### 📊 En Power BI (Modelado y Visualización)

* **Optimización del Modelo (Performance):** Se aplicó una limpieza profunda en Power Query mediante la técnica de **"Quitar otras columnas"**, eliminando metadatos innecesarios y reduciendo el peso del archivo `.pbix`. Esto garantiza una mayor eficiencia en el motor de compresión y rapidez en la carga de visuales.
* **Estructura de Navegación Bloqueada:** * Se activó el **Bloqueo de Objetos** (Locked Layout) en el lienzo para evitar desplazamientos accidentales de los gráficos durante el uso interactivo del dashboard.
    * Se configuró la vista de página en modo **"Ajustar a la página"**, asegurando que el zoom y el encuadre sean constantes en cualquier resolución de pantalla.
* **Control de Interfaz y Precisión:** * Se deshabilitaron los iconos de encabezado innecesarios para reducir el **ruido visual** y prevenir que el usuario altere involuntariamente el orden de los datos.
    * La combinación de la medida de **Desempate DAX** con el filtrado por Valor Máximo asegura que los rankings (Top 5) sean exactos y estéticos, incluso ante empates en los precios.

<p align="center">
  <img src="03_Images/Modelo.png" width="650" alt="Modelo de datos">
  <br>
  <em>Arquitectura del Modelo de Datos: Implementación de Esquema de Estrella para optimización de consultas.</em>
</p>

---

## 3. Modelo y Diccionario de Datos

Diseñé un **Esquema de Estrella** para que el reporte sea rápido y eficiente.

| Tabla | Campo | Tipo de Dato | Descripción |
| :--- | :--- | :--- | :--- |
| **Fact_Catalogo** | `ISBN` | Número (Key) | Identificador único del libro (Llave Primaria). |
| **Fact_Catalogo** | `Precio` | Decimal | Valor de venta al público. |
| **Dim_Autor** | `Identidad` | Texto | Género del autor (Masculino/Femenino). |
| **Dim_Editorial** | `Editorial` | Texto | Sello responsable de la publicación. |
| **Dim_Libro** | `Año` | Fecha/Año | Año de lanzamiento o edición. |

---

## 4. Desafío Técnico: Rankings de Precisión (DAX)

Un reto interesante en Power BI es cuando varios libros tienen el mismo precio. Esto hace que un "Top 5" muestre más de 5 elementos, lo cual ensucia el reporte visualmente.

**Mi Solución:** Desarrollé una medida de desempate usando `RANKX`. La lógica consiste en tomar el precio y sumarle un valor mínimo (infinitesimal) basado en el nombre del libro, logrando así un ranking único y exacto.

```text
Desempate Libros = 
VAR Conteo = [Precio Máximo] 
VAR RankingUnico = 
    RANKX(
        ALL(Fact_Catalogo[Titulo]), 
        CALCULATE(SELECTEDVALUE(Fact_Catalogo[Titulo])), 
        , 
        ASC
    )
RETURN
IF(
    NOT ISBLANK(Conteo),
    Conteo + DIVIDE(RankingUnico, 1000000)
)

```

<p align="center">
  <img src="03_Images/DAX.png" width="650" alt="Implementación DAX">
  <br>
  <em>Lógica DAX: Implementación de la medida de desempate para garantizar la precisión en los rankings</em>
</p>

---

## 5. Visualización y Hallazgos (Insights)

* **Concentración de Mercado:** Detecté que el sello **Ivrea** lidera el volumen con **1,083 títulos**, mostrando una clara especialización en su sector.
* **Análisis por Género:** El catálogo se distribuye en un **61.13% (M)** frente a un **38.87% (F)** en autoría.
* **Benchmarking:** El precio promedio global es de **$27,817**, lo que sirve como base para comparar precios entre distintas categorías.

<p align="center">
  <img src="03_Images/Dashboard_1.gif" width="650" alt="Presentación del Dashboard">
  <br>
  <em>Navegación Interactiva: Visualización dinámica de KPIs y comportamiento del catálogo.</em>
</p>

---

## ⚠️ Limitaciones y Contexto del Análisis

Todo análisis de datos tiene un alcance definido por la naturaleza de su fuente. Para la correcta interpretación de este reporte, se deben considerar los siguientes puntos:

* **Sesgo de Temporalidad (Impacto Post-Pandemia):** Se identificó una caída significativa en el volumen de títulos registrados entre **2020 y 2021**. Tras auditar el proceso de ETL, se confirmó que no se trata de una pérdida de datos, sino de un reflejo real del cese de actividades y distribución editorial durante la crisis sanitaria global.
* **Representatividad de la Fuente:** Los datos provienen de un *web scraping* de una librería líder. Si bien la muestra es masiva (44k+ registros), los precios y el stock reflejan la realidad de dicho comercio y pueden variar respecto a otras cadenas o mercados internacionales.
* **Enriquecimiento de Género:** La identidad de género fue asignada mediante un proceso de cruce de datos y validación manual. Existe un pequeño porcentaje de autores (menos del 5%) categorizados como "No identificado" debido a nombres ambiguos o falta de información pública disponible.
* **Dependencia de Proveedores:** El análisis de concentración muestra que el catálogo está fuertemente influenciado por los tres sellos principales (Ivrea, Planeta y Penguin), lo que debe tenerse en cuenta al extrapolar tendencias de precios a editoriales independientes.

---

## 6. Estructura del Repositorio

* 📁 **01_Data**: Carpeta con los datos `Raw` y los datos finales `Processed`.
* 📁 **02_Report**: El archivo `.pbix` con el dashboard interactivo y reporte ejecutivo.
* 📁 **03_Images**: Capturas de pantalla y demostraciones visuales.

---

## 🚀 Cómo utilizar este proyecto

1. Descarga o clona este repositorio.
2. Los datos limpios están en `01_Data/Processed`.
3. Abre el archivo de Power BI y, si es necesario, actualice la ruta de los datos en **Configuración de origen de datos** para que apunte a tu carpeta local.

---

## 💡 Nota del Autor

Este proyecto me permitió fortalecer mis habilidades en el ciclo completo del dato, desde la extracción hasta la visualización estratégica. Si tienes alguna sugerencia o feedback, ¡será muy bienvenido!
