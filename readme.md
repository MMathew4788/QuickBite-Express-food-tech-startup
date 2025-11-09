# QuickBite Express Crisis Recovery Dashboard

## 📌 Overview

This repository was created as part of the **https://codebasics.io/challenges/codebasics-resume-project-challenge/23**. It contains a **Power BI dashboard project** designed to analyze and visualize the impact of a business crisis on **QuickBite Express**, a food delivery startup. The dashboard provides **actionable insights** to support **business recovery** and **strategic decision-making**.

## 🔍 View the live interactive preview of Dashboard

🔗[Live Preview](https://app.powerbi.com/view?r=eyJrIjoiNWU3MjIzZjItZmZlYS00Mzg5LTgwMTAtMmRjNThiMjhhNTIwIiwidCI6ImM2ZTU0OWIzLTVmNDUtNDAzMi1hYWU5LWQ0MjQ0ZGM1YjJjNCJ9)

---

## ✅ Key Features

- Interactive Power BI dashboard for crisis impact analysis.
- Visualizations for revenue trends, customer behavior, and operational performance.
- Data-driven recommendations for recovery strategies.

## 🚀 Getting Started

- Clone this repository to your local machine.
- Open the Power BI (.pbix) file included in the project.
- Review the data model and dashboard structure.

## 📂 Project Structure

- Data sources and metadata
- Power BI dashboard file
- Supporting documentation

## 🧮 Few DAX Measures used in the Analysis

#### Segment customers into two categories:

- Old: Customers who placed orders before the crisis date.
- New: Customers who joined after the crisis started.

```DAX
Customer_Type_Table =
VAR PreCrisis =
    DISTINCT(
        SELECTCOLUMNS(
            FILTER(fact_orders, fact_orders[order_timestamp] < DATE(2025, 6, 1)),
            "customer_id", fact_orders[customer_id]
        )
    )
VAR PostCrisis =
    DISTINCT(
        SELECTCOLUMNS(
            FILTER(fact_orders, fact_orders[order_timestamp] >= DATE(2025, 6, 1)),
            "customer_id", fact_orders[customer_id]
        )
    )
RETURN
    ADDCOLUMNS(
        PostCrisis,
        "Customer_Type",
        IF([customer_id] IN PreCrisis, "Old", "New")
    )
```

#### Churn Rate: calculates the percentage of customers who stopped ordering after the crisis date (June 1, 2025)

```DAX
Churn_Rate_% =
VAR PreCrisis =
    CALCULATETABLE(
        VALUES(fact_orders[customer_id]),
        fact_orders[order_timestamp] < DATE(2025, 6, 1)
    )
VAR PostCrisis =
    CALCULATETABLE(
        VALUES(fact_orders[customer_id]),
        fact_orders[order_timestamp] >= DATE(2025, 6, 1)
    )
VAR Lost_Customers = COUNTROWS(EXCEPT(PreCrisis, PostCrisis))
VAR Total_PreCrisis = COUNTROWS(PreCrisis)
RETURN
    DIVIDE(Lost_Customers, Total_PreCrisis, 0)

```

#### On Time Delivery %: calculates the percentage of deliveries completed on time by dividing the count of on-time deliveries by the total number of deliveries

```
On Time Delivery % =
VAR ON_Time_Count = CALCULATE(COUNTROWS(fact_delivery_performance),fact_delivery_performance[On_time]="Yes")
Var Total_Count =COUNTROWS(fact_delivery_performance)
RETURN
DIVIDE(ON_Time_Count,Total_Count,0)
```

#### Order Decline %: calculates the percentage decline in orders after the crisis compared to the pre-crisis period, showing the impact of the crisis on order volume

```
Order Decline % =
Var Orders_PreCrisis = CALCULATE([Total Orders], fact_orders[order_timestamp] <= DATE(2025,5,31))
Var Orders_Crisis = CALCULATE([Total Orders], fact_orders[order_timestamp] >= DATE(2025,6,1))
Return
DIVIDE((Orders_PreCrisis-Orders_Crisis), Orders_PreCrisis)

```

#### Percentage Decline in Average Rating: calculates the percentage decrease in average customer rating after the crisis compared to the pre-crisis period, highlighting the impact on service quality

```
Percentage Decline in Average Rating =
Var Avg_Rating_PreCrisis = CALCULATE([Avg Rating], fact_orders[order_timestamp] <= DATE(2025,5,31))
Var Avg_Rating_Crisis = CALCULATE([Avg Rating], fact_orders[order_timestamp] >= DATE(2025,6,1))
Return
DIVIDE((Avg_Rating_PreCrisis-Avg_Rating_Crisis), Avg_Rating_PreCrisis)

```

#### This DAX table creates a list of customers who had positive spending before the crisis, by summarizing their orders and adding a calculated column for pre-crisis spend, then filtering out those with zero spend.

```
Pre_Crisis_Customer_Spend_Table =
    Filter(ADDCOLUMNS(
        SUMMARIZE(fact_orders, fact_orders[customer_id]),
        "Pre_Crisis_Spend", [Pre Crisis Customer Spend]
    ),[Pre Crisis Customer Spend]>0)
```

#### Spend_Threshold: alculates the 95th percentile of pre-crisis customer spend from the Pre_Crisis_Customer_Spend_Table, setting a threshold to identify high-value customers

```
Spend_Threshold =
    PERCENTILEX.INC(Pre_Crisis_Customer_Spend_Table, [Pre_Crisis_Spend], 0.95)
```

#### This DAX measure identifies high-value loyal customers by filtering those who meet a spending threshold before the crisis and have continued ordering after the crisis, while also having valid ratings and order data in both periods.

```
High_Value_Loyal_Customers =
    FILTER(
        ADDCOLUMNS(
            SUMMARIZE(fact_orders, fact_orders[customer_id]),
            "Pre_Crisis_Spend",
                CALCULATE(
                    SUM(fact_orders[total_amount]),
                    fact_orders[order_timestamp] <= DATE(2025,5,31)
                ),
            "Post_Crisis_Spend",
                CALCULATE(
                    SUM(fact_orders[total_amount]),
                    fact_orders[order_timestamp] >= DATE(2025,6,1)
                ),
            "Pre_Crisis_Total_Orders",
                CALCULATE(
                    COUNTROWS(fact_orders),
                    fact_orders[order_timestamp] <= DATE(2025,5,31)
                ),
            "Post_Crisis_Total_Orders",
                CALCULATE(
                    COUNTROWS(fact_orders),
                    fact_orders[order_timestamp] >= DATE(2025,6,1)
                ),
            "Pre_Crisis_Avg_Rating",
                CALCULATE(
                    AVERAGE(fact_ratings[rating]),
                    fact_orders[order_timestamp] <= DATE(2025,5,31)
                ),
            "Post_Crisis_Avg_Rating",
                CALCULATE(
                    AVERAGE(fact_ratings[rating]),
                    fact_orders[order_timestamp] >= DATE(2025,6,1)
                )
        ),
        [Pre_Crisis_Spend] >= Pre_Crisis_Customer_Spend_Table[Spend_Threshold] &&
        NOT(ISBLANK([Pre_Crisis_Spend])) &&
        NOT(ISBLANK([Post_Crisis_Spend])) &&
        NOT(ISBLANK([Pre_Crisis_Avg_Rating])) &&
        NOT(ISBLANK([Post_Crisis_Avg_Rating]))

    )

```

## 📥 Download the Dashboard

To explore all DAX measures and related visuals, download the Power BI file: QuickBite Express.pbix
