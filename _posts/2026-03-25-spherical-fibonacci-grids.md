---
layout: distill
title: Spherical Fibonacci Grids
date: 2026-03-25 11:12:00-0400
description: An interactive explanation of Spherical Fibonacci Grids and its properties.
tags:
categories: [interactive]
related_posts: false

bibliography: 2026-03-25-spherical-fibonacci-grids.bib

toc:
  - name: Theoretical Background
    subsections:
      - name: Golden Ratio
        subsections:
          - name: Further Golden Ratio Identities
      - name: Fibonacci Theory
  - name: Spherical Fibonacci Grid
  - name: Basis Vectors of a SFG
    subsections:
      - name: Unit Cell
      - name: Interpolation

_styles: >
  d-title {
    padding: '4rem 0 0.5rem'
  }
---

The main problem in **Physically-Based Ray Tracing** (*PBRT*) is converging the value of the illumination integral that appears in the **Light Transport Equation** (*LTE*) (eq. \eqref{LTE}). Integral equations generally do not have an analytic solution, so numerical integration techniques are used. The most common approach is **Monte Carlo integration** (*MC*), which is based on randomization:

\begin{equation}
  \label{LTE}
  L_o(p, \omega_o) = L_e(p, \omega_o) + \int_{S^2} f(p, \omega_o, \omega_i)\, L_i(p, \omega_i)\, |\cos\theta_i|\, d\omega_i
\end{equation}

Further improvements focus on manipulating the random sampling to minimize convergence time — that is, distributing the samples so that we obtain the minimum error with the minimum number of samples. These methods fall under the category of **Quasi-Monte Carlo integration** (*QMC*).

In this context, we find utility in low-discrepancy distributions such as *Fibonacci-based spherical distributions*. The main strength of these point sets is an extremely uniform distribution which is near-optimal in terms of *spherical cap discrepancy*. There are two families of such point sets:

- **SFIL** — Spherical Fibonacci point sets based on *planar Fibonacci integration lattices*.
- **SFG** — Spherical Fibonacci point sets based on *planar Fibonacci grids*.

*SFIL*s constrain point set sizes to be Fibonacci numbers. *SFG*s, on the other hand, allow generating point sets with an arbitrary number of points — which is strongly important in **Physically-Based Rendering** (*PBR*). For this reason, we focus on *SFG*s.

<br>

## Theoretical Background

---

### Golden Ratio

In mathematics, two quantities are in **golden ratio** if their ratio is the same as the ratio of their sum to the larger of the two quantities:

$$\frac{a+b}{a} = \frac{a}{b} = \Phi \qquad \text{such that} \quad a > b > 0$$

$$\Phi$$ (also called the extreme and *mean ratio*, or *divine proportion*) satisfies the quadratic equation and is an irrational number. Some people call it the most irrational number, and this property makes it extremely useful when distributing samples without generating repetition patterns, therefore implying a low discrepancy.

\begin{equation}
  \label{golden ratio}
  \Phi = \frac{1 + \sqrt{5}}{2} = 1.618\ldots
\end{equation}

The following key identity is what makes it unique (the derivation can be found in the following subsection):

\begin{equation}
  \label{gr key identity}
  \frac{1}{\Phi} = \Phi - 1 \quad \Rightarrow \quad \frac{1}{\Phi} + 1 = \Phi \quad \Rightarrow \quad 1 + \Phi = \Phi^{2}   
\end{equation}

This is mathematically exact and unique to the golden ratio because it satisfies the quadratic equation. We can also sustitute $$\Phi$$ in eq. \eqref{gr key identity} to see it in a continued fraction expansion form. This shows that the golden ratio is the **most irrational number**, meaning that its rational approximation converges slower than for any other irrational number.

$$\Phi = 1 + \frac{1}{\Phi} =  1 + \frac{1}{1 + \frac{1}{1 + \frac{1}{\ldots}}}$$

#### Further Golden Ratio Identities

<details>
<summary>$\quad \Phi^{-1} = \frac{\sqrt{5} - 1}{2}$</summary>
$$\Phi^{-1} = \frac{1}{\Phi} = \frac{2}{\sqrt{5} + 1} = \frac{2}{\sqrt{5} + 1} · \frac{\sqrt{5} - 1}{\sqrt{5} - 1} = 
\frac{2 (\sqrt{5} - 1)}{\sqrt{5} \sqrt{5} - \sqrt{5} + \sqrt{5} - 1}  = \frac{2 (\sqrt{5} - 1)}{4} = \frac{\sqrt{5} - 1}{2}$$
</details>

<details>
<summary>$\quad \Phi - \Phi^{-1} = 1$</summary>
$$\Phi - \Phi^{-1} = \frac{1 + \sqrt{5}}{2} - \frac{\sqrt{5} - 1}{2} = \frac{2}{2} = 1$$
</details>

<details>
<summary>$\quad \Phi^{2} = \frac{3 + \sqrt{5}}{2}$</summary>
$$\Phi^{2} = (\frac{1 + \sqrt{5}}{2})^{2} = \frac{1 + \sqrt{5} + \sqrt{5} + \sqrt{5}^{2}}{4} = \frac{6 + 2 \sqrt{5}}{4} = \frac{3 + \sqrt{5}}{2}$$
</details>

<details>
<summary>$\quad \Phi + 1 = \frac{3 + \sqrt{5}}{2}$</summary>
$$\Phi + 1 = \frac{1 + \sqrt{5}}{2} + 1 = \frac{1 + \sqrt{5} + 2}{2} = \frac{3 + \sqrt{5}}{2}$$
</details>

<details>
<summary>$\quad \Phi^{-2} = \frac{3 - \sqrt{5}}{2}$</summary>
$$\Phi^{-2} = \frac{1}{\Phi^{2}} = \frac{2}{3 + \sqrt{5}} = \frac{2}{3 + \sqrt{5}} · \frac{3 - \sqrt{5}}{3 - \sqrt{5}} = 
\frac{2 (3 - \sqrt{5})}{9 - 3 \sqrt{5} + 3 \sqrt{5} - \sqrt{5}^{2}} = \frac{6 - 2 \sqrt{5}}{9 - 5} = \frac{3 - \sqrt{5}}{2}$$
</details>

<br>

### Fibonacci Theory

In mathematics, the *Fibonacci sequence* is defined such that each element is the sum of the two preceding ones:

$$F_m = F_{m-1} + F_{m-2} \quad \text{for } m > 1, \qquad \text{where } F_0 = 0 \text{ and } F_1 = 1$$

Fibonacci numbers appear unexpectedly often in mathematics, and are applied in multiple areas such as computer algorithms and biological settings (*phyllotaxis*, *shell spirals*, *branching patterns*).

Fibonacci numbers are also strongly related to the golden ratio. As the sequence progresses, the ratio of successive Fibonacci numbers approaches a specific irrational number, the golden ratio $$\Phi$$:

$$\lim_{m \to \infty} \frac{F_{m+1}}{F_m} = \Phi \approx 1.618\ldots$$

Due to this relation, Fibonacci-based distributions are frequently found in many applications requiring uniform, low-discrepancy coverage of a domain.

<br>

## Spherical Fibonacci Grid

---

The *Cartesian* coordinates $(x_j,\, y_j)$ of the $$j$$*-th* point of a **Planar Fibonacci Grid** ($F_G$) with $$N$$ samples are given by:

$$
\left.
\begin{aligned}
x_j &= \frac{j}{N} \\
y_j &= \operatorname{frac}\!\left(\frac{j}{\Phi}\right)
\end{aligned}
\right\}
\quad
0 \le j \le N
\label{eq:pfg}
$$

<span class="tt2-wrap">
  <span class="tt2">frac()</span>
  <span class="tt2-box">
    <div class="tt2-title">frac(x) = x mod 1</div>
    <div class="tt2-body">Takes the fractional part of a number.</div>
  </span>
</span>

where $frac()$ denotes the fractional part, keeping values inside the unit square $[0,1)^2$.

This gives a distribution of samples in the **unit square** where each coordinate has a different *delta* (space between points): $$\frac{1}{N}$$ in the *x*-coordinate, and $$\frac{1}{\Phi}$$ in the *y*-coordinate.

---

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

---

We can directly define the distribution on the **unit sphere** by using the **Lambert cylindrical equal-area projection**. The result gives the samples in terms of the elevation angle $$\theta$$ and the azimuth angle $$\phi$$. To maintain the uniformity of the samples along the vertical axis, the *x*-coordinate is mapped to $$z = \cos(\theta)$$ instead of just $$\theta$$. This projection gives us the following relation equations:

\begin{equation}
  x = \frac{1 - \cos(\theta)}{2} \\
  y = \frac{\phi}{2\pi}
\end{equation}

$$
\begin{equation}
  \begin{aligned}
    x &= \frac{1 - \cos(\theta)}{2} \\
    y &= \frac{\phi}{2\pi}
  \end{aligned}
\end{equation}
$$

By substituting in the previous equation we obtain the polar coordinates of the $$j$$*-th* point of a **Spherical Fibonacci Grid**:

$$
\left.
\begin{aligned}
\theta_j = \arccos\!\left(1 - \frac{2j}{N}\right) \\
\phi_j = 2\pi j\,\Phi^{-1} \bmod 2\pi
\end{aligned}
\right\}
\quad 0 \le j \le N
\label{eq:sfg}
$$

$$
\begin{equation}
  \label{eq:sfg}
  \left.
  \begin{aligned}
    \theta_j &= \arccos\!\left(1 - \frac{2j}{N}\right) \\
    \phi_j &= 2\pi j\,\Phi^{-1} \bmod 2\pi
  \end{aligned}
  \right\}
  \quad 0 \le j \le N
\end{equation}
$$

Also, *SFG*s are sometimes expressed with a shift in the *z*-axis. This shift symmetrizes the *z*-coordinate distribution so that the first and last points are at equal distances from their closest pole, which **improves the spherical discrepancy** of the point set.

This shift is **half the distance between samples** in the *z*-axis (half the delta). It can be computed either in the unit square or on the unit sphere:

- **Unit Square:** $$\Delta x = \frac{1}{N} \Rightarrow$$ (Half of this distance) $$\Rightarrow \frac{1}{2N}$$
- **Unit Sphere:** $$z = \cos(\theta) = 1 - 2x = \frac{1 - 2j}{N} \Rightarrow \Delta z = \frac{2}{N} \Rightarrow$$ (Half of this distance) $$\Rightarrow \frac{1}{N}$$

The symmetrized formulas become:

$$
\left.
\begin{aligned}
\theta_j = \arccos\!\left(1 - \frac{2j+1}{N}\right) \\
\phi_j = 2\pi j\,\Phi^{-1} \bmod 2\pi
\end{aligned}
\right\}
\quad 0 \le j \le N
\label{eq:shift sfg}
$$

To finish this section, the previous equations will be presented in terms of the unit hemishpere instead of the unit sphere. This is quite necessary, since many uses cases in PBR are computed on the hemisphere (only accounting for the reflection effect and omitting the transmittance).

**[HEMISPHERE EQUATIONS ...]**

---

## Basis Vectors of a SFG

An alternative definition of a planar Fibonacci grid exists using a pair of consecutive **basis vectors**. In this case, the points are not restricted to the unit square but they exist in a **planar Fibonacci lattice** ($F_L$).

$$F_L = \{ p = z_0 b_k + z_1 b_{k+1} : (z_0, z_1) \in \mathbb{Z}^2 \}$$

Swinbank and Purser {% cite swinbank&purser %} observed that the basis vectors of a $F_L$ are expressed by:

$$b_k = (\frac{F_k}{N}, \frac{(-1)^{k-1}}{\Phi^k}), \qquad \text{where } k = 0, 1, \ldots, k_m \qquad \text{and } k_m \text{such that } F_{k_m} \leq N < F_{k_m+1}$$

$N$ is the number of points on the grid and $F_{k_m}$ the largest Fibonacci number smaller or equal to $N$. The basis vectors satisfy a recurrence relationship similar to that of Fibonacci numbers. 

$$b_{k+1} = b_k + b_{k-1}$$

### Unit Cell

Using any pair of basis vectors ($b_k, b_{k+1}$) we can obtain a parallelogram area called **unit cell**. By definition the unit cell does not contain any point in its interior. We can compute the area of the parallelogram by doing the determinant of the matrix composed by the basis vectors of that parallelogram. 

<aside><p>This is a a common linear algebra rule. A detail video demonstrating it can be found [here](https://youtu.be/n-S63_goDFg")</p>.</aside>

$$M = 
\begin{bmatrix}
  b_k & b_{k+1}
\end{bmatrix}
=
\begin{bmatrix}
  \frac{F_k}{N} & \frac{F_{k+1}}{N} \\
  \frac{(-1)^{k-1}}{\Phi^{k}} & \frac{(-1)^{k}}{\Phi^{k+1}}
\end{bmatrix}
$$

$$det(M) = \Delta_M = \frac{F_k}{N} · \frac{(-1)^{k}}{\Phi^{k+1}} - \frac{F_{k+1}}{N} · \frac{(-1)^{k-1}}{\Phi^{k}} = \frac{1}{N}$$

We prove that the area of the unit cell of a Fibonacci distribution for any **N** will be $$\frac{1}{N}$$. This confirms us that it is a suitable lattice for *QMC* numerical integration. 

### Interpolation

...


## References
- Marques, R. et al. (2021), *Extensible Spherical Fibonacci Grids*, IEEE Transactions on Visualization and Computer Graphics.


Distributed under the terms of the [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/) License.