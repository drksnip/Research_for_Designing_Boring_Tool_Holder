# Final Version of Multi Objective Optimisation for tuned mass damper in Rotary boring tool holder
This branch has integrated NSGA and GA searches for all the parameters of the project statement and selects best normalised score of all functions.

# Boring Bar — GA-Driven Multi-Objective Optimization

Optimization of an anti-vibration **boring bar** (an internal-turning tool with a
tuned-mass absorber embedded in an internal cavity) using nested genetic
algorithms coupled to a rotordynamic finite-element / stability model.

## What this notebook does

The bar is modeled as a 5-section cantilevered beam (tapered shank → hollow
body → absorber cavity → solid tip). A rotordynamic FEM/modal model predicts
its tip frequency response, which feeds a chatter-stability criterion. A
tuned-mass absorber sits inside a machined cavity; its spring/damping and the
bar's own geometry are optimized to push chatter-free cutting depth as high
as possible while keeping the tool stiff and light.

Two nested GAs do the search:

- **Outer GA — NSGA-II**, 6 geometric design variables
  `x = [D_cav, L_cav, D_abs, L_abs, Z_TIP, D_body_in]`
  Objectives: maximize stability limit `alim`, maximize static stiffness,
  minimize total mass.
- **Inner GA**, single-objective, tunes the absorber's `[kd, cd]`
  (stiffness, damping) for each outer candidate geometry — replacing a
  classical Den Hartog closed-form tuning with a warm-started evolutionary
  search.

## Pipeline

| # | Section | Purpose |
|---|---------|---------|
| 1 | Fixed parameters | Material props, bar envelope, GA hyperparameters, variable bounds |
| 2 | Cross-section (`xsec`) | Area / inertia at any axial position for the 5-section geometry |
| 3 | FEM assembly (`build_MK`) | 60-element beam FEM → stiffness, mass, gyroscopic, translational-mass matrices |
| 4 | Modal reduction (`modal_reduce`) | Reduces to the lowest 6 modes for fast repeated evaluation |
| 5 | Absorber parameters | Absorber mass and axial location from cavity/absorber geometry |
| 6 | Rotordynamic FRF (`frf`) | Complex impedance solve incl. gyroscopic + Coriolis coupling and centrifugal softening, with/without absorber |
| 7 | Directional coefficients | Direction-averaged cutting-force coefficients for the stability criterion |
| 8 | Stability limit (`alim_from_FRF`) | Critical chatter-free depth of cut from the FRF |
| 9 | Static stiffness & mass | Tip stiffness (reuses FEM `Kf`) and total bar+absorber mass |
| 10 | Stability lobe diagram | Mode-resolved depth-vs-RPM diagram (diagnostic, not a GA objective) |
| 10.5 | Absorber clearance check | Confirms the absorber mass doesn't collide with the cavity wall under vibration |
| 11 | GA operators | SBX crossover, polynomial mutation, binary tournament (shared by both GAs) |
| 12 | Inner GA | Tunes `[kd, cd]` to maximize mean `alim` across representative spindle speeds |
| 13 | Feasibility repair | Clips/repairs a 6-gene chromosome to satisfy all geometric constraints |
| 14 | Individual evaluation | Full evaluation of one outer-GA candidate (calls inner GA + all physics) |
| 15 | Outer NSGA-II | Non-dominated sorting + crowding distance + elitist generational loop |
| 16 | Post-processing | Extracts result arrays, Pareto front, combined score |
| 17 | Plotting | Convergence history, parameter-vs-score panels, FRF/lobe diagrams, best-design geometry |
| 18 | Entry point | Runs the full optimization and generates the plots |

## How the "best" design is selected

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

This is a simple unweighted-average scalarization over the Pareto set, not
a weighted or preference-driven decision method — all three objectives are
treated as equally important.

## Design variables

| Variable | Meaning |
|----------|---------|
| `D_cav`, `L_cav` | Absorber cavity diameter / length |
| `D_abs`, `L_abs` | Absorber (tuned mass) diameter / length |
| `Z_TIP` | Axial offset of the cavity from the free (cutting) end |
| `D_body_in` | Internal bore diameter through the main body |

## Objectives

- **Maximize** `alim` — chatter-free stability limit (depth of cut)
- **Maximize** static tip stiffness
- **Minimize** total mass

## Key constraints

- Absorber must fit inside the cavity (`D_abs ≤ D_cav`, `L_abs ≤ L_cav`)
- Absorber volume ≤ 70% of cavity volume
- Minimum 3 mm wall thickness (cavity and body bore)
- Minimum solid tip length
- Body bore ≤ cavity bore

## Requirements

```
numpy
scipy
matplotlib
```

## Running

Run all cells in order (or execute as a script). The entry-point cell runs
the outer NSGA-II loop and produces a single multi-panel figure summarizing
convergence, the Pareto front, per-parameter trends, FRFs, the stability
lobe diagram, and the recommended best design's geometry and tuning.
