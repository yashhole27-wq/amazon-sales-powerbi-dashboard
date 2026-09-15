# DAX Measures — Amazon E-Commerce BI Dashboard

Create these as a new Measure table in Power BI (Modeling > New Table, name it `_Measures`,
then add each measure below via New Measure). Keeping measures in their own table instead of
scattered across your fact table is a best practice worth mentioning in interviews.

---

## Revenue Measures

```
Total Revenue =
SUM(Amazon_Sales_Fact_Cleaned[Amount])
```

```
Net Revenue (Valid Orders) =
CALCULATE(
    [Total Revenue],
    Amazon_Sales_Fact_Cleaned[Is Valid Order] = TRUE
)
```

```
Cancelled/Returned Revenue (Lost) =
CALCULATE(
    [Total Revenue],
    Amazon_Sales_Fact_Cleaned[Is Valid Order] = FALSE
)
```

```
Average Order Value =
DIVIDE([Net Revenue (Valid Orders)], [Total Orders])
```

---

## Order & Volume Measures

```
Total Orders =
DISTINCTCOUNT(Amazon_Sales_Fact_Cleaned[Order ID])
```

```
Total Line Items =
COUNTROWS(Amazon_Sales_Fact_Cleaned)
```

```
Total Units Sold =
CALCULATE(
    SUM(Amazon_Sales_Fact_Cleaned[Qty]),
    Amazon_Sales_Fact_Cleaned[Is Valid Order] = TRUE
)
```

```
Items Per Order =
DIVIDE([Total Line Items], [Total Orders])
```

---

## Cancellation / Fulfillment Health

```
Cancelled Orders =
CALCULATE(
    DISTINCTCOUNT(Amazon_Sales_Fact_Cleaned[Order ID]),
    Amazon_Sales_Fact_Cleaned[Status] = "Cancelled"
)
```

```
Cancellation Rate =
DIVIDE([Cancelled Orders], [Total Orders])
```

```
Delivered Orders =
CALCULATE(
    DISTINCTCOUNT(Amazon_Sales_Fact_Cleaned[Order ID]),
    Amazon_Sales_Fact_Cleaned[Status] = "Shipped - Delivered to Buyer"
)
```

```
Delivery Rate =
DIVIDE([Delivered Orders], [Total Orders])
```

---

## Time Intelligence (requires the Dim_Date table, marked as a Date Table, related on Date)

```
Revenue MTD =
TOTALMTD([Net Revenue (Valid Orders)], Dim_Date[Date])
```

```
Revenue Previous Month =
CALCULATE(
    [Net Revenue (Valid Orders)],
    DATEADD(Dim_Date[Date], -1, MONTH)
)
```

```
Revenue MoM % Change =
DIVIDE(
    [Net Revenue (Valid Orders)] - [Revenue Previous Month],
    [Revenue Previous Month]
)
```

```
7-Day Rolling Revenue =
CALCULATE(
    [Net Revenue (Valid Orders)],
    DATESINPERIOD(Dim_Date[Date], MAX(Dim_Date[Date]), -7, DAY)
)
```

---

## Ranking / Top-N Helpers (for dynamic Top 5 Category / State visuals)

```
Category Revenue Rank =
RANKX(
    ALL(Amazon_Sales_Fact_Cleaned[Category]),
    [Net Revenue (Valid Orders)],
    ,
    DESC
)
```

```
State Revenue Rank =
RANKX(
    ALL(Amazon_Sales_Fact_Cleaned[Ship State]),
    [Net Revenue (Valid Orders)],
    ,
    DESC
)
```

---

## B2B vs B2C Split

```
B2B Revenue =
CALCULATE(
    [Net Revenue (Valid Orders)],
    Amazon_Sales_Fact_Cleaned[B2B] = TRUE
)
```

```
B2C Revenue =
CALCULATE(
    [Net Revenue (Valid Orders)],
    Amazon_Sales_Fact_Cleaned[B2B] = FALSE
)
```

```
B2B Revenue % =
DIVIDE([B2B Revenue], [Net Revenue (Valid Orders)])
```

---

### Why these specific measures (for your own understanding / interview prep)

- **`Total Orders` uses `DISTINCTCOUNT(Order ID)`, not `COUNTROWS`** — because ~8,600 rows share
  an Order ID (multi-SKU orders). Using COUNTROWS would silently overcount orders. This distinction
  is exactly the kind of data-grain issue a hiring manager wants to see you catch.
- **`Net Revenue` filters on `Is Valid Order`** rather than deleting cancelled rows from the model —
  so you can still analyze cancellations (rate, by category, by state) without losing that signal
  from the dataset, while keeping the headline revenue number honest.
- **Time intelligence measures depend on `Dim_Date` being marked as an official Date Table** in
  Power BI (Modeling ribbon > Mark as Date Table) with a proper 1-to-many relationship to the fact
  table's Date column — flat "just use the Date column from the fact table" approaches break once
  you add rolling/MTD/YoY calculations.
