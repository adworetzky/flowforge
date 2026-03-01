# FlowForge — Claude Session Notes

## Architecture

Single self-contained file: `index.html` (~1300 lines). No build step, no dependencies, no package.json. Open directly in a browser. All JS, CSS, HTML, and GLSL shaders are inline.

**Rendering pipeline (WebGL2):**
1. Curl → Vorticity (+ gravity bias) → Divergence → Pressure solve (Jacobi) → Gradient subtract → Advection (velocity then dye) → Display

**Key globals:** `gl`, `canvas`, `config`, `programs`, `mouse`, `touchPoints`, `velocity`, `dye`, `curl`, `pressure`, `divergence`

**Config object** drives all simulation parameters; sliders in `initUI()` write directly to it. `config.paused` gates the `step()` call in the animation loop.

---

## WebGL Gotchas

### EXT_color_buffer_float is mandatory
The simulation uses half-float FBOs (RGBA16F, RG16F, R16F). Without `EXT_color_buffer_float`, framebuffer status is `FRAMEBUFFER_INCOMPLETE_ATTACHMENT` and all render-to-texture silently no-ops — the simulation produces nothing. Check immediately after acquiring the extension:
```js
const ext = gl.getExtension('EXT_color_buffer_float');
if (!ext) { /* show error, throw */ }
```

### Uniform array caching — mobile driver quirk
`createProgram()` auto-caches uniforms via `getActiveUniform`, but some mobile WebGL drivers (iOS Metal) only expose `palette[0]` as an active uniform for the `palette[4]` array, leaving `palette[1..3]` uncached as `undefined`. Calling `gl.uniform3f(undefined, ...)` is a silent no-op, causing fluid to render as black-on-black.

**Fix:** After creating the display program, explicitly cache all array element locations:
```js
for (let i = 0; i < 4; i++) {
    programs.display.uniforms[`palette[${i}]`] =
        gl.getUniformLocation(programs.display.program, `palette[${i}]`);
}
```
Never call `getUniformLocation` inside the render loop — it is a GPU sync point.

### Y-axis orientation
- Canvas/screen space: Y=0 at top, increases downward
- WebGL UV space: Y=0 at bottom, increases upward
- Splat point normalisation: `1.0 - y / canvas.height` (flips Y)
- Touch/mouse delta: `-mouse.dy` passed to splat (inverts screen-space delta)
- Gravity in vorticity shader: `vec2(0.0, -gravity) * dt` — negative Y pulls fluid downward visually

### DPR scaling
Canvas internal resolution = `window.innerWidth * min(devicePixelRatio, 2)`. Touch/mouse coordinates arrive in CSS pixels and must be scaled by `canvas.width / rect.width` before passing to `splat()`.

---

## Mobile-Specific Bugs (session learnings)

### The panel blocks the canvas
The 280px right-aligned control panel has `pointer-events: auto`. On a ~390px-wide phone in portrait mode it covers ~72% of the canvas, consuming all taps. On touch devices, collapse the panel at init:
```js
if ('ontouchstart' in window) {
    document.getElementById('panel').classList.add('collapsed');
}
```

### Touch handlers need `touchcancel`
OS interruptions (incoming calls, notification drawer) fire `touchcancel` not `touchend`. Without a `touchcancel` handler, `mouse.down` stays `true` permanently. Always mirror `touchend` logic in `touchcancel`.

### Dye splat radius matters for tap visibility
Radius `0.0005` (in normalised coords) creates a tiny Gaussian that dissipates below the display threshold (`t > 0.01`) in ~15 frames (0.25 s) with zero velocity. `0.0015` is the minimum for a tap to feel responsive. Radius for velocity splat is `config.viscosity * 0.01` (affects fluid force width, not dye size).

### Multi-touch tracking
Use a `Map` keyed by `touch.identifier` (set in `touchstart`, cleaned in `touchend`/`touchcancel`). Do not rely on `e.touches[0]` for multi-touch — use `e.changedTouches` in move/end handlers to process only the touches that actually changed.

---

## Shader Locations (line approximate, may shift with edits)

| Shader | Lines | Notes |
|--------|-------|-------|
| `baseVertex` | ~538 | Shared vertex shader; passes `vUv`, `vL`, `vR`, `vT`, `vB` |
| `advectionShader` | ~575 | Used for both velocity and dye; `dissipation` uniform controls fade |
| `vorticityShader` | ~619 | Applies curl force + gravity bias to velocity |
| `splatShader` | ~559 | Gaussian splat for velocity and dye; `radius` uniform controls width |
| `displayShader` | ~674 | Palette mapping, bloom, chromatic aberration, vignette, film grain |

---

## Gravity Implementation

Gravity lives in the **vorticity** shader (not advection) because that is where force terms are applied to the velocity field. The advection shader is used for both velocity and dye, so modifying it would apply gravity to dye transport as well.

Uniform: `gravity` (float, range −0.5 to 0.5, default 0.0)
Applied as: `fragColor += vec2(0.0, -gravity) * dt`
Passed in `step()` alongside the curl/vorticity uniforms.

---

## Common Mistakes to Avoid

- **Do not call `getUniformLocation` in the render loop.** Cache at init.
- **Do not use `blit(null)` with the clear shader before the display pass.** The display shader overwrites every pixel; the preceding clear is a wasted full-screen GPU pass.
- **Do not assume `getActiveUniform` exposes all array elements.** Explicitly cache `uniform[i]` for each array index.
- **Do not skip `touchcancel`.** Any touch handler that modifies state on `touchstart`/`touchend` must also handle `touchcancel`.
- **Do not test touch on desktop only.** The control panel blocking issue is invisible on desktop because the mouse can reach anywhere on screen regardless of the panel.
