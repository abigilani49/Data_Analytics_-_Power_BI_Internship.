# Superstore Executive Dashboard – Power BI Build Guide

Everything below uses the cleaned star-schema CSVs in `data/` (already produced from your file).

## 0. What I found in your CSV (data prep notes)
Your file is **three sheets stacked into one CSV**:
| Rows | Content | Handling |
|---|---|---|
| 1 – 9,994 | Orders (21 columns) | → Fact + dimensions |
| next 5 | People (Regional Manager ↔ Region) | → merged into Dim_Geography |
| last ~800 | Returns (Order ID flagged "Yes") | → `Returned` flag on Fact_Orders (296 unique orders) |

Other fixes: Postal Code converted to text with leading zeros; 1 missing postal code (Burlington, VT → 05401) filled.
Date range 2015-01-03 → 2018-12-30; 5,009 orders; 793 customers; 1,894 products.
Product ID alone is not unique (some IDs have 2 names), so Dim_Product uses a surrogate **ProductKey**.

## 1. Load data (Home → Get Data → Text/CSV)
Load the 5 CSVs from `data/`: Fact_Orders, Dim_Customer, Dim_Product, Dim_Geography, Dim_Date.
In Power Query set types: DateKey/OrderDateKey/ShipDateKey/ProductKey/GeoKey = Whole Number; Dim_Date[Date] = Date; Sales/Profit/Discount = Decimal; Postal Code = Text. Close & Apply.

## 2. Model (Model view) – Star schema
| From (many) | To (one) | Active |
|---|---|---|
| Fact_Orders[OrderDateKey] | Dim_Date[DateKey] | Yes |
| Fact_Orders[ShipDateKey] | Dim_Date[DateKey] | No (use USERELATIONSHIP) |
| Fact_Orders[Customer ID] | Dim_Customer[Customer ID] | Yes |
| Fact_Orders[ProductKey] | Dim_Product[ProductKey] | Yes |
| Fact_Orders[GeoKey] | Dim_Geography[GeoKey] | Yes |

All 1:many, single direction (Dim → Fact). Select Dim_Date → **Mark as date table** → column `Date`.
Sort `Month` by `MonthNumber`, `MonthShort` by `MonthNumber`, `DayOfWeek` by `WeekdayNumber`.
Hide key columns from Report view.

## 3. DAX measures (create a table `_Measures` via Enter Data, then add:)
```dax
Total Sales = SUM ( Fact_Orders[Sales] )

Total Profit = SUM ( Fact_Orders[Profit] )

Profit Margin % = DIVIDE ( [Total Profit], [Total Sales], 0 )

Total Orders = DISTINCTCOUNT ( Fact_Orders[Order ID] )

Total Quantity = SUM ( Fact_Orders[Quantity] )

Avg Discount % = AVERAGE ( Fact_Orders[Discount] )

Sales PY = CALCULATE ( [Total Sales], SAMEPERIODLASTYEAR ( Dim_Date[Date] ) )

YoY Sales Growth % =
VAR PY = [Sales PY]
RETURN IF ( NOT ISBLANK ( PY ), DIVIDE ( [Total Sales] - PY, PY ) )

Profit PY = CALCULATE ( [Total Profit], SAMEPERIODLASTYEAR ( Dim_Date[Date] ) )

YoY Profit Growth % =
VAR PY = [Profit PY]
RETURN IF ( NOT ISBLANK ( PY ), DIVIDE ( [Total Profit] - PY, PY ) )

Sales MoM Change =
VAR Prev = CALCULATE ( [Total Sales], DATEADD ( Dim_Date[Date], -1, MONTH ) )
RETURN [Total Sales] - Prev

Returned Orders =
CALCULATE ( DISTINCTCOUNT ( Fact_Orders[Order ID] ), Fact_Orders[Returned] = "Yes" )

Return Rate % = DIVIDE ( [Returned Orders], [Total Orders], 0 )

Loss-Making Flag = IF ( [Total Profit] < 0, "Loss", "Profit" )
```
Formatting: Sales/Profit = Currency $#,0; Margin/YoY/Return Rate = Percentage 1 dp; Orders = Whole number.
Note: SAMEPERIODLASTYEAR returns blank for 2015 (no 2014 data) — that is expected.

## 4. Report page 1 – Executive Overview (Canvas 16:9)
1. **Slicers (top strip):** Dim_Date[Date] → *Between* (date slicer); Dim_Date[Year] (tile); Dim_Geography[Region] (tile/dropdown); optional Segment.
2. **KPI cards (5):** `[Total Sales]`, `[Total Profit]`, `[Profit Margin %]`, `[Total Orders]`, `[YoY Sales Growth %]`. (Or use the KPI visual with Dim_Date[Year] as trend axis.)
3. **Monthly sales line chart:** X = Dim_Date[MonthShort], Legend = Dim_Date[Year], Y = `[Total Sales]`.
4. **Monthly waterfall:** Category = Dim_Date[YearMonth] (or Month), Y = `[Sales MoM Change]`; filter to one year via slicer.
5. **Category/Sub-Category treemap:** Group = Dim_Product[Category], Details = Dim_Product[Sub-Category], Values = `[Total Sales]`, Tooltips = `[Profit Margin %]`. Conditional format colours: Format → Colors → fx → Rules based on `[Profit Margin %]` (<0 red, 0–10% amber, >10% green).
6. **Regional treemap:** Group = Dim_Geography[Region], Values = `[Total Sales]`, same margin-based colouring.

## 5. Report page 2 – Product Profitability
- **Matrix:** Rows = Dim_Product[Category] > [Sub-Category]; Columns = Dim_Geography[Region]; Values = `[Profit Margin %]` and `[Total Profit]`. Conditional format background colour (gradient red → green, centre 0).
- **Scatter:** Details = Sub-Category, X = `[Total Sales]`, Y = `[Profit Margin %]`, Size = `[Total Profit]`.
- **Column chart:** Profit by discount band (add calculated column in Fact_Orders:
  `Discount Band = SWITCH(TRUE(), [Discount]=0,"No discount",[Discount]<=0.2,"1-20%",[Discount]<=0.4,"21-40%",">40%")`).

## 6. Interactivity
Format → Edit interactions so slicers drive every visual; add a Bookmark "Reset filters"; enable cross-filtering on the treemaps; add page navigation buttons.

## 7. Theme & polish
View → Themes → use navy `#1F2A44`, blue `#2F6DB5`, green `#2E9E5B`, red `#C8453B`, orange `#E8833A`. Background `#F4F6FA`, white card visuals with rounded corners.

## 8. Export & publish
- **.pbix:** File → Save As → `Superstore_Executive_Dashboard.pbix`.
- **PDF:** File → Export → Export to PDF (or Export as PDF in Power BI Service).
- **Publish:** Home → Publish → choose workspace (needs a work/school account; free "My workspace" works for a personal tenant).
