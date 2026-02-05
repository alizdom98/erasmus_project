# Erasmus+ Mobility Analysis (2017-2024)

[Read in English](README.md) | [Leer en Español](docs/es/README.md)

**Author:** Andrés Liz Domínguez

This project analyzes Erasmus+ mobility data for higher education, with special emphasis on how COVID-19 affected student mobility patterns in Europe. The analysis covers the period 2017-2024 and works with approximately 2.1 million mobility records.

---

## About the Project

This is a portfolio project I created to demonstrate what I've learned about data analysis and visualization. The goal was to take real (and quite messy) data from the Erasmus+ program and transform it into something useful that could be analyzed in Power BI.

The idea originally came from my undergraduate thesis, but I completely refocused it for my portfolio. I was particularly interested in seeing how student mobility changed after the pandemic: which countries gained or lost students, which fields of study became more popular, and whether the profile of the Erasmus student changed in any way.

### What I've done

- Cleaning and normalization of approximately 6 million records spread across 11 different CSV files
- Unification of data structures that changed between years (columns were not the same across all files)
- Creation of a data model in Power BI with separate dimension tables
- Interactive dashboard with 4 analysis pages
- Complete process documentation (this repository)

### Tools used

- **Python and Pandas**: For all data cleaning
- **Jupyter Notebook**: To document the process step by step
- **Power BI**: For the final dashboard and visualizations
- **DAX**: To create measures and calculated columns in Power BI

---

## Repository Structure

```
erasmus_project/
│
├── README.md                   # This file (English)
├── data_erasmus.ipynb          # Notebook with entire cleaning process (English)
├── data_erasmus_ES.ipynb       # Notebook in Spanish
├── Proyecto_erasmus_bi.pbix    # Power BI Dashboard
├── requirements.txt            # Required Python libraries
│
├── bases_datos_erasmus/        # Original data (download required)
│   └── *.csv                   # The 11 CSV files (2014-2024)
│
├── docs/
│   ├── en/                     # English documentation
│   │   ├── DATA_DICTIONARY.md
│   │   ├── DAX_MEASURES.md
│   │   ├── EXECUTIVE_SUMMARY.md
│   │   ├── NAVIGATION.md
│   │   ├── PIPELINE.md
│   │   ├── POWER_BI_GUIDE.md
│   │   ├── PROJECT_STRUCTURE.md
│   │   └── QUICK_START.md
│   └── es/                     # Spanish documentation
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

## Getting Started

### Prerequisites

- Python 3.8 or higher
- Jupyter Notebook
- Power BI Desktop (if you want to view the dashboard)

### Installation

1. **Clone the repository:**
```bash
git clone https://github.com/alizdom98/erasmus_project.git
cd erasmus_project
```

2. **Install Python dependencies:**
```bash
pip install -r requirements.txt
```

3. **Download the original data:**

The data is NOT included due to its size. Download it from:
- **Official source:** [EU Open Data Portal](https://data.europa.eu/data/datasets?query=erasmus+KA1+mobility&locale=en)
- **Search for:** "Erasmus+ KA1 Mobility Data"
- **Required files:** CSV from 2014 to 2024 (11 files in total)

**Place them in the folder:**
```
bases_datos_erasmus/
├── Erasmus-KA1-Mobility-Data-2014.csv
├── Erasmus-KA1-Mobility-Data-2015.csv
├── Erasmus-KA1-Mobility-Data-2016.csv
├── ...
└── Erasmus-KA1-Mobility-Data-2024.csv
```

**Approximate sizes:** Files range from 14MB (2021) to 232MB (2019). Total: ~1.7GB

4. **Run the notebook** `data_erasmus.ipynb` to process the data

5. **Open the dashboard** `Proyecto_erasmus_bi.pbix` in Power BI Desktop

---

## Dataset

### Original data

The data comes from the European Commission and covers all mobilities of the Erasmus+ KA1 (Key Action 1) program from 2014 to 2024. In total, there are about 6 million records, but I focused on higher education (ISCED 6-8) for the period 2017-2024, which leaves approximately 2.1 million records.

**Why only 2017-2024:** For the Pre/Post-COVID analysis, I didn't need to go back that far. With 2017-2019 as the Pre-COVID period, it was sufficient, and thus reduced the dataset size without losing information relevant to the study.

**Why only higher education:** I filtered the dataset to ISCED 6-8 (higher education) to have a more manageable and focused subset. Additional filters (for example, only Learners for certain analyses) were applied directly in Power BI according to the needs of each visual.

### Processed data

The final dataset has 2,082,071 records and 26 columns. Main variables include:

**Temporal:**
- Academic year (format "2019-20")
- Mobility start date
- Duration in days

**Geographic:**
- Sending and receiving country (with ISO2 code)
- Participant's country of origin

**Demographic:**
- Age, gender
- Whether belonging to groups with "fewer opportunities" (disadvantaged situation)

**Academic:**
- ISCED level (6=Bachelor, 7=Master, 8=Doctorate)
- Field of study (ISCED-F classification)

For more details, see [DATA_DICTIONARY.md](docs/en/DATA_DICTIONARY.md).

---

## Key Findings

### Nearly complete post-COVID recovery

The Erasmus+ program has recovered virtually all its mobility volume. I calculated a "Recovery Rate" that compares Post-COVID (2022-2024) with Pre-COVID (2017-2019), and it comes out to 99.77%. That is, we've almost recovered the level before the pandemic.

**The numbers:**
- Pre-COVID (2017-2019): approximately 2 million mobilities
- COVID (2020-2021): sharp drop to 485,000 mobilities
- Post-COVID (2022-2024): recovery to almost 2 million

The year 2023-24 was a historic record with 950,000 mobilities in a single academic year.

---

### Important geographic changes

#### Countries that grew after COVID

The big winners were countries in southern and eastern Europe:

| Country | Change | Observation |
|---------|--------|-------------|
| Croatia | +37% | The highest relative growth in reception |
| Greece | +35% | Probably due to climate and costs |
| Turkey | +32% | Increasingly attractive programs |
| Romania | +26% | Eastern Europe gaining ground |

**Special case - Ukraine:** Outgoing mobilities increased by 137%. This is clearly related to the war: many Ukrainian students are seeking to study abroad.

#### Countries that lost after COVID

| Country | Change | Why |
|---------|--------|-----|
| Russia | -90% | Ukraine war + international sanctions |
| United Kingdom | -70% | Brexit (no longer participates in Erasmus+) |
| Poland, France, Germany | Moderate drops | Losing share to emerging destinations |

**What's interesting:** Spain, France, and Germany remain the destinations with the highest volume in absolute terms, but they are losing market share to more economical destinations.

---

### Boom in technology, decline in humanities

The most striking change in fields of study:

**Post-COVID winners:**
- ICT (Computer Science): +41.3%
- Natural Sciences: +14.4%
- Agriculture: +10.4%
- Health: +9.7%

**Post-COVID losers:**
- Arts and Humanities: -21.7%
- Services: -7.4%
- Business: -6.6%

**My interpretation:** The digitalization accelerated by the pandemic caused more people to become interested in technology.

Business remains the field with the highest absolute volume, but it is losing relative appeal.

---

### Greater inclusivity

One of the most positive findings: participants with "fewer opportunities" (disadvantaged groups) almost doubled.

- Pre-COVID: 68,128 participants
- Post-COVID: 149,444 participants
- Increase: +119%

**Germany leads this change:** contributed with +38,963 more participants. I'm not entirely sure why, but it could be:
1. Changes in German inclusion policies
2. Better recording of this data
3. The economic impact of COVID made more students qualify as "fewer opportunities"

---

### Shorter mobilities

After COVID, mobilities tend to be shorter:

- 6-9 month mobilities decreased notably
- 3-6 month mobilities remain the most common (one academic semester)
- Very short mobilities (<3 months) increased relatively

**Possible reasons:**
- Economic caution (shorter mobilities cost less)
- Intensive formats (summer schools) gaining popularity
- Fewer people committing to a full year abroad

---

### Constant female predominance

Unlike other changes, the gender distribution did not change significantly between Pre and Post-COVID:
- Both Pre and Post-COVID maintain approximately 60% women, 40% men
- It's remarkable that females clearly predominate in Erasmus+ mobilities
- COVID does not appear to have affected this gender balance

---

## The Power BI Dashboard

I created an interactive dashboard with 4 main pages:

### Page 1: Overview

General view with key metrics and controls to explore. Includes:
- Cards with totals (Pre-COVID, COVID, Post-COVID, Recovery Rate)
- Geographic map with bubbles by country
- Time evolution chart
- Top 10 countries ranking

It has slicers to switch between viewing data of countries receiving students vs. countries sending.

### Page 2: COVID Impact on Mobility Flows

The most complex page. The main visual is a scatter plot with four quadrants:
- **Grow Both:** countries that increased both in sending and receiving
- **Grow Rcv:** countries that receive more but send less
- **Grow Snd:** countries that send more but receive less
- **Down both:** countries that decreased in both

Countries are represented as bubbles (size = total pre-COVID volume) and colored according to their quadrant.

There are also bar charts with the largest increases and decreases.

### Page 3: Participant Profile

Compares Pre vs Post demographic distribution:
- Gender (stable)
- Learner vs Staff profile (stable)
- Fewer opportunities (large increase)
- Mobility duration (shorter post-COVID)

### Page 4: Fields of Study

Interactive table showing changes by ISCED-F field. When you select a field, a side chart shows the main sending countries for that field.

This is where you can clearly see the ICT boom (+41%) and the decline in Arts & Humanities (-22%).

---

## Data Model in Power BI

I used a star schema with:

**Fact table:**
- `df_he_pbi`: the 2.1 million mobility records

**Dimension tables:**
- `Dim_Country_Sending`: sending countries
- `Dim_Country_Receiving`: receiving countries
- `DimDate`: calendar with automatic Pre/COVID/Post classification

**Why two country tables?** At first I tried to use a single DimCountry table, but I realized that to do a flow analysis (Sending vs Receiving) I needed both relationships active simultaneously. With a single table you can only have one active relationship, the other becomes inactive and you have to activate it with `USERELATIONSHIP()` in each measure, which is a mess.

**Auxiliary tables without relationships:**
- `AxisCountry`: a "virtual axis" that joins all countries for cross-sectional analyses
- `Direction`: for the Receiving/Sending slicer

These tables have no physical relationships but are used with `TREATAS()` and `SELECTEDVALUE()` to make dynamic measures.

For more technical details, see [POWER_BI_GUIDE.md](docs/en/POWER_BI_GUIDE.md) and [DAX_MEASURES.md](docs/en/DAX_MEASURES.md).

---

## Limitations and Methodological Decisions

### COVID periods definition

I defined three periods:
- **Pre-COVID:** 2017-2019 (completely normal years)
- **COVID:** 2020-2021 (active restrictions)
- **Post-COVID:** 2022 onwards

**Why is 2021 in COVID and not in Post?** Because in 2021 there were still restrictions in many countries. Mobility did not fully normalize until 2022.

### Minimum volume filter

In the quadrant analysis, I excluded countries with less than 2,000 mobilities in Pre-COVID. Why? Because very small countries have very volatile and not very representative percentage changes. For example, if a country goes from 10 to 30 mobilities, it's +200% but it's not really significant.

### Interpretation of "Fewer Opportunities"

The 119% increase may have multiple causes:
1. Real inclusion (more scholarships, more support)
2. Changes in how this data is recorded
3. Economic impact of COVID (more students qualify)

Without qualitative data from the European Commission, it's difficult to know which is the main cause.

### 2024 data

If the dataset was generated mid-2024, that year's data could be incomplete. There is a seasonal bias: September always has more mobilities than January.

---

## Additional Resources

- [PIPELINE.md](docs/en/PIPELINE.md): Step-by-step explanation of how I cleaned the data
- [QUICK_START.md](docs/en/QUICK_START.md): Quick guide to reproduce the project
- [POWER_BI_GUIDE.md](docs/en/POWER_BI_GUIDE.md): Complete dashboard guide with interpretations
- [DAX_MEASURES.md](docs/en/DAX_MEASURES.md): All Power BI measures explained
- [DATA_DICTIONARY.md](docs/en/DATA_DICTIONARY.md): Description of the 26 final variables
- [EXECUTIVE_SUMMARY.md](docs/en/EXECUTIVE_SUMMARY.md): Executive project summary
- [NAVIGATION.md](docs/en/NAVIGATION.md): Project navigation
- [PROJECT_STRUCTURE.md](docs/en/PROJECT_STRUCTURE.md): Repository structure

---

## Data

The original data is from the European Commission and is under Creative Commons Attribution 4.0 International (CC BY 4.0) license.

Source: https://data.europa.eu/
More info: https://creativecommons.org/licenses/by/4.0/

---

## Contact

Andrés Liz Domínguez

If you have questions about the project or find any errors, open an issue on GitHub.
