# orrery

A hand-built universe in a single HTML file — real gravitational physics, rendered in real time on WebGL2, tuned to be genuinely beautiful on an OLED display.

Open `orrery.html` in a modern browser (Chrome, Firefox, Safari, Edge). No build step, no dependencies.

## What it is

Nothing here is on rails. Bodies move under Newtonian gravity (velocity-Verlet),
particles are integrated on the GPU by the hundreds of thousands via transform
feedback, and the black hole bends light along real null geodesics. You can
drag to orbit, scroll to zoom, click a world to follow it, and fling new worlds
into existing systems and watch their predicted trajectories.

## Scales

A single continuous zoom (scroll / pinch) travels the whole ladder — from a living
cell (~10⁻⁴ m) out to the cosmic web (~10²⁵ m) — with no hard cut. Pressing **S**
jumps between *scenes* (different things to find at the solar scale).

Zoom **in** past the planets and the ladder turns inward:

- **A Living World** — a procedural planet: oceans, continents, weather, a terminator
  with city lights on the night side, and the thin blue line of an atmosphere; it
  fills the view as you descend toward the surface.
- **The Biosphere** — life itself, rendered as a continuous cellular automaton
  (Lenia) on a GPU float field: *Orbium* gliders — true self-propelled organisms
  that swim, turn and reproduce. The living world's ocean dissolves directly into
  this pond as you descend. Zoom to explore.

### The Cosmic Calendar

A scrubbable bar across the bottom compresses all 13.8 billion years into a single
year (Sagan's *Cosmic Calendar*). Drag the playhead — the date and the era's headline
event update live ("Nov 7 · photosynthesis", "Dec 31 · 11:59 pm · now"). Rewind past
the origin of life and the biosphere empties; scrub forward and it fills back in. Left
alone it simply runs toward *now*.

### Recurrence

Cross the threshold between the **stellar neighbourhood** and the **galaxy** and a single
cloud of particles performs a *match-cut*: a starling **flock** (murmuration) gathers into
a **globular cluster**, which unwinds into a two-armed **spiral galaxy** — the same forms
returning at ever-larger scale, the simulation's recurring visual rhyme.

Scenes (press **S**):

- **Solar** — the Sun and its worlds: Mercury through Neptune, the Galilean
  moons and Earth's Moon, ringed Saturn and a tipped, ring-girdled Uranus, a
  main asteroid belt and a far Kuiper belt, and a long-period comet on a steep
  eccentric plunge. Planets can perturb the belt (n-body); worlds can collide
  and merge.

- **Nursery** — *cosmic ethology.* A star-forming region where objects behave:
  stars run a lifecycle (protostar → main sequence → red giant → **supernova**
  or **planetary nebula**), each death seeding the dust that births the next;
  a binary pair **inspirals**, speeding up to a **chirp** and merging into a
  black hole; massive deaths leave black holes behind. Turn on sound for the
  booms and chirps.

- **Galaxy** — a differentially-rotating stellar disk on a *flat rotation
  curve* (a dark-matter halo term supplies the missing gravity), a luminous
  nucleus, and density-wave spiral arms lit like HII regions.

- **Black hole** — *feeding.* A Schwarzschild black hole with a ray-marched accretion
  disk, gravitational lensing, photon ring and shadow, and relativistic
  pericenter precession of the orbiting stars.

- **Cosmic web** — *structure formation, running.* A near-uniform field of
  particles in an expanding universe: gravity pulls matter toward a jittered
  lattice of dark-matter halos while a Hubble term (tuned to roughly cancel the
  mean infall, as in a comoving frame) lets the density *fluctuations* grow.
  Clusters condense, filaments bridge them, and voids open between — the
  cosmic web, assembled in front of you from gravity and expansion alone.

## Controls

- **drag** orbit · **scroll / pinch** travel through scale · **click** follow a body · **Esc** stop following
- **S** change system · **M** top-down "map" view · **F** fling worlds (then **1/2/3** comet / world / giant)
- **space** pause · **[ ]** slower / faster · **T** trails · **G** belt self-gravity
- **B** star ↔ black hole · **K** relativistic gravity · **C** particle count · **R** reset
- **drag the calendar bar** scrub through cosmic time (13.8 Gyr compressed to one year)
- **copy / load** serialize the whole universe to a seed string and share it

The HUD shows the current scale — `logScale` (log₁₀ of metres across the screen), a
human-readable span ("67 AU", "stellar neighbourhood"), bodies, particles and fps.

## How it works

Everything is one file. The simulation core:

- **Big bodies** (suns, planets, moons, halos) integrate on the CPU with
  velocity-Verlet and pairwise gravity, including a relativistic correction
  near black-hole horizons and inelastic merging.
- **Particle fields** (belts, disks, the cosmic web — up to ~1,000,000 points)
  integrate on the GPU in a transform-feedback shader, attracted by up to 64
  massive bodies, with optional dark-matter-halo and Hubble-expansion terms.
- **Rendering** is HDR into a float-ish target with threshold bloom, ACES tone
  mapping, subtle filmic grain and vignette, a procedural nebula sky, and a
  full-screen geodesic integrator for the black hole's lensed background.

Tunable physics constants live at the top of each preset in `buildScene()`.

## Scale architecture (the bones for a continuous multi-scale zoom)

The camera's defining state is a single scalar, **`logScale`** = log₁₀ of the metres
visible across one screen-height — the scale "protagonist". Rendering is organised
into **bands**: self-contained sim+render modules, each alive only near its scale.

```js
band = { name, logRange:[lo,hi], localUnit /* metres per local unit */, update(dt,rate), render(alpha) }
```

The renderer owns the camera, the shared background and the post chain; each frame it
derives every band's cross-fade `alpha` from `logScale` vs its `logRange`, updates only
in-range bands, and draws them back-to-front into one shared HDR target. Two techniques
keep float32 honest across huge spans:

- **Per-band local frames** — each band renders in its own sane-magnitude units (the
  solar band in AU); the renderer scales its frame to the screen from `logScale`.
- **Floating origin** — geometry is drawn camera-relative (positions minus the camera
  anchor in the vertex shaders), so drawn coordinates stay near zero at any pan/zoom.

Three bands ship today, and a single continuous zoom (scroll / pinch) travels between
them with no hard cut:

- **`SolarBand`** — the orrery itself (AU frame): Mercury→Neptune, ringed Saturn and a
  tipped Uranus, the distinct Galilean moons (Io · Europa · Ganymede · Callisto) and
  Saturn's Rhea + hazy Titan, a Great Red Spot on the gas giants, plus belts and a
  comet. Click a world to follow it and zoom right up to it (even a single moon).
- **`OortBand`** — a faint shell of ~7,000 sleeping comets (AU frame) you pass through
  as the planets shrink away and the Sun becomes a lone point of light.
- **`NeighbourhoodBand`** — the local stellar neighbourhood (light-year frame); the Sun
  becomes one star in a field of ~3,000 neighbours coloured by spectral type, drifting
  with slow proper motion, with a handful of real nearby stars named (Sirius, Alpha
  Centauri, Vega, …) on projected labels.
- **`GalacticBand`** — a differentially-rotating spiral galaxy on a flat rotation
  curve (kiloparsec frame): density-wave arms that gently *breathe*, dust lanes along
  their inner edges, faint Hα (pink) glow in star-forming arm regions, and a luminous
  bulge — the Sun ~8 kpc out in the Orion Arm, marked by a "you are here" reticle.
  Press **M** for a face-on top-down "map" framing that slowly auto-orbits.
- **`CosmicWebBand`** — the largest structure there is (megaparsec frame): zoom out past
  the galaxy and the whole spiral becomes one mote in a lattice of ~24,000 galaxies —
  clusters piled in nodes, spirals strung along filaments, immense voids between, the
  Milky Way still marked at the centre. Galaxy distances run log-uniform from the Local
  Group out to ~400 Mpc, so the climb out of the galaxy is never empty: neighbours, then
  groups, then the grand web. The web's branching is the recurrence motif at its largest —
  the same form as a neuron, a river delta, the Lenia organisms far below.

All bands share the Sun as the common world origin, so the transition is pure scale
cross-fade. The chapter card names the dominant scale by how *centered* `logScale` sits
in each band's range ("The Stellar Neighbourhood", "The Galaxy", "The Cosmic Web"), and
the backdrop nebula dims as you leave the solar system. Adding the next scale is one more
`Band` with a declared `logRange`; the renderer loop needs no change.
