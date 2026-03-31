# 3D Solar System Simulator — Project Plan

Build a single-file HTML/CSS/JS interactive 3D solar system with accurate astronomical data, full planet rendering (including major moons and Saturn's rings), mouse/touch camera controls, planet-centering on click, and adjustable orbit speed.

---

## Tech Stack

- **Rendering:** Raw WebGL (no external libraries) — consistent with the existing `Earth.html` approach
- **Single file:** All HTML, CSS, JS, and GLSL shaders in one `.html` file
- **No textures:** Procedural shading with per-planet color gradients, band patterns (Jupiter/Saturn), and polar ice caps — avoids CORS/loading issues

---

## Architecture

### 1. Data Layer — Astronomical Constants
- Sun + 8 planets (Mercury → Neptune) with real orbital data:
  - Semi-major axis (AU), eccentricity, orbital period (years), inclination
  - Planet radius (km, log-scaled for visibility), axial tilt, rotation period
  - Visual color palette per planet (multi-color gradient to approximate real appearance)
- **Major moons** (≈12 total):
  - Earth: Moon
  - Mars: Phobos, Deimos
  - Jupiter: Io, Europa, Ganymede, Callisto
  - Saturn: Titan, Enceladus
  - Uranus: Titania
  - Neptune: Triton
- **Saturn's rings:** Rendered as a flat annular disc with transparency

### 2. Rendering Engine
- **Sphere geometry:** Shared UV-sphere mesh (configurable lat/long segments)
- **Vertex shader:** Model-view-projection transform with per-instance position/scale
- **Fragment shader:** Per-planet procedural coloring:
  - Gas giants: horizontal banding (sinusoidal color variation by latitude)
  - Rocky planets: solid color with subtle noise
  - Sun: emissive glow with animated pulsing
- **Ring shader:** Separate flat-ring geometry for Saturn with alpha blending
- **Orbit lines:** GL_LINE_LOOP for each planet's elliptical path
- **Starfield background:** Random point sprites or small quads

### 3. Physics / Orbital Mechanics
- Kepler's equation solver (Newton-Raphson iteration for eccentric anomaly)
- True anomaly → 3D position using orbital elements (a, e, i, Ω, ω)
- Time parameter `t` drives all positions; scaled by user speed multiplier
- Moon orbits computed relative to parent planet position

### 4. Camera System
- **Orbit camera:** Spherical coordinates (azimuth, elevation, distance) around a focus target
- **Focus target:** Defaults to Sun (0,0,0); click a planet to re-center
- Smooth animated transition when switching focus target (lerp over ~1s)
- **Mouse controls:**
  - Left-drag: rotate azimuth/elevation
  - Scroll wheel: zoom in/out
- **Touch controls:**
  - Single-finger drag: rotate
  - Pinch: zoom
- Distance clamped to reasonable min/max per focus target

### 5. Interaction & UI
- **Click/tap detection:** Ray-cast from screen point into 3D scene; find nearest planet/moon intersection
- **Planet info panel:** On click, show name, radius, orbital period, distance from Sun
- **Speed slider:** HTML range input controlling time multiplier (0.1× to 100×)
- **Pause/play button**
- **Reset view button:** Return to Sun-centered default view
- **Planet labels:** Screen-space text labels rendered via 2D canvas overlay or HTML divs positioned via 3D→2D projection
- **Orbit trails toggle:** Show/hide orbital paths

### 6. Visual Polish
- Dark space background with starfield
- Sun glow effect (additive blending or radial gradient overlay)
- Subtle ambient + directional lighting from Sun position
- Anti-aliased orbit lines
- Responsive canvas (fills viewport, handles resize)

---

## Scale Strategy

Real solar system distances make inner planets invisible next to outer ones. Strategy:
- **Logarithmic distance scaling** for orbital radii (compresses outer orbits, expands inner)
- **Exaggerated planet radii** (cube-root or log scale) so all planets are visible
- Display real data in the info panel, even though rendering is scaled

---

## File Structure (single file)

```
SolarSystem.html
├── <style>        — CSS for UI controls, info panel, labels
├── <body>
│   ├── <canvas>   — WebGL viewport
│   ├── UI overlay — speed slider, buttons, info panel, labels
│   └── <script>
│       ├── Astronomical data tables
│       ├── Matrix math utilities
│       ├── Kepler equation solver
│       ├── Sphere/ring geometry generators
│       ├── GLSL shaders (vertex + fragment, embedded as strings)
│       ├── WebGL setup & render pipeline
│       ├── Camera controller (mouse + touch)
│       ├── Ray-cast picking
│       ├── Animation loop
│       └── UI event handlers
```

---

## Implementation Steps

1. **Scaffold HTML + CSS** — canvas, UI overlay, controls, dark theme
2. **WebGL boilerplate** — context, shaders, matrix lib (reuse patterns from Earth.html)
3. **Sphere mesh generator** — parameterized lat/long UV-sphere
4. **Planet data table** — all orbital elements, colors, sizes
5. **Kepler solver + orbital positions** — compute positions from time
6. **Render planets + Sun** — procedural shading per body
7. **Orbit path lines** — elliptical GL_LINE_LOOP per planet
8. **Camera system** — orbit camera with mouse drag + scroll zoom
9. **Touch gesture support** — single-drag rotate, pinch zoom
10. **Click-to-focus** — ray-cast picking, smooth camera transition
11. **Moon orbits** — attach moons to parent planets
12. **Saturn rings** — flat annular geometry with alpha
13. **Starfield background** — random point sprites
14. **Sun glow** — additive blending effect
15. **Speed control + UI** — slider, pause, reset, labels, info panel
16. **Polish & test** — responsive resize, edge cases, performance

---

## Estimated Complexity

~1500–2500 lines of HTML/CSS/JS. Largest sections will be the planet data table, the Kepler solver, and the WebGL rendering pipeline. The single-file constraint is feasible since there are no external dependencies.
