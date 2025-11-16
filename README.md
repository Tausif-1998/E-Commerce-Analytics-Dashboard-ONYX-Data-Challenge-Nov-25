# **E-Commerce Analytics Dashboard | Onyx Data Challenge – November 2025**

A full Power BI analytics solution built for the *Onyx Data Challenge – Nov 2025*, analyzing global e-commerce behavior across sales, loyalty, discount campaigns, refunds, product performance, and pricing. Includes 4 interactive dashboard pages, optimized data modeling, and 12+ DAX calculations for deep business insights.

---

## **📁 Project Overview**

This repository contains my submission for the **Onyx Data | ZoomCharts | Smart Frames UI | Data DNA** E-Commerce Analytics Challenge (Nov 2025).
The goal: transform raw events data into a business-ready analytics dashboard that answers key questions around loyal customers, sales drivers, discounts, pricing, and product performance.

---

## **📊 Dashboard Pages**

### **1. Executive Overview**

High-level KPIs, sales performance, loyal customer contribution, and revenue trends.

<img width="1000" height="550" alt="Screenshot 2025-11-11 165433" src="https://github.com/user-attachments/assets/64f6554d-3b2f-44e9-a5a0-8fd29143b3ea" />

---

### **2. Customer & Loyalty**

Customer segmentation (new vs. repeat), days-to-second purchase, and loyalty-driven revenue.

<img width="1000" height="550" alt="Screenshot 2025-11-11 165520" src="https://github.com/user-attachments/assets/007878ad-a47c-4af8-ae49-b74472cb8307" />


---

### **3. Product & Pricing Performance**

Top products, revenue per customer, pricing plans (annual vs. monthly), ASP, and attach-rate analysis.

<img width="1000" height="550" alt="Screenshot 2025-11-11 165553" src="https://github.com/user-attachments/assets/c233de0d-bf66-488b-9e43-de607742b090" />

---

### **4. Campaigns & Refunds**

Discount usage, repeat % by discount code, refund hotspots, and channel-level issues.
<img width="1000" height="550" alt="Screenshot 2025-11-11 165626" src="https://github.com/user-attachments/assets/65aa9044-b67a-4ce1-b642-24fa2e346b5e" />

---

## **🔍 Key Insights**

* Loyal customers contribute **over 50% of revenue**.
* **Annual plans outperform** monthly plans in CLV and repeat purchases.
* Discounts vary highly in effectiveness — some drive loyalty, others do not.
* Refund issues cluster in specific channels and products.
* Attach rate reveals strong cross-sell opportunities with add-ons.
* Customer repeat time (days to 2nd purchase) highlights key retention windows.

---

## **🧭 Dashboard Navigation Guide**

1. **Executive Overview** – Start here for key KPIs and total business performance.
2. **Customer & Loyalty** – Dig into retention patterns and customer segmentation.
3. **Product & Pricing** – Evaluate pricing model performance, top products, and ASP.
4. **Campaigns & Refunds** – Analyze discounts, refunds, and marketing effectiveness.

Each page includes dynamic slicers:
Country, Date, Product, Channel, Campaign (discount code), Billing cycle, Currency.

---

## **📂 Dataset & Challenge Brief**

The challenge required building a dashboard that answers:

* How do sales trend monthly?
* Which channels drive repeat revenue?
* What percent of sales come from loyal customers?
* Which products and plans perform best?
* How long do customers take to make a second purchase?
* What impact do discounts have on repeat purchases?
* Which areas are refund hotspots?
* What is the attach rate of add-ons to core products?

Dataset includes:
Events (orders, refunds), customers, products, pricing plans, discount codes.

---

# **🧾 DAX Calculated Columns & Measures**

> All DAX below is clean, formatted, and ready for direct use in Power BI.

---

## **Calculated Columns**

### **Customer Type — Customers**

```DAX
Customer Type =
VAR Purchases =
    CALCULATE(
        DISTINCTCOUNT(Events[event_id]),
        FILTER(Events, Events[customer_id] = Customers[customer_id] && Events[event_type] = "order")
    )
RETURN
    IF(Purchases > 1, "Repeat", "New")
```

### **IsOrder — Events**

```DAX
IsOrder = IF(Events[event_type] = "order", 1, 0)
```

---

## **Core Measures**

(Here—unchanged, complete set of all 18 original measures.)

### **Total Sales**

```DAX
Total Sales =
CALCULATE(SUM(Events[net_revenue_usd]), Events[event_type] = "order")
```

### **Total Orders**

```DAX
Total Orders =
CALCULATE(DISTINCTCOUNT(Events[event_id]), Events[event_type] = "order")
```

### **Total Customers**

```DAX
Total Customers = DISTINCTCOUNT(Events[customer_id])
```

### **Purchase Count (per customer)**

```DAX
Purchase Count (per customer) =
CALCULATE(
    DISTINCTCOUNT(Events[event_id]),
    FILTER(ALL(Events), Events[customer_id] = SELECTEDVALUE(Events[customer_id]) && Events[event_type] = "order")
)
```

### **Repeat Customers**

```DAX
Repeat Customers =
CALCULATE(
    DISTINCTCOUNT(Customers[customer_id]),
    FILTER(ALL(Customers), Customers[Customer Type] = "Repeat")
)
```

### **Non-Repeat Customers**

```DAX
Non-Repeat Customers =
CALCULATE(
    DISTINCTCOUNT(Customers[customer_id]),
    FILTER(ALL(Customers), Customers[Customer Type] = "New")
)
```

### **Loyal Customer Sales**

```DAX
Loyal Customer Sales =
CALCULATE(
    [Total Sales],
    KEEPFILTERS( FILTER( Customers, Customers[Customer Type] = "Repeat" ) )
)
```

### **% Sales from Loyal Customers**

```DAX
Pct Sales from Loyal Customers =
DIVIDE([Loyal Customer Sales], [Total Sales], 0)
```

### **ASP (Average Selling Price)**

```DAX
ASP = DIVIDE(SUM(Events[net_revenue_usd]), SUM(Events[quantity]), 0)
```

### **Refund Count**

```DAX
Refund Count =
CALCULATE(COUNTROWS(Events), Events[is_refunded] = TRUE(), Events[event_type] = "order")
```

### **Refund Sales**

```DAX
Refund Sales =
CALCULATE(SUM(Events[net_revenue_usd]), Events[is_refunded] = TRUE(), Events[event_type] = "order")
```

### **Refund Rate**

```DAX
Refund Rate = DIVIDE([Refund Count], [Total Orders], 0)
```

### **Revenue per Customer**

```DAX
Revenue per Customer =
DIVIDE([Total Sales], DISTINCTCOUNT(Events[customer_id]), 0)
```

### **Revenue per Customer by Plan**

```DAX
Revenue per Customer by Plan =
DIVIDE(
    CALCULATE([Total Sales], Events[event_type] = "order"),
    CALCULATE(DISTINCTCOUNT(Events[customer_id]), Events[event_type] = "order"),
    0
)
```

### **Days to 2nd Purchase**

```DAX
Days to 2nd Purchase =
VAR Cust = SELECTEDVALUE(Events[customer_id])
VAR FirstDate =
    CALCULATE(
        MIN(Events[event_date]),
        FILTER(Events, Events[customer_id] = Cust && Events[event_type] = "order")
    )
VAR SecondDate =
    CALCULATE(
        MINX(
            FILTER(Events, Events[customer_id] = Cust && Events[event_type] = "order" && Events[event_date] > FirstDate),
            Events[event_date]
        )
    )
RETURN
    IF(ISBLANK(SecondDate), BLANK(), DATEDIFF(FirstDate, SecondDate, DAY))
```

### **Discount Usage Count**

```DAX
Discount Usage Count =
CALCULATE(
    DISTINCTCOUNT(Events[event_id]),
    Events[event_type] = "order",
    NOT(ISBLANK(Events[discount_code]))
)
```

### **Repeat % by Discount**

```DAX
Repeat % by Discount =
VAR Code = SELECTEDVALUE(Events[discount_code])
VAR CustWithCode =
    CALCULATETABLE(
        DISTINCT(Events[customer_id]),
        Events[event_type] = "order",
        Events[discount_code] = Code
    )
VAR RepeatCustWithCode =
    CALCULATE(
        DISTINCTCOUNT(Events[customer_id]),
        Events[event_type] = "order",
        Events[discount_code] = Code,
        RELATED(Customers[Customer Type]) = "Repeat"
    )
RETURN
    IF(COUNTROWS(CustWithCode)=0, BLANK(), DIVIDE(RepeatCustWithCode, COUNTROWS(CustWithCode), 0))
```

### **Attach Rate (Selected Core vs Add-on)**

```DAX
Attach Rate Selected =
VAR CoreProduct = SELECTEDVALUE(CoreProductSelector[product_id])
VAR AddonProduct = SELECTEDVALUE(AddonProductSelector[product_id])
VAR CoreCust =
    CALCULATETABLE(
        DISTINCT(Events[customer_id]),
        Events[product_id] = CoreProduct,
        Events[event_type] = "order"
    )
VAR AddonCust =
    CALCULATETABLE(
        DISTINCT(Events[customer_id]),
        Events[product_id] = AddonProduct,
        Events[event_type] = "order"
    )
VAR CoreCount = COUNTROWS(CoreCust)
VAR BothCount = COUNTROWS(INTERSECT(CoreCust, AddonCust))
RETURN
    IF(CoreCount = 0, BLANK(), DIVIDE(BothCount, CoreCount, 0))
```

---

# **🏗️ Tech Stack**

* Power BI Desktop
* DAX
* Power Query (M)
* Star-schema Data Modeling
* ZoomCharts Advanced Visuals

---

# **🔗 Useful Links**

* LinkedIn Post Announcement: *[LinKedIn](https://www.linkedin.com/posts/tausif16_datadna-powerbi-onyxdata-activity-7393986258001199104-h4ME?utm_source=share&utm_medium=member_desktop&rcm=ACoAAC38tCMBH7WL5X3BYX0PPO9CjODO_kM-2jo)*
* Publish-to-Web Dashboard: *[E-Commerce Analytics Dashboard](https://lnkd.in/dWJhAx6T)*


---

# **📜 License**
This project is licensed under the MIT License – see the [LICENSE](LICENSE) file for details.  

---


# **Contributing to E-Commerce Analytics Dashboard**

I appreciate your interest in contributing!
This project welcomes suggestions, improvements, and enhancements.



