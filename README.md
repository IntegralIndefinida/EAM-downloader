# Descargador y Extractor de Microdatos EAM 🏭🇨🇴

[![Python Version](https://img.shields.io/badge/python-3.8%2B-blue.svg)](https://www.python.org/)
[![Licencia: MIT](https://img.shields.io/badge/Licencia-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Fuente de Datos: DANE](https://img.shields.io/badge/Fuente%20de%20Datos-DANE%20ANDA-green.svg)](https://microdatos.dane.gov.co/)

Herramienta en Python diseñada para automatizar el descubrimiento, descarga, extracción y estandarización del **histórico completo de microdatos de la Encuesta Anual Manufacturera (EAM)** del DANE Colombia (**1992 – 2024**, 33 años continuos) directamente desde el Archivo Nacional de Datos (ANDA).

---

## 📌 Descripción General

La **Encuesta Anual Manufacturera (EAM)** es la principal fuente de información estructural del sector industrial colombiano a nivel de planta/establecimiento (producción, empleo, costos laborales, consumo de materias primas, inversión en activos fijos y consumo energético).

Descargar su serie histórica manualmente desde el portal del DANE presenta retos particulares:
- **Estructura multianual de catálogos:** A diferencia de otras encuestas, el DANE agrupó la EAM histórica en catálogos multianuales antes de 2018 (por ejemplo, el catálogo 494 cubre 8 años entre 2000 y 2007).
- **Variación de formatos según la época:** 
  - La mayoría de años (1992–2022) contienen archivos `.csv`.
  - El año **2023** no incluyó `.csv`, publicándose en formato `.txt` delimitado por tabulaciones.
  - El año **2024** no incluyó ni `.csv` ni `.txt` (publicado únicamente en `.dta`, `.sav` y `.xlsx`).

**EAM Downloader** automatiza este proceso garantizando una serie continua y homogénea en texto plano delimitado.

---

## 🚀 Características Principales

- **Cobertura Histórica Integral (1992 – 2024):** Descarga los 33 años oficiales disponibles en ANDA mediante un mapeo de 13 catálogos.
- **Jerarquía Inteligente de Formatos:**
  1. **Prioridad 1 (`.csv`):** Extrae directamente el archivo CSV cuando está presente (1992 a 2022).
  2. **Prioridad 2 (`.txt`):** Extrae el archivo de texto delimitado si no hay CSV (caso 2023).
  3. **Prioridad 3 (`.dta` a `.txt`):** Convierte el microdato Stata a texto plano tabulado (`.txt`) si no existe ni CSV ni TXT (caso 2024), asegurando que la serie histórica nunca quede incompleta.
- **Descargas Incrementales:** Detecta qué años ya fueron procesados localmente y los salta en 0 segundos.
- **Tolerancia a Fallos de Red:** Descargas por fragmentos (*streaming*) y reintentos automáticos con pausas progresivas.
- **Soporte para Módulos Satélite (EAM-TIC):** Opción para descargar adicionalmente los módulos de tecnologías de la información disponibles entre 2008 y 2018.

---

## ⚠️ Notas Metodológicas Importantes

> [!IMPORTANT]
> **1. Cambios en la Clasificación Industrial (CIIU):**
> La serie histórica de la EAM abarca tres nomenclaturas industriales diferentes:
> - **1992 – 1999:** Clasificación **CIIU Rev. 2 A.C.** (códigos de 4 dígitos, ej. `3111`).
> - **2000 – 2013:** Clasificación **CIIU Rev. 3 A.C.** (códigos de 4 dígitos, ej. `1511`).
> - **2014 – 2024:** Clasificación **CIIU Rev. 4 A.C.** (códigos de 4 dígitos, ej. `1011`).
> Si se construyen series de tiempo o paneles de largo plazo que crucen estas décadas, se recomienda emplear tablas de concordancia o correlativas oficiales del DANE.

> [!NOTE]
> **2. Identificadores de Establecimiento y Seguimiento de Panel:**
> - Las variables clave de identificación son `nordemp` (número de orden de la empresa) y `nordest` (número de orden del establecimiento o planta).
> - En los microdatos públicos anonimizados de ANDA, verificar la consistencia temporal del código de anonimización según el período analizado.

---

## 🛠️ Instalación y Requisitos

1. **Clonar o descargar el repositorio:**
   ```bash
   cd /ruta/a/tu/proyecto
   ```

2. **Crear y activar un entorno virtual (recomendado):**
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate   # En Windows: .venv\Scripts\activate
   ```

3. **Instalar dependencias:**
   ```bash
   pip install -r requirements.txt
   ```

---

## 💻 Guía de Uso y Comandos CLI

### 1. Descargar la Serie Histórica Completa (1992–2024)
Descarga y extrae los 33 años en la carpeta `data/datos/`:
```bash
python3 eam_downloader.py
```

### 2. Descargar Años Específicos
```bash
# Descargar un solo año
python3 eam_downloader.py --years 2024

# Descargar varios años puntuales
python3 eam_downloader.py --years 1995 2005 2015 2023 2024
```

### 3. Ahorrar Espacio en Disco (`--delete-zips`)
Elimina automáticamente los archivos `.zip` descargados tras extraer los datos limpios:
```bash
python3 eam_downloader.py --delete-zips
```

### 4. Incluir Módulos Satélite de Tecnologías de Información (EAM-TIC)
Descarga además los paquetes de tecnologías TIC para los años 2008 a 2018:
```bash
python3 eam_downloader.py --include-tic
```

### 5. Personalizar Carpeta de Destino
```bash
python3 eam_downloader.py --base-dir "/ruta/personalizada"
```

---

## 📂 Estructura de Carpetas Generada

```text
data/
├── metadata/                   # JSONs metodológicos oficiales de cada catálogo
│   ├── catalogo_563.json
│   └── ...
├── raw_zips/                   # Paquetes ZIP anuales originales (si no se usa --delete-zips)
│   └── 2024/
│       └── BDATOS-EAM-2024.zip
└── datos/                      # Archivos de microdatos limpios por año
    ├── 1992/
    │   └── EAM_1992.csv
    ├── ...
    ├── 2023/
    │   └── EAM_ANONIMIZADA_2023.txt      # Formato delimitado oficial
    └── 2024/
        └── EAM_ANONIMIZADA_2024.txt      # Convertido automáticamente desde Stata
```

---

## 📋 Parámetros de Línea de Comandos

| Parámetro | Valor por Defecto | Descripción |
| :--- | :--- | :--- |
| `--years` | `None` (Todos) | Lista de años a procesar separados por espacio (ej. `--years 2020 2021 2022`). |
| `--base-dir` | `"data"` | Directorio raíz donde se almacenan las descargas y extracciones. |
| `--include-tic` | `False` | Descarga los módulos TIC de la manufactura (2008–2018). |
| `--delete-zips`| `False` | Borra los archivos `.zip` tras extraer los datos para optimizar espacio. |
| `--no-extract` | `False` | Descarga únicamente los paquetes ZIP sin descomprimirlos. |

---

## ⚖️ Licencia y Citación

- **Licencia del Código:** Distribuido bajo licencia [MIT](LICENSE).
- **Titularidad de los Datos:** Microdatos públicos producidos por el **Departamento Administrativo Nacional de Estadística (DANE)** de Colombia.
- **Cita Sugerida:**
  > *Departamento Administrativo Nacional de Estadística (DANE). Encuesta Anual Manufacturera (EAM) [Microdatos]. Archivo Nacional de Datos (ANDA), Colombia.*
