# The Missing T⁰ⁱ Momentum Flux in Lattice QCD — Simulation Code

This repository contains the simulation, validation, and figure-generation code
for the paper *The Missing T⁰ⁱ Momentum Flux in Lattice QCD: Chromomagnetic Vacuum Structure, Mass Gap, and Gravitational Analogue*. All scripts operate
on standard Wilson-action SU(3) gauge configurations; the Curvature–Transport
Correspondence (CTC) observables are read from the same link variables Uμ(x) the
Wilson Monte Carlo produces, without modifying the sampling measure.

The principal result (Figure 3) is produced by `ctc_beta5.py` under pure Wilson
sampling: Im Tr[P₀ᵢ] is reconstructed and read via m\* = θ / |∇·F|, and the mass
gap appears as a volume-stable topological floor Δ. The α-augmented action
(`ctc_T0i_action_save_v4.py`) is a separate stress test, not the core result.


## Quick start

```
# 1. Baseline production run (Figure 3 result, Δ, β-scan, volume-scan)
python3 ctc_beta5.py --N 24 --beta 5.0 --sweeps 1000

# 2. Regenerate the Forbidden-Zone / mass-gap figure from a saved checkpoint
python3 forbidden_zone_from_U.py U_final_N24_b5.0_s42.npy

# 3. Regenerate the 9-panel dashboard from a saved records.json
python3 dashboard_from_json.py records.json
```


## Production engines

| Script | Role |
|--------|------|
| `ctc_beta5.py` | Baseline Wilson production. Generates `records.json`, the 9-panel dashboard, and the mass-population data. Source of the Δ floor, β-scan, and volume-scan. The load-bearing engine. |
| `ctc_T0i_action_save_v4.py` | T⁰ⁱ-augmented action α-scan (N=24, β=5.8, two seeds, α ∈ {0, 1.0, 1.5, 1.8, 2.0}). Stress test of Δ under amplification of the discarded operator. |


## Engine validation (Section 5.1)

| Script | Produces |
|--------|----------|
| `su3_plaquette_validation.py` | **Figure 1.** Equilibrium mean plaquette ⟨P⟩ vs the Wilson benchmark across the β-scan and finite-volume study (Table 2). |
| `su3_wilson_loop_validation.py` | Static QQ̄ string tension σ_QQ̄ from Wilson loops, un-smeared (Table 3). |
| `su3_wilson_loop_validation_smeared.py` | Smeared driver; imports the un-smeared engine and applies APE smearing on spatial links at measurement time. |
| `ape_smearing.py` | APE smearing helper (Takahashi–Suganuma 2002 / Bali–Schilling 1992 recipe). |
| `plaquette_figure_addon.py` | Figure + CSV companion to `su3_plaquette_validation.py`. |
| `check_div_epsilon.py` | Diagnostic for the +1e-6 softening in the divergence: audits whether the regulator ever bound on production configurations. |


## Figures and dashboards

| Script | Produces |
|--------|----------|
| `flow_decomposition_4x4_all.py` | **Figure 2.** Gauge-invariant flow and its Helmholtz decomposition (longitudinal / transverse / stiffness) across four temporal slices. |
| `forbidden_zone_from_U.py` | **Figure 3.** Four-panel Forbidden-Zone / mass-gap figure from a saved `U_final_*.npy` (phase space, mass distribution, spatial mass map, m\* vs θ). Renamed from `figure1.py`. |
| `dashboard_from_json.py` | 9-panel dashboard for baseline `ctc_beta5.py` runs (Supplement A, S1a–S1f). |
| `dashboard_lite_from_json.py` | 9-panel dashboard for augmented-action runs (T⁰ⁱ panels replace the hadron-formation panels). |


## CP-symmetry verification (Supplement S3b, S3c)

| Script | Produces |
|--------|----------|
| `cp_test.py` | CP-symmetry verification on a saved `U_final_*.npy`: signed Im Tr[P₀ᵢ] mean vs RMS, with algebraic per-site and ensemble checks (Tables S1, S2). |
| `cp_test_minimal.py` | Minimal-figure version of `cp_test.py` for the main-text CP-symmetry figure. |


## §5.9 sub-analyses (coupling, polarity, stiffness, selectivity)

| Script | Role |
|--------|------|
| `mstar_T0i_correlation.py` | Spatial coupling r(m\*, |T⁰ⁱ|²) and the joint (m\*, |T⁰ⁱ|²) distribution. |
| `mstar_distribution_alpha_stack.py` | Stacked m\*(x) distribution across the α-scan. |
| `figure1_panelB_alpha_stack.py` | Mass-distribution panel stacked across the α-scan (panel-b logic shared with the Forbidden-Zone figure). |
| `polarity_analysis.py` | Five-test polarity battery and net-orientation analysis on the augmented anchor configuration. |
| `tube_interior_mstar_by_E_bin.py` | Long-tube selectivity test (binned m\* vs |E|). |
| `velocity_arm_validity.py` | Validity tests (V1–V4) for the positive structured-correlation claim. |
| `rgb_meson_twist_scan.py` | RGB meson twist scan. |
| `ctc_meson_persistence.py` | Meson-persistence analysis across sweeps. |
| `local_stiffness_slices.py` | Local stiffness κ = 1/g sampled across multiple slices. |
| `local_stiffness_smoothed.py` | Smoothed local stiffness map. |
| `stiffness_smoothed_curvature.py` | Coarse-grained (physically smoothed) stiffness map. |
| `stiffness_all_in_one.py` | Single-pass stiffness map. |
| `vacuum_stiffness_full.py` | Full-lattice stiffness (every plaquette, no slicing). |
| `flow_vs_stiffness_raw.py` | κ and twist from the identical raw flux field F. |
| `flow_vs_stiffness_same_config.py` | Flux flow and smoothed stiffness from the same configuration. |
| `plot_real_stiffness.py` | Plotting companion for the stiffness analyses. |


## Movies (Supplement S4, S5)

| Script | Role |
|--------|------|
| `ctc_meson_baryon_movie.py` | Meson/baryon formation movie. |
| `ctc_z3_rgb_movie.py` | Z₃ RGB-domain movie. |


## Batch drivers

| Script | Role |
|--------|------|
| `run_all_analyses.py` | Master driver: runs the post-processing analyses on a baseline configuration. |
| `run_all_analyses_T0i.py` | Master driver for the augmented-action configurations. |
| `run_flow_topology_batch.py` | Batch driver for the flow / topology decomposition. |
| `run_all_movies.py`, `run_movies_batch.py`, `run_ctc_movies_batch.py` | Movie batch drivers. |


## Data files

| File | Contents |
|------|----------|
| `records.json` / `records_beta_*.json` | Per-sweep records from a production run (consumed by the dashboards and `forbidden_zone_from_U.py`). |
| `U_final_*.npy` | Saved equilibrated link-field configurations (one per run, after equilibration). Input to the figure and CP-test scripts. |
| `CTC_Simulation_Data.xlsx` | Complete raw data: Summary, β-scaling raw, T⁰ⁱ comparison raw, and chart-data sheets. |


## Notes on naming

`forbidden_zone_from_U.py` was renamed from `figure1.py` so the script name no
longer encodes a figure number and stays correct under manuscript renumbering.
The figure it produces is the four-panel Forbidden-Zone / mass-gap result,
currently Figure 3. The baseline and augmented-action dashboards are separate
scripts (`dashboard_from_json.py` and `dashboard_lite_from_json.py`) because the
two record sets carry different panel content.
