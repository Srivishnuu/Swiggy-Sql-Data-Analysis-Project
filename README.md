# Swiggy_SQL_Analysis

# 🍽️ Swiggy SQL Data Analysis Project  
<img width="1020" height="400" alt="image" src="https://github.com/user-attachments/assets/b19c2fc9-acf7-40b5-b44e-bca3bff88f17" />

## 📊 Overview  

This project demonstrates **end-to-end SQL data analysis** for **Swiggy**, India's leading online food delivery platform.  
It includes **database creation, data cleaning, complex querying, optimization, and insight generation**—simulating real-world analytical problem-solving for business intelligence.  

The goal is to uncover operational patterns, customer behavior, restaurant performance, and delivery efficiency, and to propose actionable recommendations for Swiggy’s business growth.  

---

## 🧩 Project Structure  
- **Database Design** – Created schema for `customers`, `restaurants`, `orders`, `riders`, and `deliveries`.
- **Data Import & Cleaning** – Managed missing values and validated dataset integrity.  
- **Exploratory Data Analysis (EDA)** – Conducted outlier detection and column consistency checks.  
- **Business Queries (20+)** – Solved real-world analytical problems using advanced SQL (joins, CTEs, window functions).  
- **Stored Procedures** – Automated data retrieval for restaurants and customer order history.  
- **Query Optimization** – Used indexing and efficient query structures to enhance performance.  
- **Insights & Recommendations** – Delivered meaningful conclusions to improve decision-making.  

---
### ERD
<img width="642" height="678" alt="ERD" src="https://github.com/user-attachments/assets/41f43fa8-abb8-4afd-97c5-298621f810d9" />

---
### Database Structure
  ```sql
  CREATE TABLE customer (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone VARCHAR(15),
    city VARCHAR(50),
    registration_date DATE,
    premium_member VARCHAR(5) CHECK (premium_member IN ('true', 'false')) DEFAULT 'false');

  CREATE TABLE restaurants (
    restaurant_id INT PRIMARY KEY,
    restaurant_name VARCHAR(150) NOT NULL,
    cuisine_type VARCHAR(50),
    city VARCHAR(50),
    rating DECIMAL(2,1),
    average_delivery_time INT, -- in minutes
    is_active VARCHAR(5) CHECK (is_active IN ('true', 'false')) DEFAULT 'false',
    commission_rate DECIMAL(4,2) -- percentage);

  CREATE TABLE riders (
    rider_id INT PRIMARY KEY,
    rider_name VARCHAR(100) NOT NULL,
    phone VARCHAR(15),
    city VARCHAR(50),
    joining_date DATE,
    vehicle_type VARCHAR(20), -- bike, scooter, bicycle
    is_active VARCHAR(5) CHECK (is_active IN('true','false')) DEFAULT 'TRUE');

  CREATE TABLE menu_items (
    item_id INT PRIMARY KEY,
    restaurant_id INT NOT NULL,
    item_name VARCHAR(150) NOT NULL,
    category VARCHAR(50), -- starter, main course, dessert, beverage
    price DECIMAL(8,2),
    is_available VARCHAR(5) CHECK (is_available IN('true','false')) DEFAULT 'TRUE',
    FOREIGN KEY (restaurant_id) REFERENCES restaurants(restaurant_id));

  CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    restaurant_id INT NOT NULL,
    rider_id INT,
    order_date DATE NOT NULL,
    order_time TIME NOT NULL,
    delivery_date DATE NOT NULL,
    delivery_time TIME NOT NULL,
    order_status VARCHAR(20), -- placed, confirmed, preparing, picked_up, delivered, cancelled
    total_amount DECIMAL(10,2),
    delivery_fee DECIMAL(6,2),
    discount_amount DECIMAL(8,2) DEFAULT 0,
    payment_method VARCHAR(20), -- credit_card, debit_card, upi, wallet, cod
    delivery_time_mins INT, -- actual delivery time in minutes
    rating DECIMAL(2,1), -- customer rating after delivery
    FOREIGN KEY (customer_id) REFERENCES customer(customer_id),
    FOREIGN KEY (restaurant_id) REFERENCES restaurants(restaurant_id),
    FOREIGN KEY (rider_id) REFERENCES riders(rider_id));

  CREATE TABLE order_details (
    order_detail_id INT PRIMARY KEY,
    order_id INT NOT NULL,
    item_id INT NOT NULL,
    quantity INT NOT NULL,
    item_price DECIMAL(8,2),
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (item_id) REFERENCES menu_items(item_id));
```


## 🧮 Business Problems Solved  

## Q1. Top 5 Most Frequently Ordered Dishes  
```sql
SELECT 
    mi.item_name, 
    SUM(od.quantity) AS total_orders
    FROM order_details as od
JOIN menu_items mi ON od.item_id = mi.item_id
GROUP BY mi.item_id, mi.item_name
ORDER BY total_orders DESC
LIMIT 5;
```

## Q2. Peak Order Time Slot Analysis  
```sql
SELECT 
    CASE
        WHEN HOUR(order_time) BETWEEN 6 AND 10 THEN 'Breakfast (06:00-10:00)'
        WHEN HOUR(order_time) BETWEEN 11 AND 15 THEN 'Lunch (11:00-15:00)'
        WHEN HOUR(order_time) BETWEEN 16 AND 18 THEN 'Snacks/Tea (16:00-18:00)'
        WHEN HOUR(order_time) BETWEEN 19 AND 23 THEN 'Dinner (19:00-23:00)'
        ELSE 'Late Night (00:00-05:00)'
    END AS time_slot,
    COUNT(order_id) AS total_orders,
    CONCAT('₹', FORMAT(SUM(total_amount), 2)) AS revenue,
    ROUND(AVG(total_amount), 2) AS avg_order_value
FROM orders
WHERE order_status = 'Delivered'
GROUP BY time_slot
ORDER BY total_orders DESC;
```

## Q3. Order Value Distribution  
```sql
SELECT 
    CASE 
        WHEN total_amount < 200 THEN 'Low Value (< ₹200)'
        WHEN total_amount BETWEEN 200 AND 500 THEN 'Medium Value (₹200-500)'
        WHEN total_amount BETWEEN 501 AND 1000 THEN 'High Value (₹501-1000)'
        ELSE 'Premium (> ₹1000)'
    END AS order_category,
    COUNT(order_id) AS order_count,
    CONCAT('₹', FORMAT(SUM(total_amount), 2)) AS total_revenue,
    CONCAT('₹', FORMAT(AVG(total_amount), 2)) AS avg_order_value,
    ROUND(COUNT(order_id) * 100.0 / (SELECT COUNT(*) FROM orders), 2) AS percentage
FROM orders
WHERE order_status = 'Delivered'
GROUP BY order_category
ORDER BY MIN(total_amount);
```

## Q4. Top 10 High-Value Customers
```sql
SELECT 
    c.customer_id,
    c.customer_name,
    c.city,
    c.premium_member,
    COUNT(o.order_id) AS total_orders,
    CONCAT('₹', FORMAT(SUM(o.total_amount), 2)) AS lifetime_value,
    CONCAT('₹', FORMAT(AVG(o.total_amount), 2)) AS avg_order_value,
    MAX(o.order_date) AS last_order_date
FROM customer c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_status = 'Delivered'
GROUP BY c.customer_id, c.customer_name, c.city, c.premium_member
ORDER BY SUM(o.total_amount) DESC
LIMIT 10;
```

## Q5.Orders with Missing or Delayed Delivery Assignment  
```sql
SELECT 
    o.order_id,
    o.order_date,
    c.customer_name,
    r.restaurant_name,
    o.total_amount,
    o.order_status,
    CASE 
        WHEN o.rider_id IS NULL THEN 'No Rider Assigned'
        WHEN o.delivery_time IS NULL AND o.order_status = 'Delivered' THEN 'Missing Delivery Time'
        ELSE 'Assigned but Pending'
    END AS delivery_issue
FROM orders o
JOIN customer c ON o.customer_id = c.customer_id
JOIN restaurants r ON o.restaurant_id = r.restaurant_id
WHERE o.rider_id IS NULL 
   OR (o.delivery_time IS NULL AND o.order_status = 'Delivered')
ORDER BY o.order_date DESC;
```

## Q6. Restaurant Revenue Ranking with Performance Metrics 
```sql
SELECT 
    r.restaurant_id,
    r.restaurant_name,
    r.cuisine_type,
    r.city,
    COUNT(o.order_id) AS total_orders,
    CONCAT('₹', FORMAT(SUM(o.total_amount), 2)) AS total_revenue,
    CONCAT('₹', FORMAT(SUM(o.total_amount * r.commission_rate / 100), 2)) AS commission_earned,
    CONCAT('₹', FORMAT(AVG(o.total_amount), 2)) AS avg_order_value,
    ROUND(AVG(r.rating), 2) AS avg_rating,
    RANK() OVER (ORDER BY SUM(o.total_amount) DESC) AS revenue_rank
FROM restaurants r
LEFT JOIN orders o ON r.restaurant_id = o.restaurant_id 
    AND o.order_status = 'Delivered'
WHERE r.is_active = 'TRUE'
GROUP BY r.restaurant_id, r.restaurant_name, r.cuisine_type, r.city, r.commission_rate
ORDER BY revenue_rank DESC;
```

## Q7. Most Popular Dish by City
```sql
WITH CityDishPopularity AS (
    SELECT 
        r.city,
        mi.item_name,
        SUM(od.quantity) AS total_quantity_sold,
        ROW_NUMBER() OVER (PARTITION BY r.city ORDER BY SUM(od.quantity) DESC) AS rank_in_city
    FROM order_details od
    JOIN orders o ON od.order_id = o.order_id
    JOIN menu_items mi ON od.item_id = mi.item_id
    JOIN restaurants r ON o.restaurant_id = r.restaurant_id
    GROUP BY r.city, mi.item_name
)
SELECT 
    city,
    item_name AS most_popular_dish,
    total_quantity_sold
FROM CityDishPopularity
WHERE rank_in_city = 1
ORDER BY city;
```

## Q8. Customer Churn Analysis (Inactive Customers)
```sql
SELECT 
    CASE 
        WHEN days_dormant BETWEEN 300 AND 450 THEN 'VIP Dormant'
        WHEN days_dormant BETWEEN 451 AND 600 THEN 'Mid-Value Dormant'
        WHEN days_dormant BETWEEN 601 AND 750 THEN 'Low-Value Dormant'
        ELSE 'Permanently Churned'
    END AS customer_segment,
    
    COUNT(*) AS customer_count,
    
    ROUND(100.0 * COUNT(*) / (
        SELECT COUNT(DISTINCT customer_id) FROM (
            SELECT c.customer_id, DATEDIFF(CURDATE(), MAX(o.order_date)) AS days_dormant
            FROM customer c
            LEFT JOIN orders o ON c.customer_id = o.customer_id
            GROUP BY c.customer_id
            HAVING DATEDIFF(CURDATE(), MAX(o.order_date)) >= 300
        ) AS temp
    ), 2) AS percentage_of_dormant,
    
    ROUND(AVG(days_dormant), 0) AS avg_days_dormant,
    MIN(days_dormant) AS min_days_dormant,
    MAX(days_dormant) AS max_days_dormant,
    SUM(total_orders) AS total_orders_segment,
    ROUND(AVG(total_orders), 2) AS avg_orders_per_customer

FROM (
    SELECT 
        c.customer_id,
        DATEDIFF(CURDATE(), MAX(o.order_date)) AS days_dormant,
        COUNT(DISTINCT o.order_id) AS total_orders
    FROM customer c
    LEFT JOIN orders o ON c.customer_id = o.customer_id
    GROUP BY c.customer_id
    HAVING DATEDIFF(CURDATE(), MAX(o.order_date)) >= 300
) AS subquery

GROUP BY customer_segment
ORDER BY 
    CASE 
        WHEN customer_segment = 'VIP Dormant' THEN 1
        WHEN customer_segment = 'Mid-Value Dormant' THEN 2
        WHEN customer_segment = 'Low-Value Dormant' THEN 3
        ELSE 4
    END;
```

## Q9. Day of Week Analysis for Demand Forecasting  
```sql
SELECT 
    DAYNAME(order_date) AS day_of_week,
    DAYOFWEEK(order_date) AS day_number,
    COUNT(order_id) AS total_orders,
    CONCAT('₹', FORMAT(SUM(total_amount), 2)) AS total_revenue,
    CONCAT('₹', FORMAT(AVG(total_amount), 2)) AS avg_order_value,
    COUNT(DISTINCT customer_id) AS unique_customers,
    COUNT(DISTINCT restaurant_id) AS active_restaurants,
    ROUND(AVG(delivery_time), 2) AS avg_delivery_time,
    ROUND(AVG(rating), 2) AS avg_rating,
    ROUND(COUNT(order_id) * 100.0 / SUM(COUNT(order_id)) OVER(), 2) AS order_percentage
FROM orders
WHERE order_status = 'Delivered'
GROUP BY DAYNAME(order_date), DAYOFWEEK(order_date)
ORDER BY total_orders DESC;
```

## Q10. Delivery Efficiency Analysis by Vehicle Type and City
```sql
SELECT 
    r.city AS rider_city,
    r.vehicle_type,
    COUNT(o.order_id) AS total_deliveries,
    ROUND(AVG(o.delivery_time_mins), 2) AS avg_delivery_time_mins,
    MIN(o.delivery_time_mins) AS min_delivery_time_mins,
    MAX(o.delivery_time_mins) AS max_delivery_time_mins
FROM orders o
JOIN riders r 
    ON o.rider_id = r.rider_id
WHERE o.delivery_time_mins IS NOT NULL
GROUP BY r.city, r.vehicle_type
ORDER BY avg_delivery_time_mins ASC;
```

## Q11. Monthly Restaurant Growth Rate 
```sql
WITH MonthlyRevenue AS (
    SELECT 
        r.restaurant_id,
        r.restaurant_name,
        DATE_FORMAT(o.order_date, '%Y-%m') AS month,
        SUM(o.total_amount) AS monthly_revenue,
        COUNT(o.order_id) AS monthly_orders
    FROM restaurants r
    JOIN orders o ON r.restaurant_id = o.restaurant_id
    WHERE o.order_status = 'Delivered'
    GROUP BY r.restaurant_id, r.restaurant_name, DATE_FORMAT(o.order_date, '%Y-%m')
),
GrowthCalculation AS (
    SELECT 
        restaurant_id,
        restaurant_name,
        month,
        monthly_revenue,
        monthly_orders,
        LAG(monthly_revenue) OVER (PARTITION BY restaurant_id ORDER BY month) AS prev_month_revenue,
        LAG(monthly_orders) OVER (PARTITION BY restaurant_id ORDER BY month) AS prev_month_orders
    FROM MonthlyRevenue
)
SELECT 
    restaurant_name,
    month,
    CONCAT('₹', FORMAT(monthly_revenue, 2)) AS monthly_revenue,
    monthly_orders,
    CONCAT('₹', FORMAT(prev_month_revenue, 2)) AS prev_month_revenue,
    CASE 
        WHEN prev_month_revenue IS NULL THEN 'N/A'
        ELSE CONCAT(ROUND((monthly_revenue - prev_month_revenue) * 100.0 / prev_month_revenue, 2), '%')
    END AS growth_rate,
    CASE 
        WHEN prev_month_revenue IS NULL THEN 'First Month'
        WHEN monthly_revenue > prev_month_revenue THEN 'Growth'
        WHEN monthly_revenue < prev_month_revenue THEN 'Decline'
        ELSE 'Stable'
    END AS trend
FROM GrowthCalculation
ORDER BY restaurant_name, month DESC;
```

## Q12. Customer Segmentation by Behavior 
```sql
SELECT 
    CASE 
        WHEN total_orders >= 20 AND avg_order_value >= 500 THEN 'VIP Customers'
        WHEN total_orders >= 10 AND avg_order_value >= 300 THEN 'Loyal Customers'
        WHEN total_orders >= 5 THEN 'Regular Customers'
        WHEN total_orders >= 2 THEN 'Occasional Customers'
        ELSE 'New Customers'
    END AS customer_segment,
    COUNT(DISTINCT customer_id) AS customer_count,
    CONCAT('₹', FORMAT(SUM(total_spent), 2)) AS total_revenue,
    CONCAT('₹', FORMAT(AVG(total_spent), 2)) AS avg_customer_value,
    ROUND(AVG(total_orders), 2) AS avg_orders_per_customer
FROM (
    SELECT 
        c.customer_id,
        COUNT(o.order_id) AS total_orders,
        AVG(o.total_amount) AS avg_order_value,
        SUM(o.total_amount) AS total_spent
    FROM customer c
    LEFT JOIN orders o ON c.customer_id = o.customer_id 
        AND o.order_status = 'Delivered'
    GROUP BY c.customer_id
) AS customer_stats
GROUP BY customer_segment
ORDER BY total_revenue DESC;
```

## Q13. Rider Monthly Earnings (10% Commission)  
```sql
SELECT 
    r.rider_id,
    r.rider_name,
    r.city,
    r.vehicle_type,
    DATE_FORMAT(o.order_date, '%Y-%m') AS month,
    COUNT(o.order_id) AS deliveries_completed,
    CONCAT('₹', FORMAT(SUM(o.total_amount), 2)) AS total_order_value,
    CONCAT('₹', FORMAT(SUM(o.total_amount * 0.10), 2)) AS rider_earnings,
    CONCAT('₹', FORMAT(SUM(o.delivery_fee), 2)) AS delivery_fees_collected,
    ROUND(AVG(o.delivery_time), 2) AS avg_delivery_time
FROM riders r
JOIN orders o ON r.rider_id = o.rider_id
WHERE o.order_status = 'Delivered'
GROUP BY r.rider_id, r.rider_name, r.city, r.vehicle_type, DATE_FORMAT(o.order_date, '%Y-%m')
ORDER BY month DESC, rider_earnings DESC;
```

## Q14. Rider Performance Rating Distribution 
```sql
SELECT 
    r.rider_id,
    r.rider_name,
    r.city,
    COUNT(CASE WHEN o.delivery_time < 30 THEN 1 END) AS five_star_deliveries,
    COUNT(CASE WHEN o.delivery_time BETWEEN 30 AND 45 THEN 1 END) AS four_star_deliveries,
    COUNT(CASE WHEN o.delivery_time > 45 THEN 1 END) AS three_star_deliveries,
    COUNT(o.order_id) AS total_deliveries,
    ROUND(
        (COUNT(CASE WHEN o.delivery_time < 30 THEN 1 END) * 5 +
         COUNT(CASE WHEN o.delivery_time BETWEEN 30 AND 45 THEN 1 END) * 4 +
         COUNT(CASE WHEN o.delivery_time > 45 THEN 1 END) * 3) * 1.0 / COUNT(o.order_id), 
    2) AS performance_score,
    ROUND(AVG(o.delivery_time), 2) AS avg_delivery_time
FROM riders r
JOIN orders o ON r.rider_id = o.rider_id
WHERE o.order_status = 'Delivered' 
    AND o.delivery_time IS NOT NULL
GROUP BY r.rider_id, r.rider_name, r.city
ORDER BY performance_score DESC;
```

## Q15. Weekly Order Pattern Analysis by Restaurant
```sql
WITH DailyOrders AS (
    SELECT 
        r.restaurant_id,
        r.restaurant_name,
        DAYNAME(o.order_date) AS day_of_week,
        DAYOFWEEK(o.order_date) AS day_number,
        COUNT(o.order_id) AS order_count,
        SUM(o.total_amount) AS daily_revenue
    FROM restaurants r
    JOIN orders o ON r.restaurant_id = o.restaurant_id
    WHERE o.order_status = 'Delivered'
    GROUP BY r.restaurant_id, r.restaurant_name, DAYNAME(o.order_date), DAYOFWEEK(o.order_date)
),
RankedDays AS (
    SELECT 
        *,
        RANK() OVER (PARTITION BY restaurant_id ORDER BY order_count DESC) AS popularity_rank
    FROM DailyOrders
)
SELECT 
    restaurant_name,
    day_of_week AS peak_day,
    order_count AS peak_day_orders,
    CONCAT('₹', FORMAT(daily_revenue, 2)) AS peak_day_revenue
FROM RankedDays
WHERE popularity_rank = 1
ORDER BY order_count DESC;
```

## Q16.  Customer Lifetime Value (CLV) Analysis 
```sql
SELECT 
    c.customer_id,
    c.customer_name,
    c.email,
    c.city,
    c.premium_member,
    c.registration_date,
    COUNT(o.order_id) AS total_orders,
    CONCAT('₹', FORMAT(SUM(o.total_amount), 2)) AS lifetime_value,
    CONCAT('₹', FORMAT(AVG(o.total_amount), 2)) AS avg_order_value,
    MIN(o.order_date) AS first_order_date,
    MAX(o.order_date) AS last_order_date,
    DATEDIFF(MAX(o.order_date), MIN(o.order_date)) AS customer_lifespan_days,
    ROUND(COUNT(o.order_id) * 1.0 / NULLIF(DATEDIFF(MAX(o.order_date), MIN(o.order_date)), 0) * 30, 2) AS avg_orders_per_month
FROM customer c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_status = 'Delivered'
GROUP BY c.customer_id, c.customer_name, c.email, c.city, c.premium_member, c.registration_date
ORDER BY SUM(o.total_amount) DESC;
```

## Q17. Month-over-Month Sales Trend Analysis
```sql
WITH MonthlySales AS (
    SELECT 
        DATE_FORMAT(order_date, '%Y-%m') AS month,
        COUNT(order_id) AS total_orders,
        SUM(total_amount) AS total_sales,
        AVG(total_amount) AS avg_order_value
    FROM orders
    WHERE order_status = 'Delivered'
    GROUP BY DATE_FORMAT(order_date, '%Y-%m')
),
SalesWithPrevious AS (
    SELECT 
        month,
        total_orders,
        total_sales,
        avg_order_value,
        LAG(total_sales) OVER (ORDER BY month) AS prev_month_sales,
        LAG(total_orders) OVER (ORDER BY month) AS prev_month_orders
    FROM MonthlySales
)
SELECT 
    month,
    total_orders,
    CONCAT('₹', FORMAT(total_sales, 2)) AS monthly_sales,
    CONCAT('₹', FORMAT(avg_order_value, 2)) AS avg_order_value,
    CONCAT('₹', FORMAT(prev_month_sales, 2)) AS prev_month_sales,
    CASE 
        WHEN prev_month_sales IS NULL THEN 'N/A'
        ELSE CONCAT(ROUND((total_sales - prev_month_sales) * 100.0 / prev_month_sales, 2), '%')
    END AS sales_growth,
    CASE 
        WHEN prev_month_orders IS NULL THEN 'N/A'
        ELSE CONCAT(ROUND((total_orders - prev_month_orders) * 100.0 / prev_month_orders, 2), '%')
    END AS order_growth,
    CASE 
        WHEN prev_month_sales IS NULL THEN 'First Month'
        WHEN total_sales > prev_month_sales * 1.1 THEN 'Strong Growth'
        WHEN total_sales > prev_month_sales THEN 'Growth'
        WHEN total_sales < prev_month_sales * 0.9 THEN 'Significant Decline'
        WHEN total_sales < prev_month_sales THEN 'Decline'
        ELSE 'Stable'
    END AS trend
FROM SalesWithPrevious
ORDER BY month DESC;
```

## Q18. Rider Efficiency Comparison  
```sql
WITH RiderStats AS (
    SELECT 
        r.rider_id,
        r.rider_name,
        r.city,
        r.vehicle_type,
        COUNT(o.order_id) AS total_deliveries,
        ROUND(AVG(o.delivery_time_mins), 2) AS avg_delivery_time,
        MIN(o.delivery_time_mins) AS best_time,
        MAX(o.delivery_time_mins) AS worst_time,
        ROUND(STDDEV(o.delivery_time_mins), 2) AS time_consistency
        FROM riders r
    JOIN orders o ON r.rider_id = o.rider_id
    WHERE o.order_status = 'Delivered' 
        AND o.delivery_time IS NOT NULL
        AND r.is_active = 'True'
    GROUP BY r.rider_id, r.rider_name, r.city, r.vehicle_type
)
SELECT 
    rider_name,
    city,
    vehicle_type,
    total_deliveries,
    avg_delivery_time,
    best_time,
    worst_time,
    time_consistency,
    CASE 
        WHEN avg_delivery_time< 25 THEN 'Excellent'
        WHEN avg_delivery_time< 35 THEN 'Good'
        WHEN avg_delivery_time< 45 THEN 'Average'
        ELSE 'Needs Improvement'
    END AS efficiency_rating,
    RANK() OVER (ORDER BY avg_delivery_time ASC) AS efficiency_rank
FROM RiderStats
ORDER BY avg_delivery_time ASC;
```

## Q19. Menu Item Popularity Trend Over Time 
```sql
SELECT 
    mi.item_name,
    DATE_FORMAT(o.order_date, '%Y-%m') AS order_month,
    SUM(od.quantity) AS total_quantity_ordered
FROM 
    order_details od
JOIN 
    orders o ON od.order_id = o.order_id
JOIN 
    menu_items mi ON od.item_id = mi.item_id
GROUP BY 
    mi.item_name,
    order_month
ORDER BY 
    mi.item_name,
    order_month;
```

## Q20. City-wise Revenue Ranking (Last Year - 2023)
```sql
select city, sum(total_amount) as Total_revenue,
dense_rank() over( order by sum(total_amount) desc) as rnk
from restaurants r left join orders o
on r.Restaurant_id=o.Restaurant_id
group by city
having total_revenue is not null;
```

## 🧰 Stored Procedures  

### 1️⃣ Customer Complete Order History
Fetch all customer history.  
```sql
DELIMITER //

CREATE PROCEDURE GetCustomerOrderHistory(IN cust_id INT)
BEGIN
    SELECT 
        c.customer_name,
        o.order_id,
        r.restaurant_name,
        o.order_date,
        CONCAT('₹', FORMAT(o.total_amount, 2)) AS total_amount,
        o.order_status,
        o.payment_method,
        CONCAT('₹', FORMAT(o.delivery_fee, 2)) AS delivery_fee,
        o.rating,
        ri.rider_name,
        o.delivery_time
    FROM customer c
    JOIN orders o ON c.customer_id = o.customer_id
    JOIN restaurants r ON o.restaurant_id = r.restaurant_id
    LEFT JOIN riders ri ON o.rider_id = ri.rider_id
    WHERE c.customer_id = cust_id
    ORDER BY o.order_date DESC;
END //

DELIMITER
CALL GetCustomerOrderHistory(1);
```
### 2️⃣ Restaurant Performance Dashboard

Fetch detailed performance metrics for a specific restaurant within a given date range.
```sql
DELIMITER //

CREATE PROCEDURE GetRestaurantPerformance(IN rest_id INT, IN start_date DATE, IN end_date DATE)
BEGIN
    SELECT 
        r.restaurant_name,
        r.cuisine_type,
        r.city,
        COUNT(o.order_id) AS total_orders,
        CONCAT('₹', FORMAT(SUM(o.total_amount), 2)) AS total_revenue,
        CONCAT('₹', FORMAT(AVG(o.total_amount), 2)) AS avg_order_value,
        ROUND(AVG(o.rating), 2) AS avg_rating,
        COUNT(CASE WHEN o.order_status = 'Delivered' THEN 1 END) AS successful_orders,
        COUNT(CASE WHEN o.order_status = 'Cancelled' THEN 1 END) AS cancelled_orders,
        ROUND(
            COUNT(CASE WHEN o.order_status = 'Cancelled' THEN 1 END) 
            * 100.0 / COUNT(o.order_id), 2
        ) AS cancellation_rate,
        COUNT(DISTINCT o.customer_id) AS unique_customers
    FROM restaurants r
    LEFT JOIN orders o 
        ON r.restaurant_id = o.restaurant_id
        AND o.order_date BETWEEN start_date AND end_date
    WHERE r.restaurant_id = rest_id
    GROUP BY 
        r.restaurant_id, 
        r.restaurant_name, 
        r.cuisine_type, 
        r.city;
END //

DELIMITER ;

CALL GetRestaurantPerformance(1, '2024-01-01', '2024-12-31');
```
---

## 🧠 Key Insights  

- **🌙 Late-Night Dominance:** ₹8.06 Cr revenue from 11,481 orders (00:00–05:00). Premium AOV of ₹702 shows high-value night customers.  
- **💰 High-Value Orders Engine:** ₹501–₹1000 range contributes ₹17.29 Cr, making it the largest revenue segment.  
- **👑 VIP Churn Crisis:** 4,256 dormant VIP users (85% of inactive base) → urgent re-engagement needed.  
- **🏙️ City Performance:** Mumbai leads with ₹4.78 Cr, followed by Kolkata at ₹4.66 Cr.  
- **🍟 Universal Bestseller:** French Fries top in 5 of 6 cities — cross-sell opportunity with premium combos.  
- **📅 Weekend Spending Premium:** Sunday peaks with 6,711 orders; Friday shows lowest demand.  
- **💎 Loyal Customers:** 2,188 loyal users generate ₹1.82 Cr with ~12 orders each — the most valuable customer cohort.  

---

## 🚀 Actionable Recommendations  

| Focus Area | Recommendations |
|-------------|----------------|
| **Customer Retention** | Launch VIP loyalty tier; re-target dormant users with ₹500–₹1000 offers. |
| **Monetization** | Introduce “Night Owl Premium” plan with 24/7 guaranteed delivery (₹20–₹30 fee). |
| **Demand Smoothing** | Run “Friday Feast” discount campaigns and apply surge pricing on weekends. |
| **Menu Strategy** | Bundle “Burger + Fries + Drink” to boost AOV and centralize suppliers for 12–15% savings. |
| **Geographic Expansion** | Scale “Local Legends” partnerships in Hyderabad and expand user base in Pune. |
| **Operational Excellence** | Monitor restaurants with >2 months decline using an early-warning system. |

---
## 🛠️ Tools & Technologies  

| Category | Tools Used |
|-----------|------------|
| Database | MySQL |
| Visualization | Power BI, PowerPoint |
| Documentation | GitHub |
| Techniques | CTEs, Window Functions, Indexing, Stored Procedures |

---
## Notice
All customer names and data used in this project are computer-generated using AI and random functions. They do not represent real data associated with Swiggy or any other entity. This project is solely for learning and educational purposes, and any resemblance to actual persons, businesses, or events is purely coincidental.

