#  Data Dictionary - Erasmus+ HE Dataset (2017-2024)

[Read in English](../en/DATA_DICTIONARY.md) | [Leer en Español](DATA_DICTIONARY.md)

Este documento describe todas las variables del dataset final preparado para análisis en Power BI.

##  Descripción General

- **Dataset**: `erasmus_he_2017_2024.parquet` / `.csv`
- **Registros**: 2,082,071
- **Columnas**: 21
- **Período**: 2017-2024 (años de inicio de movilidad)
- **Enfoque**: Educación Superior (ISCED 6-8)

---

##  Variables por Categoría

###  Variables Temporales

| Variable | Tipo | Valores | Descripción |
|----------|------|---------|-------------|
| `academic_year` | string | "2017-18" ... "2024-25" | Año académico en formato "YYYY-YY". Representa el curso académico completo durante el cual ocurrió la movilidad. |
| `year_start` | Int64 | 2017-2024 | Año numérico de inicio de la movilidad. Extraído del academic_year (por ejemplo, "2019-20" → 2019). Útil para filtros y series temporales. |
| `mobility_start_ym` | datetime | 2017-01 ... 2024-12 | Fecha exacta (año-mes) de inicio de la movilidad en formato datetime. Permite análisis de series temporales detalladas. |
| `mobility_start_year` | Int64 | 2017-2024 | Año de inicio de la movilidad (duplicado de year_start, mantenido por compatibilidad). |
| `mobility_start_month_num` | Int64 | 1-12 | Mes de inicio de la movilidad (1=enero, 12=diciembre). Septiembre (9) es el mes más común (712,470 movilidades, 34.22%). |

**Notas sobre temporalidad**:
- El **año académico** puede extenderse entre dos años naturales (por ejemplo, 2019-20 incluye septiembre 2019 a agosto 2020).
- El **año de inicio** (year_start) es el año natural cuando comienza la movilidad.
- Para análisis de tendencias, usar `year_start` o `mobility_start_year`.
- Para análisis estacional, usar `mobility_start_month_num`.

---

###  Variables Geográficas

#### Países Emisores (Sending)

| Variable | Tipo | Valores | Descripción |
|----------|------|---------|-------------|
| `sending_country_code` | category | "ES", "FR", "DE", ... | Código ISO2 del país emisor (institución que envía al estudiante). 33 códigos únicos. |
| `sending_country_name` | category | "Spain", "France", "Germany", ... | Nombre canónico del país emisor en inglés. Normalizado por código ISO2 para evitar variantes. |

**Formato completo**: En el dataset original aparece como `"ES - Spain"`, pero aquí está separado en dos columnas para facilitar análisis y joins.

#### Países Receptores (Receiving)

| Variable | Tipo | Valores | Descripción |
|----------|------|---------|-------------|
| `receiving_country_code` | category | "ES", "FR", "DE", ... | Código ISO2 del país receptor (institución que recibe al estudiante). 33 códigos únicos. |
| `receiving_country_name` | category | "Spain", "France", "Germany", ... | Nombre canónico del país receptor en inglés. Normalizado por código ISO2. |

**Top 5 países receptores (2017-2024)**:
1. España (ES)
2. Francia (FR)
3. Alemania (DE)
4. Italia (IT)
5. Portugal (PT)

#### País del Participante

| Variable | Tipo | Valores | Descripción |
|----------|------|---------|-------------|
| `participant_country` | category | "Spain", "Turkey", "Poland", ... | País de origen/nacionalidad del participante. 255 valores únicos (incluye territorios no-UE). Sin código ISO2 en el dataset original. |

**Diferencia con sending_country**:
- `sending_country`: País de la institución emisora (puede ser diferente del país de origen del estudiante)
- `participant_country`: Nacionalidad del participante
- Ejemplo: Un estudiante polaco en una universidad alemana que va a Francia tendría:
  - participant_country: "Poland"
  - sending_country: "Germany"
  - receiving_country: "France"

---

###  Variables Demográficas

| Variable | Tipo | Valores | Descripción | Missing |
|----------|------|---------|-------------|---------|
| `participant_age` | Int64 | 10-65 | Edad del participante en años. Valores fuera del rango 10-65 fueron convertidos a NA (outliers imposibles como edades negativas o >100). | 9.69% |
| `participant_gender` | string | "Female", "Male", "Undefined", NA | Género del participante. | 0.00% |
| `participant_profile` | category | "Learner", "Staff", "Other", NA | Perfil del participante. **Learner** (90.28%): Estudiante. **Staff** (9.72%): Personal docente/administrativo. | 0.1% |
| `fewer_opportunities` | category | "Yes", "No", NA | Indica si el participante pertenece a un grupo con **menos oportunidades** (fewer opportunities background) según criterios de Erasmus+: situación económica desfavorable, discapacidad, área rural remota, migrante/refugiado, etc. | 0.81% |

**Distribución de género (datos disponibles)**:
- Female: 59.60%
- Male: 40.27%
- Undefined: 0.13%

**¿Qué es "fewer opportunities"?**
Es una categoría de la Comisión Europea para participantes que enfrentan barreras adicionales:
- Desventaja económica
- Discapacidad física, mental o de salud
- Dificultades educativas
- Diferencias culturales (migrantes, refugiados)
- Ubicación geográfica (áreas rurales remotas)

---

###  Variables Educativas

#### Nivel ISCED

| Variable | Tipo | Valores | Descripción |
|----------|------|---------|-------------|
| `isced_level` | Int64 | 6, 7, 8 | Nivel educativo según clasificación **ISCED 2011** (UNESCO): **6** = Bachelor (grado/licenciatura), **7** = Master (máster), **8** = Doctorate (doctorado). |
| `isced_group` | category | "HE (6-8)" | Grupo simplificado. En este dataset solo aparece "HE (6-8)" porque ya está filtrado por educación superior. |

**Distribución por nivel**:
- ISCED 6 (Bachelor): 62.49% de movilidades HE
- ISCED 7 (Master): 33.77%
- ISCED 8 (Doctorate): 3.73%

#### Campo de Estudio

| Variable | Tipo | Valores | Descripción |
|----------|------|---------|-------------|
| `isced_macro` | category | "01 - Education", "02 - Arts and humanities", ... | Campo de estudio según clasificación **ISCED-F 2013** (broad fields, 2 dígitos). 11 categorías amplias + "Not classified". |

**Categorías ISCED-F (Broad Fields)**:
- **00**: Generic programmes and qualifications
- **01**: Education (formación de profesores)
- **02**: Arts and humanities
- **03**: Social sciences, journalism and information
- **04**: Business, administration and law
- **05**: Natural sciences, mathematics and statistics
- **06**: Information and Communication Technologies (ICT)
- **07**: Engineering, manufacturing and construction
- **08**: Agriculture, forestry, fisheries and veterinary
- **09**: Health and welfare
- **10**: Services (turismo, deportes, servicios personales)
- **99**: Not classified

**Top 5 campos de estudio**:
1. **04 - Business, administration and law** (22.80%)
2. **02 - Arts and humanities** (18.44%)
3. **07 - Engineering, manufacturing and construction** (13.90%)
4. **03 - Social sciences, journalism and information** (13.12%)
5. **09 - Health and welfare** (7.40%)

#### Tipo de Actividad

| Variable | Tipo | Valores | Descripción |
|----------|------|---------|-------------|
| `activity_group` | category | "HE", "Staff/Training", "Youth/Volunteering", "VET", "School", "Adult" | Tipo de actividad de movilidad agrupada. Para este dataset HE, la mayoría es **"HE"** (student mobility for studies/traineeships). |

**Valores en este dataset**:
- **HE** (90.66%): Student Mobility for Studies (SMS) o Traineeships (SMT) en educación superior
- **Staff/Training** (6.16%): Movilidad de personal docente/administrativo (aunque no son estudiantes, están en contexto HE)
- **Other** (3.18%): Otras actividades en contexto de educación superior

---

###  Variables Métricas

| Variable | Tipo | Valores | Descripción | Missing |
|----------|------|---------|-------------|---------|
| `mobility_duration` | float64 | 1-845 | Duración de la movilidad en **días naturales**. Valores = 0 fueron convertidos a NA. | 0.38% |
| `actual_participants` | Int64 | 1-... | Número de participantes para registros agregados. En la mayoría de los casos = 1 (un registro por participante). Valores con decimales o = 0 fueron convertidos a NA. | ~0% |

**Estadísticas de duración**:
- **Mínimo**: 1 día (visitas preparatorias muy cortas)
- **Mediana**: 130 días (~4 meses, duración típica de un semestre académico)
- **Media**: 129.3 días (distribución casi simétrica centrada en un semestre)
- **Máximo**: 845 días (~2.3 años, probablemente programas de doctorado o dobles titulaciones)

**Rangos típicos de duración**:
- **1-90 días**: Movilidades cortas (verano, intensivos, visitas)
- **90-180 días**: Semestre académico (~3-6 meses) - rango más frecuente
- **180-365 días**: Año académico completo o programas largos
- **>365 días**: Programas de doctorado, dobles titulaciones (muy minoritarios)

---

###  Indicadores y Variables de Control

| Variable | Tipo | Valores | Descripción |
|----------|------|---------|-------------|
| `he_strict` | boolean | True, False | Flag conservador que indica si el registro cumple criterio estricto de HE: **ISCED 6-8 AND activity_group = "HE"**. True para 90.66% del dataset. False incluye staff mobility en instituciones HE y casos edge. |

**¿Cuándo usar he_strict?**
- **False (todo el dataset)**: Análisis amplio de movilidades relacionadas con educación superior (incluye staff)
- **True (filtro estricto)**: Análisis conservador de movilidad **solo de estudiantes** de grado, máster y doctorado

---

##  Casos de Uso por Variable

### Análisis Temporal
- `year_start`, `academic_year`: Tendencias anuales, análisis pre/postcovid
- `mobility_start_month_num`: Estacionalidad (septiembre es pico)
- `mobility_start_ym`: Series temporales detalladas

### Análisis Geográfico
- `sending_country_code`, `receiving_country_code`: Flujos de movilidad entre países
- `participant_country`: Nacionalidades más móviles
- Mapas en Power BI: Usar `_code` (ISO2) para reconocimiento automático

### Análisis Demográfico
- `participant_age`: Distribución etaria (pico en 20-24 años)
- `participant_gender`: Balance de género en movilidad
- `fewer_opportunities`: Inclusividad del programa

### Análisis Educativo
- `isced_level`: Comparar Bachelor vs Master vs Doctorate
- `isced_macro`: Popularidad de campos de estudio
- `activity_group` + `he_strict`: Diferenciar estudiantes vs staff

### Análisis de Impacto COVID
- Comparar `year_start` 2017-2019 (pre) vs 2022-2024 (post)
- Filtrar por país receptor para ver qué destinos se recuperaron más rápido

---

##  Limitaciones Conocidas

### Missing Values (% del dataset)

| Variable | Missing | Motivo |
|----------|---------|--------|
| `participant_age` | 9.69% | Dato opcional en el formulario + outliers convertidos a NA |
| `isced_macro` | 1.88% | Campo de estudio no siempre se registra |
| `fewer_opportunities` | 0.81% | No siempre se declara |
| `mobility_duration` | 0.38% | Valores = 0 convertidos a NA |
| `participant_gender` | 0.00% | Dato completo en dataset HE 2017-2024 |

### Ciudades Excluidas

Las variables `sending_city` y `receiving_city` **NO están incluidas** en este dataset final por:
- Demasiadas categorías únicas (46K y 94K respectivamente)
- Problemas de encoding (mojibake) en ~1.4% de registros
- Baja utilidad para análisis agregado en Power BI

Si se necesita análisis por ciudad, se puede recuperar del dataset intermedio `df_he` (celda 27 del notebook).

### Período Excluido

Años 2014-2016 fueron excluidos porque para el análisis Pre/Post-COVID no era necesario remontarse tan atrás. Con 2017-2019 como período Pre-COVID era suficiente, lo que permitió reducir el tamaño de datos a analizar sin perder información relevante.

---

##  Referencias

- **ISCED 2011**: [UNESCO - International Standard Classification of Education](http://uis.unesco.org/en/topic/international-standard-classification-education-isced)
- **ISCED-F 2013**: [UNESCO - Fields of Education and Training](https://uis.unesco.org/en/topic/isced-fields-education-and-training)
- **ISO 3166-1 alpha-2**: [Códigos de país de dos letras](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2)
- **Erasmus+ Programme Guide**: [European Commission](https://erasmus-plus.ec.europa.eu/programme-guide)

---

**Última actualización**: Febrero 2026
**Versión del dataset**: v1.0
