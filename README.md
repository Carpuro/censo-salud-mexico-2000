## Análisis de Acceso a Servicios de Salud — Censo INEGI 2000

![Python](https://img.shields.io/badge/Python-3.10-blue)
![SQL Server](https://img.shields.io/badge/SQL%20Server-17.0-red)
![Power BI](https://img.shields.io/badge/Power%20BI-Dashboard-yellow)

> Proyecto desarrollado para la materia **Programación II** de la **Maestría en Ciencia de Datos (MCD)** — Universidad de Guadalajara, CUCEA.

---

##  Descripción

Análisis completo del Censo General de Población y Vivienda 2000 del INEGI con más de **10 millones de registros** para estudiar la desigualdad estructural en el acceso a servicios de salud en México. El proyecto implementa un pipeline completo de datos desde la importación hasta la visualización y modelado estadístico.

---

##  Pregunta de Investigación

**¿Qué factores determinan el acceso a servicios de salud en México en el año 2000?**

- **Modelo Explicativo (OLS):** ¿Cuánto afecta cada variable demográfica a la derechohabiencia?
- **Modelo Predictivo (Random Forest):** ¿Podemos predecir la edad de muerte usando variables demográficas y acceso a salud?

---

##  Hallazgos Principales

| Indicador | Valor |
|-----------|-------|
| Población sin derechohabiencia | **65.6%** |
| Población rural sin cobertura | **83.7%** |
| Población indígena sin cobertura | **86.3%** |
| Edad promedio de la población | **25.88 años** |
| Total de registros analizados | **17 millones** |

---

##  Stack Tecnológico

```
CSV (1.69GB) → SQL Server → Python (Jupyter) → Power BI
```

| Herramienta | Uso |
|-------------|-----|
| **SQL Server 17.0** | Almacenamiento y procesamiento (BULK INSERT, 13 vistas, 5 dimensiones) |
| **Python 3.10** | Análisis estadístico y modelado |
| **Power BI** | Dashboard de storytelling |
| **Grafana** | Monitoreo del pipeline |
| **Orange** | Minería de datos y clustering |

---

##  Librerías Python

```python
pandas          # Manipulación de datos
numpy           # Cálculo numérico
pyodbc          # Conexión a SQL Server
statsmodels     # Modelado explicativo (OLS)
scikit-learn    # Modelado predictivo (Random Forest)
matplotlib      # Visualización
```

---

##  Estructura del Proyecto

```
censo-salud-mexico-2000/
│
├── notebooks/
│   ├── Actividad_Defunciones_SQL.ipynb        # Análisis general con Pandas/NumPy
│   └── Modelado_Predictivo_vs_Explicativo.ipynb  # Modelos OLS vs Random Forest
│
├── sql/
│   ├── 01_crear_base_datos.sql                # Creación de base de datos y tabla
│   ├── 02_bulk_insert.sql                     # Importación del CSV
│   ├── 03_vistas.sql                          # 13 vistas de análisis
│   └── 04_dimensiones.sql                     # 5 tablas dimensión
│
├── data/
│   ├── acceso_salud.csv                       # Vista exportada
│   ├── grupos_edad.csv
│   ├── urbano_rural.csv
│   ├── indigena_salud.csv
│   ├── escolaridad.csv
│   ├── ingresos.csv
│   ├── mortalidad_infantil.csv
│   ├── salud_educacion.csv
│   ├── salud_por_estado.csv
│   ├── kpis_generales.csv
│   ├── perfil_demografico.csv
│   ├── discapacidad_salud.csv
│   └── estcivil_salud.csv
│
└── README.md
```

---

##  Base de Datos SQL Server

### Vistas creadas (13)
| Vista | Descripción |
|-------|-------------|
| `vw_acceso_salud` | Acceso a servicios por estado, sexo y edad |
| `vw_salud_por_estado` | Cobertura agregada por estado |
| `vw_perfil_demografico` | Distribución por edad y sexo |
| `vw_escolaridad` | Nivel educativo por estado |
| `vw_ingresos` | Ingresos por situación laboral |
| `vw_mortalidad_infantil` | Mortalidad menores de 5 años |
| `vw_salud_educacion` | Cobertura por nivel educativo |
| `vw_kpis_generales` | KPIs resumidos |
| `vw_grupos_edad` | Cobertura por grupo de edad |
| `vw_urbano_rural` | Cobertura urbano vs rural |
| `vw_indigena_salud` | Cobertura población indígena |
| `vw_discapacidad_salud` | Cobertura por tipo de discapacidad |
| `vw_estcivil_salud` | Cobertura por estado civil |

### Tablas dimensión (5)
| Tabla | Registros |
|-------|-----------|
| `dim_estados` | 32 |
| `dim_sexo` | 3 |
| `dim_nivel_educativo` | 9 |
| `dim_servicios_salud` | 6 |
| `dim_localidad` | 5 |

---

##  Modelos Estadísticos

### Modelo 1 — Explicativo (statsmodels OLS)
- **Variable dependiente:** `tiene_derechohabiencia` (0/1)
- **Variables independientes:** ENT, SEXO, EDAD, NIVELACAD, TAM_LOC
- **Métricas:** R², coeficientes, p-valores, intervalos de confianza

### Modelo 2 — Predictivo (Random Forest Regressor)
- **Variable dependiente:** `EDAD` (edad de muerte)
- **Variables independientes:** ENT, SEXO, NIVELACAD, TAM_LOC, `tiene_derechohabiencia`
- **Métricas:** R², RMSE
- **Configuración:** 100 árboles, profundidad máxima 10, carga por chunks de 100K

### Comparativa
| | Explicativo (OLS) | Predictivo (Random Forest) |
|---|---|---|
| Variable dependiente | Derechohabiencia | Edad de muerte |
| Métrica principal | R², p-valores | R², RMSE |
| Fortaleza | Interpretable | Preciso |
| Limitación | Solo lineal | Caja negra |
| Responde | ¿Por qué? | ¿Qué tan bien? |

---

##  Cómo ejecutar

### 1. Requisitos
```bash
pip install pandas numpy pyodbc statsmodels scikit-learn matplotlib
```

### 2. Configurar SQL Server
```sql
-- Ejecutar en orden:
01_crear_base_datos.sql
02_bulk_insert.sql
03_vistas.sql
04_dimensiones.sql
```

### 3. Configurar conexión en notebooks
```python
conn = pyodbc.connect(
    'DRIVER={ODBC Driver 17 for SQL Server};'
    'SERVER=localhost;'
    'DATABASE=Defunciones_2000;'
    'UID=tu_usuario;'
    'PWD=tu_password;'
)
```

### 4. Ejecutar notebooks
```bash
jupyter notebook
```
Abrir y ejecutar en orden:
1. `Actividad_Defunciones_SQL.ipynb`
2. `Modelado_Predictivo_vs_Explicativo.ipynb`

---

##  Autor

**Carlos Pulido Rosas**
Maestría en Ciencia de Datos (MCD) — Semestre 2
Universidad de Guadalajara, CUCEA
Programación II

---

##  Fuente de Datos

Instituto Nacional de Estadística y Geografía (INEGI)
**Censo General de Población y Vivienda 2000**
[https://www.inegi.org.mx](https://www.inegi.org.mx)
