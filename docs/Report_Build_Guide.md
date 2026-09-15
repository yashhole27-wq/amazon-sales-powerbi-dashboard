# Amazon E-Commerce BI Dashboard — Build Guide

## Files you have
- `Amazon_Sales_Fact_Cleaned.csv` — your cleaned fact table (128,949 rows)
- `Dim_Date.csv` — a standalone date dimension table (91 days, Mar 31–Jun 29 2022)
- `DAX_Measures.md` — every measure you need, ready to paste into Power BI

---

## Step 1: Import and build the data model

1. Open Power BI Desktop → Get Data → Text/CSV → import both `Amazon_Sales_Fact_Cleaned.csv`
   and `Dim_Date.csv`.
2. Go to **Model view**. Drag a relationship from `Dim_Date[Date]` to
   `Amazon_Sales_Fact_Cleaned[Date]`. Set it as **1-to-many**, single direction
   (Dim_Date is the "1" side).
3. Click `Dim_Date` table → Table Tools ribbon → **Mark as Date Table** → select the `Date` column.
   This is required for all the time-intelligence DAX (MTD, MoM, rolling totals) to work correctly.
4. In Power Query (Transform Data), double check `Date`, `Amount`, `Qty` came in as the correct
   types (Date, Decimal Number, Whole Number respectively) — Power BI sometimes misreads these
   from CSV.
5. Create the `_Measures` table: Modeling ribbon → New Table → type `_Measures = {}` just to get an
   empty table to hang your measures on, then add each measure from `DAX_Measures.md` via New Measure.

This gives you an actual **star schema** — a fact table plus a dimension table — rather than one
flat sheet. Mentioning "built a star schema with a dedicated date dimension" on your resume signals
real modeling knowledge, not just drag-and-drop visuals.

---

## Step 2: Report pages

Build these as separate pages (tabs) in the report:

### Page 1 — Executive Overview
- **KPI cards along the top**: Net Revenue (Valid Orders), Total Orders, Average Order Value,
  Cancellation Rate
- **Line chart**: Net Revenue by Date (trend across the full Mar–Jun window)
- **Bar chart**: Net Revenue by Category (horizontal bar, sorted descending)
- **Donut/pie chart**: Revenue split by Fulfilment (Amazon vs Merchant)
- **Slicers at top or in a side panel**: Date range, Category, Fulfilment, B2B

### Page 2 — Geography
- **Filled map or shape map of India**: Net Revenue by Ship State (this is the standout visual —
  few beginner projects use a proper geo visual)
- **Table**: Top 10 states by revenue, with Orders, Revenue, Average Order Value columns
- **Bar chart**: Top 10 cities by revenue

### Page 3 — Product Performance
- **Bar chart**: Revenue by Category (use `Category Revenue Rank` measure to build a dynamic Top N)
- **Matrix visual**: Category × Size, values = Total Units Sold — shows which sizes sell best per category
- **Scatter plot**: Category on one axis, Average Order Value vs Total Units Sold — spots
  high-value-low-volume vs high-volume-low-value categories

### Page 4 — Order Fulfillment & Cancellations
- **KPI cards**: Cancellation Rate, Delivery Rate, Cancelled/Returned Revenue (Lost)
- **Bar chart**: Order Status breakdown (all 12 status values)
- **Bar chart**: Cancellation Rate by Category — this is a genuinely useful business insight
  (which product categories get cancelled most) that most portfolio projects don't think to build
- **B2B vs B2C** card/donut using the B2B Revenue measures

---

## Step 3: Polish (this is what separates portfolio-tier from tutorial-tier)

- Use a **consistent color theme** (View ribbon → Themes) rather than Power BI's default colors
- Add a **title text box** at the top of each page and consistent page navigation buttons
- Format all currency values with the ₹ symbol and thousands separators (this data is in INR)
- Use **tooltips** on charts to show secondary detail (e.g., hovering a state shows Top 3 categories there)
- Turn off unnecessary gridlines/borders in visuals for a cleaner look
- Add a **bookmark-based toggle** between "Include cancelled orders" and "Valid orders only" views —
  this is an advanced touch that demonstrates DAX/bookmark fluency

---

## Step 4: Known data limitations (mention these — it shows analytical maturity, not weakness)

- `Amount` is missing for ~7,794 rows (mostly cancelled orders with no transaction value)
- A handful of `Ship State` entries are inconsistent/ambiguous (e.g. "APO", abbreviations) —
  worth a note that in a real business setting you'd flag these with the data source team
  rather than silently guessing
- Dataset covers a single quarter (Mar 31–Jun 29, 2022), so year-over-year comparisons aren't
  possible with this data alone — frame trend analysis as month-over-month/weekly instead
