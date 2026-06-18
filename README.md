# orrery

A hand-built universe in a single HTML file — real gravitational physics, rendered in real time on WebGL2, tuned to be genuinely beautiful on an OLED display.

Open `orrery.html` in a modern browser (Chrome, Firefox, Safari, Edge). No build step, no dependencies.

## What it is

Nothing here is on rails. Bodies move under Newtonian gravity (velocity-Verlet),
particles are integrated on the GPU by the hundreds of thousands via transform
feedback, and the black hole bends light along real null geodesics. You can
drag to orbit, scroll to zoom, click a world to follow it, and fling new worlds
into existing systems and watch their predicted trajectories.

## Scales (press **S** to travel between them)

- **Solar** — the Sun and its worlds: Mercury through Neptune, the Galilean
  moons and Earth's Moon, ringed Saturn and a tipped, ring-girdled Uranus, a
  main asteroid belt and a far Kuiper belt, and a long-period comet on a steep
  eccentric plunge. Planets can perturb the belt (n-body); worlds can collide
  and merge.

- **Galaxy** — a differentially-rotating stellar disk on a *flat rotation
  curve* (a dark-matter halo term supplies the missing gravity), a luminous
  nucleus, and density-wave spiral arms lit like HII regions.

- **Black hole** — a Schwarzschild black hole with a ray-marched accretion
  disk, gravitational lensing, photon ring and shadow, and relativistic
  pericenter precession of the orbiting stars.

- **Cosmic web** — *structure formation, running.* A near-uniform field of
  particles in an expanding universe: gravity pulls matter toward a jittered
  lattice of dark-matter halos while a Hubble term (tuned to roughly cancel the
  mean infall, as in a comoving frame) lets the density *fluctuations* grow.
  Clusters condense, filaments bridge them, and voids open between — the
  cosmic web, assembled in front of you from gravity and expansion alone.

## Controls

- **drag** orbit · **scroll / pinch** zoom · **click** follow a body · **Esc** stop following
- **S** change scale · **F** fling worlds (then **1/2/3** comet / world / giant)
- **space** pause · **[ ]** slower / faster · **T** trails · **G** belt self-gravity
- **B** star ↔ black hole · **K** relativistic gravity · **C** particle count · **R** reset
- **copy / load** serialize the whole universe to a seed string and share it

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
