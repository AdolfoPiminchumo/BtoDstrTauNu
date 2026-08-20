# BDT_BtoDstrtanu_updated_v2.ipynb

BDT-based signal/background classifier for the Belle II analysis of
**B0 -> D*- tau+ nu_tau**, with tau+ reconstructed as **tau+ -> pi+ pi- pi+ nu_tau_bar**.
Main physics goal: measure the T-odd triple-product asymmetry as a probe of
transverse tau polarization (sensitive to New Physics, SM predicts zero).

## Requirements

- Python 3.9+
- `numpy`, `pandas`, `matplotlib`, `scipy`, `scikit-learn`, `uproot`, `mplhep`
- Access to the ROOT ntuples at
  `/net/pr2/projects/plgrid/plggbelle2ml/ppss_2026/samples_v2/`
  (`sigB0_v2_part.root`, `ccB0_v2_part.root`, `chB0_v2_part.root`,
  `mixB0_v2_part.root`) — e.g. run from the Polish cluster or `lxplus`/SWAN
  with that path mounted.

Run cells top to bottom; later cells depend on state built up earlier
(dataframe columns, the `PRECUTS` / `features` / `bdt` objects, etc.), so it
is not safe to run cells out of order.

## Pipeline

1. **Data loading** — signal MC + three background samples (`ccB0`, `chB0`,
   `mixB0`); `mixB0` is filtered to `sigissig == 0` before combining, so no
   truth-matched signal leaks in as background.
2. **Feature engineering** — `delta_M`, `M_3pi`, CMS polar-angle cosines
   (D0/D*/B_tag), pi-pi pair masses (`M_pipi_12/13/23`), fast/slow
   companion-pion masses, and the **triple product** `triple_product_reco`
   (main observable — computed consistently in the **LAB frame** for all
   three input vectors: tau direction, D* momentum, 3-pion-system momentum).
3. **Pre-cuts** — defined once in a `PRECUTS` dict (single source of truth
   used for both the diagnostic plots and the actual selection) and applied
   to `df_all`; events are then labeled signal (`sigissig == 1`) vs.
   background.
4. **rho(770) resonance study** — folded pi-pi mass plot and Dalitz plots
   (`M13^2` vs `M23^2`, plus a fast/slow cross-check), using the physically
   correct tau -> 3pi kinematic range.
5. **Correlation study** — signal/background correlation matrices over the
   full candidate feature set; every pair with |r| > 0.5 is resolved by
   comparing each feature's KS (signal-vs-background) separation power, and
   the weaker one is dropped automatically into the final `features` list.
6. **BDT training** — `AdaBoostClassifier` (depth-2 trees), ROC/AUC,
   feature importances, an ablation check (drop low-importance features,
   confirm AUC holds) and a 5-fold CV AUC as a basic overtraining check.
7. **Signal extraction** — Punzi-optimized BDT cut, then a Gaussian(signal)
   + ARGUS(background) fit to `tagmbc`.
8. **Triple-product asymmetry** — sign-count (`T>0` vs `T<0`) asymmetry with
   propagated uncertainties, shown separately for signal/background and
   before/after the BDT cut.
9. **Forward-backward asymmetry** — exploratory; the physically correct
   `A_FB` needs the tau direction boosted into the signal-B rest frame
   (not yet implemented — flagged as future work).

## Known caveats / things to verify against real data

- **Pion charge convention**: pi1/pi2 are assumed to be the same-sign pair
  (pi+ pi+) and pi3 the opposite-sign pion (pi-) — confirm against the
  actual ntuple index assignment.
- **Significance / yields are not yet luminosity-normalized** — `FRAC_BK`
  and cross-section/branching-fraction scaling still need to be applied for
  a physically meaningful significance (flagged in-notebook as next step).
- **A_FB in the signal-B rest frame** requires an explicit Lorentz boost —
  not implemented; current cells only show lab-frame and Upsilon(4S)-CMS
  cross-checks.
