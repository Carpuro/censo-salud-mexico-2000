## Análisis de Acceso a Servicios de Salud — Censo INEGI 2000

![Python](https://img.shields.io/badge/Python-3.13-blue)
![SQL Server](https://img.shields.io/badge/SQL%20Server-2025-red)
![PySpark](https://img.shields.io/badge/PySpark-3.5.6-orange)

> Proyecto desarrollado para la materia **Programación II** de la **Maestría en Ciencia de Datos (MCD)** — Universidad de Guadalajara, CUCEA.

---

## Descripción

Análisis completo del Censo General de Población y Vivienda 2000 del INEGI con más de **10 millones de registros** para estudiar la desigualdad estructural en el acceso a servicios de salud en México. El proyecto implementa un pipeline completo de datos desde la importación hasta la visualización y modelado estadístico.

---

## Pregunta de Investigación

**¿Qué factores determinan el acceso a servicios de salud en México en el año 2000?**

- **Modelo Explicativo (Logit / OLS):** ¿Cuánto afecta cada variable demográfica a la derechohabiencia?
- **Modelo Predictivo (Decision Tree / Random Forest):** ¿Podemos predecir el acceso y la edad de muerte usando variables demográficas?

---

## Hallazgos Principales

| Indicador | Valor |
|-----------|-------|
| Población sin derechohabiencia | **65.3%** |
| Población rural sin cobertura | **83.7%** |
| Población indígena sin cobertura | **86.3%** |
| Total de registros analizados | **8,896,028** (de 10,099,182 tras limpieza) |
| Variable predictora dominante | **Nivel educativo** (77–81% de importancia) |
| Efecto derechohabiencia en edad de muerte | **+3.46 años de vida** (OLS, p < 0.0001) |

---

## Stack Tecnológico

```
CSV (1.69GB) → SQL Server 2025 → PySpark + Python (Jupyter) → Power BI
```

| Herramienta | Uso |
|-------------|-----|
| **SQL Server 2025** | Almacenamiento y procesamiento (BULK INSERT, 13 vistas, 5 dimensiones, tablas materializadas) |
| **Apache Spark 3.5.6** | Procesamiento distribuido de los 10M registros sin cargar en RAM |
| **Python 3.13** | Análisis estadístico y modelado |
| **Power BI** | Dashboard de storytelling |
| **Orange3** | Minería de datos y clustering |

---

## Librerías Python

```python
pandas          # Manipulación de datos
numpy           # Cálculo numérico
pyodbc          # Conexión a SQL Server
pyspark         # Procesamiento distribuido (tabla completa)
findspark       # Localizar instalación de Spark
statsmodels     # Modelado explicativo (Logit, OLS)
scikit-learn    # Modelado predictivo (Decision Tree, Random Forest)
matplotlib      # Visualización
rpy2            # Pruebas estadísticas en R (chi-cuadrado)
python-dotenv   # Credenciales desde .env
```

---

## Estructura del Proyecto

```
censo-salud-mexico-2000/
│
├── notebooks/
│   ├── Actividad_Defunciones_SQL.ipynb           # Pipeline completo: Spark + NumPy + rpy2 + Scikit-learn
│   └── Modelado_Predictivo_vs_Explicativo.ipynb  # Comparativa OLS vs Random Forest
│
├── figures/                                       # Visualizaciones PNG (trackeadas en git)
├── sql/                                           # Scripts SQL (gitignored por tamaño)
└── README.md
```

---

## Base de Datos SQL Server

### Estrategia de acceso a datos

Dado el volumen de 10M registros se implementaron dos estrategias:

1. **Tablas materializadas con Pandas** — para análisis descriptivo (carga en < 1 segundo)
2. **Apache Spark vía JDBC** — para modelado predictivo sobre la tabla completa

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

### Tablas materializadas (7)

| Tabla | Registros | Descripción |
|-------|-----------|-------------|
| `tbl_kpis` | 1 | KPIs generales: totales, porcentajes, edad promedio |
| `tbl_acceso_resumen` | 3 | Resumen por tipo de servicio |
| `tbl_estados_resumen` | 32 | Cobertura por estado con IMSS, ISSSTE, PEMEX |
| `tbl_urbano_resumen` | 7 | Cobertura por tipo de localidad (rural → megalópolis) |
| `tbl_escolaridad_resumen` | 11 | Cobertura por nivel educativo (códigos INEGI reales) |
| `tbl_indigena_resumen` | 6 | Cobertura por hablante de lengua indígena |
| `tbl_grupos_edad_resumen` | 6 | Cobertura por grupo de edad (0-14, 15-29, ..., 75+) |

### Tablas de modelos — Power BI (3)

| Tabla | Descripción |
|-------|-------------|
| `tbl_metricas_modelos` | Pseudo R², Accuracy, R², RMSE por modelo y enfoque |
| `tbl_importancia_variables` | Importancia de features filtrable por modelo y algoritmo |
| `tbl_coeficientes_ols` | Coeficientes OLS con p-valor, significancia e interpretación |

### Tablas dimensión (6)

| Tabla | Registros | Notas |
|-------|-----------|-------|
| `dim_estados` | 32 | clave_estado, nombre_estado |
| `dim_sexo` | 3 | 1=Hombre, 2=Mujer, 9=No esp. |
| `dim_nivel_educativo` | 9 | Niveles simplificados — no usar para joins con defunciones |
| `dim_nivelacad` | 13 | Códigos INEGI reales 2 dígitos + categoría — usar esta |
| `dim_servicios_salud` | 6 | Instituciones de salud |
| `dim_localidad` | 7 | Claves 1-7 incluyendo ciudades grandes y megalópolis |

---

## Modelos Estadísticos

### Notebook 1 — Actividad_Defunciones_SQL

| Paso | Herramienta | Descripción |
|------|-------------|-------------|
| Carga | PySpark JDBC | 10M registros desde SQL Server |
| Análisis | NumPy | Estadísticas de cobertura y brecha urbano-rural |
| Prueba estadística | rpy2 (R) | Chi-cuadrado de bondad de ajuste |
| Modelo | Decision Tree | Clasificación de derechohabiencia |

### Notebook 2 — Modelado Predictivo vs Explicativo

#### Modelo 1 — Derechohabiencia

| Enfoque | Modelo | Métrica | Resultado |
|---------|--------|---------|-----------|
| Explicativo | Logit (statsmodels) | Pseudo R² | **0.1224** |
| Predictivo | Decision Tree (max_depth=5) | Accuracy | **71.45%** |

**Variable más importante:** Nivel Educativo (77.1%), seguido de Edad (16.2%)

#### Modelo 2 — Edad de Muerte

| Enfoque | Modelo | Métrica | Resultado |
|---------|--------|---------|-----------|
| Explicativo | OLS (statsmodels) | R² | **0.028** |
| Predictivo | Random Forest (100 árboles, profundidad 10) | R² | **0.2696** |
| Predictivo | Random Forest | RMSE | **14.61 años** |

**Variable más importante:** Nivel Educativo (81.4%)

**Efecto de la derechohabiencia en la edad de muerte:** +3.46 años (p < 0.0001)

#### Nota sobre el R² del OLS

El R² bajo (0.028) en el modelo OLS para edad de muerte es esperado en datos de defunciones: la edad de muerte tiene alta varianza individual y las variables demográficas disponibles no capturan factores genéticos, de estilo de vida ni causa específica de muerte. El Random Forest extrae relaciones no lineales que el OLS no puede capturar (R² = 0.27).

### Comparativa general

| | Explicativo | Predictivo |
|---|---|---|
| Fortaleza | Interpretabilidad | Precisión |
| Limitación | Supuestos paramétricos | Caja negra |
| Pregunta | ¿Por qué? | ¿Qué tan bien? |

---

## Cómo ejecutar

### 1. Activar entorno

```bash
conda activate MCD
```

### 2. Configurar credenciales

Crear archivo `.env` en la raíz del proyecto:

```
DB_SERVER=192.168.100.11   # o localhost si corres en homelab
DB_PORT=1433
DB_NAME=Defunciones_2000
DB_USER=tu_usuario
DB_PASSWORD=tu_password
```

### 3. Configurar Spark (homelab Linux)

```python
import findspark
findspark.init('/opt/spark')
```

> En Windows (Katana): `findspark.init('C:\\spark')`

### 4. Ejecutar notebooks

```bash
jupyter notebook
```

Abrir y ejecutar en orden:
1. `Actividad_Defunciones_SQL.ipynb`
2. `Modelado_Predictivo_vs_Explicativo.ipynb`

> El notebook detecta automáticamente el entorno: usa backend `Agg` en Linux headless y muestra gráficas inline en Jupyter con pantalla. Soporta ODBC Driver 17 y 18.

---

## Autor

**Carlos Pulido Rosas**
Maestría en Ciencia de Datos (MCD) — Semestre 2
Universidad de Guadalajara, CUCEA
Programación II

---

## Fuente de Datos

Instituto Nacional de Estadística y Geografía (INEGI)
**Censo General de Población y Vivienda 2000**
[https://www.inegi.org.mx](https://www.inegi.org.mx)
