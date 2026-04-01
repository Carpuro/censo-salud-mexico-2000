# censo-salud-mexico-2000 — Proyecto Programación II

## Contexto
Proyecto de la materia Programación II (MCD, UdeG CUCEA). Repo separado del repo de la materia por el tamaño de los datos y notebooks.
Análisis de la base de defunciones/censo INEGI 2000 (10M registros) — exploración, modelado predictivo y explicativo.

## Estructura del repo
- `data/` — datos fuente (gitignored, solo .gitkeep)
- `notebooks/` — análisis y modelos
  - `Actividad_Defunciones_SQL.ipynb` — análisis descriptivo + Decision Tree básico
  - `Modelado_Predictivo_vs_Explicativo.ipynb` — Logit + OLS + Decision Tree + Random Forest
- `outputs/visualizations/` — PNGs generados (gitignored o incluidos según entrega)
- `sql/` — scripts SQL (gitignored por tamaño)
- `.env` — credenciales DB (gitignored)

## Entorno
- `conda activate MCD` antes de correr notebooks
- Credenciales en `.env` (ver README para estructura)
- Spark en homelab: `/opt/spark`; en Katana (Windows): `C:\spark`
- ODBC: el notebook detecta automáticamente Driver 18 o 17

## Base de datos
- SQL Server 2025 en Docker (homelab: `localhost,1433`; compañeros: `192.168.100.11,1433`)
- Base: `Defunciones_2000`, tabla principal: `defunciones`, 10,099,182 registros, 81 columnas VARCHAR
- Usuario compartido: `readonly` / `MCD_2000`
- CRITICAL: todas las columnas son VARCHAR — siempre CAST en SQL o pd.to_numeric en Python
- CRITICAL: NOTIEDER=5 = sin derechohabiencia; NOTIEDER=NULL = con cobertura (no filtrar IS NOT NULL)
- CRITICAL: 999/9999 = no especificado — filtrar en SQL o en Python antes de modelar
- SQL Server limitado a 4GB RAM — usar tablas materializadas, no queries directos sin filtros

## Resultados de los modelos (ejecutados 2026-04-01)

### Datos
- 8,896,028 registros tras limpieza (de 10,099,182 cargados)
- 65.3% sin derechohabiencia / 34.7% con cobertura

### Modelo 1 — Derechohabiencia
- Logit Pseudo R²: 0.1224
- Decision Tree Accuracy: 71.45%
- Variable más importante: Nivel Educativo (77.1%), luego Edad (16.2%), Estado casi 0% con max_depth=5

### Modelo 2 — Edad de Muerte (adultos ≥15 años, n=6,451,966)
- OLS R²: 0.028 — efecto derechohabiencia: +3.46 años de vida (p < 0.0001)
- Random Forest R²: 0.2696, RMSE: 14.61 años
- Variable más importante: Nivel Educativo (81.4%)

## Reglas
- No hacer data leakage — remover variables post-hoc antes de modelar
- Crear variable target (tiene_derechohabiencia) ANTES de aplicar dropna
- Visualizaciones guardadas como PNG en outputs/visualizations/ Y mostradas inline (plt.show())
- Escritura académica: técnica pero conversacional, primera persona, sin frases AI-sounding
- Cross-platform: matplotlib usa Agg solo en Linux sin DISPLAY; en Windows corre inline automáticamente
