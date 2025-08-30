# DataLemur Hard Difficulty Problem Solving

## 1. Active User Retention
Assume you're given a table containing information on Facebook user actions. Write a query to obtain number of monthly active users (MAUs) in July 2022, including the month in numerical format "1, 2, 3".

**Hint:**

An active user is defined as a user who has performed actions such as 'sign-in', 'like', or 'comment' in both the current month and the previous month.

`user_actions` table:

|Column Name	|Type                                    |
|-------------|----------------------------------------|
|user_id      |integer                                 |
|event_id     |integer                                 |
|event_type   |string ("sign-in, "like", "comment")    |
|event_date   |datetime                                |

Query:

    SELECT
      july.month AS mth,
      COUNT(DISTINCT july.user_id) AS monthly_active_users
    FROM
      (SELECT
        DISTINCT user_id,
        EXTRACT(MONTH FROM event_date) AS month
      FROM user_actions
      WHERE EXTRACT(MONTH FROM event_date) = 7) AS july,
      (SELECT
        DISTINCT user_id,
        EXTRACT(MONTH FROM event_date) AS month
      FROM user_actions
      WHERE EXTRACT(MONTH FROM event_date) = 6) AS june
    WHERE july.user_id = june.user_id
    GROUP BY july.month;

Output:

|mth	|monthly_active_users|
|-----|--------------------|
|7	  |2                   |

## 2. Y-on-Y Growth Rate
Assume you're given a table containing information about Wayfair user transactions for different products. Write a query to calculate the year-on-year growth rate for the total spend of each product, grouping the results by product ID.

The output should include the year in ascending order, product ID, current year's spend, previous year's spend and year-on-year growth percentage, rounded to 2 decimal places.

`user_transactions` table:
|Column Name        |Type        |
|-------------------|------------|
|transaction_id     |integer     |
|product_id         |integer     |
|spend              |decimal     |
|transaction_date   |datetime    |

Query:

    SELECT
      DISTINCT (EXTRACT(YEAR FROM transaction_date)) AS year,
      product_id,
      SUM(spend) AS curr_year_spend,
      LAG(spend, 1) OVER(PARTITION BY product_id ORDER BY EXTRACT(YEAR FROM transaction_date)) AS prev_year_spend,
      ROUND((((SUM(spend)) 
          - (LAG(spend, 1) OVER(PARTITION BY product_id ORDER BY EXTRACT(YEAR FROM transaction_date)))) 
          / LAG(spend, 1) OVER(PARTITION BY product_id ORDER BY EXTRACT(YEAR FROM transaction_date)) 
          * 100), 2) AS yoy_rate
    FROM user_transactions
    GROUP BY year, product_id, spend
    ORDER BY year, product_id;

Output (first 5 row):

|year    |product_id    |curr_year_spend    |prev_year_spend    |yoy_rate    |
|--------|--------------|-------------------|-------------------|------------|
|2019    |123424        |1500.60            |NULL               |NULL        |
|2020    |123424        |1000.20            |1500.60            |-33.35      |
|2021    |123424        |1246.44            |1000.20            |24.62       |
|2022    |123424        |2145.32            |1246.44            |72.12       |
|2019    |234412        |1800.00            |NULL               |NULL        |

