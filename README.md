# GradienViz

**Visualizador interactivo de descenso por gradiente en regresión lineal**

Herramienta pedagógica para cursos de inteligencia artificial, aprendizaje automático y optimización numérica. Implementada en p5.js como un único archivo HTML autocontenido — no requiere instalación ni servidor.

Está acompañado de cinco documentos en LaTeX que abordan el mismo problema desde perspectivas complementarias:

| Documento | Perspectiva | Público |
|---|---|---|
| **GradienVizMateBasic.tex** | Cálculo diferencial | Estudiantes de primer semestre |
| **GradienVizMate.tex** | Cálculo · Álgebra lineal · Estadística | Estudiantes de IA y aprendizaje automático |
| **GradienVizAlgLin.tex** | Álgebra lineal y proyección ortogonal | Cursos de álgebra lineal |
| **GradienVizMaxVer.tex** | Máxima verosimilitud | Cursos de estadística inferencial |
| **GradienVizMasAlla.tex** | Generalización a otros modelos | Cualquier nivel, como cierre |

Los cinco documentos comparten notación, formato y estilo. Pueden usarse de forma independiente o como colección progresiva.

---

## Uso

Descarga `GradienViz.html` y ábrelo en cualquier navegador moderno. No requiere conexión a internet una vez descargado.

```
git clone https://github.com/mancpato/GradienViz.git
cd GradienViz
# Abrir GradienViz.html en el navegador
```

---

## ¿Qué muestra?

El visualizador tiene tres paneles y un panel de controles:

### Panel izquierdo — Espacio de datos
Muestra los puntos de entrenamiento en [0,1]². Durante el entrenamiento, la recta naranja corresponde a la trayectoria en foco y converge visualmente hacia la recta óptima (gris punteada).

En la esquina superior derecha se muestra el **número de condición κ(XᵀX)**:
- κ bajo (gris): datos bien distribuidos, convergencia rápida
- κ alto (amarillo/naranja): datos mal condicionados, convergencia lenta o fallida

### Panel central — Espacio de parámetros (m, b)
El panel principal. Muestra la **superficie de costo J(m, b)** usando el colormap viridis (azul = costo bajo, amarillo = costo alto), el **campo vectorial ∇J** con flechas que indican la dirección de descenso, y las **trayectorias** del descenso por gradiente desde múltiples puntos iniciales.

El óptimo analítico `(m*, b*)` se marca con una cruz amarilla ⊕.

Al terminar, el color de cada trayectoria indica qué tan cerca llegó al óptimo:
- 🟢 Verde: convergió cerca del óptimo
- 🔴 Rojo: convergió lejos del óptimo
- ⚪ Gris: no convergió (límite de 2000 iteraciones alcanzado)

### Panel inferior izquierdo — Curva J(t)
Evolución del costo promedio sobre todas las trayectorias a lo largo de las iteraciones, con banda de desviación estándar ±σ y línea del mínimo teórico J*.

---

## Controles

### Datos
| Acción | Resultado |
|---|---|
| Click en área vacía | Agrega punto (máximo 20) |
| Arrastrar punto | Mueve el punto |
| Ctrl+Click sobre punto | Elimina el punto |
| Botón *Puntos aleatorios* | Genera entre 5 y 15 puntos aleatorios |
| Botón *Perturbar* | Añade ruido aleatorio pequeño a las coordenadas x de los puntos |

Cualquier cambio en los datos recalcula automáticamente la solución óptima y limpia las trayectorias anteriores.

### Modelo
- **Heatmap res**: resolución de la malla de la superficie J(m,b) — valores {6, 8, 10, 12, 16}
- **Trayectorias**: número de trayectorias por lado de la grilla — valores {3, 4, 5, 6} → entre 9 y 36 trayectorias totales
- **η (lr)**: tasa de aprendizaje en escala logarítmica, rango [0.001, 0.5]

### Visualización
- **Dejar rastro**: si está activo, las trayectorias dejan traza completa; si no, solo muestra el punto actual
- **Foco tray.**: índice de la trayectoria destacada en naranja (visible en ambos paneles)

### Velocidad
Controla cuántas iteraciones se ejecutan por frame. El valor 1× es suficiente para observar la convergencia en tiempo real. Para avanzar paso a paso usar los botones **+1**, **+10**, **+50** con la animación detenida.

### Entrenamiento
- **Train**: inicia el descenso por gradiente desde una grilla uniforme de puntos iniciales
- **Stop**: pausa la animación (se puede reanudar con Train)
- **Reset**: limpia trayectorias y vuelve al estado inicial

---

## Conceptos que se pueden explorar

### Convergencia y tasa de aprendizaje
Variar η permite observar tres regímenes:
- η muy pequeño (≈ 0.001): convergencia correcta pero extremadamente lenta
- η moderado (≈ 0.05): convergencia suave, visible en el panel central
- η grande (≈ 0.3–0.5): oscilaciones o divergencia; las trayectorias se alejan del óptimo

### Mal condicionamiento
Colocar los puntos casi en una línea vertical (x ≈ constante) produce κ(XᵀX) muy alto. El bowl de J(m,b) se alarga en la dirección de m y el descenso por gradiente puede requerir miles de iteraciones sin llegar al óptimo — ilustrando por qué los métodos de primer orden son sensibles al condicionamiento de los datos.

### Recuperarse de un caso mal condicionado
El botón *Perturbar* añade ruido aleatorio a las coordenadas x de los puntos, lo que aumenta su varianza y reduce κ(XᵀX) — sin alterar el modelo ni la función de costo, solo los datos. Es útil como demostración en vivo: entrenar con datos casi verticales (no converge), perturbar, y entrenar de nuevo (converge). Detalles matemáticos de por qué esta operación es preferible a regularizar la matriz artificialmente se documentan en `Perturb.md`.

### Paisaje de la función de costo
El heatmap muestra que J(m,b) es una función cuadrática convexa — siempre tiene un único mínimo global. El campo vectorial muestra que las flechas apuntan siempre hacia ese mínimo, independientemente del punto de partida.

### Dependencia del punto inicial
Con el colormap verde→rojo se puede observar que todas las trayectorias convergen al mismo punto (o muy cercano) independientemente del punto inicial — propiedad de los problemas convexos. En casos mal condicionados esta propiedad puede fallar en la práctica por el límite de iteraciones.

---

## Requisitos

- Navegador moderno con JavaScript habilitado (Chrome, Firefox, Edge, Safari)
- No requiere instalación, servidor local ni conexión a internet

---

## Contexto

Desarrollado para los cursos de Inteligencia Artificial y similares del Departamento Académico de Sistemas Computacionales (DASC), Universidad Autónoma de Baja California Sur (UABCS).

Forma parte de una colección de herramientas de visualización pedagógica implementadas en p5.js.

---

## Elaborado por

*Miguel Ángel Norzagaray Cosío*, DASC/UABCS — concepción, diseño pedagógico, ediciones menores y todas las decisiones del proyecto.
Asistencia técnica: *Claude Sonnet* (Anthropic) — implementación, depuración, optimización y documentación bajo dirección del autor.