---
layout: post
title: Spherical Fibonacci Grids
date: 2026-03-25 11:12:00-0400
description: An interactive explanation of Spherical Fibonacci Grids and its properties.
tags:
categories: [interactive]
related_posts: false
---

Distributed under the terms of the [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/) License.

The main problem in *Physically-Based Ray Tracing* is converging the value of the *illumination integral* that appears in the *Light Transport Equation* (LTE). Integral equations generally do not have an analytic solution, so numerical integration techniques are used. The most common approach is *Monte Carlo integration* (MC), which is based on randomization:

$$L_o(\mathbf{p}, \omega_o) = L_e(\mathbf{p}, \omega_o) + \int_{S^2} f(\mathbf{p}, \omega_o, \omega_i)\, L_i(\mathbf{p}, \omega_i)\, |\cos\theta_i|\, d\omega_i$$

Further improvements focus on manipulating the random sampling to minimize convergence time — that is, distributing the samples so that we obtain the minimum error with the minimum number of samples. These methods fall under the category of *Quasi-Monte Carlo integration* (QMC).

In this context, we find utility in low-discrepancy distributions such as Fibonacci-based spherical distributions. The main strength of these point sets is an extremely uniform distribution which is near-optimal in terms of *spherical cap discrepancy*. There are two families of such point sets:

- **(i) SFIL** — Spherical Fibonacci point sets based on planar Fibonacci integration lattices.
- **(ii) SFG** — Spherical Fibonacci point sets based on planar Fibonacci grids.

SFILs constrain point set sizes to be Fibonacci numbers. SFGs, on the other hand, allow generating point sets with an arbitrary number of points — which is strongly important in Physically-Based Rendering. For this reason, we focus on SFGs.

---

## Theoretical Background

### Golden Ratio

In mathematics, two quantities are in the golden ratio if their ratio is the same as the ratio of their sum to the larger of the two quantities:

$$\frac{a+b}{a} = \frac{a}{b} = \Phi \qquad \text{such that} \quad a > b > 0$$

$$\Phi$$ (also called the **extreme and mean ratio**, or **divine proportion**) satisfies a quadratic equation and is an irrational number. Most importantly, it is *super-irrational* — some people call it the most irrational number. This property makes it extremely useful when distributing samples without generating repetition patterns, therefore implying a low discrepancy.

$$\Phi = \frac{1 + \sqrt{5}}{2} = 1.618\ldots$$

### The Golden Angle

The golden ratio gives rise to the **golden angle** $$\alpha$$, defined as the smaller of the two angles created by dividing a full circle in the golden ratio:

$$\alpha = 2\pi \left(1 - \frac{1}{\Phi}\right) = 2\pi\,\frac{2}{1+\sqrt{5}} \approx 137.508°$$

Because $$\Phi$$ is the hardest number to approximate by rationals, successive points placed at intervals of $$\alpha$$ never nearly coincide — they fill space as uniformly as possible.

---

The interactive viewer below lets you explore how these distributions behave. Toggle between the **planar** (2D Fibonacci lattice) and **spherical** (3D SFG) projections, and observe the effect of changing the number of samples.

Try adjusting the divergence angle slightly away from $$137.508°$$ — the perfectly uniform packing collapses into coarse radial spokes, illustrating why the golden angle is uniquely optimal.

<style>
  .sfg-wrap { margin: 2rem 0; }
  .sfg-tabs {
    display: flex;
    gap: 0;
    margin-bottom: 0;
    border-bottom: 1px solid var(--global-divider-color, #ddd);
  }
  .sfg-tab {
    padding: 0.45rem 1.2rem;
    font-size: 0.85rem;
    font-weight: 600;
    cursor: pointer;
    border: 1px solid transparent;
    border-bottom: none;
    border-radius: 6px 6px 0 0;
    background: transparent;
    color: var(--global-text-color-light, #888);
    transition: background 0.15s, color 0.15s;
  }
  .sfg-tab.active {
    background: var(--global-bg-color, white);
    border-color: var(--global-divider-color, #ddd);
    color: var(--global-theme-color, #6c63ff);
    margin-bottom: -1px;
  }
  .sfg-panel {
    padding: 1.2rem;
    border: 1px solid var(--global-divider-color, #ddd);
    border-top: none;
    border-radius: 0 0 8px 8px;
    background: var(--global-bg-color, white);
  }
  .sfg-controls {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem 1.5rem;
    margin-bottom: 1.2rem;
    padding: 0.8rem 1rem;
    background: var(--global-code-bg-color, #f8f8f8);
    border-radius: 6px;
    align-items: flex-end;
  }
  .sfg-ctrl { display: flex; flex-direction: column; gap: 3px; min-width: 140px; }
  .sfg-ctrl label { font-size: 0.75rem; font-weight: 600; text-transform: uppercase; letter-spacing: 0.05em; color: var(--global-text-color-light, #888); }
  .sfg-ctrl .sfg-val { font-size: 0.88rem; font-weight: 600; color: var(--global-theme-color, #6c63ff); }
  .sfg-ctrl input[type=range] { accent-color: var(--global-theme-color, #6c63ff); width: 100%; cursor: pointer; }
  .sfg-ctrl select { font-size: 0.85rem; padding: 0.25rem 0.4rem; border-radius: 5px; border: 1px solid var(--global-divider-color, #ccc); background: var(--global-bg-color, white); color: var(--global-text-color, #333); cursor: pointer; }
  .sfg-canvas-wrap { display: flex; justify-content: center; }
  canvas.sfg-canvas { border-radius: 50%; border: 1px solid var(--global-divider-color, #eee); display: block; }
  .sfg-caption { text-align: center; font-size: 0.8rem; color: var(--global-text-color-light, #888); margin-top: 0.6rem; font-style: italic; }
  .sfg-reset { padding: 0.3rem 0.8rem; font-size: 0.82rem; border-radius: 5px; border: 1px solid var(--global-theme-color, #6c63ff); color: var(--global-theme-color, #6c63ff); background: transparent; cursor: pointer; transition: 0.15s; align-self: flex-end; }
  .sfg-reset:hover { background: var(--global-theme-color, #6c63ff); color: white; }
</style>

<div class="sfg-wrap">
  <div class="sfg-tabs">
    <button class="sfg-tab active" data-tab="planar">Planar lattice (2D)</button>
    <button class="sfg-tab" data-tab="sphere">Spherical SFG (3D)</button>
  </div>

  <!-- PLANAR PANEL -->
  <div class="sfg-panel" id="tab-planar">
    <div class="sfg-controls">
      <div class="sfg-ctrl">
        <label>Points</label>
        <span class="sfg-val" id="p-n-val">600</span>
        <input type="range" id="p-n-sl" min="50" max="2000" step="50" value="600">
      </div>
      <div class="sfg-ctrl">
        <label>Angle (°)</label>
        <span class="sfg-val" id="p-a-val">137.508°</span>
        <input type="range" id="p-a-sl" min="130" max="145" step="0.001" value="137.508">
      </div>
      <div class="sfg-ctrl">
        <label>Point size</label>
        <span class="sfg-val" id="p-s-val">3.5</span>
        <input type="range" id="p-s-sl" min="1" max="8" step="0.5" value="3.5">
      </div>
      <div class="sfg-ctrl">
        <label>Color</label>
        <select id="p-c-sel">
          <option value="spiral">Spiral</option>
          <option value="radius">Radius</option>
          <option value="index">Index</option>
          <option value="solid">Solid</option>
        </select>
      </div>
      <button class="sfg-reset" id="p-rst">↺ Reset</button>
    </div>
    <div class="sfg-canvas-wrap"><canvas class="sfg-canvas" id="p-canvas"></canvas></div>
    <p class="sfg-caption" id="p-cap">Golden angle (137.508°) — 600 points</p>
  </div>

  <!-- SPHERE PANEL -->
  <div class="sfg-panel" id="tab-sphere" style="display:none;">
    <div class="sfg-controls">
      <div class="sfg-ctrl">
        <label>Points</label>
        <span class="sfg-val" id="s-n-val">500</span>
        <input type="range" id="s-n-sl" min="50" max="1500" step="50" value="500">
      </div>
      <div class="sfg-ctrl">
        <label>Point size</label>
        <span class="sfg-val" id="s-s-val">3</span>
        <input type="range" id="s-s-sl" min="1" max="7" step="0.5" value="3">
      </div>
      <div class="sfg-ctrl">
        <label>Projection</label>
        <select id="s-proj">
          <option value="ortho">Orthographic</option>
          <option value="stereo">Stereographic</option>
        </select>
      </div>
      <div class="sfg-ctrl">
        <label>Hemisphere</label>
        <select id="s-hemi">
          <option value="full">Full sphere</option>
          <option value="upper">Upper only</option>
        </select>
      </div>
      <button class="sfg-reset" id="s-rst">↺ Reset</button>
    </div>
    <div class="sfg-canvas-wrap"><canvas class="sfg-canvas" id="s-canvas"></canvas></div>
    <p class="sfg-caption" id="s-cap">SFG — 500 points uniformly distributed on S²</p>
  </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/d3@7/dist/d3.min.js"></script>
<script>
(function () {
  const GOLDEN = 137.50776405003785;
  const PHI = (1 + Math.sqrt(5)) / 2;

  // ---- Color helpers ----
  function hslSpiral(t) { return `hsl(${(t*360).toFixed(1)},72%,52%)`; }
  function viridis(t) { return `rgb(${Math.round(68+t*187)},${Math.round(1+t*210)},${Math.round(84+(1-t)*171)})`; }
  function plasma(t)  { return `rgb(${Math.round(13+t*237)},${Math.round(8+t*105)},${Math.round(135-t*10)})`; }
  function getColor(i, n, angle, mode) {
    if (mode === 'solid')  return '#7F77DD';
    if (mode === 'radius') return viridis(Math.sqrt(i/n));
    if (mode === 'index')  return plasma(i/n);
    return hslSpiral(((i * angle) % 360) / 360);
  }

  // ---- Resize canvas ----
  function initCanvas(id) {
    const c = document.getElementById(id);
    const w = Math.min(500, c.parentElement.parentElement.clientWidth - 48);
    c.width = w; c.height = w;
    return c;
  }

  // ===================== PLANAR =====================
  const PDEF = { n: 600, angle: GOLDEN, size: 3.5, color: 'spiral' };
  let pState = { ...PDEF };
  const pCanvas = initCanvas('p-canvas');
  const pCtx = pCanvas.getContext('2d');

  function drawPlanar() {
    const { n, angle, size, color } = pState;
    const W = pCanvas.width, cx = W/2, cy = W/2, maxR = W * 0.46;
    const aRad = angle * Math.PI / 180;
    pCtx.clearRect(0, 0, W, W);
    pCtx.save();
    pCtx.beginPath(); pCtx.arc(cx, cy, maxR, 0, Math.PI*2); pCtx.clip();
    for (let i = 0; i < n; i++) {
      const r = maxR * Math.sqrt((i + 0.5) / n);
      const a = i * aRad;
      pCtx.beginPath();
      pCtx.arc(cx + r*Math.cos(a), cy + r*Math.sin(a), size, 0, Math.PI*2);
      pCtx.fillStyle = getColor(i, n, angle, color);
      pCtx.globalAlpha = 0.88;
      pCtx.fill();
    }
    pCtx.restore();
    const isGolden = Math.abs(angle - GOLDEN) < 0.01;
    document.getElementById('p-cap').textContent = isGolden
      ? `Golden angle (${angle.toFixed(3)}°) — ${n} points with maximum packing efficiency`
      : `Angle: ${angle.toFixed(3)}° — ${n} points  (golden = ${GOLDEN.toFixed(3)}°)`;
  }

  document.getElementById('p-n-sl').addEventListener('input', e => { pState.n = +e.target.value; document.getElementById('p-n-val').textContent = pState.n; drawPlanar(); });
  document.getElementById('p-a-sl').addEventListener('input', e => { pState.angle = +e.target.value; document.getElementById('p-a-val').textContent = pState.angle.toFixed(3)+'°'; drawPlanar(); });
  document.getElementById('p-s-sl').addEventListener('input', e => { pState.size = +e.target.value; document.getElementById('p-s-val').textContent = pState.size; drawPlanar(); });
  document.getElementById('p-c-sel').addEventListener('change', e => { pState.color = e.target.value; drawPlanar(); });
  document.getElementById('p-rst').addEventListener('click', () => {
    pState = { ...PDEF };
    document.getElementById('p-n-sl').value = pState.n; document.getElementById('p-a-sl').value = pState.angle;
    document.getElementById('p-s-sl').value = pState.size; document.getElementById('p-c-sel').value = pState.color;
    document.getElementById('p-n-val').textContent = pState.n; document.getElementById('p-a-val').textContent = pState.angle.toFixed(3)+'°';
    document.getElementById('p-s-val').textContent = pState.size; drawPlanar();
  });

  // ===================== SPHERICAL SFG =====================
  const SDEF = { n: 500, size: 3, proj: 'ortho', hemi: 'full' };
  let sState = { ...SDEF };
  let sAngle = 0; // auto-rotate angle
  let sRAF = null;

  const sCanvas = initCanvas('s-canvas');
  const sCtx = sCanvas.getContext('2d');

  // Generate SFG points on S²
  // Using the Spherical Fibonacci Grid formula:
  // theta_i = arccos(1 - 2(i+0.5)/n),  phi_i = 2*pi*i/PHI
  function sfgPoints(n, hemi) {
    const pts = [];
    const count = hemi === 'upper' ? n * 2 : n; // generate more, filter half
    for (let i = 0; i < (hemi === 'upper' ? n : n); i++) {
      const idx = hemi === 'upper' ? i + n/2 : i; // shift to upper half
      const cosTheta = hemi === 'upper'
        ? 1 - (i + 0.5) / n           // [0,1] -> upper hemisphere
        : 1 - 2*(i + 0.5) / n;        // [-1,1] -> full sphere
      const sinTheta = Math.sqrt(Math.max(0, 1 - cosTheta*cosTheta));
      const phi = 2 * Math.PI * i / PHI;
      pts.push({
        x: sinTheta * Math.cos(phi),
        y: sinTheta * Math.sin(phi),
        z: cosTheta,
        t: i / n
      });
    }
    return pts;
  }

  function projectOrtho(p, rotY, W) {
    const cx = W/2, cy = W/2, R = W * 0.44;
    const cosY = Math.cos(rotY), sinY = Math.sin(rotY);
    const rx = p.x * cosY + p.z * sinY;
    const ry = p.y;
    const rz = -p.x * sinY + p.z * cosY;
    return { px: cx + rx * R, py: cy - ry * R, depth: rz };
  }

  function projectStereo(p, rotY, W) {
    const cx = W/2, cy = W/2, R = W * 0.44;
    const cosY = Math.cos(rotY), sinY = Math.sin(rotY);
    const rx = p.x * cosY + p.z * sinY;
    const ry = p.y;
    const rz = -p.x * sinY + p.z * cosY;
    const denom = 1 - rz * 0.9 + 0.001;
    return { px: cx + (rx / denom) * R, py: cy - (ry / denom) * R, depth: rz };
  }

  function drawSphere() {
    const { n, size, proj, hemi } = sState;
    const W = sCanvas.width, cx = W/2, cy = W/2, R = W*0.44;
    sCtx.clearRect(0, 0, W, W);

    // sphere outline
    sCtx.beginPath(); sCtx.arc(cx, cy, R, 0, Math.PI*2);
    sCtx.strokeStyle = 'rgba(150,150,150,0.3)'; sCtx.lineWidth = 1; sCtx.stroke();

    // equator hint
    sCtx.beginPath();
    sCtx.ellipse(cx, cy, R, R * Math.abs(Math.sin(sAngle + Math.PI/2)) * 0.15 + R*0.08, 0, 0, Math.PI*2);
    sCtx.strokeStyle = 'rgba(150,150,150,0.12)'; sCtx.lineWidth = 0.5; sCtx.stroke();

    const pts = sfgPoints(n, hemi);
    const projected = pts.map((p, i) => {
      const pp = proj === 'stereo' ? projectStereo(p, sAngle, W) : projectOrtho(p, sAngle, W);
      return { ...pp, t: p.t, i };
    });
    projected.sort((a, b) => a.depth - b.depth);

    projected.forEach(p => {
      const alpha = proj === 'ortho' ? (p.depth < 0 ? 0.18 : 0.88) : 0.88;
      const r = size * (proj === 'stereo' ? 1 : (0.7 + 0.3 * (p.depth + 1) / 2));
      sCtx.beginPath();
      sCtx.arc(p.px, p.py, r, 0, Math.PI*2);
      sCtx.fillStyle = hslSpiral(p.t);
      sCtx.globalAlpha = alpha;
      sCtx.fill();
    });
    sCtx.globalAlpha = 1;
    document.getElementById('s-cap').textContent =
      `SFG — ${n} points on ${hemi === 'upper' ? 'upper hemisphere' : 'S²'} (${proj === 'ortho' ? 'orthographic' : 'stereographic'} projection)`;

    sAngle += 0.004;
    sRAF = requestAnimationFrame(drawSphere);
  }

  document.getElementById('s-n-sl').addEventListener('input', e => { sState.n = +e.target.value; document.getElementById('s-n-val').textContent = sState.n; });
  document.getElementById('s-s-sl').addEventListener('input', e => { sState.size = +e.target.value; document.getElementById('s-s-val').textContent = sState.size; });
  document.getElementById('s-proj').addEventListener('change', e => { sState.proj = e.target.value; });
  document.getElementById('s-hemi').addEventListener('change', e => { sState.hemi = e.target.value; });
  document.getElementById('s-rst').addEventListener('click', () => {
    sState = { ...SDEF };
    document.getElementById('s-n-sl').value = sState.n; document.getElementById('s-s-sl').value = sState.size;
    document.getElementById('s-proj').value = sState.proj; document.getElementById('s-hemi').value = sState.hemi;
    document.getElementById('s-n-val').textContent = sState.n; document.getElementById('s-s-val').textContent = sState.size;
  });

  // ---- Tab switching ----
  document.querySelectorAll('.sfg-tab').forEach(btn => {
    btn.addEventListener('click', () => {
      document.querySelectorAll('.sfg-tab').forEach(b => b.classList.remove('active'));
      btn.classList.add('active');
      const tab = btn.dataset.tab;
      document.getElementById('tab-planar').style.display = tab === 'planar' ? '' : 'none';
      document.getElementById('tab-sphere').style.display = tab === 'sphere' ? '' : 'none';
      if (tab === 'sphere') {
        if (!sRAF) drawSphere();
      } else {
        if (sRAF) { cancelAnimationFrame(sRAF); sRAF = null; }
      }
    });
  });

  // Init
  drawPlanar();
})();
</script>

---

### Fibonacci Theory

In mathematics, the Fibonacci sequence is defined such that each element is the sum of the two preceding ones:

$$F_m = F_{m-1} + F_{m-2} \quad \text{for } m > 1, \qquad \text{where } F_0 = 0 \text{ and } F_1 = 1$$

The sequence begins: $$0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, \ldots$$

Fibonacci numbers appear unexpectedly often in mathematics, and are applied in areas as diverse as computer algorithms and biological settings (phyllotaxis, shell spirals, branching patterns).

Fibonacci numbers are also strongly related to the golden ratio. As the sequence progresses, the ratio of successive Fibonacci numbers approaches a specific irrational number, the golden ratio $$\Phi$$:

$$\lim_{m \to \infty} \frac{F_{m+1}}{F_m} = \Phi \approx 1.618\ldots$$

Due to this relation, Fibonacci-based distributions are frequently found in many applications requiring uniform, low-discrepancy coverage of a domain.

---

### Spherical Fibonacci Grid

The Cartesian coordinates $$(x_j,\, y_j)$$ of the $$j$$-th point of a **Planar Fibonacci Grid** with $$N$$ samples are given by:

$$x_j = \frac{j}{N}, \qquad y_j = \text{frac}\!\left(\frac{j}{\Phi}\right) \qquad 0 \leq j < N$$

where $$\text{frac}(\cdot)$$ denotes the fractional part, keeping values inside the unit square $$[0,1)^2$$.

This gives a distribution of samples in the unit square where each coordinate has a different delta (space between points): $$1/N$$ in the $$x$$ coordinate, and $$1/\Phi$$ in the $$y$$ coordinate.

---

The viewer below shows the Planar Fibonacci Grid in the unit square. Observe how the points fill the square uniformly as you increase $$N$$. The vertical stripes at spacing $$1/N$$ and the diagonal offset at $$1/\Phi$$ are annotated to match the formula above.

<style>
  .pfg-wrap { margin: 2rem 0; }
  .pfg-controls {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem 1.5rem;
    padding: 0.8rem 1rem;
    background: var(--global-code-bg-color, #f8f8f8);
    border-radius: 6px;
    margin-bottom: 1.2rem;
    align-items: flex-end;
  }
  .pfg-ctrl { display: flex; flex-direction: column; gap: 3px; min-width: 140px; }
  .pfg-ctrl label { font-size: 0.75rem; font-weight: 600; text-transform: uppercase; letter-spacing: 0.05em; color: var(--global-text-color-light, #888); }
  .pfg-ctrl .pfg-val { font-size: 0.88rem; font-weight: 600; color: var(--global-theme-color, #6c63ff); }
  .pfg-ctrl input[type=range] { accent-color: var(--global-theme-color, #6c63ff); width: 100%; cursor: pointer; }
  .pfg-ctrl input[type=checkbox] { accent-color: var(--global-theme-color, #6c63ff); cursor: pointer; width: 16px; height: 16px; margin-top: 4px; }
  .pfg-ctrl select { font-size: 0.85rem; padding: 0.25rem 0.4rem; border-radius: 5px; border: 1px solid var(--global-divider-color, #ccc); background: var(--global-bg-color, white); color: var(--global-text-color, #333); cursor: pointer; }
  .pfg-canvas-wrap { display: flex; justify-content: center; }
  #pfg-canvas { border: 1px solid var(--global-divider-color, #ddd); border-radius: 4px; display: block; background: white; }
  .pfg-caption { text-align: center; font-size: 0.8rem; color: var(--global-text-color-light, #888); margin-top: 0.6rem; font-style: italic; }
  .pfg-reset { padding: 0.3rem 0.8rem; font-size: 0.82rem; border-radius: 5px; border: 1px solid var(--global-theme-color, #6c63ff); color: var(--global-theme-color, #6c63ff); background: transparent; cursor: pointer; transition: 0.15s; align-self: flex-end; }
  .pfg-reset:hover { background: var(--global-theme-color, #6c63ff); color: white; }
  .check-row { display: flex; align-items: center; gap: 6px; margin-top: 2px; }
  .check-row label { font-size: 0.82rem; color: var(--global-text-color, #444); text-transform: none; letter-spacing: 0; font-weight: 400; }
</style>

<div class="pfg-wrap">
  <div class="pfg-controls">
    <div class="pfg-ctrl">
      <label>Points N</label>
      <span class="pfg-val" id="pfg-n-val">30</span>
      <input type="range" id="pfg-n-sl" min="5" max="200" step="1" value="30">
    </div>
    <div class="pfg-ctrl">
      <label>Point size</label>
      <span class="pfg-val" id="pfg-s-val">6</span>
      <input type="range" id="pfg-s-sl" min="2" max="12" step="0.5" value="6">
    </div>
    <div class="pfg-ctrl">
      <label>Color mode</label>
      <select id="pfg-c-sel">
        <option value="index">By index</option>
        <option value="x">By x</option>
        <option value="y">By y</option>
        <option value="solid">Solid</option>
      </select>
    </div>
    <div class="pfg-ctrl">
      <label>Annotations</label>
      <div class="check-row">
        <input type="checkbox" id="pfg-grid-chk" checked>
        <label for="pfg-grid-chk">Show grid (1/N stripes)</label>
      </div>
      <div class="check-row">
        <input type="checkbox" id="pfg-ann-chk" checked>
        <label for="pfg-ann-chk">Show 1/Φ marker</label>
      </div>
    </div>
    <button class="pfg-reset" id="pfg-rst">↺ Reset</button>
  </div>
  <div class="pfg-canvas-wrap"><canvas id="pfg-canvas"></canvas></div>
  <p class="pfg-caption" id="pfg-cap">Planar Fibonacci Grid — 30 points in [0,1)²</p>
</div>

<script src="https://cdn.jsdelivr.net/npm/d3@7/dist/d3.min.js"></script>
<script>
(function () {
  const PHI = (1 + Math.sqrt(5)) / 2;

  const DEFAULTS = { n: 30, size: 6, color: 'index', grid: true, ann: true };
  let st = { ...DEFAULTS };

  // canvas sizing
  const canvas = document.getElementById('pfg-canvas');
  const PAD = { top: 24, right: 24, bottom: 48, left: 52 };

  function resize() {
    const maxW = Math.min(520, (canvas.parentElement.parentElement.clientWidth || 560) - 32);
    canvas.width  = maxW;
    canvas.height = maxW * 0.82;
  }
  resize();

  function frac(x) { return x - Math.floor(x); }

  function getColor(j, n, mode) {
    const t = j / Math.max(n - 1, 1);
    if (mode === 'solid') return '#4A90D9';
    if (mode === 'x')     return d3.interpolateViridis(j / n);
    if (mode === 'y')     return d3.interpolatePlasma(frac(j / PHI));
    // index
    return d3.interpolateRainbow(t);
  }

  function draw() {
    const { n, size, color, grid, ann } = st;
    const W = canvas.width, H = canvas.height;
    const iW = W - PAD.left - PAD.right;
    const iH = H - PAD.top  - PAD.bottom;
    const ctx = canvas.getContext('2d');

    ctx.clearRect(0, 0, W, H);

    // background
    ctx.fillStyle = '#ffffff';
    ctx.fillRect(PAD.left, PAD.top, iW, iH);

    // grid stripes (1/N spacing in x)
    if (grid) {
      ctx.strokeStyle = 'rgba(180,180,200,0.45)';
      ctx.lineWidth = 0.8;
      for (let j = 0; j <= n; j++) {
        const gx = PAD.left + (j / n) * iW;
        ctx.beginPath(); ctx.moveTo(gx, PAD.top); ctx.lineTo(gx, PAD.top + iH); ctx.stroke();
      }
    }

    // 1/Φ horizontal annotation
    if (ann) {
      const gy = PAD.top + (1 - 1/PHI) * iH;
      ctx.setLineDash([4, 3]);
      ctx.strokeStyle = 'rgba(220,80,80,0.55)';
      ctx.lineWidth = 1.2;
      ctx.beginPath(); ctx.moveTo(PAD.left, gy); ctx.lineTo(PAD.left + iW, gy); ctx.stroke();
      ctx.setLineDash([]);
      ctx.fillStyle = 'rgba(220,80,80,0.85)';
      ctx.font = '11px monospace';
      ctx.fillText('1/Φ', PAD.left - 30, gy + 4);
    }

    // axes
    ctx.strokeStyle = '#999';
    ctx.lineWidth = 1;
    ctx.strokeRect(PAD.left, PAD.top, iW, iH);

    // axis labels
    ctx.fillStyle = '#666';
    ctx.font = '12px sans-serif';
    ctx.textAlign = 'center';
    ctx.fillText('x', PAD.left + iW / 2, H - 6);
    ctx.fillText('0', PAD.left, PAD.top + iH + 16);
    ctx.fillText('1', PAD.left + iW, PAD.top + iH + 16);

    // 1/N tick on x axis
    const tickX = PAD.left + (1/n) * iW;
    ctx.fillStyle = '#999';
    ctx.font = '10px monospace';
    ctx.fillText('1/N', tickX, PAD.top + iH + 16);
    ctx.beginPath(); ctx.moveTo(tickX, PAD.top + iH); ctx.lineTo(tickX, PAD.top + iH + 5);
    ctx.strokeStyle = '#aaa'; ctx.lineWidth = 1; ctx.stroke();

    ctx.save(); ctx.translate(14, PAD.top + iH / 2);
    ctx.rotate(-Math.PI/2); ctx.fillStyle = '#666'; ctx.font = '12px sans-serif';
    ctx.textAlign = 'center'; ctx.fillText('y', 0, 0); ctx.restore();

    // y axis ticks
    ['0', '1'].forEach((lbl, i) => {
      const ty = PAD.top + (1 - i) * iH;
      ctx.fillStyle = '#888'; ctx.font = '11px sans-serif'; ctx.textAlign = 'right';
      ctx.fillText(lbl, PAD.left - 6, ty + 4);
    });

    // points
    for (let j = 0; j < n; j++) {
      const xj = j / n;
      const yj = frac(j / PHI);
      const px = PAD.left + xj * iW;
      const py = PAD.top  + (1 - yj) * iH;
      ctx.beginPath();
      ctx.arc(px, py, size / 2, 0, Math.PI * 2);
      ctx.fillStyle = getColor(j, n, color);
      ctx.globalAlpha = 0.9;
      ctx.fill();
      ctx.globalAlpha = 1;
      ctx.strokeStyle = 'rgba(255,255,255,0.6)';
      ctx.lineWidth = 0.8;
      ctx.stroke();
    }

    document.getElementById('pfg-cap').textContent =
      `Planar Fibonacci Grid — ${n} points in [0,1)²  ·  spacing: 1/N = ${(1/n).toFixed(3)} in x,  1/Φ ≈ ${(1/PHI).toFixed(3)} in y`;
  }

  // controls
  document.getElementById('pfg-n-sl').addEventListener('input', e => {
    st.n = +e.target.value; document.getElementById('pfg-n-val').textContent = st.n; draw();
  });
  document.getElementById('pfg-s-sl').addEventListener('input', e => {
    st.size = +e.target.value; document.getElementById('pfg-s-val').textContent = st.size; draw();
  });
  document.getElementById('pfg-c-sel').addEventListener('change', e => { st.color = e.target.value; draw(); });
  document.getElementById('pfg-grid-chk').addEventListener('change', e => { st.grid = e.target.checked; draw(); });
  document.getElementById('pfg-ann-chk').addEventListener('change',  e => { st.ann  = e.target.checked; draw(); });
  document.getElementById('pfg-rst').addEventListener('click', () => {
    st = { ...DEFAULTS };
    document.getElementById('pfg-n-sl').value  = st.n;
    document.getElementById('pfg-s-sl').value  = st.size;
    document.getElementById('pfg-c-sel').value = st.color;
    document.getElementById('pfg-grid-chk').checked = st.grid;
    document.getElementById('pfg-ann-chk').checked  = st.ann;
    document.getElementById('pfg-n-val').textContent = st.n;
    document.getElementById('pfg-s-val').textContent = st.size;
    draw();
  });

  draw();
})();
</script>

### Reading the grid

The vertical lines in the plot are spaced exactly $$1/N$$ apart — one column per point. Within each column there is exactly one sample, placed at a $$y$$-height of $$\text{frac}(j/\Phi)$$. Because $$\Phi$$ is irrational, these heights never repeat, and consecutive heights always differ by the golden ratio modulo 1 — the most uniform possible stepping.

The red dashed line marks $$y = 1/\Phi \approx 0.618$$, corresponding to the $$y$$-offset of the first point after $$j=0$$.

---

...


## References
- Marques et al. (2021), *Extensible Spherical Fibonacci Grids*, ACM Transactions on Graphics.