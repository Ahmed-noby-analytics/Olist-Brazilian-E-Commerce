Olist Brazilian E-Commerce – Sales, Delivery & Customer Satisfaction Dashboard
An interactive Power BI dashboard built with Power Query, a star-schema data model, and DAX to analyze sales, delivery performance, customer satisfaction, products, and sellers for the Olist Brazilian e-commerce marketplace.

The project explores revenue, order volume, late deliveries, customer ratings, product categories, seller performance, and freight cost by state. The goal is to turn raw transactional data into insights that can support decisions on logistics, seller management, and product strategy.
<p align="center">
  <img src="image/w1.PNG" alt="Welcome Page" width="100%">
</p>Table of Contents
1. Business Problem
2. Business Questions
3. Dataset and Source
4. Data Cleaning and Validation
5. Data Model
6. Metric Definitions
7. Dashboard Overview
8. Key Results
9. Key Insights
10. Business Recommendations
11. Limitations
12. Tools Used
13. Repository Contents
1. Business Problem
Olist connects small sellers to large online marketplaces across Brazil. As order volume grows, the business needs a consolidated view of how sales, deliveries, and customer experience are developing.

This analysis focuses on understanding where revenue comes from, how often orders arrive late, how delays affect customer ratings, and which categories, sellers, and states drive performance or cost.

The objective is to give a clear overview of business performance and to highlight opportunities for improving delivery reliability, seller management, and category strategy.
2. Business Questions
The project addresses the following business questions:
- What are total sales, delivered orders, and average order value?
- How do sales change over time, and how do 2017 and 2018 compare?
- Which product categories and states generate the most sales?
- How much of the sales value is consumed by freight, and where is freight most expensive?
- How often do orders arrive late, and when do delays peak?
- Do late deliveries reduce customer ratings, and by how much?
- Which states have the weakest delivery performance?
- How concentrated are sales among the top sellers?
- Which categories sell the most units, and which earn the most revenue?
3. Dataset and Source
The dataset is the Brazilian E-Commerce Public Dataset by Olist, published on Kaggle. It contains anonymized real commercial data from orders placed on the Olist store between 2016 and 2018.

Company and partner names in the review text were anonymized by the publisher. The dataset page lists the license as CC BY-NC-SA 4.0 (non-commercial, share-alike), so this project is for portfolio and learning purposes only.

The source files and their approximate sizes:
Table (Power BI name)	Original file	Approx. rows	Description
o_orders	olist_orders_dataset	≈ 99.4K	Order status and purchase, approval, shipping, delivery, and estimated delivery dates
o_items	olist_order_items_dataset	≈ 112K	Items per order with seller, price, and freight value
o_customers	olist_customers_dataset	≈ 99.4K	Customer city and state
o_products	olist_products_dataset	≈ 33K	Product category and physical attributes
o_sellers	olist_sellers_dataset	≈ 3.1K	Seller city and state
o_reviews	olist_order_reviews_dataset	≈ 99K	Review scores and dates
o_category	product_category_name_translation	≈ 71	Portuguese-to-English category names
o_payments	olist_order_payments_dataset	≈ 104K	Payment type, installments, and value (not used in the final model)
o_geolocation	olist_geolocation_dataset	≈ 1M	Zip-code coordinates (not used in the final model)


Row counts are approximate and follow the dataset documentation.

Period used in the dashboard: 2017 (full year) and 2018 (January – August). The few 2016 months were excluded because they contain very low order volume.

Currency note: Prices and freight values are in Brazilian reais (BRL). The dashboard shows them as plain numbers without a currency symbol.
4. Data Cleaning and Validation
Before building the dashboard, I reviewed each table for data types, missing values, and relationships using Power Query's column quality and distribution profiling.
Check	Result
Naming	Tables were renamed with a short o_ prefix for readability
Data types	Dates converted to Date, price / freight values to Decimal, and zip-code prefixes kept as Text
Order status	Eight statuses exist: delivered, shipped, canceled, unavailable, invoiced, processing, created, approved
Sales scope	Sales metrics use delivered orders only
Missing delivery dates	Blank for orders that were canceled, unavailable, or not yet delivered; these rows were kept and handled in the measures
Product categories	Portuguese names merged with the translation table; categories without a translation were labeled Unknown
Product columns	Unused description columns and the Portuguese category column were removed after the merge
Review columns	Free-text comment columns were removed because no text analysis was performed
Customer key	customer_id changes with every order, so customer_unique_id is the key for repeat-customer analysis
Geolocation	The table has around one million rows with repeated zip prefixes. It was not loaded because state-level analysis was sufficient
Time range	2016 excluded because of very low volume; 2018 data in the dashboard ends in August


These checks established a consistent foundation for the data model and the analysis.
5. Data Model
I built a star-schema data model in Power BI.

o_items and o_orders act as the fact tables. They connect to the o_products, o_sellers, o_customers, and o_calendar dimensions, and o_reviews connects to o_orders.
From (one side)	To (many side)	Key
o_orders	o_items	order_id
o_products	o_items	product_id
o_sellers	o_items	seller_id
o_orders	o_reviews	order_id
o_customers	o_orders	customer_id (one-to-one)
o_calendar	o_orders	Date → order_purchase_timestamp


Four columns were added to o_orders in Power Query to measure delivery performance:
- delivery_days: days from purchase to customer delivery.
- delay_days: customer delivery date minus estimated delivery date (positive means late).
- is_late: 1 if the order was delivered after the estimated date, otherwise 0.
- delay_gr: delay group (On time, 1–3 days late, 4–7 days late, or 8+ days late).
In o_items, a total_value column (price + freight_value) was also added.

The o_calendar table was created with DAX and marked as the date table. The Portuguese-to-English category table was merged into o_products, and o_payments, o_geolocation, and o_category are kept in Power Query with load disabled.
<p align="center">
  <img src="image/m1.PNG" alt="Data Model" width="100%">
</p>6. Metric Definitions
The following metrics were used to evaluate performance:
Metric	Definition
Total Sales	Sum of item price for delivered orders (freight excluded)
Total Freight	Sum of freight_value for delivered orders
Freight % of Sales	Total Freight ÷ Total Sales
Delivered Orders	Distinct count of order_id with status delivered
Total Orders	Distinct count of order_id across all statuses
Avg Order Value (AOV)	Total Sales ÷ Delivered Orders
Items Sold	Number of order lines for delivered orders
Avg Price	Total Sales ÷ Items Sold
Sellers / Products	Distinct sellers / products in delivered orders
Sales per Seller	Total Sales ÷ Sellers
Avg Rating	Average review_score
Avg Delivery Days	Average of delivery_days
Late Delivery %	Late orders ÷ delivered orders that have a delivery date
Cancel %	Canceled orders ÷ Total Orders
Delay Group (delay_gr)	Bucket of delay_days: On time, 1–3, 4–7, or 8+ days late


Sales are based on item price and do not include freight. The dataset contains no product cost data, so profit and margin cannot be calculated.

The measures were written in DAX and stored in a dedicated measures table.
<details>
<summary><b>View the DAX measures</b></summary>-- Sales
Total Sales = CALCULATE(SUM(o_items[price]), o_orders[order_status] = "delivered")

Total Freight = CALCULATE(SUM(o_items[freight_value]), o_orders[order_status] = "delivered")

Delivered Orders = CALCULATE(DISTINCTCOUNT(o_orders[order_id]), o_orders[order_status] = "delivered")

Total Orders = DISTINCTCOUNT(o_orders[order_id])

Items Sold = CALCULATE(COUNTROWS(o_items), o_orders[order_status] = "delivered")

Avg Order Value = DIVIDE([Total Sales], [Delivered Orders])

Freight % of Sales = DIVIDE([Total Freight], [Total Sales])

Sales LY = CALCULATE([Total Sales], SAMEPERIODLASTYEAR(o_calendar[Date]))

Sales YoY % = DIVIDE([Total Sales] - [Sales LY], [Sales LY])

-- Delivery and satisfaction
Canceled Orders = CALCULATE([Total Orders], o_orders[order_status] = "canceled")

Cancel % = DIVIDE([Canceled Orders], [Total Orders])

Avg Delivery Days = AVERAGE(o_orders[delivery_days])

Late Orders = SUM(o_orders[is_late])

Late Delivery % = DIVIDE([Late Orders], COUNT(o_orders[is_late]))

On-Time % = 1 - [Late Delivery %]

Avg Rating = AVERAGE(o_reviews[review_score])

Rating Late = CALCULATE([Avg Rating], o_orders[is_late] = 1)

Rating On-Time = CALCULATE([Avg Rating], o_orders[is_late] = 0)

-- Products and sellers
Sellers = CALCULATE(DISTINCTCOUNT(o_items[seller_id]), o_orders[order_status] = "delivered")

Products = CALCULATE(DISTINCTCOUNT(o_items[product_id]), o_orders[order_status] = "delivered")

Avg Price = DIVIDE([Total Sales], [Items Sold])

Sales per Seller = DIVIDE([Total Sales], [Sellers])
</details>7. Dashboard Overview
The project includes a multi-page interactive Power BI report with a landing page, three analysis pages, and a What-If page. Navigation buttons on every page let users move between pages.

Slicers: Year (2017 / 2018) on all pages, customer state on the Sales and Delivery pages, and product category on the Products page.
Welcome Page
A landing page with three navigation cards: Sales, Delivery, and Products.
Page 1 – Sales
- KPIs: Total Sales, Delivered Orders, Avg Rating, Avg Order Value, Late Delivery %.
- Charts: monthly Total Sales trend, top 10 categories by sales, Freight % of Sales by state (top 5), and a state map.
<p align="center">
  <img src="image/s1.PNG" alt="Sales Dashboard" width="100%">
</p>Page 2 – Delivery and Customer Satisfaction
- KPIs: Late Orders, Avg Rating, Avg Delivery Days, Late Delivery %, Cancel %.
- Charts: monthly Late Delivery % trend, customer ratings distribution, Avg Rating by delay group, and a state table with orders, delivery days, late %, and rating.
<p align="center">
  <img src="image/d1.PNG" alt="Delivery and Customer Satisfaction Dashboard" width="100%">
</p>Page 3 – Products and Sellers
- KPIs: Total Orders, Products, Sellers, Avg Price, Sales per Seller, Freight % of Sales.
- Charts: top 10 sellers by sales, Items Sold by category, and a Total Sales treemap by category.
<p align="center">
  <img src="image/p1.PNG" alt="Products and Sellers Dashboard" width="100%">
</p>What-If Page
A scenario page where a what-if parameter slider drives the Adjusted AOV and Projected Revenue cards, shown next to the actual Avg Order Value.
<p align="center">
  <img src="image/wi1.PNG" alt="What-If Scenario Page" width="100%">
</p>8. Key Results
Full Dataset (all years, including the 2016 months excluded from the dashboard views)
KPI	Result
Total Sales	13.22M
Delivered Orders	96K
Average Order Value	137
Average Rating	4.09
Late Delivery %	6.77%
Freight % of Sales	16.63%
Products Sold	32K
Active Sellers	3K


2017 vs 2018
2018 covers January to August only, so totals are not directly comparable. Ratios and averages are.
KPI	2017 (12 months)	2018 (Jan – Aug)
Total Sales	5.96M	7.22M
Delivered Orders	43,428	52,783
Average Order Value	137	137
Late Delivery %	5.65%	7.73%
Avg Delivery Days	12.98	12.06
Cancel %	0.59%	0.62%
Avg Rating	4.09	4.09
Freight % of Sales	16.08%	17.09%
Avg Price	120.33	119.66
Sales per Seller	3.53K	3.10K


Customer Rating by Delivery Delay
Delay group	2017	2018
On time	4.28	4.30
1–3 days late	3.25	3.31
4–7 days late	2.22	2.04
8+ days late	1.76	1.66


Top Product Categories by Sales
Rank	2017	Sales	2018	Sales
1	bed_bath_table	490.6K	health_beauty	755.7K
2	watches_gifts	475.6K	watches_gifts	687.9K
3	health_beauty	473.8K	bed_bath_table	532.4K
4	sports_leisure	435.7K	sports_leisure	517.2K
5	computers_accessories	391.8K	computers_accessories	496.3K


Highest Freight Share by State (top 5)
2017	Freight %	2018	Freight %
RR	42.84%	MA	27.76%
RO	25.67%	TO	27.66%
SE	25.04%	AM	25.89%
MA	24.72%	AC	25.74%
RN	23.79%	PI	24.93%


9. Key Insights
Late delivery is the main driver of low ratings. Orders delivered on time average about 4.3 stars. A delay of only one to three days drops the average to about 3.3, and a delay of eight days or more drops it to about 1.7. The pattern is almost identical in 2017 and 2018.

Delivery performance worsened in 2018 even though delivery time improved. Average delivery time fell from 12.98 to 12.06 days, but the late-delivery rate rose from 5.65% to 7.73%. Delays peaked at 18.96% in March 2018 and 14.14% in February 2018, higher than the 2017 peak of 12.40% in November. The data does not explain why. One possible cause worth testing is that estimated delivery dates became tighter.

Delays and delivery time are different problems. In 2017, Alagoas (AL) averaged 25.4 delivery days with 20.7% late orders and a 3.78 rating, while Amazonas (AM) averaged 25.6 days with 1.4% late orders and a 4.38 rating. Similar delivery times produced very different ratings, which suggests customers react to missing the promised date more than to a long wait. AM has only 73 orders in that year, so this is a directional finding.

November 2017 was the sales peak and also the worst delivery month of that year. Sales reached about 1.0M in November against roughly 0.6M in October, while late deliveries reached 12.40% compared with 4.18% in October. Seasonal promotions such as Black Friday are a plausible explanation, but the dataset does not confirm it.

Sales grew mainly through volume. Between January and August 2018 sales were 7.22M, already above the full 2017 total of 5.96M, and more than double the same months of 2017. The average order value stayed at 137 in both years, so growth came from more orders rather than larger baskets. After a peak in May 2018 (977.5K), monthly sales stayed in a range between 826K and 978K.

The seller base is widening. The number of sellers grew while sales per seller fell from 3.53K to 3.10K. The top 10 sellers contributed about 18% of sales in 2017 and about 13% in 2018, so concentration decreased.

Category leadership changed. health_beauty moved from third place in 2017 to first in 2018, and watches_gifts stayed second. Volume and value tell different stories: furniture_decor ranked second by items sold in 2017 but only seventh by sales, while watches_gifts ranked second by sales with far fewer items sold, which points to a much higher price per item.

Freight is a heavier burden in northern and north-eastern states. MA, TO, AM, AC, PI, and RR consistently show freight at roughly 25% to 28% of sales (RR reached 42.84% in 2017, though its order count is small), compared with a 16% to 17% average overall.

Customer satisfaction is high overall. About 78% of reviews give four or five stars in both years, and 10% to 11% give one star. Cancellations are rare at roughly 0.6%.
10. Business Recommendations
Based on the findings, I would recommend the following actions:
1. Proactively contact customers whose orders are likely to be late. The rating falls by about one star after only one to three days of delay, so early communication and compensation could protect satisfaction.
2. Investigate the February–March 2018 delivery spike. Identify whether it came from specific carriers, states, sellers, or from changes in the promised delivery date.
3. Review how estimated delivery dates are set. Compare the promised delivery window with actual delivery time in 2017 and 2018, because customers react more to missed promises than to long waits.
4. Prepare for peak season in advance. Late deliveries more than doubled in November 2017, so plan carrier capacity and seller stock before peak demand.
5. Prioritize states where delay volume is highest. Large states such as Bahia (BA), Ceará (CE), and Espírito Santo (ES) affect more customers than small states with higher percentages.
6. Review shipping policies for northern and north-eastern states. Regional carriers, minimum order values, or free-shipping thresholds could reduce the freight burden.
7. Support mid-sized sellers while keeping large ones engaged. The seller base is widening and the top sellers' share is falling.
8. Tailor category strategy. For high-volume, lower-price categories such as furniture_decor, bundles or minimum basket offers may raise order value. For health_beauty and watches_gifts, which drive growth, protect availability and delivery quality.
These recommendations are intended as starting points for further analysis rather than definitive business decisions.
11. Limitations
Several limitations should be considered when interpreting the results:
- The dataset is a sample from 2016 to 2018 and is not current. Olist's operations may have changed since.
- 2018 covers only January to August, so totals are not directly comparable with the full year 2017. Comparisons rely on ratios, averages, and matching months.
- 2016 was excluded because it contains only a few months with very low volume.
- The dataset has no cost or profit data, so margins cannot be calculated. Sales exclude freight.
- Sales metrics use delivered orders only.
- Late-delivery and rating analyses are observational. Correlation does not prove that delays alone cause lower ratings, because late orders may also have other problems.
- Customer reviews are voluntary, so the very satisfied and very dissatisfied customers are likely over-represented. Rating metrics are averaged across review records.
- Small states have few orders, so their percentages are unstable. The state table on the Delivery page was reviewed for the states visible on the page only.
- The monthly 2017 sales figures read from the chart are rounded, so the 2017 versus 2018 same-month comparison is approximate.
- The map visual is illustrative only. State-level analysis relies on the bar charts and the state table.
- Explanations such as the Black Friday effect in November are hypotheses and are not confirmed by the data.
12. Tools Used
The project was developed using the following tools:
- Power BI Desktop: Report design, interactivity, and visuals.
- Power Query: Data cleaning, type conversion, merges, and calculated columns.
- DAX: Business measures, KPIs, and the date table.
- Data Modeling: Star schema with one-to-many relationships.
- Custom HTML visuals: Landing page and navigation design.
- What-If Parameter: Scenario analysis on average order value.
13. Repository Contents
The repository contains the following files and folders:
File or Folder	Description
Olist_Dashboard.pbix	Power BI report containing the data model, measures, and dashboard pages
image/	Dashboard screenshots and data model
README.md	Project documentation, methodology, results, and recommendations


To interact with the dashboard, download Olist_Dashboard.pbix and open it in Power BI Desktop on Windows. The raw dataset is not included in this repository because of its size and license. Download it from Kaggle and update the data source paths in Power Query if you want to refresh the report.

GitHub does not display Power BI interactivity, so the screenshots in this README provide a visual preview of the analysis.
Data Source: Olist Brazilian E-Commerce Public Dataset, Kaggle.
About the Author
Ahmed Noby | Aspiring Data Analyst

Interested in turning raw data into meaningful insights through data analysis, visualization, and business reporting.

Skills demonstrated in this project:
- Power BI: Data Modeling, Interactive Dashboards, Navigation, and What-If Analysis
- Power Query: Data Cleaning and Transformation
- DAX
- Data Validation
- Data Visualization
- Business Reporting
Live Power BI Dashboard
Open the interactive Power BI Dashboard
