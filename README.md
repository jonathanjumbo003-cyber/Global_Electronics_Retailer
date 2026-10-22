# Global_Electronics_Retailer

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


