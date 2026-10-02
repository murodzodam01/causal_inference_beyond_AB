# Causal Inference Beyond A/B Testing

## Propensity-score matching and synthetic control

Two observational case studies explore how to construct counterfactuals when randomized experiments are unavailable: **UN interventions and conflict duration**, and **German reunification and GDP per capita**.

The project demonstrates treatment-assignment modeling, matching design, overlap and balance checks, matched outcome comparisons, constrained donor weighting, and placebo diagnostics. Its findings also show why predictive performance and a fitted counterfactual are not sufficient to establish causality.

**[Explore the notebook](Matching_and_Synthetic_Control.ipynb)** · [Data notes](data/README.md)

## At a glance

| | UN interventions | German reunification |
|---|---|---|
| Question | How does duration differ for treated snapshots with comparable controls? | How does West Germany's GDP path compare with a weighted donor path? |
| Data | 1,227 snapshots; 16 treated observations | 748 country-year observations; 17 countries; 1960–2003 |
| Methods | Logistic/RF propensity scores; nearest-neighbor matching; balance diagnostics | Unconstrained regression benchmark; constrained synthetic control; placebo comparisons |
| Main saved finding | Logistic design matches 13 treated observations; forest matches none | GDP-fitted comparison assigns most weight to Austria and the USA |
| Interpretation | Imprecise duration contrasts with residual imbalance | Post-cutoff path divergence; formal inference limited by the preserved implementation |

## Case study 1 — Matching UN-intervention snapshots

The analysis uses 14 covariates to model UN intervention, constructs nearest-neighbor matches with replacement, and evaluates common support and standardized mean differences. It then compares designs requesting up to one, two, or three controls per treated observation.

The fitted random forest perfectly predicts treatment **in sample**, but its treated scores (0.61–0.83) do not overlap with control scores (0–0.10). It produces zero matches. Logistic matching retains **13 of 16 treated observations**, although several covariates remain materially imbalanced.

![Original saved propensity-score overlap plots](figures/propensity_overlap.png)

### Saved outcome comparisons

| Maximum controls per treated unit | Retained treated | Equal-treated matched contrast | Pooled regression contrast | Reported matched-set-clustered SE |
|---|---:|---:|---:|---:|
| 1 | 13 | −2.923 | −2.923 | 13.216 |
| 2 | 13 | 1.846 | 0.529 | 12.994 |
| 3 | 13 | 6.769 | 5.256 | 13.238 |

The values are in **months**: every `dur` value in the supplied file equals the calendar-month difference between its dates. Original figures retain “days” labels because code and outputs are preserved.

The pair-average and regression estimates differ when matched sets contain different numbers of controls. The reported regression intervals all include zero. Those intervals are not a definitive inference result: controls can be reused across matched sets, source conflicts contribute repeated snapshots, and important imbalance remains.

**Design lesson:** successful treatment classification does not establish a usable causal comparison. Overlap, balance, sample retention, and the dependence structure matter separately.

## Case study 2 — A synthetic comparison for West Germany

The notebook first fits unconstrained regression weights, then minimizes pre-cutoff GDP prediction error subject to nonnegative weights that sum to one. The constrained optimizer uses **only the GDP-per-capita trajectory**. Other predictors are inspected afterward, not included as optimized balance targets.

The code fits through **1990 inclusive** and defines the post-period as **1991–2003**. This timing convention is retained and noted because reunification occurred during 1990.

![Original saved West Germany and synthetic-control trajectories](figures/synthetic_germany.png)

The plot contains GDP levels despite its retained “Gap in gdp” axis label.

### Saved donor weights

| Donor | Rounded weight |
|---|---:|
| Austria | 0.29 |
| USA | 0.27 |
| Italy | 0.19 |
| Netherlands | 0.13 |
| Switzerland | 0.08 |
| France | 0.03 |

Other donor weights round to zero. The unrounded weights sum to approximately one; rounded values sum to 0.99.

The saved pre-cutoff GDP means are **8,566.45** for West Germany and **8,561.56** for the synthetic comparison. Post-cutoff means are **24,709.15** and **26,377.58**. These summarize the fitted paths, rather than independently validating a causal effect.

The notebook also constructs country-placebo trajectories and an earlier, 1975-cutoff fit. The main printed “year 2000” effect has a date mismatch: it subtracts the **2003 synthetic value** from observed **2000 GDP**. Its saved tail fraction of **0.0 is therefore not reported here as a valid permutation p-value**. The 1975-cutoff comparison is evaluated in 2000, after actual reunification, so it is not an entirely pre-treatment falsification test.

**Design lesson:** close pre-period averages and a post-period gap are useful evidence to inspect, but credible inference also requires aligned dates, a defensible donor pool, suitable placebo comparisons, and stable counterfactual assumptions.

## Methods and implementation scope

| Implemented | Scope or limitation |
|---|---|
| Logistic and random-forest propensity estimation | Training-sample predictions; logistic score column is overwritten with earlier scikit-learn predictions |
| Caliper matching with replacement | Logistic matching uses logit scores; forest matching uses raw probabilities |
| SMDs and Love plots | Post-match treated SD is recomputed; several covariates remain imbalanced |
| Matched outcome comparisons | Target is the retained treated snapshots, not all interventions |
| Matched-set-clustered regression | Shared controls and repeated source periods are not fully represented by those cluster IDs |
| Constrained synthetic control | GDP-only optimization; no entropy-balancing implementation |
| Placebo diagnostics | Saved date and comparison limitations prevent strong formal significance claims |

No bootstrap, entropy-balancing model, or corrected reanalysis is claimed. These are possible future extensions rather than completed features.

## Repository contents

| Path | Contents |
|---|---|
| `Matching_and_Synthetic_Control.ipynb` | Revised English narrative with all original code and outputs |
| `data/war_pre_snapshots.dta` | Supplied conflict snapshot data, unchanged |
| `data/repgermany.dta` | Supplied country panel, unchanged |
| `data/README.md` | Dataset structure, field caveats, and provenance |
| `figures/` | Two images copied directly from saved notebook outputs |
| `requirements.txt` | Unpinned package list for the existing imports |

## Viewing and local setup

The notebook contains saved figures and tables and can be reviewed on GitHub without execution. All results reported here come from those existing outputs; models were not rerun for this edition.

For local use, keep the directory structure intact so the original `data/...` paths resolve. The notebook imports `rpy2`, so a compatible **R installation** is required in addition to Python packages. `pip` alone does not install R.

```bash
python -m pip install -r requirements.txt
python -m jupyterlab
```

Open the notebook from the repository root. The original environment is not fully specified, so dependencies are unpinned and a successful fresh run is not claimed. Saved warnings remain visible. The existing notebook also changes the process-wide SSL context and requests eight parallel workers; these settings are unchanged, not recommendations of this README.

## Preservation and attribution

This is a **presentation-only portfolio edition** of a collaborative university project. Only Markdown cells and documentation were revised. Every original code cell—including code comments, outputs, warnings, execution counts, and metadata—was compared for exact equality against the supplied notebook. Both data files are byte-identical copies. No models were rerun and no saved results were replaced.

**Contributors:** Jonathan Sauer, Muhammad Murodzoda, Sebastian Aguilar.  
**Portfolio presentation:** Muhammad Murodzoda.

The conflict application cites Gilligan and Sergenti (2008), *Do UN Interventions Cause Peace? Using Matching to Improve Causal Inference*, [Quarterly Journal of Political Science](https://nowpublishers.com/article/Details/QJPS-7051). The synthetic-control code acknowledges Matheus Facure's [Causal Inference for the Brave and True](https://matheusfacure.github.io/python-causality-handbook/15-Synthetic-Control.html).

The data extracts were supplied with the notebook; their complete provenance and redistribution terms were not supplied. This project does not claim original data collection, exact replication of a published study, or a blanket license over third-party material.
