# Power BI Dashboard Guide

[Read in English](POWER_BI_GUIDE.md) | [Leer en Español](../es/POWER_BI_GUIDE.md)

This document explains how the dashboard I created in Power BI works and the decisions I made in the data model.

---

## General Structure

The dashboard has 4 main pages, each focused on a different aspect of the analysis:

1. **Overview**: Interactive general view
2. **COVID Impact on Mobility Flows**: Analysis of changes in sending and receiving flows by country (quadrant scatter plot)
3. **Participant Profile**: Changes in demographic profile
4. **Fields of Study**: Changes in educational fields

---

## Page 1: Overview

This is the entry page. I thought of it as a "general explorer" where you can quickly get an idea of how things are.

**Main visuals:**

- **Top cards**: Show totals for Pre-COVID, COVID, Post-COVID, and Recovery Rate (99.77%)
- **Geographic map**: With bubbles proportional to the number of participants using the measure `[Participants (View)]`. When you click on a country, it filters everything else. **Note**: It's filtered to the top 30 countries to avoid having bubbles everywhere and saturating the map
- **Timeline chart**: Shows month-by-month evolution with the 3 periods in different colors (Pre-COVID, COVID, Post-COVID). Clear peaks in September (start of academic year) are visible
- **Top 10 ranking**: Shows the 10 most active countries according to selected direction

**Slicers (controls):**

- **COVID Period**: Pre-COVID / COVID / Post-COVID
- **Direction (Flow type)**: Receiving / Sending

The direction slicer is interesting because it dynamically changes **all visuals** on the page. If you select "Receiving", you see data for countries receiving students. If you select "Sending", you see countries sending. It affects:
- **Geographic map**: Bubble size changes according to selected direction
- **Top 10 ranking**: Shows top receivers or top senders
- **Cards**: Pre/COVID/Post/Recovery values change according to chosen perspective
- **Temporal chart**: Line reflects the metric corresponding to the direction

The chart title also changes automatically using a DAX measure:
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

## Page 2: COVID Impact on Mobility Flows

This is the most technically complex page. The goal was to see which countries "won" and which "lost" after COVID, differentiating between:

- **Sending**: Countries that **send** students to other countries (mobility origins)
- **Receiving**: Countries that **receive** students from other countries (mobility destinations)

**Main visual: Quadrant Scatter Plot**

This chart shows Post-COVID vs Pre-COVID change in two dimensions simultaneously:

- **X-axis (horizontal)**: % Change Sending - How much the country changed as **origin** of students
- **Y-axis (vertical)**: % Change Receiving - How much the country changed as **destination** of students
- **Bubble size**: Pre Total (Sending + Receiving) - Country's total Pre-COVID volume
- **Color**: Quadrant to which it belongs

**Change formula:** `DIVIDE(Post - Pre, Pre)` for Sending and Receiving separately.

**The four quadrants (what they mean):**

1. **Grow Both** (top-right): Country increased BOTH in sending and receiving students
   - Example: Countries that became more active in Erasmus+ in both directions

2. **Grow Rcv** (top-left): Country **receives more** students but **sends fewer**
   - Example: Emerging destinations that attract more but their students go out less

3. **Grow Snd** (bottom-right): Country **sends more** students but **receives fewer**
   - Example: Ukraine (+137% sending) - due to war, more Ukrainian students study abroad

4. **Down both** (bottom-left): Country decreased in both directions
   - Example: Russia and UK due to war/Brexit respectively

**Minimum volume filter (Pre < 2,000 excluded):**

This filter applies to **all charts** on the page. I excluded countries with less than 2,000 total Pre-COVID mobilities because their percentages are very volatile. A country that goes from 10 to 30 mobilities is +200% but not significant.

**Side bar charts (4 charts):**

The 4 charts complement the scatter by showing specific rankings:

**Top left - Largest Increases (Receiving):**
- Top countries that most **increased as destinations**
- Croatia +37%, Greece +35%, Turkey +32%, etc.
- These countries receive more students Post-COVID than Pre-COVID

**Bottom left - Largest Decreases (Receiving):**
- Top countries that most **decreased as destinations**
- Russia -90%, UK -70%, Poland, France, Germany (moderate drops)
- These countries receive fewer students Post-COVID

**Top right - Largest Increases (Sending):**
- Top countries that most **increased as origins**
- Ukraine +137% (war), Estonia +59%, Romania, Cyprus, etc.
- These countries send more students Post-COVID

**Bottom right - Largest Decreases (Sending):**
- Top countries that most **decreased as origins**
- Russia -90%, UK -87%, Serbia, Denmark, Turkey (send fewer)
- These countries send fewer students Post-COVID

**Interactivity:**

When you click on a country in the scatter plot, it **highlights** in the side bar charts **only if that country appears in them**. If the country is not in the top rankings, nothing is highlighted in the sidebars (because it's not visible in those charts).

**Technical note**: Highlighting is used instead of cross-filtering so you can see the full ranking context while visually identifying where the selected country is.

**Key findings from this page:**

- **Winners in Receiving**: Southern and Eastern Europe (Croatia, Greece, Romania, Turkey) - emerging more economical destinations
- **Losers in Receiving**: Russia (war), UK (Brexit), and loss of share by France/Germany/Poland
- **Surprise in Sending**: Ukraine +137% - War caused more Ukrainian students to study abroad
- **Losers in both**: Russia and UK fell in both sending and receiving due to geopolitical reasons

---

## Page 3: Participant Profile

Here I analyzed whether the Erasmus student profile changed after COVID.

**Distribution charts (Pre vs Post):**

- **Gender Distribution**: Remains stable between Pre and Post COVID, although we can observe that females predominate with around 60/40 (59.6% F, 40.3% M), which is remarkable
- **Profile Distribution** (Learner vs Staff): Also stable (**90.3% Learner, 9.7% Staff**)
- **Fewer Opportunities Distribution**: **Large increase** - almost doubles:
  - Pre-COVID: 68,128 participants
  - Post-COVID: 149,444 participants
  - Increase: **+81,316 (+119%)**

**Bar chart - Top Contributors to "Fewer Opportunities" Increase:**

Shows which countries contributed most to the increase in participants with fewer opportunities. Germany leads with +38,963 participants.

**Why did "Fewer Opportunities" increase so much?** The exact reason is not clear in the data, but it could be due to:
- Changes in EU inclusivity policies after COVID
- Better recording/declaration of this data in the Erasmus+ system
- Greater effort by countries like Germany to include disadvantaged groups

**Duration distribution (Pre vs Post):**

A histogram comparing duration ranges. **Note**: This chart is filtered to `participant_profile = Learner` because for Staff all mobilities are less than 1 month, which would distort the comparison.

It's clearly seen that:
- 6-9 month mobilities decreased
- 3-6 months remain the most common
- Short mobilities (<3 months) increased relatively

**Interpretation:** After COVID, people prefer shorter mobilities (probably due to economic caution or preference for intensive programs).

---

## Page 4: Fields of Study

Here you see which fields of study gained or lost popularity. **Note**: This page is filtered to `activity_group = HE` (Higher Education) and `participant_profile = Learner` to focus on university student mobility.

**Main visual:**

A horizontal bar chart showing all ISCED-F fields with stacked bars:
- Yellow/gold: Pre-COVID
- Blue: Post-COVID

**Interactive table:**

Shows % Change by field. When you select a row, a side chart shows the top sending countries for that field with **stacked bars (Pre-COVID in yellow/gold, Post-COVID in blue)**, allowing comparison of each country's volume in both periods.

**Key findings:**

**Big winners:**
- ICT: +41.3% (tech boom)
- Natural sciences: +14.4%
- Agriculture: +10.4%
- Health: +9.7%

**Big losers:**
- Arts & Humanities: -21.7%
- Services: -7.4%
- Business: -6.6%

---

## Data Model

This is where I made some important decisions.

### Star Schema

I designed the model as a classic star:

```
        df_he_pbi (Fact Table)
              |
    __________|__________
   |          |          |
Dim_Country  Dim_Country  DimDate
_Sending     _Receiving
```

**Fact table:**
- `df_he_pbi`: The 2.1 million mobility records

**Dimension tables:**
- `Dim_Country_Sending`: Sending countries (33 unique countries)
- `Dim_Country_Receiving`: Receiving countries (33 unique countries)
- `DimDate`: Date table with automatic Pre/COVID/Post classification

**Relationships:**
- `Dim_Country_Sending[sending_country_code]` → `df_he_pbi[sending_country_code]` **(1:M, active)**
- `Dim_Country_Receiving[receiving_country_code]` → `df_he_pbi[receiving_country_code]` **(1:M, active)**
- `DimDate[Date]` → `df_he_pbi[mobility_start_ym]` **(1:M, active)**

All relationships follow **one-to-many (1:M)** cardinality, which is standard in a star model: one row in the dimension relates to many rows in the fact table.

### Why TWO Country Tables?

At first I tried using a single `DimCountry` table with two relationships:
- One active for `sending_country_code`
- One inactive for `receiving_country_code`

**The problem:** For flow analysis (Sending × Receiving) I needed both relationships active simultaneously. With a single table you can only have one active relationship. The other remains inactive and you have to use `USERELATIONSHIP()` in each measure, which is tedious.

**The solution:** Split them into two separate tables. Now both relationships are active all the time.

This is what's called a "role-playing dimension" in data modeling: the same entity (Country) appears in different roles (Sending vs Receiving). It's similar to how in a sales model you might have DimCustomer and DimSalesperson (both are people, but they play different roles).

### Auxiliary Tables (without physical relationships)

**AxisCountry:**

It's a calculated table that joins all country codes:

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

**What's it for?** Acts as a "virtual axis" for cross-cutting analysis. It has no physical relationships with the fact table, but I use it with `TREATAS()` to filter dynamically:

```dax
Receiving Participants =
CALCULATE(
    [Participants],
    TREATAS(VALUES(AxisCountry[country_code]),
            df_he_pbi[receiving_country_code])
)
```

This allows me to calculate metrics by country regardless of whether it's sender or receiver.

**Direction:**

Another table without relationships, with only two rows:

```dax
Direction = DATATABLE("Mode", STRING, {
    {"Receiving (Destination)"},
    {"Sending (Origin)"}
})
```

I use it for the perspective slicer (Receiving/Sending). Measures read the selected value with `SELECTEDVALUE(Direction[Mode])` and change their behavior.

### DimDate

I created this table with DAX instead of importing it:

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

The `CovidPeriod` column automatically classifies each date. This makes it very easy to filter by period without having to write repetitive conditions.

**Why is 2021 in COVID and not Post?** Because in 2021 there were still mobility restrictions in many European countries. Complete normalization didn't arrive until 2022.

### Power Query Transformations

The model includes some transformations in Power Query that I made to clean the data structure:

**Fact table duplication:**

When loading the data, I duplicated the fact table (instead of referencing it) before creating the dimension tables `Dim_Country_Sending` and `Dim_Country_Receiving`.

**Why?** I wanted to remove the country name columns from the final fact table (to avoid redundancy, since that data is in the dimensions). The problem is that if you create tables by reference and then delete columns from the original table, they're also deleted from the referenced tables.

**The solution:** Duplicate instead of reference. This creates an independent copy. So I could:
1. Create the country dimensions by reference from the duplicate
2. Remove the country name columns from the duplicate
3. The dimensions keep the columns because they were created before deleting them

**Alternatives I considered:**
- Simply not delete the columns (accept redundancy) - valid in Power BI
- Hide the columns in the model instead of deleting them in Power Query
- Create the dimensions independently without references

Duplication works well although it uses a bit more memory during loading.

---

## Main DAX Measures

### Recovery Rate

```dax
Recovery Rate =
DIVIDE([Post-COVID Participants], [Pre-COVID Participants])
```

Simple but effective. Compares Post/Pre and gives 0.9977 (99.77%).

### Pct Change (with minimum volume filter)

```dax
Pct Change Receiving (MinBase) =
VAR pre = [Pre Receiving]
VAR pos = [Post Receiving]
RETURN
IF(
    pre < 2000,  -- Minimum volume filter
    BLANK(),
    DIVIDE(pos - pre, pre)
)
```

The `IF(pre < 2000, BLANK(), ...)` is crucial. Without it, countries with 10 mobilities that go to 30 would show +200% and distort the scatter plot.

### Quadrant (calculated column in AxisCountry)

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

Classifies each country into one of the 4 quadrants based on its change in sending and receiving.

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

This measure "reads" which option is selected in the Direction slicer and returns the corresponding metric. It's what makes the visuals change dynamically.

### Duration Bin (calculated column in df_he_pbi)

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

Groups durations into ranges for the Page 3 chart.

---

## Interactions

**Cross-filtering:**

Visuals are configured so that:
- Click on map → filters ranking and updates cards
- Click on ranking bar → filters map
- Slicers → filter entire page

**Cross-filtering (Page 4):**

In the Fields of Study table (Page 4), when you select a row, the side chart automatically updates to show the top sending countries for that field.

---

## Limitations and Considerations

**1. Period definition:**
The decision to put 2021 in COVID instead of Post is debatable. Some might argue it should be in Post. This would slightly affect the Recovery Rate.

**2. Minimum volume filter:**
Excludes small countries from quadrant analysis. If you're specifically interested in small countries, a separate analysis would be needed.

**3. Causality vs Correlation:**
The dashboard shows correlations (e.g., UK dropped after Brexit) but cannot prove causality. There are always multiple factors.

---

## Additional Resources

- [DAX_MEASURES.md](DAX_MEASURES.md): Complete documentation of all measures with examples
- [DATA_DICTIONARY.md](DATA_DICTIONARY.md): What each variable means
- [PIPELINE.md](PIPELINE.md): How the data was cleaned
- [QUICK_START.md](QUICK_START.md): Quick start guide

---

**Author**: Andrés Liz Domínguez
**Last updated**: February 2026
