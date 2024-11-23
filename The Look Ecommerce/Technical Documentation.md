# The Look Ecommerce Data Analysis Technical Documentation

## Data Preparation
* The data used in this project is the [theLook eCommerce](https://console.cloud.google.com/marketplace/product/bigquery-public-data/thelook-ecommerce?inv=1&invt=AbiOdQ&project=portfolio-datasets-437213) which is a fictitious dataset available for public querying via the BigQuery.

* The dataset contains information about customers, products, orders, logistics, web events and digital marketing campaigns. The contents of the dataset are synthetic, and are provided to industry practitioners for the purpose of product discovery, testing, and evaluation.

* For the purposes of this project, we only collect data on **completed orders** which are the successful orders from the time the products were ordered up to the time the items were delivered to the customers with no returns.

* We take a look at metrics such as order volume, total sales, and revenue.

* Upon intial observation of the dataset, these are the tables we will be joining and the columns we will be using:
  - **Table A:** Orders - order_id, created_at, user_id, status = Completed, gender
  - **Table B:** Order_items - order_id, product_id, sale_price
  - **Table C:** Products - id, cost, category, name, brand, retail_price, department, distribution_center_id
  - **Table D:** Users - id, age, gender, state, city, country, latitude, longitude, traffic_source


## Data Processing

* **SQL Query #1: Create the final table to be analyzed, JOINing all relevant tables and columns.**

```
CREATE OR REPLACE TABLE `portfolio-datasets-437213.thelook_ecommerce.orders_sales_users` AS
WITH a AS (
  SELECT order_id, created_at, user_id, status
  FROM `bigquery-public-data.thelook_ecommerce.orders`
  WHERE status = "Complete"
), b AS (
  SELECT order_id, product_id, sale_price
  FROM `bigquery-public-data.thelook_ecommerce.order_items`
), c AS (
  SELECT id, cost, category, name, brand, retail_price, department, distribution_center_id
  FROM `bigquery-public-data.thelook_ecommerce.products`
), d AS (
  SELECT id, age, gender, state, city, country, latitude, longitude, traffic_source
  FROM `bigquery-public-data.thelook_ecommerce.users`
)

SELECT a.order_id, a.created_at AS order_date, b.product_id AS product_id, c.category AS category, c.name AS name, c.brand AS brand, c.retail_price AS retail_price, c.department AS department, c.distribution_center_id AS distribution_center, b.sale_price AS sale_price, a.status AS order_status, d.age AS age, d.gender AS gender, d.state AS state, d.city AS city, d.country AS country, d.latitude AS lat, d.longitude AS long, d.traffic_source AS traffic_source
FROM a
JOIN b
ON a.order_id = b.order_id
JOIN c
ON b.product_id = c.id
JOIN d
ON a.user_id = d.id;

```

Let's try answering a few business questions using our table.

* **SQL Query #2: Total number of sales broken down by product in descending order.**

```

SELECT name, SUM(sale_price) AS total_sales
FROM `portfolio-datasets-437213.thelook_ecommerce.orders_sales_users`
GROUP BY name
ORDER BY total_sales DESC
LIMIT 10;

```

*Result:*

| name  | total_sales |
| ------------- | ------------- |
| Canada Goose Men's The Chateau Jacket | 7335.0 |
| The North Face Denali Down Mens Jacket 2013 | 4515.0 |
| The North Face Apex Bionic Soft Shell Jacket - Men's | 4515.0 |
| Darla | 3996.0 |
| Canada Goose Women's Mystique | 3750.0 |
| The North Face Apex Bionic Mens Soft Shell Ski Jacket 2013 | 3612.0 |
| Nobis Yatesy Parka | 2850.0 |
| Arc'teryx Women's Caliber Cardigan | 2796.0 |
| The North Face Freedom Mens Ski Pants 2013 | 2709.0 |
| Mens Nike AirJordan Varsity Hoodie Jacket Grey / Black 451582-066 | 2709.0 |

* **SQL Query #3: The store’s top 10 customers with the highest average price per order.**

```

SELECT first_name, last_name, AVG(sale_price) AS AOP
FROM `portfolio-datasets-437213.thelook_ecommerce.orders_sales_users`
GROUP BY first_name, last_name
ORDER BY AOP DESC
LIMIT 10;

```

*Result:*

| first_name | last_name | AOP |
| ------------- | ------------- | ------------- |
| Brooke | Banks | 903.0 |
| Dwayne | Keller | 903.0 |
| Joshua | Bowers | 903.0 |
| Lucas | Cardenas | 903.0 |
| Tommy | Carter | 903.0 |
| Larry | Archer| 903.0 |
| Travis | Davis| 903.0 |
| Julia | Davis | 825.0 |
| Jonathan| Zimmerman | 815.0 |
| Dennis | Gibson| 815.0 |

* **SQL Query #4: Product volume by department.**

```

SELECT department, COUNT(name) AS product_count
FROM `bigquery-public-data.thelook_ecommerce.products`
GROUP BY department

```
*Result:*

| department | product_count |
| ------------- | ------------- |
| Women | 15988 |
| Men | 13130 |

## Conclusion:

* The final resulting table from SQL Query #1 can be accessed The Look [here](Ecommerce/thelook_ecommerce.csv).

* The EDA notebook of this dataset can be accessed [here](https://github.com/teejtolentino/Data-Projects/blob/67b5676f96fab5afac525028c3555c1361d1eb9e/The%20Look%20Ecommerce/theLook%20ecommerce%20EDA.ipynb).

