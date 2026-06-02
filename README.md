# The Missing T⁰ⁱ Momentum Flux in Lattice QCD — Simulation Code

This repository contains the simulation, validation, figure, and verification
code for the paper *The Missing T⁰ⁱ Momentum Flux in Lattice QCD*. Every script
operates on standard Wilson-action SU(3) gauge configurations. The
Curvature–Transport Correspondence (CTC) observables are read from the same
link variables Uμ(x) the Wilson Monte Carlo produces; the sampling measure is
never modified.

The principal result (Figure 3) comes from `ctc_beta5.py` under pure Wilson
sampling: Im Tr[P₀ᵢ] is reconstructed and read via m* = θ / |∇·F|, and the mass
gap appears as a volume-stable topological floor Δ. The α-augmented action
(`ctc_T0i_action_save_v4.py`) is a separate stress test, not the core result.

Only scripts that produce a result, figure, table, or verification reported in
the paper are included here.


## Quick start

```
# Baseline production run (Figure 3 result, Δ, β-scan, volume-scan)
python3 ctc_beta5.py --N 24 --beta 5.0 --sweeps 1000

# Regenerate the Forbidden-Zone / mass-gap figure from a saved checkpoint
python3 forbidden_zone_from_U.py U_final_N24_b5.0_s42.npy

# Regenerate the 9-panel dashboard from a saved records.json
python3 dashboard_from_json.py records.json
```


## Production engines

**`ctc_beta5.py`** — Vectorized SU(3) Wilson Monte Carlo, the load-bearing
engine. Thermalizes a hypercubic lattice at the requested β, then at each
measurement sweep computes the mean plaquette, the CTC fields (curvature θ,
divergence |∇·F|, local mass m*), eddy cores, the ℤ₃ Polyakov-loop sector
field, and the equilibrium mass gap Δ via KDE gap-detection. Writes a
per-sweep `records.json`, a nine-panel dashboard PNG, and a saved
`U_final_*.npy` checkpoint. The hadron-counting routines use per-color
sampling caps so meson and baryon counts scale with eddy-core density.
Usage: `python3 ctc_beta5.py --N 24 --beta 5.0 --sweeps 1000`.

**`ctc_T0i_action_save_v4.py`** — T⁰ⁱ-augmented-action α-scan. Adds a CP-even
term α·(Im Tr P₀ᵢ)² to the Metropolis acceptance with the sign that drives the
chromomagnetic twist to larger amplitude, then measures how Δ responds. Run at
N=24, β=5.8, two seeds, over α ∈ {0, 1.0, 1.5, 1.8, 2.0}. The central test of
whether Δ is invariant under amplification of the operator the Wilson action
discards.


## Engine validation (Section 5.1, Tables 2–3)

**`su3_plaquette_validation.py`** — Independent Wilson Monte Carlo that
thermalizes at the requested β and measures the equilibrium mean plaquette ⟨P⟩
across decorrelated configurations, comparing to the established Wilson
benchmark (Figure 1, Table 2). Also reports the spatial/temporal plaquette
sub-averages, the symmetry check, and the Polyakov-loop magnitude. Requires
`plaquette_figure_addon.py`.

**`plaquette_figure_addon.py`** — Figure + CSV writer imported by
`su3_plaquette_validation.py` (provides `write_plaquette_figure`). Required
dependency; not run on its own.

**`su3_wilson_loop_validation.py`** — Measures planar rectangular Wilson loops
W(r,t) on the equilibrated configurations, extracts the static QQ̄ potential
V(r) from the effective-potential plateau, fits the Cornell form
V(r) = −A/r + σr + C, and compares the string tension σ to Bali–Schilling 1992
and Takahashi–Suganuma 2002 (Table 3, un-smeared rows). Reuses the Metropolis
engine from `su3_plaquette_validation.py`.

**`su3_wilson_loop_validation_smeared.py`** — Separate driver that imports the
un-smeared engine and applies APE smearing to a deep copy of the spatial links
at each measurement step, leaving the Markov chain untouched (Table 3, smeared
rows). Imports `su3_wilson_loop_validation.py` and `ape_smearing.py`.

**`ape_smearing.py`** — APE smearing of SU(3) spatial links (Takahashi–Suganuma
2002 / Bali–Schilling 1992 recipe), with an SVD-based SU(3) projection and a
self-test. Imported by the smeared validation driver.

**`check_div_epsilon.py`** — Diagnostic auditing whether the +1e-6 softening
floor in the divergence ever bound on production configurations, confirming the
regulator never actively affected the reported results.


## Figures and dashboards

**`flow_decomposition_4x4_all.py`** — Renders the gauge-invariant flow and its
Helmholtz decomposition (Figure 2): four temporal slices in columns, with rows
for flow streamlines, longitudinal |div F|², transverse |curl F|², and
stiffness κ = 1/g. Tests the global closure g = |div F|² + |curl F|² and prints
the residual and transverse fraction. Usage:
`python3 flow_decomposition_4x4_all.py U_final_N32_b6.0_s42.npy`.

**`forbidden_zone_from_U.py`** — Regenerates the four-panel Forbidden-Zone /
mass-gap figure (Figure 3) from a saved `U_final_*.npy`: phase-space Forbidden
Zone, mass distribution, spatial mass map, and m*-vs-θ clouds colored by
|∇·F|. Reads Δ from the matching `records.json` by the same plateau-average
formula the dashboard uses, with a KDE single-snapshot fallback that is labelled
explicitly when no records.json is found. (Renamed from `figure1.py`; the name
no longer encodes a figure number.) Usage:
`python3 forbidden_zone_from_U.py U_final_N24_b5.0_s42.npy`.

**`dashboard_from_json.py`** — Nine-panel post-processing dashboard for the
baseline `ctc_beta5.py` runs, built from a saved `records.json`: ⟨P⟩, vortex %,
acceptance, ℤ₃ counts and balance, hadron formation, Δ, Δ-vs-eddy-count, and
mass ratio (Supplement A). Usage:
`python3 dashboard_from_json.py records.json --outdir figs/`.

**`dashboard_lite_from_json.py`** — Companion dashboard for the augmented-action
runs; structurally identical but replaces the hadron-formation panels with
T⁰ⁱ diagnostics. Its load-bearing panel is Δ vs ⟨T⁰ⁱ⟩, which shows that Δ does
not respond as α drives T⁰ⁱ to large amplitude. Usage:
`python3 dashboard_lite_from_json.py records_N24_b5.8_a1.00_s42.json`.


## CP-symmetry verification (Supplement S3b, S3c)

**`cp_test.py`** — Runs two CP checks on a saved `U_final_*.npy`: (A) an
algebraic per-site identity check that Im Tr P₀ᵢ is CP-odd and (Im Tr P₀ᵢ)² is
CP-even, passing at machine epsilon; and (B) a statistical ensemble check of the
signed mean ⟨Im Tr P₀ᵢ⟩ against its SEM, reporting |mean|/SEM significance,
variance, and skewness, combined and per color channel (Tables S1, S2). Outputs
a nine-panel figure and a JSON summary per configuration. Usage:
`python3 cp_test.py U_final_N32_b5.8_a0.00_s137.npy`.

**`cp_test_minimal.py`** — Two-panel main-text version of `cp_test.py`
(algebraic residual; statistical ensemble check) that still computes and writes
the full numerical record to JSON. Supports recursive auto-discovery of
`U_final_*.npy` under `--root` with `--beta` / `--seed` / `--alpha` filtering.


## Movies (Supplement S4, S5)

**`ctc_meson_baryon_movie.py`** — Continuation-run movie of the chromomagnetic
flux field on a spatial slice at t = N/2. Thresholds |Im Tr P₀ᵢ|, runs
connected-component analysis, and overlays both a ℤ₃-sector classifier and an
F-aspect classifier to identify meson tubes and baryon junctions, with their
overlap reported. Writes a rotating-volume GIF and a per-frame history JSON.

**`ctc_z3_rgb_movie.py`** — Companion movie of the ℤ₃ R/G/B Polyakov-loop sector
field across a continuation run, with volume, domain-wall, slice, and panel
render modes. Visualises the RGB domain composition and the domain-wall network
that underlie the confinement picture.


## Data files

**`records.json` / `records_beta_*.json`** — Per-sweep records from a production
run; consumed by the dashboards and `forbidden_zone_from_U.py`.

**`U_final_*.npy`** — Saved equilibrated link-field configurations (one per run).
Input to the figure, CP-test, and movie scripts.

**`CTC_Simulation_Data.xlsx`** — Complete raw data: Summary, β-scaling raw,
T⁰ⁱ comparison raw, and chart-data sheets.


## Dependency notes

The validation scripts form an import chain:
`su3_wilson_loop_validation_smeared.py` → `su3_wilson_loop_validation.py` →
`su3_plaquette_validation.py`, and the smeared driver also imports
`ape_smearing.py`. `su3_plaquette_validation.py` imports
`plaquette_figure_addon.py`. Keep these files in the same directory.
