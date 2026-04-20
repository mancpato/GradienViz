# GradienViz — Estado de desarrollo

*Referencia: GradienVizSpec.md · GradienVizPlan.md*  
*Archivo activo: GradienViz.html*

---

## Resumen de etapas

| Etapa | Nombre | Estado |
|---|---|---|
| **0** | Setup, layout, infraestructura | ✅ Completa |
| **1** | Panel 1 interactivo | ⚠️ Casi completa (falta recta foco, Etapa 3) |
| **2** | Panel 2: heatmap + campo vectorial | ⚠️ Parcial (falta campo ∇J, halo, labels ejes) |
| **3** | Animación, trayectorias, Panel 3 | ❌ Pendiente |

---

## Etapa 0 — Setup ✅ Completa

- HTML/CSS: grid layout `1fr 220px`, `height: calc(100vh - 36px)` ✅
- Canvas p5.js modo global, `calcularPaneles()`, `windowResized()` ✅
- Estado global `G` con todos los campos del plan ✅
- Funciones de mapeo: `d2c()`, `p2c()`, `j2c()` (stub) ✅
- Paletas: `VIRIDIS`, `viridis()`, `convColor()` ✅
- Sistema tipográfico: `FS=14`, `PAD={l:52, r:16, t:18, b:60}` ✅
- Controles DOM: sliders, botones, checkbox — creados y conectados ✅
- Helpers de layout: `dentroDePanel()`, `encontrarPunto()`, `canvasToData()`, `agregarPunto()` ✅
- Función base: `dibujarPanel()` con marco y título centrado en PAD.t ✅

---

## Etapa 1 — Panel 1 ⚠️ Parcial

### Implementado

- `calcRegresion()`, `calcRango()`, `calcJ_mb()`, `recalcularTodo()` ✅
- `dibujarEjes()` reutilizable con ticks y etiquetas en `FS` ✅
- `dibujarPanel1()`: ejes [0,1]×[0,1], 5 ticks por eje ✅
- Recta óptima gris punteada (`drawingContext.setLineDash`, `stroke(150,150,155)`) ✅
- Puntos azules `#2563eb` con borde blanco 1.5px, `∅=12px` ✅
- Grid interior sutil cada 0.2 (`dibujarGridP1`, dash 2/4) ✅
- `mousePressed()`: agregar punto (máx 20), iniciar drag, `Ctrl+Click` borrar ✅
- `mouseDragged()`: mover punto, `recalcularTodo()` en tiempo real ✅
- `mouseReleased()`: liberar drag ✅
- Footer informativo: `N/20 pts | MSE | m* | b*` (dos líneas, solo con ≥3 puntos) ✅

### Pendiente (según spec §2)

- [ ] Recta naranja de trayectoria en foco (se implementa en Etapa 3)

---

## Etapa 2 — Panel 2 ⚠️ Parcial

### Implementado

- `construirHeatmap()`: grilla N×N, doble pasada (calc J → normalizar → viridis) ✅
- Cache con `G.bufDirty` — solo recalcula cuando cambian los datos ✅
- `dibujarHeatmap()`: rectángulos coloreados, corrección de inversión Y ✅
- `dibujarPanel2()`: heatmap + ejes + óptimo, reactivo a cambios de puntos ✅
- `dibujarEjes()` sobre el heatmap con rangos dinámicos `m_lo/hi`, `b_lo/hi` ✅
- `dibujarPuntoOptimo_P2()`: cruz amarilla + círculo en `(m*, b*)` ✅

### Pendiente (según spec §3.2–3.4)

- [ ] `calcGradiente(m, b)` — función pura `∂J/∂m`, `∂J/∂b` (necesaria aquí y en Etapa 3)
- [ ] Campo vectorial ∇J: grilla 9×9, flechas antiparalelas al gradiente, `G.bufCampo`
- [ ] Halo pulsante en `(m*, b*)` — radio oscilante con `sin(frameCount * 0.05)`
- [ ] Labels de ejes: "m" bajo eje X, "b" rotado en eje Y
- [ ] Footer informativo (se activa en Etapa 3): `"iter N/2000 | conv K/M | estado"`

---

## Etapa 3 — Animación y Panel 3 ❌ Pendiente

Todo pendiente. Puntos de entrada en el código:

- Botón Train: `() => { /* Etapa 3 */ }` (línea ~475 del script)
- `G.trayectorias = []`, `G.histJ = []`, `G.paso = 0` — campos listos en G
- `G.estado`: maneja `'idle' | 'animando' | 'convergido' | 'limite' | 'detenido'`
- `actualizarEstadoControles()` ya deshabilita/habilita controles según estado

### Por implementar (plan §3)

- [ ] `calcGradiente(m, b)` — gradiente analítico `∂J/∂m`, `∂J/∂b`
- [ ] `inicializarTrayectorias()` — grilla uniforme sobre `m_rango × b_rango`
- [ ] `pasoSimulacion()` — un paso GD por frame, criterio convergencia + divergencia
- [ ] `asignarColorConvergencia(idx)` — colorización retroactiva HSL
- [ ] `dibujarTrayectorias()` en Panel 2 (rastro completo o solo último segmento)
- [ ] Recta naranja en Panel 1 (trayectoria `G.trajEnFoco`)
- [ ] `dibujarPanel3()` con curva J_media(t), banda ±σ, línea J* punteada
- [ ] `j2c()` correctamente escalado (actualmente stub)
- [ ] Train/Stop/Reset conectados a `inicializarTrayectorias()` y `G.estado`
- [ ] Detección de divergencia: clampeo a `3 × rango`, marcar `tr.div = true`
- [ ] Parada automática cuando todas convergen o `paso >= maxIter`

---

## Notas de implementación acumuladas

| Tema | Decisión tomada |
|---|---|
| Tipografía | `FS=14` base única y mínimo; títulos `FS*1.2`; nunca usar submúltiplos |
| Heatmap buffer | Array de objetos `{J, col}`, no `p5.Graphics` — suficiente para N≤16 |
| Borrar puntos | `Ctrl+Click` (también `Shift+Click`) vía `keyIsDown(CONTROL)` |
| Límite de puntos | 20 máximo, 3 mínimo para entrenar |
| Archivo de referencia | `GradientViz_prototipo.html` — solo visual, nunca editar |
| Modo p5.js | Global exclusivamente — sin `this.`, sin `new p5()` |
| Líneas punteadas | `drawingContext.setLineDash([d,g])` + reset con `setLineDash([])` — no olvidar el reset |
