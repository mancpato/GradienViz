
# Sobre la perturbación del sistema singular

**Qué controla realmente el condicionamiento**

Hay una identidad que hace todo transparente:

$$\det(\mathbf{X}^\top\mathbf{X}) = n\sum x_i^2 - \left(\sum x_i\right)^2 = n^2\,\mathrm{Var}(x)$$

El determinante depende **únicamente de la varianza de los $x_i$** — no de su media, no de su orden, no de los $y_i$. Y cuando la matriz está mal condicionada, $\lambda_1 \approx \mathrm{tr}$, así que:

$$\kappa \approx \frac{\mathrm{tr}^2}{\det} = \frac{(\sum x_i^2 + n)^2}{n^2\,\mathrm{Var}(x)}$$

$\kappa$ es inversamente proporcional a la varianza de $x$. Todo el problema de condicionamiento en este modelo se reduce a una sola cantidad escalar.

---

**Qué hace el jitter aleatorio**

Cada punto recibe un desplazamiento independiente $\varepsilon_i \sim U[-a, a]$, con $a = 0.05$. Algunos suman, otros restan — sí, es aleatorio punto por punto. Como $\varepsilon$ es independiente de $x$:

$$\mathrm{Var}(x + \varepsilon) \approx \mathrm{Var}(x) + \mathrm{Var}(\varepsilon) = \mathrm{Var}(x) + \frac{a^2}{3}$$

Con $a = 0.05$ eso inyecta $\approx 8.3\times10^{-4}$ de varianza. En tu caso de la captura ($\kappa = 488\,451$, lo que implica $\sigma_x \approx 0.0017$), la varianza pasa de $\sim 3\times10^{-6}$ a $\sim 8.4\times10^{-4}$: un factor de ~300. Y $\kappa$ cae proporcionalmente, de ~488 000 a ~1 600.

De "no converge nunca" a "converge lento pero converge". Esa es la demostración.

---

**El contraste con $\mathbf{A} + \delta\mathbf{I}$**

Son operaciones estructuralmente distintas:

| | $\mathbf{X}^\top\mathbf{X} + \delta\mathbf{I}$ | Jitter en los datos |
|---|---|---|
| Entrada $(1,1) = \sum x_i^2$ | $+\delta$ | cambia |
| Entrada $(2,2) = n$ | $+\delta$ | **no cambia** |
| Fuera de diagonal $\sum x_i$ | **no cambia** | cambia |
| ¿Corresponde a datos reales? | No | Sí |
| Solución resultante | Ridge, sesgada | Mínimos cuadrados, insesgada |

La perturbación de matriz altera solo la diagonal y produce una $\mathbf{X}^\top\mathbf{X}$ que **no corresponde a ningún conjunto de datos**: es una matriz ficticia cuya solución está sesgada hacia el origen. El jitter produce una $\mathbf{X}^\top\mathbf{X}$ genuina, la de otros datos.

Tu observación sobre "la diagonal contraria" apunta justo a esto: en $\mathbf{A}+\delta\mathbf{I}$ la elección de qué entradas tocar es una decisión de diseño con consecuencias. Con jitter no hay tal decisión — las entradas cambian como consecuencia de mover datos reales, no por construcción.

---

**Por si hay objeciones:**

**"¿Es distinto alterar puntos del centro que puntos extremos?"** — Sí, y tienes razón. La varianza es $\frac{1}{n}\sum(x_i - \bar{x})^2$: mover un punto que ya está lejos de $\bar{x}$ aún más lejos aumenta la varianza más que mover uno central. Con ruido aleatorio independiente eso se promedia, pero en una realización particular el efecto varía. Por eso dos clics consecutivos de [Perturbar] no dan el mismo $\kappa$.

**"Puntos con $y$ máxima"** — Aquí hay que separar dos cosas. Para $\kappa$ no importa en absoluto: la $y$ no aparece en $\mathbf{X}^\top\mathbf{X}$. Pero para la *solución* sí importa mucho: mover el $x$ de un punto con $y$ extremo tiene alto apalancamiento sobre la recta ajustada. Entonces el estudiante verá dos efectos simultáneos — $\kappa$ baja **y** la cruz ⊕ se mueve. Eso es honesto: cambiaron los datos, cambió la respuesta.

---

**Una alternativa determinista, y por qué no sirve aquí**

Se podría dilatar los puntos respecto a su media: $x_i \to \bar{x} + (1+\alpha)(x_i - \bar{x})$, lo que multiplica la varianza por $(1+\alpha)^2$ de forma exacta y controlada, sin aleatoriedad.

Pero falla precisamente en el caso que motivó todo esto: si todos los $x_i$ son idénticos, las desviaciones son cero y escalarlas sigue dando cero. Para salir de la singularidad exacta hay que inyectar información nueva, y eso requiere aleatoriedad.
