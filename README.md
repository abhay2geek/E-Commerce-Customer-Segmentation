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
