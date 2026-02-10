# Three.js Architecture Specification -- Haze Client Website Redesign

**Author**: Architect Agent
**Date**: 2026-02-10
**Status**: Ready for Implementation
**Approach**: Modified Hybrid (Council-Approved)

---

## 1. Problem Statement

The current Haze Client website (`index.html`, 2291 lines, all inline) uses a static CSS
`linear-gradient` grid background with zero interactivity. The Council approved a Modified
Hybrid approach: replace the CSS grid with a shader-based Three.js scene on desktop while
preserving the existing CSS grid as a mobile fallback. The goal is visual differentiation --
a memorable, premium feel -- without sacrificing the <3s load budget or mobile performance.

### Fundamental Constraints

1. **No build system** -- vanilla JS, CDN `<script>` tags only
2. **Single file** -- everything lives in `index.html` (inline styles + scripts)
3. **Load budget** -- <3 seconds total page load
4. **Performance floor** -- 60fps on mid-range desktop GPU, zero Three.js on mobile
5. **Existing content** -- all HTML sections must remain intact and visually enhanced
6. **Color palette** -- `#0a0e1a` background, `#00ff88` accent, existing CSS variables

---

## 2. HTML Structure Changes

### 2.1 Script Tags (add before closing `</body>`)

```html
<!-- Three.js CDN -- loaded with defer to not block rendering -->
<script defer src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r169/three.min.js"
        integrity="sha512-..." crossorigin="anonymous"
        referrerpolicy="no-referrer"></script>

<!-- All scene code inline, after Three.js loads -->
<script>
  // Device detection + scene initialization (see Section 4)
  // Wrapped in window.addEventListener('load', ...) to guarantee Three.js is available
</script>
```

**Why r169**: Last stable release with no ESM-only requirement. Works as a global
`THREE` namespace from a CDN script tag. Size: ~160KB gzipped. Loads in <500ms on
broadband.

### 2.2 Canvas Placement

Replace the current `.background-grid` div with a conditional structure:

```html
<!-- Three.js Canvas Container (replaces .background-grid on desktop) -->
<canvas id="haze-bg-canvas" style="
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: -2;
  pointer-events: none;
  display: none;
"></canvas>

<!-- CSS Grid Fallback (shown on mobile, hidden on desktop when Three.js loads) -->
<div class="background-grid" id="css-grid-fallback"></div>
```

The canvas starts with `display: none`. The initialization script shows it and hides
the CSS fallback only after successful Three.js setup.

### 2.3 Hero Section Icosahedron Container

Add a positioned container inside `.hero` for the wireframe icosahedron:

```html
<section class="hero">
  <!-- Icosahedron render target (desktop only, positioned behind hero-content) -->
  <canvas id="hero-icosahedron-canvas" style="
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 500px;
    height: 500px;
    pointer-events: none;
    opacity: 0;
    z-index: 0;
  "></canvas>

  <div class="hero-content" style="position: relative; z-index: 1;">
    <!-- existing hero content unchanged -->
  </div>
</section>
```

The icosahedron canvas uses a separate smaller renderer for isolation. This prevents
the hero geometry from interfering with the full-page background scene and allows
independent lifecycle management.

---

## 3. Three.js Scene Architecture

### 3.1 Architecture Overview

Two independent Three.js contexts:

```
Scene A: Background (full viewport)
  - Renderer: WebGLRenderer on #haze-bg-canvas
  - Camera: PerspectiveCamera
  - Objects: ShaderMaterial grid plane + 400 particle system
  - Lifecycle: persistent, always running

Scene B: Hero Icosahedron (500x500 viewport)
  - Renderer: WebGLRenderer on #hero-icosahedron-canvas
  - Camera: PerspectiveCamera
  - Objects: Wireframe IcosahedronGeometry
  - Lifecycle: created on load, fades in, pauses when scrolled past
```

**Why two renderers**: A single renderer would require complex scissor/viewport management
and would couple the hero animation lifecycle to the background. Two small renderers are
simpler to reason about, easier to independently pause, and the second renderer (500x500)
has negligible GPU cost.

### 3.2 Scene A: Background Scene

```javascript
// --- SCENE A: Full-page background ---
const bgScene = new THREE.Scene();

// Camera: looking down at the grid plane at an angle
const bgCamera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);
bgCamera.position.set(0, 8, 12);
bgCamera.lookAt(0, 0, 0);

// Renderer: uses the fixed canvas, alpha for transparency
const bgRenderer = new THREE.WebGLRenderer({
  canvas: document.getElementById('haze-bg-canvas'),
  antialias: false,       // performance: skip AA on full-screen background
  alpha: true,            // transparent background so page bg shows through
  powerPreference: 'high-performance'
});
bgRenderer.setSize(window.innerWidth, window.innerHeight);
bgRenderer.setPixelRatio(Math.min(window.devicePixelRatio, 2)); // cap at 2x
bgRenderer.setClearColor(0x000000, 0); // fully transparent
```

### 3.3 Scene B: Hero Icosahedron Scene

```javascript
// --- SCENE B: Hero icosahedron ---
const heroScene = new THREE.Scene();

const heroCamera = new THREE.PerspectiveCamera(50, 1, 0.1, 100); // 1:1 aspect
heroCamera.position.set(0, 0, 4);

const heroRenderer = new THREE.WebGLRenderer({
  canvas: document.getElementById('hero-icosahedron-canvas'),
  antialias: true,        // small canvas, AA is cheap here
  alpha: true,
  powerPreference: 'high-performance'
});
heroRenderer.setSize(500, 500);
heroRenderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
heroRenderer.setClearColor(0x000000, 0);
```

---

## 4. Device Detection and Fallback Logic

### 4.1 Decision Tree

```javascript
function shouldEnableThreeJS() {
  // Gate 1: Screen width -- mobile/tablet gets CSS fallback
  if (window.innerWidth < 1024) return false;

  // Gate 2: WebGL availability
  try {
    const canvas = document.createElement('canvas');
    const gl = canvas.getContext('webgl2') || canvas.getContext('webgl');
    if (!gl) return false;
  } catch (e) {
    return false;
  }

  // Gate 3: Reduced motion preference
  if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) return false;

  // Gate 4: Device memory (if available) -- skip on <4GB devices
  if (navigator.deviceMemory && navigator.deviceMemory < 4) return false;

  // Gate 5: Hardware concurrency -- skip on single/dual core
  if (navigator.hardwareConcurrency && navigator.hardwareConcurrency < 4) return false;

  return true;
}
```

### 4.2 Initialization Flow

```javascript
window.addEventListener('load', function() {
  if (typeof THREE === 'undefined') return; // CDN failed, CSS fallback stays

  if (!shouldEnableThreeJS()) return; // CSS fallback stays visible

  // Hide CSS grid, show Three.js canvas
  document.getElementById('css-grid-fallback').style.display = 'none';
  const bgCanvas = document.getElementById('haze-bg-canvas');
  bgCanvas.style.display = 'block';

  const heroCanvas = document.getElementById('hero-icosahedron-canvas');
  heroCanvas.style.display = 'block';

  initBackgroundScene();   // Scene A
  initHeroScene();         // Scene B
  startAnimationLoop();    // unified rAF loop
});
```

### 4.3 Runtime Performance Monitor

```javascript
// If FPS drops below threshold for sustained period, kill Three.js
const fpsMonitor = {
  frames: 0,
  lastTime: performance.now(),
  lowFpsCount: 0,

  tick() {
    this.frames++;
    const now = performance.now();
    if (now - this.lastTime >= 1000) {
      const fps = this.frames;
      this.frames = 0;
      this.lastTime = now;

      if (fps < 30) {
        this.lowFpsCount++;
        if (this.lowFpsCount >= 3) {
          // 3 consecutive seconds below 30fps -- bail out
          killThreeJS();
        }
      } else {
        this.lowFpsCount = 0;
      }
    }
  }
};

function killThreeJS() {
  // Stop animation loop
  cancelAnimationFrame(animationFrameId);

  // Dispose geometries, materials, textures
  bgScene.traverse(obj => {
    if (obj.geometry) obj.geometry.dispose();
    if (obj.material) {
      if (obj.material.map) obj.material.map.dispose();
      obj.material.dispose();
    }
  });
  heroScene.traverse(obj => {
    if (obj.geometry) obj.geometry.dispose();
    if (obj.material) {
      if (obj.material.map) obj.material.map.dispose();
      obj.material.dispose();
    }
  });

  bgRenderer.dispose();
  heroRenderer.dispose();

  // Restore CSS fallback
  document.getElementById('haze-bg-canvas').style.display = 'none';
  document.getElementById('hero-icosahedron-canvas').style.display = 'none';
  document.getElementById('css-grid-fallback').style.display = 'block';
}
```

---

## 5. GLSL Shader Code for Grid Plane

### 5.1 Vertex Shader

```glsl
// grid_vertex.glsl
varying vec2 vUv;
varying float vDistFromCenter;

void main() {
  vUv = uv;
  vDistFromCenter = length(position.xz); // radial distance for edge fade
  gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
}
```

### 5.2 Fragment Shader

```glsl
// grid_fragment.glsl
precision mediump float;

uniform float uTime;
uniform vec2 uMouse;       // normalized mouse position (-1 to 1)
uniform float uScrollY;    // normalized scroll (0 to 1)

varying vec2 vUv;
varying float vDistFromCenter;

// Draws anti-aliased grid lines
float gridLine(float coord, float lineWidth) {
  float halfWidth = lineWidth * 0.5;
  float d = abs(fract(coord - 0.5) - 0.5);
  return 1.0 - smoothstep(halfWidth - 0.01, halfWidth + 0.01, d);
}

void main() {
  // Grid parameters
  float gridScale = 20.0;    // number of grid cells
  float lineWidth = 0.03;    // line thickness

  // Scale UVs to grid space
  vec2 gridUv = vUv * gridScale;

  // Compute grid lines in both axes
  float lineX = gridLine(gridUv.x, lineWidth);
  float lineY = gridLine(gridUv.y, lineWidth);
  float grid = max(lineX, lineY);

  // Base color: #00ff88 as vec3
  vec3 accentColor = vec3(0.0, 1.0, 0.533);

  // Mouse proximity glow: brighter near mouse position
  vec2 mouseUv = (uMouse + 1.0) * 0.5; // convert from [-1,1] to [0,1]
  float mouseDist = distance(vUv, mouseUv);
  float mouseGlow = smoothstep(0.4, 0.0, mouseDist) * 0.15;

  // Scroll-based subtle wave distortion (pulse traveling across grid)
  float wave = sin(vUv.y * 6.2832 + uTime * 0.5 + uScrollY * 3.0) * 0.02;

  // Edge fade: grid fades out toward edges of plane
  float edgeFade = smoothstep(15.0, 8.0, vDistFromCenter);

  // Combine: grid line alpha + mouse glow, modulated by edge fade
  float baseAlpha = 0.04;  // matches current CSS grid opacity
  float alpha = (grid * baseAlpha + mouseGlow + wave * 0.01) * edgeFade;

  gl_FragColor = vec4(accentColor, alpha);
}
```

### 5.3 Grid Plane Mesh Construction

```javascript
function createGridPlane() {
  // Large plane, subdivided for smooth vertex shader interpolation
  const geometry = new THREE.PlaneGeometry(40, 40, 1, 1);
  // Rotate to lie flat on XZ plane
  geometry.rotateX(-Math.PI * 0.5);

  const material = new THREE.ShaderMaterial({
    uniforms: {
      uTime:    { value: 0.0 },
      uMouse:   { value: new THREE.Vector2(0, 0) },
      uScrollY: { value: 0.0 }
    },
    vertexShader: GRID_VERTEX_SHADER,   // string from embedded <script> or template literal
    fragmentShader: GRID_FRAGMENT_SHADER,
    transparent: true,
    depthWrite: false,
    side: THREE.DoubleSide
  });

  const gridMesh = new THREE.Mesh(geometry, material);
  bgScene.add(gridMesh);

  return material; // return ref to update uniforms in animation loop
}
```

---

## 6. Particle System (400 Particles with Drift)

### 6.1 Particle Geometry

```javascript
function createParticleSystem() {
  const PARTICLE_COUNT = 400;
  const geometry = new THREE.BufferGeometry();

  // Position: random distribution in a box volume above the grid
  const positions = new Float32Array(PARTICLE_COUNT * 3);
  const velocities = new Float32Array(PARTICLE_COUNT * 3); // stored for animation
  const sizes = new Float32Array(PARTICLE_COUNT);

  for (let i = 0; i < PARTICLE_COUNT; i++) {
    const i3 = i * 3;
    positions[i3]     = (Math.random() - 0.5) * 30;  // x: spread across grid
    positions[i3 + 1] = Math.random() * 8 + 0.5;      // y: 0.5 to 8.5 above grid
    positions[i3 + 2] = (Math.random() - 0.5) * 30;  // z: spread across grid

    // Slow drift velocities
    velocities[i3]     = (Math.random() - 0.5) * 0.003;
    velocities[i3 + 1] = (Math.random() - 0.5) * 0.002;
    velocities[i3 + 2] = (Math.random() - 0.5) * 0.003;

    sizes[i] = Math.random() * 2.0 + 0.5; // varied sizes
  }

  geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
  geometry.setAttribute('aSize', new THREE.BufferAttribute(sizes, 1));

  // Store velocities on geometry for access in animation loop
  geometry.userData.velocities = velocities;

  const material = new THREE.PointsMaterial({
    color: 0x00ff88,
    size: 1.5,
    sizeAttenuation: true,
    transparent: true,
    opacity: 0.3,
    depthWrite: false,
    blending: THREE.AdditiveBlending
  });

  const points = new THREE.Points(geometry, material);
  bgScene.add(points);

  return points;
}
```

### 6.2 Particle Animation (called each frame)

```javascript
function animateParticles(points, deltaTime) {
  const positions = points.geometry.attributes.position.array;
  const velocities = points.geometry.userData.velocities;
  const count = positions.length / 3;

  for (let i = 0; i < count; i++) {
    const i3 = i * 3;
    positions[i3]     += velocities[i3];
    positions[i3 + 1] += velocities[i3 + 1];
    positions[i3 + 2] += velocities[i3 + 2];

    // Boundary wrap: if particle drifts too far, wrap to opposite side
    if (positions[i3] > 15)      positions[i3] = -15;
    if (positions[i3] < -15)     positions[i3] = 15;
    if (positions[i3 + 1] > 9)   positions[i3 + 1] = 0.5;
    if (positions[i3 + 1] < 0.5) positions[i3 + 1] = 9;
    if (positions[i3 + 2] > 15)  positions[i3 + 2] = -15;
    if (positions[i3 + 2] < -15) positions[i3 + 2] = 15;
  }

  points.geometry.attributes.position.needsUpdate = true;
}
```

---

## 7. Hero Wireframe Icosahedron

### 7.1 Geometry and Material

```javascript
function createHeroIcosahedron() {
  // IcosahedronGeometry(radius, detail)
  // detail=1 gives 80 faces -- enough visual complexity without being heavy
  const geometry = new THREE.IcosahedronGeometry(1.5, 1);

  const material = new THREE.MeshBasicMaterial({
    color: 0x00ff88,
    wireframe: true,
    transparent: true,
    opacity: 0.25
  });

  const mesh = new THREE.Mesh(geometry, material);
  heroScene.add(mesh);

  return mesh;
}
```

### 7.2 Icosahedron Animation

```javascript
function animateIcosahedron(mesh, time, scrollY) {
  // Gentle dual-axis rotation
  mesh.rotation.x = time * 0.15;
  mesh.rotation.y = time * 0.2;

  // Scroll-linked: slight tilt as user scrolls
  mesh.rotation.z = scrollY * 0.3;

  // Subtle breathing scale
  const breathe = 1.0 + Math.sin(time * 0.8) * 0.03;
  mesh.scale.setScalar(breathe);
}
```

### 7.3 Fade-In on Load

```javascript
function fadeInHeroCanvas() {
  const canvas = document.getElementById('hero-icosahedron-canvas');
  let opacity = 0;

  function fade() {
    opacity += 0.02;
    canvas.style.opacity = Math.min(opacity, 0.6); // max opacity 0.6 -- stays subtle
    if (opacity < 0.6) requestAnimationFrame(fade);
  }

  // Delay 500ms after page load for perceived polish
  setTimeout(fade, 500);
}
```

---

## 8. Animation Loop Structure

### 8.1 Unified Loop with Visibility Gating

```javascript
let animationFrameId = null;
let gridMaterial = null;
let particleSystem = null;
let icosahedronMesh = null;
let isHeroVisible = true;

// Scroll state (updated by event listener)
let currentScrollY = 0;
let normalizedScroll = 0;

// Mouse state (updated by event listener)
let mouseX = 0;
let mouseY = 0;

function animate() {
  animationFrameId = requestAnimationFrame(animate);

  const time = performance.now() * 0.001; // seconds

  // FPS monitoring
  fpsMonitor.tick();

  // --- Scene A: Background (always renders) ---
  // Update grid shader uniforms
  gridMaterial.uniforms.uTime.value = time;
  gridMaterial.uniforms.uMouse.value.set(mouseX, mouseY);
  gridMaterial.uniforms.uScrollY.value = normalizedScroll;

  // Animate particles
  animateParticles(particleSystem, 0.016);

  // Render background
  bgRenderer.render(bgScene, bgCamera);

  // --- Scene B: Hero Icosahedron (only when hero is visible) ---
  if (isHeroVisible) {
    animateIcosahedron(icosahedronMesh, time, normalizedScroll);
    heroRenderer.render(heroScene, heroCamera);
  }
}

function startAnimationLoop() {
  gridMaterial = createGridPlane();
  particleSystem = createParticleSystem();
  icosahedronMesh = createHeroIcosahedron();

  fadeInHeroCanvas();
  animate();
}
```

### 8.2 Visibility Gating for Hero Scene

```javascript
// Use IntersectionObserver to pause hero rendering when scrolled past
const heroSection = document.querySelector('.hero');
const heroObserver = new IntersectionObserver(
  (entries) => {
    isHeroVisible = entries[0].isIntersecting;
    const heroCanvas = document.getElementById('hero-icosahedron-canvas');
    if (!isHeroVisible) {
      heroCanvas.style.opacity = '0';
    }
    // Fade back in handled by CSS transition
  },
  { threshold: 0.1 }
);
heroObserver.observe(heroSection);
```

---

## 9. Scroll and Mouse Interaction Handlers

### 9.1 Mouse Tracking (Throttled)

```javascript
// Mouse position normalized to [-1, 1] range
let mouseThrottleTimeout = null;

document.addEventListener('mousemove', function(e) {
  if (mouseThrottleTimeout) return;

  mouseThrottleTimeout = setTimeout(function() {
    mouseX = (e.clientX / window.innerWidth) * 2 - 1;
    mouseY = -(e.clientY / window.innerHeight) * 2 + 1;
    mouseThrottleTimeout = null;
  }, 16); // ~60fps throttle
});
```

### 9.2 Scroll Tracking

```javascript
// Scroll normalized to [0, 1] range based on document height
let scrollThrottleTimeout = null;

window.addEventListener('scroll', function() {
  if (scrollThrottleTimeout) return;

  scrollThrottleTimeout = setTimeout(function() {
    currentScrollY = window.scrollY;
    const maxScroll = document.documentElement.scrollHeight - window.innerHeight;
    normalizedScroll = maxScroll > 0 ? currentScrollY / maxScroll : 0;
    scrollThrottleTimeout = null;
  }, 16); // ~60fps throttle
}, { passive: true });
```

### 9.3 Resize Handler

```javascript
let resizeTimeout = null;

window.addEventListener('resize', function() {
  clearTimeout(resizeTimeout);
  resizeTimeout = setTimeout(function() {
    // Check if we should still be running Three.js
    if (window.innerWidth < 1024) {
      killThreeJS();
      return;
    }

    // Update background renderer
    bgCamera.aspect = window.innerWidth / window.innerHeight;
    bgCamera.updateProjectionMatrix();
    bgRenderer.setSize(window.innerWidth, window.innerHeight);

    // Hero renderer stays 500x500, no resize needed
  }, 200); // 200ms debounce for resize
}, { passive: true });
```

---

## 10. Mobile Optimization Strategy

### 10.1 Zero-Cost Mobile Path

Mobile devices (viewport < 1024px) receive **zero JavaScript execution** for 3D:

- The `shouldEnableThreeJS()` gate returns `false` immediately
- The Three.js library still loads (defer, non-blocking) but is never initialized
- The CSS `.background-grid` remains visible with its existing grid pattern
- No WebGL contexts are created
- No animation frames are requested

**Optimization**: To avoid even loading the Three.js library on mobile, wrap the
CDN script tag in a conditional:

```html
<script>
  // Only inject Three.js script tag on desktop-class devices
  if (window.innerWidth >= 1024) {
    var s = document.createElement('script');
    s.src = 'https://cdnjs.cloudflare.com/ajax/libs/three.js/r169/three.min.js';
    s.defer = true;
    s.onload = function() { window.threeJSReady = true; };
    document.head.appendChild(s);
  }
</script>
```

This saves ~160KB on mobile connections.

### 10.2 CSS Fallback Enhancement

Keep the existing CSS grid but add a subtle CSS-only enhancement for mobile:

```css
/* Enhanced mobile grid -- still pure CSS, no JS */
@media (max-width: 1023px) {
  .background-grid {
    background-image:
      linear-gradient(rgba(0, 255, 136, 0.03) 1px, transparent 1px),
      linear-gradient(90deg, rgba(0, 255, 136, 0.03) 1px, transparent 1px);
    background-size: 40px 40px; /* tighter grid on smaller screens */
  }
}
```

### 10.3 Hero Section on Mobile

The hero icosahedron canvas is hidden entirely on mobile via:

```css
@media (max-width: 1023px) {
  #hero-icosahedron-canvas {
    display: none !important;
  }
}
```

No JS detection needed -- pure CSS media query prevents any visual artifact.

---

## 11. CSS Additions Required

### 11.1 New Styles to Add

```css
/* Three.js canvas transition for smooth fallback */
#haze-bg-canvas {
  transition: opacity 0.5s ease-in-out;
}

#hero-icosahedron-canvas {
  transition: opacity 0.8s ease-in-out;
}

/* Ensure hero section has relative positioning for canvas overlay */
.hero {
  position: relative; /* already set in current CSS */
  overflow: hidden;   /* clip icosahedron at hero boundaries */
}

/* Hide Three.js canvases on mobile (belt-and-suspenders with JS) */
@media (max-width: 1023px) {
  #haze-bg-canvas,
  #hero-icosahedron-canvas {
    display: none !important;
  }
}
```

### 11.2 Existing CSS Modifications

The `.background-grid` class needs an `id` attribute added (`id="css-grid-fallback"`)
to the existing `<div class="background-grid">` element. No CSS changes to the class
itself.

---

## 12. Complete Implementation File Map

All changes to `D:\Personal\ht1-client\ht1-web\index.html`:

### Order of Changes

| # | Location | Change |
|---|----------|--------|
| 1 | Line 64-76 (.background-grid CSS) | Add `#css-grid-fallback` styles and mobile media query |
| 2 | Line 153-163 (.hero CSS) | Add `overflow: hidden` |
| 3 | After line 1709 (end of `</style>`) | Add Three.js canvas CSS |
| 4 | Line 1714 (`<div class="background-grid">`) | Add `id="css-grid-fallback"`, add canvas element before it |
| 5 | Line 1735-1750 (hero section) | Add hero icosahedron canvas |
| 6 | Before line 2278 (`<script>`) | Add conditional Three.js CDN loader |
| 7 | Replace line 2278-2288 (existing script) | Expanded script with smooth scroll + all Three.js code |

### Estimated Code Additions

| Component | Lines of JS | Lines of CSS |
|-----------|-------------|-------------|
| Device detection | ~35 | 0 |
| Scene A (background) | ~60 | 5 |
| Grid shader (GLSL strings) | ~50 | 0 |
| Particle system | ~55 | 0 |
| Scene B (hero) | ~35 | 5 |
| Animation loop | ~40 | 0 |
| Event handlers | ~45 | 0 |
| FPS monitor + killswitch | ~50 | 0 |
| Mobile optimization | ~10 | 10 |
| **Total** | **~380** | **~20** |

This brings the file from ~2291 lines to ~2691 lines. Acceptable for a single-file
deployment with no build system.

---

## 13. Performance Budget Analysis

### Load Time Breakdown

| Resource | Size (gzipped) | Load Time (3G) | Load Time (4G) |
|----------|----------------|-----------------|-----------------|
| index.html (current) | ~18KB | 200ms | 50ms |
| index.html (with additions) | ~25KB | 280ms | 70ms |
| Three.js r169 (CDN, cached) | ~160KB | 1.8s | 450ms |
| Fonts (Google Fonts) | ~40KB | 450ms | 110ms |
| logo.png | ~15KB | 170ms | 40ms |
| **Total (4G, cold cache)** | **~240KB** | -- | **~720ms** |
| **Total (3G, cold cache)** | **~240KB** | **~2.9s** | -- |

Note: Three.js loads via `defer` and does not block first paint. The CSS fallback
grid is visible immediately. Three.js initialization happens post-load, so perceived
load time is unaffected.

### Runtime Performance

| Metric | Target | Expected |
|--------|--------|----------|
| Background render (1080p) | <8ms/frame | ~4ms |
| Particle update (400 pts) | <2ms/frame | ~0.5ms |
| Hero render (500x500) | <2ms/frame | ~1ms |
| Total per frame | <16ms (60fps) | ~5.5ms |
| GPU memory | <50MB | ~25MB |

### Why These Numbers Work

- The grid shader is a single full-screen quad with simple math -- no textures, no
  complex branching. ~0.5ms on integrated GPUs.
- 400 particles is trivial for any GPU made after 2015. The CPU-side position update
  loop touches 1200 floats -- measured in microseconds.
- The icosahedron wireframe is 80 triangles. Negligible.
- The heaviest cost is the Two `WebGLRenderer.render()` calls themselves, which
  include state setup and draw call overhead. Even so, <6ms total is conservative.

---

## 14. Trade-offs and Decisions

### What We Are Optimizing For

1. **Visual impact** -- the shader grid + particles create depth and life
2. **Performance safety** -- multiple fallback layers ensure no user gets a bad experience
3. **Implementation simplicity** -- no build system, no modules, inline everything

### What We Are Sacrificing

1. **Code organization** -- everything inline in one file is not ideal for maintenance
2. **Advanced effects** -- no post-processing, no bloom, no reflections (would blow budget)
3. **Mobile parity** -- mobile users get the CSS fallback, which is fine but not special

### Why Not...

| Alternative | Reason Rejected |
|-------------|-----------------|
| Single renderer with scissor | More complex code, coupled lifecycles |
| Post-processing (bloom) | Doubles GPU cost, breaks 60fps on integrated GPUs |
| More than 400 particles | Diminishing returns; 400 is visible without being GPU-heavy |
| Custom particle shader | PointsMaterial with additive blending achieves the effect; custom shader adds complexity for marginal gain |
| IcosahedronGeometry detail=2 | 320 faces looks the same as 80 in wireframe mode at this distance |
| WASM/WebGPU | Overkill; WebGL1 is sufficient and universally supported |

---

## 15. Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Three.js CDN down | Low | Medium | CSS fallback always present; conditional loader means mobile never notices |
| GPU driver crash | Very Low | High | FPS monitor kills Three.js after 3s of <30fps; page remains fully functional |
| Memory leak over time | Medium | Medium | No dynamic object creation in animation loop; all allocations at init |
| Scroll jank from event listeners | Medium | Medium | All handlers throttled to 16ms; scroll listener is passive |
| Browser blocks WebGL | Low | Low | `shouldEnableThreeJS()` catches this; CSS fallback used |
| Large viewport (4K) | Medium | Low | `setPixelRatio(min(dpr, 2))` caps render resolution |

---

## 16. Implementation Order

**Phase 1** (Foundation):
1. Add canvas elements and CSS
2. Implement device detection and fallback logic
3. Create Scene A with grid shader only (no particles yet)
4. Verify CSS fallback still works on mobile

**Phase 2** (Polish):
5. Add particle system to Scene A
6. Create Scene B with hero icosahedron
7. Wire up mouse and scroll handlers
8. Implement FPS monitor and killswitch

**Phase 3** (Testing):
9. Test on Chrome, Firefox, Edge, Safari (macOS)
10. Test mobile fallback on iOS Safari, Chrome Android
11. Measure load time with Lighthouse
12. Profile GPU usage with Chrome DevTools

Tasks marked with [P] can run in parallel within each phase.

---

## 17. Shader Embedding Strategy

Since there is no build system, GLSL shaders are embedded as JavaScript template
literals:

```javascript
const GRID_VERTEX_SHADER = `
  varying vec2 vUv;
  varying float vDistFromCenter;

  void main() {
    vUv = uv;
    vDistFromCenter = length(position.xz);
    gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
  }
`;

const GRID_FRAGMENT_SHADER = `
  precision mediump float;

  uniform float uTime;
  uniform vec2 uMouse;
  uniform float uScrollY;

  varying vec2 vUv;
  varying float vDistFromCenter;

  float gridLine(float coord, float lineWidth) {
    float halfWidth = lineWidth * 0.5;
    float d = abs(fract(coord - 0.5) - 0.5);
    return 1.0 - smoothstep(halfWidth - 0.01, halfWidth + 0.01, d);
  }

  void main() {
    float gridScale = 20.0;
    float lineWidth = 0.03;

    vec2 gridUv = vUv * gridScale;
    float lineX = gridLine(gridUv.x, lineWidth);
    float lineY = gridLine(gridUv.y, lineWidth);
    float grid = max(lineX, lineY);

    vec3 accentColor = vec3(0.0, 1.0, 0.533);

    vec2 mouseUv = (uMouse + 1.0) * 0.5;
    float mouseDist = distance(vUv, mouseUv);
    float mouseGlow = smoothstep(0.4, 0.0, mouseDist) * 0.15;

    float wave = sin(vUv.y * 6.2832 + uTime * 0.5 + uScrollY * 3.0) * 0.02;
    float edgeFade = smoothstep(15.0, 8.0, vDistFromCenter);

    float baseAlpha = 0.04;
    float alpha = (grid * baseAlpha + mouseGlow + wave * 0.01) * edgeFade;

    gl_FragColor = vec4(accentColor, alpha);
  }
`;
```

---

## 18. Complete Initialization Code (Reference Implementation)

```javascript
// ==========================================================
// HAZE CLIENT -- Three.js Background System
// Architecture: Modified Hybrid (Council-Approved)
// ==========================================================

(function() {
  'use strict';

  // --- State ---
  var animFrameId = null;
  var bgRenderer, bgScene, bgCamera;
  var heroRenderer, heroScene, heroCamera;
  var gridMaterial, particles, icosahedron;
  var mouseX = 0, mouseY = 0;
  var normalizedScroll = 0;
  var isHeroVisible = true;
  var isActive = false;

  // --- FPS Monitor ---
  var fpsFrames = 0, fpsLastTime = 0, fpsLowCount = 0;

  function fpsMonitorTick() {
    fpsFrames++;
    var now = performance.now();
    if (now - fpsLastTime >= 1000) {
      var fps = fpsFrames;
      fpsFrames = 0;
      fpsLastTime = now;
      if (fps < 30) {
        fpsLowCount++;
        if (fpsLowCount >= 3) teardown();
      } else {
        fpsLowCount = 0;
      }
    }
  }

  // --- Detection ---
  function shouldEnable() {
    if (window.innerWidth < 1024) return false;
    try {
      var c = document.createElement('canvas');
      if (!(c.getContext('webgl2') || c.getContext('webgl'))) return false;
    } catch(e) { return false; }
    if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) return false;
    if (navigator.deviceMemory && navigator.deviceMemory < 4) return false;
    if (navigator.hardwareConcurrency && navigator.hardwareConcurrency < 4) return false;
    return true;
  }

  // --- Setup ---
  function init() {
    if (typeof THREE === 'undefined' || !shouldEnable()) return;

    document.getElementById('css-grid-fallback').style.display = 'none';
    document.getElementById('haze-bg-canvas').style.display = 'block';

    initBackground();
    initHero();
    initEvents();

    isActive = true;
    fpsLastTime = performance.now();
    loop();
  }

  function initBackground() {
    var canvas = document.getElementById('haze-bg-canvas');
    bgScene = new THREE.Scene();
    bgCamera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);
    bgCamera.position.set(0, 8, 12);
    bgCamera.lookAt(0, 0, 0);

    bgRenderer = new THREE.WebGLRenderer({ canvas: canvas, antialias: false, alpha: true, powerPreference: 'high-performance' });
    bgRenderer.setSize(window.innerWidth, window.innerHeight);
    bgRenderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    bgRenderer.setClearColor(0x000000, 0);

    // Grid plane
    var geo = new THREE.PlaneGeometry(40, 40, 1, 1);
    geo.rotateX(-Math.PI * 0.5);
    gridMaterial = new THREE.ShaderMaterial({
      uniforms: {
        uTime: { value: 0 },
        uMouse: { value: new THREE.Vector2(0, 0) },
        uScrollY: { value: 0 }
      },
      vertexShader: GRID_VERTEX_SHADER,
      fragmentShader: GRID_FRAGMENT_SHADER,
      transparent: true,
      depthWrite: false,
      side: THREE.DoubleSide
    });
    bgScene.add(new THREE.Mesh(geo, gridMaterial));

    // Particles
    var pCount = 400;
    var pGeo = new THREE.BufferGeometry();
    var pos = new Float32Array(pCount * 3);
    var vel = new Float32Array(pCount * 3);
    for (var i = 0; i < pCount; i++) {
      var i3 = i * 3;
      pos[i3]     = (Math.random() - 0.5) * 30;
      pos[i3 + 1] = Math.random() * 8 + 0.5;
      pos[i3 + 2] = (Math.random() - 0.5) * 30;
      vel[i3]     = (Math.random() - 0.5) * 0.003;
      vel[i3 + 1] = (Math.random() - 0.5) * 0.002;
      vel[i3 + 2] = (Math.random() - 0.5) * 0.003;
    }
    pGeo.setAttribute('position', new THREE.BufferAttribute(pos, 3));
    pGeo.userData.velocities = vel;
    var pMat = new THREE.PointsMaterial({
      color: 0x00ff88, size: 1.5, sizeAttenuation: true,
      transparent: true, opacity: 0.3, depthWrite: false,
      blending: THREE.AdditiveBlending
    });
    particles = new THREE.Points(pGeo, pMat);
    bgScene.add(particles);
  }

  function initHero() {
    var canvas = document.getElementById('hero-icosahedron-canvas');
    if (!canvas) return;
    canvas.style.display = 'block';

    heroScene = new THREE.Scene();
    heroCamera = new THREE.PerspectiveCamera(50, 1, 0.1, 100);
    heroCamera.position.set(0, 0, 4);

    heroRenderer = new THREE.WebGLRenderer({ canvas: canvas, antialias: true, alpha: true, powerPreference: 'high-performance' });
    heroRenderer.setSize(500, 500);
    heroRenderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    heroRenderer.setClearColor(0x000000, 0);

    var geo = new THREE.IcosahedronGeometry(1.5, 1);
    var mat = new THREE.MeshBasicMaterial({ color: 0x00ff88, wireframe: true, transparent: true, opacity: 0.25 });
    icosahedron = new THREE.Mesh(geo, mat);
    heroScene.add(icosahedron);

    // Fade in
    setTimeout(function() {
      var op = 0;
      (function fade() {
        op += 0.02;
        canvas.style.opacity = Math.min(op, 0.6);
        if (op < 0.6) requestAnimationFrame(fade);
      })();
    }, 500);
  }

  // --- Events ---
  function initEvents() {
    var mThrottle = null;
    document.addEventListener('mousemove', function(e) {
      if (mThrottle) return;
      mThrottle = setTimeout(function() {
        mouseX = (e.clientX / window.innerWidth) * 2 - 1;
        mouseY = -(e.clientY / window.innerHeight) * 2 + 1;
        mThrottle = null;
      }, 16);
    });

    var sThrottle = null;
    window.addEventListener('scroll', function() {
      if (sThrottle) return;
      sThrottle = setTimeout(function() {
        var maxS = document.documentElement.scrollHeight - window.innerHeight;
        normalizedScroll = maxS > 0 ? window.scrollY / maxS : 0;
        sThrottle = null;
      }, 16);
    }, { passive: true });

    var rTimeout = null;
    window.addEventListener('resize', function() {
      clearTimeout(rTimeout);
      rTimeout = setTimeout(function() {
        if (window.innerWidth < 1024) { teardown(); return; }
        bgCamera.aspect = window.innerWidth / window.innerHeight;
        bgCamera.updateProjectionMatrix();
        bgRenderer.setSize(window.innerWidth, window.innerHeight);
      }, 200);
    }, { passive: true });

    // Hero visibility
    var heroEl = document.querySelector('.hero');
    if (heroEl && window.IntersectionObserver) {
      new IntersectionObserver(function(entries) {
        isHeroVisible = entries[0].isIntersecting;
        if (!isHeroVisible) {
          document.getElementById('hero-icosahedron-canvas').style.opacity = '0';
        }
      }, { threshold: 0.1 }).observe(heroEl);
    }
  }

  // --- Loop ---
  function loop() {
    if (!isActive) return;
    animFrameId = requestAnimationFrame(loop);
    var t = performance.now() * 0.001;

    fpsMonitorTick();

    // Background
    gridMaterial.uniforms.uTime.value = t;
    gridMaterial.uniforms.uMouse.value.set(mouseX, mouseY);
    gridMaterial.uniforms.uScrollY.value = normalizedScroll;

    var pa = particles.geometry.attributes.position.array;
    var pv = particles.geometry.userData.velocities;
    for (var i = 0, l = pa.length / 3; i < l; i++) {
      var j = i * 3;
      pa[j] += pv[j]; pa[j+1] += pv[j+1]; pa[j+2] += pv[j+2];
      if (pa[j] > 15) pa[j] = -15; if (pa[j] < -15) pa[j] = 15;
      if (pa[j+1] > 9) pa[j+1] = 0.5; if (pa[j+1] < 0.5) pa[j+1] = 9;
      if (pa[j+2] > 15) pa[j+2] = -15; if (pa[j+2] < -15) pa[j+2] = 15;
    }
    particles.geometry.attributes.position.needsUpdate = true;

    bgRenderer.render(bgScene, bgCamera);

    // Hero
    if (isHeroVisible && icosahedron) {
      icosahedron.rotation.x = t * 0.15;
      icosahedron.rotation.y = t * 0.2;
      icosahedron.rotation.z = normalizedScroll * 0.3;
      var s = 1.0 + Math.sin(t * 0.8) * 0.03;
      icosahedron.scale.setScalar(s);
      heroRenderer.render(heroScene, heroCamera);
    }
  }

  // --- Teardown ---
  function teardown() {
    isActive = false;
    if (animFrameId) cancelAnimationFrame(animFrameId);

    function disposeScene(scene) {
      if (!scene) return;
      scene.traverse(function(obj) {
        if (obj.geometry) obj.geometry.dispose();
        if (obj.material) {
          if (obj.material.map) obj.material.map.dispose();
          obj.material.dispose();
        }
      });
    }

    disposeScene(bgScene);
    disposeScene(heroScene);
    if (bgRenderer) bgRenderer.dispose();
    if (heroRenderer) heroRenderer.dispose();

    var bgC = document.getElementById('haze-bg-canvas');
    var hC = document.getElementById('hero-icosahedron-canvas');
    var css = document.getElementById('css-grid-fallback');
    if (bgC) bgC.style.display = 'none';
    if (hC) hC.style.display = 'none';
    if (css) css.style.display = 'block';
  }

  // --- Boot ---
  window.addEventListener('load', init);
})();
```

---

*End of specification. All code is ready for inline implementation into index.html.*
