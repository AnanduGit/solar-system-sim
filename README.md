# Solar System Simulation

A physically accurate, real-time 3D solar system simulation built with React and Three.js. Every distance, radius, and orbital element is true physical scale — no artistic exaggeration. The simulation runs a full N-body integrator with relativistic corrections, meaning bodies actually gravitationally interact with each other rather than following pre-baked Keplerian paths.

---

## Features

### Physics Engine

- **Yoshida 4th-order symplectic integrator** — replaces the naive RK4. Conserves a shadow Hamiltonian exactly, so there is no long-term energy drift. Orbits stay stable over centuries of simulation time.
- **Full N-body gravity** — every body gravitationally pulls every other body. Jupiter noticeably perturbs the inner planets. The Moon's orbit is a genuine 3-body problem with Earth and the Sun.
- **1PN general relativistic correction (Schwarzschild)** — Mercury's perihelion precesses at 43 arcseconds per century, matching the observed anomaly that Newton couldn't explain. Applied using the Newhall-Standish-Williams formulation from JPL DE405.
- **J2 oblateness perturbations** — Earth, Mars, Jupiter, and Saturn are modeled as oblate spheroids. Phobos's node precesses ~158°/year as a result. Uses Legendre P2 zonal harmonics.
- **Barycentric Sun** — the Sun is not fixed. Its position and velocity are computed from the barycenter constraint (Σ m·r = 0) so the solar system center of mass is the inertial origin.
- **ΔT correction** — the difference between Terrestrial Time and UTC (~69.2 seconds currently) is applied to all epoch calculations.

### Positional Accuracy

- **VSOP87** truncated heliocentric ecliptic series for all 8 planets — positions accurate to ~1 arcsecond over several centuries. Includes analytic velocity derivatives (not finite differences).
- **ELP2000 lunar theory** — 60-term series for the Moon's longitude, latitude, and distance. Accuracy ~0.02° vs ~0.3° for the common 20-term version.
- **Vis-viva Moon velocity** — ELP2000's angular velocity derivative has a known ~14% systematic error at certain orbital phases. The fix: ELP2000 gives accurate position; vis-viva (`v² = GM(2/r − 1/a)`) gives accurate speed; the orbital plane normal from the ascending node gives the prograde direction. This combination is exact for a Keplerian orbit and accurate to ~1% for the real perturbed Moon.
- **JPL mean orbital elements** (J2000) for all planets with secular rates per century.
- **Phobos and Deimos nodal precession** — secular dΩ/dt and dω/dt rates applied directly to Keplerian initialization.

### Bodies Simulated

**Planets (N-body, VSOP87):** Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune

**Dwarf Planets (N-body, Keplerian):** Ceres, Pluto, Eris, Haumea, Makemake

**Moons (N-body, 40+ moons):**
- Earth: Moon (ELP2000 init)
- Mars: Phobos, Deimos
- Jupiter: Metis, Amalthea, Thebe, Io, Europa, Ganymede, Callisto, Himalia, Elara, Lysithea
- Saturn: Mimas, Enceladus, Tethys, Dione, Janus, Epimetheus, Rhea, Titan, Hyperion, Iapetus, Phoebe
- Uranus: Miranda, Ariel, Umbriel, Titania, Oberon
- Neptune: Proteus, Larissa, Triton, Nereid
- Pluto: Charon

**Small Bodies (Keplerian, massless):** 1P/Halley, 4 Vesta, 2 Pallas, 99942 Apophis, 3200 Phaethon

**Asteroid Belt:** 3,000 Keplerian test particles with realistic orbital distributions. Kirkwood gaps (2.50, 2.82, 2.96 AU) are excluded. Inclinations distributed 0–20°.

### True Physical Scale

The unit system is: **1 AU = 100 scene units**. Every radius and distance uses this scale without exception.

```
Sun:     695,700 km → 0.465   scene units
Earth:     6,371 km → 0.00426 scene units
Moon:      1,737 km → 0.00116 scene units
```

At this scale, Earth is 0.004% of its orbital radius from the Sun. The planets are genuinely tiny relative to the space between them — exactly as they are in reality.

### Rendering

- **Custom GLSL body shader** — every planet and moon uses a unified shader with per-pixel diffuse lighting and physically correct shadow casting.
- **Sphere-on-sphere shadows with penumbra** — the Sun is modeled as a finite disk (not a point source). Shadows have a true umbra (full dark) and penumbra (gradient). The Moon's umbra on Earth narrows to ~70 km — consistent with real solar eclipses. The old point-source approach inflated all shadow footprints to the occluder's full geometric radius.
- **Ring shadow casting** — rings cast density-modulated shadows on their parent planet. Saturn's Cassini Division transmits light; the B ring is nearly opaque.
- **Planet-on-ring shadows** — penumbra math applied for the planet's shadow falling across its own ring system.
- **Moon-on-ring shadows** — moons occlude rings with the same cone math (up to 14 moons per ring system).
- **Procedural planet textures** — all planets use generated canvas textures. Jupiter uses domain-warped Perlin noise with accurate belt/zone color maps, a Rankine vortex for the GRS, differential rotation, festoons, and the three White Ovals. Saturn has the same band structure in the ring system (C ring / B ring / Cassini Division / A ring with Encke gap).
- **Physically structured rings** — Saturn, Uranus, Jupiter, Neptune each have their own ring density function modeled on real structure. Uranus has all 9 named narrow rings (ε dominant). Neptune has Galle, Le Verrier, and Adams rings.

### Visual Features

- Orbit trails for all bodies
- Procedural star field (6,000 stars)
- Sun glow sprite (additive blending)
- Earth axial tilt + daily rotation
- Comet nucleus, coma, dust tail, and ion tail for cometary bodies (tail points away from Sun)
- Pixel-size markers for bodies too small to resolve at current zoom
- HTML div label overlay for all bodies (toggleable)
- Focus dropdown with search — instantly jumps camera to any body
- Live orbital period measurement using forward simulation + angle accumulation
- Live distance readout (from Sun, or from parent for moons)

### Controls

| Action | Input |
|--------|-------|
| Orbit camera | Touch drag / Mouse drag |
| Zoom | Pinch / Scroll wheel |
| Focus body | Dropdown menu or search |
| Time speed | 7 speed steps |
| Pause / Play | Bottom bar button |
| Reset to now | ↺ button |
| Body labels | VIEW toggle in menu |

Time speeds range from real-time to decades per second.

---

## Technical Notes

### Coordinate System

Physics runs in heliocentric ecliptic J2000 coordinates (AU, days). Three.js is Y-up; the ecliptic plane is XZ. Conversion: `e2t(x, y, z) → [x·AU, z·AU, y·AU]`. All physics arithmetic stays in float64; positions are subtracted before uploading to shader uniforms to avoid float32 precision loss for close bodies (e.g. Phobos at 9,376 km from Mars).

### Integrator Stability

The maximum Yoshida substep is capped at 0.04 days. This is set by Phobos — the innermost moon — which has a 0.319-day orbital period. At DT_MAX = 0.04 days there are ~8 steps per Phobos orbit, keeping ω·Δt ≈ 0.79 rad, within the symplectic integrator's stability region.

### Janus and Epimetheus

These two Saturn co-orbital moons are initialized with their real semi-major axes (2.7140 × 10⁻³ AU and 2.7180 × 10⁻³ AU — differing by ~50 km). The N-body integrator will naturally reproduce their ~4-year horseshoe orbit swap without any special-casing.

### Known Accuracy Ceiling

The simulation sits at approximately **65/100 accuracy** against DE441. The main limiting factors are:

- Truncated VSOP87 (amplitude cutoff ~5×10⁻⁷ rad) — sufficient for centuries but not spacecraft navigation
- ELP2000 60-term truncation vs full Chapront-Touzé series
- No tidal dissipation (Moon spiraling outward ~3.8 cm/yr)
- No radiation pressure (relevant for comets)
- No Yarkovsky effect (relevant for small asteroids like Apophis)
- Keplerian initialization for outer moon systems introduces small initial errors that the N-body then propagates

The roadmap to ~80/100 would require: full VSOP87 term count, IERS Earth orientation parameters, and DE441-seeded initial conditions from JPL Horizons.

---

## Stack

- **React** (hooks only — `useEffect`, `useRef`, `useState`)
- **Three.js r128** — geometry, rendering, shaders
- **Custom GLSL** — body shader, ring shader
- Single `.jsx` file, no build step required beyond a standard React environment

---

## Running

Drop `solar_system.jsx` into any React project that has Three.js available:

```bash
npm install three
```

Then import and render the default export:

```jsx
import SolarSystem from './solar_system';

function App() {
  return <SolarSystem />;
}
```

The component is self-contained and full-screen. It expects to be the only thing on the page.

---

## Data Sources

- **Orbital elements:** JPL Solar System Dynamics — mean elements J2000
- **VSOP87:** Bretagnon & Francou (1988) via Meeus *Astronomical Algorithms*
- **ELP2000:** Chapront-Touzé & Chapront (1982, 1988)
- **GR correction:** Newhall, Standish & Williams (1983) — JPL DE405 formulation
- **Moon masses and radii:** NASA/JPL planetary fact sheets
- **Ring structure:** NASA ring node data + Cuzzi et al.
- **Apophis elements:** JPL Small-Body Database (SBDB)
