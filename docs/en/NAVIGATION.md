# Project Navigation Guide

[Read in English](NAVIGATION.md) | [Leer en Español](../es/NAVIGATION.md)

This file helps you quickly find what you need in the documentation.

---

## Where to Start

If this is your first time viewing the project:

1. **[README.md](../../README.md)** (in the root) - Start here to understand what the project is and the main findings
2. **[EXECUTIVE_SUMMARY.md](EXECUTIVE_SUMMARY.md)** - One-page summary if you want something shorter
3. **[QUICK_START.md](QUICK_START.md)** - If you want to reproduce the project yourself

---

## Documentation by Audience

**If you're a recruiter or evaluating the project:**
- [EXECUTIVE_SUMMARY.md](EXECUTIVE_SUMMARY.md) (5 minutes)
- [README.md](../../README.md) - just the findings section (5 minutes)

**If you're interested in the technical data cleaning part:**
- [PIPELINE.md](PIPELINE.md) - Complete step-by-step cleaning process
- [data_erasmus.ipynb](../../data_erasmus.ipynb) - Executable code with comments

**If you want to understand the Power BI dashboard:**
- [POWER_BI_GUIDE.md](POWER_BI_GUIDE.md) - How the dashboard and data model work
- [DAX_MEASURES.md](DAX_MEASURES.md) - All DAX measures explained

**If you want to know what each variable means:**
- [DATA_DICTIONARY.md](DATA_DICTIONARY.md) - Description of the 26 final columns

---

## Project Files

### In the Root

| File | Contents |
|------|----------|
| README.md | Project overview and main findings |
| data_erasmus.ipynb | Main notebook (English) |
| data_erasmus_ES.ipynb | Notebook in Spanish |
| Proyecto_erasmus_bi.pbix | Power BI Dashboard |
| LICENSE | Information about EU data |
| requirements.txt | Required Python libraries |

### In docs/en/ (English documentation)

| File | Contents |
|------|----------|
| PIPELINE.md | Step-by-step cleaning process |
| QUICK_START.md | How to reproduce the project |
| EXECUTIVE_SUMMARY.md | One-page summary |
| DATA_DICTIONARY.md | Variable descriptions |
| DAX_MEASURES.md | DAX measures reference |
| POWER_BI_GUIDE.md | Dashboard and data model guide |
| PROJECT_STRUCTURE.md | Project architecture |
| NAVIGATION.md | This file |

### In docs/es/ (Spanish documentation)

Same files in Spanish, plus a [README.md](../es/README.md) (Spanish translation of the root README).

### In bases_datos_erasmus/

```
bases_datos_erasmus/
└── *.csv                   # Original CSV data 2014-2024 (NOT in Git)
```

Data files are not in the repository due to their size (~1.7 GB). See [QUICK_START.md](QUICK_START.md) for download instructions.

---

## Finding Specific Information

**How was a variable cleaned?**
→ [PIPELINE.md](PIPELINE.md) - Search for the corresponding phase

**What does a column mean?**
→ [DATA_DICTIONARY.md](DATA_DICTIONARY.md)

**How does a DAX measure work?**
→ [DAX_MEASURES.md](DAX_MEASURES.md)

**What does a dashboard visual show?**
→ [POWER_BI_GUIDE.md](POWER_BI_GUIDE.md)

**Why did you make that decision?**
→ [PIPELINE.md](PIPELINE.md) or [POWER_BI_GUIDE.md](POWER_BI_GUIDE.md) (depending on whether it's cleaning or modeling)

**How to install and run?**
→ [QUICK_START.md](QUICK_START.md)

---

## Recommended Order if You Want to Read Everything

1. [README.md](../../README.md) (10 min) - General context
2. [EXECUTIVE_SUMMARY.md](EXECUTIVE_SUMMARY.md) (5 min) - Project summary
3. [QUICK_START.md](QUICK_START.md) (10 min) - Setup
4. [PIPELINE.md](PIPELINE.md) (1-2 hours) - Cleaning process
5. [data_erasmus.ipynb](../../data_erasmus.ipynb) (1-2 hours) - Executable code
6. [DATA_DICTIONARY.md](DATA_DICTIONARY.md) (30 min) - Variables
7. [POWER_BI_GUIDE.md](POWER_BI_GUIDE.md) (1 hour) - Dashboard
8. [DAX_MEASURES.md](DAX_MEASURES.md) (30 min) - DAX measures
9. [PROJECT_STRUCTURE.md](PROJECT_STRUCTURE.md) (15 min) - Repository organization

Total: 4-6 hours to understand the entire project in depth.

**Quick version (30 min):**
1. [EXECUTIVE_SUMMARY.md](EXECUTIVE_SUMMARY.md) (5 min)
2. [README.md](../../README.md) - just findings (10 min)
3. [POWER_BI_GUIDE.md](POWER_BI_GUIDE.md) - just page structure (15 min)

---

## Help

If something is unclear or information is missing:
- Open an issue on GitHub
- Most technical decisions are documented in [PIPELINE.md](PIPELINE.md) or [POWER_BI_GUIDE.md](POWER_BI_GUIDE.md)

---

**Author**: Andrés Liz Domínguez
**Last updated**: February 2026
