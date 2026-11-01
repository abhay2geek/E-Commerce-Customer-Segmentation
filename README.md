# E-Commerce Customer Segmentation with RFM Analysis

Segmenting retail customers by Recency, Frequency, and Monetary value using K-Means and DBSCAN, with PCA and t-SNE visualisations, to drive actionable marketing personas.

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![scikit-learn](https://img.shields.io/badge/scikit--learn-1.3%2B-orange)
![License](https://img.shields.io/badge/License-MIT-green)

---

## Overview

This project builds an end-to-end **customer segmentation pipeline** for e-commerce order data. It translates raw transactional records into **RFM (Recency, Frequency, Monetary)** features, clusters customers with both centroid-based (**K-Means**) and density-based (**DBSCAN**) methods, and reduces dimensionality with **PCA** and **t-SNE** for interpretable visualisation. The end goal is to produce **marketing personas** that can be directly consumed by CRM, growth, and lifecycle teams.

The pipeline is **schema-agnostic**: it auto-detects the customer, date, amount, and order-ID columns from an arbitrary `orders.csv`, and falls back to explicit overrides when the file uses non-standard names.

---

## Why RFM?

RFM is one of the most durable and interpretable frameworks in retail analytics:

- **Recency** — how recently did the customer purchase? (Lower is better.)
- **Frequency** — how often do they purchase? (Higher is better.)
- **Monetary** — how much do they spend? (Higher is better.)

Grouping customers along these three axes yields segments that map naturally onto **actionable marketing plays** — VIP retention, win-back campaigns, loyalty rewards, upsell funnels — without requiring a large model or labelled data.

---

## Dataset

The pipeline expects a single CSV named `orders.csv` with at minimum:

| Column | Type | Role |
|---|---|---|
| `customer_id` | str | Unique customer identifier |
| `order_date` | date | Timestamp for Recency & Frequency |
| `total_amount_usd` | float | Line total for Monetary |
| `order_id` | str | (Optional) Used to compute Frequency correctly when the file is line-item granular |
| `quantity`, `unit_price_usd` | int, float | (Optional) Fallback for computing Monetary if no amount column exists |
| `order_status` | str | (Optional) Used to exclude cancelled/returned orders |

The reference schema used during development contains **25 000 line items across 28 columns**, including product, category, payment method, delivery info, and customer rating — but only the four columns above are strictly required.

> A small synthetic `sample_orders.csv` (250 rows) is included so you can run the pipeline without real data.

---

## Methodology
orders.csv
│
▼
[1] Column resolution & cleaning
│ · auto-detect customer / date / amount / order_id
│ · drop cancelled & returned orders
│ · coerce numeric fields, filter non-positive totals
▼
[2] RFM aggregation (pandas groupby)
│ Recency = (max_date + 1d − last_order_date).days
│ Frequency = nunique(order_id) ← line-item safe
│ Monetary = sum(line_total)
▼
[3] Feature engineering
│ · log1p(Monetary) → reduce right-skew
│ · StandardScaler → zero-mean, unit-variance
▼
[4] Clustering
│ · K-Means k ∈ [2..10], choose via silhouette
│ · DBSCAN eps=0.8, min_samples=10 → outlier flag
▼
[5] Dimensionality reduction
│ · PCA 2-D linear projection (explained-variance reported)
│ · t-SNE 2-D non-linear projection (auto-subsampled > 5 k)
▼
[6] Persona derivation
│ · centroid RFM per cluster
│ · heuristic labelling: VIP, Loyal, New, Dormant, Big Spenders, Regular
│ · % customers & % revenue per segment
▼
personas.csv + customers_with_segments.csv

text

---

## Project Structure
.
├── orders.csv # input (not tracked; use your own)
├── sample_orders.csv # tiny synthetic sample for demos
├── segmentation.py # main pipeline script
├── notebooks/
│ └── exploration.ipynb # EDA + iteration
├── outputs/
│ ├── personas.csv # cluster-level RFM summaries + persona labels
│ ├── customers_with_segments.csv
│ ├── elbow_silhouette.png
│ ├── pca_projection.png
│ └── tsne_projection.png
├── requirements.txt
└── README.md

text

---

## Installation

```bash
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>

python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
requirements.txt

txt
pandas>=2.0
numpy>=1.24
scikit-learn>=1.3
matplotlib>=3.7
seaborn>=0.13
Usage
Basic run (auto-detects all columns)
bash
python segmentation.py --input orders.csv
With explicit column overrides
bash
python segmentation.py \
  --input orders.csv \
  --customer-col customer_id \
  --date-col order_date \
  --amount-col total_amount_usd \
  --order-id-col order_id
Programmatic
python
from segmentation import run_pipeline

run_pipeline(
    orders_path="orders.csv",
    customer_col="customer_id",
    date_col="order_date",
    amount_col="total_amount_usd",
    order_id_col="order_id",
    outdir="outputs/",
)
The script will:

Print detected column mapping and RFM summary statistics.

Show three plots (elbow + silhouette, PCA, t-SNE).

Print per-cluster persona interpretations.

Save personas.csv and customers_with_segments.csv to the working directory.

Outputs
personas.csv
Cluster	Size	Pct_Customers	Pct_Revenue	Avg_Recency	Avg_Frequency	Avg_Monetary
2	812	12.4 %	41.8 %	24.1	18.3	4 812
0	2 104	32.1 %	28.6 %	47.6	9.2	1 943
...	...	...	...	...	...	...
customers_with_segments.csv
Per-customer table with CustomerID, Recency, Frequency, Monetary, KMeans_Cluster, DBSCAN_Cluster, PCA1, PCA2, TSNE1, TSNE2 — ready to be joined back to CRM or exported to a BI tool.

Persona labels (heuristic)
Persona	RFM signature	Recommended action
VIP / Champions	Low R, high F, high M	Concierge service, early product access, loyalty tiers
Loyal Customers	Low R, high F	Points, referral program, premium upsell
New / Promising	Low R, low F	Onboarding series, second-purchase discount
Big Spenders (occasional)	Moderate R, low F, high M	Personalised bundles, limited editions
Dormant / At Risk	High R, low F	Win-back campaign with deep discount, or suppress
Regular / Core	Mid RFM	Cross-sell, seasonal offers, steady engagement
Design decisions worth noting
Frequency counts unique order_id, not unique dates. When the input is line-item granular, two orders placed on the same day must count as two purchases, not one.

Cancelled/returned orders are excluded before RFM aggregation. Their inclusion systematically inflates Frequency and depresses Recency accuracy.

DBSCAN runs in parallel with K-Means, not instead of it. It surfaces anomalies (bulk-buyers, wholesale accounts, test accounts) that would otherwise distort cluster centroids. Those customers are flagged with DBSCAN_Cluster == -1 and can be handled separately.

PCA and t-SNE are used together. PCA gives a linear, globally-consistent projection with a stated explained-variance ratio. t-SNE reveals local neighbourhood structure that PCA flattens. Both colour by the same K-Means labels so the two views are directly comparable.

t-SNE is subsampled above 5 000 customers. Beyond that threshold the O(n²) complexity stops being interactive, and the visual insight degrades regardless.

Future work
Dynamic RFM weighting — let analysts tilt Recency / Frequency / Monetary weights and re-segment interactively.

Supervised personas — replace heuristic labels with a decision tree trained on analyst-confirmed segments, so labels stay consistent across runs.

Time-aware segmentation — segment on a rolling 90-day RFM window and track cluster drift over quarters to catch lifecycle movement.

Uplift modelling — combine segments with treatment/control data to estimate incremental revenue per persona.

Alternative distance metrics — try Gower distance for mixed numeric/categorical customer features.

Acknowledgements
RFM segmentation has an extensive practitioner and academic lineage; this implementation follows the standard formulation popularised in early direct-marketing literature and adapted widely in e-commerce analytics. Clustering, dimensionality reduction, and evaluation are provided by scikit-learn. Visualisations use matplotlib and seaborn.

License
MIT — see LICENSE for details.
