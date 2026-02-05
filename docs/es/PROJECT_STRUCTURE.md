# Estructura del Proyecto

[Read in English](../en/PROJECT_STRUCTURE.md) | [Leer en Español](PROJECT_STRUCTURE.md)

Documentación de la organización del repositorio Erasmus+ Mobility Analysis.

---

## Árbol de Archivos

```
erasmus_project/
│
├── README.md                       # Visión general del proyecto (EMPEZAR AQUÍ)
├── data_erasmus.ipynb              # Notebook principal en inglés
├── data_erasmus_ES.ipynb           # Notebook en español
├── Proyecto_erasmus_bi.pbix        # Dashboard de Power BI
├── LICENSE                         # Información sobre los datos
├── .gitignore                      # Archivos ignorados por Git
├── requirements.txt                # Dependencias de Python
│
├── bases_datos_erasmus/            # Archivos CSV originales (2014-2024)
│   └── Erasmus-KA1-Mobility-Data-{year}.csv  (NO subidos a GitHub)
│
└── docs/
    ├── en/                         # Documentación en inglés
    │   ├── PIPELINE.md             # Proceso de limpieza detallado
    │   ├── QUICK_START.md          # Guía de inicio rápido
    │   ├── EXECUTIVE_SUMMARY.md    # Resumen ejecutivo del proyecto
    │   ├── DATA_DICTIONARY.md      # Descripción de todas las variables
    │   ├── DAX_MEASURES.md         # Medidas DAX de Power BI
    │   ├── POWER_BI_GUIDE.md       # Guía del dashboard
    │   ├── NAVIGATION.md           # Navegación del proyecto
    │   └── PROJECT_STRUCTURE.md    # Estructura del proyecto
    └── es/                         # Documentación en español
        └── (mismos archivos en español)
```

**Nota**: El notebook puede generar datos procesados en una carpeta `data/processed/` al ejecutarse, pero no es obligatorio tenerla de antemano.

---

## Guía de Documentación

### Para Nuevos Usuarios

**1. Empezar aquí**:
- **[README.md](../../README.md)**: Visión general, objetivos, hallazgos clave

**2. Instalación rápida**:
- **[QUICK_START.md](QUICK_START.md)**: Cómo ver el dashboard o reproducir el pipeline completo

**3. Entender las variables**:
- **[DATA_DICTIONARY.md](DATA_DICTIONARY.md)**: Descripción completa de las 26 columnas

### Para Usuarios Avanzados

**4. Proceso de limpieza**:
- **[PIPELINE.md](PIPELINE.md)**: Documentación técnica detallada con justificaciones de cada decisión

**5. Modificar el pipeline**:
- **[data_erasmus_ES.ipynb](../../data_erasmus_ES.ipynb)**: Notebook ejecutable con código comentado

---

## Flujo de Trabajo

El proyecto sigue este flujo:

1. Descarga de datos raw desde portal UE → carpeta `bases_datos_erasmus/`
2. Ejecución del notebook `data_erasmus.ipynb`
3. Generación de archivos procesados (Parquet y CSV)
4. Carga en Power BI (Parquet o CSV)
5. Creación de visualizaciones y análisis

---

## Archivos Clave

### Código Ejecutable

| Archivo | Descripción | Tiempo Ejecución |
|---------|-------------|------------------|
| `data_erasmus.ipynb` | Pipeline completo de limpieza | Variable según hardware |

### Datos

| Tipo | Ubicación | Tamaño | En GitHub |
|------|-----------|--------|-----------|
| **Raw** | `bases_datos_erasmus/*.csv` | ~1.7 GB | ❌ No (demasiado grande) |
| **Procesado** | Generado por notebook | Variable | ❌ No |

**Nota**: Los datos NO se suben a GitHub por su tamaño. El repositorio incluye solo código y documentación.

### Documentación

| Archivo | Propósito | Audiencia |
|---------|-----------|-----------|
| **[README.md](../../README.md)** | Visión general del proyecto | Todos |
| **[PIPELINE.md](PIPELINE.md)** | Proceso de limpieza detallado | Técnico |
| **[QUICK_START.md](QUICK_START.md)** | Guía de inicio rápido | Nuevos usuarios |
| **[DATA_DICTIONARY.md](DATA_DICTIONARY.md)** | Descripción de variables | Analistas de datos |
| **[POWER_BI_GUIDE.md](POWER_BI_GUIDE.md)** | Guía del dashboard | Analistas de datos |
| **[DAX_MEASURES.md](DAX_MEASURES.md)** | Referencia de medidas DAX | Usuarios de Power BI |
| **[EXECUTIVE_SUMMARY.md](EXECUTIVE_SUMMARY.md)** | Resumen ejecutivo del proyecto | Todos |
| **[NAVIGATION.md](NAVIGATION.md)** | Navegación del proyecto | Todos |
| **[PROJECT_STRUCTURE.md](PROJECT_STRUCTURE.md)** | Organización del repo (este archivo) | Contribuidores |

---

## Estadísticas del Proyecto

### Código

- **Lenguaje principal**: Python 3.8+
- **Librerías core**: Pandas, NumPy
- **Celdas del notebook**: 33 (código + markdown)

### Datos

- **Dataset original**: 5,955,075 registros
- **Dataset procesado**: 2,082,071 registros (HE 2017-2024)
- **Reducción**: 65% (filtrado por tipo y período)
- **Duplicados eliminados**: 9,165 (0.15%)

### Transformaciones

- **Columnas originales**: 19-21 (varía por año)
- **Columnas finales**: 26 (estandarizadas)
- **Nuevas variables creadas**: 8 (isced_level, activity_group, etc.)
- **Países normalizados**: 255 → 33 códigos ISO2 únicos

---

## Casos de Uso del Proyecto

### 1. Portfolio de Data Science
- Demostrar habilidades de limpieza de datos a gran escala
- Muestra de documentación profesional
- Ejemplo de pipeline reproducible

### 2. Análisis Educativo
- Tendencias de movilidad estudiantil europea
- Impacto de COVID-19 en educación internacional
- Análisis de inclusividad (fewer opportunities)

### 3. Visualización de Datos
- Dashboard interactivo en Power BI
- Mapas de flujos geográficos
- Series temporales de tendencias

### 4. Aprendizaje
- Ejemplo de proyecto de principio a fin de Data Science
- Buenas prácticas de limpieza y normalización
- Estructura de repositorio profesional

---

## Mantenimiento

### Actualizar con Nuevos Datos (2025+)

Cuando la Comisión Europea publique datos de 2025 o posteriores:

1. **Descargar** nuevo CSV en `bases_datos_erasmus/`
2. **Modificar** rango en notebook:
   ```python
   years = range(2014, 2026)  # Cambiar de 2025 a 2026
   ```
3. **Ejecutar** notebook completo
4. **Verificar** estructura de columnas (puede haber cambios)
5. **Actualizar** RENAME_MAP si hay nuevos nombres de columnas

### Añadir Nuevas Transformaciones

1. Añadir celdas en la sección apropiada del notebook
2. Documentar en comentarios por qué es necesaria la transformación
3. Actualizar `PIPELINE.md` con la nueva fase
4. Actualizar `DATA_DICTIONARY.md` si se crean nuevas variables

---

## Contacto

**Autor**: Andrés Liz Domínguez
- Email: a.liz.dom@gmail.com
- GitHub: Si tienes preguntas sobre el proyecto o encuentras algún error, abre un issue.

---

**Última actualización**: Febrero 2026
**Versión del proyecto**: 1.0
