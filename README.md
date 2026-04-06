# 🛒 Brazilian E-Commerce Analytics — Olist Power BI Dashboard

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)

---

## 📌 Overview

This project is a full **Business Intelligence solution** built in **Power BI** using the [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce). It covers 100,000+ orders placed between 2016 and 2018 across multiple Brazilian marketplaces.

The project involves building a **star schema dimensional model**, performing **data transformations in Power Query**, creating **DAX measures**, and answering key business questions through **interactive visualisations**.

---

## 📂 Dataset

The dataset is sourced from [Kaggle — Olist Brazilian E-Commerce](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) and consists of the following tables:

| File | Description |
|---|---|
| `olist_orders_dataset.csv` | Order lifecycle with timestamps |
| `olist_order_items_dataset.csv` | Items per order with price and freight |
| `olist_order_payments_dataset.csv` | Payment methods and values |
| `olist_order_reviews_dataset.csv` | Customer review scores and comments |
| `olist_order_customer_dataset.csv` | Customer location data |
| `olist_sellers_dataset.csv` | Seller location data |
| `olist_products_dataset.csv` | Product attributes and categories |
| `olist_geolocation_dataset.csv` | Zip code to lat/lon mapping |
| `product_category_name_translation.csv` | Portuguese to English category names |

---

## 🏗️ Dimensional Model (Star Schema)

### Fact Tables
| Table | Type | Description |
|---|---|---|
| `Fact_OrderItems` | Transactional | One row per order item with price and freight |
| `Fact_Payments` | Transactional | Combined payments and invoiced amounts per order |
| `Fact_AccSnapshot` | Accumulating Snapshot | Order lifecycle with all date milestones |
| `Fact_Reviews` | Transactional | Customer review scores and timestamps |

### Dimension Tables
| Table | Description |
|---|---|
| `Dim_Customers` | Customer details and location |
| `Dim_Sellers` | Seller details and location |
| `Dim_Products` | Product attributes with English category names |
| `Dim_Date` | Full date table with time intelligence support |

---

## 📊 Report Pages & Visualisations

| # | Question | Visual Type |
|---|---|---|
| 1 | Most popular product categories | Bar chart + Treemap |
| 2 | Geographic region with highest sales | Choropleth map + Bar chart |
| 3 | Sales on a map | Filled map / Bubble map |
| 4 | Relationship between product dimensions and freight cost | Scatter plot |
| 5 | Rounding error between invoiced and paid amounts | Table with traffic light indicators 🔴🟡🟢 |
| 6 | Distribution of approval and delivery times | Histogram / Box plot |
| 7 | Relationship between delivery times and review scores | Scatter + Line chart |
| 8 | Orders/products with extremely high freight | Table + Bar chart |

---

## 🧮 DAX Measures

### SUM / AVERAGE / MIN / MAX
```dax
Total Revenue = SUM(Fact_OrderItems[price])
Avg Freight Cost = AVERAGE(Fact_OrderItems[freight_value])
Min Review Score = MIN(Fact_Reviews[review_score])
Max Payment Value = MAX(Fact_Payments[payment_value])
```

### CALCULATE
```dax
Revenue Delivered Orders =
CALCULATE(
    SUM(Fact_OrderItems[price]),
    Fact_AccSnapshot[order_status] = "delivered"
)
```

### RANK
```dax
Category Revenue Rank =
RANKX(
    ALL(Dim_Products[product_category_name_english]),
    CALCULATE(SUM(Fact_OrderItems[price])),
    , DESC, DENSE
)
```

### RELATED / RELATEDTABLE / USERELATIONSHIP
```dax
-- RELATED: Pull customer state into fact table (calculated column)
Customer State = RELATED(Dim_Customers[customer_state])

-- RELATEDTABLE: Count orders per seller (calculated column)
Number of Orders = COUNTROWS(RELATEDTABLE(Fact_OrderItems))

-- USERELATIONSHIP: Activate inactive date relationship
Revenue by Delivery Date =
CALCULATE(
    SUM(Fact_OrderItems[price]),
    USERELATIONSHIP(Dim_Date[Date], Fact_AccSnapshot[order_delivered_customer_date])
)
```

### Rounding Error with Traffic Lights
```dax
Rounding_Error =
SUMX(
    VALUES(Fact_Payments[order_id]),
    ABS(
        CALCULATE(MAX(Fact_Payments[total_invoiced]))
        - CALCULATE(SUM(Fact_Payments[payment_value]))
    )
)

Traffic_Light_Rounding =
VAR _error = [Rounding_Error]
RETURN
SWITCH(
    TRUE(),
    ISBLANK(_error), "No Data",
    _error > 100, UNICHAR(128992),   -- 🔴 Red
    _error > 10,  UNICHAR(128993),   -- 🟡 Yellow
    UNICHAR(128994)                   -- 🟢 Green
)
```

---

## ⚙️ Power Query Transformations

- Grouped `olist_order_items_dataset` by `order_id` to calculate `total_invoiced` (price + freight)
- Merged payments with aggregated order items to create `Fact_Payments`
- Grouped `Fact_Payments` by `order_id` to avoid duplicate `total_invoiced` for multi-payment orders
- Created a custom `Dim_Date` table with full time intelligence columns
- Merged English product category names into `Dim_Products`

---

## 🔗 Model Relationships

- `Dim_Date[Date]` → `Fact_AccSnapshot[order_purchase_timestamp]` **(active)**
- `Dim_Date[Date]` → `Fact_AccSnapshot[order_delivered_customer_date]` **(inactive — used via USERELATIONSHIP)**
- `Dim_Date[Date]` → `Fact_AccSnapshot[order_estimated_delivery_date]` **(inactive)**
- `Dim_Customers[customer_id]` → `Fact_AccSnapshot[customer_id]`
- `Dim_Products[product_id]` → `Fact_OrderItems[product_id]`
- `Dim_Sellers[seller_id]` → `Fact_OrderItems[seller_id]`

---

## 🚀 How to Open

1. Download or clone this repository
2. Download the Olist dataset from [Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) and place the CSV files in a local folder
3. Open `olist_assignment.pbix` in **Power BI Desktop**
4. Update the data source path in **Transform Data → Data Source Settings** to point to your local CSV folder
5. Click **Refresh**

---

## 📁 Repository Structure

```
📦 olist-powerbi
 ┣ 📊 olist_assignment.pbix     # Main Power BI file
 ┣ 📄 README.md                 # This file
 ┗ 📁 OLIST/                     # Place Olist CSV files here (not tracked in git)
```

---

## 📝 Notes

- All text identifying real stores and partners in reviews was replaced with **Game of Thrones house names** in the original dataset
- The rounding error analysis revealed discrepancies likely due to **vouchers and split payment methods** reducing the final amount paid below the invoiced total — not purely floating point rounding
- The geolocation dataset contains multiple lat/lon coordinates per zip code prefix — careful aggregation (e.g. averaging) is required

---

## 👤 Author

**Abderrahmen Malouche**  
Business Intelligence Assignment — Olist Brazilian E-Commerce Dataset
