# Métodos de Muestreo Bayesiano

Este directorio contiene tres notebooks de Jupyter que demuestran diferentes técnicas de muestreo para generar muestras de distribuciones de probabilidad cuando solo se conoce el kernel (distribución no normalizada). Estos métodos son fundamentales en inferencia bayesiana y estadística computacional.

## Distribución Objetivo Común

Los tres notebooks trabajan con la misma distribución objetivo: una **mezcla de dos distribuciones gaussianas** definida por el kernel:

$$k(x) = \eta \cdot \mathcal{N}(x|\mu_1, \sigma_1^2) + (1-\eta) \cdot \mathcal{N}(x|\mu_2, \sigma_2^2)$$

**Parámetros:**
- $\eta = 0.3$ (peso de la primera componente)
- $\mu_1 = 0$, $\sigma_1^2 = 2.5$ (primera gaussiana)
- $\mu_2 = 10$, $\sigma_2^2 = 2.5$ (segunda gaussiana)

Esta distribución bimodal representa un caso de prueba ideal para comparar diferentes algoritmos de muestreo.

---

## Notebooks

### 1. [Acceptance_Rejection_example.ipynb](Acceptance_Rejection_example.ipynb)

**Método de Aceptación-Rechazo**

Este notebook implementa el algoritmo clásico de Aceptación-Rechazo para muestrear de una distribución cuando solo se conoce su kernel.

**Conceptos Clave:**
- **Distribución propuesta**: Se usa una gaussiana estándar $g(x) = \mathcal{N}(0, 1)$
- **Función envolvente**: Constante $M = 12$ tal que $k(x) \leq M \cdot g(x)$ para todo $x$
- **Criterio de aceptación**: Aceptar muestra $x$ con probabilidad $\frac{k(x)}{M \cdot g(x)}$
- **Tasa de aceptación**: Aproximadamente 33% de las muestras propuestas son aceptadas

**Ventajas:**
- Genera muestras **independientes** (no correlacionadas)
- Fácil de implementar y entender
- Garantiza muestras exactas de la distribución objetivo

**Dependencias:** `numpy`, `matplotlib`, `scipy.stats`

---

### 2. [RWMH_example.ipynb](RWMH_example.ipynb)

**Random Walk Metropolis-Hastings (RWMH)**

Implementación del algoritmo RWMH, un método de Monte Carlo por Cadenas de Markov (MCMC) que genera una cadena de muestras correlacionadas cuya distribución estacionaria es la distribución objetivo.

**Conceptos Clave:**
- **Caminata aleatoria**: Propuestas generadas como $y \sim \mathcal{N}(x_t, \Sigma)$ desde el estado actual
- **Criterio de Metropolis**: Aceptar con probabilidad $\alpha = \min\left(1, \frac{k(y)}{k(x_t)}\right)$
- **Burn-in**: Se descartan las primeras 10,000 muestras para alcanzar la distribución estacionaria
- **Tasa de aceptación**: Aproximadamente 33% (óptima para distribuciones unimodales)

**Diagnósticos de Convergencia:**
- **Test de Geweke (1992)**: Compara medias del primer 10% y último 50% de muestras
- **Autocorrelación**: Análisis de la correlación entre muestras consecutivas
- **Running mean**: Convergencia de la media muestral

**Ventajas:**
- Aplicable a distribuciones de alta dimensión y formas complejas
- No requiere conocer la constante de normalización
- Flexible y ampliamente utilizado en inferencia bayesiana

**Dependencias:** `numpy`, `matplotlib`, `scipy.stats`

---

### 3. [SIR_example.ipynb](SIR_example.ipynb)

**Sampling-Importance Resampling (SIR)**

Este notebook demuestra el método SIR, que combina muestreo por importancia con un paso de remuestreo para obtener muestras (aproximadamente) de la distribución objetivo.

**Conceptos Clave:**
- **Pesos de importancia**: $\omega_i = \frac{k(x_i)}{g(x_i)}$ donde $x_i \sim g(x)$
- **Probabilidades normalizadas**: $q_i = \frac{\omega_i}{\sum_j \omega_j}$
- **Remuestreo**: Muestras finales obtenidas por muestreo discreto usando la CDF inversa
- **Regla de oro**: Número de muestras remuestreadas $M \leq \frac{N}{10}$ donde $N$ es el tamaño inicial

**Algoritmo:**
1. Generar $N$ muestras de la distribución propuesta $g(x)$
2. Calcular pesos de importancia para cada muestra
3. Normalizar pesos para obtener probabilidades discretas
4. Remuestrear $M$ muestras según estas probabilidades

**Ventajas:**
- Eficiente cuando los pesos de importancia están bien calibrados
- Genera muestras aproximadamente independientes
- Útil cuando el remuestreo es computacionalmente más barato que generar nuevas muestras

**Dependencias:** `numpy`, `matplotlib`, `scipy.stats`

---

## Comparación de Métodos

| Método | Independencia | Tasa de Aceptación | Complejidad | Mejor Uso |
|--------|---------------|-------------------|-------------|-----------|
| **Aceptación-Rechazo** | Muestras independientes | ~33% | Baja | Distribuciones simples, cuando se puede encontrar buen $M$ |
| **RWMH** | Muestras correlacionadas | ~33% | Media | Distribuciones complejas, alta dimensión |
| **SIR** | Aproximadamente independientes | N/A (remuestreo) | Media | Cuando calcular pesos es más barato que generar muestras |

**Consideraciones:**
- **Aceptación-Rechazo**: Excelente cuando $M$ es pequeño (alta eficiencia). Difícil en alta dimensión.
- **RWMH**: Robusto y flexible, pero requiere diagnósticos de convergencia y manejo de autocorrelación.
- **SIR**: Balance entre independencia y eficiencia, pero muestras son aproximadas.

---

## Requisitos y Uso

### Requisitos
- Python 3.7+
- Jupyter Notebook o JupyterLab
- Librerías necesarias:
  ```bash
  pip install numpy matplotlib scipy
  ```

### Ejecución
1. Navegar al directorio Bayes
2. Iniciar Jupyter:
   ```bash
   jupyter notebook
   ```
3. Abrir cualquiera de los tres notebooks
4. Ejecutar las celdas secuencialmente

Cada notebook es autocontenido y puede ejecutarse independientemente.

---

## Referencias

- Geweke, J. (1992). "Evaluating the accuracy of sampling-based approaches to calculating posterior moments"
- Metropolis, N., et al. (1953). "Equation of State Calculations by Fast Computing Machines"
- Hastings, W.K. (1970). "Monte Carlo Sampling Methods Using Markov Chains and Their Applications"
