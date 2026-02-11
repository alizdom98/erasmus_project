# Erasmus+ Mobility Analysis - Project Summary

[Read in English](EXECUTIVE_SUMMARY.md) | [Leer en Español](../es/EXECUTIVE_SUMMARY.md)

**Author**: Andrés Liz Domínguez
**Date**: February 2026
**Type**: Portfolio Project - Data Science & Business Intelligence

---

## What is this project

This is a data analysis project I created for my portfolio. I worked with 2.1 million Erasmus+ mobility records (higher education, 2017-2024) to see how COVID-19 changed student mobility patterns in Europe.

The original data was quite messy and heterogeneous (11 CSV files with different structures), so an important part of the project was cleaning and transforming it into something usable. Then I created a dashboard in Power BI to visualize the results.

---

## Key Findings

### 1. Nearly complete recovery

The program recovered almost completely after COVID:
- Pre-COVID (2017-2019): approx. 2M mobilities
- COVID (2020-2021): drop to 485K
- Post-COVID (2022-2024): recovery to approx. 2M

The "Recovery Rate" I calculated is 99.77%. That is, we have almost returned to the previous level.

---

### 2. Change in favorite destinations

**The winners** were countries in southern and eastern Europe:
- Croatia: +37%
- Greece: +35%
- Turkey: +32%
- Ukraine: +137% in outgoing (clearly due to the war)

**The losers**:
- Russia: -90% (war + sanctions)
- UK: -70% (Brexit)
- Poland, France, Germany: moderate drops

What's interesting is that Spain, France, and Germany remain the most popular destinations in absolute terms, but they are losing share to more economical options.

---

### 3. Technology boom

The most striking changes in fields of study:

**Increased:**
- ICT (Computer Science): +41% - the biggest winner
- Natural Sciences: +14%
- Agriculture: +10%
- Health: +10%

**Decreased:**
- Arts & Humanities: -22% - the biggest drop
- Services: -7%
- Business: -7%

My interpretation: the digitalization accelerated by COVID caused more people to become interested in technology.

---

### 4. More inclusivity

Participants with "fewer opportunities" (disadvantaged groups) almost doubled:
- Pre-COVID: 68K
- Post-COVID: 149K
- Increase: +119%

Germany was the biggest contributor (+39K participants). I'm not sure of the exact reason, but it could be a change in policies or how this data is recorded.

---

### 5. Shorter mobilities

After COVID there is a trend towards shorter mobilities:
- 6-9 month mobilities decreased
- 3-6 month mobilities remain the most common
- Short mobilities (<3 months) increased relatively

Probably due to economic caution or popularity of intensive programs.

---

## Technical Part

### Data cleaning (Python)

I worked with a 9-phase pipeline on 6 million records:
1. Verify structure of annual files
2. Load and unify the 11 CSVs
3. Normalize dates and academic years
4. Remove duplicates (9,165 records)
5. Clean outliers in age and duration (600K+ values)
6. Normalize text categories
7. Create derived variables (ISCED groups, activity groups)
8. Normalize countries by ISO2 code
9. Filter and prepare the final subset

The result was a clean dataset of 2.1M records × 21 variables.

### Data model (Power BI)

I used a star schema:
- **Fact table**: df_he_pbi with the 2.1M records
- **Dimensions**: Dim_Country_Sending, Dim_Country_Receiving, DimDate
- **Disconnected tables**: AxisCountry (for cross-sectional analysis) and Direction (for slicers)

Why two country tables? At first I tried to use a single DimCountry table, but in Power BI you can only have one active relationship per table. The other becomes inactive and you have to manually activate it with USERELATIONSHIP() in each DAX measure, which is tedious and error-prone. With two separate tables, both relationships are always active and the measures are simpler.

### Dashboard (Power BI)

I created 4 pages:
1. **Overview**: General metrics and time evolution
2. **COVID Impact on Mobility Flows**: Scatter plot with quadrants (which countries gained/lost)
3. **Participant Profile**: Demographic changes
4. **Fields of Study**: Changes in educational fields

The most important DAX measures:
- `Recovery Rate = DIVIDE([Post-COVID Participants], [Pre-COVID Participants])`
- Dynamic measures that change according to slicers with `SELECTEDVALUE()`

---

## Skills I demonstrate

**Technical:**
- Python (Pandas, NumPy) for large-scale data cleaning
- Power BI with advanced DAX
- Data modeling (star schema, role-playing dimensions)
- Handling heterogeneous and messy data

**Analytical:**
- Exploratory analysis of complex datasets
- Insight identification
- Temporal comparisons (pre vs post)
- Trend interpretation

**Communication:**
- Technical process documentation
- Clear and effective visualizations
- Explanation of methodological decisions

---

## Limitations and Learnings

**Known limitations:**
- This analysis is based solely on Erasmus+ mobility data. To explain in depth some changes (why Croatia grew so much, what policies caused the increase in "fewer opportunities" in Germany, etc.) it would be necessary to complement with other country-specific data sources

---

## Available Documentation

| File | Content |
|------|---------|
| [README.md](../../README.md) | Overview and findings |
| [PIPELINE.md](PIPELINE.md) | Step-by-step cleaning process |
| [QUICK_START.md](QUICK_START.md) | How to reproduce the project |
| [POWER_BI_GUIDE.md](POWER_BI_GUIDE.md) | Dashboard guide |
| [DAX_MEASURES.md](DAX_MEASURES.md) | Power BI measures |
| [DATA_DICTIONARY.md](DATA_DICTIONARY.md) | Variable description |
| [NAVIGATION.md](NAVIGATION.md) | Project navigation |
| [PROJECT_STRUCTURE.md](PROJECT_STRUCTURE.md) | Repository structure |

---

## Contact

**Andrés Liz Domínguez**

If you're interested in the project or have questions, you can contact me or open an issue on the GitHub repository.

---

**Last update**: February 2026
