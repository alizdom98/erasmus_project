# DAX Measures - Complete Reference

[Read in English](DAX_MEASURES.md) | [Leer en Español](../es/DAX_MEASURES.md)

Documentation of all measures and calculated tables in the Power BI dashboard.

---

## Calculated Tables

### 1. DimDate (Date Table)

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

**Purpose**: Time dimension with automatic COVID period classification.

**Key columns**:
- `CovidPeriod`: Classifies each date into Pre/COVID/Post

---

### 2. Direction (Perspective Table)

```dax
Direction =
DATATABLE("Mode", STRING, {
    {"Receiving (Destination)"},
    {"Sending (Origin)"}
})
```

**Purpose**: Slicer to toggle between receiving vs sending country views.

**Usage**: Read with `SELECTEDVALUE(Direction[Mode])` in dynamic measures.

---

### 3. AxisCountry (Virtual Country Axis)

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

**Purpose**: Disconnected table (no physical relationships) that combines all unique countries from sending + receiving.

**Why it exists**: Allows having **a single country slicer** instead of two separate ones (one for sending, another for receiving).

**Where TREATAS() is used**: In the measures `[Receiving Participants]` and `[Sending Participants]` (see sections below).

**How it works**: When you select a country in the AxisCountry slicer, `TREATAS()` creates a temporary virtual relationship to filter the fact table by that country as sender or receiver.

---

## Fundamental Measures

### Participants (Base Measure)

```dax
Participants = SUM(df_he_pbi[actual_participants])
```

**Purpose**: Sum of actual participants. The column `actual_participants` is mostly 1 (one participant per record), but SUM is used in case there are cases where one record represents multiple participants.

**Usage**: Base for all derived measures.

---

### Participants (View) - Dynamic Measure

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

**Purpose**: Dynamically switches between Receiving/Sending/Total according to Direction slicer.

**Usage**: Feeds cards, rankings, and maps in Page 1.

---

### Receiving Participants

```dax
Receiving Participants =
CALCULATE(
    [Participants],
    TREATAS(VALUES(AxisCountry[country_code]), df_he_pbi[receiving_country_code])
)
```

**Purpose**: Counts mobilities where the country selected in AxisCountry is **receiving**.

**Technique**: `TREATAS()` creates a temporary virtual relationship.

---

### Sending Participants

```dax
Sending Participants =
CALCULATE(
    [Participants],
    TREATAS(VALUES(AxisCountry[country_code]), df_he_pbi[sending_country_code])
)
```

**Purpose**: Counts mobilities where the selected country is **sending**.

---

### Total Participants (In+Out)

```dax
Total Participants (In+Out) =
[Receiving Participants] + [Sending Participants]
```

**Purpose**: Total country activity (sum of sending and receiving).

**Usage**: Top 10 ranking when Direction slicer is on both.

---

## COVID Period Measures

### Pre-COVID Participants

```dax
Pre-COVID Participants =
CALCULATE(
    [Participants (View)],
    DimDate[CovidPeriod] = "Pre-COVID (<=2019)"
)
```

**Purpose**: Mobilities in pre-COVID period (2017-2019).

---

### Post-COVID Participants

```dax
Post-COVID Participants =
CALCULATE(
    [Participants (View)],
    DimDate[CovidPeriod] = "Post-COVID (>=2022)"
)
```

**Purpose**: Mobilities in post-COVID period (2022-2024).

---

### Recovery Rate

```dax
Recovery Rate =
DIVIDE(
    [Post-COVID Participants],
    [Pre-COVID Participants]
)
```

**Purpose**: Post-COVID recovery ratio (Post / Pre).

**Interpretation**:
- 1.0 (100%): Complete recovery
- 0.997 (99.77%): Global dashboard value
- >1.0: Post-COVID growth

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

**Purpose**: Pre-COVID mobilities disaggregated by direction (for quadrant analysis).

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

**Purpose**: Post-COVID mobilities disaggregated by direction.

---

### Pre Total (Sending+Receiving)

```dax
Pre Total (Sending+Receiving) =
[Pre Sending] + [Pre Receiving]
```

**Purpose**: Total pre-COVID activity. Used as **bubble size** in scatter plot (indicates base volume).

---

## Quadrant Analysis Measures (Page 2)

### Pct Change Receiving (MinBase)

```dax
Pct Change Receiving (MinBase) =
VAR pre = [Pre Receiving]
VAR pos = [Post Receiving]
RETURN
IF(
    pre < 2000,  -- Filter: Only countries with significant volume
    BLANK(),
    DIVIDE(pos - pre, pre)
)
```

**Purpose**: % change in receptions (Post vs Pre), filtering small countries.

**Filter**: `pre < 2000` → Excludes outliers with volatile changes.

**Example**: If pre=3000 and post=4000, result=0.333 (33.3% increase).

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

**Purpose**: % change in sending (Post vs Pre), filtering small countries.

---

### Pct Change Receiving (Inv)

```dax
Pct Change Receiving (Inv) =
- [Pct Change Receiving (MinBase)]
```

**Purpose**: Inverted (negative) version to sort **decreases** from largest to smallest.

**Usage**: "Largest Decreases - Receiving" chart.

---

### Pct Change Sending (Inv)

```dax
Pct Change Sending (Inv) =
- [Pct Change Sending (MinBase)]
```

**Purpose**: Inverted version to sort decreases in sending.

---

### Recovery Rate (Post/Pre) - Specific Receiving

```dax
Recovery Rate (Post/Pre) =
DIVIDE([Post Receiving], [Pre Receiving])
```

**Purpose**: Recovery ratio specific for receptions (used in detailed analysis).

**Difference with [Recovery Rate]**: This is specific to receiving, the other uses `[Participants (View)]` (dynamic).

---

## Calculated Columns

### Quadrant (in AxisCountry)

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

**Purpose**: Classifies each country into one of 4 quadrants based on % change.

**Quadrants**:
- **Grow Both** (Q1): ↑ Sending, ↑ Receiving
- **Grow Rcv** (Q2): ↓ Sending, ↑ Receiving
- **Grow Snd** (Q4): ↑ Sending, ↓ Receiving
- **Down both** (Q3): ↓ Sending, ↓ Receiving

**Usage**: Legend (color) in Page 2 scatter plot.

---

### Duration Bin (in df_he_pbi)

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

**Purpose**: Categorizes mobility durations into ranges.

**Ranges**:
- **< 1 month** (< 30 days): Summer schools, short visits
- **1-3 months** (30-89): Quarter
- **3-6 months** (90-179): **Semester** (dominant range)
- **6-9 months** (180-269): Incomplete academic year
- **9+ months** (270+): Full year, double degrees

**Usage**: Duration distribution in Page 3.

---

## UI/UX Measures

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

**Purpose**: Dynamic title for ranking chart according to Direction slicer.

**Usage**: Improves UX by automatically adapting text.

---

### COVID Participants

```dax
COVID Participants =
CALCULATE(
    [Participants (View)],
    DimDate[CovidPeriod] = "COVID (2020–2021)"
)
```

**Purpose**: Counts participants during the COVID period (2020-2021).

**Usage**: Card in Page 1 showing COVID-period volume.

---

### % Change

```dax
% Change =
VAR FielDif = [Post-COVID Participants] - [Pre-COVID Participants]
RETURN
DIVIDE(FielDif, [Pre-COVID Participants])
```

**Purpose**: Percentage change between Post-COVID and Pre-COVID periods.

**Usage**: Fields of Study table in Page 4.

---

## DAX Patterns Used

### 1. TREATAS() - Virtual Relationships

**Technique**: Create temporary filter without physical relationship.

```dax
CALCULATE(
    [Measure],
    TREATAS(VALUES(TableA[column]), TableB[column])
)
```

**When to use**: Disconnected table (AxisCountry) needs to filter fact table.

---

### 2. SELECTEDVALUE() - Reading Slicers

**Technique**: Read selected value in disconnected slicer.

```dax
VAR mode = SELECTEDVALUE(Direction[Mode], "Both")
```

**When to use**: Measures that change behavior based on slicer.

---

### 3. SWITCH(TRUE(), ...) - Conditional Logic

**Technique**: Evaluation of multiple conditions.

```dax
SWITCH(
    TRUE(),
    condition1, result1,
    condition2, result2,
    default
)
```

**When to use**: More readable alternative to nested IFs.

---

### 4. CALCULATE + Filter - Temporal Context

**Technique**: Apply specific period filter.

```dax
CALCULATE(
    [Measure],
    DimDate[CovidPeriod] = "Pre-COVID (<=2019)"
)
```

**When to use**: Pre vs Post comparisons.

---

### 5. IF(pre < threshold, BLANK(), ...) - Outlier Filtering

**Technique**: Return BLANK() for non-significant values.

```dax
IF(
    pre < 2000,
    BLANK(),
    DIVIDE(...)
)
```

**When to use**: Avoid volatile % change in small countries.

---

## Naming Conventions

### Style used:
- **Measures**: PascalCase with spaces (`Pre-COVID Participants`)
- **Calculated columns**: PascalCase without hyphens (`Duration Bin`)
- **Tables**: PascalCase (`AxisCountry`, `DimDate`)
- **Descriptive suffixes**:
  - `(View)`: Dynamic measure that changes based on slicer
  - `(MinBase)`: Includes minimum volume filter
  - `(Inv)`: Inverted (negative) version

---

## Measures by Page

### Page 1 (Overview)
- `[Participants (View)]` - Dynamic base
- `[Pre-COVID Participants]` - Card
- `[COVID Participants]` - Card
- `[Post-COVID Participants]` - Card
- `[Recovery Rate]` - Card
- `[Title Top10]` - Dynamic title

### Page 2 (COVID Impact on Mobility Flows)
- `[Pct Change Receiving (MinBase)]` - Y-axis scatter
- `[Pct Change Sending (MinBase)]` - X-axis scatter
- `[Pre Total (Sending+Receiving)]` - Bubble size
- `Quadrant` (column) - Color/legend
- `[Pct Change Receiving (Inv)]` - Decreases chart
- `[Pct Change Sending (Inv)]` - Decreases chart

### Page 3 (Participant Profile)
- `[Pre-COVID Participants]` - Distribution comparison
- `[Post-COVID Participants]` - Distribution comparison
- `Duration Bin` (column) - Duration categorization

### Page 4 (Fields of Study)
- `[Pre-COVID Participants]` - Comparative table
- `[Post-COVID Participants]` - Comparative table
- `[% Change]` - % Change in table

---

## Common Troubleshooting

### Error: "A circular dependency was detected"
**Cause**: Measure that references itself indirectly.
**Solution**: Review dependency chain with DAX Studio.

### Error: "The value for column X cannot be determined"
**Cause**: Relationship misconfigured or missing.
**Solution**: Verify data model (active relationships).

### Error: TREATAS() doesn't filter correctly
**Cause**: Data types don't match (e.g., Text vs Int).
**Solution**: Ensure both columns are of the same type.

### Unexpected blank in scatter plot
**Cause**: Country has `pre < 2000` and was filtered.
**Solution**: Adjust threshold in `[Pct Change X (MinBase)]` if too restrictive.

---

## Resources for Learning DAX

- [DAX.guide](https://dax.guide/) - Official function reference
- [SQLBI](https://www.sqlbi.com/) - Advanced tutorials
- [Excelerator BI](https://exceleratorbi.com.au/blog/) - DAX patterns

---

**Author**: Andrés Liz Domínguez
**Last updated**: February 2026
**Total measures**: 20+
**Total calculated columns**: 2
**Total calculated tables**: 3
