1.Import the dataset and do usual exploratory analysis steps like checking the
structure & characteristics of the dataset.
A.	Data type of all columns in the “customers” table.

ANS :

select *
from `Ecommerce.Target_customers`
![image](https://github.com/Poojaservi/Targetproject/assets/161363262/2018fd4b-2b9b-4a5e-9ead-335469f3742b)

 

Reference : we have imported all the customers from target.

B.	Get the time range between which the orders were placed.

select max(order_purchase_timestamp) as max, min(order_purchase_timestamp) as min
from `Ecommerce.Target_orders`
order by 1, 2

 ![image](https://github.com/Poojaservi/Targetproject/assets/161363262/2bb3b6e7-3672-449e-96e5-f97429440e41)


Reference : We could se maximum orders in 2018-10-17 and minimum orders in 2016-09-04


C.	Count the number of Cities and States in our dataset.

select count(distinct(geolocation_city)) as geolocation_city, count(distinct(geolocation_state)) as geolocation_state
from `Ecommerce.Target_geolocation`

 ![image](https://github.com/Poojaservi/Targetproject/assets/161363262/54adc121-202d-4ed2-8f87-45d457019e3c)


Reference : We could see total cities are 8011 and states are 27

II.In-depth Exploration:

A.	Is there a growing trend in the no. of orders placed over the past years?

select extract(year from order_purchase_timestamp) as year,
extract(month from order_purchase_timestamp) as month,
count(order_id) as orders
from`Ecommerce.Target_orders`
group by 1,2
order by 1,2

 ![image](https://github.com/Poojaservi/Targetproject/assets/161363262/bbb6bd63-440e-4666-9519-82ad2babed83)


Reference : In 2016 the trade had just stated, we could see drastic increase from 2017 jan month


B.	Can we see some kind of monthly seasonality in terms of the no. of orders being placed?

select extract(year from order_purchase_timestamp) as year,
extract(month from order_purchase_timestamp) as month,
count(order_id) as orders 
from`Ecommerce.Target_orders`
group by 1, 2
having count(order_id) < 7544
order by 1, 2 
limit 5
![image](https://github.com/Poojaservi/Targetproject/assets/161363262/36a394f3-07cf-4ab1-8644-2ee7aca8f818)

 

Reference : we could see in 2016 329 orders were placed and in 2017,2580 orders were made.so there is drastic increase in 2017

C. During what time of the day, do the Brazilian customers mostly place their
orders? (Dawn, Morning, Afternoon or Night)
● 0-6 hrs : Dawn
● 7-12 hrs : Mornings
● 13-18 hrs : Afternoon
● 19-23 hrs : Night

select case when extract(hour from order_purchase_timestamp) in(00, 06)
--<= 06
then 'Dawn'
when extract(hour from order_purchase_timestamp) in(07, 12)
and extract(hour from order_purchase_timestamp) >= 07 then 'Morning'
when extract(hour from order_purchase_timestamp) in(13, 18)
and extract(hour from order_purchase_timestamp) >= 13
then 'Afternoon'
else 'Night' 
end as time_of_day, count(order_id) as no_of_orders
from `Ecommerce.Target_orders`
group by 1
 ![image](https://github.com/Poojaservi/Targetproject/assets/161363262/1264785f-0e08-487d-bf1d-cfc6efb30db5)

Reference : At night time 77032 orders were made
            At afternoon 12287 orders were made
            At Morning 7226 orders were made
            At Dawn 2896 orders were made
            So orders were made more at night time because customers would be free at that time from there busy schedule.










III. Evolution of E-commerce orders in the Brazil region:

A.	Get the month on month no. of orders placed in each state.

with base as
(select a.*,
extract(month from order_purchase_timestamp) as month, b.customer_state from `Ecommerce.Target_orders`a
inner join `Ecommerce.Target_customers`b
on a.customer_id = b.customer_id)
select customer_state, month, count(distinct order_id) as no_of_orders
from base
group by 1, 2
order by 1, 2

 ![image](https://github.com/Poojaservi/Targetproject/assets/161363262/5917996a-ede2-4bed-b8cc-790d6bcd2e20)


Reference : we could see there 322 total orders from all the states

B.	How are the customers distributed across all the states?

select count(customer_unique_id) as customer,customer_state
from `Ecommerce.Target_customers`
group by customer_unique_id, customer_state

![image](https://github.com/Poojaservi/Targetproject/assets/161363262/b2139664-33dd-4e63-9d01-a46c6fb61a61)

 

 Reference : we could see unique customers from each state.


V. Analysis based on sales, freight and delivery time.
A. Find the no. of days taken to deliver each order from the order’s purchase date
as delivery time.
Also, calculate the difference (in days) between the estimated & actual delivery
date of an order.
Do this in a single query.

select time_to_deliver, diff_estimated_delivery, count(X.order_id) as no_of_orders from 
(select order_id,
date_diff (order_delivered_customer_date, order_purchase_timestamp,day) as time_to_deliver from `Ecommerce.Target_orders`) X
JOIN
(SELECT order_id,
date_diff (order_estimated_delivery_date,order_delivered_customer_date,day) as diff_estimated_delivery from `Ecommerce.Target_orders`) Y
on X.order_id = Y.order_id
group by X.time_to_deliver, Y.diff_estimated_delivery, X.order_id
order by X.time_to_deliver, Y.diff_estimated_delivery, X.order_id

![image](https://github.com/Poojaservi/Targetproject/assets/161363262/bb764277-c015-42ee-8ee0-96e6af5b134d)

 


C.	Find out the top 5 states with the highest & lowest average freight value.


select
count(oi.order_id) as count, avg(freight_value) as average_freight_value,c.customer_state
from `Ecommerce.Target_order_items`oi
join `Ecommerce.Target_orders`o
on oi.order_id = o.order_id
join `Ecommerce.Target_customers`c
on c.customer_id = o.customer_id
group by 3
order by 1, 2 asc
limit 5;

![image](https://github.com/Poojaservi/Targetproject/assets/161363262/85a3069f-4736-4e94-8419-916b79ab5bbe)
 


