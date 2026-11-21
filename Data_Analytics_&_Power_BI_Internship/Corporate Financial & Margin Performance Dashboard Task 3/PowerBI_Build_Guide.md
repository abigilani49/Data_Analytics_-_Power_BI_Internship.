# Financial KPI Dashboard: Power BI Build Kit

Files: `financial_sample.xlsx` (actuals), `Budget.csv` (budget targets), this guide.
**Reconciliation totals (your report must match):** Gross Sales 127,931,600 · Discounts 9,205,248 · Net Sales 118,726,400 · COGS 101,832,600 · Profit 16,893,700 · Gross Profit % 14.23% · COGS Ratio 85.77%.

## 0. Read this first (data findings)
1. **The dataset has no budget.** `Budget.csv` is a *placeholder I generated* (Gross Sales = 95% of actual, discounts 6.5% of gross, COGS 83% of net). Replace with your internship's real targets if provided, and say so in your report.
2. **No "cost center" column exists.** Use **Product** (it carries Manufacturing Price) as the manufacturing cost-center proxy and document that.
3. Column `Sales` is **Net Sales** (Gross − Discounts). It has a leading space (` Sales`) in the file.
4. `Discount Band` is blank for 53 rows (no discount): replace with "None".
5. Identities hold exactly: Gross − Discounts = Sales; Sales − COGS = Profit.
6. Finding worth presenting: **Enterprise segment loses money** (Profit −614,546; COGS exceeds net sales). Government is the profit engine (11.4M).

## 1. Load data (Power Query)
Home > Get Data > Excel > `financial_sample.xlsx` > Sheet1 > Transform. Then:
```
let
  Source = Excel.Workbook(File.Contents("C:\path\financial_sample.xlsx"), null, true),
  Data = Source{[Item="Sheet1",Kind="Sheet"]}[Data],
  Hdr = Table.PromoteHeaders(Data, [PromoteAllScalars=true]),
  Trim = Table.TransformColumnNames(Hdr, Text.Trim),
  Ren = Table.RenameColumns(Trim, {{"Sales","Net Sales"}}),
  Band = Table.ReplaceValue(Ren, null, "None", Replacer.ReplaceValue, {"Discount Band"}),
  Types = Table.TransformColumnTypes(Band, {{"Units Sold", type number},{"Gross Sales", Currency.Type},{"Discounts", Currency.Type},{"Net Sales", Currency.Type},{"COGS", Currency.Type},{"Profit", Currency.Type},{"Date", type date},{"Year", Int64.Type}})
in Types
```
Name it **Financials**. Import `Budget.csv` as **Budget**.

## 2. Model
- Create a **Calendar** table: `Calendar = CALENDAR(DATE(2013,1,1), DATE(2014,12,31))`, add `Year`, `Month Number`, `Month Name`; mark as date table.
- Create dimensions (Modeling > New table):
  `DimSegment = VALUES(Financials[Segment])`, `DimCountry = VALUES(Financials[Country])`, `DimProduct = VALUES(Financials[Product])`.
- Relationships (all one-to-many, single direction): DimSegment/DimCountry/DimProduct → Financials **and** → Budget; Calendar[Date] → Financials[Date].
  Budget is yearly, so relate it by Segment/Country/Product and use `Year` from a shared **DimYear** (`DimYear = VALUES(Financials[Year])` related to both tables).
- Hide the raw key columns on fact tables.

## 3. Core measures (income statement hierarchy)
```DAX
Gross Sales   = SUM(Financials[Gross Sales])
Discounts     = SUM(Financials[Discounts])
Net Sales     = [Gross Sales] - [Discounts]
COGS          = SUM(Financials[COGS])
Profit        = [Net Sales] - [COGS]

Gross Profit %  = DIVIDE([Profit], [Net Sales])
COGS Ratio      = DIVIDE([COGS], [Net Sales])
Discount Rate   = DIVIDE([Discounts], [Gross Sales])
Segment Margin  =
    VAR SegProfit = [Profit]
    VAR SegSales  = [Net Sales]
    RETURN DIVIDE(SegProfit, SegSales)
Segment Margin Contribution % =
    DIVIDE([Profit], CALCULATE([Profit], ALL(Financials[Segment])))
```
Tip: put `Segment` on a visual axis so `[Segment Margin]` evaluates per segment automatically.

## 4. Budget and variance measures
```DAX
Budget Gross Sales = SUM(Budget[Budget Gross Sales])
Budget Discounts   = SUM(Budget[Budget Discounts])
Budget Net Sales   = SUM(Budget[Budget Net Sales])
Budget COGS        = SUM(Budget[Budget COGS])
Budget Profit      = SUM(Budget[Budget Profit])

Net Sales Var      = [Net Sales] - [Budget Net Sales]
Net Sales Var %    = DIVIDE([Net Sales Var], [Budget Net Sales])
COGS Var           = [COGS] - [Budget COGS]          -- positive = overspend
Profit Var         = [Profit] - [Budget Profit]
Profit Var %       = DIVIDE([Profit Var], ABS([Budget Profit]))
Variance Flag      = IF([Profit Var] >= 0, "▲ Favorable", "▼ Adverse")
```

## 5. Scenario sliders (What-if parameters)
Modeling > New parameter > Numeric range (tick "Add slider"):

| Name | Min | Max | Step | Default |
|---|---|---|---|---|
| Volume Change % | -20% | 30% | 1% | 0% |
| Price Change % | -10% | 15% | 0.5% | 0% |
| Discount Rate Shift (pts) | -5% | 5% | 0.5% | 0% |
| COGS Inflation % | -10% | 20% | 0.5% | 0% |

```DAX
Scn Gross Sales = [Gross Sales] * (1 + [Volume Change % Value]) * (1 + [Price Change % Value])
Scn Discounts   = [Scn Gross Sales] * ([Discount Rate] + [Discount Rate Shift Value])
Scn Net Sales   = [Scn Gross Sales] - [Scn Discounts]
Scn COGS        = [COGS] * (1 + [Volume Change % Value]) * (1 + [COGS Inflation % Value])
Scn Profit      = [Scn Net Sales] - [Scn COGS]
Scn Gross Profit % = DIVIDE([Scn Profit], [Scn Net Sales])
Scn Profit vs Base = [Scn Profit] - [Profit]
```
(Power BI names the auto-generated measure `<Name> Value`; adjust to match.) COGS scales with volume but not price, which is the realistic behavior.

## 6. Waterfall chart (revenue erosion)
Power BI's native waterfall needs one category column. Build a helper table:
```DAX
Waterfall Steps =
DATATABLE("Step", STRING, "Order", INTEGER,
  {{"Gross Sales",1},{"Discounts",2},{"Net Sales",3},{"COGS",4},{"Profit",5}})

Waterfall Value =
SWITCH(SELECTEDVALUE('Waterfall Steps'[Step]),
  "Gross Sales", [Gross Sales],
  "Discounts",   -[Discounts],
  "Net Sales",   [Net Sales],
  "COGS",        -[COGS],
  "Profit",      [Profit])
```
Visual: **Waterfall chart** → Category = Step (sort by Order), Y = Waterfall Value. In Format > Sentiment colors, set Increase = green, Decrease = red, Total = blue. Right-click **Net Sales** and **Profit** bars > Set as total. Add a second copy bound to `Scn` measures for the scenario view.

## 7. Report pages
1. **Executive Overview.** Cards: Net Sales, Profit, Gross Profit %, COGS Ratio, Discount Rate; Year slicer; waterfall; monthly line (Net Sales vs Budget Net Sales).
2. **Variance Analysis.** Clustered bars of Actual vs Budget by Segment, Country, Product; matrix with Net Sales Var, Var %, COGS Var, Profit Var with conditional formatting (red/green on Variance Flag); decomposition tree on Profit Var.
3. **Segment & Product Profitability.** Segment Margin bar (Enterprise shows negative), scatter of Discount Rate vs Gross Profit % by Product, Discount Band stacked column.
4. **Scenario Planner.** The four sliders, scenario waterfall, KPI cards for Scn Profit and Scn Profit vs Base, base vs scenario clustered column.

Add bookmarks (Reset Scenario) and a page navigator. Use a consistent theme (View > Themes).

## 8. QA checklist before saving the .pbix
- [ ] Totals match the reconciliation line at the top.
- [ ] `Net Sales − COGS = Profit` in a table visual.
- [ ] Segment slicer filters Budget as well as Financials.
- [ ] Sliders at defaults give Scn Profit = Profit.
- [ ] File > Save as `Financial_KPI_Dashboard.pbix`.
