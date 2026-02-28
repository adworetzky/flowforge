# FlowForge

**Interactive Generative Art Engine**

A beautiful WebGL2-based fluid simulation that creates stunning, interactive generative art. Click and drag to create flowing patterns with realistic fluid dynamics.

![FlowForge](https://img.shields.io/badge/WebGL2-Fluid%20Simulation-blue)

## Features

- **Real-time Fluid Dynamics** - Navier-Stokes based simulation running at 60 FPS
- **8 Stunning Palettes** - Cosmic, Neon, Ocean, Fire, Mono, Sunset, Aurora, Mint
- **Interactive Controls** - Adjust speed, viscosity, vorticity, gravity, bloom, and more
- **Export Options** - Save as PNG or @2x retina images
- **Keyboard Shortcuts**:
  - `Space` - Pause/resume
  - `R` - Reset simulation
  - `Q/E` or `C` - Previous/next palette
  - `S` - Screenshot

## Usage

Open `index.html` in any modern browser. Click and drag to create flow patterns.

### Controls

- **Mouse/Touch Drag** - Add fluid and dye at cursor
- **Control Panel** (top-right) - Adjust simulation parameters
- **Bottom Toolbar** - Quick access to reset, pause, palette selection, and export

### Parameters

| Parameter | Description |
|-----------|-------------|
| Speed | Simulation speed multiplier |
| Viscosity | How "thick" the fluid feels |
| Vorticity | Amount of swirl/curl in the fluid |
| Gravity | Direction and strength of gravity |
| Bloom | Glow intensity |
| Chromatic | RGB split/aberration effect |
| Vignette | Edge darkening |
| Trails | How long color persists |

## Technical Details

- Pure WebGL2 - no external dependencies
- GPU-accelerated fluid simulation
- Half-float textures for smooth gradients
- Curl noise for organic movement

## Browser Support

Requires WebGL2 (all modern browsers support this).

## License

MIT
