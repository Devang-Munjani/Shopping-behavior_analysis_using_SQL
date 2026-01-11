## Overview

This project analyzes customer shopping data to understand how people buy products, how much they spend, and what factors influence their behavior. Using SQL and PostgreSQL, the data was explored to identify revenue patterns, customer loyalty, and the impact of discounts, subscriptions, and product choices. The goal is to help a business make better decisions about marketing, pricing, and product strategy.


## Objectives

- To identify which customer groups contribute the most to revenue.
- To understand how subscriptions and discounts affect customer spending and repeat purchases.
- To find the best-performing product categories and seasonal trends.
- To analyze payment methods, customer frequency, and locations to discover growth opportunities.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Shopping Dataset](https://www.kaggle.com/datasets/nalisha/shopping-behaviour-and-product-ranking-dateset)


## Schema

```sql
CREATE TABLE shopping(
	Customer_ID	INT PRIMARY KEY,
	Age	INT,
	Gender VARCHAR(10),
	Item_Purchased VARCHAR(100),
	Category VARCHAR(50),
	Purchase_Amount_USD  NUMERIC(10,2),
	Shop_Location VARCHAR(100),
	Product_Size VARCHAR(10),
	Color VARCHAR(30),
	Season VARCHAR(20),
	Review_Rating NUMERIC(2,1) CHECK (Review_Rating BETWEEN 0 AND 5),
	Subscription_Status BOOLEAN,
	Discount_Applied BOOLEAN,
	Previous_Purchases INT,	
	Payment_Method VARCHAR(30),
	Frequency_of_Purchases VARCHAR(30)
)
```

## Business Problems and Solutions


### Task 1: Identify which customer age groups contribute the most to total purchase amount and evaluate whether spending behavior changes significantly across age brackets.

```sql
SELECT
    age_group,
    SUM(purchase_amount_usd) AS total_purchase_amount,
    SUM(previous_purchases) AS previous_purchases_count,
    AVG(purchase_amount_usd) AS avg_revenue_per_customer,
    AVG(previous_purchases) AS avg_previous_purchases_per_customer
FROM (
    SELECT
        customer_id,
        purchase_amount_usd,
        previous_purchases,
        CASE
            WHEN age <= 20 THEN 'ADULT'
            WHEN age <= 40 THEN 'YOUNG'
            ELSE 'OLD'
        END AS age_group
    FROM shopping
) t
GROUP BY age_group
ORDER BY total_purchase_amount DESC;
````

### Older customers bring the most total money, but younger and older customers spend almost the same per person. The reason older customers bring more money is because there are more of them, not because they spend more.


### Task 2:
###  Analyze whether subscription status has any meaningful impact on:
### - Purchase amount
### - Purchase frequency
### - Previous purchase history

```sql
SELECT
    subscription_status,
    frequency_of_purchases,
    COUNT(customer_id) AS total_customers,
    SUM(total_spent) AS total_revenue,
    AVG(total_spent) AS avg_revenue_per_customer,
    AVG(previous_purchases) AS avg_previous_purchases
FROM (
    SELECT
        customer_id,
        subscription_status,
        frequency_of_purchases,
        SUM(purchase_amount_usd) AS total_spent,
        MAX(previous_purchases) AS previous_purchases
    FROM shopping
    GROUP BY
        customer_id,
        subscription_status,
        frequency_of_purchases
) t
GROUP BY
    subscription_status,
    frequency_of_purchases
ORDER BY
    subscription_status,
    total_revenue DESC;
```


### People with a subscription do not spend more or buy more often than people without one. There are just fewer subscribers. So the subscription plan is not really changing how people shop.




### Task 3: Evaluate how discount application influences:
### Purchase amount
### Repeat purchases
### Assess whether discounts are attracting loyal customers or only one-time buyers.


```sql
SELECT
    discount_applied,
    COUNT(customer_id) AS total_customers,
    SUM(total_spent) AS total_revenue,
    AVG(total_spent) AS avg_revenue_per_customer,
    AVG(previous_purchases) AS avg_previous_purchases
FROM (
    SELECT
        customer_id,
        discount_applied,
        SUM(purchase_amount_usd) AS total_spent,
        MAX(previous_purchases) AS previous_purchases
    FROM shopping
    GROUP BY customer_id, discount_applied
) t
GROUP BY discount_applied;
````

### People who use discounts do not spend more money. They only buy slightly more often. So discounts are mostly used by customers who already like the brand, not to bring in new big buyers.



### Task: 4 Revenue by Category & Season
### Find which product categories generate the most revenue and whether this changes by season.

```sql
select category, season , sum(purchase_amount_usd) as total_revenue
from shopping
group by category, season ;
```

### Clothing makes the most money in every season. Accessories are second, footwear is average, and outerwear brings the least. The main product focus should stay on clothing all year.


###Task 5 — Payment Method & Spending Identify: 
   ###       The most commonly used payment methods Whether some payment methods are linked to higher spending


```sql
SELECT
    payment_method,
    COUNT(customer_id) AS total_customers,
    SUM(total_spent) AS total_revenue,
    AVG(total_spent) AS avg_revenue_per_customer
FROM (
    SELECT
        customer_id,
        payment_method,
        SUM(purchase_amount_usd) AS total_spent
    FROM shopping
    GROUP BY customer_id, payment_method
) t
GROUP BY payment_method
ORDER BY total_revenue DESC;
```

### People use PayPal, Credit Card, and Cash the most. Customers who pay by card spend slightly more than others, so card payments should be made easy at checkout.



### Task 6 — Customer Purchase Frequency Segments

### Group customers into behavior types such as:
### Frequent buyers
### Occasional buyers
### Based on how often they shop and how much they spend.

```sql
SELECT
    frequency_of_purchases,
    COUNT(customer_id) AS total_customers,
    SUM(total_spent) AS total_revenue,
    AVG(total_spent) AS avg_revenue_per_customer,
    AVG(previous_purchases) AS avg_previous_purchases
FROM (
    SELECT
        customer_id,
        frequency_of_purchases,
        SUM(purchase_amount_usd) AS total_spent,
        MAX(previous_purchases) AS previous_purchases
    FROM shopping
    GROUP BY customer_id, frequency_of_purchases
) t
GROUP BY frequency_of_purchases
ORDER BY total_revenue DESC;
```

### Weekly, fortnightly, and monthly buyers are the most valuable customers. People who buy only once a year bring much less value. The business should try to move yearly buyers into more regular buyers.




### Task 7 — Ratings & Product Strategy

### Check whether review ratings are higher for:
### Certain categories
### Certain sizes
### Certain seasons

```sql
SELECT 'Category' AS type, category AS item, AVG(review_rating) AS avg_rating
FROM shopping
GROUP BY category
ORDER BY type, avg_rating DESC;


SELECT 'Size' AS type, product_size AS item, AVG(review_rating) AS avg_rating
FROM shopping
GROUP BY product_size
ORDER BY type, avg_rating DESC;


SELECT 'Season' AS type, season AS item, AVG(review_rating) AS avg_rating
FROM shopping
GROUP BY season
ORDER BY type, avg_rating DESC;
```


### Task 8 — Location-Based Opportunity
### Find locations where:
### Customers spend a lot
### But do not shop often

```sql
SELECT
    shop_location,
    COUNT(customer_id) AS total_customers,
    AVG(total_spent) AS avg_spend_per_customer,
    AVG(previous_purchases) AS avg_previous_purchases
FROM (
    SELECT
        customer_id,
        shop_location,
        SUM(purchase_amount_usd) AS total_spent,
        MAX(previous_purchases) AS previous_purchases
    FROM shopping
    GROUP BY customer_id, shop_location
) t
GROUP BY shop_location
HAVING
    AVG(total_spent) > (SELECT AVG(purchase_amount_usd) FROM shopping)
    AND
    AVG(previous_purchases) < (SELECT AVG(previous_purchases) FROM shopping)
ORDER BY avg_spend_per_customer DESC;
```
#### Some locations have customers who spend a lot when they shop but do not come back often. These places are good targets for loyalty programs and marketing to make customers buy more often.
