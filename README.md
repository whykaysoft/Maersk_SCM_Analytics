# Maersk Supply Chain Optimization

> Identifying $86K in storage cost drivers and 
> 56% late shipment risk using PostgreSQL + Power BI
---

## Overview

This project analyzes Maersk's supply chain to identify inefficiencies and optimize operations. Using PostgreSQL for data manipulation and Power BI for dashboards, the study focuses on inventory management, shipping operations, and customer demand. Key outputs include enhanced inventory segmentation, detailed analysis of shipment delays, and strategic recommendations for a more efficient and responsive supply chain.

---

## Objectives
The project aims to: 
- Identify key inefficiencies in shipment and inventory management
- Build interactive dashboards for three stakeholder groups
- Propose strategic recommendations to improve supply chain responsiveness

---

## Methodology

| Tool | Purpose |
|---|---|
| SQL | Data cleaning, EDA, feature engineering |
| Power BI | Interactive dashboard development |

---

### 1. Data Preprocessing
- Removed irrelevant columns and corrected data types
- Resolved inconsistencies across order, shipment, and inventory tables
- Engineered new features to measure shipment timeliness and 
  calculate business KPIs

### 2. Exploratory Data Analysis (EDA)
- Business performance metrics by region and product category
- Customer demographics and purchasing behavior patterns
- Product profitability and inventory turnover analysis

### 3. Inventory Segmentation: ABC XYZ Analysis
Categorized all SKUs based on revenue contribution (ABC) and 
demand stability (XYZ) to prioritize inventory management decisions.

| Segment | Strategy |
|---|---|
| XA, YA (High demand, stable) | Protect stock levels, prioritize reorder |
| XB, XC (Emerging) | Market research, cautious expansion |
| Low-performing segments | Reduce or discontinue to cut storage cost |

## 🗄️ Database Setup in SQL

The database was created from scratch to house Swift Bike Share's historical ride and revenue data.

### Tables Created

```sql
-- raw order table
CREATE TABLE raw_orders (
    order_id                    INT,
    order_item_id               INT,
    order_yearmonth             INT,
    order_year                  INT,
    order_month                 INT,
    order_day                   INT,
    order_time                  VARCHAR(10),
    order_quantity              INT,
    product_department          VARCHAR(100),
    product_category            VARCHAR(100),
    product_name                VARCHAR(200),
    customer_id                 INT,
    customer_market             VARCHAR(100),
    customer_region             VARCHAR(100),
    customer_country            VARCHAR(100),
    warehouse_country           VARCHAR(100),
    shipment_year               INT,
    shipment_month              INT,
    shipment_day                INT,
    shipment_mode               VARCHAR(50),
    shipment_days_scheduled     INT,
    gross_sales                 FLOAT,
    discount_pct                VARCHAR(20),  -- VARCHAR first; contains '-' values
    profit                      FLOAT
);
```

```sql
-- raw_inventory table
CREATE TABLE raw_inventory (
    product_name                VARCHAR(200),
    year_month                  INT,
    warehouse_inventory         INT,
    inventory_cost_per_unit     FLOAT
);
```

```sql
-- raw_fulfillment table
CREATE TABLE raw_fulfillment (
    product_name                        VARCHAR(200),
    warehouse_order_fulfillment_days    FLOAT
);
```

### Key SQL Queries

**1. Clean Orders Table**
```sql
CREATE TABLE cleaned_orders AS

WITH

-- Step 1: Strip whitespace issues, fix discount column, cast types
base AS (
    SELECT
        order_id,
        order_yearmonth,
        order_year,
        order_month,
        order_day,
        order_quantity,
        product_department,
        product_category,
        product_name,
        customer_id,
        customer_market,
        customer_region,

        -- Fix special character country names (replicating Python .replace())
        CASE customer_country
            WHEN 'Dominican Republic'    THEN 'Dominican Republic'
            WHEN 'Cote d Ivoire'         THEN 'Cote d Ivoire'
            WHEN 'Peru'                  THEN 'Peru'
            WHEN 'Algeria'               THEN 'Algeria'
            WHEN 'Israel'                THEN 'Israel'
            WHEN 'Benin'                 THEN 'Benin'
            ELSE customer_country
        END AS customer_country,

        warehouse_country,
        shipment_year,
        shipment_month,
        shipment_day,
        shipment_mode,
        shipment_days_scheduled,
        gross_sales,

        -- Fix discount: replace '-' with 0, cast to float
        CASE 
            WHEN TRIM(discount_pct) = '-' THEN 0.0
            ELSE CAST(TRIM(discount_pct) AS FLOAT)
        END AS discount_pct,

        profit
    FROM raw_orders
    -- Drop Order Item ID and Order Time by simply not selecting them
),

-- Step 2: Build date columns and shipping time
with_dates AS (
    SELECT
        *,
        -- Build Order Date and Shipment Date
        MAKE_DATE(order_year, order_month, order_day)           AS order_date,
        MAKE_DATE(shipment_year, shipment_month, shipment_day)  AS shipment_date,

        -- Shipping Time in days
        MAKE_DATE(shipment_year, shipment_month, shipment_day)
        - MAKE_DATE(order_year, order_month, order_day)         AS shipping_time,

        -- YearMonth string for time series grouping
        LPAD(order_month::TEXT, 2, '0') AS order_month_str,
        LPAD(order_year::TEXT, 4, '0')  AS order_year_str

    FROM base
),

-- Step 3: Remove invalid shipping times (< 0 or > 28 days)
filtered AS (
    SELECT *
    FROM with_dates
    WHERE shipping_time >= 0 AND shipping_time <= 28
),

-- Step 4: Classify delay shipment
with_delay AS (
    SELECT
        *,
        CASE 
            WHEN shipping_time > shipment_days_scheduled THEN 'Late'
            ELSE 'On time'
        END AS delay_shipment
    FROM filtered
),

-- Step 5: Create business performance features
with_metrics AS (
    SELECT
        *,
        ROUND((gross_sales - gross_sales * discount_pct)::NUMERIC, 2)  AS net_sales,
        ROUND((gross_sales / NULLIF(order_quantity, 0))::NUMERIC, 2)   AS unit_price
    FROM with_delay
)

SELECT * FROM with_metrics;
```

**2. Clean Inventory Table**
```sql
CREATE TABLE cleaned_inventory AS

WITH base AS (
    SELECT
        TRIM(product_name)          AS product_name,
        -- Convert YYYYMM integer to a proper date
        TO_DATE(year_month::TEXT, 'YYYYMM')     AS year_month,
        warehouse_inventory,
        inventory_cost_per_unit,
        -- Compute storage cost per row
        ROUND((inventory_cost_per_unit * warehouse_inventory)::NUMERIC, 5) AS storage_cost
    FROM raw_inventory
)

SELECT * FROM base;
```

**3. Product Dimension Table**
```sql
CREATE TABLE dim_product AS

SELECT DISTINCT
    product_name,
    product_category,
    product_department
FROM cleaned_orders
ORDER BY product_name;

--- Enrich Inventory with Product dimension
CREATE TABLE inventory_enriched AS

SELECT
    i.product_name,
    i.year_month,
    i.warehouse_inventory,
    i.inventory_cost_per_unit,
    i.storage_cost,
    COALESCE(p.product_category,   'None') AS product_category,
    COALESCE(p.product_department, 'None') AS product_department
FROM cleaned_inventory i
LEFT JOIN dim_product p ON i.product_name = p.product_name;
```

**4. FEATURE ENGINEERING & EDA IN SQL**
```sql
-- Business Performance Metrics:
-- Total Net Sales, Total Profit, Profit Margin
SELECT
    ROUND(SUM(net_sales)::NUMERIC, 2)                                    AS total_net_sales,
    ROUND(SUM(profit)::NUMERIC, 2)                                       AS total_profit,
    ROUND((SUM(profit) / NULLIF(SUM(net_sales), 0) * 100)::NUMERIC, 2)  AS profit_margin_pct
FROM cleaned_orders;


-- Monthly average Net Sales and Profit
SELECT
    ROUND(SUM(net_sales) / 37.0, 2)  AS avg_monthly_net_sales,
    ROUND(SUM(profit)   / 37.0, 2)   AS avg_monthly_profit
FROM cleaned_orders;


-- Orders, Net Sales, Profit over time (monthly trend)
SELECT
    DATE_TRUNC('month', order_date)         AS order_yearmonth,
    COUNT(DISTINCT order_id)                AS number_of_orders,
    ROUND(SUM(net_sales)::NUMERIC, 2)       AS total_net_sales,
    ROUND(SUM(profit)::NUMERIC, 2)          AS total_profit,
    ROUND(AVG(unit_price)::NUMERIC, 2)      AS avg_unit_price,
    ROUND(AVG(order_quantity)::NUMERIC, 2)  AS avg_order_quantity
FROM cleaned_orders
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY order_yearmonth;
```

**5. Customer Analysis**
```sql
-- Top 10 countries by customer count
SELECT
    customer_country,
    COUNT(DISTINCT customer_id) AS number_of_customers
FROM cleaned_orders
GROUP BY customer_country
ORDER BY number_of_customers DESC
LIMIT 10;


-- Customer distribution by market
SELECT
    customer_market,
    COUNT(DISTINCT customer_id) AS number_of_customers
FROM cleaned_orders
GROUP BY customer_market
ORDER BY number_of_customers DESC;


-- Customers over time by market (buying trend)
SELECT
    DATE_TRUNC('month', order_date) AS order_yearmonth,
    customer_market,
    COUNT(DISTINCT customer_id)     AS number_of_customers,
    COUNT(DISTINCT order_id)        AS number_of_orders
FROM cleaned_orders
GROUP BY DATE_TRUNC('month', order_date), customer_market
ORDER BY order_yearmonth, customer_market;


-- Orders by month of year (seasonality check)
SELECT
    order_month,
    COUNT(DISTINCT order_id) AS number_of_orders
FROM cleaned_orders
GROUP BY order_month
ORDER BY order_month;
```

**6. Product Analysis**
```sql
-- Top product categories by number of orders
SELECT
    product_category,
    COUNT(DISTINCT order_id) AS number_of_orders
FROM cleaned_orders
GROUP BY product_category
ORDER BY number_of_orders DESC;


-- Top 10 product names by orders
SELECT
    product_name,
    COUNT(DISTINCT order_id) AS number_of_orders
FROM cleaned_orders
GROUP BY product_name
ORDER BY number_of_orders DESC
LIMIT 10;


-- Top 10 product names by profit
SELECT
    product_name,
    ROUND(SUM(profit)::NUMERIC, 2) AS total_profit
FROM cleaned_orders
GROUP BY product_name
ORDER BY total_profit DESC
LIMIT 10;


-- Net Sales by Product Department over time
SELECT
    DATE_TRUNC('month', order_date) AS order_yearmonth,
    product_department,
    ROUND(SUM(net_sales)::NUMERIC, 2) AS total_net_sales
FROM cleaned_orders
GROUP BY DATE_TRUNC('month', order_date), product_department
ORDER BY order_yearmonth, product_department;
```

**7. Shipment Analysis**
```sql
-- Late shipment rate by product department
SELECT
    product_department,
    COUNT(*) FILTER (WHERE delay_shipment = 'Late')  AS late_orders,
    COUNT(*)                                          AS total_orders,
    ROUND(
        COUNT(*) FILTER (WHERE delay_shipment = 'Late') * 100.0 / COUNT(*),
    2) AS late_shipment_rate_pct
FROM cleaned_orders
GROUP BY product_department
ORDER BY late_shipment_rate_pct DESC;


-- Late shipment rate by customer market
SELECT
    customer_market,
    COUNT(*) FILTER (WHERE delay_shipment = 'Late')  AS late_orders,
    COUNT(*)                                          AS total_orders,
    ROUND(
        COUNT(*) FILTER (WHERE delay_shipment = 'Late') * 100.0 / COUNT(*),
    2) AS late_shipment_rate_pct
FROM cleaned_orders
GROUP BY customer_market
ORDER BY late_shipment_rate_pct DESC;


-- Late shipment rate over time
SELECT
    DATE_TRUNC('month', shipment_date)                                       AS shipment_yearmonth,
    COUNT(*) FILTER (WHERE delay_shipment = 'Late')                          AS late_orders,
    COUNT(*)                                                                 AS total_orders,
    ROUND(COUNT(*) FILTER (WHERE delay_shipment = 'Late') * 100.0 / COUNT(*), 2) AS lsr_pct
FROM cleaned_orders
GROUP BY DATE_TRUNC('month', shipment_date)
ORDER BY shipment_yearmonth;


-- Shipment mode distribution
SELECT
    shipment_mode,
    COUNT(DISTINCT order_id) AS number_of_orders,
    ROUND(COUNT(DISTINCT order_id) * 100.0 / SUM(COUNT(DISTINCT order_id)) OVER (), 2) AS pct_share
FROM cleaned_orders
GROUP BY shipment_mode
ORDER BY number_of_orders DESC;
```

**8. ABC-XYZ INVENTORY SEGMENTATION IN SQL**
```sql
-- ABC Segmentation (Revenue Contribution)
CREATE TABLE abc_segmentation AS

WITH product_sales AS (
    SELECT
        product_department,
        product_category,
        product_name,
        ROUND(SUM(net_sales)::NUMERIC, 2) AS total_net_sales
    FROM cleaned_orders
    GROUP BY product_department, product_category, product_name
),

total AS (
    SELECT SUM(total_net_sales) AS grand_total FROM product_sales
),

with_pct AS (
    SELECT
        ps.*,
        ROUND((ps.total_net_sales / t.grand_total)::NUMERIC, 6) AS pct_share,
        ROUND(
            SUM(ps.total_net_sales / t.grand_total) 
            OVER (ORDER BY ps.total_net_sales DESC 
                  ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
        ::NUMERIC, 6) AS cumulative_pct
    FROM product_sales ps, total t
)

SELECT
    *,
    CASE
        WHEN cumulative_pct <= 0.80 THEN 'A [High value]'
        WHEN cumulative_pct <= 0.95 THEN 'B [Medium value]'
        ELSE                              'C [Low value]'
    END AS abc_category
FROM with_pct
ORDER BY total_net_sales DESC;
```

**9. XYZ Segmentation (Demand Volatility)**
```sql
CREATE TABLE xyz_segmentation AS

WITH monthly_sales AS (
    SELECT
        product_name,
        DATE_TRUNC('month', order_date) AS order_month,
        SUM(net_sales) AS monthly_net_sales
    FROM cleaned_orders
    GROUP BY product_name, DATE_TRUNC('month', order_date)
),

cv_calc AS (
    SELECT
        product_name,
        ROUND(AVG(monthly_net_sales)::NUMERIC, 4)    AS avg_sales,
        ROUND(STDDEV(monthly_net_sales)::NUMERIC, 4) AS stddev_sales,
        ROUND(
            CASE
                WHEN AVG(monthly_net_sales) = 0 THEN 0
                ELSE STDDEV(monthly_net_sales) / AVG(monthly_net_sales)
            END
        ::NUMERIC, 4) AS cv
    FROM monthly_sales
    GROUP BY product_name
)

SELECT
    *,
    CASE
        WHEN cv < 0.25              THEN 'X [Regular demand]'
        WHEN cv BETWEEN 0.25 AND 0.5 THEN 'Y [Variable demand]'
        ELSE                              'Z [Irregular demand]'
    END AS xyz_category
FROM cv_calc;
```

**10. Combine into Final ABC-XYZ Segmentation Table**
```sql
CREATE TABLE inventory_segmentation AS

SELECT
    a.product_department,
    a.product_category,
    a.product_name,
    a.total_net_sales,
    a.pct_share,
    a.cumulative_pct,
    a.abc_category,
    x.cv,
    x.xyz_category,
    -- Combined segment label
    LEFT(x.xyz_category, 1) || LEFT(a.abc_category, 1) AS abcxyz_segment
FROM abc_segmentation a
JOIN xyz_segmentation x ON a.product_name = x.product_name;
```

**11. Apply Segmentation to Orders and Inventory**
```sql
-- Orders with segmentation
CREATE TABLE orders_segmented AS
SELECT o.*, s.abcxyz_segment
FROM cleaned_orders o
LEFT JOIN inventory_segmentation s ON o.product_name = s.product_name;


-- Inventory with segmentation
CREATE TABLE inventory_segmented AS
SELECT i.*, s.abcxyz_segment
FROM inventory_enriched i
LEFT JOIN inventory_segmentation s ON i.product_name = s.product_name;
```

**12. Supply vs Demand Analysis (Hypothesis Testing)**
```sql
-- Supply (warehouse) vs Demand (orders) per segment per month
SELECT
    o.order_yearmonth_trunc,
    o.abcxyz_segment,
    o.total_demand,
    i.total_supply
FROM (
    SELECT
        DATE_TRUNC('month', order_date)  AS order_yearmonth_trunc,
        abcxyz_segment,
        SUM(order_quantity)              AS total_demand
    FROM orders_segmented
    GROUP BY DATE_TRUNC('month', order_date), abcxyz_segment
) o
LEFT JOIN (
    SELECT
        DATE_TRUNC('month', year_month)  AS inv_month,
        abcxyz_segment,
        SUM(warehouse_inventory)         AS total_supply
    FROM inventory_segmented
    GROUP BY DATE_TRUNC('month', year_month), abcxyz_segment
) i
ON o.order_yearmonth_trunc = i.inv_month
AND o.abcxyz_segment = i.abcxyz_segment
ORDER BY o.order_yearmonth_trunc, o.abcxyz_segment;


-- Late shipment rate by segment
SELECT
    abcxyz_segment,
    COUNT(*) FILTER (WHERE delay_shipment = 'Late')   AS late_orders,
    COUNT(*)                                           AS total_orders,
    ROUND(
        COUNT(*) FILTER (WHERE delay_shipment = 'Late') * 100.0 / COUNT(*),
    2) AS lsr_pct
FROM orders_segmented
GROUP BY abcxyz_segment
ORDER BY abcxyz_segment;


-- Average fulfillment by segment (join with fulfillment table)
SELECT
    s.abcxyz_segment,
    ROUND(AVG(f.warehouse_order_fulfillment_days)::NUMERIC, 2) AS avg_fulfillment_days
FROM raw_fulfillment f
JOIN inventory_segmentation s ON f.product_name = s.product_name
GROUP BY s.abcxyz_segment
ORDER BY avg_fulfillment_days DESC;
```



---

## Key Insights

- **Sharp decline in revenue and profit**: The most likely root cause of business downturn comes from a supply chain disruption on the supplier network such as delays in sourcing, transportation, distribution or problem with suppliers could have affected the company's ability to replenish the warehouse inventory promptly, especially for the products of XA and YA segments.
- **Overstock**: The company encountered overstock situation of 31.5% more inventory than orders, causing excess inventory and wasting storage costs.
- **High late shipment rate**: The company has a rather concerning late shipment rate 56% which related to the delivery system

---

## Recommendations
Based on all the analysis I would like to make some recommendations as follows:
- **Restore revenue and profit (recommended for each inventory segmentation)**:

**XA, YA**:

The historical data shows that both segments are in high demand from customers and are also the main source of revenue for the company. Bringing these segments back will be one of the top goals. The company can find new suppliers and redesigning the supply chain so that the incident of Q4/2017 won’t happen again.

This includes further investigation to identify where supply chain disruptions are occurring and filtering out suitable alternative suppliers. For example, items such as clothing, footwear and apparel are often imported from Southeast Asia, West Asia and Central America. Meanwhile, Electronics and Technology products are manufactured in both North America and East Asia. Therefore, we are not sure where the disruption occurred.

**XB, XC**:

These segments show a certain growth potential that cannot be ignored. XB and XC had surged and shouldered all of the company's revenue by the time the disruption occurred. So the first thing to do is conducting market research to understand current consumption trends and market's demands for the emerging products.

Analyze both local and global markets to identify gaps and opportunities is also necessary. Furthermore, monitoring the evolution of this segment and forecast customer demand (if possible) to optimize inventory management is recommended.

**YB, YC, ZB, ZC**:

These segments do not contribute significantly to the company's revenue. They also have low or no demand. I recommend dropping these segments or reducing inventory to optimize storage cost.
- **Overstock**:

If it is available, demand forecasting is crucial. By analyzing historical sales data, tracking market trends, and employing predictive analytics, the company can anticipate future demand, allowing to maintain an optimal inventory level. Additionally, setting up reorder points for each product is essential. These points serve as indicators for reordering, factoring in lead time, sales velocity, and desired safety stock levels, ensuring the company replenish stock in a timely manner without risking overstocking. Speaking of safety stock, maintaining this buffer inventory is vital. It acts as a safeguard against unexpected demand fluctuations or supply delays, mitigating the chances of stockouts and the associated revenue loss while also minimizing the risk of excessive stock accumulation.

Recalling again that customers by each market have a strong increase at a certain time. In Q4/2017, it was customers from Asia Pacific market, previously Europe. Based on this cycle, I expect customers from Asia Pacific to make up the majority in Q1/2018, followed by North America in the following quarters. The company can rely on this information to calculate the appropriate amount of inventory and consider predicting the product life cycle for each region.
- **High late shipment rate**:

In order to effectively curb the issue of high late shipment rates, the company should consider implementing a multifaceted approach.

Firstly, the optimization of the delivery system through a redesigned transportation route, such as adopting a cross-docking strategy, can significantly enhance the efficiency of shipments.

Furthermore, collaboration with local logistics companies to leverage their existing resources and expertise is not a bad idea, specially with markets far away from USA and Peuter Rico.

Additionally, to bolster the overall delivery system and expand the company's global reach, establishing a warehouse in a strategic logistics hub like Singapore in Asia would be beneficial. This move would not only improve the delivery process but also facilitate smoother operations across various international markets.

  

---


## Power BI Dashboard:

- **Sales Management Dashboard**: Tracks revenue, profit, orders, 
  and customer activity
- **Inventory Management Dashboard**: Monitors stock levels, 
  order fulfillment rates, and distribution costs
- **Shipping Management Dashboard**: Tracks delivery timeliness 
  and route performance

![ghfhdh](https://github.com/whykaysoft/Maersk_SCM_Analytics/blob/main/MAERSK%20Dashboard%201,2%20&%203.png?raw=true)


---

## About This Project

Built as part of my transition into Supply Chain Data Analytics. 
My background as Founder & CEO of WhykaySoft Logistics gave me 
direct operational context for every insight in this analysis. I've managed the exact challenges this data describes.
