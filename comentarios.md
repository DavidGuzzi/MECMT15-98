# Revisión técnica – Ejercicio 1: *Shocks and Crashes*

Este documento resume **posibles ajustes y mejoras** para el trabajo correspondiente al Ejercicio 1 del examen de *Métodos Empíricos en Macroeconomía Estructural*.  

Se divide en dos partes:

1. Ajustes sugeridos para el **documento PDF (respuesta escrita)**  
2. Ajustes sugeridos para el **Notebook de implementación**

El objetivo no es corregir errores graves (el trabajo está bien realizado), sino **mejorar precisión econométrica, claridad conceptual y reproducibilidad**.

---

# 1. Posibles ajustes a realizar en el PDF

## 1.1 Justificación del estimador DOLS

En la sección donde se estima la relación de cointegración, se menciona el uso de **Dynamic OLS**, pero no se explica brevemente su motivación econométrica.

Agregar 2–3 líneas como:

> El estimador Dynamic OLS (Stock y Watson, 1993) corrige el sesgo de pequeña muestra presente en regresiones cointegrantes tradicionales al incluir leads y lags de las primeras diferencias de las variables explicativas. Esto permite obtener estimadores superconsistentes del vector de cointegración aun en presencia de endogeneidad y correlación serial en los errores.

Esto muestra comprensión metodológica y no solo implementación.

---

## 1.2 Interpretación económica del vector de ajuste γ (VECM)

Se reportan los coeficientes de ajuste del VECM, pero falta interpretación económica.

Por ejemplo, si:

| Variable | γ |
|---|---|
| c | -0.016 |
| a | 0.086 |
| y | 0.019 |

Se puede agregar algo como:

> El coeficiente de ajuste es mayor para la riqueza financiera, lo que sugiere que los desequilibrios respecto de la relación de cointegración se corrigen principalmente a través de ajustes en la riqueza más que en consumo o ingreso laboral.

Esto muestra comprensión de la dinámica de corrección de error.

---

## 1.3 Construcción explícita de γ⊥ en Gonzalo–Ng

En la sección de identificación estructural se menciona que:

> γ⊥ es el espacio nulo de γ′.

Pero la consigna pedía seguir la **nota al pie 7 del paper**. Conviene mostrar explícitamente cómo se construye.

Agregar algo como:

Si

γ = (γ_c, γ_a, γ_y)′

entonces una matriz que genera el espacio ortogonal es:

γ⊥ =
[ 1   0   −γ_c/γ_y ]  
[ 0   1   −γ_a/γ_y ]

o cualquier base equivalente del *null space*.

Esto deja claro que la construcción **no es arbitraria**.

---

## 1.4 Interpretación económica de las IRF

Las IRF se presentan correctamente, pero falta profundizar su interpretación económica.

Conviene agregar comentarios como:

**Shock permanente 1 (TFP)**  
- genera aumentos permanentes en consumo, riqueza y salario.

**Shock permanente 2 (labor share)**  
- redistribuye ingresos entre capital y trabajo  
- riqueza y salario se mueven en direcciones opuestas.

**Shock transitorio**  
- produce fluctuaciones persistentes en riqueza  
- pero sin efectos permanentes en el sistema.

Esto conecta mejor la econometría con la macroeconomía.

---

## 1.5 Aclaración sobre el orden de variables

El paper utiliza el vector:

x_t = (c_t, a_t, y_t)′

Mientras que en parte de la consigna aparece:

(c_t, y_t, a_t)′

Conviene aclarar explícitamente en el trabajo:

> En este ejercicio se sigue el orden (c, a, y) utilizado en Lettau y Ludvigson para mantener consistencia con la identificación estructural y la interpretación de las IRF.

Esto evita confusión porque **Cholesky depende del orden**.

---

# 2. Posibles ajustes a realizar en el Notebook

El notebook está bien estructurado y reproduce correctamente la lógica del paper.  
Sin embargo, hay algunos puntos donde se puede mejorar **rigor econométrico y reproducibilidad**.

---

## 2.1 Ruta del archivo de datos (hardcoded path)

Actualmente el CSV se carga con una ruta fija, por ejemplo:

    df = pd.read_csv('consignas/cay_current.csv', parse_dates=['date'])

Esto puede romper el notebook si cambia la estructura de carpetas.

Se recomienda usar `pathlib`:

    from pathlib import Path

    DATA_PATH = Path("cay_current.csv")
    df = pd.read_csv(DATA_PATH, parse_dates=["date"])

Esto mejora la **portabilidad del notebook**.

---

## 2.2 Regla ad-hoc para anular coeficientes γ

En la función de identificación Gonzalo–Ng se aplica un umbral arbitrario:

    threshold = 0.30
    gamma_norm = np.abs(gamma) / np.linalg.norm(gamma)
    gamma_thresh = gamma.copy()
    gamma_thresh[gamma_norm < threshold] = 0.0

Esto **no sigue la lógica econométrica del paper**.

La consigna indica que deben anularse los coeficientes **no significativos**.

La regla debería basarse en **t-statistics o p-values**, por ejemplo:

    gamma_thresh = gamma.copy()
    gamma_thresh[np.abs(gamma_tstats) < 1.96] = 0.0

Esto hace que la identificación dependa de **significancia estadística**, no de magnitud relativa.

Este es uno de los ajustes metodológicos más importantes.

---

## 2.3 Bootstrap sin reestimar cointegración

En el bootstrap se regeneran las series pero se mantiene fijo el vector de cointegración:

    cay_boot = c_boot - alpha_a * a_boot - alpha_y * y_boot

Esto implica que **la etapa DOLS no se reestima**.

El bootstrap correcto debería repetir **todo el procedimiento de estimación**:

1. Reestimar DOLS  
2. Recalcular cay  
3. Reestimar VECM  
4. Reidentificar shocks  
5. Recalcular IRF

Es decir, dentro del loop bootstrap debería repetirse todo el pipeline econométrico.

De lo contrario, las bandas de confianza pueden quedar **demasiado optimistas**, porque no incorporan la incertidumbre de la etapa de cointegración.

---

## 2.4 Parámetro `nlags` no utilizado en VECM

La función se define como:

    def estimate_vecm(..., nlags=1):

pero el parámetro `nlags` no se utiliza realmente.

Si el modelo estimado es VECM(1), conviene simplificar:

    def estimate_vecm_1lag(...):

Esto evita confusión sobre la especificación.

---

## 2.5 Manejo de errores en el bootstrap

Actualmente cuando una réplica falla se copia la anterior:

    irf_draws[..., draw] = irf_draws[..., draw-1]

Esto introduce dependencia artificial entre draws.

Es mejor usar `NaN` y luego ignorarlos al calcular percentiles:

    irf_draws[..., draw] = np.nan

y posteriormente:

    np.nanpercentile(irf_draws, ...)

Esto mantiene independencia entre réplicas bootstrap.

---

## 2.6 Normalización de signos duplicada

La lógica para fijar el signo de los shocks aparece repetida en varias celdas.

Conviene encapsularla en una función única, por ejemplo:

    def normalize_shock_signs(irf_levels, irf_diff=None):

        sign_refs = [
            irf_levels[40,0,0],  # shock permanente 1
            irf_levels[40,1,1],  # shock permanente 2
            irf_levels[0,1,2],   # shock transitorio
        ]

        for s in range(3):
            if sign_refs[s] < 0:
                irf_levels[:,:,s] *= -1
                if irf_diff is not None:
                    irf_diff[:,:,s] *= -1

        return irf_levels, irf_diff

Esto mejora la **claridad y consistencia del código**.

---

## 2.7 Documentación del orden de variables

El código usa consistentemente:

(c, a, y)

Conviene documentarlo en una celda markdown inicial del notebook:

> Todas las matrices se construyen siguiendo el orden (consumption, assets, labor income) utilizado en Lettau y Ludvigson (2014).

Esto evita errores futuros al interpretar matrices estructurales o aplicar Cholesky.

---

# Evaluación final

El trabajo está **bien ejecutado y metodológicamente sólido**.

Las mejoras sugeridas apuntan principalmente a:

- mayor claridad econométrica en el PDF  
- mayor rigor en bootstrap e identificación en el código  
- mayor reproducibilidad del notebook  

Con estos ajustes el ejercicio quedaría **técnicamente muy robusto para evaluación académica**.