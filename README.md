# Global_Electronics_Retailer

## Introduction

In this project, I used SQL to analyze Maven Analytics' Global Electronics Retailer dataset. The goal was to explore customer locations, product demand, sales trends, delivery performance, and differences in average order value between online and in-store sales.

## Background

The dataset contains sales transactions, customer details, product information, store records, and exchange rates. I used these tables to investigate four key business questions:

- What types of products does the company sell, and where are its customers located?
- Are there seasonal patterns or trends in order volume and revenue?
- How long does delivery take on average, and has it changed over time?
- Is there a difference in average order value (AOV) between online and in-store sales?

## Tools I Used

- **SQL:** Data exploration, table joins, aggregations, Common Table Expressions (CTEs), date functions, and business analysis.
- **Google BigQuery:** Querying and analyzing the dataset.
- **GitHub:** Documenting the project, queries, and findings in Markdown.

## The Analysis

### 1. Product Types and Customer Locations

```sql
-- This is gonna be interesting

WITH customer_total AS (
  SELECT
    c.country AS country,
    COUNT(DISTINCT s.customer_key) AS number_of_customers,
    SUM(s.quantity) AS total_quantity
  FROM global_electronics_retailer.sales AS s
  INNER JOIN global_electronics_retailer.customers AS c
    ON s.customer_key = c.customer_key
  GROUP BY
    c.country
),
customer_product_table AS (
  SELECT
    c.country AS country,
    p.category AS category,
    p.subcategory AS subcategory,
    SUM(s.quantity) AS quantity
  FROM global_electronics_retailer.sales AS s
  INNER JOIN global_electronics_retailer.customers AS c
    On s.customer_key = c.customer_key
  INNER JOIN global_electronics_retailer.products AS p
    ON s.product_key = p.product_key
  GROUP BY
    c.country,
    p.category,
    p.subcategory
)

SELECT
  ct.country,
  ct.number_of_customers,
  cp.category,
  cp.subcategory,
  cp.quantity,
  ct.total_quantity,
  ROUND(SAFE_DIVIDE(cp.quantity, ct.total_quantity) * 100, 2) AS pct_of_country_unit
FROM customer_total AS ct
INNER JOIN customer_product_table AS cp
  On ct.country = cp.country
ORDER BY
  ct.total_quantity DESC,
  ct.country,
  pct_of_country_unit DESC
```

To understand the company's product offerings and customer distribution, I joined the `sales`, `customers`, and `products` tables using their corresponding customer and product keys.

I examined the number of purchasing customers and units sold across countries, then compared product categories and subcategories to identify the types of products contributing most to sales volume.

#### Customer Distribution by Country

| Country | Purchasing Customers | Units Sold | Units per Purchasing Customer |
|---|---:|---:|---:|
| United States | 5,706 | 106,407 | 18.65 |
| United Kingdom | 1,570 | 25,298 | 16.11 |
| Germany | 1,150 | 18,943 | 16.47 |
| Canada | 1,179 | 16,793 | 14.24 |
| Australia | 780 | 9,209 | 11.81 |
| Italy | 530 | 8,470 | 15.98 |
| Netherlands | 534 | 7,252 | 13.58 |
| France | 438 | 5,385 | 12.29 |
| **Total** | **11,887** | **197,757** | — |

**Key findings:**

- The United States had the largest customer base among purchasing customers, with 5,706 customers and 106,407 units sold. It accounted for approximately 53.8% of the total units recorded across the eight countries.
- The United Kingdom ranked second in sales volume, with 25,298 units sold, followed by Germany with 18,943 units.
- Germany sold more units than Canada despite having fewer purchasing customers. This shows why customer count alone does not fully explain differences in sales volume.
- The United States recorded the highest average unit volume per purchasing customer, at 18.65 units.

#### Sales Volume by Product Category

I aggregated the quantities sold by product category to understand which broad product types contributed most to the company's sales volume.

| Product Category | Units Sold | Share of Total Units |
|---|---:|---:|
| Computers | 44,151 | 22.32% |
| Cell phones | 31,477 | 15.92% |
| Music, Movies and Audio Books | 28,802 | 14.56% |
| Audio | 23,490 | 11.88% |
| Games and Toys | 22,591 | 11.42% |
| Home Appliances | 18,401 | 9.30% |
| Cameras and camcorders | 17,609 | 8.90% |
| TV and Video | 11,236 | 5.68% |
| **Total** | **197,757** | **100%** |

**Key findings:**

- Computers was the leading product category, accounting for 44,151 units, or approximately 22.32% of total units sold.
- Cell phones ranked second, followed by Music, Movies and Audio Books.
- TV and Video recorded the lowest unit volume among the eight product categories, with 11,236 units.
- Computers was also the leading category by unit volume in each of the eight countries examined.

#### Top-Selling Product Subcategories

To gain a more detailed understanding of product demand, I compared sales quantities at the subcategory level.

| Product Subcategory | Units Sold | Share of Total Units |
|---|---:|---:|
| Movie DVD | 28,802 | 14.56% |
| Desktops | 20,626 | 10.43% |
| Bluetooth Headphones | 13,653 | 6.90% |

Movie DVD recorded the highest unit volume among the product subcategories, followed by Desktops and Bluetooth Headphones. Movie DVD was also the leading subcategory in each of the eight countries examined.

#### Conclusion

The analysis showed that the company had purchasing customers across eight countries, with the United States contributing the largest share of customer activity and unit sales. Computers was the strongest product category by unit volume, while Movie DVD was the leading individual subcategory.

The results also showed that countries with more purchasing customers did not always have proportionally higher sales volumes. Comparing customer counts with quantities sold provided a more complete picture of geographical demand.

These findings describe **sales volume rather than revenue or profitability**. Further analysis of product prices, costs, and currencies would be required to determine which products and markets contribute most to the company's financial performance.

### 2. Seasonal Patterns and Trends in Order Volume and Revenue

To investigate whether the business experienced seasonal patterns or trends, I analyzed monthly order volume and estimated revenue from 2016 to 2020. I grouped sales by year and month, compared annual totals, and calculated each month's percentage contribution to its annual total.

```sql
-- Comparing revenue acroos the years

WITH price_table AS (
  SELECT
    s.order_number AS order_number,
    s.order_date AS order_date,
    s.quantity AS quantity,
    p.unit_price_usd AS price,
    s.quantity * p.unit_price_usd AS order_price
  FROM global_electronics_retailer.sales AS s
  INNER JOIN global_electronics_retailer.products AS p
    ON s.product_key = p.product_key
  WHERE
    S.order_date IS NOT NULL
),
year_month_revenue AS (
  SELECT
    EXTRACT(YEAR FROM pt.order_date) AS years,
    EXTRACT(MONTH FROM pt.order_date) AS months,
    SUM(PT.order_price) AS revenue
  FROM price_table AS pt
  GROUP BY
    years,
    months
),
years_revenue AS (
  SELECT
    EXTRACT(YEAR FROM pt.order_date) AS years,
    SUM(order_price) AS total_revenue
  FROM price_table AS pt
  GROUP BY
    years
)

SELECT
  ym.years,
  ym.months,
  ROUND(ym.revenue, 2),
  ROUND(yr.total_revenue, 2),
  ROUND(SAFE_DIVIDE(ym.revenue, yr.total_revenue) * 100, 2) AS pct_contribution
FROM year_month_revenue AS ym
INNER JOIN years_revenue AS yr
  ON ym.years = yr.years
WHERE
  ym.years BETWEEN 2016 AND 2020
ORDER BY
  ym.years,
  ym.months
```

For the revenue analysis, I estimated sales value by multiplying the quantity sold by the listed USD unit price of each product.

#### Annual Order Volume and Revenue Trends

| Year | Total Orders | Order Volume Change | Estimated Revenue (USD) | Revenue Change |
|---|---:|---:|---:|---:|
| 2016 | 2,865 | — | $6,946,793.56 | — |
| 2017 | 3,280 | +14.49% | $7,421,422.27 | +6.83% |
| 2018 | 5,965 | +81.86% | $12,788,960.66 | +72.32% |
| 2019 | 9,083 | +52.27% | $18,264,382.48 | +42.81% |
| 2020 | 4,635 | -48.97% | $9,294,632.14 | -49.11% |

**Key findings:**

- Order volume and estimated revenue increased overall between 2016 and 2019.
- The strongest annual growth occurred in 2018, when order volume increased by 81.86% and estimated revenue grew by 72.32%.
- In 2019, both measures reached their highest annual totals within the period analyzed, with 9,083 orders and approximately $18.26 million in estimated revenue.
- In 2020, order volume fell by 48.97%, while estimated revenue declined by 49.11%. The similar declines suggest that the reduction in order volume was closely associated with the decline in estimated revenue.

#### Monthly Contribution to Annual Order Volume and Revenue

To compare monthly performance across years with different annual totals, I calculated each month's percentage contribution to the corresponding year's total.

| Year | April Order Share | December Order Share | April Revenue Share | December Revenue Share |
|---|---:|---:|---:|---:|
| 2016 | 1.43% | 14.97% | 1.59% | 14.37% |
| 2017 | 0.79% | 16.74% | 0.79% | 17.34% |
| 2018 | 0.72% | 18.26% | 0.55% | 16.28% |
| 2019 | 0.83% | 13.76% | 0.82% | 13.56% |
| 2020 | 1.73% | 6.86% | 2.34% | 7.01% |

**Key findings:**

- **December was a recurring peak between 2016 and 2019.** It contributed between 13.76% and 18.26% of annual order volume and between 13.56% and 17.34% of annual estimated revenue during these years.
- **April was consistently a low-volume month.** Its contribution remained below 1.5% of annual orders and below 1.6% of annual estimated revenue from 2016 through 2019.
- The similar patterns in order volume and estimated revenue suggest that the December peaks and April troughs were not limited to the number of orders; they were also reflected in the estimated value of sales.
- The pattern changed considerably in 2020. December contributed only 6.86% of annual orders and 7.01% of estimated revenue, while January and February together accounted for 46.41% of annual orders and 46.22% of estimated revenue.

#### Conclusion

The analysis identified recurring monthly patterns in both order volume and estimated revenue from 2016 to 2019. December consistently represented a strong month, while April contributed a relatively small share of annual activity.

Order volume and estimated revenue also followed similar annual trends, growing through 2019 before declining sharply in 2020. The 2020 results differed from the earlier monthly pattern, indicating that the observed seasonal behavior was not consistent throughout the entire period.

These findings suggest possible seasonality in the business, particularly during 2016–2019. However, the analysis identifies patterns rather than their causes; further investigation would be needed to establish why these changes occurred.

**Note:** Revenue in this analysis is estimated using quantity multiplied by the listed USD unit price. It should not be interpreted as confirmed actual revenue because transaction-specific prices, discounts, and other adjustments were not incorporated.

### 3. Average Delivery Time and Changes Over Time

```sql
-- Change in average delivery time over time

WITH order_delivery AS (
  SELECT DISTINCT
    order_number,
    order_date,
    delivery_date,
    DATE_DIFF(delivery_date, order_date, DAY) AS delivery_time
  FROM global_electronics_retailer.sales
  WHERE
    order_number IS NOT NULL AND
    order_date IS NOT NULL AND
    delivery_date IS NOT NULL
),
year_avg As (
  SELECT
    EXTRACT(YEAR FROM order_date) AS years,
    ROUND(AVG(delivery_time), 2) AS yearly_avg
  FROM order_delivery
  GROUP BY
    years
),
year_month_avg AS (
  SELECT
    EXTRACT(YEAR FROM order_date) AS years,
    EXTRACT(MONTH FROM order_date) AS months,
    ROUND(AVG(delivery_time), 2) AS monthly_avg
  FROM order_delivery
  GROUP BY
    years,
    months
)

SELECT
  ym.years,
  ym.months,
  ym.monthly_avg,
  ya.yearly_avg,
  CASE
    WHEN ym.monthly_avg > ya.yearly_avg THEN 'Slower Than Year Avg'
    WHEN ym.monthly_avg < ya.yearly_avg THEN 'Faster Than Year Avg'
    ELSE 'Same As Year Avg'
  END AS performance
FROM year_avg AS ya
INNER JOIN year_month_avg AS ym
  ON ya.years = ym.years
WHERE
  ym.years BETWEEN 2016 AND 2020
ORDER BY
  ym.years,
  ym.months
```

To understand the company's delivery performance, I calculated the average number of days between the order date and delivery date. I examined yearly averages to identify long-term changes and compared monthly averages against their respective yearly averages to identify periods with faster or slower deliveries.

To avoid counting repeated line items as separate orders, I used distinct combinations of order number, order date, and delivery date before calculating the averages.

#### Average Delivery Time by Year

| Year | Average Delivery Time (Days) | Change from Previous Year (Days) |
|---|---:|---:|
| 2016 | 7.17 | — |
| 2017 | 5.20 | -1.97 |
| 2018 | 4.36 | -0.84 |
| 2019 | 4.10 | -0.26 |
| 2020 | 4.03 | -0.07 |

**Key findings:**

- Average delivery time decreased every year between 2016 and 2020.
- The largest year-to-year reduction occurred between 2016 and 2017, when average delivery time fell by 1.97 days.
- Average delivery time declined from 7.17 days in 2016 to 4.03 days in 2020, representing an overall reduction of approximately 43.8%.
- The annual improvement became smaller over time, with the average falling by only 0.07 days between 2019 and 2020.

These results indicate that deliveries generally became faster over the period analyzed, although the data alone does not establish the reasons for the improvement.

#### Monthly Delivery Time Comparison

I also compared monthly average delivery times with their respective yearly averages to identify months when deliveries were relatively slower or faster.

| Year | Slowest Month | Average Days | Fastest Month | Average Days |
|---|---|---:|---|---:|
| 2016 | January | 8.44 | December | 6.32 |
| 2017 | January | 5.90 | December | 4.76 |
| 2018 | April | 5.60 | November | 4.11 |
| 2019 | September | 4.30 | April | 3.76 |
| 2020 | September | 4.19 | August | 3.51 |

**Key findings:**

- January recorded the longest average delivery times in both 2016 and 2017, while December recorded the shortest.
- From 2018 to 2020, the months with the longest and shortest average delivery times varied.
- Despite these monthly fluctuations, the yearly average continued to decline throughout the period.
- A month being faster than its yearly average does not necessarily mean it was faster than the same month in another year. Monthly performance must be interpreted alongside the overall trend.

#### Conclusion

The analysis showed a consistent improvement in average delivery time from 2016 to 2020. The average fell from 7.17 days to 4.03 days, a reduction of approximately 43.8%.

Although monthly delivery times fluctuated, the overall trend suggests that deliveries generally became faster over the period. The largest improvement occurred between 2016 and 2017, while delivery times appeared to stabilize around four days in 2019 and 2020.

Further investigation would be needed to determine what caused this improvement, as the analysis identifies changes in delivery time rather than the operational factors behind them.

**Note:** The analysis excludes records with missing order or delivery dates. It assumes that each order has consistent dates across its line items, and the averages represent orders with valid date information.

### 4. Average Order Value (AOV): Online vs. In-Store Sales

```sql
-- just a combined sql for online stores and in-stores

WITH order_price AS (
  SELECT
    EXTRACT(YEAR FROM s.order_date) AS order_year,
    s.order_number AS order_number,
    CASE
      WHEN s.store_key = 0 THEN 'Online'
      ELSE 'In-store'
    END AS sales_channel,
    s.quantity * p.unit_price_usd AS line_value
  FROM global_electronics_retailer.sales AS s
  INNER JOIN global_electronics_retailer.products AS p
    ON s.product_key = p.product_key
  WHERE
    s.order_date IS NOT NULL AND
    s.order_number IS NOT NULL AND
    s.store_key IS NOT NULL
),
transition AS (
  SELECT
    order_year,
    order_number,
    sales_channel,
    SUM(line_value) AS order_price
  FROM order_price
  GROUP BY
    order_year,
    order_number,
    sales_channel
),
year_aov AS (
  SELECT
    order_year,
    sales_channel,
    COUNT(*) AS total_orders,
    AVG(order_price) AS aov
  FROM transition
  GROUP BY
    order_year,
    sales_channel
)

SELECT
  order_year,
  MAX(IF(sales_channel = 'Online', total_orders, NULL)) AS online_order,
  ROUND(MAX(IF(sales_channel = 'Online', aov, NULL)), 2) AS online_aov,
  MAX(IF(sales_channel = 'In-store', total_orders, NULL)) AS in_store_orders,
  ROUND(MAX(IF(sales_channel = 'In-store', aov, NULL)), 2) AS in_store_aov,
  ROUND(MAX(IF(sales_channel = 'In-store', aov, NULL)) - MAX(IF(sales_channel = 'Online', aov, NULL)), 2) AS in_store_aov_minus_online_store_aov
FROM year_aov
WHERE
  order_year BETWEEN 2016 AND 2020
GROUP BY
  order_year
ORDER BY
  order_year
```

To investigate whether average order value differed between online and in-store sales, I calculated the estimated value of each order by multiplying the quantity sold by the listed USD unit price of each product. I then aggregated sales-line values into individual order values and calculated the average order value for each sales channel by year.

#### Annual AOV Comparison

| Year | Online AOV (USD) | In-Store AOV (USD) | Higher-AOV Channel | AOV Difference (USD) |
|---|---:|---:|---|---:|
| 2016 | $2,456.55 | $2,418.37 | Online | $38.18 |
| 2017 | $2,156.05 | $2,288.67 | In-Store | $132.61 |
| 2018 | $2,074.13 | $2,162.43 | In-Store | $88.30 |
| 2019 | $1,948.40 | $2,028.53 | In-Store | $80.12 |
| 2020 | $1,932.73 | $2,027.13 | In-Store | $94.40 |
| 2021* | $2,070.85 | $2,093.09 | In-Store | $22.23 |

*2021 contains only January and February in the available order-volume results and is therefore treated as a partial year.*

The AOV difference is calculated as in-store AOV minus online AOV. A positive difference means in-store AOV was higher, while a negative difference means online AOV was higher.

**Key findings:**

- **In-store AOV was higher in four consecutive full years, from 2017 to 2020.** Online AOV was slightly higher in 2016, by $38.18.
- The largest difference occurred in 2017, when in-store AOV exceeded online AOV by approximately $132.61.
- In 2020, in-store AOV was $2,027.13 compared with online AOV of $1,932.73, a difference of $94.40.
- Both channels experienced a decline in AOV between 2016 and 2020. Online AOV decreased by approximately 21.32%, while in-store AOV declined by approximately 16.18%.

#### Conclusion

The analysis found that in-store orders generally had a higher estimated average order value than online orders during 2017–2020. Online AOV was slightly higher in 2016, but the relationship reversed in the following year.

The results also showed that estimated AOV declined across both channels from 2016 to 2020, with a larger percentage reduction in the online channel. This suggests that the listed-price value per order decreased over the period, although the analysis does not establish the reasons behind the change.

**Limitation:** AOV was estimated using quantity multiplied by listed USD product prices. Transaction-specific prices, discounts, and other adjustments were not incorporated, so these figures should not be interpreted as confirmed actual customer spending. The 2021 results cover only part of the year and should not be compared directly with complete annual totals.

## Conclusion

This project allowed me to analyze the Global Electronics Retailer dataset using SQL and Google BigQuery to investigate customer distribution, product demand, sales trends, delivery performance, and average order value across online and in-store sales.

The analysis revealed several key findings:

- **Customer and product demand:** The United States had the highest number of purchasing customers and the largest sales volume among the eight countries analyzed. Computers was the leading product category, while Movie DVD was the top-selling subcategory by unit volume.
- **Seasonal patterns:** Order volume and estimated revenue generally increased from 2016 to 2019, with December consistently representing a strong month and April a low-activity month. Both measures declined sharply in 2020, when the earlier monthly pattern changed.
- **Delivery performance:** Average delivery time declined from 7.17 days in 2016 to 4.03 days in 2020, a reduction of approximately 43.8%. Although monthly delivery times fluctuated, the overall trend showed that deliveries became faster over the period.
- **Online vs. in-store AOV:** In-store orders had a higher estimated average order value from 2017 through 2020, while online AOV was slightly higher in 2016. Both channels experienced declining estimated AOV between 2016 and 2020.

Overall, this project strengthened my ability to use SQL to join multiple tables, aggregate data, calculate business metrics, compare trends over time, and translate query results into meaningful findings.

The analysis also highlighted the importance of interpreting metrics carefully. Revenue and AOV were estimated using quantities and listed USD product prices, rather than verified transaction amounts. The findings identify patterns and differences in the available data but do not establish their underlying causes.

This project improved my understanding of how data analysis can help businesses evaluate customer demand, monitor performance, and identify areas for further investigation.
