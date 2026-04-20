# PLAN DE DESARROLLO: GradienViz
### Visualizador de Descenso por Gradiente · Regresión Lineal
**Versión**: 1.0 — p5.js standalone  
**Referencia**: GradienVizSpe.md  

---

## Arquitectura del archivo

Un único archivo HTML autocontenido con tres bloques internos:

```
GradienViz.html
├── <head>        Google Fonts + CSS (variables, layout, controles)
├── <body>        Estructura DOM: header, contenedor principal, sidebar
└── <script>
    ├── BLOQUE A: Constantes y paletas de color
    ├── BLOQUE B: Estado global (objeto G)
    ├── BLOQUE C: Matemáticas puras (sin efectos secundarios)
    ├── BLOQUE D: Máquina de estados y lógica de animación
    ├── BLOQUE E: Renderizado por panel (P1, P2, P3)
    ├── BLOQUE F: Controles DOM (sliders, botones, checkboxes)
    └── BLOQUE G: setup() / draw() / eventos p5.js
```

**Sin dependencias externas** salvo:
- `p5.js` vía CDN (unpkg)
- `IBM Plex Mono` + `IBM Plex Sans` vía Google Fonts (o locales)

Usa p5.js en modo global exclusivamente.
Nunca uses prefijos p. ni instancias new p5().
Todas las funciones (fill, stroke, rect, map, constrain...) se llaman directamente.

---

## Estado global — objeto `G`

```javascript
const G = {
  // Datos
  puntos: [],          // [{x, y}], x,y ∈ [0,1]
  m_star: 0,
  b_star: 0,
  J_star: 0,

  // Rango dinámico del Panel 2
  m_lo: 0, m_hi: 1,
  b_lo: 0, b_hi: 1,

  // Parámetros de control
  resHeatmap:     8,    // {6,8,10,12,16}
  numTrayPorLado: 4,    // {3,4,5,6}
  eta:            0.05,
  maxIter:        2000,
  tolerancia:     1e-4,
  dejarRastro:    true,
  trajEnFoco:     0,

  // Estado de animación
  estado: 'idle',       // 'idle' | 'animando' | 'convergido' | 'limite' | 'detenido'
  paso:   0,

  // Trayectorias
  trayectorias: [],     // [{m0,b0, hist:[{m,b,J}], conv,iterConv, colorFinal}]

  // Historial global J(t)
  histJ: [],            // [{iter, media, desv}]

  // Buffers precalculados
  bufHeatmap: null,     // p5.Graphics (Panel 2, Capa 1)
  bufCampo:   null,     // datos del campo vectorial [{cx,cy,dx,dy}]
  bufDirty:   true,     // true = recalcular en próximo draw
};
```

---

## Etapa 0 — Setup, layout y paletas

**Objetivo**: HTML/CSS correcto, canvas p5.js dividido en zonas, paletas definidas.  
**Resultado al final**: pantalla oscura con los tres paneles delimitados (rectángulos vacíos) y el sidebar con todos los controles renderizados (no funcionales).

### 0.1 HTML + CSS

- Layout: `display: grid; grid-template-columns: 1fr 1fr 220px`
- Filas: Panel 1 y Panel 2 comparten altura; Panel 3 debajo de Panel 2 (`height: 160px`)
- Sidebar ocupa las dos filas
- Fuentes: `IBM Plex Mono` (monospace) + `IBM Plex Sans` (sans)
- Variables CSS: `--bg`, `--surface`, `--border`, `--accent`, `--orange`, `--green`, `--yellow`

### 0.2 Canvas y coordenadas

Calcular en `setup()` y en `windowResized()`:

```javascript
// Para cada panel: origen (ox, oy) y dimensiones (cw, ch)
// en píxeles del canvas p5.js
paneles = {
  P1: { ox, oy, cw, ch },   // espacio de datos
  P2: { ox, oy, cw, ch },   // espacio (m, b)
  P3: { ox, oy, cw, ch },   // curva J(t)
};
// Padding interno de cada panel: {l:38, r:14, t:12, b:30}
```

Funciones de mapeo (definir una vez, usar en todos los paneles):

```javascript
// Panel 1: [0,1]² → canvas
function d2c(panel, xd, yd) { ... }   // data to canvas
// Panel 2: [m_lo,m_hi]×[b_lo,b_hi] → canvas
function p2c(panel, m, b)  { ... }   // params to canvas
// Panel 3: iteración×J → canvas
function j2c(panel, iter, j) { ... } // J-history to canvas
```

### 0.3 Paletas de color (constantes)

```javascript
// Viridis: tabla de 16 paradas RGB
const VIRIDIS = [
  [68,1,84],[72,35,116],[64,67,135],[52,94,141],
  [41,120,142],[32,144,140],[34,167,132],[68,190,112],
  [121,209,81],[189,222,38],[253,231,37],[253,231,37],
  ...
];
function viridis(t) { /* interpolación lineal entre paradas */ }

// Trayectorias convergidas: HSL
function convColor(t) { return color(`hsl(${120-120*t},90%,55%)`); }
```

### 0.4 Controles DOM

Crear en JS (no en HTML estático) para poder habilitar/deshabilitar desde el estado:

```javascript
function crearControles() {
  // Botón "Puntos aleatorios"
  // Slider resHeatmap (discreto: 6,8,10,12,16)
  // Slider numTrayPorLado (discreto: 3,4,5,6)
  // Slider eta (logarítmico: 0.001–0.5)
  // Checkbox dejarRastro
  // Input trajEnFoco
  // Botón Train/Stop
  // Botón Reset
}

function actualizarEstadoControles() {
  // Habilitar/deshabilitar según G.estado
  // Llamar cada vez que cambia el estado
}
```

**Checkpoint 0**: tres rectángulos vacíos + sidebar completo renderizado.

---

## Etapa 1 — Panel 1: espacio de datos

**Objetivo**: Panel 1 completamente funcional (puntos, arrastre, regresión, recta).  
**Resultado**: Se pueden agregar/mover/borrar puntos y ver la recta óptima actualizada en tiempo real.

### 1.1 Cálculo de regresión analítica

```javascript
function calcRegresion(puntos) {
  // Retorna {m_star, b_star, J_star}
  // Fórmulas normales: §7.1 de la especificación
  // Si n < 3: retorna null
}
```

### 1.2 Rango dinámico Panel 2

```javascript
function calcRango(m_star, b_star) {
  const dm = Math.max(0.8, Math.abs(m_star) * 0.6);
  const db = Math.max(0.6, Math.abs(b_star) * 0.6);
  return {
    m_lo: m_star - dm, m_hi: m_star + dm,
    b_lo: b_star - db, b_hi: b_star + db
  };
}
```

Llamar siempre que cambian los puntos; resultado va a `G.m_lo`, etc., y marca `G.bufDirty = true`.

### 1.3 Interacciones de ratón en Panel 1

```javascript
function mousePressed() {
  if (G.estado === 'animando') return;
  if (dentroDePanel(P1, mouseX, mouseY)) {
    const punto = encontrarPunto(mouseX, mouseY, umbral=10);
    if (punto) G.puntoDrag = punto;
    else if (G.puntos.length < 20) agregarPunto(mouseX, mouseY);
    recalcularTodo();
  }
}

function mouseDragged() {
  if (G.puntoDrag) {
    moverPunto(G.puntoDrag, mouseX, mouseY);
    recalcularTodo();
  }
}

function keyPressed() {
  if ((key === 'Delete' || key === 'Backspace') && G.puntoSelec) {
    eliminarPunto(G.puntoSelec);
    recalcularTodo();
  }
}
```

`recalcularTodo()`:
```javascript
function recalcularTodo() {
  const res = calcRegresion(G.puntos);
  if (!res) return;
  Object.assign(G, res, calcRango(res.m_star, res.b_star));
  G.bufDirty = true;
  if (G.estado === 'animando') pausarAnimacion();
  limpiarTrayectorias();
}
```

### 1.4 Renderizado Panel 1

```javascript
function dibujarPanel1() {
  // Fondo + borde
  // Ejes [0,1]×[0,1]: ticks cada 0.2, etiquetas monospace
  // Grid interior: líneas cada 0.2, color sutil
  // Recta óptima: gris punteado, siempre visible si n≥3
  // Recta en foco: naranja, solo si animando o convergido
  // Puntos: círculos azules, r=6, borde blanco
  // Punto seleccionado: anillo exterior
  // Footer: "Puntos: N/20 | MSE: X.XXXX | m*=X.XXX b*=X.XXX"
  //         "[→ m=X.XXX b=X.XXX J=X.XXXX]" si animando
}
```

**Checkpoint 1**: Panel 1 interactivo completo, recta se actualiza en tiempo real.

---

## Etapa 2 — Panel 2: espacio de parámetros

**Objetivo**: heatmap J(m,b) + campo vectorial + óptimo marcado.  
**Resultado**: Panel 2 visualmente completo, reactivo a cambios de puntos y parámetros. Sin trayectorias aún.

### 2.1 Heatmap precalculado en `p5.Graphics`

```javascript
function recalcularHeatmap() {
  // Crear o reutilizar G.bufHeatmap = createGraphics(cw, ch)
  const res = G.resHeatmap;
  const celdaW = cw / res, celdaH = ch / res;

  for (let i = 0; i < res; i++) {
    for (let j = 0; j < res; j++) {
      const m = map(i + 0.5, 0, res, G.m_lo, G.m_hi);
      const b = map(j + 0.5, 0, res, G.b_hi, G.b_lo); // eje b invertido
      const jval = calcJ(m, b, G.puntos);
      const t = normalizarJ(jval);   // mapear a [0,1]
      const [r,g,bv] = viridis(t);
      G.bufHeatmap.fill(r, g, bv);
      G.bufHeatmap.noStroke();
      G.bufHeatmap.rect(i*celdaW, j*celdaH, celdaW+1, celdaH+1);
    }
  }
  // Suavizado: drawingContext.imageSmoothingEnabled = true al renderizar
}
```

Normalización de J para el colormap:
```javascript
function normalizarJ(jval) {
  // jval = J(m,b)
  // Calcular j_min ≈ J_star, j_max = percentil 95 de la malla
  // Evitar saturación: t = (jval - j_min) / (j_max - j_min)
  return constrain((jval - G.j_malla_min) / (G.j_malla_rango), 0, 1);
}
```

### 2.2 Campo vectorial precalculado

```javascript
function recalcularCampo() {
  const VRES = 9;
  G.bufCampo = [];
  for (let i = 0; i < VRES; i++) {
    for (let j = 0; j < VRES; j++) {
      const m = map(i+0.5, 0, VRES, G.m_lo, G.m_hi);
      const b = map(j+0.5, 0, VRES, G.b_hi, G.b_lo);
      const {gm, gb} = calcGradiente(m, b, G.puntos);
      G.bufCampo.push({
        cx: p2c(P2, m, b).x,
        cy: p2c(P2, m, b).y,
        gm, gb,
        norm: Math.sqrt(gm*gm + gb*gb)
      });
    }
  }
}
```

Ambos se recalculan en `draw()` solo si `G.bufDirty === true`, luego `G.bufDirty = false`.

### 2.3 Renderizado Panel 2

```javascript
function dibujarPanel2() {
  // Fondo oscuro (#0b0e14)
  // image(G.bufHeatmap, ox+pad.l, oy+pad.t)   // heatmap suavizado
  // Dibujar flechas del campo vectorial
  //   — dirección: (-gm, gb) normalizado a maxLen * factor
  //   — color: rgba(255,255,255,0.3)
  // Trayectorias (Etapa 3 las agrega aquí)
  // Cruz amarilla en (m*, b*)
  //   — con halo pulsante (radio oscilante via sin(frameCount*0.05))
  // Ejes: ticks adaptativos, etiquetas, títulos de ejes
  // Colorbar lateral (opcional: franja de 8px a la derecha)
  // Footer: "iter N/2000 | conv K/M | ..."
}
```

### 2.4 Sliders funcionales

Conectar eventos `oninput` de los tres sliders:

```javascript
sliderHeatmap.oninput = () => {
  if (G.estado === 'animando') { mostrarAviso('Detén primero'); return; }
  G.resHeatmap = [6,8,10,12,16][sliderHeatmap.value];
  G.bufDirty = true;
  limpiarTrayectorias();
};
// Análogo para numTrayPorLado y eta
```

**Checkpoint 2**: Panel 2 muestra heatmap y campo vectorial. Cambiar puntos o parámetros actualiza automáticamente.

---

## Etapa 3 — Trayectorias, animación y Panel 3

**Objetivo**: implementar el descenso por gradiente animado y la curva J(t).  
**Resultado**: GradienViz completamente funcional.

### 3.1 Inicialización de trayectorias

```javascript
function inicializarTrayectorias() {
  G.trayectorias = [];
  const N = G.numTrayPorLado;
  for (let i = 0; i < N; i++) {
    for (let j = 0; j < N; j++) {
      const m0 = map(i, 0, N-1, G.m_lo + 0.05*(G.m_hi-G.m_lo),
                                  G.m_hi - 0.05*(G.m_hi-G.m_lo));
      const b0 = map(j, 0, N-1, G.b_lo + 0.05*(G.b_hi-G.b_lo),
                                  G.b_hi - 0.05*(G.b_hi-G.b_lo));
      G.trayectorias.push({
        m0, b0,
        hist: [{m: m0, b: b0, J: calcJ(m0, b0, G.puntos)}],
        conv: false, iterConv: null,
        div:  false,           // divergida
        colorFinal: null       // asignado al converger
      });
    }
  }
  G.histJ = [];
  G.paso  = 0;
}
```

### 3.2 Paso de simulación

```javascript
function pasoSimulacion() {
  if (G.estado !== 'animando') return;

  let todasConv = true;

  G.trayectorias.forEach((tr, i) => {
    if (tr.conv || tr.div) return;
    todasConv = false;

    const {m, b} = tr.hist[tr.hist.length - 1];
    const {gm, gb} = calcGradiente(m, b, G.puntos);
    const m_new = m - G.eta * gm;
    const b_new = b - G.eta * gb;

    // Detección de divergencia
    const fuera = 3;
    if (m_new < G.m_lo - fuera*(G.m_hi-G.m_lo) ||
        m_new > G.m_hi + fuera*(G.m_hi-G.m_lo)) {
      tr.div = true; return;
    }

    const J_new = calcJ(m_new, b_new, G.puntos);
    tr.hist.push({m: m_new, b: b_new, J: J_new});

    const norm = Math.sqrt(gm*gm + gb*gb);
    if (norm < G.tolerancia) {
      tr.conv = true;
      tr.iterConv = G.paso;
      asignarColorConvergencia(i);  // retroactivo
    }
  });

  // Actualizar histJ
  const vals = G.trayectorias.map(tr => tr.hist[tr.hist.length-1].J);
  const media = vals.reduce((a,b) => a+b, 0) / vals.length;
  const desv  = Math.sqrt(vals.reduce((a,v) => a+(v-media)**2, 0) / vals.length);
  G.histJ.push({iter: G.paso, media, desv});

  G.paso++;

  // Condiciones de parada
  if (todasConv) { G.estado = 'convergido'; actualizarEstadoControles(); }
  if (G.paso >= G.maxIter) { G.estado = 'limite'; actualizarEstadoControles(); }
}
```

### 3.3 Colorización retroactiva

```javascript
function asignarColorConvergencia(idx) {
  // Calcular distancias de todas las trayectorias convergidas hasta ahora
  const dists = G.trayectorias
    .filter(tr => tr.conv)
    .map(tr => {
      const {m, b} = tr.hist[tr.hist.length-1];
      return Math.sqrt((m - G.m_star)**2 + (b - G.b_star)**2);
    });
  const d_max = Math.max(...dists, 0.001);

  // Reasignar colores a todas las convergidas
  G.trayectorias.forEach(tr => {
    if (!tr.conv) return;
    const {m, b} = tr.hist[tr.hist.length-1];
    const d = Math.sqrt((m - G.m_star)**2 + (b - G.b_star)**2);
    const t = Math.min(d / d_max, 1);
    tr.colorFinal = color(`hsl(${120 - 120*t}, 90%, 55%)`);
  });
}
```

### 3.4 Renderizado de trayectorias en Panel 2

```javascript
function dibujarTrayectorias() {
  G.trayectorias.forEach((tr, i) => {
    if (tr.hist.length < 2) return;

    // Color: final si convergida, blanco semitransparente si no
    const c = tr.conv ? tr.colorFinal
            : tr.div  ? color(255, 60, 60, 80)
            : (i === G.trajEnFoco ? color('#fb923c')
                                  : color(255, 255, 255, 140));
    const grosor = (i === G.trajEnFoco) ? 3 : 1.5;

    stroke(c); strokeWeight(grosor); noFill();

    if (G.dejarRastro) {
      // Dibujar toda la traza
      beginShape();
      tr.hist.forEach(({m,b}) => {
        const {x,y} = p2c(P2, m, b);
        vertex(x, y);
      });
      endShape();
    } else {
      // Solo el segmento del último paso
      if (tr.hist.length >= 2) {
        const prev = p2c(P2, tr.hist[tr.hist.length-2].m, tr.hist[tr.hist.length-2].b);
        const cur  = p2c(P2, tr.hist[tr.hist.length-1].m, tr.hist[tr.hist.length-1].b);
        line(prev.x, prev.y, cur.x, cur.y);
      }
    }

    // Punto actual
    const {m,b} = tr.hist[tr.hist.length-1];
    const {x,y} = p2c(P2, m, b);
    fill(c); noStroke();
    circle(x, y, i === G.trajEnFoco ? 8 : 5);
  });
}
```

### 3.5 Panel 3: curva J(t)

```javascript
function dibujarPanel3() {
  // Fondo oscuro
  // Si histJ.length < 2: mostrar "Presiona Train para comenzar"
  // Rango eje x: [0, max(histJ.length, 20)]
  // Rango eje y: [J_star*0.85, histJ[0].media * 1.1]

  // Línea J* punteada amarilla
  // Banda ±σ (fillBetween con dos arrays de puntos)
  // Línea media azul
  // Punto naranja en la iteración actual
  // Línea vertical naranja en iter actual
  // Ticks y etiquetas (monospace, tamaño 9)
}
```

### 3.6 Botones Train / Stop / Reset

```javascript
btnTrain.onclick = () => {
  if (G.estado === 'animando') return;
  if (G.puntos.length < 3) { mostrarAviso('Necesitas al menos 3 puntos'); return; }
  inicializarTrayectorias();
  G.estado = 'animando';
  actualizarEstadoControles();
};

btnStop.onclick = () => {
  if (G.estado !== 'animando') return;
  G.estado = 'detenido';
  actualizarEstadoControles();
};

btnReset.onclick = () => {
  G.estado = 'idle';
  G.trayectorias = [];
  G.histJ = [];
  G.paso  = 0;
  G.bufDirty = true;
  actualizarEstadoControles();
};
```

**Checkpoint 3**: GradienViz completamente funcional. Todos los experimentos pedagógicos de §8.2 de la especificación son reproducibles.

---

## Loop principal `draw()`

```javascript
function draw() {
  background(BG_COLOR);

  // Recalcular buffers si necesario (una sola vez por cambio)
  if (G.bufDirty && G.puntos.length >= 3) {
    recalcularHeatmap();
    recalcularCampo();
    G.bufDirty = false;
  }

  // Paso de simulación (solo si animando)
  if (G.estado === 'animando') pasoSimulacion();

  // Renderizado
  dibujarPanel1();
  dibujarPanel2();   // incluye heatmap, campo, trayectorias, óptimo
  dibujarPanel3();

  // Separadores visuales entre paneles
  dibujarSeparadores();
}
```

---

## Funciones matemáticas puras

Todas sin efectos secundarios; reciben los datos como parámetros:

```javascript
// Costo MSE
function calcJ(m, b, puntos) {
  if (puntos.length === 0) return 0;
  return puntos.reduce((sum, {x,y}) => sum + (m*x + b - y)**2, 0) / puntos.length;
}

// Gradiente analítico
function calcGradiente(m, b, puntos) {
  const n = puntos.length;
  let gm = 0, gb = 0;
  puntos.forEach(({x,y}) => {
    const r = m*x + b - y;
    gm += r * x;
    gb += r;
  });
  return { gm: 2*gm/n, gb: 2*gb/n };
}

// Regresión analítica
function calcRegresion(puntos) {
  const n = puntos.length;
  if (n < 3) return null;
  let Sx=0, Sy=0, Sxy=0, Sx2=0;
  puntos.forEach(({x,y}) => { Sx+=x; Sy+=y; Sxy+=x*y; Sx2+=x*x; });
  const denom = n*Sx2 - Sx*Sx;
  if (Math.abs(denom) < 1e-10) return null;
  const m_star = (n*Sxy - Sx*Sy) / denom;
  const b_star = (Sy - m_star*Sx) / n;
  const J_star = calcJ(m_star, b_star, puntos);
  return {m_star, b_star, J_star};
}
```

---

## Verificación pedagógica final

Antes de considerar completado el proyecto, reproducir estos experimentos:

| Experimento | Parámetro clave | Resultado esperado |
|---|---|---|
| η muy pequeño (0.001) | slider → 0.001 | Curva J(t) cae muy lento, trayectorias avanzan despacio |
| η grande (0.3–0.4) | slider → 0.4 | Trayectorias oscilan o divergen, J(t) no cae monotónamente |
| Punto outlier | Drag punto lejos | `(m*, b*)` cambia, contorno se deforma |
| 3 trayectorias/lado | numTray → 3 | 9 trayectorias, convergencia observable sin saturación |
| 6 trayectorias/lado | numTray → 6 | 36 trayectorias, ver colorización verde→rojo completa |
| Sin rastro | dejarRastro = false | Solo puntos móviles, útil para η grande |
| Datos colineales | Puntos en línea recta | `J*≈0`, contorno muy elongado en dirección de m |

---

## Resumen de etapas

| Etapa | Nombre | Entregable verificable |
|---|---|---|
| **0** | Setup, layout, paletas | 3 paneles delimitados + sidebar completo |
| **1** | Panel 1 interactivo | Puntos arrastrables, recta óptima en tiempo real |
| **2** | Panel 2: heatmap + campo | Mapa de calor + flechas, reactivo a cambios |
| **3** | Animación + Panel 3 | Train/Stop/Reset, trayectorias, J(t) |

---

*GradienViz — Plan v1.0 · Miguel Ángel Norzagaray · DASC-UABCS*