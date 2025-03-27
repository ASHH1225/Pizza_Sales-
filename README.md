# Pizza_Sales-
This project analyzes pizza sales data using SQL queries to answer various key business questions and provide valuable insights.

Tools Used: My SQL

List of Questions :
1. Retrieve the total number of orders placed.
2. Calculate the total revenue generated from pizza sales.
3. Identify the highest-priced pizza.
4. Identify the most common pizza size ordered.
5. List the top 5 most ordered pizza types along with their quantities.
6. Join the necessary tables to find the total quantity of each pizza category ordered.
7. Determine the distribution of orders by hour of the day.
8. Join relevant tables to find the category-wise distribution of pizzas.
9. Group the orders by date and calculate the average number of pizzas ordered per day.
10. Determine the top 3 most ordered pizza types based on revenue.
11. Calculate the percentage contribution of each pizza type to total revenue.
12. Analyze the cumulative revenue generated over time.
13. Determine the top 3 most ordered pizza types based on revenue for each pizza category.

Answers:

#1.Retrieve the total number of orders placed.
SELECT 
    COUNT(order_id) AS total_orders
FROM
    orders;  
            
    
#2.Calculate the total revenue generated from pizza sales.
SELECT 
    ROUND(SUM(order_details.quantity * pizzas.price),
            2) AS total_revenue
FROM
    order_details
        JOIN
    pizzas ON pizzas.pizza_id = order_details.pizza_id;


#3.Identify the highest-priced pizza.
SELECT 
    pizza_types.name, pizzas.price
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
ORDER BY pizzas.price DESC
LIMIT 1; 



#4.Identify the most common pizza size ordered.
SELECT 
    pizzas.size , SUM(pizzas.price*order_details.quantity) AS Sales_Amount , 
    COUNT(order_details.order_details_id) AS order_count
FROM
    pizzas
        JOIN
    order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizzas.size
ORDER BY order_count DESC
LIMIT 1;



#5.List the top 5 most ordered pizza types along with their quantities.
#Take 3 tables, first 2 then again use JOIN stmt and then the third. 
SELECT 
    pizza_types.name,
    SUM(order_details.quantity) AS Pizza_quantity
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizza_types.name
ORDER BY Pizza_quantity DESC
LIMIT 5;

#6. Join the necessary tables to find the total quantity of each pizza category ordered.
SELECT 
    pizza_types.category,
    SUM(order_details.quantity) AS Total_Quantity
FROM
    pizzas
        JOIN
    pizza_types ON pizzas.pizza_type_id = pizza_types.pizza_type_id
        JOIN
    order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category
ORDER BY Total_Quantity desc;


#7. Determine the distribution of orders by hour of the day.
SELECT 
    HOUR(order_time), COUNT(order_id) AS Order_count
FROM
    orders
GROUP BY HOUR(order_time);

#8. Join relevant tables to find the category-wise distribution of pizzas.
SELECT Category , COUNT(pizza_types.name) FROM pizza_types
GROUP BY  Category;

#9. Group the orders by date and calculate the average number of pizzas ordered per day.
SELECT 
    ROUND(AVG(Quantity), 0) AS Avg_Pizza_perday
FROM
    (SELECT 
        orders.order_date, SUM(order_details.quantity) AS Quantity
    FROM
        orders
    JOIN order_details ON orders.order_id = order_details.order_id
    GROUP BY orders.order_date) AS Order_quantity;    
    
#10. Determine the top 3 most ordered pizza types based on revenue.
SELECT 
    pizza_types.name,
    SUM(pizzas.price * order_details.quantity) AS revenue
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizza_types.name
ORDER BY revenue desc limit 3;




#11. Calculate the percentage contribution of each pizza type to total revenue.
SELECT pizza_types.category , 
ROUND(SUM(pizzas.price * order_details.quantity)/ (SELECT
    ROUND(SUM(order_details.quantity * pizzas.price),
            2) AS total_revenue
FROM
    order_details
        JOIN
    pizzas ON pizzas.pizza_id = order_details.pizza_id) * 100,2) AS revenue
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON pizzas.pizza_id = order_details.pizza_id
    GROUP BY pizza_types.category ORDER BY revenue desc;


#12. Analyze the cumulative revenue generated over time.
SELECT order_date, SUM(revenue)  OVER(ORDER BY order_date) AS cumulative_revenue
FROM
(SELECT orders.order_date, SUM(order_details.quantity*pizzas.price) AS revenue
FROM
    order_details
        JOIN
    pizzas ON pizzas.pizza_id = order_details.pizza_id
    JOIN orders ON orders.order_id = order_details.order_id
    GROUP BY orders.order_date) AS sales;
    
    
#13. Determine the top 3 most ordered pizza types based on revenue for each pizza category.
SELECT name, revenue FROM
(SELECT category, name, revenue,
rank() OVER(partition by category order by revenue desc) AS rn
FROM
(SELECT pizza_types.category , pizza_types.name , SUM(order_details.quantity*pizzas.price)  AS revenue
FROM
pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON pizzas.pizza_id = order_details.pizza_id
    GROUP BY pizza_types.category , pizza_types.name) AS a) AS b
    WHERE rn<=3;
