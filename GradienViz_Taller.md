---
marp: true
theme: default
paginate: true
size: 16:9
style: |
  section {
    font-size: 26px;
  }
  h1 {
    color: #1e3a5f;
  }
  h2 {
    color: #2563eb;
  }
  code {
    background: #f0f4ff;
  }
  .small {
    font-size: 20px;
  }
  .center {
    text-align: center;
  }
---

<!-- _class: lead -->
# GradienViz

## Un problema, cinco puertas de entrada

**De la regresión lineal al aprendizaje de redes neuronales**

Taller de Matemáticas — UABCS
Miguel Ángel Norzagaray Cosío

---

## Antes de empezar

Descarguen o abran ahora **GradienViz.html** en su equipo:

```
https://github.com/mancpato/GradienViz
```

- Un solo archivo HTML, sin instalación
- Funciona sin conexión a internet
- Vamos a usarlo varias veces durante la charla

<!--
Dar 2-3 minutos para que abran el repositorio y el archivo.
Si hay wifi, mostrar QR. Si no, USB con el archivo listo.
-->

---

## De qué se trata esta charla

No es una charla sobre regresión lineal.

Es una charla sobre **cómo un mismo problema matemático puede tener
varias puertas de entrada**, según las herramientas que ya domina
el estudiante — y cómo eso se puede convertir en material didáctico
reutilizable.

El ejemplo concreto: **GradienViz**, una herramienta interactiva
más cinco documentos que resuelven el mismo problema desde:
cálculo, álgebra lineal, estadística, y su generalización.

---

<!-- _class: lead -->
# Acto 1
## El problema está en todas partes

---

## La regresión lineal no es un ejercicio de libro

Es la herramienta cuantitativa más usada en ciencia aplicada:

- **Biología** — alometría: relación entre tamaño corporal y metabolismo
- **Economía** — series de tiempo simples, elasticidad precio-demanda
- **Física / Ingeniería** — calibración de instrumentos de medición
- **Geología** — tendencias en series de datos ambientales
- **Ciencias sociales** — correlación entre variables socioeconómicas
- **Astronomía** — la propia Ley de Hubble es una regresión lineal

---

## El mismo modelo, siempre

$$y = mx + b$$

Dos parámetros. Una recta. Un conjunto de datos que nunca cae
exactamente sobre ella.

La pregunta que todo estudiante enfrenta tarde o temprano:

> **¿Cuáles son los mejores valores de $m$ y $b$?**

---

<!-- _class: lead -->
# Acto 2
## La solución que ya conocemos

---

## La fórmula que todos enseñamos

$$
m^* = \frac{n\sum x_iy_i - \sum x_i \sum y_i}{n\sum x_i^2 - (\sum x_i)^2}
\qquad
b^* = \frac{\sum y_i - m^*\sum x_i}{n}
$$

Mínimos cuadrados. Una fórmula cerrada, exacta, de un solo paso.

**Este es el punto de partida común** — lo que la mayoría de
nuestros estudiantes ya conocen antes de llegar a este taller.

---

## La pregunta incómoda

Si ya existe la fórmula exacta...

> **¿por qué enseñar un algoritmo iterativo para resolver
> el mismo problema?**

La respuesta a esa pregunta es el hilo conductor de todo
lo que sigue — y es también la puerta hacia el aprendizaje
automático moderno.

---

<!-- _class: lead -->
# Acto 3
## Cinco preguntas, cinco respuestas

---

## El mismo problema, cinco perspectivas

| Documento | Pregunta central |
|---|---|
| **Cálculo** | ¿Hacia dónde bajar? |
| **Álgebra lineal** | ¿Por qué siempre hay un único mínimo? |
| **Máxima verosimilitud** | ¿Por qué el cuadrado y no otra cosa? |
| **Generalización** | ¿Y si el modelo no es una recta? |
| **Condicionamiento** | ¿Qué tan mal puede salir? |

Cada una usa las herramientas de una materia distinta.
Mismo destino, caminos diferentes.

---

## Perspectiva 1 — Cálculo: ¿hacia dónde bajar?

La función de costo $J(m,b)$ es una superficie con forma de tazón.
Las derivadas parciales indican la dirección de mayor descenso:

$$
\frac{\partial J}{\partial m}, \quad \frac{\partial J}{\partial b}
$$

El algoritmo: dar pasos pequeños en esa dirección, repetidamente.

**🖥️ Prueben ahora:** en GradienViz, arrastren un punto en el
Panel de datos y observen cómo se mueve la recta naranja durante
el entrenamiento.

---

## Perspectiva 2 — Álgebra lineal: ¿por qué un único mínimo?

$$
J(\mathbf{x}) = \frac{1}{n}\lVert \mathbf{A}\mathbf{x} - \mathbf{b} \rVert^2
$$

Esta función es **siempre convexa**: su Hessiana
$\frac{2}{n}\mathbf{A}^\top\mathbf{A}$ es semidefinida positiva
para cualquier conjunto de datos.

No hay mínimos locales que puedan engañar al algoritmo.

**🖥️ Prueben ahora:** observen el mapa de calor del Panel 2 —
sin importar los datos, siempre es un único valle.

---

## Perspectiva 3 — Estadística: ¿por qué el cuadrado?

Si asumimos que el ruido en los datos es gaussiano:

$$
y_i = mx_i + b + \varepsilon_i, \qquad \varepsilon_i \sim \mathcal{N}(0,\sigma^2)
$$

entonces minimizar el error cuadrático medio
**equivale exactamente** a maximizar la verosimilitud
de haber observado esos datos.

No es una elección arbitraria. Es la consecuencia de un supuesto
estadístico explícito.

---

## El cuadrado no es la única opción

| Supuesto de ruido | Función de costo resultante |
|---|---|
| Gaussiano | Error cuadrático medio |
| Laplace | Error absoluto medio |
| Bernoulli | Entropía cruzada (clasificación) |

**Elegir una función de costo es elegir una hipótesis
sobre los datos.**

---

## Perspectiva 4 — ¿Y si el modelo no es una recta?

Muchos modelos no lineales se **linealizan**:

| Modelo | Transformación |
|---|---|
| Exponencial $y=ae^{bx}$ | $\ln y = \ln a + bx$ |
| Potencial $y=ax^b$ | $\ln y = \ln a + b\ln x$ |
| Polinomial | Agregar columnas $x^2, x^3,\ldots$ |
| Múltiple | Agregar columnas $x_1, x_2, \ldots$ |

**La ecuación normal $\mathbf{A}^\top\mathbf{A}\mathbf{x}^*=\mathbf{A}^\top\mathbf{b}$
no cambia** — solo cambian las columnas de $\mathbf{A}$.

---

## Perspectiva 5 — ¿Qué tan mal puede salir?

El número de condición $\kappa(\mathbf{A}^\top\mathbf{A})$ predice
la dificultad de convergencia. Datos casi colineales en $x$
producen $\kappa$ enorme y un valle extremadamente elongado.

**🖥️ Prueben ahora:** coloquen 3 puntos casi en línea vertical,
entrenen (no converge), presionen **Perturbar**, entrenen de nuevo.

Vean cómo cambia $\kappa$ y cómo cambia el comportamiento.

---

<!-- _class: lead -->
# Acto 4
## El puente hacia las redes neuronales

---

## Contando parámetros

Regresión lineal simple: **2 parámetros** — $m$ y $b$.

Una red neuronal minúscula, 2 entradas → 4 neuronas ocultas → 1 salida:

$$
(2\times 4) + 4 + (4\times 1) + 1 = 17 \text{ parámetros}
$$

Un modelo de lenguaje grande actual: **cientos de miles de millones**
de parámetros.

---

## Lo que se rompe al escalar

1. **No existe fórmula cerrada.** La función de activación no lineal
   hace que $J(\boldsymbol\theta)$ deje de ser cuadrática.

2. **La superficie deja de ser un tazón.** Puede tener múltiples
   mínimos, puntos de silla, regiones planas extensas.

3. **El espacio deja de visualizarse.** $\mathbb{R}^{17}$,
   $\mathbb{R}^{17\,000}$, $\mathbb{R}^{10^{11}}$ — ninguno cabe
   en una pantalla.

---

## Un término para quien quiera profundizar

<div class="small">

La geometría del espacio de parámetros de una red neuronal no es
un elipsoide: tiene **singularidades** — puntos donde la Hessiana
pierde rango, análogas a una curva con auto-intersección o un
cono con vértice.

Es un tema de investigación activa (teoría de aprendizaje estadístico
singular), no material de un curso estándar. Se menciona aquí
solo para que sepan que existe, si algún estudiante quiere
profundizar en geometría diferencial o teoría algebraica aplicada
al aprendizaje automático.

</div>

---

## Lo único que no cambia

$$
\boldsymbol\theta^{(t+1)} = \boldsymbol\theta^{(t)} - \eta\,\nabla J(\boldsymbol\theta^{(t)})
$$

Esta es exactamente la misma regla que vieron hace unos minutos
en GradienViz.

Cambia el modelo. Cambia la dimensión. Cambia cómo se calcula
el gradiente (retropropagación en vez de fórmulas directas).

**La iteración es idéntica.**

---

<!-- _class: lead -->
# Acto 5
## El patrón, no solo la herramienta

---

## Lo que GradienViz realmente demuestra

No es solo una herramienta de regresión lineal.

Es un **patrón replicable**:

1. Un problema matemático genuinamente transversal
2. Una herramienta interactiva honesta — usa datos y gradientes reales
3. Múltiples documentos, cada uno desde una materia distinta
4. Notación y estilo uniformes entre todos ellos

---

## Una honestidad importante

Un primer prototipo de GradienViz usaba una función de costo
**inventada** — un paraboloide artificial elegido para verse bonito.

La versión final calcula $J(m,b)$ con **los datos reales** que el
usuario coloca.

Cuando los datos están mal condicionados, el algoritmo
**realmente** tarda o falla en converger — porque así se comporta
el gradiente descendente en la práctica.

**Una simulación pedagógica solo enseña bien si no miente.**

---

## Invitación

¿Qué problema de su materia tiene:

- una versión con solución exacta conocida,
- una versión que requiere un método iterativo o aproximado,
- y varias herramientas matemáticas que lo abordan
  desde ángulos distintos?

Ese problema es candidato para el mismo tratamiento.

---

<!-- _class: lead -->
# Gracias

**Repositorio completo:**
`github.com/mancpato/GradienViz`

Herramienta interactiva + 5 documentos LaTeX
(cálculo, álgebra lineal, estadística, generalización, y esta presentación)

Miguel Ángel Norzagaray Cosío
DASC — UABCS
