# Amazon E-Commerce Business Intelligence Dashboard

An interactive Power BI dashboard analyzing 128,949 Amazon India apparel orders (Q2 2022) to surface revenue trends, product performance, geographic demand, and order fulfillment health.

![Dashboard Preview](screenshots/executive-overview.png)
*(Screenshot — add after building; see `/screenshots` folder)*

## Business Problem

An Amazon third-party seller needed visibility into three questions their raw order exports couldn't answer directly:
1. Where is revenue actually coming from — which categories, states, and fulfillment channels drive the business?
2. How much revenue is being lost to cancellations and returns, and does that vary by product category?
3. Is the business more B2B or B2C, and does that split matter for revenue?

## Key Insights

- Net revenue across the quarter: ₹70,261,362 (~₹7.03 crore) across 120,352 orders
- Cancellation rate: 14.3% overall — kurta, Set, and Saree show the highest cancellation rates among categories (~14.6% each), suggesting cancellations aren't concentrated in one product line but spread fairly evenly across top sellers
- Top state by revenue: Maharashtra, accounting for ~17.2% of total revenue (₹1.2 crore), followed by Karnataka and Telangana
- B2B orders represent only 0.79% of revenue despite being 0.66% of order volume — this business is almost entirely B2C (99.2%+), so B2B-specific strategies would affect a negligible share of revenue

## Dashboard Pages

| Page | What it shows |
|---|---|
| **Executive Overview** | Headline KPIs (net revenue, orders, AOV, cancellation rate), revenue trend, category and fulfillment breakdown |
| **Geography** | Revenue by state on a map of India, top states/cities table |
| **Product Performance** | Revenue and units by category, category × size matrix, value-vs-volume scatter |
| **Fulfillment & Cancellations** | Order status breakdown, cancellation rate by category, B2B vs B2C split |

## Data

- **Source**: [Amazon Sale Report — Kaggle](https://www.kaggle.com/datasets/thedevastator/unlock-profits-with-e-commerce-sales-data) (public dataset, India apparel seller, Mar 31–Jun 29 2022)
- **`/data/sample_sales_data.csv`** — a 2,000-row representative sample of the cleaned fact table, included here so the repo stays lightweight. The full 128,949-row cleaned dataset (~68MB) isn't committed directly; download the original from the Kaggle link above and run the cleaning steps below to reproduce it.
- **`/data/Dim_Date.csv`** — a standalone date dimension table built for proper time-intelligence support in the model (91 days, Mar 31–Jun 29 2022).

## Data Model

Built as a star schema rather than a single flat table:

```
Dim_Date (1) ──────< (many) Amazon_Sales_Fact
   Date                        Date
   Year                        Order ID, Status, Category, Amount, Qty, ...
   Month Name
   Quarter
   Is Weekend
```

`Dim_Date` is marked as an official Date Table in Power BI, enabling proper MTD/rolling/period-over-period DAX.

## Data Cleaning

Starting from the raw export, the following cleaning steps were applied (see `/dax/DAX_Measures.md` for the full reasoning):

- Dropped an empty trailing column (`Unnamed: 22`)
- Standardized `Ship State` / `Ship City` casing (raw data mixed ALL CAPS and Title Case, which breaks map visuals and groupings)
- Added an `Is Valid Order` flag distinguishing cancelled/returned orders from fulfilled ones, so headline revenue metrics aren't inflated by cancelled transactions — while keeping those rows in the model for cancellation analysis
- Verified column data types (Date, Decimal, Whole Number, True/False) before loading into the model

## DAX Measures

~15 measures covering revenue, orders, cancellation/delivery rates, time intelligence, and B2B/B2C splits. Full list with explanations in [`/dax/DAX_Measures.md`](dax/DAX_Measures.md).

Notable design decisions:
- **`Total Orders` uses `DISTINCTCOUNT(Order ID)`**, not row count — ~8,600 rows in the raw data share an Order ID because of multi-SKU orders, so a naive row count would overcount orders by ~7%.
- **`Net Revenue` filters on `Is Valid Order`** instead of deleting cancelled rows from the dataset, preserving the ability to analyze cancellations as their own metric rather than hiding that signal.

## Tools Used

- **Power BI Desktop** — data modeling, DAX, report design
- **Power Query** — data type correction and load
- **Python (pandas)** — initial data cleaning and star-schema prep (see `/data`)

## How to Reproduce

1. Download the raw dataset from the Kaggle link above.
2. Open `Amazon_BI_Dashboard.pbix` in Power BI Desktop (or rebuild using the steps in [`/docs/Report_Build_Guide.md`](docs/Report_Build_Guide.md)).
3. Point the data source to your downloaded CSV, or use `/data/sample_sales_data.csv` for a quick preview with a smaller dataset.

## Repo Structure

```
├── README.md
├── Amazon_BI_Dashboard.pbix       # (add once built)
├── data/
│   ├── sample_sales_data.csv      # 2,000-row sample of the cleaned fact table
│   └── Dim_Date.csv               # date dimension table
├── dax/
│   └── DAX_Measures.md            # all measures + design reasoning
├── docs/
│   └── Report_Build_Guide.md      # full page-by-page build guide
└── screenshots/
    └── (dashboard page screenshots)
```

---

*Built as part of a Data Analyst portfolio project by Yash Sachin Hole.*
