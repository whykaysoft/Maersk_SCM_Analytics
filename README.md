# Maersk Supply Chain Optimization

> Analyzing supply chain operations through data-driven insights and 
> interactive dashboards to improve inventory management, shipping 
> efficiency, and customer demand fulfillment.
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

## Methodology

### 1. Data Preprocessing
- Removed irrelevant columns and corrected data types
- Resolved inconsistencies across order, shipment, and inventory tables
- Engineered new features to measure shipment timeliness and 
  calculate business KPIs

### 2. Exploratory Data Analysis (EDA)
- Business performance metrics by region and product category
- Customer demographics and purchasing behavior patterns
- Product profitability and inventory turnover analysis

### 3. Inventory Segmentation — ABC XYZ Analysis
Categorized all SKUs based on revenue contribution (ABC) and 
demand stability (XYZ) to prioritize inventory management decisions.

| Segment | Strategy |
|---|---|
| XA, YA (High demand, stable) | Protect stock levels, prioritize reorder |
| XB, XC (Emerging) | Market research, cautious expansion |
| Low-performing segments | Reduce or discontinue to cut storage cost |

---

## Key Insights & Recommendations

- **Transportation redesign** — Rerouting high-delay corridors 
  reduces average shipment delay by an estimated X days
- **Strategic warehouse placement** — Positioning a hub in a 
  high-volume region improves serviceability for international orders
- **Demand forecasting integration** — Implementing reorder point 
  logic reduces stockout incidents in high-velocity segments
- **Local logistics partnerships** — Collaborating with last-mile 
  providers improves delivery performance in underserved markets

---

## Power BI Dashboard:

- **Sales Management Dashboard** — Tracks revenue, profit, orders, 
  and customer activity
- **Inventory Management Dashboard** — Monitors stock levels, 
  order fulfillment rates, and distribution costs
- **Shipping Management Dashboard** — Tracks delivery timeliness 
  and route performance

![](your-powerbi-screenshot.png)


---

## 📁 Repository Structure
