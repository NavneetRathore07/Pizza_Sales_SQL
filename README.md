-- Q1. Retrieve the total number of orders placed.
SELECT 
    *
FROM
    orders;
SELECT 
    COUNT(*) AS Total_orders
FROM
    orders;
-- Q2. Calculate the total revenue generated from pizza sales.
SELECT 
    ROUND(SUM(orders_details.quantity * pizzas.price),
            2) AS Total_sales
FROM
    orders_details
        JOIN
    pizzas ON orders_details.pizaa_id = pizzas.pizza_id;
    
-- Q3. Identify the highest-priced pizza.

SELECT 
    p.pizza_id, pt.name, pt.category, p.price
FROM
    pizzas AS p
        JOIN
    pizza_types AS pt ON p.pizza_type_id = pt.pizza_type_id
ORDER BY p.price DESC
LIMIT 1;

-- Q4. Identify the most common pizza size ordered.

SELECT 
    p.size, SUM(od.quantity) AS Total_orders
FROM
    pizzas AS p
        JOIN
    orders_details AS od ON p.pizza_id = od.pizaa_id
GROUP BY p.size
ORDER BY Total_orders DESC;

-- Q5. List the top 5 most ordered pizza types along with their quantities.

SELECT 
    pt.name, SUM(od.quantity) AS Total_quantity_ordered
FROM
    pizza_types AS pt
        JOIN
    pizzas AS p ON pt.pizza_type_id = p.pizza_type_id
        JOIN
    orders_details AS od ON p.pizza_id = od.pizaa_id
GROUP BY pt.name
ORDER BY Total_quantity_ordered DESC
LIMIT 5;

-- Q6. Join the necessary tables to find the total quantity of each pizza category ordered.
SELECT 
    pt.category, SUM(od.quantity) AS Total_quantity_ordered
FROM
    pizza_types AS pt
        JOIN
    pizzas AS p ON pt.pizza_type_id = p.pizza_type_id
        JOIN
    orders_details AS od ON p.pizza_id = od.pizaa_id
GROUP BY pt.category
ORDER BY Total_quantity_ordered DESC;

-- Q7. Determine the distribution of orders by hour of the day.

SELECT 
    HOUR(order_time) AS hour,
    COUNT(order_id) AS number_of_orders
FROM
    orders
GROUP BY hour;

-- Q8. find the category-wise distribution of pizzas.

SELECT 
    category, COUNT(name) AS pizza_count
FROM
    pizza_types
GROUP BY category;

-- Q9. Group the orders by date and calculate the average number of pizzas ordered per day.

SELECT 
    ROUND(AVG(quantity_ordered), 2) AS avg_order_count
FROM
    (SELECT 
        o.order_date AS Day, SUM(od.quantity) AS quantity_ordered
    FROM
        orders AS o
    JOIN orders_details AS od ON o.order_id = od.order_id
    GROUP BY o.order_date) AS data_for_avg;

-- Q10. Determine the top 3 most ordered pizza types based on revenue

SELECT 
    pt.name, ROUND(SUM(p.price * od.quantity), 2) AS price
FROM
    orders_details AS od
        JOIN
    pizzas AS p ON od.pizaa_id = p.pizza_id
        JOIN
    pizza_types AS pt ON pt.pizza_type_id = p.pizza_type_id
GROUP BY pt.name
ORDER BY price DESC
LIMIT 3;

-- Q11. Calculate the percentage contribution of each pizza category to total revenue.

SELECT 
    pt.category,
    ROUND((SUM(p.price * od.quantity) / (SELECT 
                    SUM(pizzas.price * orders_details.quantity)
                FROM
                    orders_details
                        JOIN
                    pizzas ON orders_details.pizaa_id = pizzas.pizza_id)) * 100,
            2) AS Revenue_percentage
FROM
    orders_details AS od
        JOIN
    pizzas AS p ON od.pizaa_id = p.pizza_id
        JOIN
    pizza_types AS pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.category
ORDER BY Revenue_percentage DESC;


-- Q12. Analyze the cumulative revenue generated over time.

select order_date , round(sum(revenue) over(order by order_date),2) as cumulative_revenue
from
(select o.order_date , sum(od.quantity * p.price) as revenue
from orders_details as od join pizzas as p
on od.pizaa_id = p.pizza_id
join orders as o 
on od.order_id = o.order_id
group by o.order_date) as Total_revenue;

-- Q13. Determine the top 3 most ordered pizza types based on revenue for each pizza category.

select * from 
(select pt.category , pt.name , round(sum(od.quantity * p.price),2) as revenue,
rank() over(partition by pt.category order by sum(od.quantity * p.price) desc ) as Rank_
from orders_details as od join pizzas as p
on od.pizaa_id = p.pizza_id
join pizza_types as pt 
on p.pizza_type_id = pt.pizza_type_id
group by pt.category , pt.name) as a 
where Rank_ <= 3;

