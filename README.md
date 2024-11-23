# 1_total_orders_placed.sql
-- Retrieve the total number of orders placed
SELECT COUNT(order_id) AS total_order FROM orders;

2_total_revenue_generated.sql
-- Calculate the total revenue generated from pizza sales
SELECT ROUND(SUM(orders_details.quantity * pizzas.price), 0) AS total_sales 
FROM orders_details 
JOIN pizzas ON orders_details.pizza_id = pizzas.pizza_id;

3_highest_priced_pizza.sql
-- Identify the highest-priced pizza
SELECT pizza_types.name, pizzas.price 
FROM pizza_types 
JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id 
ORDER BY pizzas.price DESC 
LIMIT 1;

4_most_common_pizza_size.sql
-- Identify the most common pizza size ordered
SELECT pizzas.size, COUNT(orders_details.order_details_id) AS order_count 
FROM pizzas 
JOIN orders_details ON pizzas.pizza_id = orders_details.pizza_id 
GROUP BY pizzas.size 
ORDER BY order_count DESC;

5_top_5_most_ordered_pizzas.sql
-- List the top 5 most ordered pizza types along with their quantities
SELECT pizza_types.name, SUM(orders_details.quantity) AS top_5 
FROM pizzas 
JOIN pizza_types ON pizzas.pizza_type_id = pizza_types.pizza_type_id 
JOIN orders_details ON pizzas.pizza_id = orders_details.pizza_id 
GROUP BY pizza_types.name 
ORDER BY top_5 DESC 
LIMIT 5;

6_total_quantity_per_pizza_category.sql
-- Find the total quantity of each pizza category ordered
SELECT pizza_types.category, SUM(orders_details.quantity) AS total_quantity 
FROM orders_details 
JOIN pizzas ON orders_details.pizza_id = pizzas.pizza_id 
JOIN pizza_types ON pizzas.pizza_type_id = pizza_types.pizza_type_id 
GROUP BY pizza_types.category;

7_order_distribution_by_hour.sql
-- Determine the distribution of orders by hour of the day
SELECT HOUR(order_time), COUNT(order_id) AS order_count 
FROM orders 
GROUP BY HOUR(order_time);

8_categorywise_pizza_distribution.sql
-- Find the category-wise distribution of pizzas
SELECT category, COUNT(name) 
FROM pizza_types 
GROUP BY category;

9_avg_pizza_ordered_per_day.sql
-- Group the orders by date and calculate the average number of pizzas ordered per day
SELECT ROUND(AVG(quantity), 0) AS avg_pizza_ordered_perday 
FROM (
  SELECT orders.order_date, SUM(orders_details.quantity) AS quantity 
  FROM orders 
  JOIN orders_details ON orders.order_id = orders_details.order_id 
  GROUP BY orders.order_date
) AS order_count;

10_top_3_most_ordered_pizza_types_by_revenue.sql
-- Determine the top 3 most ordered pizza types based on revenue
SELECT pizza_types.name, SUM(orders_details.quantity * pizzas.price) AS revenue 
FROM pizza_types 
JOIN pizzas ON pizzas.pizza_type_id = pizza_types.pizza_type_id 
JOIN orders_details ON orders_details.pizza_id = pizzas.pizza_id 
GROUP BY pizza_types.name 
ORDER BY revenue DESC 
LIMIT 3;

11_percentage_contribution_of_pizza_type_to_revenue.sql
-- Calculate the percentage contribution of each pizza type to total revenue
SELECT pizza_types.category, 
       ROUND(SUM(orders_details.quantity * pizzas.price) / 
             (SELECT ROUND(SUM(orders_details.quantity * pizzas.price), 2) AS total_sales 
              FROM orders_details 
              JOIN pizzas ON pizzas.pizza_id = orders_details.pizza_id) * 100, 0) AS revenue 
FROM pizza_types 
JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id 
JOIN orders_details ON orders_details.pizza_id = pizzas.pizza_id 
GROUP BY pizza_types.category 
ORDER BY revenue DESC;

12_cumulative_revenue_over_time.sql
-- Analyze the cumulative revenue generated over time
SELECT order_date, 
       SUM(revenue) OVER (ORDER BY order_date) AS cum_revenue 
FROM (
  SELECT orders.order_date, 
         SUM(orders_details.quantity * pizzas.price) AS revenue 
  FROM orders_details 
  JOIN pizzas ON orders_details.pizza_id = pizzas.pizza_id 
  JOIN orders ON orders.order_id = orders_details.order_id 
  GROUP BY orders.order_date
) AS sales;

13_top_3_pizza_types_per_category_by_revenue.sql
-- Determine the top 3 most ordered pizza types based on revenue for each pizza category
SELECT name, revenue 
FROM (
  SELECT category, name, revenue, 
         RANK() OVER (PARTITION BY category ORDER BY revenue DESC) AS rk 
  FROM (
    SELECT pizza_types.category, 
           pizza_types.name, 
           SUM(orders_details.quantity * pizzas.price) AS revenue 
    FROM pizza_types 
    JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id 
    JOIN orders_details ON orders_details.pizza_id = pizzas.pizza_id 
    GROUP BY pizza_types.category, pizza_types.name
  ) AS a
) b 
WHERE rk <= 3;
