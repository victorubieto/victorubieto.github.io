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


The **Fibonacci lattice** (also known as the *sunflower spiral* or *Fermat spiral*) is one of the most beautiful patterns in mathematics. It appears everywhere in nature: the seeds of a sunflower, the scales of a pine cone, the petals of a daisy. The underlying mechanism is surprisingly simple — place each point at an angle of one **golden ratio turn** from the previous one, and pack them outward radially.

## The Golden Angle

The golden ratio $$\varphi = \frac{1 + \sqrt{5}}{2} \approx 1.618$$ gives rise to the **golden angle**:

$$\theta = 2\pi \left(1 - \frac{1}{\varphi}\right) \approx 137.5°$$

Each new seed in a sunflower is placed at exactly this angle from the previous one. Because $$\varphi$$ is the "most irrational" number — the hardest to approximate by rationals — this angle packs seeds with maximum efficiency, leaving no wasted space.

## The Lattice Formula

For the $$n$$-th point, the polar coordinates are:

$$r_n = \sqrt{n}, \quad \alpha_n = n \cdot \theta$$

The $$\sqrt{n}$$ spacing ensures the points are distributed uniformly in area (equal area per point). Together, this produces the characteristic spiral arms you can see in nature.

## Interactive Visualization

Use the controls below to explore how the distribution changes. Try varying the divergence angle slightly away from the golden angle — the spiral arms collapse into coarse lines. Only near $$137.5°$$ does the perfect packing emerge.

<style>
  .fib-container {
    font-family: inherit;
    margin: 2rem 0;
  }
  .fib-controls {
    display: flex;
    flex-wrap: wrap;
    gap: 1.5rem;
    margin-bottom: 1.5rem;
    padding: 1rem 1.2rem;
    background: var(--global-bg-color, #f8f8f8);
    border-radius: 8px;
    border: 1px solid var(--global-divider-color, #ddd);
  }
  .fib-control-group {
    display: flex;
    flex-direction: column;
    gap: 0.3rem;
    min-width: 180px;
  }
  .fib-control-group label {
    font-size: 0.82rem;
    font-weight: 600;
    color: var(--global-text-color-light, #666);
    text-transform: uppercase;
    letter-spacing: 0.05em;
  }
  .fib-control-group .value-display {
    font-size: 0.9rem;
    font-weight: 600;
    color: var(--global-theme-color, #6c63ff);
  }
  .fib-control-group input[type=range] {
    width: 100%;
    accent-color: var(--global-theme-color, #6c63ff);
    cursor: pointer;
  }
  .fib-control-group select {
    padding: 0.3rem 0.5rem;
    border-radius: 5px;
    border: 1px solid var(--global-divider-color, #ccc);
    background: var(--global-bg-color, white);
    color: var(--global-text-color, #333);
    font-size: 0.9rem;
    cursor: pointer;
  }
  #fib-svg-container {
    display: flex;
    justify-content: center;
  }
  #fib-svg-container svg {
    border-radius: 50%;
    background: var(--global-bg-color, #fff);
  }
  .fib-caption {
    text-align: center;
    font-size: 0.82rem;
    color: var(--global-text-color-light, #888);
    margin-top: 0.8rem;
    font-style: italic;
  }
  .fib-reset-btn {
    padding: 0.35rem 0.9rem;
    border-radius: 5px;
    border: 1px solid var(--global-theme-color, #6c63ff);
    background: transparent;
    color: var(--global-theme-color, #6c63ff);
    font-size: 0.85rem;
    cursor: pointer;
    transition: background 0.15s, color 0.15s;
    align-self: flex-end;
  }
  .fib-reset-btn:hover {
    background: var(--global-theme-color, #6c63ff);
    color: white;
  }
</style>

<div class="fib-container">
  <div class="fib-controls">
    <div class="fib-control-group">
      <label>Points</label>
      <span class="value-display" id="n-display">800</span>
      <input type="range" id="n-slider" min="50" max="2000" step="50" value="800">
    </div>
    <div class="fib-control-group">
      <label>Divergence angle (°)</label>
      <span class="value-display" id="angle-display">137.508°</span>
      <input type="range" id="angle-slider" min="130" max="145" step="0.001" value="137.508">
    </div>
    <div class="fib-control-group">
      <label>Point size</label>
      <span class="value-display" id="size-display">3.5</span>
      <input type="range" id="size-slider" min="1" max="8" step="0.5" value="3.5">
    </div>
    <div class="fib-control-group">
      <label>Color mode</label>
      <select id="color-mode">
        <option value="spiral">Spiral angle</option>
        <option value="radius">Radius</option>
        <option value="index">Index</option>
        <option value="solid">Solid</option>
      </select>
    </div>
    <button class="fib-reset-btn" id="reset-btn">↺ Reset</button>
  </div>

  <div id="fib-svg-container"></div>
  <p class="fib-caption" id="fib-caption">
    Golden angle (137.508°) — 800 points packed with maximum efficiency
  </p>
</div>

<script src="https://cdn.jsdelivr.net/npm/d3@7"></script>
<script>
(function() {
  const GOLDEN_ANGLE = 137.50776405003785;
  const DEFAULTS = { n: 800, angle: GOLDEN_ANGLE, size: 3.5, colorMode: 'spiral' };

  let state = { ...DEFAULTS };

  // --- SVG setup ---
  const containerEl = document.getElementById('fib-svg-container');
  const W = Math.min(560, containerEl.parentElement.clientWidth || 560);
  const H = W;
  const cx = W / 2, cy = H / 2;
  const maxR = W * 0.46;

  const svg = d3.select('#fib-svg-container')
    .append('svg')
    .attr('width', W)
    .attr('height', H)
    .attr('viewBox', `0 0 ${W} ${H}`);

  // Background circle clip
  const defs = svg.append('defs');
  defs.append('clipPath').attr('id', 'circle-clip')
    .append('circle').attr('cx', cx).attr('cy', cy).attr('r', maxR);

  // Subtle background circle border
  svg.append('circle')
    .attr('cx', cx).attr('cy', cy).attr('r', maxR)
    .attr('fill', 'none')
    .attr('stroke', '#ccc')
    .attr('stroke-width', 1);

  const g = svg.append('g').attr('clip-path', 'url(#circle-clip)');

  // --- Color scales ---
  function getColor(i, n, angle) {
    if (state.colorMode === 'solid') return 'var(--global-theme-color, #6c63ff)';
    if (state.colorMode === 'radius') {
      return d3.interpolateViridis(Math.sqrt(i / n));
    }
    if (state.colorMode === 'index') {
      return d3.interpolatePlasma(i / n);
    }
    // spiral: color by angle in the spiral
    const spiralAngle = ((i * angle) % 360) / 360;
    return d3.interpolateRainbow(spiralAngle);
  }

  // --- Draw ---
  function draw() {
    const { n, angle, size } = state;
    const angleRad = (angle * Math.PI) / 180;

    const data = d3.range(n).map(i => {
      const r = maxR * Math.sqrt((i + 0.5) / n);
      const a = i * angleRad;
      return {
        x: cx + r * Math.cos(a),
        y: cy + r * Math.sin(a),
        i,
        r
      };
    });

    const dots = g.selectAll('circle.dot').data(data, d => d.i);

    // Enter
    dots.enter()
      .append('circle')
      .attr('class', 'dot')
      .attr('cx', d => d.x)
      .attr('cy', d => d.y)
      .attr('r', 0)
      .attr('fill', d => getColor(d.i, n, angle))
      .attr('opacity', 0.85)
      .transition().duration(300)
      .attr('r', size);

    // Update
    dots.transition().duration(120)
      .attr('cx', d => d.x)
      .attr('cy', d => d.y)
      .attr('r', size)
      .attr('fill', d => getColor(d.i, n, angle));

    // Exit
    dots.exit().transition().duration(200).attr('r', 0).remove();

    // Caption
    const isGolden = Math.abs(angle - GOLDEN_ANGLE) < 0.01;
    const caption = isGolden
      ? `Golden angle (${angle.toFixed(3)}°) — ${n} points packed with maximum efficiency`
      : `Divergence angle: ${angle.toFixed(3)}° — ${n} points (golden angle = ${GOLDEN_ANGLE.toFixed(3)}°)`;
    document.getElementById('fib-caption').textContent = caption;
  }

  // --- Controls ---
  const nSlider    = document.getElementById('n-slider');
  const angleSlider = document.getElementById('angle-slider');
  const sizeSlider  = document.getElementById('size-slider');
  const colorSelect = document.getElementById('color-mode');
  const resetBtn    = document.getElementById('reset-btn');

  nSlider.addEventListener('input', () => {
    state.n = +nSlider.value;
    document.getElementById('n-display').textContent = state.n;
    draw();
  });

  angleSlider.addEventListener('input', () => {
    state.angle = +angleSlider.value;
    document.getElementById('angle-display').textContent = state.angle.toFixed(3) + '°';
    draw();
  });

  sizeSlider.addEventListener('input', () => {
    state.size = +sizeSlider.value;
    document.getElementById('size-display').textContent = state.size;
    draw();
  });

  colorSelect.addEventListener('change', () => {
    state.colorMode = colorSelect.value;
    draw();
  });

  resetBtn.addEventListener('click', () => {
    state = { ...DEFAULTS };
    nSlider.value = state.n;
    angleSlider.value = state.angle;
    sizeSlider.value = state.size;
    colorSelect.value = state.colorMode;
    document.getElementById('n-display').textContent = state.n;
    document.getElementById('angle-display').textContent = state.angle.toFixed(3) + '°';
    document.getElementById('size-display').textContent = state.size;
    draw();
  });

  draw();
})();
</script>

## Why does the golden angle work?

The Fibonacci numbers $$1, 1, 2, 3, 5, 8, 13, 21, \ldots$$ appear because their ratios converge to $$\varphi$$. When you look at a sunflower, you always find two families of spirals — one going clockwise, one counter-clockwise — and their counts are always consecutive Fibonacci numbers (e.g. 34 and 55, or 55 and 89).

This is a direct consequence of the golden angle: consecutive Fibonacci-numbered spiral arms are the *best rational approximations* to the golden ratio, so they dominate the visual pattern.

Try dragging the **divergence angle** slider slightly away from 137.508° and watch the tightly-packed arrangement dissolve into coarse radial spokes — a vivid demonstration of why irrationality is, paradoxically, the most efficient packing strategy nature could have chosen.
