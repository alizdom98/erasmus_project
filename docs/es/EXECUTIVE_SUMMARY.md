# Erasmus+ Mobility Analysis - Resumen del Proyecto

[Read in English](../en/EXECUTIVE_SUMMARY.md) | [Leer en Español](EXECUTIVE_SUMMARY.md)

**Autor**: Andrés Liz Domínguez
**Fecha**: Febrero 2026
**Tipo**: Proyecto de Portfolio - Data Science & Business Intelligence

---

## Qué es este proyecto

Este es un proyecto de análisis de datos que hice para mi portfolio. Trabajé con 2.1 millones de registros de movilidades Erasmus+ (educación superior, 2017-2024) para ver cómo COVID-19 cambió los patrones de movilidad estudiantil en Europa.

Los datos originales eran bastante sucios y heterogéneos (11 archivos CSV con estructuras diferentes), así que una parte importante del proyecto fue limpiarlos y transformarlos en algo utilizable. Después creé un dashboard en Power BI para visualizar los resultados.

---

## Principales hallazgos

### 1. Recuperación casi completa

El programa se recuperó casi por completo después de COVID:
- Pre-COVID (2017-2019): aprox. 2M movilidades
- COVID (2020-2021): caída a 485K
- Post-COVID (2022-2024): recuperación a aprox. 2M

El "Recovery Rate" que calculé es 99.77%. Es decir, casi hemos vuelto al nivel anterior.

---

### 2. Cambio de destinos favoritos

**Los ganadores** fueron países del sur y este de Europa:
- Croacia: +37%
- Grecia: +35%
- Turquía: +32%
- Ucrania: +137% en envíos (claramente por la guerra)

**Los perdedores**:
- Rusia: -90% (guerra + sanciones)
- Reino Unido: -70% (Brexit)
- Polonia, Francia, Alemania: caídas moderadas

Lo interesante es que España, Francia y Alemania siguen siendo los destinos más populares en términos absolutos, pero están perdiendo cuota frente a opciones más económicas.

---

### 3. Boom de tecnología

Los cambios más llamativos en campos de estudio:

**Aumentaron:**
- ICT (Informática): +41% - el mayor ganador
- Natural Sciences: +14%
- Agriculture: +10%
- Health: +10%

**Disminuyeron:**
- Arts & Humanities: -22% - la mayor caída
- Services: -7%
- Business: -7%

Mi interpretación: la digitalización acelerada por COVID hizo que más gente se interesara por tecnología.

---

### 4. Más inclusividad

Los participantes con "fewer opportunities" (grupos desfavorecidos) casi se duplicaron:
- Pre-COVID: 68K
- Post-COVID: 149K
- Aumento: +119%

Alemania fue el mayor contribuidor (+39K participantes). No estoy seguro de la razón exacta, pero podría ser un cambio en políticas o en cómo se registra este dato.

---

### 5. Movilidades más cortas

Después de COVID hay una tendencia a movilidades más cortas:
- Las de 6-9 meses bajaron
- Las de 3-6 meses siguen siendo las más comunes
- Las movilidades cortas (<3 meses) aumentaron relativamente

Probablemente por cautela económica o popularidad de programas intensivos.

---

## Parte técnica

### Limpieza de datos (Python)

Trabajé con un pipeline de 9 fases sobre 6 millones de registros:
1. Verificar estructura de los archivos anuales
2. Cargar y unificar los 11 CSVs
3. Normalizar fechas y años académicos
4. Eliminar duplicados (9,165 registros)
5. Limpiar outliers en edad y duración (600K+ valores)
6. Normalizar categorías de texto
7. Crear variables derivadas (grupos ISCED, grupos de actividad)
8. Normalizar países por código ISO2
9. Filtrar y preparar el subset final

El resultado fue un dataset limpio de 2.1M registros × 21 variables.

### Modelo de datos (Power BI)

Usé un modelo estrella (star schema):
- **Tabla de hechos**: df_he_pbi con los 2.1M registros
- **Dimensiones**: Dim_Country_Sending, Dim_Country_Receiving, DimDate
- **Tablas desconectadas**: AxisCountry (para análisis transversales) y Direction (para slicers)

¿Por qué dos tablas de países? Al principio intenté usar una sola tabla DimCountry, pero en Power BI solo puedes tener una relación activa por tabla. La otra queda inactiva y tienes que activarla manualmente con USERELATIONSHIP() en cada medida DAX, lo cual es tedioso y propenso a errores. Con dos tablas separadas, ambas relaciones están siempre activas y las medidas son más simples.


### Dashboard (Power BI)

Creé 4 páginas:
1. **Overview**: Métricas generales y evolución temporal
2. **COVID Impact on Mobility Flows**: Scatter plot con cuadrantes (qué países ganaron/perdieron)
3. **Participant Profile**: Cambios demográficos
4. **Fields of Study**: Cambios en campos educativos

Las medidas DAX más importantes:
- `Recovery Rate = DIVIDE([Post-COVID Participants], [Pre-COVID Participants])`

- Medidas dinámicas que cambian según slicers con `SELECTEDVALUE()`

---

## Habilidades que demuestro

**Técnicas:**
- Python (Pandas, NumPy) para limpieza de datos a gran escala
- Power BI con DAX avanzado
- Modelado de datos (star schema, role-playing dimensions)
- Manejo de datos heterogéneos y sucios

**Analíticas:**
- Análisis exploratorio de datasets complejos
- Identificación de insights
- Comparaciones temporales (pre vs post)
- Interpretación de tendencias

**Comunicación:**
- Documentación técnica del proceso
- Visualizaciones claras y efectivas
- Explicación de decisiones metodológicas

---

## Limitaciones y aprendizajes

**Limitaciones conocidas:**
- Este análisis se basa únicamente en datos de movilidad de Erasmus+. Para explicar en profundidad algunos cambios (por qué Croatia creció tanto, qué políticas causaron el aumento de "fewer opportunities" en Alemania, etc.) haría falta complementar con otras fuentes de datos específicas por país

---

## Documentación disponible

| Archivo | Contenido |
|---------|-----------|
| [README.md](../../README.md) | Visión general y hallazgos |
| [PIPELINE.md](PIPELINE.md) | Proceso de limpieza paso a paso |
| [QUICK_START.md](QUICK_START.md) | Cómo reproducir el proyecto |
| [POWER_BI_GUIDE.md](POWER_BI_GUIDE.md) | Guía del dashboard |
| [DAX_MEASURES.md](DAX_MEASURES.md) | Medidas de Power BI |
| [DATA_DICTIONARY.md](DATA_DICTIONARY.md) | Descripción de variables |
| [NAVIGATION.md](NAVIGATION.md) | Navegación del proyecto |
| [PROJECT_STRUCTURE.md](PROJECT_STRUCTURE.md) | Estructura del repositorio |

---

## Contacto

**Andrés Liz Domínguez**

Si te interesa el proyecto o tienes preguntas, puedes contactarme o abrir un issue en el repositorio de GitHub.

---

**Última actualización**: Febrero 2026
