# Esquema Power BI — Acceso a Servicios de Salud México 2000

## Conexión
- Servidor: `192.168.100.11,1433` (red local) o `100.64.0.11,1433` (Tailscale)
- Base de datos: `Defunciones_2000`
- Usuario: ver credenciales compartidas por el equipo
- Autenticación: SQL Server

## Tablas a importar
Importar solo las tablas materializadas — NO conectar directo a `defunciones` (10M registros).

| Tabla | Modo |
|-------|------|
| `tbl_kpis` | Import |
| `tbl_acceso_resumen` | Import |
| `tbl_estados_resumen` | Import |
| `tbl_urbano_resumen` | Import |
| `tbl_escolaridad_resumen` | Import |
| `tbl_indigena_resumen` | Import |
| `tbl_grupos_edad_resumen` | Import |
| `tbl_metricas_modelos` | Import |
| `tbl_importancia_variables` | Import |
| `tbl_coeficientes_ols` | Import |
| `dim_estados` | Import |
| `dim_localidad` | Import |

---

## Medidas DAX

```dax
-- KPIs generales
Total Poblacion = SUM(tbl_kpis[total_poblacion])

Pct Sin Cobertura = 
    DIVIDE(SUM(tbl_estados_resumen[sin_derechohabiencia]),
           SUM(tbl_estados_resumen[total_poblacion])) * 100

Pct Con IMSS = 
    DIVIDE(SUM(tbl_estados_resumen[con_imss]),
           SUM(tbl_estados_resumen[total_poblacion])) * 100

Pct Con ISSSTE = 
    DIVIDE(SUM(tbl_estados_resumen[con_issste]),
           SUM(tbl_estados_resumen[total_poblacion])) * 100

-- Brecha urbano-rural
Brecha Urbano Rural = 
    VAR pct_rural = CALCULATE(
        DIVIDE(SUM(tbl_urbano_resumen[sin_derechohabiencia]),
               SUM(tbl_urbano_resumen[total])) * 100,
        tbl_urbano_resumen[tipo_localidad_codigo] = 1
    )
    VAR pct_metro = CALCULATE(
        DIVIDE(SUM(tbl_urbano_resumen[sin_derechohabiencia]),
               SUM(tbl_urbano_resumen[total])) * 100,
        tbl_urbano_resumen[tipo_localidad_codigo] = 6
    )
    RETURN pct_rural - pct_metro

-- Efecto derechohabiencia en años de vida
Efecto Derechohabiencia Anos = 
    CALCULATE(
        MAX(tbl_coeficientes_ols[coeficiente]),
        tbl_coeficientes_ols[variable] = "Derechohabiencia"
    )
```

---

## Páginas del Reporte

### Página 1 — Portada / KPIs
**Objetivo:** primera impresión con los números más impactantes.

| Visual | Tipo | Tabla | Campos | Notas |
|--------|------|-------|--------|-------|
| Total de personas analizadas | Tarjeta | `tbl_kpis` | `total_poblacion` | Formato: 10.1M |
| % sin derechohabiencia | Tarjeta | `tbl_kpis` | `pct_sin_derechohabiencia` | Color rojo |
| % con IMSS | Tarjeta | `tbl_kpis` | `pct_imss` | Color azul |
| Edad promedio de muerte | Tarjeta | `tbl_kpis` | `edad_promedio` | |
| Distribución de cobertura | Gráfico de dona | `tbl_acceso_resumen` | `tipo_servicio`, `total` | Colores: rojo=sin cobertura, azul=IMSS |
| Imagen narrativa | Imagen | — | `figures/narrativa_salud_2000.png` | |

**Slicers:** ninguno en esta página (es portada).

---

### Página 2 — Análisis Geográfico
**Objetivo:** mostrar desigualdad entre estados.

| Visual | Tipo | Tabla | Campos | Notas |
|--------|------|-------|--------|-------|
| Mapa coroplético | Mapa de forma (Shape Map) | `tbl_estados_resumen` | `nombre_estado`, `pct_sin_cobertura` | Escala rojo claro → rojo oscuro |
| Ranking de estados | Gráfico de barras horizontales | `tbl_estados_resumen` | `nombre_estado`, `pct_sin_cobertura` | Ordenado descendente |
| Detalle del estado | Tabla | `tbl_estados_resumen` | `nombre_estado`, `con_imss`, `con_issste`, `sin_derechohabiencia`, `total_poblacion`, `pct_sin_cobertura` | Se filtra con slicer |

**Slicers:**
- Estado (segmentación de datos → `dim_estados[nombre_estado]`)

**Interacciones:** clic en mapa filtra ranking y tabla.

---

### Página 3 — Factores Sociodemográficos
**Objetivo:** mostrar cómo educación, urbanización e indigenismo determinan la cobertura.

| Visual | Tipo | Tabla | Campos | Notas |
|--------|------|-------|--------|-------|
| Cobertura por nivel educativo | Gráfico de barras agrupadas | `tbl_escolaridad_resumen` | `nivel_educativo`, `con_imss`, `sin_derechohabiencia` | Ordenar por `orden` ASC |
| Cobertura urbano vs rural | Gráfico de barras agrupadas | `tbl_urbano_resumen` | `tipo_localidad`, `con_imss`, `sin_derechohabiencia` | Excluir tipo_localidad_codigo=7 |
| Brecha indígena | Gráfico de barras | `tbl_indigena_resumen` | `habla_indigena`, `pct_sin_cobertura` | Destacar la diferencia |
| Cobertura por grupo de edad | Gráfico de líneas o barras | `tbl_grupos_edad_resumen` | `grupo_edad`, `con_imss`, `sin_derechohabiencia` | |
| % sin cobertura (medida) | Tarjeta dinámica | Medida DAX | `Pct Sin Cobertura` | Cambia con slicers |

**Slicers:**
- Tipo de localidad (`tbl_urbano_resumen[tipo_localidad]`)
- Grupo de edad (`tbl_grupos_edad_resumen[grupo_edad]`)

---

### Página 4 — Modelos Estadísticos
**Objetivo:** presentar los resultados de los modelos predictivo vs explicativo.

| Visual | Tipo | Tabla | Campos | Notas |
|--------|------|-------|--------|-------|
| Métricas por modelo | Tabla matricial | `tbl_metricas_modelos` | `modelo`, `enfoque`, `algoritmo`, `metrica`, `valor` | Formato condicional: verde=predictivo, azul=explicativo |
| Importancia de variables | Gráfico de barras horizontales | `tbl_importancia_variables` | `variable`, `importancia` | Filtrado por slicer de modelo |
| Coeficientes OLS | Gráfico de barras horizontales | `tbl_coeficientes_ols` | `variable`, `coeficiente` | Color verde=positivo, rojo=negativo |
| Significancia estadística | Tabla | `tbl_coeficientes_ols` | `variable`, `coeficiente`, `p_valor`, `significativo`, `interpretacion` | |
| Comparativa R² | Gráfico de barras agrupadas | `tbl_metricas_modelos` | `algoritmo`, `valor` | Filtrar solo métrica=R2 |
| Imagen importancia M1 | Imagen | — | `figures/importancia_modelo1.png` | |
| Imagen resultados M2 | Imagen | — | `figures/modelo2_resultados.png` | |

**Slicers:**
- Modelo (`tbl_metricas_modelos[modelo]`) → filtra importancia y métricas
- Enfoque (`tbl_metricas_modelos[enfoque]`) → Explicativo / Predictivo

---

### Página 5 — Conclusiones
**Objetivo:** cierre narrativo con los hallazgos principales.

| Visual | Tipo | Contenido |
|--------|------|-----------|
| Hallazgo 1 | Tarjeta + texto | "65.6% sin derechohabiencia — 2 de cada 3 mexicanos" |
| Hallazgo 2 | Tarjeta + texto | "Zona rural: 83.7% sin cobertura vs 43.1% en megalópolis" |
| Hallazgo 3 | Tarjeta + texto | "+3.46 años de vida con derechohabiencia (p<0.0001)" |
| Hallazgo 4 | Tarjeta + texto | "Nivel educativo: variable más determinante (77-81%)" |
| Imagen comparativa final | Imagen | `figures/comparativa_final.png` |
| Imagen narrativa | Imagen | `figures/narrativa_salud_2000.png` |

**Slicers:** ninguno (página de cierre).

---

## Relaciones entre tablas
No se requieren relaciones complejas — cada tabla es independiente y se filtra con slicers.
Si se quiere cruzar estados con urbano/rural, se puede agregar `dim_estados` como tabla puente,
pero para este reporte no es necesario.

---

## Paleta de colores recomendada

| Elemento | Color HEX |
|----------|-----------|
| Sin derechohabiencia | `#E53935` |
| Con IMSS | `#1E88E5` |
| Con ISSSTE | `#43A047` |
| Con PEMEX/SEDENA | `#FB8C00` |
| Otro / neutro | `#9E9E9E` |
| Fondo | `#FAFAFA` |
| Acento / título | `#1A237E` |

---

## Orden de construcción recomendado
1. Conectar a SQL Server e importar todas las tablas
2. Crear las medidas DAX
3. Página 1 — KPIs (más simple, sirve para verificar conexión)
4. Página 3 — Factores (la más densa, mejor hacerla con tiempo)
5. Página 2 — Mapa (requiere Shape Map con mapa de México)
6. Página 4 — Modelos
7. Página 5 — Conclusiones
