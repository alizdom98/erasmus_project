# Quick Start Guide

[Read in English](QUICK_START.md) | [Leer en Español](../es/QUICK_START.md)

There are two ways to use this project: simply view the finished dashboard, or reproduce the entire process from scratch.

---

## Option A: I just want to view the dashboard (5 minutes)

If you just want to see the analysis without executing anything:

### 1. Clone the repository
```bash
git clone https://github.com/alizdom98/erasmus_project.git
cd erasmus_project
```

### 2. Open the dashboard

Simply open `Proyecto_erasmus_bi.pbix` with Power BI Desktop.

The dashboard is already complete with:
- 4 analysis pages
- All transformations applied
- Configured data model
- Interactive visualizations

You can explore, filter, and analyze directly.

**Note**: The dashboard includes Power Query transformations that are not documented in detail. If you wanted to replicate it from scratch, you would have to manually reconstruct the data model and transformations.

---

## Option B: I want to run the complete pipeline

If you want to reproduce the data cleaning process:

### 1. Clone the repository
```bash
git clone https://github.com/alizdom98/erasmus_project.git
cd erasmus_project
```

### 2. Install dependencies
```bash
pip install -r requirements.txt
```

Main libraries are:
- `pandas` to manipulate data
- `numpy` for numerical operations
- `pyarrow` to export to Parquet format

### 3. Download the original data

The data is not in the repository due to its size (~1.7 GB). Download it from:

1. Go to the [EU Open Data Portal](https://data.europa.eu/data/datasets?query=erasmus+KA1+mobility&locale=en)
2. Search for "Erasmus+ KA1 Mobility Data"
3. Download CSV files for 2014-2024
4. Place them in `bases_datos_erasmus/` with these names:
   - `Erasmus-KA1-Mobility-Data-2014.csv`
   - `Erasmus-KA1-Mobility-Data-2015.csv`
   - ... (up to 2024)

### 4. Run the notebook

```bash
jupyter notebook data_erasmus.ipynb
```

Execute all cells with `Cell > Run All`. Execution time depends on your hardware.

At the end it generates two files in `data/processed/`:
- `erasmus_he_2017_2024.parquet` (recommended)
- `erasmus_he_2017_2024.csv`

### 5. Explore the data in Power BI

Once the data is processed, you can load it into Power BI to do your own analysis:

**Recommended option - Parquet:**
1. Open Power BI Desktop
2. `Get Data > More... > Parquet`
3. Select `data/processed/erasmus_he_2017_2024.parquet`
4. `Load`

**Alternative option - CSV:**
1. `Get Data > Text/CSV`
2. Select `data/processed/erasmus_he_2017_2024.csv`
3. Verify data types (Power BI sometimes detects them incorrectly)
4. `Load`

From here you can create your own visualizations. To understand the variables, consult [DATA_DICTIONARY.md](DATA_DICTIONARY.md).

---

## Common Problems

### Error: "File not found"

Verify that CSV files are in the correct folder:
```
bases_datos_erasmus/Erasmus-KA1-Mobility-Data-2014.csv
bases_datos_erasmus/Erasmus-KA1-Mobility-Data-2015.csv
...
```

Names must be exact (capitals, hyphens, etc.).

### Error: "UnicodeDecodeError"

The notebook handles different encodings automatically, but if it fails:
1. Verify that you downloaded the original files without modification
2. Try opening the CSV in a text editor and save it with UTF-8 encoding

### The notebook is very slow

If you have little RAM (<8 GB):
1. Close other applications
2. Execute the notebook in sections instead of all at once
3. Processing may take longer

---

## System Requirements

**Minimum:**
- RAM: 8 GB
- Python 3.8+
- Disk space: 5 GB

**Recommended:**
- RAM: 16 GB
- SSD (faster)

**Approximate times:**
- Data download: Variable depending on connection
- Notebook execution: Variable depending on hardware
- Load in Power BI (Parquet): Seconds

---

## Additional Documentation

- **[README.md](../../README.md)**: Project overview and main findings
- **[PIPELINE.md](PIPELINE.md)**: Step-by-step cleaning process with justifications
- **[DATA_DICTIONARY.md](DATA_DICTIONARY.md)**: Complete description of the 21 variables
- **[POWER_BI_GUIDE.md](POWER_BI_GUIDE.md)**: Dashboard and data model guide
- **[DAX_MEASURES.md](DAX_MEASURES.md)**: DAX measures explained
- **[EXECUTIVE_SUMMARY.md](EXECUTIVE_SUMMARY.md)**: Executive project summary
- **[PROJECT_STRUCTURE.md](PROJECT_STRUCTURE.md)**: Repository structure
- **[NAVIGATION.md](NAVIGATION.md)**: Project navigation guide

---

## Help

If something doesn't work:
1. Check that all files are in the correct folders
2. Verify that you installed dependencies (`pip install -r requirements.txt`)
3. Read the "Common Problems" section above
4. Open an issue on GitHub describing the problem

---

**Author**: Andrés Liz Domínguez
**Last update**: February 2026
