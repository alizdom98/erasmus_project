# Pipeline de Limpieza de Datos - Proceso Completo

[Read in English](../en/PIPELINE.md) | [Leer en Español](PIPELINE.md)

Este documento explica paso a paso cómo limpié los datos del programa Erasmus+. Intenté documentar no solo qué hice, sino también por qué tomé cada decisión.

---

## Contexto

Los datos originales vienen de 11 archivos CSV (uno por año, 2014-2024) publicados por la Comisión Europea. En total son unos 6 millones de registros, pero había varios problemas:

1. Las columnas no tenían los mismos nombres en todos los años
2. Algunos años tenían 20 columnas, otros 19
3. Los encodings variaban (UTF-8, UTF-8-BOM, Latin1...)
4. Había muchos duplicados, outliers y valores raros
5. Los nombres de países no eran consistentes

El objetivo era transformar todo esto en un dataset limpio y utilizable.

---

## Fase 1: Verificación de Estructura

**Lo primero que hice** fue ver qué diferencias había entre los archivos antes de cargarlos todos. No quería encontrarme con sorpresas a mitad del proceso.

### El problema

Cada año podía tener:
- Nombres de columnas diferentes (`organisation` vs `organization`)
- Columnas que aparecen en unos años y en otros no
- Formatos diferentes (por ejemplo, fechas como `"2019/09"` vs `"2019-09"`)

### Lo que hice

Creé una función que:
1. Lee solo las cabeceras de cada CSV (sin cargar todos los datos)
2. Normaliza los nombres de columnas (minúsculas, guiones bajos)
3. Compara las columnas entre años

```python
def normalize_columns(columns):
    normalized = []
    for col in columns:
        col = col.strip().lower()
        col = re.sub(r"\s+", " ", col)  # Colapsar espacios múltiples
        col = col.replace(" ", "_")     # Espacios a guiones bajos
        normalized.append(col)
    return normalized
```

### Qué encontré

- **Años 2014-2019**: 20 columnas
- **Años 2020-2024**: 19 columnas (falta `project_reference`)
- Cambios de nombres:
  - `sending_organisation` → `sending_organization` (británico a americano)
  - `mobility_duration_-_calendar_days` → `mobility_duration` (simplificado)
  - `mobility_start_year/month` → `mobility_start_month` (formato más limpio)

**Por qué esto es importante:** Si no lo hubiera detectado antes, me habría encontrado con errores extraños al intentar unir los archivos.

---

## Fase 2: Carga y Unificación

Ahora sí, cargar todos los archivos y unirlos en uno solo.

### El problema del encoding

No todos los archivos usaban el mismo encoding. Algunos eran UTF-8, otros UTF-8-BOM (con marca de orden de bytes), y otros Latin1. Si usas el encoding equivocado, los caracteres especiales (tildes, ñ) salen mal.

### Solución

Creé una función que prueba varios encodings hasta que uno funcione:

```python
def read_csv_with_fallback(path, sep=";"):
    encodings = ["utf-8", "utf-8-sig", "latin1"]
    for enc in encodings:
        try:
            df = pd.read_csv(path, sep=sep, encoding=enc)
            return df, enc
        except:
            continue
    raise Exception("No se pudo leer el archivo con ningún encoding")
```

### Unificación de columnas

Como las columnas no eran iguales entre años, hice esto:

1. Normalicé los nombres con la función de antes
2. Apliqué un "mapa de renombrado" para unificar variantes:
   ```python
   RENAME_MAP = {
       "sending_organisation": "sending_organization",
       "mobility_duration_-_calendar_days": "mobility_duration",
       # etc.
   }
   ```
3. Eliminé columnas basura tipo "Unnamed: 0" que a veces aparecían
4. Añadí una columna `source_file_year` para saber de qué año venía cada registro
5. Alineé todas las columnas (si falta una columna en un año, la relleno con NaN)
6. Concatené todo en un solo DataFrame

### Resultado

**5,955,075 registros** con **21 columnas** unificadas.

---

## Fase 3: Normalización Temporal

Los datos de fechas estaban en formatos inconsistentes y necesitaba estandarizarlos.

### academic_year

El año académico venía en dos formatos:
- `"2020-2021"` (completo)
- `"2020-21"` (reducido)

Necesitaba que todos siguieran el mismo formato. Elegí `"YYYY-YY"` porque es el formato oficial de Erasmus+.

```python
def norm_academic_year(x):
    if pd.isna(x):
        return pd.NA
    s = str(x).strip()

    # "2020-2021" → "2020-21"
    m = re.fullmatch(r"(\d{4})-(\d{4})", s)
    if m:
        return f"{m.group(1)}-{m.group(2)[-2:]}"

    # "2020-21" → dejar igual
    return s
```

### mobility_start_month

Esta columna venía como texto (`"2019-09"`) pero la necesitaba como fecha para poder:
- Filtrar por fechas
- Extraer el año y mes por separado
- Hacer series temporales en Power BI

```python
df_total["mobility_start_ym"] = pd.to_datetime(
    df_total["mobility_start_month"],
    format="%Y-%m",
    errors="coerce"
)

df_total["mobility_start_year"] = df_total["mobility_start_ym"].dt.year
df_total["mobility_start_month_num"] = df_total["mobility_start_ym"].dt.month
```


---

## Fase 4: Limpieza de Duplicados

Encontré 9,165 registros duplicados exactos (0.15% del total). Son filas idénticas en todas las columnas.

### ¿Son todos errores?

Analicé dos tipos de "duplicados":
1. **Duplicados exactos**: Misma fila repetida → Claramente un error
2. **Repetidos multi-año**: Misma información pero en diferentes años académicos → Podría ser legítimo (un estudiante que va dos veces)

Encontré solo 96 casos del segundo tipo, que decidí mantener porque podrían ser reales.

### Lo que hice

```python
df_total = df_total.drop_duplicates(keep="first").reset_index(drop=True)
```

Mantuve la primera aparición de cada duplicado.

**Resultado:** 5,945,910 registros (eliminé 9,165).

---

## Fase 5: Limpieza de Variables Numéricas

Aquí encontré muchos valores imposibles que necesitaban limpiarse.

### actual_participants

Encontré UN registro con valor `4.7`. No puedes tener 4.7 participantes. También había algunos con valor `0`.

Solución: Convertir a NA valores con decimales o igual a 0, y usar tipo `Int64` (nullable integer) en lugar de float.

### participant_age

**Problemas encontrados:**
- 82,785 registros con edad ≤ 0 (incluyendo edades negativas como `-1`, `-2`)
- 16,993 registros con edad ≥ 65 (incluyendo outliers extremos como `1922`, `823`)

**Por qué existen estos valores:**
- Negativos probablemente son códigos de "no especificado" del sistema fuente
- `0` indica dato no disponible
- Valores extremos son errores de entrada (alguien escribió el año en lugar de la edad)

**Solución:** Definí un rango razonable (10-65 años) basándome en que:
- Mínimo 10: Erasmus+ incluye algunas movilidades escolares (aunque son minoritarias)
- Máximo 65: Límite razonable para estudiantes universitarios

```python
age = pd.to_numeric(df_total["participant_age"], errors="coerce")
age = age.mask((age < 10) | (age > 65))
df_total["participant_age"] = age.astype("Int64")
```

**Resultado:** 621,084 valores nulos (10.5% del dataset), pero los que quedan son razonables.

### mobility_duration

16,069 registros tenían duración = 0 días, lo cual no tiene sentido. Los convertí a NA.

También vi duraciones muy largas (>730 días, o sea >2 años), pero las dejé porque pueden ser legítimas (programas de doctorado conjunto, dobles titulaciones).

**Estadísticas después de limpiar (dataset HE 2017-2024):**
- Mínimo: 1 día
- Mediana: 130 días (aprox. 4 meses, un semestre académico)
- Media: 129.3 días
- Máximo: 845 días (aprox. 2.3 años)

---

## Fase 6: Normalización de Valores Categóricos

En las columnas de texto había muchos valores que en realidad significaban "desconocido" pero estaban escritos de formas diferentes.

### Tokens de nulos

Encontré estas variantes:
- `"unknown"`, `"not specified"`, `"none"`, `"n/a"`, `"na"`
- `"?"`, `"??"`, `"???"`, `"????"`
- `"-"`, `"--"`, `"---"`
- `"_"`, `"__"`
- Cadenas vacías

**Lo que hice:** Convertir todos estos valores a `pd.NA` (nulo oficial de pandas).

```python
missing_tokens = {"unknown", "not specified", "none", "n/a", "na",
                  "?", "??", "???", "-", "--", "_", "__"}

for col in text_cols:
    s = df_total[col].str.strip().str.lower()
    df_total[col] = s.mask((s == "") | (s.isin(missing_tokens)), pd.NA)
```

**Caso especial:** En `receiving_city` aparecía el string `"0"` que claramente no es una ciudad. También lo convertí a NA.

### participant_profile

Valores en diferentes formatos: `"LEARNERS"` vs `"Learner"` vs `"STAFF"` vs `"Staff"`.

Unifiqué a:
- `"Learner"` para estudiantes
- `"Staff"` para personal

Y convertí la columna a tipo `category` para ahorrar memoria.

### fewer_opportunities

Venía codificada como `0`/`1` (a veces texto, a veces número). La convertí a `"No"`/`"Yes"` para que sea más legible.

**¿Qué significa "fewer opportunities"?** Son estudiantes con situación desfavorecida: económica, discapacidad, áreas rurales, migrantes/refugiados, etc.

---

## Fase 7: Enriquecimiento de Datos

Aquí creé nuevas variables a partir de las existentes.

### ISCED Level y Groups

La columna `education_level` tenía textos largos como:
```
"ISCED-6 - First cycle / Bachelor's degree or equivalent (EQF 6)"
```

Extraje solo el número ISCED:

```python
df_total["isced_level"] = df_total["education_level"]\
    .str.extract(r"ISCED-(\d)", expand=False)\
    .astype("Int64")
```

Y creé grupos simplificados:
- **HE (6-8)**: Educación superior (Bachelor, Master, Doctorate)
- **Pre-tertiary (1-5)**: Niveles previos
- **ISCED-9 / Other**: Otros

**Resultado:** 50.8% del dataset (3.02M) es educación superior.

### Activity Groups

La columna `activity_mob` tenía más de 40 tipos diferentes. Los agrupé en 6 categorías amplias:

- **HE** (Higher Education): Movilidad de estudiantes universitarios
- **VET** (Vocational Education): Formación profesional
- **Youth/Volunteering**: Intercambios juveniles, voluntariado
- **Staff/Training**: Movilidad de personal
- **School**: Movilidad escolar
- **Adult**: Educación de adultos

Usé tanto códigos (cuando existían) como palabras clave del texto para clasificar.

### ISCED-F Broad Fields

La columna `field_of_education` tenía códigos de 4 dígitos como `"0410 - Business and administration"`. Los reduje a 2 dígitos para tener categorías amplias:

- **00**: Programas genéricos
- **01**: Educación
- **02**: Artes y humanidades
- **03**: Ciencias sociales
- **04**: Business, administración, derecho
- **05**: Ciencias naturales, matemáticas
- **06**: ICT
- **07**: Ingeniería, manufactura
- **08**: Agricultura, veterinaria
- **09**: Salud y bienestar social
- **10**: Servicios
- **99**: No clasificado

Para los registros sin código, usé palabras clave del texto para clasificarlos.

---

## Fase 8: Normalización Geográfica

Esta fue una de las partes más complicadas.

### El problema con los países

El mismo país podía aparecer escrito de muchas formas:
- `"Türkiye"` vs `"Turkey"`
- `"Czechia"` vs `"Czech Republic"`
- `"Iran (Islamic Republic of)"` vs `"Iran"`
- En sending/receiving: `"ES - Spain"` vs `"ES - España"` (mismo código, nombre diferente)

**Por qué es un problema:** En Power BI, si un país tiene 2 nombres diferentes, aparece dos veces en gráficos y tablas, fragmentando los datos.

### Estrategia de normalización

**Para `participant_country`** (solo nombre, sin código):
Creé un diccionario manual de equivalencias:

```python
name_map = {
    "Türkiye": "Turkey",
    "Czechia": "Czech Republic",
    "Iran (Islamic Republic of)": "Iran",
    # ...30+ variantes más
}
```

**Para `sending_country` y `receiving_country`** (formato "XX - Name"):
Normalicé por código ISO2:

1. Extraje el código (XX) y el nombre por separado
2. Para cada código, elegí el nombre más frecuente como "canónico"
3. Apliqué este nombre canónico a todos los registros de ese código

```python
# Ejemplo: todos los registros con código "ES" usan "Spain" (el más frecuente)
canon_name = df.groupby("code")["name"]\
    .agg(lambda x: x.value_counts().idxmax())
```

4. Creé columnas separadas para código y nombre:
   - `sending_country_code`
   - `sending_country_name`
   - (y lo mismo para receiving)

**¿Por qué separar código y nombre?** En Power BI:
- El código (ISO2) se usa para relaciones y mapas (Power BI lo reconoce automáticamente)
- El nombre se usa para visualización (más legible para el usuario)

**Estándar global:** Aseguré que sending y receiving usen el mismo nombre canónico por código, para que "ES" sea siempre "Spain" en ambas columnas.

### El problema con las ciudades

Las ciudades tenían varios problemas:
- Espacios extra
- Sufijos postales como "Lyon CEDEX 07"
- Distritos: "Paris 16", "Dublin 2"
- Tildes inconsistentes: "Málaga" vs "Malaga"
- Exónimos: "Praha" vs "Prague", "Wien" vs "Vienna"
- **Mojibake**: Caracteres mal codificados tipo "Cefal�" en lugar de "Cefalù"

**Lo que hice (limpieza básica):**
- Normalicé espacios
- Quité sufijos CEDEX y números de distrito
- Eliminé tildes (normalización Unicode)
- Unifiqué exónimos comunes a inglés
- Capitalicé correctamente (Title Case)

**Lo que NO hice:**
- No arreglé el mojibake porque ya está en los datos originales y no hay forma de saber el carácter correcto con certeza (afecta a ~1.4% de los registros)
- No normalicé al 100% el nombre de las ciudades - una buena cantidad están mal escritas
- Una normalización completa requeriría análisis país por país, lo cual tomaría mucho tiempo

**Resultado:**
- 255 países únicos en `participant_country`
- 0 códigos con nombres múltiples en sending/receiving (100% normalizado)
- 94,344 ciudades únicas en receiving_city
- 46,021 ciudades únicas en sending_city

**Nota importante:** Por estas limitaciones en la calidad de los nombres de ciudades (misspellings, mojibake, alta cardinalidad), las ciudades NO fueron incluidas en el análisis final de Power BI. El análisis se centra en países, que sí están completamente normalizados.

---

## Fase 9: Preparación del Dataset Final

Ya con todo limpio, preparé el subset específico para mi análisis.

### Filtros aplicados

**Por período:** Solo 2017-2024

**¿Por qué no incluir 2014-2016?**
Para el análisis Pre vs Post COVID no necesitaba remontarme tan atrás. Con 2017-2019 como período Pre-COVID era suficiente, y así pude reducir el tamaño de datos a analizar sin perder información relevante para el estudio.

**Por tipo de educación:** Solo ISCED 6-8 (educación superior)

**¿Por qué este criterio?**

Comparé varios criterios para definir qué es "Educación Superior":

| Criterio | Registros | % sobre 2017-2024 |
|----------|-----------|-------------------|
| **ISCED 6-8** (criterio base elegido) | 2,082,071 | 49.80% |
| activity_group == "HE" | 2,058,366 | 49.23% |
| field == "Higher Education" | 2,312,575 | 55.31% |
| **ISCED 6-8 AND activity_group == "HE"** (estricto) | 1,887,691 | 45.15% |

**Decisión:** Elegí **ISCED 6-8** como criterio base porque:
- Es el estándar internacional más robusto (UNESCO)
- Incluye Bachelor (6), Master (7) y Doctorate (8)
- Cubre movilidades HE de estudiantes y staff en contexto universitario

También creé una bandera `he_strict` para permitir análisis más conservadores:

```python
he_strict = (ISCED 6-8) AND (activity_group = "HE")  # Solo estudiantes HE, excluye staff
```

Esta bandera permite filtrar a solo estudiantes (excluye staff mobility) si se necesita un análisis más estricto. Los filtros adicionales (por ejemplo, solo Learners) se aplicaron directamente en Power BI según las necesidades de cada visual.

### Selección de columnas

De las 37 columnas que tenía en ese punto, seleccioné solo 26 para el análisis final:

**Incluí:**
- Todas las variables temporales
- Códigos y nombres de países (por separado)
- Variables demográficas
- Clasificaciones educativas (ISCED, campos de estudio)
- Métricas (duración, edad, participantes)
- Flag he_strict

**Excluí:**
- Ciudades (demasiadas categorías únicas + problemas de mojibake)
- Organizaciones (nombres largos e inconsistentes)
- project_reference (41% missing)
- Columnas de texto largo ya procesadas (education_level, field_of_education, activity_mob)
- Variables técnicas (source_file_year)

### Dataset Final

**2,082,071 registros** × **26 columnas**

Tamaño en memoria: ~413 MB
Tamaño exportado (Parquet): ~80 MB
Tamaño exportado (CSV): ~400 MB

---

## Resumen del Pipeline

| Fase | Input | Output | Cambio |
|------|-------|--------|--------|
| 1. Verificación | 11 CSVs | Reporte de diferencias | Análisis de estructura |
| 2. Carga | 5,955,075 registros | df_total unificado | +source_file_year |
| 3. Temporal | Formatos mixtos | Fechas normalizadas | academic_year unificado |
| 4. Duplicados | 9,165 duplicados | 5,945,910 registros | -0.15% |
| 5. Numéricas | Outliers | Valores limpios | 621K edades a NA |
| 6. Categóricas | Tokens mixtos | Categorías unificadas | Nulos estandarizados |
| 7. Enriquecimiento | Texto largo | Clasificaciones | +8 variables |
| 8. Geografía | Nombres inconsistentes | ISO2 + canónicos | 255 países normalizados |
| 9. Subset HE | 5.9M mixtos | 2.1M HE (2017-2024) | Filtrado ISCED 6-8 |

---

## Lecciones aprendidas

### Lo que funcionó bien

- **Verificar antes de cargar**: Revisar las estructuras primero me ahorró muchos dolores de cabeza
- **Funciones con fallback**: La función que prueba múltiples encodings fue esencial
- **Documentar decisiones**: Escribir el "por qué" me ayudó cuando volví al código semanas después
- **Crear variables intermedias**: Mantener tanto mobility_start_month (texto) como mobility_start_ym (fecha) dio flexibilidad

### Lo que fue más difícil de lo esperado

- **Normalización de países**: Pensé que sería simple pero hay muchas variantes y casos especiales
- **Decidir qué es un outlier**: No siempre está claro. ¿Una duración de 800 días es error o es real? Tuve que investigar el programa para entender qué valores eran posibles
- **Balance entre limpiar y preservar**: A veces no estaba seguro si un valor raro era error o dato real. En casos de duda, preferí convertir a NA en lugar de eliminar el registro completo

### Si lo hiciera de nuevo

- Haría el análisis exploratorio (EDA) más temprano para entender mejor los datos antes de limpiar
- Probaría con una muestra pequeña primero (ej. solo un año) antes de procesar todo
- Documentaría los casos especiales y excepciones en un archivo separado

---

## Referencias

- **ISCED 2011**: http://uis.unesco.org/en/topic/international-standard-classification-education-isced
- **ISCED-F 2013**: http://uis.unesco.org/en/topic/international-standard-classification-education-isced
- **ISO 3166-1 alpha-2**: https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2
- **Erasmus+ Programme Guide**: https://erasmus-plus.ec.europa.eu/programme-guide

---

**Autor**: Andrés Liz Domínguez
**Fecha**: Febrero 2026
