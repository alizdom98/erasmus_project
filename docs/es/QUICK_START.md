# Guía Rápida para Empezar

[Read in English](../en/QUICK_START.md) | [Leer en Español](QUICK_START.md)

Hay dos formas de usar este proyecto: simplemente ver el dashboard terminado, o reproducir todo el proceso desde cero.

---

## Opción A: Solo quiero ver el dashboard (5 minutos)

Si solo quieres ver el análisis sin ejecutar nada:

### 1. Clona el repositorio
```bash
git clone https://github.com/alizdom98/erasmus_project.git
cd erasmus_project
```

### 2. Abre el dashboard

Simplemente abre `Proyecto_erasmus_bi.pbix` con Power BI Desktop.

El dashboard ya está completo con:
- 4 páginas de análisis
- Todas las transformaciones aplicadas
- Modelo de datos configurado
- Visualizaciones interactivas

Puedes explorar, filtrar y analizar directamente.

**Nota**: El dashboard incluye transformaciones de Power Query que no están documentadas en detalle. Si quisieras replicarlo desde cero, tendrías que reconstruir manualmente el modelo de datos y las transformaciones.

---

## Opción B: Quiero ejecutar el pipeline completo

Si quieres reproducir el proceso de limpieza de datos:

### 1. Clona el repositorio
```bash
git clone https://github.com/alizdom98/erasmus_project.git
cd erasmus_project
```

### 2. Instala las dependencias
```bash
pip install -r requirements.txt
```

Las principales librerías son:
- `pandas` para manipular los datos
- `numpy` para operaciones numéricas
- `pyarrow` para exportar a formato Parquet

### 3. Descarga los datos originales

Los datos no están en el repositorio por su tamaño (~1.7 GB). Descárgalos desde:

1. Ve al [Portal de Datos Abiertos de la UE](https://data.europa.eu/data/datasets?query=erasmus+KA1+mobility&locale=en)
2. Busca "Erasmus+ KA1 Mobility Data"
3. Descarga los archivos CSV para 2014-2024
4. Colócalos en `bases_datos_erasmus/` con estos nombres:
   - `Erasmus-KA1-Mobility-Data-2014.csv`
   - `Erasmus-KA1-Mobility-Data-2015.csv`
   - ... (hasta 2024)

### 4. Ejecuta el notebook

```bash
jupyter notebook data_erasmus.ipynb
```

También puedes usar `data_erasmus_ES.ipynb` si prefieres la versión en español.

Ejecuta todas las celdas con `Cell > Run All`. El tiempo de ejecución depende de tu hardware.

Al final genera dos archivos en `data/processed/`:
- `erasmus_he_2017_2024.parquet` (recomendado)
- `erasmus_he_2017_2024.csv`

### 5. Explora los datos en Power BI

Una vez procesados los datos, puedes cargarlos en Power BI para hacer tu propio análisis:

**Opción recomendada - Parquet:**
1. Abre Power BI Desktop
2. `Get Data > More... > Parquet`
3. Selecciona `data/processed/erasmus_he_2017_2024.parquet`
4. `Load`

**Opción alternativa - CSV:**
1. `Get Data > Text/CSV`
2. Selecciona `data/processed/erasmus_he_2017_2024.csv`
3. Verifica tipos de datos (Power BI a veces los detecta mal)
4. `Load`

Desde aquí puedes crear tus propias visualizaciones. Para entender las variables, consulta `docs/es/DATA_DICTIONARY.md`.

---

## Problemas comunes

### Error: "File not found"

Verifica que los archivos CSV estén en la carpeta correcta:
```
bases_datos_erasmus/Erasmus-KA1-Mobility-Data-2014.csv
bases_datos_erasmus/Erasmus-KA1-Mobility-Data-2015.csv
...
```

Los nombres deben ser exactos (mayúsculas, guiones, etc.).

### Error: "UnicodeDecodeError"

El notebook maneja diferentes encodings automáticamente, pero si falla:
1. Verifica que descargaste los archivos originales sin modificar
2. Intenta abrir el CSV en un editor de texto y guárdalo con encoding UTF-8

### El notebook es muy lento

Si tienes poco RAM (<8 GB):
1. Cierra otras aplicaciones
2. Ejecuta el notebook por secciones en lugar de todo de golpe
3. El procesamiento puede tardar más tiempo

---

## Requisitos de sistema

**Mínimo:**
- RAM: 8 GB
- Python 3.8+
- Espacio en disco: 5 GB

**Recomendado:**
- RAM: 16 GB
- SSD (más rápido)

**Tiempos aproximados:**
- Descarga de datos: Variable según conexión
- Ejecución del notebook: Variable según hardware
- Carga en Power BI (Parquet): Segundos

---

## Documentación adicional

- **[README.md](../../README.md)**: Visión general del proyecto y hallazgos principales
- **[PIPELINE.md](PIPELINE.md)**: Proceso de limpieza paso a paso con justificaciones
- **[DATA_DICTIONARY.md](DATA_DICTIONARY.md)**: Descripción completa de las 26 variables
- **[POWER_BI_GUIDE.md](POWER_BI_GUIDE.md)**: Guía del dashboard y modelo de datos
- **[DAX_MEASURES.md](DAX_MEASURES.md)**: Medidas DAX explicadas
- **[EXECUTIVE_SUMMARY.md](EXECUTIVE_SUMMARY.md)**: Resumen ejecutivo del proyecto
- **[PROJECT_STRUCTURE.md](PROJECT_STRUCTURE.md)**: Estructura del repositorio

---

## Ayuda

Si algo no funciona:
1. Revisa que todos los archivos estén en las carpetas correctas
2. Verifica que instalaste las dependencias (`pip install -r requirements.txt`)
3. Lee la sección "Problemas comunes" más arriba
4. Abre un issue en GitHub describiendo el problema

---

**Autor**: Andrés Liz Domínguez
**Última actualización**: Febrero 2026
