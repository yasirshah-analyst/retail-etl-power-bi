# Retail Sales Performance Dashboard: Multi-Source ETL & Analysis

An end-to-end ETL project using Power Query (Power BI) to extract, clean, and merge data from four different source formats — CSV, Excel, a pipe-delimited text file, and (in an earlier iteration) a live web API — into a single unified sales model, then visualized as an interactive dashboard.

---

## Business Question

> A retail company sells across three channels (Online, In-Store, Wholesale), with data scattered across separate files by department. Management needs a unified view: which channels and product categories drive the most revenue and profit, and how has performance trended over time?

---

## Data Source

**Type**: Synthetic dataset, generated specifically for this project to guarantee clean referential integrity across sources (every `OrderID` in `order_items` matches a real order; every `ProductID` matches a real product).

**Sources**:
| File | Format | Rows | Contents |
|---|---|---|---|
| `orders.csv` | CSV | 500 | OrderID, CustomerID, OrderDate, Channel, StoreID |
| `order_items.csv` | CSV | 1,043 | OrderID, ProductID, Quantity, UnitCost, UnitPrice |
| `products.xlsx` | Excel | 26 | ProductID, ProductName, Category |
| `stores.txt` | Pipe-delimited text | 10 | StoreID, StoreName, City, Country |

Date range: January 2024 – December 2025.

---

## Methodology

### 1. Extract
Each source was loaded into Power Query using its corresponding connector (Text/CSV, Excel, and Text/CSV with a pipe delimiter for the `.txt` file).

### 2. Transform
- Promoted headers and set correct data types for every column (dates as Date, IDs as Text, monetary fields as Decimal).
- Verified referential integrity between `orders` and `order_items` — confirmed zero orphaned rows (every line item traces back to a real order).

### 3. Merge
- `order_items` merged with `orders` on `OrderID` (Left Outer) to bring in date, channel, and customer context.
- Result merged with `products` on `ProductID` (Left Outer) to bring in product name and category.
- Combined into a single query, renamed `FactSales`.

### 4. Calculated Columns
| Column | Formula | Purpose |
|---|---|---|
| `TotalAmount` | `Quantity × UnitPrice` | Revenue per line item |
| `TotalCost` | `Quantity × UnitCost` | Cost per line item |
| `Profit` | `TotalAmount − TotalCost` | Profit per line item |
| `Profit Margin %` | `Total Profit ÷ Total Revenue` | Overall profitability |

### 5. Load & Visualize
`FactSales` and `stores` were both loaded into the data model. `orders`, `order_items`, and `products` were disabled from loading once their data was merged into `FactSales`, avoiding redundant tables and unwanted auto-generated relationships. `stores` was connected to `FactSales` via a **model relationship** on `StoreID` (one-to-many), rather than a Power Query merge — since `stores` is a small, purely descriptive lookup table, a relationship avoids unnecessarily duplicating store details across every sales row.

---

## Dashboard

Built in Power BI:
- **KPI Cards**: Total Revenue ($1.46M), Total Profit ($517.06K), Profit Margin % (35%)
- **Monthly Revenue Trend**: line chart, Jan 2024 – Dec 2025, using Year + Month (not the plain Month field, to avoid collapsing the same month across different years into one bucket)
- **Revenue by Sales Channel**: Online leads ($0.77M), followed by In-Store ($0.57M), then Wholesale ($0.13M)
- **Revenue by Product Category**: Electronics leads ($0.34M), followed closely by Home Goods, Beauty, Sporting Goods, and Apparel (all in the $0.27M–$0.32M range)
- **Revenue by Country**: store-level performance rolled up to country, using the `stores` relationship

---

## Key Findings

**1. Online is the dominant channel, but Wholesale is a meaningful growth gap.**
Online revenue ($0.77M) is nearly 6x Wholesale ($0.13M) — a large enough gap to warrant investigating whether this reflects a genuine market difference or an under-invested sales channel.

**2. Electronics leads, but category revenue is fairly evenly distributed overall.**
Aside from Electronics' modest lead, the remaining four categories cluster closely together (all within $0.05M of each other) — no single category is dramatically underperforming.

**3. Revenue shows clear month-to-month volatility, with a pronounced peak around mid-2025.**
The trend line shows several moderate peaks and one sharp spike (~$0.12M) around July 2025, worth investigating further (e.g., a seasonal promotion or large wholesale order) in future work.

---

## Recommendations

- Investigate the Wholesale channel specifically — determine whether the revenue gap reflects limited demand or limited sales effort/investment in that channel.
- Examine the July 2025 revenue spike to identify its cause, and assess whether it's repeatable (e.g., a seasonal pattern) or a one-time event.
- Given the relatively even category performance, consider a category-level profit margin analysis (not just revenue) to identify which categories are most worth prioritizing.
- Use the new country-level breakdown to identify whether specific regions are under-served by certain channels — a natural follow-up combining channel, category, and geographic views.

---

## Limitations

- The dataset is synthetic, generated specifically to ensure clean joins across multiple file formats for this ETL exercise — findings demonstrate the analysis method, not real market conditions.
- The live web API (currency exchange rate) source from the original project scope was set aside during this build due to a data model conflict; the pipeline currently reflects 4 of the originally planned 5 source types (CSV, CSV, Excel, and pipe-delimited text).

---

## Project Structure

```
ETL-Power-BI/
│
├── dashboard/
│   ├── ETL.pbix
│   └── dashboard.png
│
├── dataset/
│   ├── order_items.csv
│   ├── orders.csv
│   ├── products.xlsx
│   └── stores.txt
│
└── README.md
```
---

## Tools Used

Power BI (Power Query, DAX measures, dashboard design) · Excel · CSV/text file handling
