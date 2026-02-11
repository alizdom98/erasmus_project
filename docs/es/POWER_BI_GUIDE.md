# Guía del Dashboard de Power BI

[Read in English](../en/POWER_BI_GUIDE.md) | [Leer en Español](POWER_BI_GUIDE.md)

Este documento explica cómo funciona el dashboard que creé en Power BI y las decisiones que tomé en el modelo de datos.

---

## Estructura General

El dashboard tiene 4 páginas principales, cada una enfocada en un aspecto diferente del análisis:

1. **Overview**: Vista general interactiva
2. **COVID Impact on Mobility Flows**: Análisis de cambios en flujos emisores y receptores por país (scatter plot de cuadrantes)
3. **Participant Profile**: Cambios en el perfil demográfico
4. **Fields of Study**: Cambios en campos educativos

---

## Página 1: Overview

Esta es la página de entrada. Pensé en ella como un "explorador general" donde puedes hacerte una idea rápida de cómo están las cosas.

**Visuales principales:**

- **Cards en la parte superior**: Muestran los totales para Pre-COVID, COVID, Post-COVID y el Recovery Rate (99.77%)
- **Mapa geográfico**: Con burbujas proporcionales al número de participantes usando la medida `[Participants (View)]`. Cuando haces clic en un país, filtra todo lo demás. **Nota**: Está filtrado a los top 30 países para evitar que haya burbujas en todos sitios y el mapa quede saturado
- **Gráfico de línea temporal**: Muestra la evolución mes a mes con los 3 períodos en colores diferentes (Pre-COVID, COVID, Post-COVID). Se ven claros los picos en septiembre (inicio del curso académico)
- **Ranking Top 10**: Muestra los 10 países más activos según la dirección seleccionada

**Slicers (controles):**

- **Período COVID**: Pre-COVID / COVID / Post-COVID
- **Dirección (Flow type)**: Receiving / Sending

El slicer de dirección es interesante porque cambia dinámicamente **todos los visuales** de la página. Si seleccionas "Receiving", ves datos de países que reciben estudiantes. Si seleccionas "Sending", ves países que envían. Afecta a:
- **Mapa geográfico**: El tamaño de las burbujas cambia según la dirección seleccionada
- **Ranking Top 10**: Muestra top receivers o top senders
- **Cards**: Los valores de Pre/COVID/Post/Recovery cambian según la perspectiva elegida
- **Gráfico temporal**: La línea refleja la métrica correspondiente a la dirección

El título del gráfico también cambia automáticamente usando una medida DAX:
```dax
Title Top10 =
VAR mode = SELECTEDVALUE(Direction[Mode])
RETURN
SWITCH(TRUE(),
    mode = "Receiving (Destination)", "Top 10 Receiving Countries",
    mode = "Sending (Origin)", "Top 10 Sending Countries",
    "Top 10 Country Activity (Sending + Receiving)"
)
```

---

## Página 2: COVID Impact on Mobility Flows

Esta es la página más compleja técnicamente. El objetivo era ver qué países "ganaron" y cuáles "perdieron" después de COVID, diferenciando entre:

- **Sending (Envío)**: Países que **envían** estudiantes a otros países (orígenes de movilidad)
- **Receiving (Recepción)**: Países que **reciben** estudiantes de otros países (destinos de movilidad)

**Visual principal: Scatter Plot de Cuadrantes**

Este gráfico muestra el cambio Post-COVID vs Pre-COVID en dos dimensiones simultáneamente:

- **Eje X (horizontal)**: % Change Sending - Cuánto cambió el país como **origen** de estudiantes
- **Eje Y (vertical)**: % Change Receiving - Cuánto cambió el país como **destino** de estudiantes
- **Tamaño de burbuja**: Pre Total (Sending + Receiving) - Volumen total Pre-COVID del país
- **Color**: Cuadrante al que pertenece

**Fórmula del cambio:** `DIVIDE(Post - Pre, Pre)` para Sending y Receiving por separado.

**Los cuatro cuadrantes (qué significan):**

1. **Grow Both** (arriba-derecha): El país aumentó TANTO en enviar como en recibir estudiantes
   - Ejemplo: Países que se volvieron más activos en Erasmus+ en ambas direcciones

2. **Grow Rcv** (arriba-izquierda): El país **recibe más** estudiantes pero **envía menos**
   - Ejemplo: Destinos emergentes que atraen más pero sus estudiantes salen menos

3. **Grow Snd** (abajo-derecha): El país **envía más** estudiantes pero **recibe menos**
   - Ejemplo: Ukraine (+137% sending) - por la guerra, más estudiantes ucranianos estudian fuera

4. **Down both** (abajo-izquierda): El país bajó en ambas direcciones
   - Ejemplo: Russia y UK por guerra/Brexit respectivamente

**Filtro de volumen mínimo (Pre < 2,000 excluidos):**

Este filtro se aplica a **todos los gráficos** de la página. Excluí países con menos de 2,000 movilidades totales Pre-COVID porque sus porcentajes son muy volátiles. Un país que pasa de 10 a 30 movilidades es +200% pero no es significativo.

**Gráficos de barras laterales (4 gráficos):**

Los 4 gráficos complementan el scatter mostrando los rankings específicos:

**Superior izquierda - Largest Increases (Receiving):**
- Top países que más **aumentaron como destinos**
- Croatia +37%, Greece +35%, Turkey +32%, etc.
- Estos países reciben más estudiantes Post-COVID que Pre-COVID

**Inferior izquierda - Largest Decreases (Receiving):**
- Top países que más **disminuyeron como destinos**
- Russia -90%, UK -70%, Poland, France, Germany (caídas moderadas)
- Estos países reciben menos estudiantes Post-COVID

**Superior derecha - Largest Increases (Sending):**
- Top países que más **aumentaron como orígenes**
- Ukraine +137% (guerra), Estonia +59%, Romania, Cyprus, etc.
- Estos países envían más estudiantes Post-COVID

**Inferior derecha - Largest Decreases (Sending):**
- Top países que más **disminuyeron como orígenes**
- Russia -90%, UK -87%, Serbia, Denmark, Turkey (envían menos)
- Estos países envían menos estudiantes Post-COVID

**Interactividad:**

Cuando haces clic en un país en el scatter plot, se **resalta (highlight)** en los gráficos de barras laterales **solo si ese país aparece en ellos**. Si el país no está en los top rankings, no se resalta nada en los laterales (porque no está visible en esos gráficos).

**Nota técnica**: Se usa highlight (resaltado) en lugar de filtrado cruzado para que puedas ver el contexto completo del ranking mientras identificas visualmente dónde está el país seleccionado.

**Hallazgos clave de esta página:**

- **Ganadores en Receiving**: Sur y Este de Europa (Croatia, Greece, Romania, Turkey) - destinos emergentes más económicos
- **Perdedores en Receiving**: Russia (guerra), UK (Brexit), y pérdida de cuota de Francia/Alemania/Polonia
- **Sorpresa en Sending**: Ukraine +137% - La guerra provocó que más estudiantes ucranianos estudien fuera
- **Perdedores en ambos**: Russia y UK cayeron tanto en envío como en recepción por razones geopolíticas

---

## Página 3: Participant Profile

Aquí analicé si el perfil del estudiante Erasmus cambió después de COVID.

**Gráficos de distribución (Pre vs Post):**

- **Gender Distribution**: Se mantiene estable entre Pre y Post COVID, aunque podemos observar que el género femenino predomina con alrededor de un 60/40 (59.6% F, 40.3% M), lo cual es algo remarcable
- **Profile Distribution** (Learner vs Staff): También estable (**90.3% Learner, 9.7% Staff**)
- **Fewer Opportunities Distribution**: **Gran aumento** - casi se duplica:
  - Pre-COVID: 68,128 participantes
  - Post-COVID: 149,444 participantes
  - Aumento: **+81,316 (+119%)**

**Gráfico de barras - Top Contributors to "Fewer Opportunities" Increase:**

Muestra qué países contribuyeron más al aumento de participantes con fewer opportunities. Germany lidera con +38,963 participantes.

**¿Por qué aumentó tanto "Fewer Opportunities"?** La razón exacta no está clara en los datos, pero podría deberse a:
- Cambios en políticas de inclusividad de la UE después de COVID
- Mejor registro/declaración de este dato en el sistema Erasmus+
- Mayor esfuerzo de países como Alemania en incluir grupos desfavorecidos

**Distribución de duración (Pre vs Post):**

Un histograma que compara rangos de duración. **Nota**: Este gráfico está filtrado a `participant_profile = Learner` porque para Staff todas las movilidades son de menos de 1 mes, lo cual distorsionaría la comparación.

Se ve claramente que:
- Las movilidades de 6-9 meses bajaron
- Las de 3-6 meses siguen siendo las más comunes
- Las movilidades cortas (<3 meses) aumentaron relativamente

**Interpretación:** Después de COVID, la gente prefiere movilidades más cortas (probablemente por cautela económica o preferencia por programas intensivos).

---

## Página 4: Fields of Study

Aquí se ve qué campos de estudio ganaron o perdieron popularidad. **Nota**: Esta página está filtrada a `activity_group = HE` (Higher Education) y `participant_profile = Learner` para centrarnos en movilidad estudiantil universitaria.

**Visual principal:**

Un gráfico de barras horizontales que muestra todos los campos ISCED-F con barras apiladas:
- Amarillo/dorado: Pre-COVID
- Azul: Post-COVID

**Tabla interactiva:**

Muestra el % Change por campo. Cuando seleccionas una fila, un gráfico lateral muestra los top países emisores para ese campo con **barras apiladas (Pre-COVID en amarillo/dorado, Post-COVID en azul)**, permitiendo comparar el volumen de cada país en ambos períodos.

**Hallazgos clave:**

**Grandes ganadores:**
- ICT: +41.3% (el boom de tecnología)
- Natural sciences: +14.4%
- Agriculture: +10.4%
- Health: +9.7%

**Grandes perdedores:**
- Arts & Humanities: -21.7%
- Services: -7.4%
- Business: -6.6%

---

## Modelo de Datos

Aquí es donde tomé algunas decisiones importantes.

### Modelo Estrella (Star Schema)

Diseñé el modelo como una estrella clásica:

```
        df_he_pbi (Fact Table)
              |
    __________|__________
   |          |          |
Dim_Country  Dim_Country  DimDate
_Sending     _Receiving
```

**Tabla de hechos:**
- `df_he_pbi`: Los 2.1 millones de registros de movilidad

**Tablas dimensión:**
- `Dim_Country_Sending`: Países emisores (33 países únicos)
- `Dim_Country_Receiving`: Países receptores (33 países únicos)
- `DimDate`: Tabla de fechas con clasificación Pre/COVID/Post automática

**Relaciones:**
- `Dim_Country_Sending[sending_country_code]` → `df_he_pbi[sending_country_code]` **(1:M, activa)**
- `Dim_Country_Receiving[receiving_country_code]` → `df_he_pbi[receiving_country_code]` **(1:M, activa)**
- `DimDate[Date]` → `df_he_pbi[mobility_start_ym]` **(1:M, activa)**

Todas las relaciones siguen cardinalidad **uno a muchos (1:M)**, lo cual es estándar en un modelo estrella: una fila en la dimensión se relaciona con muchas filas en la tabla de hechos.

### ¿Por qué DOS tablas de países?

Al principio intenté usar una sola tabla `DimCountry` con dos relaciones:
- Una activa para `sending_country_code`
- Una inactiva para `receiving_country_code`

**El problema:** Para el análisis de flujos (Sending × Receiving) necesitaba ambas relaciones activas simultáneamente. Con una sola tabla solo puedes tener una relación activa. La otra queda inactiva y tienes que usar `USERELATIONSHIP()` en cada medida, lo cual es tedioso.

**La solución:** Dividirlas en dos tablas separadas. Ahora ambas relaciones son activas todo el tiempo.

Esto es lo que se llama "role-playing dimension" en modelado de datos: la misma entidad (Country) aparece en roles diferentes (Sending vs Receiving). Es similar a cómo en un modelo de ventas podrías tener DimCustomer y DimSalesperson (ambos son personas, pero juegan roles distintos).

### Tablas Auxiliares (sin relaciones físicas)

**AxisCountry:**

Es una tabla calculada que une todos los códigos de país:

```dax
AxisCountry =
DISTINCT(
    UNION(
        SELECTCOLUMNS(Dim_Country_Sending,
            "country_code", [sending_country_code],
            "country_name", [sending_country_name]),
        SELECTCOLUMNS(Dim_Country_Receiving,
            "country_code", [receiving_country_code],
            "country_name", [receiving_country_name])
    )
)
```

**¿Para qué sirve?** Actúa como un "eje virtual" para análisis transversales. No tiene relaciones físicas con la tabla de hechos, pero la uso con `TREATAS()` para filtrar dinámicamente:

```dax
Receiving Participants =
CALCULATE(
    [Participants],
    TREATAS(VALUES(AxisCountry[country_code]),
            df_he_pbi[receiving_country_code])
)
```

Esto me permite calcular métricas por país independientemente de si es emisor o receptor.

**Direction:**

Otra tabla sin relaciones, con solo dos filas:

```dax
Direction = DATATABLE("Mode", STRING, {
    {"Receiving (Destination)"},
    {"Sending (Origin)"}
})
```

La uso para el slicer de perspectiva (Receiving/Sending). Las medidas leen el valor seleccionado con `SELECTEDVALUE(Direction[Mode])` y cambian su comportamiento.

### DimDate

Esta tabla la creé con DAX en lugar de importarla:

```dax
DimDate =
VAR MinDate = MIN(df_he_pbi[mobility_start_ym])
VAR MaxDate = MAX(df_he_pbi[mobility_start_ym])
RETURN
ADDCOLUMNS(
    CALENDAR(MinDate, MaxDate),
    "Year", YEAR([Date]),
    "MonthNum", MONTH([Date]),
    "MonthName", FORMAT([Date], "MMMM"),
    "YearMonth", FORMAT([Date], "YYYY-MM"),
    "Quarter", "Q" & FORMAT([Date], "Q"),
    "CovidPeriod",
        SWITCH(TRUE(),
            YEAR([Date]) <= 2019, "Pre-COVID (<=2019)",
            YEAR([Date]) IN {2020, 2021}, "COVID (2020–2021)",
            YEAR([Date]) >= 2022, "Post-COVID (>=2022)"
        )
)
```

La columna `CovidPeriod` clasifica automáticamente cada fecha. Esto hace que sea muy fácil filtrar por período sin tener que escribir condiciones repetitivas.

**¿Por qué 2021 está en COVID y no en Post?** Porque en 2021 todavía había restricciones de movilidad en muchos países europeos. La normalización completa no llegó hasta 2022.

### Transformaciones en Power Query

El modelo incluye algunas transformaciones en Power Query que hice para limpiar la estructura de datos:

**Duplicación de la fact table:**

Al cargar los datos, duplicué la tabla de hechos (en lugar de referenciarla) antes de crear las tablas dimensión `Dim_Country_Sending` y `Dim_Country_Receiving`.

**¿Por qué?** Quería eliminar las columnas de nombres de países de la fact table final (para evitar redundancia, ya que esos datos están en las dimensiones). El problema es que si creas tablas por referencia y luego eliminas columnas de la tabla original, también se eliminan de las tablas referenciadas.

**La solución:** Duplicar en lugar de referenciar. Esto crea una copia independiente. Así pude:
1. Crear las dimensiones de países por referencia desde el duplicado
2. Eliminar las columnas de nombres de países del duplicado
3. Las dimensiones mantienen las columnas porque se crearon antes de eliminarlas

**Alternativas que consideré:**
- Simplemente no eliminar las columnas (aceptar la redundancia) - es válido en Power BI
- Ocultar las columnas en el modelo en lugar de eliminarlas en Power Query
- Crear las dimensiones de forma independiente sin referencias

La duplicación funciona bien aunque usa un poco más de memoria durante la carga.

---

## Medidas DAX Principales

### Recovery Rate

```dax
Recovery Rate =
DIVIDE([Post-COVID Participants], [Pre-COVID Participants])
```

Simple pero efectiva. Compara Post/Pre y da 0.9977 (99.77%).

### Pct Change (con filtro de volumen mínimo)

```dax
Pct Change Receiving (MinBase) =
VAR pre = [Pre Receiving]
VAR pos = [Post Receiving]
RETURN
IF(
    pre < 2000,  -- Filtro de volumen mínimo
    BLANK(),
    DIVIDE(pos - pre, pre)
)
```

El `IF(pre < 2000, BLANK(), ...)` es crucial. Sin él, países con 10 movilidades que pasan a 30 mostrarían +200% y distorsionarían el scatter plot.

### Quadrant (columna calculada en AxisCountry)

```dax
Quadrant =
VAR send = CALCULATE([Pct Change Sending (MinBase)])
VAR recv = CALCULATE([Pct Change Receiving (MinBase)])
RETURN
IF(
    ISBLANK(send) || ISBLANK(recv),
    BLANK(),
    SWITCH(TRUE(),
        send >= 0 && recv >= 0, "Grow Both",
        send <  0 && recv >= 0, "Grow Rcv",
        send >= 0 && recv <  0, "Grow Snd",
        "Down both"
    )
)
```

Clasifica cada país en uno de los 4 cuadrantes basándose en su cambio en sending y receiving.

### Participants (View) - Medida Dinámica

```dax
Participants (View) =
VAR mode = SELECTEDVALUE(Direction[Mode], "Both")
RETURN
SWITCH(
    mode,
    "Receiving (Destination)", [Receiving Participants],
    "Sending (Origin)", [Sending Participants],
    "Both", [Total Participants (In+Out)],
    [Total Participants (In+Out)]
)
```

Esta medida "lee" qué opción está seleccionada en el slicer Direction y devuelve la métrica correspondiente. Es lo que hace que los visuales cambien dinámicamente.

### Duration Bin (columna calculada en df_he_pbi)

```dax
Duration Bin =
SWITCH(TRUE(),
    df_he_pbi[mobility_duration] < 30,  "< 1 month",
    df_he_pbi[mobility_duration] < 90,  "1–3 months",
    df_he_pbi[mobility_duration] < 180, "3–6 months",
    df_he_pbi[mobility_duration] < 270, "6–9 months",
    "9+ months"
)
```

Agrupa las duraciones en rangos para el gráfico de la Página 3.

---

## Interacciones

**Cross-filtering:**

Los visuales están configurados para que:
- Clic en mapa → filtra ranking y actualiza cards
- Clic en barra de ranking → filtra mapa
- Slicers → filtran toda la página

**Cross-filtering (Página 4):**

En la tabla de Fields of Study (Página 4), cuando seleccionas una fila, el gráfico lateral se actualiza automáticamente para mostrar los top países emisores de ese campo.

---

## Limitaciones y Consideraciones

**1. Definición de períodos:**
La decisión de poner 2021 en COVID en lugar de Post es debatible. Algunos podrían argumentar que debería estar en Post. Esto afectaría ligeramente el Recovery Rate.

**2. Filtro de volumen mínimo:**
Excluye países pequeños del análisis de cuadrantes. Si te interesan específicamente países pequeños, habría que hacer un análisis separado.

**3. Causalidad vs Correlación:**
El dashboard muestra correlaciones (ej. UK cayó después de Brexit) pero no puede probar causalidad. Siempre hay múltiples factores.

---

## Recursos Adicionales

- [DAX_MEASURES.md](DAX_MEASURES.md): Documentación completa de todas las medidas con ejemplos
- [DATA_DICTIONARY.md](DATA_DICTIONARY.md): Qué significa cada variable
- [PIPELINE.md](PIPELINE.md): Cómo se limpiaron los datos
- [QUICK_START.md](QUICK_START.md): Guía de inicio rápido

---

**Autor**: Andrés Liz Domínguez
**Última actualización**: Febrero 2026
