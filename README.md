# Online Retail II — Exploratory Data Analysis & Problem Framing

`#evn-ds-epochs26-day01`

## 📦 Dataset Overview

**Source:** [Online Retail II](https://archive.ics.uci.edu/dataset/502/online+retail+ii) — UCI Machine Learning Repository

The dataset contains all transactions for a UK-based, registered non-store online retailer that
sells mostly unique, all-occasion gift items. Many of its customers are wholesalers. It covers
two sheets, **"Year 2009-2010"** and **"Year 2010-2011"**, spanning **01 Dec 2009 – 09 Dec 2011**.

Combined, the dataset has:

| Property | Value |
|---|---|
| Rows (transaction line items) | 1,067,371 |
| Columns | 8 |
| Unique invoices | 53,628 |
| Unique products (`StockCode`) | 5,305 |
| Unique customers (`Customer ID`) | 5,942 |
| Countries | 43 |
| Date range | 2009-12-01 → 2011-12-09 |

**Columns:**

| Column | Type | Description |
|---|---|---|
| `Invoice` | object | Invoice number. Starts with `C` for cancellations. |
| `StockCode` | object | Product/item code. |
| `Description` | object | Product name. |
| `Quantity` | int | Units per line item (negative = return/cancellation). |
| `InvoiceDate` | datetime | Date/time of the transaction. |
| `Price` | float | Unit price in GBP (£). |
| `Customer ID` | float | Customer identifier (missing ≈ 23% of rows — guest orders). |
| `Country` | object | Customer's country. |

Each row is a **single product line within an invoice** — i.e. transaction-level, not
customer- or invoice-level, data. This shapes every business problem below: most useful
framings require *aggregating* rows up to the invoice, customer, or product level first.

## 💼 Business Problem(s)

This transactional log directly supports several business problems for an online retailer:

1. **Customer segmentation** — Group customers by purchasing behavior (how recently, how
   often, how much they spend) so marketing and account teams can target high-value
   customers, at-risk customers, and bargain-focused wholesalers differently, instead of
   treating all ~5,900 customers the same way.
2. **Customer churn prediction** — Flag customers who are likely to stop ordering, so the
   business can intervene (discounts, outreach) before losing them — valuable given how
   concentrated revenue is in repeat wholesale buyers.
3. **Demand / revenue forecasting** — Predict next month's sales volume or revenue, useful
   for inventory and cash-flow planning, especially given the strong pre-Christmas seasonality
   visible in the data.
4. **Product association / market basket analysis** — Identify which products are frequently
   bought together, to drive cross-sell recommendations and bundle offers.

Of these, this repository focuses on **customer segmentation** as the primary problem, since
the dataset's customer-level transaction history maps onto it most directly and with the
least additional assumption-building, while noting forecasting and churn as natural extensions.

## 🤖 ML Problem Framing

**Primary: Clustering (unsupervised learning) — customer segmentation.**

- There is no pre-existing "segment" label in the data, so this cannot be posed as
  classification out of the box — it's a discovery problem: find natural groupings in
  customer behavior. Clustering (e.g. K-Means, hierarchical clustering, or DBSCAN on
  standardized RFM features) is the natural fit.
- Justification: the raw data is one row per invoice line, not one row per customer. To
  segment customers we first aggregate to **Recency, Frequency, Monetary (RFM)** features per
  customer, then cluster on that feature space — a textbook clustering setup with no ground
  truth labels to train against.

**Secondary framings** (natural next steps, not built out in this notebook):

- **Regression** for revenue/demand forecasting (target: monthly or weekly revenue), since the
  monthly trend shows clear seasonality that a time-aware regression model could capture.
- **Classification** for churn prediction, once a churn definition (e.g. "no purchase in the
  next N days") is decided and used to create a binary label per customer.

## 🎯 Target Variable & Key Features

Since the chosen framing is **clustering**, there is **no target variable** — the model finds
structure without labels. The features are engineered per customer from the raw transaction
log:

| Feature | Definition |
|---|---|
| **Recency** | Days since the customer's most recent purchase (relative to a snapshot date). |
| **Frequency** | Number of distinct invoices (orders) placed. |
| **Monetary** | Total revenue (`Quantity × Price`, summed) from the customer. |

For the secondary framings, target variables would be:
- **Forecasting:** monthly/weekly aggregated `Revenue` (regression target).
- **Churn:** a binary "churned / active" label derived from a chosen inactivity window
  (classification target).

## 🔑 Three Key Observations

1. **~23% of transaction rows have no `Customer ID`.** These are guest/unattributed orders
   and cannot be tied to a specific customer, which limits any customer-level model (RFM,
   segmentation, churn, CLV) to the ~77% of rows that do carry an ID.
2. **Revenue is strongly seasonal**, peaking every September–November ahead of Christmas and
   dropping sharply afterward — this should inform both demand forecasting models and how a
   "churn" window is defined (a quiet month in mid-year isn't necessarily churn).
3. **Cancellations and returns are non-trivial**: invoices starting with `"C"` account for
   ~1.8% of rows, and negative `Quantity` appears in 22,950 rows overall. These must be
   filtered out (or modeled as their own signal) before computing revenue, RFM features, or
   any downstream target — otherwise metrics like a customer's `Monetary` value get distorted.

## 📁 Repository Contents

- `analysis.ipynb` — Pandas-based exploration: shape, dtypes, missing values, summary
  statistics, data-quality checks, and RFM feature construction.
- `README.md` — this file.

## 🛠️ How to Reproduce

1. Download `online_retail_II.xlsx` from the [UCI dataset page](https://archive.ics.uci.edu/dataset/502/online+retail+ii)
   and place it alongside `analysis.ipynb`.
2. `pip install pandas numpy matplotlib openpyxl`
3. Run `analysis.ipynb` top to bottom.
