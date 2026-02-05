# Data Dictionary - Erasmus+ HE Dataset (2017-2024)

[Read in English](DATA_DICTIONARY.md) | [Leer en Español](../es/DATA_DICTIONARY.md)

This document describes all variables in the final dataset prepared for analysis in Power BI.

## General Description

- **Dataset**: `erasmus_he_2017_2024.parquet` / `.csv`
- **Records**: 2,082,071
- **Columns**: 26
- **Period**: 2017-2024 (mobility start years)
- **Focus**: Higher Education (ISCED 6-8)

---

## Variables by Category

### Temporal Variables

| Variable | Type | Values | Description |
|----------|------|--------|-------------|
| `academic_year` | string | "2017-18" ... "2024-25" | Academic year in "YYYY-YY" format. Represents the complete academic year during which the mobility occurred. |
| `year_start` | Int64 | 2017-2024 | Numeric year of mobility start. Extracted from academic_year (e.g., "2019-20" → 2019). Useful for filters and time series. |
| `mobility_start_ym` | datetime | 2017-01 ... 2024-12 | Exact date (year-month) of mobility start in datetime format. Allows detailed time series analysis. |
| `mobility_start_year` | Int64 | 2017-2024 | Year of mobility start (duplicate of year_start, maintained for compatibility). |
| `mobility_start_month_num` | Int64 | 1-12 | Month of mobility start (1=January, 12=December). September (9) is the most common month (712,470 mobilities, 34.22%). |

**Notes on temporality**:
- The **academic year** may extend between two calendar years (e.g., 2019-20 includes September 2019 to August 2020).
- The **start year** (year_start) is the calendar year when the mobility begins.
- For trend analysis, use `year_start` or `mobility_start_year`.
- For seasonal analysis, use `mobility_start_month_num`.

---

### Geographic Variables

#### Sending Countries

| Variable | Type | Values | Description |
|----------|------|--------|-------------|
| `sending_country_code` | category | "ES", "FR", "DE", ... | ISO2 code of the sending country (institution sending the student). 33 unique codes. |
| `sending_country_name` | category | "Spain", "France", "Germany", ... | Canonical name of the sending country in English. Normalized by ISO2 code to avoid variants. |

**Full format**: In the original dataset appears as `"ES - Spain"`, but here it's separated into two columns to facilitate analysis and joins.

#### Receiving Countries

| Variable | Type | Values | Description |
|----------|------|--------|-------------|
| `receiving_country_code` | category | "ES", "FR", "DE", ... | ISO2 code of the receiving country (institution receiving the student). 33 unique codes. |
| `receiving_country_name` | category | "Spain", "France", "Germany", ... | Canonical name of the receiving country in English. Normalized by ISO2 code. |

**Top 5 receiving countries (2017-2024)**:
1. Spain (ES)
2. France (FR)
3. Germany (DE)
4. Italy (IT)
5. Portugal (PT)

#### Participant Country

| Variable | Type | Values | Description |
|----------|------|--------|-------------|
| `participant_country` | category | "Spain", "Turkey", "Poland", ... | Participant's country of origin/nationality. 255 unique values (includes non-EU territories). No ISO2 code in the original dataset. |

**Difference with sending_country**:
- `sending_country`: Country of the sending institution (may differ from student's country of origin)
- `participant_country`: Participant's nationality
- Example: A Polish student at a German university going to France would have:
  - participant_country: "Poland"
  - sending_country: "Germany"
  - receiving_country: "France"

---

### Demographic Variables

| Variable | Type | Values | Description | Missing |
|----------|------|--------|-------------|---------|
| `participant_age` | Int64 | 10-65 | Participant age in years. Values outside the 10-65 range were converted to NA (impossible outliers such as negative ages or >100). | 9.69% |
| `participant_gender` | string | "Female", "Male", "Undefined", NA | Participant gender. | 0.00% |
| `participant_profile` | category | "Learner", "Staff", "Other", NA | Participant profile. **Learner** (90.28%): Student. **Staff** (9.72%): Teaching/administrative staff. | 0.1% |
| `fewer_opportunities` | category | "Yes", "No", NA | Indicates whether the participant belongs to a group with **fewer opportunities** (fewer opportunities background) according to Erasmus+ criteria: disadvantaged economic situation, disability, remote rural area, migrant/refugee, etc. | 0.81% |

**Gender distribution (available data)**:
- Female: 59.60%
- Male: 40.27%
- Undefined: 0.13%

**What is "fewer opportunities"?**
It's a European Commission category for participants facing additional barriers:
- Economic disadvantage
- Physical, mental, or health disability
- Educational difficulties
- Cultural differences (migrants, refugees)
- Geographic location (remote rural areas)

---

### Educational Variables

#### ISCED Level

| Variable | Type | Values | Description |
|----------|------|--------|-------------|
| `isced_level` | Int64 | 6, 7, 8 | Educational level according to **ISCED 2011** classification (UNESCO): **6** = Bachelor (undergraduate/bachelor's degree), **7** = Master (master's degree), **8** = Doctorate (PhD). |
| `isced_group` | category | "HE (6-8)" | Simplified group. In this dataset only "HE (6-8)" appears because it's already filtered for higher education. |

**Distribution by level**:
- ISCED 6 (Bachelor): 62.49% of HE mobilities
- ISCED 7 (Master): 33.77%
- ISCED 8 (Doctorate): 3.73%

#### Field of Study

| Variable | Type | Values | Description |
|----------|------|--------|-------------|
| `isced_macro` | category | "01 - Education", "02 - Arts and humanities", ... | Field of study according to **ISCED-F 2013** classification (broad fields, 2 digits). 11 broad categories + "Not classified". |

**ISCED-F Categories (Broad Fields)**:
- **00**: Generic programmes and qualifications
- **01**: Education (teacher training)
- **02**: Arts and humanities
- **03**: Social sciences, journalism and information
- **04**: Business, administration and law
- **05**: Natural sciences, mathematics and statistics
- **06**: Information and Communication Technologies (ICT)
- **07**: Engineering, manufacturing and construction
- **08**: Agriculture, forestry, fisheries and veterinary
- **09**: Health and welfare
- **10**: Services (tourism, sports, personal services)
- **99**: Not classified

**Top 5 fields of study**:
1. **04 - Business, administration and law** (22.80%)
2. **02 - Arts and humanities** (18.44%)
3. **07 - Engineering, manufacturing and construction** (13.90%)
4. **03 - Social sciences, journalism and information** (13.12%)
5. **09 - Health and welfare** (7.40%)

#### Activity Type

| Variable | Type | Values | Description |
|----------|------|--------|-------------|
| `activity_group` | category | "HE", "Staff/Training", "Youth/Volunteering", "VET", "School", "Adult" | Type of mobility activity grouped. For this HE dataset, most is **"HE"** (student mobility for studies/traineeships). |

**Values in this dataset**:
- **HE** (90.66%): Student Mobility for Studies (SMS) or Traineeships (SMT) in higher education
- **Staff/Training** (6.16%): Teaching/administrative staff mobility (although not students, they're in HE context)
- **Other** (3.18%): Other activities in higher education context

---

### Metrics Variables

| Variable | Type | Values | Description | Missing |
|----------|------|--------|-------------|---------|
| `mobility_duration` | float64 | 1-845 | Mobility duration in **calendar days**. Values = 0 were converted to NA. | 0.38% |
| `actual_participants` | Int64 | 1-... | Number of participants for aggregated records. In most cases = 1 (one record per participant). Values with decimals or = 0 were converted to NA. | ~0% |

**Duration statistics**:
- **Minimum**: 1 day (very short preparatory visits)
- **Median**: 130 days (~4 months, typical duration of an academic semester)
- **Mean**: 129.3 days (almost symmetric distribution centered on one semester)
- **Maximum**: 845 days (~2.3 years, probably doctoral programs or double degrees)

**Typical duration ranges**:
- **1-90 days**: Short mobilities (summer, intensive, visits)
- **90-180 days**: Academic semester (~3-6 months) - most frequent range
- **180-365 days**: Full academic year or long programs
- **>365 days**: Doctoral programs, double degrees (very rare)

---

### Flags and Control Variables

| Variable | Type | Values | Description |
|----------|------|--------|-------------|
| `he_strict` | boolean | True, False | Conservative flag indicating whether the record meets strict HE criteria: **ISCED 6-8 AND activity_group = "HE"**. True for 90.66% of the dataset. False includes staff mobility in HE institutions and edge cases. |

**When to use he_strict?**
- **False (entire dataset)**: Broad analysis of mobilities related to higher education (includes staff)
- **True (strict filter)**: Conservative analysis of mobility **only of undergraduate, master's, and doctoral students**

---

## Use Cases by Variable

### Temporal Analysis
- `year_start`, `academic_year`: Annual trends, pre/post-COVID analysis
- `mobility_start_month_num`: Seasonality (September is peak)
- `mobility_start_ym`: Detailed time series

### Geographic Analysis
- `sending_country_code`, `receiving_country_code`: Mobility flows between countries
- `participant_country`: Most mobile nationalities
- Maps in Power BI: Use `_code` (ISO2) for automatic recognition

### Demographic Analysis
- `participant_age`: Age distribution (peak at 20-24 years)
- `participant_gender`: Gender balance in mobility
- `fewer_opportunities`: Program inclusivity

### Educational Analysis
- `isced_level`: Compare Bachelor vs Master vs Doctorate
- `isced_macro`: Popularity of fields of study
- `activity_group` + `he_strict`: Differentiate students vs staff

### COVID Impact Analysis
- Compare `year_start` 2017-2019 (pre) vs 2021-2024 (post)
- Filter by receiving country to see which destinations recovered fastest

---

## Known Limitations

### Missing Values (% of dataset)

| Variable | Missing | Reason |
|----------|---------|--------|
| `participant_age` | 9.69% | Optional field in the form + outliers converted to NA |
| `isced_macro` | 1.88% | Field of study not always recorded |
| `fewer_opportunities` | 0.81% | Not always declared |
| `mobility_duration` | 0.38% | Values = 0 converted to NA |
| `participant_gender` | 0.00% | Complete data in HE 2017-2024 dataset |

### Cities Excluded

The variables `sending_city` and `receiving_city` are **NOT included** in this final dataset due to:
- Too many unique categories (46K and 94K respectively)
- Encoding problems (mojibake) in ~1.4% of records
- Low utility for aggregated analysis in Power BI

If city-level analysis is needed, it can be recovered from the intermediate dataset `df_he` (cell 27 of the notebook).

### Excluded Period

Years 2014-2016 were excluded because for the Pre/Post-COVID analysis it wasn't necessary to go that far back. With 2017-2019 as the Pre-COVID period it was sufficient, which allowed reducing the data size to analyze without losing relevant information.

---

## References

- **ISCED 2011**: [UNESCO - International Standard Classification of Education](http://uis.unesco.org/en/topic/international-standard-classification-education-isced)
- **ISCED-F 2013**: [UNESCO - Fields of Education and Training](https://uis.unesco.org/en/topic/isced-fields-education-and-training)
- **ISO 3166-1 alpha-2**: [Two-letter country codes](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2)
- **Erasmus+ Programme Guide**: [European Commission](https://erasmus-plus.ec.europa.eu/programme-guide)

---

**Last updated**: February 2026
**Dataset version**: v1.0
