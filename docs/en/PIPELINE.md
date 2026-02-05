# Data Cleaning Pipeline - Complete Process

[Read in English](PIPELINE.md) | [Leer en Español](../es/PIPELINE.md)

This document explains step by step how I cleaned the Erasmus+ program data. I tried to document not only what I did, but also why I made each decision.

---

## Context

The original data comes from 11 CSV files (one per year, 2014-2024) published by the European Commission. In total there are about 6 million records, but there were several problems:

1. Columns didn't have the same names across all years
2. Some years had 20 columns, others 19
3. Encodings varied (UTF-8, UTF-8-BOM, Latin1...)
4. There were many duplicates, outliers and strange values
5. Country names were not consistent

The goal was to transform all of this into a clean and usable dataset.

---

## Phase 1: Structure Verification

**The first thing I did** was see what differences existed between the files before loading them all. I didn't want to encounter surprises halfway through the process.

### The Problem

Each year could have:
- Different column names (`organisation` vs `organization`)
- Columns that appear in some years and not others
- Different formats (for example, dates as `"2019/09"` vs `"2019-09"`)

### What I Did

I created a function that:
1. Reads only the headers of each CSV (without loading all the data)
2. Normalizes column names (lowercase, underscores)
3. Compares columns between years

```python
def normalize_columns(columns):
    normalized = []
    for col in columns:
        col = col.strip().lower()
        col = re.sub(r"\s+", " ", col)  # Collapse multiple spaces
        col = col.replace(" ", "_")     # Spaces to underscores
        normalized.append(col)
    return normalized
```

### What I Found

- **Years 2014-2019**: 20 columns
- **Years 2020-2024**: 19 columns (missing `project_reference`)
- Name changes:
  - `sending_organisation` → `sending_organization` (British to American)
  - `mobility_duration_-_calendar_days` → `mobility_duration` (simplified)
  - `mobility_start_year/month` → `mobility_start_month` (cleaner format)

**Why this is important:** If I hadn't detected it beforehand, I would have encountered strange errors when trying to merge the files.

---

## Phase 2: Loading and Unification

Now it was time to load all the files and merge them into one.

### The Encoding Problem

Not all files used the same encoding. Some were UTF-8, others UTF-8-BOM (with byte order mark), and others Latin1. If you use the wrong encoding, special characters (accents, ñ) come out wrong.

### Solution

I created a function that tries several encodings until one works:

```python
def read_csv_with_fallback(path, sep=";"):
    encodings = ["utf-8", "utf-8-sig", "latin1"]
    for enc in encodings:
        try:
            df = pd.read_csv(path, sep=sep, encoding=enc)
            return df, enc
        except:
            continue
    raise Exception("Could not read file with any encoding")
```

### Column Unification

Since the columns were not equal between years, I did this:

1. Normalized names with the previous function
2. Applied a "rename map" to unify variants:
   ```python
   RENAME_MAP = {
       "sending_organisation": "sending_organization",
       "mobility_duration_-_calendar_days": "mobility_duration",
       # etc.
   }
   ```
3. Removed junk columns like "Unnamed: 0" that sometimes appeared
4. Added a `source_file_year` column to know which year each record came from
5. Aligned all columns (if a column is missing in a year, I fill it with NaN)
6. Concatenated everything into a single DataFrame

### Result

**5,955,075 records** with **21 unified columns**.

---

## Phase 3: Temporal Normalization

The date data was in inconsistent formats and needed to be standardized.

### academic_year

The academic year came in two formats:
- `"2020-2021"` (complete)
- `"2020-21"` (reduced)

I needed them all to follow the same format. I chose `"YYYY-YY"` because it is the official Erasmus+ format.

```python
def norm_academic_year(x):
    if pd.isna(x):
        return pd.NA
    s = str(x).strip()

    # "2020-2021" → "2020-21"
    m = re.fullmatch(r"(\d{4})-(\d{4})", s)
    if m:
        return f"{m.group(1)}-{m.group(2)[-2:]}"

    # "2020-21" → leave as is
    return s
```

### mobility_start_month

This column came as text (`"2019-09"`) but I needed it as a date to be able to:
- Filter by dates
- Extract year and month separately
- Create time series in Power BI

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

## Phase 4: Duplicate Cleaning

I found 9,165 exact duplicate records (0.15% of the total). These are identical rows in all columns.

### Are They All Errors?

I analyzed two types of "duplicates":
1. **Exact duplicates**: Same row repeated → Clearly an error
2. **Multi-year repeats**: Same information but in different academic years → Could be legitimate (a student who goes twice)

I found only 96 cases of the second type, which I decided to keep because they could be real.

### What I Did

```python
df_total = df_total.drop_duplicates(keep="first").reset_index(drop=True)
```

I kept the first appearance of each duplicate.

**Result:** 5,945,910 records (removed 9,165).

---

## Phase 5: Numeric Variable Cleaning

Here I found many impossible values that needed to be cleaned.

### actual_participants

I found ONE record with value `4.7`. You can't have 4.7 participants. There were also some with value `0`.

Solution: Convert to NA values with decimals or equal to 0, and use type `Int64` (nullable integer) instead of float.

### participant_age

**Problems found:**
- 82,785 records with age ≤ 0 (including negative ages like `-1`, `-2`)
- 16,993 records with age ≥ 65 (including extreme outliers like `1922`, `823`)

**Why these values exist:**
- Negatives are probably "not specified" codes from the source system
- `0` indicates data not available
- Extreme values are data entry errors (someone typed the year instead of the age)

**Solution:** I defined a reasonable range (10-65 years) based on:
- Minimum 10: Erasmus+ includes some school mobilities (although they are a minority)
- Maximum 65: Reasonable limit for university students

```python
age = pd.to_numeric(df_total["participant_age"], errors="coerce")
age = age.mask((age < 10) | (age > 65))
df_total["participant_age"] = age.astype("Int64")
```

**Result:** 621,084 null values (10.5% of the dataset), but the ones that remain are reasonable.

### mobility_duration

16,069 records had duration = 0 days, which doesn't make sense. I converted them to NA.

I also saw very long durations (>730 days, i.e. >2 years), but I left them because they could be legitimate (joint doctorate programs, dual degrees).

**Statistics after cleaning (HE dataset 2017-2024):**
- Minimum: 1 day
- Median: 130 days (approx. 4 months, an academic semester)
- Mean: 129.3 days
- Maximum: 845 days (approx. 2.3 years)

---

## Phase 6: Categorical Value Normalization

In text columns there were many values that actually meant "unknown" but were written in different ways.

### Null Tokens

I found these variants:
- `"unknown"`, `"not specified"`, `"none"`, `"n/a"`, `"na"`
- `"?"`, `"??"`, `"???"`, `"????"`
- `"-"`, `"--"`, `"---"`
- `"_"`, `"__"`
- Empty strings

**What I did:** Convert all these values to `pd.NA` (official pandas null).

```python
missing_tokens = {"unknown", "not specified", "none", "n/a", "na",
                  "?", "??", "???", "-", "--", "_", "__"}

for col in text_cols:
    s = df_total[col].str.strip().str.lower()
    df_total[col] = s.mask((s == "") | (s.isin(missing_tokens)), pd.NA)
```

**Special case:** In `receiving_city` the string `"0"` appeared which is clearly not a city. I also converted it to NA.

### participant_profile

Values in different formats: `"LEARNERS"` vs `"Learner"` vs `"STAFF"` vs `"Staff"`.

I unified to:
- `"Learner"` for students
- `"Staff"` for personnel

And converted the column to `category` type to save memory.

### fewer_opportunities

It came coded as `0`/`1` (sometimes text, sometimes number). I converted it to `"No"`/`"Yes"` to make it more readable.

**What does "fewer opportunities" mean?** These are students with a disadvantaged situation: economic, disability, rural areas, migrants/refugees, etc.

---

## Phase 7: Data Enrichment

Here I created new variables from existing ones.

### ISCED Level and Groups

The `education_level` column had long texts like:
```
"ISCED-6 - First cycle / Bachelor's degree or equivalent (EQF 6)"
```

I extracted only the ISCED number:

```python
df_total["isced_level"] = df_total["education_level"]\
    .str.extract(r"ISCED-(\d)", expand=False)\
    .astype("Int64")
```

And created simplified groups:
- **HE (6-8)**: Higher education (Bachelor, Master, Doctorate)
- **Pre-tertiary (1-5)**: Previous levels
- **ISCED-9 / Other**: Others

**Result:** 50.8% of the dataset (3.02M) is higher education.

### Activity Groups

The `activity_mob` column had more than 40 different types. I grouped them into 6 broad categories:

- **HE** (Higher Education): University student mobility
- **VET** (Vocational Education): Vocational training
- **Youth/Volunteering**: Youth exchanges, volunteering
- **Staff/Training**: Staff mobility
- **School**: School mobility
- **Adult**: Adult education

I used both codes (when they existed) and text keywords to classify.

### ISCED-F Broad Fields

The `field_of_education` column had 4-digit codes like `"0410 - Business and administration"`. I reduced them to 2 digits to have broad categories:

- **00**: Generic programs
- **01**: Education
- **02**: Arts and humanities
- **03**: Social sciences
- **04**: Business, administration, law
- **05**: Natural sciences, mathematics
- **06**: ICT
- **07**: Engineering, manufacturing
- **08**: Agriculture, veterinary
- **09**: Health and welfare
- **10**: Services
- **99**: Not classified

For records without a code, I used text keywords to classify them.

---

## Phase 8: Geographic Normalization

This was one of the most complicated parts.

### The Problem with Countries

The same country could appear written in many ways:
- `"Türkiye"` vs `"Turkey"`
- `"Czechia"` vs `"Czech Republic"`
- `"Iran (Islamic Republic of)"` vs `"Iran"`
- In sending/receiving: `"ES - Spain"` vs `"ES - España"` (same code, different name)

**Why this is a problem:** In Power BI, if a country has 2 different names, it appears twice in charts and tables, fragmenting the data.

### Normalization Strategy

**For `participant_country`** (name only, no code):
I created a manual equivalence dictionary:

```python
name_map = {
    "Türkiye": "Turkey",
    "Czechia": "Czech Republic",
    "Iran (Islamic Republic of)": "Iran",
    # ...30+ more variants
}
```

**For `sending_country` and `receiving_country`** (format "XX - Name"):
I normalized by ISO2 code:

1. Extracted the code (XX) and name separately
2. For each code, chose the most frequent name as "canonical"
3. Applied this canonical name to all records with that code

```python
# Example: all records with code "ES" use "Spain" (the most frequent)
canon_name = df.groupby("code")["name"]\
    .agg(lambda x: x.value_counts().idxmax())
```

4. Created separate columns for code and name:
   - `sending_country_code`
   - `sending_country_name`
   - (and the same for receiving)

**Why separate code and name?** In Power BI:
- The code (ISO2) is used for relationships and maps (Power BI recognizes it automatically)
- The name is used for visualization (more readable for the user)

**Global standard:** I ensured that sending and receiving use the same canonical name per code, so "ES" is always "Spain" in both columns.

### The Problem with Cities

Cities had several problems:
- Extra spaces
- Postal suffixes like "Lyon CEDEX 07"
- Districts: "Paris 16", "Dublin 2"
- Inconsistent accents: "Málaga" vs "Malaga"
- Exonyms: "Praha" vs "Prague", "Wien" vs "Vienna"
- **Mojibake**: Incorrectly encoded characters like "Cefal�" instead of "Cefalù"

**What I did (basic cleaning):**
- Normalized spaces
- Removed CEDEX suffixes and district numbers
- Removed accents (Unicode normalization)
- Unified common exonyms to English
- Capitalized correctly (Title Case)

**What I did NOT do:**
- Did not fix mojibake because it's already in the original data and there's no way to know the correct character with certainty (affects ~1.4% of records)
- Did not normalize 100% of city names - a good number are misspelled
- Complete normalization would require country-by-country analysis, which would take too long

**Result:**
- 255 unique countries in `participant_country`
- 0 codes with multiple names in sending/receiving (100% normalized)
- 94,344 unique cities in receiving_city
- 46,021 unique cities in sending_city

**Important note:** Due to these limitations in city name quality (misspellings, mojibake, high cardinality), cities were NOT included in the final Power BI analysis. The analysis focuses on countries, which are fully normalized.

---

## Phase 9: Final Dataset Preparation

With everything cleaned, I prepared the specific subset for my analysis.

### Filters Applied

**By period:** Only 2017-2024

**Why not include 2014-2016?**
For the Pre vs Post COVID analysis I didn't need to go back that far. With 2017-2019 as the Pre-COVID period it was sufficient, and this way I could reduce the size of data to analyze without losing relevant information for the study.

**By education type:** Only ISCED 6-8 (higher education)

**Why this criterion?**

I compared several criteria to define what is "Higher Education":

| Criterion | Records | % of 2017-2024 |
|----------|-----------|-------------------|
| **ISCED 6-8** (chosen base criterion) | 2,082,071 | 49.80% |
| activity_group == "HE" | 2,058,366 | 49.23% |
| field == "Higher Education" | 2,312,575 | 55.31% |
| **ISCED 6-8 AND activity_group == "HE"** (strict) | 1,887,691 | 45.15% |

**Decision:** I chose **ISCED 6-8** as the base criterion because:
- It is the most robust international standard (UNESCO)
- Includes Bachelor (6), Master (7) and Doctorate (8)
- Covers HE mobilities of students and staff in university context

I also created an `he_strict` flag to allow more conservative analyses:

```python
he_strict = (ISCED 6-8) AND (activity_group = "HE")  # Only HE students, excludes staff
```

This flag allows filtering to only students (excludes staff mobility) if a stricter analysis is needed. Additional filters (for example, only Learners) were applied directly in Power BI according to the needs of each visual.

### Column Selection

From the 37 columns I had at that point, I selected only 26 for the final analysis:

**Included:**
- All temporal variables
- Country codes and names (separately)
- Demographic variables
- Educational classifications (ISCED, fields of study)
- Metrics (duration, age, participants)
- Flag he_strict

**Excluded:**
- Cities (too many unique categories + mojibake problems)
- Organizations (long and inconsistent names)
- project_reference (41% missing)
- Long text columns already processed (education_level, field_of_education, activity_mob)
- Technical variables (source_file_year)

### Final Dataset

**2,082,071 records** × **26 columns**

Size in memory: ~413 MB
Exported size (Parquet): ~80 MB
Exported size (CSV): ~400 MB

---

## Pipeline Summary

| Phase | Input | Output | Change |
|------|-------|--------|--------|
| 1. Verification | 11 CSVs | Difference report | Structure analysis |
| 2. Loading | 5,955,075 records | Unified df_total | +source_file_year |
| 3. Temporal | Mixed formats | Normalized dates | Unified academic_year |
| 4. Duplicates | 9,165 duplicates | 5,945,910 records | -0.15% |
| 5. Numerics | Outliers | Clean values | 621K ages to NA |
| 6. Categoricals | Mixed tokens | Unified categories | Standardized nulls |
| 7. Enrichment | Long text | Classifications | +8 variables |
| 8. Geography | Inconsistent names | ISO2 + canonical | 255 countries normalized |
| 9. HE Subset | 5.9M mixed | 2.1M HE (2017-2024) | ISCED 6-8 filter |

---

## Lessons Learned

### What Worked Well

- **Verify before loading**: Reviewing structures first saved me many headaches
- **Functions with fallback**: The function that tries multiple encodings was essential
- **Documenting decisions**: Writing the "why" helped me when I returned to the code weeks later
- **Creating intermediate variables**: Keeping both mobility_start_month (text) and mobility_start_ym (date) gave flexibility

### What Was Harder Than Expected

- **Country normalization**: I thought it would be simple but there are many variants and special cases
- **Deciding what is an outlier**: It's not always clear. Is a duration of 800 days an error or real? I had to research the program to understand what values were possible
- **Balance between cleaning and preserving**: Sometimes I wasn't sure if a strange value was an error or real data. When in doubt, I preferred to convert to NA instead of deleting the entire record

### If I Did It Again

- I would do the exploratory data analysis (EDA) earlier to better understand the data before cleaning
- I would try with a small sample first (e.g. just one year) before processing everything
- I would document special cases and exceptions in a separate file

---

## References

- **ISCED 2011**: http://uis.unesco.org/en/topic/international-standard-classification-education-isced
- **ISCED-F 2013**: http://uis.unesco.org/en/topic/international-standard-classification-education-isced
- **ISO 3166-1 alpha-2**: https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2
- **Erasmus+ Programme Guide**: https://erasmus-plus.ec.europa.eu/programme-guide

---

**Author**: Andrés Liz Domínguez
**Date**: February 2026
