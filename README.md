# Research Study on Designing a Boring Tool Holder integrated with Tuned Mass Damper
# Boring Bar — GA-Driven Multi-Objective Optimization

Optimization of an anti-vibration **boring bar** (an internal-turning tool
with a tuned-mass absorber embedded in an internal cavity), using nested
genetic algorithms coupled to a finite-element / chatter-stability model.

Two notebooks are provided for the two physical configurations a boring bar
can operate in:

| Notebook | Configuration |
|----------|---------------|
| `OptimiseComplete.ipynb` | **Rotary bar** — the bar itself spins (self-rotating boring head) |
| `StaticBore.ipynb` | **Static bar** — the bar is clamped stationary; the workpiece rotates (classical lathe/turning-centre boring) |

Both notebooks share the same geometry, GA machinery, objectives, and
constraints — they differ only in the beam physics required by whether the
bar rotates about its own axis. `StaticBore.ipynb` is a simplified
derivative of `OptimiseComplete.ipynb` for the case where those rotation
effects don't exist.

## Common model

The bar is a 5-section cantilevered beam (tapered shank → hollow body →
absorber cavity → solid tip) with a tuned-mass absorber (a TC slug on a
spring/damper) inside the cavity. A finite-element / modal model predicts
the tip frequency response, which feeds a regenerative-chatter stability
criterion. Two nested GAs search the design:

- **Outer GA — NSGA-II**, 6 geometric design variables
  `x = [D_cav, L_cav, D_abs, L_abs, Z_TIP, D_body_in]`
  Objectives: maximize stability limit `alim`, maximize static stiffness,
  minimize total mass.
- **Inner GA**, single-objective, tunes the absorber's `[kd, cd]`
  (stiffness, damping) for each outer candidate geometry.

### Design variables

| Variable | Meaning |
|----------|---------|
| `D_cav`, `L_cav` | Absorber cavity diameter / length |
| `D_abs`, `L_abs` | Absorber (tuned mass) diameter / length |
| `Z_TIP` | Axial offset of the cavity from the free (cutting) end |
| `D_body_in` | Internal bore diameter through the main body |

### Objectives

- **Maximize** `alim` — chatter-free stability limit (depth of cut)
- **Maximize** static tip stiffness
- **Minimize** total mass

### Key constraints

- Absorber must fit inside the cavity (`D_abs ≤ D_cav`, `L_abs ≤ L_cav`)
- Absorber volume ≤ 70% of cavity volume
- Minimum 3 mm wall thickness (cavity and body bore)
- Minimum solid tip length
- Body bore ≤ cavity bore

## Pipeline (both notebooks)

| # | Section | Purpose |
|---|---------|---------|
| 1 | Fixed parameters | Material props, bar envelope, GA hyperparameters, variable bounds |
| 2 | Cross-section (`xsec`) | Area / inertia at any axial position for the 5-section geometry |
| 3 | FEM assembly (`build_MK`) | Beam FEM → stiffness & mass matrices (+ gyroscopic/translational-mass matrices, rotary only) |
| 4 | Modal reduction (`modal_reduce`) | Reduces to the lowest 6 modes for fast repeated evaluation |
| 5 | Absorber parameters | Absorber mass and axial location from cavity/absorber geometry |
| 6 | Modal FRF (`frf`) | Frequency-response solve, with/without the absorber |
| 7 | Directional coefficients | Direction-averaged cutting-force coefficients for the stability criterion |
| 8 | Stability limit (`alim_from_FRF`) | Critical chatter-free depth of cut from the FRF |
| 9 | Static stiffness & mass | Tip stiffness (reuses FEM `Kf`) and total bar+absorber mass |
| 10 | Stability lobe diagram | Mode-resolved depth-vs-RPM diagram (diagnostic, not a GA objective) |
| 10.5 | Absorber clearance check | Confirms the absorber mass doesn't collide with the cavity wall under vibration |
| 11 | GA operators | SBX crossover, polynomial mutation, binary tournament (shared by both GAs) |
| 12 | Inner GA | Tunes `[kd, cd]` to maximize `alim` |
| 13 | Feasibility repair | Clips/repairs a 6-gene chromosome to satisfy all geometric constraints |
| 14 | Individual evaluation | Full evaluation of one outer-GA candidate (calls inner GA + all physics) |
| 15 | Outer NSGA-II | Non-dominated sorting + crowding distance + elitist generational loop |
| 16 | Post-processing | Extracts result arrays, Pareto front, combined score |
| 17 | Plotting | Convergence history, parameter-vs-score panels, FRF/lobe diagrams, best-design geometry |
| 18 | Entry point | Runs the full optimization and generates the plots |

## Rotary vs. Static: what actually changes

`StaticBore.ipynb` sets the bar's own rotation speed `Omega_bar = 0` at all
times, which removes every effect that only exists because a beam spins
about its own axis:

| Effect | Rotary bar (`OptimiseComplete.ipynb`) | Static bar (`StaticBore.ipynb`) |
|--------|----------------------------------------|-----------------------------------|
| Gyroscopic coupling (`ρIp`) | Present | **Absent** |
| Rotating-frame translational Coriolis (`ρA`) | Present | **Absent** |
| Centrifugal softening of bending stiffness | Present | **Absent** |
| x/y cross-coupling | `Hxy ≠ 0` (rotation couples the two bending planes) | `Hxy ≡ 0` (axisymmetric static beam: x and y are independent and identical) |
| FEM matrices needed | `Mf, Kf, Gf, Ma` (mass, stiffness, gyroscopic kernel, translational-only mass) | `Mf, Kf` only |
| Modal quantities | Adds `Gm`, `Mam` (projected gyroscopic/translational-mass terms) | Neither needed |
| FRF solve | Coupled 2N(+2)-DOF system, function of bar `Omega` | Uncoupled N(+1)-DOF system, **no `Omega` argument at all** |
| Per-individual speed sweep | `alim` averaged over a 0–6000 RPM sweep of the bar's own rotation (inner GA: 3 speeds; outer GA: 5 speeds) | **Single** static FRF solve, with and without the absorber — one design ⇒ one FRF |
| Absorber dynamics | Includes Coriolis + centrifugal terms on the lumped absorber mass | Plain spring-mass-damper, no extra terms |
| Workpiece rotation | Still governs regenerative chatter via phase delay `T = 60/RPM`, used identically in both notebooks for the stability-lobe diagram | Same — the workpiece is what regenerates chatter in both configurations |

Everything else — geometry, GA operators, feasibility repair, objective
definitions, constraint set, post-processing, and the best-design selection
— is identical between the two notebooks. Because the static case needs only
one FRF solve per design instead of a multi-speed sweep, it runs noticeably
faster for the same population/generation settings.

## How the "best" design is selected (both notebooks)

1. Every evaluated individual (not just the final Pareto front) is
   Pareto-ranked on `[-alim, -stiffness, mass]`.
2. Each of the three objectives is min–max normalized to `[0, 1]` across
   the whole population.
3. A **combined score = unweighted average of the three normalized
   objectives** is computed per individual.
4. The individual with `argmax(score)` is reported as the
   **"★ Best balanced"** design — distinct from the separate single-best
   individuals for `alim`, stiffness, and mass alone, which are also
   identified for reference.

This is a simple unweighted-average scalarization over the evaluated
population, not a weighted or preference-driven decision method — all three
objectives are treated as equally important.

## Requirements

```
numpy
scipy
matplotlib
```

## Running

Run all cells of either notebook in order (or execute as a script). The
entry-point cell runs the outer NSGA-II loop and produces a single
multi-panel figure summarizing convergence, the Pareto front, per-parameter
trends, FRF(s), the stability lobe diagram, and the recommended best
design's geometry and tuning.

- Use `OptimiseComplete.ipynb` to design a **self-rotating** boring bar.
- Use `StaticBore.ipynb` to design a **stationary** boring bar used in
  conventional lathe/turning-centre boring.
