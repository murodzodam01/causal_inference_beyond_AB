# Causal Inference Beyond A/B Testing

Two observational case studies showing how causal questions can be approached when randomized experiments are unavailable.

This repository focuses on **propensity-score matching** and **synthetic control**, with particular attention to the diagnostics that determine whether a counterfactual comparison is credible.

**Methods:** Propensity Scores · Matching · Overlap Diagnostics · Covariate Balance · Synthetic Control · Placebo Tests · Causal Identification

[Open the notebook](Matching_and_Synthetic_Control.ipynb)

## Bottom line

### 1. Did UN intervention reduce conflict duration?

**No clear evidence.**

The matched estimates do not consistently show shorter conflicts after UN intervention, the estimates are imprecise, and several covariates remain materially imbalanced after matching.

The logistic propensity-score design retains **13 of 16 treated observations**, while the fitted random forest produces **no usable matches** under the implemented caliper.

The strongest conclusion from this case is therefore methodological rather than substantive: **the available matched comparison is not strong enough to support a reliable causal claim that UN intervention reduced conflict duration.**

### 2. Did German reunification reduce West Germany's GDP per capita relative to its synthetic counterfactual?

**The saved analysis suggests yes, but not conclusively.**

The synthetic-control series tracks West Germany closely before the cutoff and lies above the observed West German GDP path in much of the post-cutoff period. The saved post-cutoff means are approximately **24,709 for West Germany** and **26,378 for the synthetic comparison**.

This pattern is consistent with a negative post-reunification GDP effect relative to the estimated counterfactual. However, the original implementation has important limitations, including fitting through 1990, optimizing on the GDP trajectory only, and a date mismatch in one saved placebo-style calculation.

The result should therefore be interpreted as **suggestive evidence rather than a definitive causal estimate**.

---

## Why this project matters

A/B tests are often the preferred way to estimate causal effects, but many real-world product, policy, and business questions cannot be randomized.

This project explores two common alternatives:

1. **Matching:** Can treated observations be compared with sufficiently similar untreated observations?
2. **Synthetic control:** Can a weighted combination of untreated units approximate the counterfactual trajectory of a treated unit?

The main lesson is that obtaining a numerical estimate is not enough. Credible causal analysis also depends on **overlap, balance, identification assumptions, counterfactual quality, and robustness diagnostics**.

---

## Case Study 1 — UN Interventions and Conflict Duration

### Research question

Did UN intervention reduce observed conflict duration relative to comparable untreated conflict snapshots?

The dataset contains **1,227 conflict snapshots**, including **16 treated observations**.

Because UN intervention is not randomly assigned, treated and untreated conflicts can differ systematically in severity, political context, geography, and other pre-treatment characteristics.

### Approach

The analysis:

- models treatment assignment using logistic regression and a random forest;
- estimates propensity scores;
- evaluates common support between treated and untreated observations;
- performs nearest-neighbor matching with a caliper;
- checks covariate balance using standardized mean differences;
- compares outcomes after matching;
- examines sensitivity to the number of available controls.

### Key findings

The logistic propensity-score design retains **13 of 16 treated observations**.

The fitted random forest separates treated and control observations so strongly that it produces **no usable matches** under the implemented caliper.

The saved matched contrasts vary across specifications rather than pointing consistently in one direction, and the reported intervals include zero.

This illustrates an important causal-design lesson:

> **Better treatment prediction does not necessarily produce better causal comparisons.**

A model can classify treatment extremely well while creating poor overlap between treatment groups, making counterfactual estimation difficult or impossible.

Several covariates also remain materially imbalanced after matching. For that reason, the analysis does **not** support a strong causal conclusion that UN intervention reduced conflict duration.

### What this case demonstrates

- predictive accuracy and causal identification are different objectives;
- overlap must be checked before estimating treatment effects;
- propensity-score matching can change the target population by discarding treated units;
- balance diagnostics remain necessary after matching;
- small treated samples and repeated observations can substantially limit inference.

---

## Case Study 2 — German Reunification and Synthetic Control

### Research question

Did German reunification reduce West Germany's GDP-per-capita trajectory relative to a synthetic counterfactual constructed from other countries?

The panel contains **17 countries observed from 1960 to 2003**.

### Approach

The analysis:

- uses West Germany as the treated unit;
- constructs an unconstrained regression benchmark;
- estimates non-negative donor weights constrained to sum to one;
- builds a synthetic comparison using the pre-cutoff GDP trajectory;
- compares observed and synthetic GDP paths;
- interprets donor contributions;
- constructs country-placebo trajectories;
- examines an earlier-cutoff diagnostic.

### Main synthetic-control weights

| Donor country | Approx. weight |
|---|---:|
| Austria | 0.29 |
| USA | 0.27 |
| Italy | 0.19 |
| Netherlands | 0.13 |
| Switzerland | 0.08 |
| France | 0.03 |

The fitted synthetic series tracks West Germany closely in the pre-cutoff period and diverges more visibly afterward.

The saved pre-cutoff GDP-per-capita means are approximately **8,566 for West Germany** and **8,562 for the synthetic comparison**. In the post-cutoff period, the corresponding means are approximately **24,709** and **26,378**.

This pattern is consistent with West Germany underperforming its synthetic counterfactual after reunification, but it is **not by itself proof of a causal effect**.

### What this case demonstrates

- construction of a counterfactual from a donor pool;
- constrained optimization of synthetic-control weights;
- interpretation of donor contributions;
- comparison of pre- and post-treatment trajectories;
- use of placebo-style diagnostics;
- importance of treatment timing and correctly aligned comparison periods.

---

## Important interpretation notes

This repository preserves the original analysis code and saved outputs from a university project while improving the explanatory narrative.

Several implementation details limit the strength of causal claims:

- the matching analysis has only **16 treated observations** and retains 13 after matching;
- meaningful post-match covariate imbalance remains;
- the random-forest propensity model is evaluated in sample and should not be interpreted as a superior causal design;
- one balance diagnostic includes an outcome-derived interaction term and should not be interpreted as a pre-treatment covariate diagnostic;
- the synthetic-control implementation optimizes on the **GDP trajectory only**;
- the original synthetic-control code fits through **1990 inclusive**;
- one saved "year 2000" calculation uses the 2003 synthetic value, so it should not be interpreted as a valid permutation p-value;
- the earlier-cutoff diagnostic is partly evaluated after the actual reunification event.

These limitations are intentionally documented rather than hidden. They illustrate why causal inference requires scrutiny of the research design, not just model output.

---

## Repository structure

```text
.
├── Matching_and_Synthetic_Control.ipynb
├── README.md
└── data/
    ├── repgermany.dta
    └── war_pre_snapshots.dta
```

---

## Tools

Python · pandas · NumPy · scikit-learn · statsmodels · SciPy · DuckDB · matplotlib · seaborn · joblib · rpy2

> The notebook imports `rpy2`, so reproducing the full environment requires a compatible R installation in addition to the Python dependencies.

---

## Key causal-inference takeaways

This project extends experimentation knowledge beyond randomized A/B tests and demonstrates how observational causal designs can be evaluated critically.

In particular, it shows how to:

- define a counterfactual when randomization is unavailable;
- distinguish prediction from causal estimation;
- diagnose overlap and balance before interpreting treatment effects;
- construct and evaluate a synthetic comparison;
- recognize when implementation or identification problems limit a causal claim;
- communicate uncertainty rather than overstate results.

---

## References

The conflict application references Gilligan and Sergenti (2008), *Do UN Interventions Cause Peace? Using Matching to Improve Causal Inference*.

The synthetic-control implementation acknowledges Matheus Facure's *Causal Inference for the Brave and True*.

The supplied datasets are included for the analysis; this repository does not claim original data collection.
