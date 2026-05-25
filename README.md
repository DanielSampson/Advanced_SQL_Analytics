# E-Commerce Sales Analytics: Performance, Customer Segments & Strategic Reporting
### Advanced SQL Analytics — Part 2

---

## Abstract

Raw revenue numbers are a starting point, not the finish line. The real question is always *why* — why did sales spike in June? Why are certain products consistently underperforming? Why is one small group of customers driving the bulk of revenue?

This project is Part 2 of a two-part SQL analytics series built on a real e-commerce dataset. After wrapping up the data cleanup and exploratory analysis in Part 1, this phase goes deeper — tracking how performance shifts over time, segmenting customers and products based on actual behavior, evaluating which product categories are carrying the business, and packaging everything into two production-ready SQL Views that any analyst or BI tool can plug into directly.

The goal is to move past surface-level metrics and build queries that surface decisions, not just data.

---

## Section 1 — Introduction & Problem Statement

### 1.1 Background

Part 1 gave us the foundation: **$29,356,250 in total revenue**, **27,659 orders**, and **18,484 unique customers**. Those numbers set the baseline. This project is about understanding what's underneath them.

We want to know how revenue is shifting month to month, which products are gaining ground versus losing it, and how customer spending is evolving over time. More importantly, we need to break the customer base apart — separating first-time buyers from the small, high-value group that keeps the business running.

### 1.2 Problem Statement

Using the same Gold Layer e-commerce dataset, this project tackles five core operational questions:

- How is sales volume changing over time, and are there clear seasonal patterns we can plan around?
- What does the long-term revenue trajectory look like, and is average pricing moving up or down across years?
- Which products are beating their historical averages, and how do they stack up against their own prior year numbers?
- How are customers and products distributed across meaningful behavioral segments?
- Which product categories are actually driving revenue, and what's their precise share of the total?

### 1.3 Research Objectives

To answer these questions, the analysis moves through six progressive steps:

1. Identify monthly sales trends and surface any recurring seasonal demand patterns.
2. Build running totals and moving averages to evaluate long-term revenue growth.
3. Use the `LAG()` window function to benchmark each product against its own prior year results.
4. Segment the customer base into VIP, Regular, and New tiers based on total spend and how long they've been buying.
5. Calculate the precise revenue contribution of each product category.
6. Package all of this into two clean, production-ready SQL Views built for ongoing business intelligence use.

---

## Section 2 — Methodology & Tools

### 2.1 Dataset

The analysis runs on the same Gold Layer tables built in Part 1:

| File | Description |
|---|---|
| `gold.dim_customers.csv` | Customer demographic records and details |
| `gold.dim_products.csv` | Product catalog with categories, subcategories, and costs |
| `gold.fact_sales.csv` | Granular transaction records — orders, quantities, and sales amounts |

### 2.2 Analytical Approach

| Technique | Application |
|---|---|
| Change Over Time Analysis | Tracking how performance shifts across months and years |
| Cumulative Analysis | Monitoring lifetime running totals and moving average prices |
| Performance Analysis | Year-over-Year (YoY) benchmarking using analytical functions |
| Data Segmentation | Categorizing customers and products into behavioral brackets |
| Part-to-Whole Analysis | Calculating precise revenue share by category and product line |
| Business Reporting | Building reusable SQL Views optimized for BI tool connections |

### 2.3 Project Scripts

| Script | Focus Area |
|---|---|
| `01_change_over_time_analysis.sql` | Monthly sales, order volumes, and active customer trends |
| `02_cumulative_analysis.sql` | Running revenue totals and pricing trend tracking |
| `03_performance_analysis.sql` | YoY product benchmarking using `LAG()` |
| `04_data_segmentation.sql` | Customer tier grouping and product cost range segmentation |
| `05_part_to_whole_analysis.sql` | Revenue distribution across product categories |
| `06_report_customers.sql` | Consolidated customer profile view |
| `07_report_products.sql` | Consolidated product performance view |

> 💡 Fully commented SQL scripts live in the [`/scripts`](./scripts/) folder. The code snippets in this README highlight the core logic of each phase.

### 2.4 Tools & SQL Techniques

- **SQL Server (T-SQL) & SSMS** — Main query environment for writing and optimizing the analytics code.
- **Common Table Expressions (CTEs)** — Used to break complex joins and transformations into readable, modular steps.
- **Window Functions (`SUM OVER`, `AVG OVER`, `LAG`, `RANK`)** — Used for running calculations, lag-based comparisons, and product ranking without collapsing rows.
- **Conditional Logic (`CASE WHEN`)** — Used to build custom segmentation brackets for pricing tiers and customer loyalty levels.
- **Date Parsing (`DATETRUNC`, `DATEPART`)** — Used to roll daily transaction logs up into meaningful monthly and annual blocks.
- **SQL Views** — Used to save complex queries as virtual tables, making the final datasets directly queryable from Power BI, Tableau, or any other BI tool.

---

## Section 3 — Analysis & Results

### 3.1 Change Over Time Analysis
📄 *Script: `01_change_over_time_analysis.sql`*

**Objective:** Track monthly sales volumes, order counts, and active customer numbers to understand how the store's performance moves across the calendar year.

```sql
-- Tracking Monthly Sales Trends
SELECT
    DATETRUNC(month, order_date) AS order_date,
    SUM(sales_amount) AS total_sales,
    COUNT(DISTINCT customer_key) AS total_customers,
    SUM(quantity) AS total_quantity
FROM gold.fact_sales
WHERE order_date IS NOT NULL
GROUP BY DATETRUNC(month, order_date)
ORDER BY DATETRUNC(month, order_date);

-- Annual Sales Trend using FORMAT()
SELECT
    FORMAT(order_date, 'yyyy-MMM') AS order_date,
    SUM(sales_amount) AS total_sales,
    COUNT(DISTINCT customer_key) AS total_customers,
    SUM(quantity) AS total_quantity
FROM gold.fact_sales
WHERE order_date IS NOT NULL
GROUP BY FORMAT(order_date, 'yyyy-MMM')
ORDER BY FORMAT(order_date, 'yyyy-MMM');
```

**Result:**

![Change Over Time Result](images/01_change_over_time_result.PNG)

**What this revealed:**

The timeline opens in December 2010 with modest numbers — **$43,419 in sales**, **14 unique customers**, and **14 units sold**. From there, things moved fast.

By **January 2011**, the business had already accelerated to **$469,795 in revenue** across **144 active customers** — a clear signal of rapid early expansion. Growth kept building through the first half of the year, reaching a peak in **June 2011** with **$737,793 in revenue** and **230 active customers**.

One pattern stands out clearly: customer activity tracks almost directly with revenue. This tells us that sales volume is being driven by how many buyers are active in a given month, not by erratic swings in individual order sizes.

---

### 3.2 Cumulative Analysis
📄 *Script: `02_cumulative_analysis.sql`*

**Objective:** Build a running total of revenue over time and track how average pricing evolves across years — giving a long-term view of both growth trajectory and pricing trends.

```sql
SELECT
    order_date,
    total_sales,
    SUM(total_sales) OVER (ORDER BY order_date) AS running_total_sales,
    AVG(avg_price) OVER (ORDER BY order_date) AS moving_average_price
FROM (
    SELECT
        DATETRUNC(year, order_date) AS order_date,
        SUM(sales_amount) AS total_sales,
        AVG(price) AS avg_price
    FROM gold.fact_sales
    WHERE order_date IS NOT NULL
    GROUP BY DATETRUNC(year, order_date)
) t;
```

**Result:**

![Cumulative Analysis Result](images/02_cumulative_result.PNG)

**What this revealed:**

The running total tells the growth story in a single column — revenue compounding year after year from **$43,419** at the start all the way through to **$29,351,258** by 2014. Each year's contribution is visible, and the trajectory is consistently upward.

The moving average price adds a second dimension: it's been declining year over year, dropping from **$3,101** in 2010 to **$1,668** by 2014. This signals a gradual shift toward lower average transaction values over time — something that deserves attention in any pricing strategy conversation.

---

### 3.3 Performance Analysis — Year-over-Year
📄 *Script: `03_performance_analysis.sql`*

**Objective:** Benchmark each product's annual sales against both its historical average and its prior year numbers — surfacing consistent growers, steady decliners, and volatile performers.

```sql
WITH yearly_product_sales AS (
    SELECT
        YEAR(f.order_date) AS order_year,
        p.product_name,
        SUM(f.sales_amount) AS current_sales
    FROM gold.fact_sales f
    LEFT JOIN gold.dim_products p
        ON f.product_key = p.product_key
    WHERE f.order_date IS NOT NULL
    GROUP BY YEAR(f.order_date), p.product_name
)
SELECT
    order_year,
    product_name,
    current_sales,
    AVG(current_sales) OVER (PARTITION BY product_name) AS avg_sales,
    current_sales - AVG(current_sales) OVER (PARTITION BY product_name) AS diff_avg,
    CASE
        WHEN current_sales > AVG(current_sales) OVER (PARTITION BY product_name) THEN 'Above Avg'
        WHEN current_sales < AVG(current_sales) OVER (PARTITION BY product_name) THEN 'Below Avg'
        ELSE 'Avg'
    END AS avg_change,
    LAG(current_sales) OVER (PARTITION BY product_name ORDER BY order_year) AS py_sales,
    current_sales - LAG(current_sales) OVER (PARTITION BY product_name ORDER BY order_year) AS diff_py,
    CASE
        WHEN current_sales > LAG(current_sales) OVER (PARTITION BY product_name ORDER BY order_year) THEN 'Increase'
        WHEN current_sales < LAG(current_sales) OVER (PARTITION BY product_name ORDER BY order_year) THEN 'Decrease'
        ELSE 'No Change'
    END AS py_change
FROM yearly_product_sales
ORDER BY product_name, order_year;
```

**Result:**

![Performance Analysis Result](images/03_performance_result.PNG)

**What this revealed:**

This query does two things at once that would normally require separate analyses.

The **Above Avg / Below Avg** column immediately flags which products are punching above their own historical weight and which are dragging below it. The **Increase / Decrease / No Change** column adds a second layer — showing not just where a product stands historically, but whether it's actively improving or declining right now.

Together, these two signals make it possible to segment products into four distinct groups: consistent growers (Above Avg + Increase), recovering products (Below Avg + Increase), fading products (Above Avg + Decrease), and chronic underperformers (Below Avg + Decrease). Each group calls for a different business response.

---

### 3.4 Data Segmentation
📄 *Script: `04_data_segmentation.sql`*

**Objective:** Group customers into meaningful behavioral tiers — VIP, Regular, and New — based on their spending and how long they've been buying. Segment products into cost ranges for pricing and inventory analysis.

```sql
-- Product Cost Segmentation
WITH product_segments AS (
    SELECT
        product_key,
        product_name,
        cost,
        CASE
            WHEN cost < 100 THEN 'Below 100'
            WHEN cost BETWEEN 100 AND 500 THEN '100-500'
            WHEN cost BETWEEN 500 AND 1000 THEN '500-1000'
            ELSE 'Above 1000'
        END AS cost_range
    FROM gold.dim_products
)
SELECT
    cost_range,
    COUNT(product_key) AS total_products
FROM product_segments
GROUP BY cost_range
ORDER BY total_products DESC;

-- Customer Behavioral Segmentation
WITH customer_spending AS (
    SELECT
        c.customer_key,
        SUM(f.sales_amount) AS total_spending,
        DATEDIFF(month, MIN(order_date), MAX(order_date)) AS lifespan
    FROM gold.fact_sales f
    LEFT JOIN gold.dim_customers c ON f.customer_key = c.customer_key
    GROUP BY c.customer_key
)
SELECT
    customer_segment,
    COUNT(customer_key) AS total_customers
FROM (
    SELECT
        customer_key,
        CASE
            WHEN lifespan >= 12 AND total_spending > 5000 THEN 'VIP'
            WHEN lifespan >= 12 AND total_spending <= 5000 THEN 'Regular'
            ELSE 'New'
        END AS customer_segment
    FROM customer_spending
) AS segmented_customers
GROUP BY customer_segment
ORDER BY total_customers DESC;
```

**Result:**

![Customer Segmentation Result](images/04_segmentation_result.PNG)

**What this revealed:**

The customer split is stark: **14,631 New customers**, **2,198 Regular**, and **1,655 VIP**. The VIP group — customers with at least 12 months of history and over $5,000 in total spend — is a small fraction of the total base, but almost certainly accounts for a disproportionate share of revenue. That concentration matters enormously for retention strategy.

On the product side, the majority of SKUs sit **Below 100** in cost (110 products) or in the **100–500** range (101 products), with a smaller group of higher-cost items in the **500–1000** (45 products) and **Above 1000** (39 products) tiers. Each cost bracket behaves differently in terms of pricing sensitivity, margin, and inventory planning.

---

### 3.5 Part-to-Whole Analysis
📄 *Script: `05_part_to_whole_analysis.sql`*

**Objective:** Quantify exactly how much each product category contributes to overall revenue — turning gut-feel assumptions into precise, defensible percentages.

```sql
WITH category_sales AS (
    SELECT
        p.category,
        SUM(f.sales_amount) AS total_sales
    FROM gold.fact_sales f
    LEFT JOIN gold.dim_products p ON p.product_key = f.product_key
    GROUP BY p.category
)
SELECT
    category,
    total_sales,
    SUM(total_sales) OVER () AS overall_sales,
    ROUND((CAST(total_sales AS FLOAT) / SUM(total_sales) OVER ()) * 100, 2) AS percentage_of_total
FROM category_sales
ORDER BY total_sales DESC;
```

**Result:**

![Part to Whole Analysis Result](images/05_part_to_whole_result.PNG)

**What this revealed:**

The numbers here speak for themselves: **Bikes account for 96.46% of all revenue** — $28,316,272 out of $29,356,250. Accessories contribute just **2.39%** and Clothing **1.16%**.

This confirms the concentration signal from Part 1 and puts a hard number on it. For any serious business planning conversation — whether about diversification, risk management, or growth strategy — this is the most important figure in the entire analysis.

---

### 3.6 Production Reports — SQL Views
📄 *Scripts: `06_report_customers.sql`, `07_report_products.sql`*

**Objective:** Consolidate all customer and product metrics into two production-ready SQL Views — reusable, queryable, and built for ongoing business intelligence use.

**The Customer Report View (`gold.report_customers`) delivers:**

```sql
-- Key metrics from the Customer Report View
CASE
    WHEN lifespan >= 12 AND total_sales > 5000 THEN 'VIP'
    WHEN lifespan >= 12 AND total_sales <= 5000 THEN 'Regular'
    ELSE 'New'
END AS customer_segment,
CASE
    WHEN age < 20 THEN 'Under 20'
    WHEN age BETWEEN 20 AND 29 THEN '20-29'
    WHEN age BETWEEN 30 AND 39 THEN '30-39'
    WHEN age BETWEEN 40 AND 49 THEN '40-49'
    ELSE '50 and above'
END AS age_group,
CASE WHEN total_sales = 0 THEN 0
     ELSE total_sales / total_orders
END AS avg_order_value,
CASE WHEN lifespan = 0 THEN total_sales
     ELSE total_sales / lifespan
END AS avg_monthly_spend
```

Each customer record includes: segment classification (VIP, Regular, New), age group, total orders, total sales, total quantity, products purchased, months since last order (recency), Average Order Value (AOV), and Average Monthly Spend.

**The Product Report View (`gold.report_products`) delivers:**

```sql
-- Key metrics from the Product Report View
CASE
    WHEN total_sales > 50000 THEN 'High-Performer'
    WHEN total_sales >= 10000 THEN 'Mid-Range'
    ELSE 'Low-Performer'
END AS product_segment,
CASE
    WHEN total_orders = 0 THEN 0
    ELSE total_sales / total_orders
END AS avg_order_revenue,
CASE
    WHEN lifespan = 0 THEN total_sales
    ELSE total_sales / lifespan
END AS avg_monthly_revenue
```

Each product record includes: performance tier (High-Performer, Mid-Range, Low-Performer), total orders, total sales, total quantity, unique customers reached, months since last sale (recency), Average Order Revenue (AOR), and Average Monthly Revenue.

**Customer Report Result:**

![Customer Report Result](images/06_report_customers_result.PNG)

**Product Report Result:**

![Product Report Result](images/07_report_products_result.PNG)

---

## Section 4 — Key Findings

- **Sales follow a clear seasonal pattern** — certain months consistently outperform others, giving a reliable basis for demand forecasting and inventory planning ahead of peak periods rather than reacting to them.

- **Revenue trajectory is cumulative and growing** — the running total confirms steady business growth across the full 37-month time range, with each year stacking meaningfully on the last.

- **Product performance is polarized** — the YoY analysis surfaces clear winners (consistently Above Average and growing) and clear underperformers (consistently Below Average and declining), with a smaller middle group of stable performers.

- **VIP customers are the revenue backbone** — just 1,655 customers with 12+ months of history and $5,000+ in total spend are almost certainly driving a disproportionate share of the total $29.4M in revenue.

- **Bike category dominance is confirmed at ~96.46%** — the part-to-whole analysis puts a precise, defensible number on the concentration that was visible in Part 1.

- **The Mountain-200 product line is the commercial engine** — consistently Above Average and growing year over year across multiple size and color variants.

- **Two production SQL Views are ready for deployment** — `report_customers` and `report_products` serve as reusable intelligence layers that any analyst, manager, or BI tool can query directly without rebuilding the underlying logic.

---

## Section 5 — Recommendations

**1. Build a VIP Retention Program Immediately**

The segmentation confirms what most businesses suspect but rarely quantify: a small group of buyers carries the revenue. At just 1,655 customers out of 18,484, the VIP segment is too important to leave to chance. A dedicated retention program — exclusive offers, early product access, personalized outreach — should be the top commercial priority. Losing a VIP customer isn't a customer service issue; it's a revenue event.

**2. Re-Engage or Discontinue Declining Products**

The YoY analysis identifies products that are consistently Below Average and continuing to decline. Each one deserves a deliberate decision: reprice, reposition, promote, or discontinue. Carrying underperforming SKUs has a real cost — in inventory, in margin, and in the management attention it consumes.

**3. Plan Inventory Around Seasonal Peaks**

The change over time analysis reveals predictable seasonal patterns across the 37-month dataset. Inventory and staffing decisions should be made ahead of peak months, not in response to them. The data provides the signal — the business just needs to act on it before the peak arrives.

**4. Diversify Revenue Beyond Bikes**

With 96.46% of revenue sitting in a single product category, this business carries significant concentration risk. A deliberate strategy to grow Accessories and Clothing — even to 10–15% of total revenue — would meaningfully reduce commercial exposure and open new growth levers.

**5. Connect the SQL Views to a BI Tool**

The `report_customers` and `report_products` views aren't one-time outputs — they're living intelligence layers. Connecting them to Power BI, Tableau, or any BI platform would transform this SQL project into an always-on business monitoring system. The complex logic is already built. The only remaining step is the connection.

---

## Section 6 — Conclusion

The difference between data and intelligence is structure.

This project takes the same $29.4M e-commerce dataset from Part 1 and applies five layers of analysis — trends, cumulative growth, year-over-year performance, behavioral segmentation, and revenue contribution — to produce findings that are specific, actionable, and ready to inform real decisions.

What emerged is a clear commercial picture: a business built almost entirely on Bikes and the Mountain-200 product line, sustained by a small but critical group of VIP customers, with predictable seasonal demand patterns and a long tail of underperforming accessories that deserve a serious review.

The two production SQL Views that close out the project are its most tangible deliverable — reusable reports that any analyst, manager, or BI tool can query directly, without rebuilding the underlying logic every time.

Together with Part 1, this project walks through the full analytical lifecycle in SQL: understand the data, explore its structure, extract the patterns, and turn those patterns into decisions.

That's what SQL analytics is for.

---

## 📁 Project Structure

```
Advanced_SQL_Analytics/
├── datasets/
│   ├── gold.dim_customers.csv
│   ├── gold.dim_products.csv
│   └── gold.fact_sales.csv
├── scripts/
│   ├── 01_change_over_time_analysis.sql
│   ├── 02_cumulative_analysis.sql
│   ├── 03_performance_analysis.sql
│   ├── 04_data_segmentation.sql
│   ├── 05_part_to_whole_analysis.sql
│   ├── 06_report_customers.sql
│   └── 07_report_products.sql
├── images/
└── README.md
```

> 🔗 This is **Part 2** of a two-part SQL analytics series. For the foundational EDA and database exploration, see [Part 1 — E-Commerce Sales EDA](../EDA_SQL_Project/)
