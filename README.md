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


