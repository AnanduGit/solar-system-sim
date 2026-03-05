# 🌍 Solar System Simulator

A real-time, physically accurate 3D visualization of the solar system built with React and Three.js. Watch planets, dwarf planets, and moons orbit the Sun with proper gravitational dynamics and realistic shadows.

## Features

### 🎯 Core Functionality
- **Accurate Orbital Mechanics**: JPL Mean Orbital Elements (J2000) with century-based calculations for all major planets and dwarf planets
- **N-Body Physics**: RK4 (4th-order Runge-Kutta) integration for realistic gravitational interactions
- **Realistic Shadows**: Physically accurate shadow rendering for:
  - Sphere-to-sphere shadows (planets casting shadows on each other)
  - Ring shadows (Saturn and Uranus rings with umbra/penumbra)
- **Interactive Camera Controls**: Spherical coordinate system with zoom, pan, and focus capabilities
- **Orbital Trails**: Visualize planetary paths with configurable trail length (300 bodies maximum)
- **Multi-Body Support**: 
  - 8 classical planets (Mercury through Neptune)
  - 5 dwarf planets (Pluto, Ceres, Eris, Makemake, Haumea)
  - 3 moons (Earth's Moon, Phobos, Deimos)

### 🎨 Visual Features
- Milky Way ESO panorama skybox
- Procedural planet texturing with base colors
- Real-time lighting from the Sun
- Ambient lighting for depth perception
- Exponential fog for atmospheric effect
- High-DPI display support

### ⚙️ Interactive Controls
- **Playback Speed**: 1× to 24× real-time speed (default: 1 day/frame)
- **Pause/Play**: Freeze simulation to examine positions
- **Time Seek**: Jump to specific dates
- **Focus Mode**: Center camera on any celestial body
- **Trail Display**: Toggle orbital paths on/off
- **Labels**: Show/hide planet and moon names
- **Surface View**: First-person perspective on planetary surfaces

## Technical Details

### Physics Engine

#### Constants
- **AU** (Astronomical Unit): 100 units = 1 AU
- **GM**: Standard gravitational parameter (2.9591220828559e-4)
- **DEG**: Degree to radian conversion (π/180)
- **TRAIL_N**: Maximum trail points per body (300)
- **SUN_R**: Solar radius in AU (0.465)

#### Orbital Elements (J2000 Epoch)
The simulation uses JPL MEAN ORBITAL ELEMENTS with per-century variations:
- **a0, da**: Semi-major axis and its change per century
- **e0, de**: Eccentricity and its change per century
- **i0, di**: Orbital inclination and its change per century
- **L0, dL**: Mean longitude and its change per century
- **w0, dw**: Argument of perihelion and its change per century
- **O0, dO**: Longitude of ascending node and its change per century

#### Kepler Solver
Uses iterative Kepler equation solver:
```
E = M + e·sin(M)
```
With Newton-Raphson iterations (80 iterations, 1e-13 tolerance) to find eccentric anomaly from mean anomaly.

#### Integration Method
**RK4 Integration** with adaptive time stepping:
1. Calculates N-body accelerations using gravitational law
2. Performs four stages of integration (k1, k2, k3, k4)
3. Updates positions and velocities each frame

#### Moon State Calculation
Special handling for Earth's Moon using geocentric JPL elements:
- Solves 3-dimensional orbital mechanics
- Accounts for nodal precession
- Returns position and velocity in Earth-centered frame

### Shader System

#### Body Shader (Unified Material)
All planets and moons use a single physically-based shader:
- **Vertex Shader**: Transforms coordinates and normals to world space
- **Fragment Shader**:
  - Computes diffuse lighting from Sun
  - Raycasts toward Sun to detect occlusions
  - Tests up to 16 sphere occluders (for planetary shadows)
  - Tests up to 4 ring planes (for ring shadows)
  - Returns pixel color with shadow factor (0.15–1.0 range)

#### Ring Shader
Special material for Saturn and Uranus rings:
- Renders transparent annulus geometry
- Calculates soft shadows from planet blocking sunlight
- Simulates umbra and penumbra regions
- Double-sided rendering with depth writing disabled

### Scene Graph
```
Scene
├── Ambient Light (0x224466, intensity 0.35)
├── Point Light (Sun at origin)
├── Camera (Perspective, 52° FOV)
├── Skybox (Milky Way ESO panorama)
├── Planet Group
│   ├── Sun (center)
│   ├── Planets (with trails)
│   ├── Dwarf Planets
│   └── Moons
└── Ring Groups (Saturn, Uranus)
```

## Usage

### Installation
```bash
npm install
npm start
```

### Dependencies
- **React**: UI state management
- **Three.js**: 3D rendering engine
- **Modern Browser**: WebGL 2.0 support required

### Keyboard/Mouse Controls
- **Mouse Drag**: Rotate camera
- **Scroll**: Zoom in/out
- **Click Planet**: Focus on body
- **Space**: Play/Pause
- **Speed Slider**: Adjust simulation speed
- **UI Buttons**: Toggle trails, labels, surface view

### State Management
The component uses React hooks for UI state:
- `displayDate`: Current simulation date
- `speed`: Playback speed (hours per frame)
- `paused`: Simulation pause state
- `focus`: Currently focused celestial body
- `showTrails`: Toggle orbital paths
- `showLabels`: Toggle name labels
- `surfaceView`: First-person planet view
- `menuOpen`: UI menu visibility

## Data Sources

### Orbital Elements
- **Source**: NASA JPL Horizons
- **Epoch**: J2000.0 (2000 January 1.5 TT)
- **Elements**: Mean orbital elements with per-century variations
- **Accuracy**: Valid for dates ±3000 years from epoch

### Body Properties
- **Planet Data**: NASA Planetary Fact Sheets
- **Dwarf Planets**: Minor Planet Center (MPC) and JPL Small-Body Database
- **Moon Data**: NASA Moon Fact Sheet and JPL Ephemeris

### Skybox
- **Milky Way**: ESO eso0932a panorama
- **License**: ESO Free License
- **Format**: 360° JPEG panorama encoded in base64

## Performance Considerations

### Optimization Techniques
1. **Single Unified Shader**: All bodies use one material, reducing draw calls
2. **Occlusion Arrays**: Pre-allocated arrays for shadow testing (no runtime allocation)
3. **Limited Trail Length**: 300 points per body prevents excessive memory
4. **Frustum Culling**: Three.js automatically culls off-screen objects
5. **PixelRatio Clamping**: Limited to 2× for high-DPI devices

### Typical Performance
- **FPS Target**: 60 fps at 1080p
- **Sphere Occluders**: 16 maximum
- **Ring Occluders**: 4 maximum
- **Total Bodies**: ~20 (8 planets + 5 dwarf planets + 3 moons + Sun)

## Browser Compatibility

| Browser | Version | Support |
|---------|---------|---------|
| Chrome/Edge | 90+ | ✅ Full |
| Firefox | 88+ | ✅ Full |
| Safari | 14+ | ✅ Full |
| Mobile Safari | 14+ | ⚠️ Limited |

**Requirements**: WebGL 2.0, ES6+ JavaScript

## Future Enhancements

- [ ] Additional moons (Jupiter's Galilean moons, Saturn's major moons)
- [ ] Asteroid belt visualization
- [ ] Comet orbits
- [ ] Historical date presets
- [ ] Export orbital data
- [ ] VR mode support
- [ ] Improved mobile performance
- [ ] Customizable color schemes

## Known Limitations

1. **Pluto & Dwarf Planets**: Limited orbital element updates (some use fixed elements)
2. **Moon Accuracy**: Earth's Moon uses simplified geocentric model
3. **Solar Radiation Pressure**: Not included in dynamics
4. **Relativistic Effects**: Negligible for this scale
5. **Ring Particle Dynamics**: Rings are static geometry, not simulated particles

## License

This project is provided as-is for educational and visualization purposes.

## Credits

- **Orbital Mechanics**: NASA JPL Horizons System
- **3D Rendering**: Three.js community
- **Physics Integration**: RK4 numerical integration method
- **Skybox**: ESO (European Southern Observatory)
- **React Hooks Pattern**: Modern React best practices

---

**Made with ❤️ for space enthusiasts and physics nerds everywhere** 🚀✨