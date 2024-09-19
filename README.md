# Danny-Diner-Customers-Behaviour-Analysis--MSSQL-Server
This project is part of a case study that is available [here](https://8weeksqlchallenge.com/case-study-1/). It explores patterns, trends, and factors influencing customer spending to uncover insights into their preferences and purchasing habits. The findings can help identify potential improvements in menu offerings and marketing strategies for a dining establishment.

### Background
Danny seriously loves Japanese food so at the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favorite foods: sushi, curry, and ramen. Danny’s Diner needs your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from its few months of operation but has no idea how to use its data to help it run the business.

### Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent, and also which menu items are their favorite. Having this deeper connection with his customers will help him deliver a better and more personalized experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally, he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

### Entity Relationship Diagram
![image](https://github.com/user-attachments/assets/837bb3b8-52a3-4a2e-898f-b8e506938900)

### Some SQL Skills Applied
- JOINs
- AGGREGATES
- CTEs
- Windows Function

###  Questions
1. What is the total amount each customer spent at the restaurant?
2. How many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
8. What is the total items and amount spent for each member before they became a member?
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
#### Bonus Questions
1. The following questions are related to creating basic data tables that Danny and his team can use to quickly derive insights without needing to join the underlying tables using SQL.
Recreate the following table output using the available data:
2. Danny also requires further information about the ranking of products. he purposely does not need the ranking of non-member purchases so he expects NULL ranking values for customers who are not yet part of the loyalty program.

### Some SQL queries for the above questions

**2. What was the first item from the menu purchased by each customer?**
```sql
WITH customer_first_purc AS (
	SELECT customer_id, MIN(order_date) AS first_purc_date
	FROM sales 
	GROUP BY customer_id
)
SELECT cfp.customer_id, cfp.first_purc_date, m.product_name
FROM customer_first_purc cfp
JOIN sales s ON s.customer_id = cfp.customer_id
AND cfp.first_purc_date = s.order_date
JOIN menu m ON m.product_id = s.product_id;
```

**6. Which item was purchased first by the customer after they became a member?**
```sql
WITH first_purch_after_memship AS(
--first select the first purchase date after the membership
	SELECT s.customer_id, MIN(s.order_date) AS first_purc_date
	FROM sales s
	JOIN members mbr ON s.customer_id = mbr.customer_id
	WHERE s.order_date >= mbr.join_date
	GROUP BY s.customer_id
)
--now select the item 
SELECT fpam.customer_id, m.product_name
FROM first_purch_after_memship fpam
JOIN sales s ON s.customer_id = fpam.customer_id
AND fpam.first_purc_date = s.order_date
JOIN menu m ON s.product_id = m.product_id;
```

**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**
```sql
SELECT s.customer_id, SUM(
	CASE
		WHEN m.product_name = 'sushi' THEN m.price*20
		ELSE m.price*10
		END
		) AS total_points
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY s.customer_id
```

**Bonus Question 2. Danny also requires further information about the ranking of products. he purposely does not need the ranking of non-member purchases so he expects NULL ranking values for customers who are not yet part of the loyalty program.**
```sql
WITH customers_data AS (
SELECT s.customer_id, s.order_date, m.product_name, m.price,
	CASE
		WHEN s.order_date < mb.join_date THEN 'N'
		WHEN s.order_date >= mb.join_date THEN 'Y'
		ELSE 'N'
		END AS member
FROM sales s
JOIN menu m ON s.product_id = m.product_id
LEFT JOIN members mb ON s.customer_id = mb.customer_id
)
SELECT *, 
CASE WHEN member = 'N' THEN NULL
ELSE RANK() OVER(PARTITION BY customer_id, member ORDER BY order_date)
END AS ranking
FROM customers_data
ORDER BY customer_id, order_date;
```

### Insights from the Analysis
- Customer A loves ramen, Customer C loves only ramen whereas Customer B seems to enjoy sushi, curry, and ramen equally.
- Customer B is the most frequent visitor with 6 visits in Jan 2021.
- The most popular item is ramen, followed by curry and sushi at Danny's Diner.
- The last items ordered by Customers A and B before they became members were sushi and curry.
