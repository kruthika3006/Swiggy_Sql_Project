# Swiggy_Sql_Project
# SQL Project: Data Analysis for Swiggy - A Food Delivery Company

## Overview

This project demonstrates my SQL problem-solving skills through the analysis of data for Swiggy, a popular food delivery company in India. The project involves setting up the Database, Importing data, Handling null values, and Solving a variety of business problems using complex SQL queries, Creating Stored Procedures, Query Optimization and generatiing Insights and Actionable recommendation.

## Project Structure

- **Database Setup:** Creation of the `Swiggy_db` database and the required tables.
- **Data Import:** Inserting sample data into the tables.
- **Data Cleaning:** Handling null values and ensuring data integrity.
- **Business Problems:** Solving 20 specific business problems using SQL queries.
- **Stored Procedure:** Creating Stored Procedures
- **Query Optimization:** Optimized the queries using index and best practices

## Database Setup
```sql
CREATE DATABASE Swiggy_db;
```
-- 1. Creating Tables
```sql
create table customers (
    Customer_id int primary key,
    Customer_name varchar(30),
    Reg_date date
);

Create table restaurants (
	Restaurant_id int primary key,
    Restaurant_name varchar(55),
    City varchar(30),
    Opening_hours varchar(55)
);
create table orders(
    Order_id int primary key,
    Customer_id int ,
    Restaurant_id int,
    Order_item varchar(55),
    Order_date date,
    Order_time time,
    Order_status varchar(30),
    Total_amount float,
    foreign key (Customer_id) references customers (Customer_id),
    foreign key (Restaurant_id) references restaurants (restaurant_id)
);

create table riders(
    Rider_id int primary key,
    Rider_name varchar(30),
    Sign_up date
);
create table deliveries(
    delivery_id int primary key,
    order_id int ,
    delivery_status varchar(35),
    delivery_time time,
    rider_id int,
    foreign key (order_id) references orders(order_id),
    foreign key (rider_id) references riders(rider_id)
);
```
## Data Import

## Data Cleaning and Handling Null Values
```sql
select * from customers
where customer_name is null or reg_date is null;

select * from restaurants
where Restaurant_name is null or city is null or Opening_hours is null ;

select * from orders
where customer_id is null or Restaurant_id is null or Order_item is null
or order_date is null  or order_time is null or Order_status is null  or
Total_amount is null  ;

select * from riders
where rider_name  is null or sign_up is null;

select * from deliveries
where order_id  is null or delivery_status is null or delivery_time is null or rider_id is null;
```
## Feature Engineering

set sql_safe_updates=0;

Alter table orders 
add column Time_of_Day varchar(20);

Update orders 
set time_of_day =(
                 case 
                    when order_time between "00:00:00" and "12:00:00" then "Morning"
					when order_time between "12:01:00" and "16:00:00" then "Afternoon"
					else "Evening"
				  end );
                  
Alter table deliveries
add column Time_of_Day varchar(20);

Update deliveries 
set time_of_day =(
                 case 
                    when delivery_time between "00:00:00" and "12:00:00" then "Morning"
					when delivery_time between "12:01:00" and "16:00:00" then "Afternoon"
					else "Evening"
				  end );
      
## Business Problems Solved

### Q1. Top 5 Most Frequently Ordered Dishes
-- Write a query to find the top 5 most frequently ordered dishes by the customer "Arjun Mehta" in the last 2 year.

select customer_name, order_item, count(order_id) as Total_orders
from customers inner join orders
on customers.customer_id=orders.customer_id
where customer_name="Arjun Mehta" and year(Order_date)>=year(current_date())-2
group by customer_name, order_item
order by count(order_id) desc 
limit 5;

### Q2. Popular Time Slots
-- Identify the time slots during which the most orders are placed, based on 2-hour intervals.

select 
     case when hour(order_time) between 0 and 1 then "00:00:00 AM-02:00:00AM"
		  when hour(order_time) between 2 and 3 then "02:00:00 AM-04:00:00 AM"
          when hour(order_time) between 4 and 5 then "04:00:00 AM-06:00:00 AM"
          when hour(order_time) between 6 and 7 then "06:00:00 AM-08:00:00 AM"
		  when hour(order_time) between 8 and 9 then "08:00:00 AM-10:00:00 AM"
          when hour(order_time) between 10 and 11 then "10:00:00 AM-12:00:00 PM"
          when hour(order_time) between 12 and 13 then "12:00:00 PM-02:00:00 PM"
          when hour(order_time) between 14 and 15 then "02:00:00 PM-04:00:00 PM"
          when hour(order_time) between 16 and 17 then "04:00:00 PM-06:00:00 PM"
          when hour(order_time) between 18 and 19 then "06:00:00 PM-08:00:00 PM"
          when hour(order_time) between 20 and 21 then "08:00:00 PM-10:00:00 PM"
          when hour(order_time) between 22 and 23 then "10:00:00 PM-00:00:00 AM"
          end as Time_slot,
	count(order_id) as Total_order
    from orders
    group by time_slot
    order by total_order desc
    limit 1;

    create index idx_customername on customers(customer_name);
    
### Q3. Order Value Analysis
-- Find the average order value (AOV) per customer who has placed more than 750 orders. 
-- Return: customer_name, aov (average order value).

select customers.customer_id,customer_name ,round(avg(total_amount),2) as AOV,count(order_id) as total_orders
from orders inner join customers
on orders.customer_id=customers.customer_id
group by customers.customer_id,customer_name
having total_orders>750;

### Q4. High-Value Customers
-- List the customers who have spent more than 100K in total on food orders. 
-- Return: customer_name, customer_id.

select customer_name,customers.customer_id, sum(total_amount) as Total_order_amt
from orders inner join customers
on orders.customer_id=customers.customer_id
group by customer_name,customers.customer_id
having total_order_amt>100000
order by total_order_amt desc;

### Q5. Orders Without Delivery
-- Write a query to find orders that were placed but not delivered.
-- Return: restaurant_name, city, and the number of not delivered orders.

select restaurants.restaurant_id,restaurant_name,city , sum(case when delivery_status="not delivered" then 1 else 0 end ) as Not_delivered_orders
from orders inner join restaurants
on orders.restaurant_id=restaurants.restaurant_id
inner join deliveries
on deliveries.order_id=orders.order_id
where order_status = "Completed" and delivery_status="Not Delivered"
group by restaurants.restaurant_id,restaurant_name,city
order by not_delivered_orders desc;

create index idx_orderstatus on orders(order_status);
create index idx_deliverystatus on deliveries(delivery_status);

### Q6. Restaurant Revenue Ranking
-- Rank restaurants by their total revenue from the last 2 year.
-- Return: restaurant_name, total_revenue, and their rank within their city.

select restaurants.Restaurant_id, Restaurant_name,city, sum(total_amount) as revenue,
dense_rank() over( partition by city order by sum(total_amount)  desc ) as rnk
from orders inner join restaurants 
on orders.restaurant_id = restaurants.restaurant_id
where year(order_date)=year(current_date())-2
group by restaurants.Restaurant_id,restaurant_name,city;

create index idx_orderdate on orders(order_date);
 
### Q7. Most Popular Dish by City
-- Identify the most popular dish in each city based on the number of orders.
 
 with cte as (
 select city,order_item, count(order_id) as No_of_orders,
 dense_rank() over(partition by city order by count(order_id) desc) as rnk
 from restaurants inner join orders
 on restaurants.restaurant_id=orders.restaurant_id
 group by city,order_item)
 select cte.* from cte 
 where rnk=1;
 
### Q8. Customer Churn
-- Find customers who haven’t placed an order in 2024 but did in 2023.

select customers.customer_id,customer_name 
from customers left join orders
on customers.customer_id=orders.customer_id
where  year(order_date)='2023' 
and customers.customer_id not in 
          ( select customers.customer_id from customers  left join orders
            on customers.customer_id=orders.customer_id where year(order_date)='2024')
group by customers.customer_id,customer_name;
 
### Q9. Cancellation Rate Comparison
-- Calculate the cancellation rate for each restaurant between the 2023 and 2024.

with cte as (
select restaurants.Restaurant_id,restaurant_name , count(order_id)  as cancellation_count
from orders inner join restaurants
on restaurants.restaurant_id=orders.restaurant_id
where order_status="Not Fulfilled" and year(order_date) between "2023" and "2024"
group by restaurants.Restaurant_id,restaurant_name),

 total_count as (
select restaurants.Restaurant_id,Restaurant_name , count(order_id)  as total_counts
from orders inner join restaurants
on restaurants.restaurant_id=orders.restaurant_id
group by restaurants.Restaurant_id,restaurant_name)

select cte.Restaurant_id,cte.restaurant_name, 
concat(round(cte.cancellation_count * 100 / total_count.total_counts,2)," ","%") as cancellation_rate
from cte inner join total_count
on cte.Restaurant_id=total_count.Restaurant_id
group by cte.Restaurant_id,cte.restaurant_name
order by cancellation_rate desc;

### Q10. Rider Average Delivery Time
-- Determine each rider's average delivery time.

with cte as (
select riders.rider_id,rider_name,
case when delivery_time>ORDER_time then round(timestampdiff(minute,order_time,delivery_time),2)
     else round(timestampdiff(minute,order_time,delivery_time + interval 1 day),2) end as delivery_min
from riders left join deliveries
on riders.rider_id=deliveries.rider_id
inner join orders
on orders.order_id=deliveries.order_id)

select rider_id,rider_name, round(avg(delivery_min),2) as Avg_delivery_time_mints 
from cte
group by rider_id,rider_name
order by avg_delivery_time_mints desc;

### Q11. Monthly Restaurant Growth Ratio
-- Calculate each restaurant's growth ratio for monthly, based on the total number of delivered orders since its joining.

with cte as(
select r.restaurant_id,restaurant_name,date_format(order_date, "%m/%Y") as Month_no,
count(o.order_id) as Total_orders,
lag(count(o.order_id) ,1) over(partition by restaurant_id order by date_format(order_date, "%m/%Y")) as previous_month_order
from restaurants r left join orders o
on r.Restaurant_id=o.Restaurant_id
left join deliveries d
on d.order_id=o.order_id
where delivery_status = "Delivered"
group by  r.restaurant_id,restaurant_name,month_no
order by r.Restaurant_id,month_no)

select cte.restaurant_id,cte.restaurant_name,cte.month_no, cte.total_orders,previous_month_order,
round((cte.total_orders-previous_month_order)/previous_month_order*100,2) as Monthly_growth_ratio
from cte 
group by cte.restaurant_id,cte.restaurant_name,cte.month_no, cte.total_orders,
previous_month_order,Monthly_growth_ratio;

### Q12. Customer Segmentation
-- Segment customers into 'Gold' or 'Silver' groups based on their total spending compared to the average order value (AOV). If a customer's total spending exceeds the AOV, 
-- label them as 'Gold'; otherwise, label them as 'Silver'.
-- Return: The total number of orders and total revenue for each segment.

with cte as (
select customers.customer_id, round( avg(total_amount),2) as AOV
from customers inner join orders
group by customers.customer_id), 

cust_seg as (
select cte.customer_id,
case when sum(total_amount) > aov then "Gold"
     else "Silver"
     end as customer_segmentation
from cte inner join orders
on cte.customer_id=orders.customer_id
group by customer_id
order by customer_id)

select customer_segmentation,sum(total_amount) as Total_revenue, count(order_id) as Total_orders
from cust_seg inner join orders
on cust_seg.customer_id=orders.customer_id
group by customer_segmentation;

### Q13. Rider Monthly Earnings
-- Calculate each rider's total monthly earnings, assuming they earn 8% of the order amount.

select riders.rider_id,rider_name, date_format(order_date,"%m/%Y") as Month_no,round((sum(total_amount)*0.08),2) as Monthly_Earnings
from riders left join deliveries
on riders.rider_id=deliveries.rider_id
left join orders
on orders.order_id=deliveries.order_id
where order_status="Completed" and delivery_status="Delivered"
group by riders.rider_id,rider_name,month_no
order by riders.rider_id,month_no;

### Q14. Rider Ratings Analysis
-- Find the number of 5-star, 4-star, and 3-star ratings each rider has. Riders receive 
-- ratings based on delivery time:
-- ●	5-star: Delivered in less than 30 minutes
-- ●	4-star: Delivered between 30 and 45 minutes 
-- ●	3-star: Delivered after 45 minutes

with cte as (
select riders.rider_id,rider_name,
 case when delivery_time>order_time then round(timestampdiff(minute,order_time,delivery_time),2) 
      else round(timestampdiff(minute,order_time,delivery_time + interval 1 day),2)  end as Delivery_min
from riders left join deliveries
on riders.rider_id=deliveries.rider_id
left join orders
on orders.order_id=deliveries.order_id
where Order_status="completed" and delivery_status="delivered"
)
select rider_id, rider_name,
sum(case when delivery_min < 30 then 1 else 0 end) as Five_star,
sum(case when delivery_min between 30 and 45 then 1 else 0 end) as Four_star,
sum(case when delivery_min>45 then 1 else 0 end) as Three_star
from cte
group by rider_id,rider_name
order by rider_id;

### Q15. Order Frequency by Day
-- Analyze order frequency per day of the week and identify the peak day for each restaurant.

with cte as (
select restaurants.Restaurant_id,Restaurant_name,count(order_id) as Total_orders, dayname(order_date) as Day_of_week 
from orders inner join restaurants
on orders.Restaurant_id=restaurants.Restaurant_id
where order_status="completed"
group by restaurants.Restaurant_id,day_of_week
order by restaurants.Restaurant_id),
 
hightest_order_day as (
select restaurant_id, restaurant_name,day_of_week,total_orders,
dense_rank() over(partition by restaurant_id order by total_orders desc) as rnk
from cte)

select restaurant_id, restaurant_name,day_of_week,total_orders from hightest_order_day
where rnk=1;

### Q16. Customer Lifetime Value (CLV)
-- Calculate the total revenue generated by each customer over all their orders.

select customers.customer_id,customer_name, sum(total_amount) as CLV
from customers left join orders
on customers.customer_id=orders.Customer_id
group by customers.customer_id,customer_name;

### Q17. Monthly Sales Trends
-- Identify sales trends by comparing each month's total sales to the previous month.

with cte as (
select date_format(order_date,"%Y") as Year_no, date_format(order_date,"%m") as month_no,
sum(total_amount) as Total_sales, 
lag(sum(total_amount),1) over ( order by date_format(order_date,"%Y"), date_format(order_date,"%m")) as Previous_month_sales
from orders
group by year_no,month_no
order by year_no,month_no)

select Year_no,Month_no,Total_sales, Previous_month_sales,concat(round((total_sales-previous_month_sales)/previous_month_sales*100,2)," ","%") as Growth_ratio
from cte;

### Q18. Rider Efficiency
-- Evaluate rider efficiency by determining average delivery times and identifying those with the lowest and highest averages.

with cte as (
select r.rider_id,rider_name, 
case when delivery_time>order_time then round(timestampdiff(minute,order_time,delivery_time),2) 
     else round(timestampdiff(minute,order_time,delivery_time + interval 1 day),2) end as delivery_time_took
from riders r left join deliveries d
on r.rider_id=d.rider_id
left join orders o 
on o.order_id=d.order_id
where order_status="completed" and delivery_status="delivered"
group by r.rider_id,rider_name,delivery_time_took),

avg_deliv_time as (
select rider_id,rider_name,avg(delivery_time_took) as avg_delivery_time_took
from cte
group by rider_id,rider_name
)
select min(avg_delivery_time_took) as min_avg_delivery_time,
max(avg_delivery_time_took) as Max_avg_delivery_time
from avg_deliv_time;

### Q19. Order Item Popularity
-- Track the popularity of specific order items over time and identify seasonal demand spikes.

 with cte as(
 select order_item,count(order_id) as Total_orders,
    case 
       when month(order_date) between 3 and 5 then "Spring"
       when month(order_date) between 6 and 8 then "Summer"
       when month(Order_date) between 9 and 11 then "Autumn"
       else "Winter"
       end as Seasons
from orders 
where order_status="completed"
group by order_item,seasons),

Trending_item as (
select  order_item,Total_orders, seasons,dense_rank() over(partition by order_item order by total_orders desc) as rnk
from cte)

select order_item,Total_orders,seasons from trending_item where rnk=1;

### Q20. City Revenue Ranking
-- Rank each city based on the total revenue for the last year (2023).

select city, sum(total_amount) as Total_revenue,
dense_rank() over( order by sum(total_amount) desc) as rnk
from restaurants r left join orders o
on r.Restaurant_id=o.Restaurant_id
group by city
having total_revenue is not null;

## Stored Procedure

-- 1. create a stored Procedure to return all restaurants in a given city
-- Return : Restaurant_name,cusines type,opening hours

delimiter //
create procedure Restaurant_details ( in city varchar(30))
begin
     select distinct restaurant_name,city, order_item,opening_hours
     from restaurants r inner join orders o
     on r.Restaurant_id=o.Restaurant_id
     where city=city;
end //
delimiter ;
call restaurant_details('chennai');

-- 2. create a stored procedure to show the full order history of a customer sorted by most recent
-- return customer_name,order_id,restaurant_name, order_date,total amount
delimiter //
create procedure customer_details( in customer_id int)
begin
     select customer_name,order_id,restaurant_name,order_date,total_amount 
     from customers c left join orders o
     on c.Customer_id=o.Customer_id
     left join restaurants r
     on r.Restaurant_id=o.Restaurant_id
     where c.Customer_id=customer_id
     order by order_date desc;
end //
delimiter ;
call customer_details(1);

## Query Optimization 

-- Indexed customer_name, order_date, order_status, delivery_status columns
-- Avoided SELECT *, given proper columns in select statement
-- Used CTE’s and JOINS over subqueries 
-- Used JOINS wisely

## Key Insights 

-- Peak Ordering Time: Most orders are placed between 2 PM– 4 PM, while 12 AM – 2 AM sees minimal activity.

-- Top Customers: Customer id 6,7,5 placed over 750 orders each with an average order value of ₹300–₹350.

-- High Non-Deliveries: Restaurant id  5,7,3 show the highest non-delivery rates.

-- Seasonal Trends: Spring has the highest dish sales, while winter has the lowest, indicating seasonal food demand.

-- Underperforming Riders: Rider id 1,2,3,4 consistently show longer delivery times and receive 3-star ratings.

-- Top-Selling Items by City: Dishes like Chicken Biryani, Mutton Rogan Josh, and Paneer Butter Masala are best-sellers across major cities.

## Actionable Recommendations

-- Introduce Late-Night Offers: Encourage more orders during 12 AM – 2 AM with special discounts or combos.

-- Reward Loyal Customers: Launch a VIP program or exclusive deals for high-frequency users to boost retention.

-- Improve Restaurant Reliability: Audit and support restaurants with high non-delivery rates to reduce cancellations.

-- Optimize Delivery Staff: Provide training or reassignment for low-rated riders to enhance customer satisfaction.

-- Plan Seasonal Campaigns: Launch season-specific menus or offers during spring, and create winter discounts to balance demand.

-- Promote Best-Selling Dishes: Use regional best-sellers in targeted ads and bundled offers to drive repeat sales.

## Conclusion

This project highlights my ability to handle complex SQL queries and provides solutions to real-world business problems in the context of a food delivery service like Swiggy. The approach taken here demonstrates a structured problem-solving methodology, data manipulation skills, and the ability to derive actionable insights from data.

## Notice 
All customer names and data used in this project are computer-generated using AI and random functions. They do not represent real data associated with Swiggy or any other entity. This project is solely for learning and educational purposes, and any resemblance to actual persons, businesses, or events is purely coincidental.


