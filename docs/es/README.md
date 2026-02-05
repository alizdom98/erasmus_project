# Erasmus+ Mobility Analysis (2017-2024)

[Read in English](../../README.md) | [Leer en Español](README.md)

**Autor:** Andrés Liz Domínguez

Este proyecto analiza datos de movilidad del programa Erasmus+ para educación superior, con especial énfasis en cómo COVID-19 afectó a los patrones de movilidad estudiantil en Europa. El análisis cubre el período 2017-2024 y trabaja con aproximadamente 2.1 millones de registros de movilidades.

---

## Sobre el Proyecto

Este es un proyecto de portfolio que hice para demostrar lo que he aprendido sobre análisis de datos y visualización. El objetivo era tomar datos reales (y bastante sucios) del programa Erasmus+ y transformarlos en algo útil que pudiera analizarse en Power BI.

La idea surgió originalmente de mi TFG, pero lo he reenfocado completamente para mi portfolio. Me interesaba especialmente ver cómo cambió la movilidad estudiantil después de la pandemia: qué países ganaron o perdieron estudiantes, qué campos de estudio se volvieron más populares, y si el perfil del estudiante Erasmus cambió de alguna manera.

### Lo que he hecho

- Limpieza y normalización de aproximadamente 6 millones de registros repartidos en 11 archivos CSV diferentes
- Unificación de estructuras de datos que cambiaban entre años (las columnas no eran las mismas en todos los archivos)
- Creación de un modelo de datos en Power BI con tablas dimensión separadas
- Dashboard interactivo con 4 páginas de análisis
- Documentación del proceso completo (este repositorio)

### Herramientas utilizadas

- **Python y Pandas**: Para toda la limpieza de datos
- **Jupyter Notebook**: Para documentar el proceso paso a paso
- **Power BI**: Para el dashboard final y las visualizaciones
- **DAX**: Para crear las medidas y columnas calculadas en Power BI

---

## Estructura del Repositorio

```
erasmus_project/
│
├── README.md                   # Visión general (inglés)
├── data_erasmus.ipynb          # Notebook con todo el proceso (inglés)
├── data_erasmus_ES.ipynb       # Notebook en español
├── Proyecto_erasmus_bi.pbix    # Dashboard de Power BI
├── requirements.txt            # Librerías de Python que necesitas
│
├── bases_datos_erasmus/        # Datos originales (hay que descargarlos)
│   └── *.csv                   # Los 11 archivos CSV (2014-2024)
│
├── docs/
│   ├── en/                     # Documentación en inglés
│   │   ├── DATA_DICTIONARY.md
│   │   ├── DAX_MEASURES.md
│   │   ├── EXECUTIVE_SUMMARY.md
│   │   ├── NAVIGATION.md
│   │   ├── PIPELINE.md
│   │   ├── POWER_BI_GUIDE.md
│   │   ├── PROJECT_STRUCTURE.md
│   │   └── QUICK_START.md
│   └── es/                     # Documentación en español
│       ├── README.md
│       ├── DATA_DICTIONARY.md
│       ├── DAX_MEASURES.md
│       ├── EXECUTIVE_SUMMARY.md
│       ├── NAVIGATION.md
│       ├── PIPELINE.md
│       ├── POWER_BI_GUIDE.md
│       ├── PROJECT_STRUCTURE.md
│       └── QUICK_START.md
```

---

## Cómo empezar

### Requisitos previos

- Python 3.8 o superior
- Jupyter Notebook
- Power BI Desktop (si quieres ver el dashboard)

### Instalación

1. **Clona el repositorio:**
```bash
git clone https://github.com/alizdom98/erasmus_project.git
cd erasmus_project
```

2. **Instala las dependencias de Python:**
```bash
pip install -r requirements.txt
```

3. **Descarga los datos originales:**

Los datos NO están incluidos por su tamaño. Descárgalos desde:
- **Fuente oficial:** [Portal de Datos Abiertos de la UE](https://data.europa.eu/data/datasets?query=erasmus+KA1+mobility&locale=en)
- **Busca:** "Erasmus+ KA1 Mobility Data"
- **Archivos necesarios:** CSV de 2014 a 2024 (11 archivos en total)

**Colócalos en la carpeta:**
```
bases_datos_erasmus/
├── Erasmus-KA1-Mobility-Data-2014.csv
├── Erasmus-KA1-Mobility-Data-2015.csv
├── Erasmus-KA1-Mobility-Data-2016.csv
├── ...
└── Erasmus-KA1-Mobility-Data-2024.csv
```

**Tamaños aproximados:** Los archivos van de 14MB (2021) a 232MB (2019). Total: ~1.7GB

4. **Ejecuta el notebook** `data_erasmus.ipynb` para procesar los datos

5. **Abre el dashboard** `Proyecto_erasmus_bi.pbix` en Power BI Desktop

---

## Dataset

### Datos originales

Los datos vienen de la Comisión Europea y cubren todas las movilidades del programa Erasmus+ KA1 (Key Action 1) desde 2014 hasta 2024. En total son unos 6 millones de registros, pero yo me centré en educación superior (ISCED 6-8) para el período 2017-2024, lo que deja aproximadamente 2.1 millones de registros.

**Por qué solo 2017-2024:** Para el análisis Pre/Post-COVID no necesitaba remontarme tan atrás. Con 2017-2019 como período Pre-COVID era suficiente, y así reduje el tamaño de datos a analizar sin perder información relevante para el estudio.

**Por qué solo educación superior:** Filtré el dataset a ISCED 6-8 (educación superior) para tener un subset más manejable y enfocado. Los filtros adicionales (por ejemplo, solo Learners para ciertos análisis) se aplicaron directamente en Power BI según las necesidades de cada visual.

### Datos procesados

El dataset final tiene 2,082,071 registros y 26 columnas. Las principales variables incluyen:

**Temporales:**
- Año académico (formato "2019-20")
- Fecha de inicio de la movilidad
- Duración en días

**Geográficas:**
- País emisor y receptor (con código ISO2)
- País de origen del participante

**Demográficas:**
- Edad, género
- Si pertenece a grupos con "fewer opportunities" (situación desfavorecida)

**Académicas:**
- Nivel ISCED (6=Bachelor, 7=Master, 8=Doctorate)
- Campo de estudio (clasificación ISCED-F)

Para más detalles, ver [DATA_DICTIONARY.md](DATA_DICTIONARY.md).

---

## Principales hallazgos

### Recuperación casi completa postcovid

El programa Erasmus+ recuperó prácticamente todo su volumen de movilidades. Calculé un "Recovery Rate" que compara Post-COVID (2022-2024) con Pre-COVID (2017-2019), y sale 99.77%. Es decir, casi recuperamos el nivel anterior a la pandemia.

**Los números:**
- Pre-COVID (2017-2019): aproximadamente 2 millones de movilidades
- COVID (2020-2021): gran caída a 485,000 movilidades
- Post-COVID (2022-2024): recuperación a casi 2 millones

El año 2023-24 fue récord histórico con 950,000 movilidades en un solo año académico.

---

### Cambios geográficos importantes

#### Países que crecieron después de COVID

Los grandes ganadores fueron países del sur y este de Europa:

| País | Cambio | Observación |
|------|--------|-------------|
| Croatia | +37% | El mayor crecimiento relativo en recepción |
| Greece | +35% | Probablemente por clima y costos |
| Turkey | +32% | Programas cada vez más atractivos |
| Romania | +26% | Europa del Este ganando terreno |

**Caso especial - Ukraine:** Los envíos aumentaron un 137%. Esto claramente está relacionado con la guerra: muchos estudiantes ucranianos buscan estudiar fuera.

#### Países que perdieron después de COVID

| País | Cambio | Por qué |
|------|--------|---------|
| Russia | -90% | Guerra de Ucrania + sanciones internacionales |
| United Kingdom | -70% | Brexit (ya no participa en Erasmus+) |
| Polonia, Francia, Alemania | Caídas moderadas | Pierden cuota ante destinos emergentes |

**Lo interesante:** España, Francia y Alemania siguen siendo los destinos con más volumen en términos absolutos, pero están perdiendo cuota de mercado frente a destinos más económicos.

---

### Boom de tecnología, caída de humanidades

El cambio más llamativo en campos de estudio:

**Ganadores postcovid:**
- ICT (Informática): +41.3%
- Ciencias Naturales: +14.4%
- Agricultura: +10.4%
- Salud: +9.7%

**Perdedores postcovid:**
- Artes y Humanidades: -21.7%
- Servicios: -7.4%
- Business: -6.6%

**Mi interpretación:** La digitalización acelerada por la pandemia hizo que más gente se interesara por tecnología.

Business sigue siendo el campo con más volumen absoluto, pero está perdiendo atractivo relativo.

---

### Mayor inclusividad

Uno de los hallazgos más positivos: los participantes con "fewer opportunities" (grupos desfavorecidos) casi se duplicaron.

- Pre-COVID: 68,128 participantes
- Post-COVID: 149,444 participantes
- Aumento: +119%

**Alemania lidera este cambio:** contribuyó con +38,963 participantes más. No estoy completamente seguro de por qué, pero podría ser:
1. Cambios en políticas de inclusión alemanas
2. Mejor registro de estos datos
3. El impacto económico de COVID hizo que más estudiantes califiquen como "fewer opportunities"

---

### Movilidades más cortas

Después de COVID, las movilidades tienden a ser más cortas:

- Las movilidades de 6-9 meses bajaron notablemente
- Las de 3-6 meses siguen siendo las más comunes (un semestre académico)
- Las movilidades muy cortas (<3 meses) aumentaron relativamente

**Posibles razones:**
- Cautela económica (movilidades más cortas cuestan menos)
- Formatos intensivos (summer schools) ganando popularidad
- Menos gente comprometiéndose a un año completo fuera

---

### Predominio femenino constante

A diferencia de otros cambios, la distribución por género no cambió significativamente entre Pre y Post-COVID:
- Tanto Pre como Post-COVID mantienen aproximadamente 60% mujeres, 40% hombres
- Es remarcable que el género femenino predomina claramente en las movilidades Erasmus+
- COVID no parece haber afectado este equilibrio de género

---

## El Dashboard de Power BI

Creé un dashboard interactivo con 4 páginas principales:

### Página 1: Overview

Vista general con métricas clave y controles para explorar. Incluye:
- Cards con totales (Pre-COVID, COVID, Post-COVID, Recovery Rate)
- Mapa geográfico con burbujas por país
- Gráfico de evolución temporal 
- Ranking de top 10 países

Tiene slicers para cambiar entre ver datos de países que reciben estudiantes vs países que envían.

### Página 2: COVID Impact on Mobility Flows

La página más compleja. El visual principal es un scatter plot con cuatro cuadrantes:
- **Grow Both:** países que aumentaron tanto en envío como en recepción
- **Grow Rcv:** países que reciben más pero envían menos
- **Grow Snd:** países que envían más pero reciben menos
- **Down both:** países que bajaron en ambos

Los países se representan como burbujas (tamaño = volumen total pre-COVID) y se colorean según su cuadrante.

También hay gráficos de barras con los mayores aumentos y caídas.

### Página 3: Perfil del Participante

Compara la distribución demográfica Pre vs Post:
- Género (estable)
- Perfil Learner vs Staff (estable)
- Fewer opportunities (gran aumento)
- Duración de movilidades (más cortas postcovid)

### Página 4: Campos de Estudio

Tabla interactiva que muestra cambios por campo ISCED-F. Cuando seleccionas un campo, un gráfico lateral muestra los principales países emisores para ese campo.

Aquí es donde se ve claramente el boom de ICT (+41%) y la caída de Arts & Humanities (-22%).

---

## Modelo de datos en Power BI

Usé un modelo estrella (star schema) con:

**Tabla de hechos:**
- `df_he_pbi`: los 2.1 millones de registros de movilidad

**Tablas dimensión:**
- `Dim_Country_Sending`: países emisores
- `Dim_Country_Receiving`: países receptores
- `DimDate`: calendario con clasificación automática Pre/COVID/Post

**¿Por qué dos tablas de países?** Al principio intenté usar una sola tabla DimCountry, pero me di cuenta de que para hacer un análisis de flujos (Sending vs Receiving) necesitaba ambas relaciones activas simultáneamente. Con una sola tabla solo puedes tener una relación activa, la otra queda inactiva y tienes que activarla con `USERELATIONSHIP()` en cada medida, lo cual es un lío.

**Tablas auxiliares sin relaciones:**
- `AxisCountry`: un "eje virtual" que une todos los países para análisis transversales
- `Direction`: para el slicer de Receiving/Sending

Estas tablas no tienen relaciones físicas pero se usan con `TREATAS()` y `SELECTEDVALUE()` para hacer medidas dinámicas.

Para más detalles técnicos, ver [POWER_BI_GUIDE.md](POWER_BI_GUIDE.md) y [DAX_MEASURES.md](DAX_MEASURES.md).

---

## Limitaciones y decisiones metodológicas

### Definición de períodos COVID

Definí tres períodos:
- **Pre-COVID:** 2017-2019 (años completamente normales)
- **COVID:** 2020-2021 (restricciones activas)
- **Post-COVID:** 2022 en adelante

**¿Por qué 2021 está en COVID y no en Post?** Porque en 2021 todavía había restricciones en muchos países. La movilidad no se normalizó completamente hasta 2022.

### Filtro de volumen mínimo

En el análisis de cuadrantes, excluí países con menos de 2,000 movilidades en Pre-COVID. ¿Por qué? Porque países muy pequeños tienen porcentajes de cambio muy volátiles y poco representativos. Por ejemplo, si un país pasa de 10 a 30 movilidades, es +200% pero no es realmente significativo.

### Interpretación de "Fewer Opportunities"

El aumento del 119% puede tener múltiples causas:
1. Inclusión real (más becas, más apoyo)
2. Cambios en cómo se registra este dato
3. Impacto económico de COVID (más estudiantes califican)

Sin datos cualitativos de la Comisión Europea, es difícil saber cuál es la causa principal.

### Datos de 2024

Si el dataset se generó a mitad de 2024, los datos de ese año podrían estar incompletos. Hay un sesgo estacional: septiembre siempre tiene más movilidades que enero.

---

## Recursos adicionales

- [PIPELINE.md](PIPELINE.md): Explicación paso a paso de cómo limpié los datos
- [QUICK_START.md](QUICK_START.md): Guía rápida para reproducir el proyecto
- [POWER_BI_GUIDE.md](POWER_BI_GUIDE.md): Guía completa del dashboard con interpretaciones
- [DAX_MEASURES.md](DAX_MEASURES.md): Todas las medidas de Power BI explicadas
- [DATA_DICTIONARY.md](DATA_DICTIONARY.md): Descripción de las 26 variables finales
- [EXECUTIVE_SUMMARY.md](EXECUTIVE_SUMMARY.md): Resumen ejecutivo del proyecto
- [NAVIGATION.md](NAVIGATION.md): Navegación del proyecto
- [PROJECT_STRUCTURE.md](PROJECT_STRUCTURE.md): Estructura del repositorio

---

## Datos

Los datos originales son de la Comisión Europea y están bajo licencia Creative Commons Attribution 4.0 International (CC BY 4.0).

Fuente: https://data.europa.eu/
Más info: https://creativecommons.org/licenses/by/4.0/

---

## Contacto

Andrés Liz Domínguez

Si tienes preguntas sobre el proyecto o encuentras algún error, abre un issue en GitHub.
