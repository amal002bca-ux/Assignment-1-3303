# MapleFreight delay prediction (Slack alerts)

AML 3303 Assessment 1 · Student ID c0959761

## Problem

MapleFreight moves about 8,000 shipments a week across Ontario, Quebec and the Prairies. On-time performance dropped from 94% to 88%. Late loads are often noticed only after the promised window, when credits and customer calls have already started.

This repo trains a late-delivery classifier and posts high-risk shipments to Slack (`#dispatch-alerts`) while there is still time to react.

## Dataset

`data/maplefreight_delivery_delay_dataset.csv` - 6,035 shipments, 23 fields.

| Item | Notes |
|------|--------|
| Target | `delivered_late` |
| Late share | ~20.6% |
| Leakage removed | `actual_transit_hours` |
| Cleaning | duplicates dropped; `service_level` casing fixed; missing values imputed in the pipeline |

## Layout

```
.
├── data/maplefreight_delivery_delay_dataset.csv
├── screenshots/slack_alert.png
├── MapleFreight_Delivery_Delay_Prediction_c0959761.ipynb
├── requirements.txt
├── .env.example
├── .gitignore
└── README.md
```

## Run

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate
pip install -r requirements.txt
```

Slack:

1. Channel `#dispatch-alerts`
2. Slack app + Incoming Webhook on that channel
3. Copy `.env.example` to `.env` and set `SLACK_WEBHOOK_URL`

Open the notebook and run all cells from a clean kernel.

## Modelling

| Item | Choice |
|------|--------|
| Baseline | Majority-class dummy (~79% accuracy, 0 late recall) |
| Compared | Balanced logistic regression, random forest, hist gradient boosting |
| Selected | Balanced logistic regression (best late-class F1 on the holdout) |
| Split | 75/25 stratified, `random_state=21` |
| Extra features | `pickup_delay_hours`, `weight_per_stop`, `winter_month`, province codes |
| Alert threshold | 0.50 |
| Holdout @ 0.50 | Precision ~0.43, recall ~0.75, F1 ~0.54 on the late class |

Why 0.50: among thresholds with late recall ≥ 0.72, 0.50 had the strongest F1. Lower cuts raised alert volume without a clear F1 benefit on this split.

## Findings

- Contract Owner-Op and some partner carriers are late more often than MapleFreight Fleet.
- Storm and snow days are much worse than clear weather.
- Late trips usually leave pickup later, cover more distance and sit in heavier traffic.
- Winter months (Dec-Feb) show a higher late rate than the rest of the year in this export.

## Recommendations

1. Tighten Economy freight on storm/snow days for long lanes.
2. Cap owner-op share on premium windows until recent late counts improve.
3. Re-check ETA when pickup is already ~20+ minutes late.
4. Prefer fleet capacity on long, congested routes when possible.
5. Pilot Slack at 0.50, then raise the cut if the queue is too large.


## Checklist

- [x] Notebook named with student ID and executed with outputs
- [x] Baseline + final model, precision / recall / F1 / confusion matrix
- [x] Threshold chosen and explained
- [x] `screenshots/slack_alert.png`
- [x] `.env` gitignored; `.env.example` included
- [x] Dataset under `data/`
