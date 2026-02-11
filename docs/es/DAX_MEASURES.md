#  DAX Measures - Referencia Completa

[Read in English](../en/DAX_MEASURES.md) | [Leer en Español](DAX_MEASURES.md)

Documentación de todas las medidas y tablas calculadas del dashboard de Power BI.

---

##  Tablas Calculadas

### 1. DimDate (Tabla de Fechas)

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
        SWITCH(
            TRUE(),
            YEAR([Date]) <= 2019, "Pre-COVID (<=2019)",
            YEAR([Date]) IN {2020, 2021}, "COVID (2020–2021)",
            YEAR([Date]) >= 2022, "Post-COVID (>=2022)"
        )
)
```

**Propósito**: Dimensión de tiempo con clasificación automática de períodos COVID.

**Columnas clave**:
- `CovidPeriod`: Clasifica cada fecha en Pre/COVID/Post

---

### 2. Direction (Tabla de Perspectiva)

```dax
Direction =
DATATABLE("Mode", STRING, {
    {"Receiving (Destination)"},
    {"Sending (Origin)"}
})
```

**Propósito**: Slicer para alternar entre vista de países receptores vs emisores.

**Uso**: Se lee con `SELECTEDVALUE(Direction[Mode])` en medidas dinámicas.

---

### 3. AxisCountry (Eje Virtual de Países)

```dax
AxisCountry =
DISTINCT(
    UNION(
        SELECTCOLUMNS(
            Dim_Country_Sending,
            "country_code", Dim_Country_Sending[sending_country_code],
            "country_name", Dim_Country_Sending[sending_country_name]
        ),
        SELECTCOLUMNS(
            Dim_Country_Receiving,
            "country_code", Dim_Country_Receiving[receiving_country_code],
            "country_name", Dim_Country_Receiving[receiving_country_name]
        )
    )
)
```

**Propósito**: Tabla desconectada (sin relaciones físicas) que combina todos los países únicos de sending + receiving.

**Por qué existe**: Permite tener **un solo slicer de países** en lugar de dos separados (uno para sending, otro para receiving).

**Dónde se usa TREATAS()**: En las medidas `[Receiving Participants]` y `[Sending Participants]` (ver secciones más abajo).

**Cómo funciona**: Cuando seleccionas un país en el slicer de AxisCountry, `TREATAS()` crea una relación virtual temporal para filtrar la tabla de hechos por ese país como emisor o receptor.

---

## Medidas Fundamentales

### Participants (Medida Base)

```dax
Participants = SUM(df_he_pbi[actual_participants])
```

**Propósito**: Suma de participantes reales. La columna `actual_participants` es en su gran mayoría 1 (un participante por registro), pero se usa SUM por si hubiera casos donde un registro representa múltiples participantes.

**Uso**: Base para todas las medidas derivadas.

---

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

**Propósito**: Cambia dinámicamente entre Receiving/Sending/Total según slicer Direction.

**Uso**: Alimenta cards, rankings y mapas en Página 1.

---

### Receiving Participants

```dax
Receiving Participants =
CALCULATE(
    [Participants],
    TREATAS(VALUES(AxisCountry[country_code]), df_he_pbi[receiving_country_code])
)
```

**Propósito**: Cuenta movilidades donde el país seleccionado en AxisCountry es **receptor**.

**Técnica**: `TREATAS()` crea una relación virtual temporal.

---

### Sending Participants

```dax
Sending Participants =
CALCULATE(
    [Participants],
    TREATAS(VALUES(AxisCountry[country_code]), df_he_pbi[sending_country_code])
)
```

**Propósito**: Cuenta movilidades donde el país seleccionado es **emisor**.

---

### Total Participants (In+Out)

```dax
Total Participants (In+Out) =
[Receiving Participants] + [Sending Participants]
```

**Propósito**: Actividad total del país (suma de envíos y recepciones).

**Uso**: Ranking Top 10 cuando slicer Direction está en ambos.

---

## Medidas de Períodos COVID

### Pre-COVID Participants

```dax
Pre-COVID Participants =
CALCULATE(
    [Participants (View)],
    DimDate[CovidPeriod] = "Pre-COVID (<=2019)"
)
```

**Propósito**: Movilidades en período precovid (2017-2019).

---

### Post-COVID Participants

```dax
Post-COVID Participants =
CALCULATE(
    [Participants (View)],
    DimDate[CovidPeriod] = "Post-COVID (>=2022)"
)
```

**Propósito**: Movilidades en período postcovid (2022-2024).

---

### Recovery Rate

```dax
Recovery Rate =
DIVIDE(
    [Post-COVID Participants],
    [Pre-COVID Participants]
)
```

**Propósito**: Ratio de recuperación postcovid (Post / Pre).

**Interpretación**:
- 1.0 (100%): Recuperación completa
- 0.997 (99.77%): Valor global del dashboard
- >1.0: Crecimiento postcovid

---

### Pre Receiving / Pre Sending

```dax
Pre Receiving =
CALCULATE(
    [Receiving Participants],
    DimDate[CovidPeriod] = "Pre-COVID (<=2019)"
)

Pre Sending =
CALCULATE(
    [Sending Participants],
    DimDate[CovidPeriod] = "Pre-COVID (<=2019)"
)
```

**Propósito**: Movilidades precovid desagregadas por dirección (para análisis de cuadrantes).

---

### Post Receiving / Post Sending

```dax
Post Receiving =
CALCULATE(
    [Receiving Participants],
    DimDate[CovidPeriod] = "Post-COVID (>=2022)"
)

Post Sending =
CALCULATE(
    [Sending Participants],
    DimDate[CovidPeriod] = "Post-COVID (>=2022)"
)
```

**Propósito**: Movilidades postcovid desagregadas por dirección.

---

### Pre Total (Sending+Receiving)

```dax
Pre Total (Sending+Receiving) =
[Pre Sending] + [Pre Receiving]
```

**Propósito**: Actividad total precovid. Se usa como **tamaño de burbuja** en scatter plot (indica volumen base).

---

## Medidas de Análisis de Cuadrantes (Página 2)

### Pct Change Receiving (MinBase)

```dax
Pct Change Receiving (MinBase) =
VAR pre = [Pre Receiving]
VAR pos = [Post Receiving]
RETURN
IF(
    pre < 2000,  -- Filtro: Solo países con volumen significativo
    BLANK(),
    DIVIDE(pos - pre, pre)
)
```

**Propósito**: % de cambio en recepciones (Post vs Pre), filtrando países pequeños.

**Filtro**: `pre < 2000` → Excluye outliers con cambios volátiles.

**Ejemplo**: Si pre=3000 y post=4000, resultado=0.333 (33.3% aumento).

---

### Pct Change Sending (MinBase)

```dax
Pct Change Sending (MinBase) =
VAR pre = [Pre Sending]
VAR pos = [Post Sending]
RETURN
IF(
    pre < 2000,
    BLANK(),
    DIVIDE(pos - pre, pre)
)
```

**Propósito**: % de cambio en envíos (Post vs Pre), filtrando países pequeños.

---

### Pct Change Receiving (Inv)

```dax
Pct Change Receiving (Inv) =
- [Pct Change Receiving (MinBase)]
```

**Propósito**: Versión invertida (negativa) para ordenar **decreases** de mayor a menor.

**Uso**: Gráfico "Largest Decreases - Receiving".

---

### Pct Change Sending (Inv)

```dax
Pct Change Sending (Inv) =
- [Pct Change Sending (MinBase)]
```

**Propósito**: Versión invertida para ordenar decreases en envíos.

---

### Recovery Rate (Post/Pre) - Específica Receiving

```dax
Recovery Rate (Post/Pre) =
DIVIDE([Post Receiving], [Pre Receiving])
```

**Propósito**: Ratio de recuperación específico para recepciones (usado en análisis detallado).

**Diferencia con [Recovery Rate]**: Esta es específica receiving, la otra usa `[Participants (View)]` (dinámica).

---

##  Columnas Calculadas

### Quadrant (en AxisCountry)

```dax
Quadrant =
VAR send = CALCULATE([Pct Change Sending (MinBase)])
VAR recv = CALCULATE([Pct Change Receiving (MinBase)])
RETURN
IF(
    ISBLANK(send) || ISBLANK(recv),
    BLANK(),
    SWITCH(
        TRUE(),
        send >= 0 && recv >= 0, "Grow Both",
        send <  0 && recv >= 0, "Grow Rcv",
        send >= 0 && recv <  0, "Grow Snd",
        "Down both"
    )
)
```

**Propósito**: Clasifica cada país en uno de 4 cuadrantes según % de cambio.

**Cuadrantes**:
-  **Grow Both** (Q1): ↑ Sending, ↑ Receiving
-  **Grow Rcv** (Q2): ↓ Sending, ↑ Receiving
-  **Grow Snd** (Q4): ↑ Sending, ↓ Receiving
-  **Down both** (Q3): ↓ Sending, ↓ Receiving

**Uso**: Leyenda (color) en scatter plot de Página 2.

---

### Duration Bin (en df_he_pbi)

```dax
Duration Bin =
SWITCH(
    TRUE(),
    df_he_pbi[mobility_duration] < 30,  "< 1 month",
    df_he_pbi[mobility_duration] < 90,  "1–3 months",
    df_he_pbi[mobility_duration] < 180, "3–6 months",
    df_he_pbi[mobility_duration] < 270, "6–9 months",
    "9+ months"
)
```

**Propósito**: Categoriza duraciones de movilidad en rangos.

**Rangos**:
- **< 1 mes** (< 30 días): Summer schools, visitas cortas
- **1-3 meses** (30-89): Trimestre
- **3-6 meses** (90-179): **Semestre** (rango dominante)
- **6-9 meses** (180-269): Año académico incompleto
- **9+ meses** (270+): Año completo, dobles titulaciones

**Uso**: Distribución de duración en Página 3.

---

##  Medidas de UI/UX

### Title Top10

```dax
Title Top10 =
VAR mode = SELECTEDVALUE(Direction[Mode])
RETURN
SWITCH(
    TRUE(),
    mode = "Receiving (Destination)", "Top 10 Receiving Countries",
    mode = "Sending (Origin)", "Top 10 Sending Countries",
    "Top 10 Country Activity (Sending + Receiving)"
)
```

**Propósito**: Título dinámico para gráfico de ranking según slicer Direction.

**Uso**: Mejora UX al adaptar texto automáticamente.

---

### COVID Participants

```dax
COVID Participants =
CALCULATE(
    [Participants (View)],
    DimDate[CovidPeriod] = "COVID (2020–2021)"
)
```

**Propósito**: Cuenta participantes durante el período COVID (2020-2021).

**Uso**: Card en Página 1 mostrando volumen del período COVID.

---

### % Change

```dax
% Change =
VAR FielDif = [Post-COVID Participants] - [Pre-COVID Participants]
RETURN
DIVIDE(FielDif, [Pre-COVID Participants])
```

**Propósito**: Cambio porcentual entre Post-COVID y Pre-COVID.

**Uso**: Tabla de campos de estudio en Página 4.

---

##  Patrones DAX Utilizados

### 1. TREATAS() - Relaciones Virtuales

**Técnica**: Crear filtro temporal sin relación física.

```dax
CALCULATE(
    [Medida],
    TREATAS(VALUES(TablaA[columna]), TablaB[columna])
)
```

**Cuándo usar**: Tabla desconectada (AxisCountry) necesita filtrar tabla de hechos.

---

### 2. SELECTEDVALUE() - Lectura de Slicers

**Técnica**: Leer valor seleccionado en slicer desconectado.

```dax
VAR mode = SELECTEDVALUE(Direction[Mode], "Both")
```

**Cuándo usar**: Medidas que cambian comportamiento según slicer.

---

### 3. SWITCH(TRUE(), ...) - Lógica Condicional

**Técnica**: Evaluación de múltiples condiciones.

```dax
SWITCH(
    TRUE(),
    condicion1, resultado1,
    condicion2, resultado2,
    default
)
```

**Cuándo usar**: Alternativa más legible a IF anidados.

---

### 4. CALCULATE + Filtro - Contexto Temporal

**Técnica**: Aplicar filtro específico de período.

```dax
CALCULATE(
    [Medida],
    DimDate[CovidPeriod] = "Pre-COVID (<=2019)"
)
```

**Cuándo usar**: Comparaciones Pre vs Post.

---

### 5. IF(pre < umbral, BLANK(), ...) - Filtrado de Outliers

**Técnica**: Retornar BLANK() para valores no significativos.

```dax
IF(
    pre < 2000,
    BLANK(),
    DIVIDE(...)
)
```

**Cuándo usar**: Evitar % de cambio volátil en países pequeños.

---

## Convenciones de Nombres

### Estilo utilizado:
- **Medidas**: PascalCase con espacios (`Pre-COVID Participants`)
- **Columnas calculadas**: PascalCase sin guiones (`Duration Bin`)
- **Tablas**: PascalCase (`AxisCountry`, `DimDate`)
- **Sufijos descriptivos**:
  - `(View)`: Medida dinámica que cambia según slicer
  - `(MinBase)`: Incluye filtro de volumen mínimo
  - `(Inv)`: Versión invertida (negativa)

---

##  Medidas por Página

### Página 1 (Overview)
- `[Participants (View)]` - Base dinámica
- `[Pre-COVID Participants]` - Card
- `[COVID Participants]` - Card
- `[Post-COVID Participants]` - Card
- `[Recovery Rate]` - Card
- `[Title Top10]` - Título dinámico

### Página 2 (COVID Impact on Mobility Flows)
- `[Pct Change Receiving (MinBase)]` - Eje Y scatter
- `[Pct Change Sending (MinBase)]` - Eje X scatter
- `[Pre Total (Sending+Receiving)]` - Tamaño burbuja
- `Quadrant` (columna) - Color/leyenda
- `[Pct Change Receiving (Inv)]` - Gráfico decreases
- `[Pct Change Sending (Inv)]` - Gráfico decreases

### Página 3 (Participant Profile)
- `[Pre-COVID Participants]` - Comparación distribuciones
- `[Post-COVID Participants]` - Comparación distribuciones
- `Duration Bin` (columna) - Categorización duración

### Página 4 (Fields of Study)
- `[Pre-COVID Participants]` - Tabla comparativa
- `[Post-COVID Participants]` - Tabla comparativa
- `[% Change]` - % Change en tabla

---

## Solución de Problemas Frecuentes

### Error: "A circular dependency was detected"
**Causa**: Medida que se referencia a sí misma indirectamente.
**Solución**: Revisar cadena de dependencias con DAX Studio.

### Error: "The value for column X cannot be determined"
**Causa**: Relación mal configurada o faltante.
**Solución**: Verificar modelo de datos (relaciones activas).

### Error: TREATAS() no filtra correctamente
**Causa**: Tipos de datos no coinciden (ej: Text vs Int).
**Solución**: Asegurar que ambas columnas sean del mismo tipo.

### Blank inesperado en scatter plot
**Causa**: País tiene `pre < 2000` y fue filtrado.
**Solución**: Ajustar umbral en `[Pct Change X (MinBase)]` si es muy restrictivo.

---

## Recursos para Aprender DAX

- [DAX.guide](https://dax.guide/) - Referencia oficial de funciones
- [SQLBI](https://www.sqlbi.com/) - Tutoriales avanzados
- [Excelerator BI](https://exceleratorbi.com.au/blog/) - Patrones DAX

---

**Autor**: Andrés Liz Domínguez
**Última actualización**: Febrero 2026
**Total de medidas**: 20+
**Total de columnas calculadas**: 2
**Total de tablas calculadas**: 3
