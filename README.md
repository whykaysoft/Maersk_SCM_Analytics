# Maersk Supply Chain Optimization

> Identifying $86K in storage cost drivers and 
> 98% late shipment risk using PostgreSQL + Power BI
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

---

## Key Insights

- **Sharp decline in revenue and profit**: The most likely root cause of business downturn comes from a supply chain disruption on the supplier network such as delays in sourcing, transportation, distribution or problem with suppliers could have affected the company's ability to replenish the warehouse inventory promptly, especially for the products of XA and YA segments.
- **Overstock and Understock**: The company often encounters overstock situation, causing excess inventory and wasting storage costs. Sometimes the company is also understocked. This shows that inventory management is not really implemented effectively yet
- **High late shipment rate**: The company has a rather concerning late shipment rate (30 - 50%) which related to the delivery system

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
- **Overstock and Understock**:

If it is available, demand forecasting is crucial. By analyzing historical sales data, tracking market trends, and employing predictive analytics, the company can anticipate future demand, allowing to maintain an optimal inventory level. Additionally, setting up reorder points for each product is essential. These points serve as indicators for reordering, factoring in lead time, sales velocity, and desired safety stock levels, ensuring the company replenish stock in a timely manner without risking overstocking. Speaking of safety stock, maintaining this buffer inventory is vital. It acts as a safeguard against unexpected demand fluctuations or supply delays, mitigating the chances of stockouts and the associated revenue loss while also minimizing the risk of excessive stock accumulation.

Recalling again that customers by each market have a strong increase at a certain time. In Q4/2017, it was customers from Asia Pacific market, previously Europe. Based on this cycle, I expect customers from Asia Pacific to make up the majority in Q1/2018, followed by North America in the following quarters. The company can rely on this information to calculate the appropriate amount of inventory and consider predicting the product life cycle for each region.
- **High late shipment rate**:

In order to effectively curb the issue of high late shipment rates, the company should consider implementing a multifaceted approach.

Firstly, the optimization of the delivery system through a redesigned transportation route, such as adopting a cross-docking strategy, can significantly enhance the efficiency of shipments.

Furthermore, collaboration with local logistics companien to leverage their existing resources and expertise is not a bad idea, specially with markets far away from USA and Peuter Rico.

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
