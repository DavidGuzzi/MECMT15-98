# MECMT15-98 — Métodos Empíricos en Macroeconomía Estructural

**Maestría en Econometría** | Examen 2025

---

## Estructura del Repositorio

```
MECMT15-98/
├── consignas/                        # Enunciados, papers y datos
│   ├── Examen2025.pdf                # Enunciado del examen
│   ├── Shocks and Crashes.pdf        # Lettau & Lydvigson (2014) — paper principal
│   ├── Shocks and Crashes - Appendix.pdf
│   ├── Systematic Framework for Analyzing.pdf  # Gonzalo & Ng (2001)
│   └── cay_current.csv               # Datos trimestrales 1952:Q1–2019:Q4
│
├── notas/                            # Material teórico del curso
│   ├── 1. Review_TimeSeries.pdf
│   ├── 2. Frequency_Domain.pdf
│   ├── 3. Filtering.pdf  /  3. IntroBayesian.pdf
│   ├── 3B. SlidesBVAR.pdf
│   ├── 4. Slides_SolvingDSGE.pdf  /  5. SolvingDSGE.pdf
│   ├── 5. SlidesSVAR.pdf             ← más relevante para Ej.1
│   ├── Algoritmo_RWZ.pdf             ← restricciones de largo plazo
│   ├── BayesianVARLectureNotes.pdf
│   ├── MLE_Kalman.pdf
│   ├── GMM_SMM_II.pdf
│   └── ...
│
├── programas/                        # Programas del profesor (Matlab + Python)
│   ├── Programas del set the notas SVAR/
│   │   ├── BlanchardQuah_class.ipynb ← template Python más relevante para Ej.1
│   │   ├── Long-run restrictions/BQVAR.m
│   │   ├── Long-run restrictions/var_ls.m
│   │   └── Sign restrictions/SignRestrict.m
│   ├── Programas VAR bayesianos/
│   ├── Programas GMM/
│   ├── Programas handout Maximum Likelihood/
│   ├── Programas dynamic factor models/
│   ├── Programas modelo plain RBC/
│   └── Programas New Keynesian model/
│
├── resumenes/                        # Resúmenes en Markdown
│   ├── ShocksAndCrashes_LL2014.md    # Resumen Lettau & Lydvigson (2014)
│   ├── GonzaloNg_2001.md             # Resumen Gonzalo & Ng (2001)
│   └── Notas_Relevantes_Ej1.md       # Resumen notas: SlidesSVAR, RWZ, TimeSeries
│
├── Ejercicio1_ShocksAndCrashes.ipynb ← SOLUCIÓN EJERCICIO 1
└── MEMORY.md                         # Memoria del proyecto (Claude)
```

---

## Ejercicio 1: Shocks and Crashes (Lettau & Lydvigson, 2014)

**Objetivo:** Replicar el paper "Shocks and Crashes" (NBER Macroeconomics Annual, 2014) usando un VECM estructural con la descomposición de Gonzalo-Ng (2001).

### Variables

| Variable | Descripción | Columna CSV |
|----------|-------------|-------------|
| `c_t` | Log consumo | `c` |
| `a_t` | Log riqueza financiera | `w` (renombrar a `a`) |
| `y_t` | Log ingreso laboral | `y` |
| `cay_t` | Variable de cointegración: `c - α_a·a - α_y·y` | `cay=...` |

Las tres variables son I(1) y cointegran: `cay_t = c_t - α_a·a_t - α_y·y_t ~ I(0)`.

### Incisos

| # | Tarea | Método | Referencia |
|---|-------|--------|-----------|
| 1 | Construir `cay_t` | DOLS (8 leads/lags de Δa y Δy) | — |
| 2 | Estimar VECM(1) | OLS ecuación por ecuación | Tabla 2 del paper |
| 3 | Identificar shocks | Gonzalo-Ng + Cholesky | Figura 5 del paper |
| 4 | Interpretación económica | Análisis de IRF | — |
| 5 | IRF en diferencias | Forma compañera VAR(2) | — |
| 6 | IRF en niveles | Suma acumulada de IRF en dif. | Figura 4 del paper |
| Bonus | Bandas bootstrap | Bootstrap naïve | Metodología del curso |

### Modelo

**DOLS (Inciso 1):**
```
c_t = k + α_a·a_t + α_y·y_t + Σ_{j=1}^8 [β_j·Δa_{t-j} + γ_j·Δy_{t-j}]
                              + Σ_{j=1}^8 [δ_j·Δa_{t+j} + θ_j·Δy_{t+j}] + u_t
```

**VECM(1) en forma reducida (Inciso 2):**
```
Δx_t = v + γ·cay_{t-1} + Γ·Δx_{t-1} + e_t,   E[e_t e_t'] = Ω
```

**Descomposición Gonzalo-Ng (Inciso 3):**
```
G = [γ_⊥' ; α']   (3×3)
donde: α = [1, -α_a, -α_y]',  γ_⊥ satisface γ_⊥·γ = 0
Shocks ortogonales: η_t = H⁻¹·G·e_t,  con H = chol(G·Ω·G')
```

### Archivo de solución

`Ejercicio1_ShocksAndCrashes.ipynb` — notebook Python self-contained.

**Dependencias:** `numpy`, `pandas`, `matplotlib`, `scipy`

---

## Papers de Referencia

1. **Lettau, M. & Ludvigson, S. (2014)** — "Shocks and Crashes", NBER Macroeconomics Annual.
   → Identifica 2 shocks permanentes (TFP + Labor Share) y 1 transitorio (financiero/demanda).
   → Resumen: `resumenes/ShocksAndCrashes_LL2014.md`

2. **Gonzalo, J. & Ng, S. (2001)** — "A Systematic Framework for Analyzing the Dynamic Effects of Permanent and Transitory Shocks", Journal of Economic Dynamics and Control.
   → Provee el marco teórico para la identificación estructural del VECM.
   → Resumen: `resumenes/GonzaloNg_2001.md`

---

## Resúmenes

Los archivos en `resumenes/` documentan el contenido de los papers y notas relevantes para el Ejercicio 1:

| Archivo | Fuente | Contenido principal |
|---------|--------|---------------------|
| `ShocksAndCrashes_LL2014.md` | Lettau & Lydvigson (2014) | Datos, DOLS, VECM Tabla 2, Gonzalo-Ng, interpretación de shocks, Figuras 4 y 5 |
| `GonzaloNg_2001.md` | Gonzalo & Ng (2001) | Proposición 1, construcción de G = [γ_⊥'; α'], Cholesky, caveats, paso a paso |
| `Notas_Relevantes_Ej1.md` | SlidesSVAR + RWZ + TimeSeries | SVAR, identificación BQ, algoritmo RWZ, Wold, proyecciones lineales |
