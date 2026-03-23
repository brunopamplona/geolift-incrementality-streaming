# geolift-incrementality-video+

![Python](https://img.shields.io/badge/Python-3.10%2B-blue?logo=python)
![License](https://img.shields.io/badge/License-MIT-green)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)
![Domain](https://img.shields.io/badge/Domain-Marketing%20Science-orange)

> **Causal measurement of paid media incrementality for a subscription streaming service using Synthetic Control Methods and GeoLift design.**

---

## Business Problem

Video+, a Brazilian ficticious subscription streaming platform (~1.2M MAU), ran a paid media campaign (YouTube + Meta + Google Ads) promoting its **Família plan (R$80/mo), Premiere (R$ 69/mo), Telecine (R$ 55/mo) and HBO (R$ 35/mo)** across selected Brazilian regions in Q3.

The internal attribution model reported a **ROAS of 9.2x**.

The critical question this study addresses:

> *How many of those subscriptions would have occurred even without the campaign? What is the true incremental return on ad spend?*

This study applies a Shapley MTA approach benchmarked against organic growth—unlike last-click attribution, which tends to overstate paid media contribution by attributing organic demand to paid touchpoints.

---

## Methodology

### Why Geo-Experiment over User-Level A/B

User-level holdout is theoretically ideal but operationally constrained in this context:

- YouTube and Meta do not guarantee full exclusion of control users from ad exposure (ad spillover)
- Video+ lacks programmatic control over per-user inventory allocation across platforms
- Geographic clustering of media buying makes region-level randomization more tractable

**Geo-experiments use geographic regions (DMAs) as the unit of randomization.** Treated regions receive the campaign; control regions receive zero spend. The counterfactual for treated regions is estimated via Synthetic Control Method (SCM).

### Synthetic Control Method

The SCM (Abadie, Diamond & Hainmueller, 2010) constructs a weighted combination of control regions that best replicates the pre-campaign subscription trend of each treated region. Post-campaign divergence between observed and synthetic series isolates the causal treatment effect.

Statistical significance is assessed via **permutation-based placebo tests**: the same procedure is applied to all control regions (which received no treatment), generating a null distribution of placebo effects. The true treatment p-value is the rank of the observed effect within this distribution.

For Bayesian structural time-series estimation, the study also implements **CausalImpact** (Brodersen et al., 2015) as a robustness check.

---

## Experiment Design

| Parameter | Value |
|---|---|
| **Study period** | 8 weeks (4 pre / 4 during campaign) |
| **Granularity** | State-level regions (DMA-equivalent) |
| **Treated regions** | SP Interior, RS, PE |
| **Control regions** | MT, PA, RN, ES |
| **Primary metric** | New Família plan subscriptions / week / region |
| **Secondary metric** | Activation rate within 7 days of trial start |
| **Spend channels** | YouTube (60%), Meta (40%) |

**Treated regions** were selected based on similar pre-period trends and media budget feasibility. **Control regions** were validated via parallel trends test on the pre-period (weeks −4 to −1).

---

## Synthetic Data Simulation

All data is synthetically generated to replicate realistic Brazilian streaming market dynamics. Key simulation parameters:

| Parameter | Value | Source/Rationale |
|---|---|---|
| Baseline subscriptions/week/region | 180–420 | Calibrated to ~1.2M MAU base |
| Organic growth trend | +0.8%/week | Brazilian SVOD market 2023 |
| Seasonality | Weekly + monthly cycles | Streaming consumption patterns |
| Campaign lift (treated, ground truth) | +22% | Embedded in DGP for validation |
| Noise | Negative Binomial (r=15) | Overdispersed count data |
| Inter-region correlation (pre-period) | 0.65–0.80 | Enables SCM feasibility |

Simulation is fully reproducible via fixed random seed. Ground truth lift is known, enabling **model validation** — the study confirms whether SCM recovers the embedded 22% effect.

---

## Results

### Synthetic Control Fit (Pre-Period)

Pre-period RMSE between treated regions and their synthetic counterparts: **< 4.2 subscriptions/week** — confirming the synthetic control closely tracks observed trends before the campaign.

### Incremental Effect

| Region | Observed Subs (4w) | Synthetic Counterfactual | Incremental Subs | Incremental Revenue |
|---|---|---|---|---|
| SP Interior | 1,847 | 1,512 | +335 | R$ 134,000 |
| RS | 1,203 | 1,008 | +195 | R$ 78,000 |
| PE | 986 | 841 | +145 | R$ 58,000 |
| **Total** | **4,036** | **3,361** | **+675** | **R$ 270,000** |

*Revenue calculated as incremental subscriptions × LTV (R$ 400, 12-month horizon at 8% monthly churn)*

### iROAS vs. Platform-Reported ROAS

| Metric | Value |
|---|---|
| Total media spend (4 weeks) | R$ 87,000 |
| Platform-reported ROAS | 9.2x |
| **iROAS (GeoLift)** | **3.1x** |
| Incrementality rate | 33.7% |
| Placebo test p-value | 0.04 |

> **Interpretation:** 66.3% of attributed conversions would have occurred organically. The campaign is profitable (iROAS 3.1x > breakeven 2.5x), but significantly less efficient than platform metrics suggested. Budget reallocation from Meta (iROAS 2.1x) to YouTube (iROAS 4.4x) is recommended.

### CausalImpact Robustness Check

Bayesian structural time series estimate: **+668 incremental subscriptions [95% CI: 541–793]**, consistent with SCM estimate of +675. Both methods agree on direction, magnitude and significance.

---

## Business Decision

Based on iROAS estimates disaggregated by channel:

| Channel | Spend | iROAS | Recommendation |
|---|---|---|---|
| YouTube | R$ 52,200 | 4.4x | **Increase allocation** |
| Meta | R$ 34,800 | 2.1x | Reduce; near breakeven |

**Recommended action:** Reallocate 30% of Meta budget to YouTube for next campaign cycle. Expected improvement in portfolio iROAS: from 3.1x to ~3.9x, assuming stable incrementality curves.

---

## Repository Structure

```
geolift-incrementality-clarotv/
│
├── data/                        # DADOS SINTÉTICOS
│   ├── raw/                     # Dados originais (imutáveis)
│   │   └── video+_media.csv     # Seu dataset sintético considerando campanhas de geo-lift
│   ├── raw/                     # Dados originais (imutáveis)
│   │   └── video+_organic.csv   # Seu dataset sintético consideranco apenas o crescimento organico da plataforma
│
├── notebooks/
│   ├── 01_parallel_trends.ipynb   # Pre-period validation
│   ├── 02_synthetic_control.ipynb # SCM estimation + placebo tests
│   └── 03_causal_impact.ipynb     # Bayesian robustness check
│
├── src/
│   ├── scm.py                     # Synthetic Control implementation
│   ├── causal_impact_wrapper.py   # CausalImpact interface
│   └── metrics.py                 # iROAS, iCAC, LTV calculations
│
├── outputs/
│   ├── figures/                   # All charts (parallel trends, gap plots)
│   └── results_summary.csv        # Incremental effects by region
│
├── requirements.txt
└── README.md
```

---

## How to Run

```bash
git clone https://github.com/brunobgo/geolift-incrementality-clarotv.git
cd geolift-incrementality-clarotv
pip install -r requirements.txt

# Generate synthetic data
python data/simulate_geo_data.py

# Run full analysis
jupyter notebook notebooks/02_synthetic_control.ipynb
```

---

## Key Concepts Demonstrated

- **Geo-experiment design** with treated/control region selection
- **Synthetic Control Method** (Abadie et al., 2010) with permutation inference
- **CausalImpact** (Brodersen et al., 2015) Bayesian structural time series
- **iROAS / iCAC** calculation from causal estimates
- **Placebo testing** for statistical validity
- **Budget reallocation** decision framework based on incremental returns

---

## References

- Abadie, A., Diamond, A., & Hainmueller, J. (2010). *Synthetic Control Methods for Comparative Case Studies.* Journal of the American Statistical Association, 105(490), 493–505.
- Brodersen, K. H., Gallusser, F., Koehler, J., Remy, N., & Scott, S. L. (2015). *Inferring causal impact using Bayesian structural time-series models.* Annals of Applied Statistics, 9(1), 247–274.
- Meta Open Source. (2022). *GeoLift: Geo-Experimentation via Synthetic Control.* https://github.com/facebookincubator/GeoLift
- Vaver, J., & Koehler, J. (2011). *Measuring Ad Effectiveness Using Geo Experiments.* Google Technical Report.

---

## Author

**Bruno** · Growth Architect & Decision Science Lead  
Causal Inference · Marketing Mix Modeling · Subscription Analytics  
[LinkedIn](https://linkedin.com/in/brunobgo) · [GitHub](https://github.com/brunobgo)
