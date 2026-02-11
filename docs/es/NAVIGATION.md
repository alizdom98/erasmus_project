# Guía de Navegación del Proyecto

[Read in English](../en/NAVIGATION.md) | [Leer en Español](NAVIGATION.md)

Este archivo te ayuda a encontrar rápidamente lo que necesitas en la documentación.

---

## Por dónde empezar

Si es la primera vez que ves el proyecto:

1. **[README.md](../../README.md)** (en la raíz) - Empieza aquí para entender qué es el proyecto y los hallazgos principales
2. **[EXECUTIVE_SUMMARY.md](EXECUTIVE_SUMMARY.md)** - Resumen de una página si quieres algo más corto
3. **[QUICK_START.md](QUICK_START.md)** - Si quieres reproducir el proyecto tú mismo

---

## Documentación por audiencia

**Si eres reclutador o estás evaluando el proyecto:**
- [EXECUTIVE_SUMMARY.md](EXECUTIVE_SUMMARY.md) (5 minutos)
- [README.md](../../README.md) - solo la sección de hallazgos (5 minutos)

**Si te interesa la parte técnica de limpieza de datos:**
- [PIPELINE.md](PIPELINE.md) - Proceso completo de limpieza paso a paso
- [data_erasmus_ES.ipynb](../../data_erasmus_ES.ipynb) - El código ejecutable con comentarios

**Si quieres entender el dashboard de Power BI:**
- [POWER_BI_GUIDE.md](POWER_BI_GUIDE.md) - Cómo funciona el dashboard y el modelo de datos
- [DAX_MEASURES.md](DAX_MEASURES.md) - Todas las medidas DAX explicadas

**Si quieres saber qué significa cada variable:**
- [DATA_DICTIONARY.md](DATA_DICTIONARY.md) - Descripción de las 21 columnas finales

---

## Archivos del proyecto

### En la raíz

| Archivo | Qué contiene |
|---------|--------------|
| README.md | Visión general del proyecto y hallazgos principales |
| data_erasmus.ipynb | Notebook principal (inglés) |
| data_erasmus_ES.ipynb | Notebook en español |
| Proyecto_erasmus_bi.pbix | Dashboard de Power BI |
| LICENSE | Información sobre los datos de la UE |
| requirements.txt | Librerías de Python necesarias |

### En docs/en/ (documentación en inglés)

| Archivo | Qué contiene |
|---------|--------------|
| PIPELINE.md | Proceso de limpieza paso a paso |
| QUICK_START.md | Cómo reproducir el proyecto |
| EXECUTIVE_SUMMARY.md | Resumen de una página |
| DATA_DICTIONARY.md | Descripción de variables |
| DAX_MEASURES.md | Referencia de medidas DAX |
| POWER_BI_GUIDE.md | Guía del dashboard y modelo de datos |
| PROJECT_STRUCTURE.md | Arquitectura del proyecto |
| NAVIGATION.md | Este archivo |

### En docs/es/ (documentación en español)

Los mismos archivos en español, más un [README.md](README.md) (traducción al español del README raíz).

### En bases_datos_erasmus/

```
bases_datos_erasmus/
└── *.csv                   # Datos originales CSV 2014-2024 (NO en Git)
```

Los archivos de datos no están en el repositorio por su tamaño (~1.7 GB). Ver [QUICK_START.md](QUICK_START.md) para instrucciones de descarga.

---

## Buscar información específica

**¿Cómo se limpió una variable?**
→ [PIPELINE.md](PIPELINE.md) - Busca la fase correspondiente

**¿Qué significa una columna?**
→ [DATA_DICTIONARY.md](DATA_DICTIONARY.md)

**¿Cómo funciona una medida DAX?**
→ [DAX_MEASURES.md](DAX_MEASURES.md)

**¿Qué muestra un visual del dashboard?**
→ [POWER_BI_GUIDE.md](POWER_BI_GUIDE.md)

**¿Por qué tomaste esa decisión?**
→ [PIPELINE.md](PIPELINE.md) o [POWER_BI_GUIDE.md](POWER_BI_GUIDE.md) (según si es limpieza o modelado)

**¿Cómo instalar y ejecutar?**
→ [QUICK_START.md](QUICK_START.md)

---

## Orden recomendado si quieres leerlo todo

1. [README.md](../../README.md) (10 min) - Contexto general
2. [EXECUTIVE_SUMMARY.md](EXECUTIVE_SUMMARY.md) (5 min) - Resumen del proyecto
3. [QUICK_START.md](QUICK_START.md) (10 min) - Setup
4. [PIPELINE.md](PIPELINE.md) (1-2 horas) - Proceso de limpieza
5. [data_erasmus_ES.ipynb](../../data_erasmus_ES.ipynb) (1-2 horas) - Código ejecutable
6. [DATA_DICTIONARY.md](DATA_DICTIONARY.md) (30 min) - Variables
7. [POWER_BI_GUIDE.md](POWER_BI_GUIDE.md) (1 hora) - Dashboard
8. [DAX_MEASURES.md](DAX_MEASURES.md) (30 min) - Medidas DAX
9. [PROJECT_STRUCTURE.md](PROJECT_STRUCTURE.md) (15 min) - Organización del repositorio

Total: 4-6 horas para entender todo el proyecto en profundidad.

**Versión rápida (30 min):**
1. [EXECUTIVE_SUMMARY.md](EXECUTIVE_SUMMARY.md) (5 min)
2. [README.md](../../README.md) - solo hallazgos (10 min)
3. [POWER_BI_GUIDE.md](POWER_BI_GUIDE.md) - solo estructura de páginas (15 min)

---

## Ayuda

Si algo no está claro o falta información:
- Abre un issue en GitHub
- La mayoría de decisiones técnicas están documentadas en [PIPELINE.md](PIPELINE.md) o [POWER_BI_GUIDE.md](POWER_BI_GUIDE.md)

---

**Autor**: Andrés Liz Domínguez
**Última actualización**: Febrero 2026
