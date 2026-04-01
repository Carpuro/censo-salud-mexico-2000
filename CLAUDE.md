# censo-salud-mexico-2000 — Proyecto Programación II

## Contexto
Proyecto de la materia Programación II (MCD, UdeG CUCEA). Repo separado del repo de la materia por el tamaño de los datos y notebooks.
Análisis de la base de defunciones/censo INEGI 2000 (10M registros) — exploración, modelado predictivo y explicativo.

## Estructura del repo
- `notebooks/` — análisis y modelos
  - `Actividad_Defunciones_SQL.ipynb` — análisis descriptivo + Decision Tree básico
  - `Modelado_Predictivo_vs_Explicativo.ipynb` — Logit + OLS + Decision Tree + Random Forest
- `figures/` — PNGs generados por el notebook, trackeados en git (pathlib, dinámico)
- `sql/` — scripts SQL (gitignored por tamaño)
- `.env` — credenciales DB (gitignored, cada máquina tiene el suyo)

## Entorno
- `conda activate MCD` antes de correr notebooks
- Credenciales en `.env` (ver README para estructura)
- Spark en homelab: `/opt/spark`; en Katana (Windows): `C:\spark`
- ODBC: el notebook detecta automáticamente Driver 18 o 17

## Base de datos
- SQL Server 2025 en Docker (homelab: `localhost,1433`; compañeros: `192.168.100.11,1433`)
- Base: `Defunciones_2000`, tabla principal: `defunciones`, 10,099,182 registros, 81 columnas VARCHAR
- Usuario compartido (solo lectura): `readonly` / `MCD_2000`
- CRITICAL: todas las columnas son VARCHAR — siempre CAST en SQL o pd.to_numeric en Python
- CRITICAL: NOTIEDER='5' = sin derechohabiencia; NOTIEDER=NULL = con cobertura — NUNCA filtrar IS NOT NULL
- CRITICAL: NIVELACAD usa códigos 2 dígitos INEGI (00,10,20,30,40,51,52,61,62,63,73,80,90) — NO 1-9
- CRITICAL: TAM_LOC va del 1 al 7 (no 1-4) — 5=ciudad grande, 6=megalópolis, 7=no especificado
- CRITICAL: IMSS='1', ISSSTE='2', PEMEX='3' — cada columna usa su propio código, no '1' para todos
- CRITICAL: 999/9999 = no especificado — filtrar antes de modelar
- SQL Server limitado a 4GB RAM — usar tablas materializadas, no queries directos sin filtros
- Pérdida de 1.2M registros en limpieza: NIVELACAD NULL en ~1,169,422 registros (campo no respondido en censo)

## Codificaciones clave (INEGI Censo 2000)
- IMSS: '1'=tiene IMSS, NULL=no tiene, '9'=no especificado
- ISSSTE: '2'=tiene ISSSTE, NULL=no tiene, '9'=no especificado
- PEMEX: '3'=tiene PEMEX/SEDENA/MARINA, NULL=no tiene, '9'=no especificado
- NOTIEDER: '5'=sin derechohabiencia, NULL=con cobertura
- TAM_LOC: 1=rural(<2500), 2=semirural, 3=urbano, 4=metropolitano, 5=ciudad grande, 6=megalópolis, 7=no esp.
- NIVELACAD: 00=sin instrucción, 10=preescolar, 20/30=primaria, 40=secundaria, 51/52=prepa, 61/62/63=técnico, 73=licenciatura, 80=maestría, 90=doctorado

## Tablas y vistas en SQL Server

### Dimensiones (5)
- `dim_estados` — 32 estados (clave_estado, nombre_estado)
- `dim_sexo` — sexo (1=Hombre, 2=Mujer, 9=No especificado)
- `dim_localidad` — tipos de localidad, claves 1-7
- `dim_servicios_salud` — instituciones de salud
- `dim_nivel_educativo` — niveles simplificados 1-9 (NO usar para join con defunciones)
- `dim_nivelacad` — tabla nueva con códigos INEGI reales 2 dígitos + categoría + orden (usar esta)

### Tablas materializadas (corregidas 2026-04-01)
- `tbl_kpis` — KPIs generales: total, IMSS, ISSSTE, PEMEX, sin cobertura, pct, edad promedio
- `tbl_acceso_resumen` — resumen por tipo de servicio
- `tbl_estados_resumen` — cobertura por estado (32 filas) — join correcto con clave_estado
- `tbl_urbano_resumen` — cobertura por tipo de localidad (7 filas, incluye códigos 5 y 6)
- `tbl_escolaridad_resumen` — cobertura por nivel educativo con códigos INEGI reales
- `tbl_indigena_resumen` — cobertura población hablante de lengua indígena
- `tbl_grupos_edad_resumen` — cobertura por grupo de edad (0-14, 15-29, 30-44, 45-59, 60-74, 75+)

### Tablas de modelos (para Power BI, creadas 2026-04-01)
- `tbl_metricas_modelos` — Pseudo R², Accuracy, R², RMSE por modelo y enfoque
- `tbl_importancia_variables` — importancia de features por modelo y algoritmo
- `tbl_coeficientes_ols` — coeficientes OLS con p-valor, significancia e interpretación en texto

### Vistas (13)
vw_acceso_salud, vw_salud_por_estado, vw_perfil_demografico, vw_escolaridad, vw_ingresos,
vw_mortalidad_infantil, vw_salud_educacion, vw_kpis_generales, vw_grupos_edad,
vw_urbano_rural, vw_indigena_salud, vw_discapacidad_salud, vw_estcivil_salud

## Resultados de los modelos (ejecutados 2026-04-01)

### Datos
- 8,896,028 registros tras limpieza (de 10,099,182 cargados)
- 65.3% sin derechohabiencia / 34.7% con cobertura

### Modelo 1 — Derechohabiencia
- Logit Pseudo R²: 0.1224
- Decision Tree Accuracy: 71.45%
- Variable más importante: Nivel Educativo (77.1%), luego Edad (16.2%), Estado ~0% con max_depth=5

### Modelo 2 — Edad de Muerte (adultos ≥15 años, n=6,451,966)
- OLS R²: 0.028 — efecto derechohabiencia: +3.46 años de vida (p < 0.0001)
- Random Forest R²: 0.2696, RMSE: 14.61 años
- Variable más importante: Nivel Educativo (81.4%)

## Reglas
- No hacer data leakage — remover variables post-hoc antes de modelar
- Crear variable target (tiene_derechohabiencia) ANTES de aplicar dropna
- Visualizaciones en `figures/` (pathlib, FIGURES_DIR dinámico según CWD) + inline con plt.show()
- Escritura académica: técnica pero conversacional, primera persona, sin frases AI-sounding
- Cross-platform: matplotlib usa Agg solo en Linux sin DISPLAY; en Windows corre inline automáticamente

## Git
- Sin Co-Authored-By en commits
