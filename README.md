# Data_Analytics_-_Power_BI_Internship.
Business Analytics Portfolio: Superstore & Financial Performance

A three-part data analytics project covering exploratory data analysis (Python) and interactive business-intelligence dashboards (Power BI) built on retail and corporate financial datasets.

Repository Structure
Task	Focus	Tool	Dataset	Deliverable
1	Exploratory Data Analysis on Superstore Sales	Python (Pandas, Matplotlib, Seaborn), Jupyter	task_1.csv	.ipynb notebook
2	Executive Sales & Profit Dashboard	Power BI, DAX, Power Query	task_2.csv	.pbix file + PDF report
3	Corporate Financial & Margin Performance Dashboard	Power BI, DAX	task_3.xlsx (Microsoft Financial Sample)	.pbix file.




Task 1: Superstore Sales EDA

Goal: Understand purchase volume, monthly seasonality, category revenue and discount sensitivity.

What I did

Cleaned the data: the raw file has 10,800 rows. I removed 504 duplicate rows and about 300 blank records, leaving 9,994 valid orders (2015 to 2018).
Parsed order dates and extracted Year-Month for time-series analysis.
Aggregated Sales and Profit by Category, Region and Customer Segment.
Plotted monthly sales trends and discount-vs-profit scatter plots.
Documented at least 5 annotated business findings in Markdown cells.

Key findings

Total sales of about $2.30M and profit of about $286K (roughly 12.5% margin).
Technology leads in sales and profit; Furniture has similar sales but very low profit (about $18K).
Sales grew steadily, from about $484K (2015) to about $733K (2018), and peak in Q4 (Sept, Nov and Dec).
Discounts of 30% or more turn orders loss-making, so heavy discounting is the main profit leak.
Tables, Bookcases and Supplies lose money, while Copiers, Phones and Accessories are the most profitable.




Task 2: Power BI Executive Sales & Profit Dashboard

Goal: Turn the raw sales records into an interactive, multi-page executive dashboard.

What I did

Loaded and cleaned the data with Power Query.
Modelled it as a star schema: Fact Orders, Dim Customer, Dim Product, Dim Date.
Wrote DAX measures: [Total Sales], [Total Profit], [Profit Margin %] and [YoY Sales Growth %].
Built the visuals:
KPI cards (Sales, Profit, Orders)
Monthly sales line/waterfall chart
Category and Sub-Category treemap
Regional performance view
Date and Region slicers
Exported the report as a .pbix file and a PDF.



Task 3: Corporate Financial & Margin Performance Dashboard

Goal: Analyse revenue erosion and margin performance across segments, products and countries.

What I did

Connected Power BI to the Microsoft Financial Sample dataset (700 records, Sept 2013 to Dec 2014, 5 segments, 5 countries, 6 products).
Modelled the income statement flow: Gross Sales → Discounts → Net Sales → COGS → Profit.
Created DAX measures: [Gross Profit %], [COGS Ratio] and [Segment Margin].
Built a waterfall chart showing how discounts and COGS reduce revenue.
Added variance analysis and interactive scenario-forecasting sliders.
How to Run
Task 1: open the notebook in Jupyter and run all cells (pip install pandas matplotlib seaborn).
Tasks 2 & 3: open the .pbix files in Power BI Desktop.
Tools

Python · Pandas · Matplotlib · Seaborn · Jupyter · Power BI · DAX · Power Query · Excel
