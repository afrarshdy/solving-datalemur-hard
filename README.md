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
