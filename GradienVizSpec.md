# ESPECIFICACIÓN: GradientViz v2 – Visualizador de Descenso por Gradiente en Regresión Lineal

---

## 0. ACLARACIONES DE NOTACIÓN

- `J(m, b)` = función de costo = MSE = `(1/n) · Σ(yᵢ − m·xᵢ − b)²`
- Se usa `J` en el espacio de parámetros (Panel 2, Panel 3) y `MSE` en el espacio de datos (Panel 1)
- `η` = learning rate (tasa de aprendizaje)
- `(m*, b*)` = solución analítica óptima (antes llamada `m_opt`, `b_opt`)

---

## 1. DATOS Y ESTADO GLOBAL

### 1.1 Estructura de puntos de datos

```javascript
puntos: Array<{x: number, y: number}>
```

- **Rango**: [0, 1] × [0, 1]
- **Máximo**: 20 puntos, **mínimo para entrenar**: 3
- **Editable**: antes de presionar Train
- **Arrastrables**: sí, solo cuando no está animando

### 1.2 Solución óptima (recalculada al cambiar puntos)

```javascript
m_star: number   // solución analítica
b_star: number
J_star: number   // J(m*, b*) = costo mínimo teórico
```

### 1.3 Rango dinámico del espacio (m, b) — Panel 2

```javascript
// Calculado al cambiar puntos o parámetros
m_rango: [m_star - delta_m, m_star + delta_m]  // delta_m = max(0.8, |m_star| * 0.6)
b_rango: [b_star - delta_b, b_star + delta_b]  // delta_b = max(0.6, |b_star| * 0.6)
// Siempre contiene (m*, b*) aproximadamente en el centro
// Se recalcula cada vez que cambia la solución óptima
```

**Motivación**: para datos aleatorios en [0,1]², `m*` puede exceder 1 fácilmente.
Fijar [0,1]×[0,1] haría que el óptimo quede fuera de la visualización.

### 1.4 Parámetros de visualización

```javascript
resHeatmap:     number ∈ {6, 8, 10, 12, 16}  // resolución del mapa de calor (lado)
numTrayPorLado: number ∈ {3, 4, 5, 6}        // número de trayectorias por lado
learningRate:   number ∈ [0.001, 0.5]         // escala logarítmica en slider
maxIteraciones: number = 2000                  // límite de seguridad (obligatorio)
tolerancia:     number = 1e-4                  // ||∇J|| < tolerancia → convergida
```

### 1.5 Parámetros de animación

```javascript
animando:     boolean
dejarRastro:  boolean   // checkbox, default true
pasos:        number    // iteraciones transcurridas
trajEnFoco:   number    // índice de la trayectoria destacada en Panel 1 (default 0)

trayectorias: Array<{
  m_init:    number,
  b_init:    number,
  historial: Array<{m, b, J, iter}>,
  convergida:        boolean,
  iterConvergencia:  number | null,
  colorFinal:        [r, g, b] | null   // asignado AL CONVERGER, null durante animación
}>
```

### 1.6 Historial de costo global

```javascript
historialJ: Array<{
  iter:    number,
  valores: Array<number>,   // J de cada trayectoria en este paso
  media:   number,
  desv:    number
}>
```

---

## 2. PANEL 1 – ESPACIO DE DATOS

### 2.1 Dimensiones y layout

- **Cuadrado**: `L = min(W, H) * 0.42` donde W, H = dimensiones del canvas
- **Posición**: columna izquierda, alineado arriba
- **Fondo**: blanco (`#fff`)
- **Borde**: `#ddd`, 1.5px

### 2.2 Contenido

- **Título**: "Espacio de datos"
- **Ejes**: x, y ∈ [0, 1], ticks cada 0.2, etiquetas en fuente monospace pequeña
- **Puntos**: círculos `r = 6px`, azul (`#2563eb`), borde blanco 1.5px, arrastrables
- **Recta fija**: `y = m*·x + b*` en gris claro punteado (`#aaa`, dash), grosor 1.5px
  - Siempre visible, representa la solución analítica
- **Recta dinámica**: `y = m_tray·x + b_tray` de la trayectoria en foco
  - Color: naranja (`#f97316`), grosor 2px
  - Solo visible durante animación
  - Permite ver en tiempo real cómo la recta converge al óptimo
- **Información textual** (abajo del panel, monospace pequeño):
  ```
  Puntos: N | MSE actual: X.XXXX
  m* = X.XXX   b* = X.XXX
  [Durante animación] → m = X.XXX  b = X.XXX  J = X.XXXX
  ```

### 2.3 Interacciones

- **Click en área vacía**: agrega punto (máx 20), deshabilitado si animando
- **Drag sobre punto**: mueve punto (solo si no animando)
- **Backspace / Delete** con punto seleccionado: elimina punto
- **Cualquier cambio en puntos** → recalcular `(m*, b*, J*)`, actualizar `m_rango`, `b_rango`, limpiar historial, pausar animación

---

## 3. PANEL 2 – ESPACIO DE PARÁMETROS (m, b)

### 3.1 Dimensiones y layout

- **Cuadrado**: igual tamaño que Panel 1
- **Posición**: a la derecha del Panel 1, misma alineación vertical
- **Fondo**: negro (`#111`) — contraste para el heatmap
- **Borde**: `#333`, 1.5px

### 3.2 Ejes

- **Eje m (horizontal)**: `m_rango = [m_lo, m_hi]`
- **Eje b (vertical)**: `b_rango = [b_lo, b_hi]`
- Ticks y etiquetas adaptativos (3–5 por eje), fuente monospace pequeña blanca
- Los rangos se recalculan dinámicamente centrados en `(m*, b*)`

### 3.3 Capas de renderizado (de atrás a adelante)

#### Capa 1: Mapa de calor J(m, b)

- **Resolución**: `resHeatmap × resHeatmap` celdas
- **Colormap viridis**: azul oscuro → verde → amarillo (bajo → alto costo)
- **Interpolación bilineal** entre celdas para suavidad
- **Precalculado** al cambiar puntos o rangos (no cada frame)

#### Capa 2: Campo vectorial ∇J

- **Grilla**: 9×9 (fija, independiente de `resHeatmap`)
- **En cada nodo**: flecha antiparalela a ∇J (dirección de descenso)
- **Color**: blanco semitransparente (`rgba(255,255,255,0.45)`), grosor 1px
- **Longitud**: proporcional a `||∇J||`, normalizada al 80% de la celda
- **Precalculado** con la malla (mismo trigger)

#### Capa 3: Trayectorias

- **Número**: `numTrayPorLado²` trayectorias, inicios en grilla uniforme sobre `m_rango × b_rango`
- **Durante animación**: todas en color blanco semitransparente (`rgba(255,255,255,0.6)`), grosor 1.5px
- **Al converger** (cada trayectoria individualmente):
  - Se calcula `distancia = ||(m_final − m*, b_final − b*)||`
  - Se normaliza: `t = min(distancia / d_max, 1.0)` donde `d_max` es la mayor distancia entre todas las trayectorias
  - Color: `HSL(120 − 120·t, 90%, 55%)` → verde oscuro (cerca) a rojo (lejos)
  - Este color se aplica **retroactivamente** a toda la traza de esa trayectoria
- **Checkbox "Dejar rastro"** (default checked):
  - checked: línea permanente
  - unchecked: solo el punto actual (sin traza histórica)
- **Trayectoria en foco** (`trajEnFoco`): grosor 3px, color naranja brillante (`#fb923c`)
  - Es la misma que se refleja en Panel 1

#### Capa 4: Óptimo teórico `(m*, b*)`

- **Posición**: mapeada al canvas según `m_rango`, `b_rango`
- **Forma**: cruz blanca `+`, grosor 2.5px, tamaño 16px
- **Halo**: círculo exterior semi-transparente pulsante (animación CSS suave)
- **Etiqueta**: `(m*, b*)` con valores, fuente monospace blanca, fondo negro semitransparente

### 3.4 Información textual (debajo del panel)

```
Iteración: N / 2000   |  Trayectorias convergidas: K / M
Primera convergencia: iter X   |  Estado: Entrenando… | Convergido | Detenido
```

---

## 4. PANEL 3 – CURVA J(t)

### 4.1 Dimensiones y layout

- **Ancho**: igual que Panel 2
- **Alto**: `L * 0.38` (suficiente para legibilidad)
- **Posición**: debajo de Panel 2
- **Fondo**: `#111`
- **Borde**: `#333`, 1.5px

### 4.2 Contenido

- **Eje x**: iteración, rango `[0, max_iter_observado + margen]`
- **Eje y**: J, rango dinámico `[J* · 0.9, J_max_inicial · 1.05]`
- **Línea central**: `J_media(t)` — azul (`#60a5fa`), grosor 2px
- **Banda**: `J_media ± σ(t)` — relleno azul translúcido (`rgba(96,165,250,0.15)`)
- **Línea horizontal punteada**: `J*` (mínimo teórico) — amarillo tenue (`#fbbf24`), grosor 1px
- **Etiquetas de ejes**: monospace pequeño blanco
- **Título**: "Función de costo J(t) — media ± σ"

---

## 5. CONTROLES

### 5.1 Panel de controles (columna derecha o franja inferior)

```
── Datos ──────────────────────────────
[Botón] Puntos aleatorios (5–15 pts)
  → siempre activo; pausa animación si estaba corriendo

── Modelo ─────────────────────────────
Mapa de calor (resolución por lado):
  [Slider discr.]  6 · 8 · 10 · 12 · 16   valor: 8
  Solo editable si NO animando

Trayectorias por lado:
  [Slider discr.]  3 · 4 · 5 · 6           valor: 4
  Solo editable si NO animando

Learning rate η:
  [Slider log]  0.001 ─────────────── 0.5
  Valor: 0.050   (mostrar siempre 3 decimales)
  Solo editable si NO animando

── Visualización ─────────────────────
[☑] Dejar rastro     (editable siempre)
Trayectoria en foco: [Input numérico 0-M]

── Entrenamiento ─────────────────────
[Botón Train / Stop]
  Train habilitado: puntos ≥ 3 y no animando
  Stop habilitado: solo si animando

[Botón Reset]
  Limpia trayectorias y rastros, mantiene puntos y parámetros
```

### 5.2 Comportamiento al cambiar parámetros

```
Si cambia resHeatmap, numTrayPorLado o learningRate:
  - Si NO animando: recalcular malla, campo vectorial, reiniciar trayectorias
  - Si animando: IGNORAR con aviso visual suave ("Detén el entrenamiento primero")

Si cambia dejarRastro:
  - Efecto inmediato en el renderizado (solo visual)

Si cambia trajEnFoco:
  - Actualizar resaltado en Panel 2 y recta naranja en Panel 1
```

---

## 6. LÓGICA DE ANIMACIÓN

### 6.1 Paso de simulación (por frame, frameRate ≈ 30fps)

```
Para cada trayectoria[i] NO convergida:
  1. pos_actual = (m, b) = último en historial[i]
  2. gm = (2/n) · Σ((m·xk + b − yk) · xk)    // ∂J/∂m
     gb = (2/n) · Σ(m·xk + b − yk)             // ∂J/∂b
  3. m_new = m − η · gm
     b_new = b − η · gb
  4. J_new = J(m_new, b_new)
  5. Guardar {m: m_new, b: b_new, J: J_new, iter: paso} en historial[i]
  6. Si ||∇J|| < tolerancia:
       trayectorias[i].convergida = true
       trayectorias[i].iterConvergencia = paso
       Calcular y asignar colorFinal retroactivo (ver §3.3 Capa 3)

Actualizar historialJ:
  → media y desviación estándar de J de todas las trayectorias en este paso
```

### 6.2 Condiciones de parada

```
PARADA AUTOMÁTICA: TODAS las trayectorias convergieron
  → animando = false, estado = "Convergido"
  → mostrar resumen: iters por cada trayectoria, J final media

LÍMITE DE SEGURIDAD: paso ≥ maxIteraciones (= 2000)
  → animando = false, estado = "Límite alcanzado"
  → NO es error: mostrar advertencia suave

PARADA MANUAL: usuario presiona Stop
  → animando = false, estado = "Detenido"
  → historial queda intacto, Train permite continuar
```

**Nota sobre divergencia**: con `η` grande, algunas trayectorias pueden divergir
(m o b → ±∞). Clampar posición a `3 × m_rango` / `3 × b_rango` para evitar NaN,
marcar como "divergida" en lugar de "convergida".

### 6.3 Reinicio completo

```
Al presionar Train (estando detenido o convergido):
  - Generar posiciones iniciales: grilla uniforme sobre m_rango × b_rango
  - Reiniciar historial de todas las trayectorias
  - Limpiar historialJ
  - Si dejarRastro estaba CHECKED: limpiar rastros del canvas
  - pasos = 0
  - animando = true
```

**Nota corregida**: los rastros existen solo si `dejarRastro = true`,
por eso se limpian solo en ese caso.

---

## 7. CÁLCULOS MATEMÁTICOS

### 7.1 Solución analítica

```
n  = puntos.length
Sx = Σxᵢ,  Sy = Σyᵢ,  Sxy = Σxᵢyᵢ,  Sx2 = Σxᵢ²

m* = (n·Sxy − Sx·Sy) / (n·Sx2 − Sx²)
b* = (Sy − m*·Sx) / n
J* = (1/n) · Σ(yᵢ − m*·xᵢ − b*)²
```

### 7.2 Gradiente analítico

```
∂J/∂m = (2/n) · Σ((m·xᵢ + b − yᵢ) · xᵢ)
∂J/∂b = (2/n) · Σ(m·xᵢ + b − yᵢ)
||∇J|| = sqrt((∂J/∂m)² + (∂J/∂b)²)
```

### 7.3 Rango dinámico del Panel 2

```javascript
function calcularRango(puntos) {
  const [m_star, b_star] = regresionAnalitica(puntos);
  const delta_m = Math.max(0.8, Math.abs(m_star) * 0.6);
  const delta_b = Math.max(0.6, Math.abs(b_star) * 0.6);
  return {
    m_rango: [m_star - delta_m, m_star + delta_m],
    b_rango: [b_star - delta_b, b_star + delta_b]
  };
}
```

### 7.4 Colormap viridis (aproximación analítica)

```javascript
function viridis(t) {   // t ∈ [0, 1]
  const r = Math.round(68  + t*(59  - 68)  + t*t*(253 - 59  + 68));
  // ... (implementar con tabla de 256 colores o polinomio de grado 3)
}
// Alternativa: tabla de 16 colores interpolados linealmente
const VIRIDIS_16 = [
  [68,1,84], [72,40,120], [62,83,160], [49,104,142],
  [38,130,142],[31,158,137],[53,183,121],[109,205,89],
  [180,222,44],[253,231,37], ...
];
```

### 7.5 Color de trayectoria por convergencia

```javascript
// Calculado AL CONVERGER la trayectoria i (retroactivo a toda su traza)
d_i = sqrt((m_final_i - m_star)² + (b_final_i - b_star)²)
t_i = min(d_i / max(d_j : j todas), 1.0)
HSL(120 - 120·t_i, 90%, 55%)
// t=0 → verde oscuro (cerca del óptimo)
// t=1 → rojo (lejos del óptimo)
```

---

## 8. LAYOUT GENERAL

```
┌─────────────────────────────────────────────────────────┐
│  GradientViz                                     [?]     │
├──────────────────┬──────────────────┬───────────────────┤
│                  │                  │  CONTROLES        │
│   PANEL 1        │   PANEL 2        │                   │
│   Espacio de     │   Espacio (m,b)  │  Datos            │
│   datos          │   Heatmap J      │  [Pts aleatorios] │
│                  │   + gradientes   │                   │
│   L × L          │   + trayect.     │  Modelo           │
│                  │   + óptimo       │  Heatmap: [slider]│
│                  │                  │  Traject: [slider]│
│                  │                  │  η:       [slider]│
│                  │                  │                   │
│                  ├──────────────────┤  Visualización    │
│                  │  PANEL 3         │  [☑] Rastro       │
│                  │  Curva J(t)      │  Foco: [input]    │
│                  │  L × 0.38L       │                   │
│                  │                  │  [  Train / Stop ]│
│                  │                  │  [     Reset     ]│
└──────────────────┴──────────────────┴───────────────────┘
```

*En pantallas < 800px: layout vertical (Panel 1 arriba, Panel 2 al centro, Panel 3 abajo, controles en franja inferior colapsable)*

---

## 9. ETAPAS DE DESARROLLO

### Etapa 0 – Setup
- Canvas p5.js, sistema de coordenadas por panel
- Estado global, funciones de coordenadas (`dataToCanvas`, `paramToCanvas`)
- Función `calcularRango()` y actualización dinámica de ejes

### Etapa 1 – Panel 1 funcional
- Ejes, puntos, drag, límite de 20 puntos
- Calcular `(m*, b*, J*)`, recta punteada gris
- Información textual

### Etapa 2 – Panel 2: heatmap y campo vectorial
- Precalcular malla de J en `resHeatmap × resHeatmap`
- Colormap viridis (tabla o polinomio)
- Dibujar campo 9×9 con flechas normalizadas
- Marcar `(m*, b*)` con cruz pulsante

### Etapa 3 – Trayectorias y animación
- Iniciar en grilla uniforme sobre `m_rango × b_rango`
- Paso de gradiente, criterio de convergencia
- Dibujar trazas frame a frame
- Colorización retroactiva al converger
- Recta naranja en Panel 1 (trayectoria en foco)

### Etapa 4 – Panel 3 y detalles finales
- Curva J_media(t), banda ±σ, línea J*
- Información textual completa en los 3 paneles
- Todos los controles con lógica de habilitado/deshabilitado
- Detección de divergencia y clampeo

---

## 10. NOTAS TÉCNICAS

- **p5.js**: `setup()`, `draw()` a 30fps, `mouseDragged()`, `mousePressed()`, `keyPressed()`
- **Coordenadas**: funciones de mapeo `panelToCanvas(panel, xPanel, yPanel)` para cada transformación
- **Precómputo**: malla de J y campo vectorial solo al cambiar puntos o `resHeatmap`
- **Números**: 3 decimales para m, b; 4 para J; notación científica si < 0.001
- **Framerate**: reducir a 15fps durante animación si > 25 trayectorias (optimización)
- **Sin librería de gráficos**: todo en p5.js nativo para portabilidad pedagógica
