# :sparkles: StrataScratch-Interview-Questions
[StrataScratch](https://www.stratascratch.com/) is platform where real interview questions asked by  top companies. Here I am solving SQL questions. 

🧰 Tool : MySQL

## Table of Contents:
- [Difficulty Level Medium](##Difficulty-Level-Medium)
- [Difficulty Level Hard](##Difficulty-Level-Hard)
_____________________

## Difficulty Level: Medium

📌 **Most Profitable Companies**
- [Question](https://platform.stratascratch.com/coding/10354-most-profitable-companies?code_type=3) : Find the 3 most profitable companies in the entire world. Output the result along with the corresponding company name. Sort the result based on profits in descending order.

```SQL
SELECT 
    company,
    profits
FROM(
    SELECT
        company,
        profits,
        RANK() OVER(ORDER BY profits DESC) AS comp_rank
    FROM 
        forbes_global_2010_2014) base
WHERE
    comp_rank <=3;
```
##### Answer:
![M1](https://user-images.githubusercontent.com/112051343/188779833-50f3643d-896e-499b-ae78-07e3001edbb5.JPG)
_________________________________________

📌 **Workers With The Highest Salaries**
- [Question](https://platform.stratascratch.com/coding/10353-workers-with-the-highest-salaries?code_type=3): Find the titles of workers that earn the highest salary. Output the highest-paid title or multiple titles that share the highest salary.

```SQL
SELECT
    t.worker_title AS best_paid_title
FROM 
    worker w JOIN title t ON w.worker_id = t.worker_ref_id
WHERE
    salary = (SELECT MAX(salary) FROM worker)
GROUP BY  t.worker_title
ORDER BY salary DESC, t.worker_title;
```
##### Answer:
![M2](https://user-images.githubusercontent.com/112051343/188780791-ba83d921-8387-4e06-b418-c6cd8f70666f.JPG)
_________________________________________________

📌 **Users By Average Session Time**
- [Question](https://platform.stratascratch.com/coding/10352-users-by-avg-session-time?code_type=3): Calculate each user's average session time. A session is defined as the time difference between a page_load and page_exit. For simplicity, assume a user has only 1 session per day and if there are multiple of the same events on that day, consider only the latest page_load and earliest page_exit. Output the user_id and their average session time.


**Method 1:**
```sql
WITH user_activity_cte AS(
    SELECT
        DATE(timestamp), 
        user_id,
        MAX(CASE WHEN action = 'page_load' THEN timestamp END) AS page_load_time,
        MIN(CASE WHEN action = 'page_exit' THEN timestamp END) AS page_exit_time
    FROM
        facebook_web_log
    GROUP BY 1,2)
SELECT
user_id,
    AVG(TIMESTAMPDIFF(second,page_load_time,page_exit_time)) AS timediff
FROM 
    user_activity_cte 
GROUP BY 1;
```
**Method 2:**

```SQL
WITH max_timestamp AS(
    SELECT
        user_id,
        DATE(timestamp) AS date_part, 
        MAX(timestamp) AS max_page_load
    FROM
        facebook_web_log
    WHERE
        action = 'page_load'
    GROUP BY 1,2),
min_timestamp AS(
    SELECT
        user_id,
        DATE(timestamp) AS date_part , 
        MIN(timestamp) AS min_page_exit
    FROM
    facebook_web_log
    WHERE
        action = 'page_exit'
    GROUP BY 1,2)
SELECT
T.user_id,
    AVG(TIME_TO_SEC(TIMEDIFF(min_page_exit,max_page_load))) AS timediff
FROM 
    max_timestamp T JOIN min_timestamp mt ON T.user_id=mt.user_id
    AND  T.date_part=mt.date_part
GROUP by 1;
```
##### Answer:
![M3](https://user-images.githubusercontent.com/112051343/188785846-0ef327f6-2a33-47e4-980b-dcd3b86ff842.JPG)
_____________________________________________
📌 **Finding User Purchases**
- [Question](https://platform.stratascratch.com/coding/10322-finding-user-purchases?code_type=3): Write a query that'll identify returning active users. A returning active user is a user that has made a second purchase within 7 days of any other of their purchases. Output a list of user_ids of these returning active users.

```SQL
WITH next_date_cte AS(
        SELECT 
            user_id, 
            created_at,
            LEAD(created_at) OVER(PARTITION BY  user_id ORDER BY created_at) AS second_purchase
        FROM amazon_transactions
)
SELECT 
    DISTINCT user_id 
FROM
    next_date_cte
WHERE DATEDIFF(second_purchase,created_at) <= 7;
```
##### Answer:
![M4](https://user-images.githubusercontent.com/112051343/188787974-0e75859b-69fe-4b56-aa88-eef12595611a.JPG)
___________________________________________________
📌 **Acceptance Rate By Date**
- [Question](https://platform.stratascratch.com/coding/10285-acceptance-rate-by-date?code_type=3): What is the overall friend acceptance rate by date? Your output should have the rate of acceptances by the date the request was sent. Order by the earliest date to latest.
Assume that each friend request starts by a user sending (i.e., user_id_sender) a friend request to another user (i.e., user_id_receiver) that's logged in the table with action = 'sent'. If the request is accepted, the table logs action = 'accepted'. If the request is not accepted, no record of action = 'accepted' is logged.

```SQL
SELECT 
  date,
  SUM(ACCEPT_ACTION)/SUM(SENT_ACTION) AS percentage_acceptance
FROM(
    SELECT 
        user_id_sender, 
        user_id_receiver,
        MIN(DATE) AS DATE,
        SUM(CASE WHEN ACTION = 'sent' THEN 1 ELSE 0 END) AS SENT_ACTION,
        SUM(CASE WHEN ACTION = 'accepted' THEN 1 ELSE 0 END) AS ACCEPT_ACTION
    FROM fb_friend_requests
    GROUP BY 1,2) base
GROUP BY 1
```
##### Answer:
![M5](https://user-images.githubusercontent.com/112051343/188792087-046f745e-f78e-4c24-9095-ef7e272dd91b.JPG)
_________________________________________________________________________
📌 **Ranking Most Active Guests**
- [Question](https://platform.stratascratch.com/coding/10159-ranking-most-active-guests?code_type=3): Rank guests based on the number of messages they've exchanged with the hosts. Guests with the same number of messages as other guests should have the same rank. Do not skip rankings if the preceding rankings are identical.
Output the rank, guest id, and number of total messages they've sent. Order by the highest number of total messages first.

```sql
SELECT
    DENSE_RANK()OVER(ORDER BY sum_n_messages DESC) AS RNK,
    id_guest,sum_n_messages
FROM (
    SELECT 
        id_guest,
        SUM(n_messages) AS sum_n_messages
    FROM airbnb_contacts
    GROUP BY 1) base;
```
##### Answer:
![M6](https://user-images.githubusercontent.com/112051343/188829032-bcf1605a-034c-4082-9679-50b1860d675e.JPG)
______________________________
📌 **Number Of Units Per Nationality**
- [Question](https://platform.stratascratch.com/coding/10156-number-of-units-per-nationality?code_type=3): Find the number of apartments per nationality that are owned by people under 30 years old.Output the nationality along with the number of apartments.
Sort records by the apartments count in descending order.
```sql
WITH host_cte AS (
    SELECT 
        DISTINCT(host_id), 
        nationality, age
    FROM airbnb_hosts  
    )
SELECT 
    h.nationality, 
    COUNT(u.unit_id) apartment_count
FROM 
    host_cte h JOIN airbnb_units u ON h.host_id = u.host_id
WHERE 
    h.age < 30 AND u.unit_type = 'Apartment'
GROUP BY h.nationality 
ORDER BY apartment_count DESC
```
##### Answer:
![M7](https://user-images.githubusercontent.com/112051343/188838184-d2f1c14e-dda4-4719-9b5e-d26a3f267ea4.JPG)
_______________________________________
📌 **Income By Title and Gender**
- [Question](https://platform.stratascratch.com/coding/10077-income-by-title-and-gender?code_type=3): Find the average total compensation based on employee titles and gender. Total compensation is calculated by adding both the salary and bonus of each employee. However, not every employee receives a bonus so disregard employees without bonuses in your calculation. Employee can receive more than one bonus. Output the employee title, gender (i.e., sex), along with the average total compensation.
```sql
WITH bonus_cte AS(
    SELECT 
        worker_ref_id, 
        SUM(bonus) AS total_bonus
    FROM sf_bonus
    GROUP BY worker_ref_id
    )
SELECT 
    employee_title,sex,
    AVG(salary+total_bonus) AS avg_compensation
FROM bonus_cte b JOIN sf_employee e ON b.worker_ref_id=e.id
GROUP BY employee_title,sex
```
##### Answer:
![M8](https://user-images.githubusercontent.com/112051343/188846735-32d48bba-40ca-4263-9a3e-89a28bf859d9.JPG)
___________________________________
📌 **Top Cool Votes**
- [Question](https://platform.stratascratch.com/coding/10060-top-cool-votes?code_type=3): Find the review_text that received the highest number of  'cool' votes.
Output the business name along with the review text with the highest numbef of 'cool' votes.
```sql
SELECT
    business_name, 
    review_text 
FROM(
    SELECT
        *,
        RANK()OVER(ORDER BY cool DESC) AS cool_rank
    FROM yelp_reviews) A
WHERE cool_rank =1
```
##### Answer:
![M9](https://user-images.githubusercontent.com/112051343/188852371-350de9e0-ef96-47eb-8dfd-21b83794f1cc.JPG)


























_______________________________________________________________________
## Difficulty Level: Hard

📌 **Monthly Percentage Difference**

- [Question](https://platform.stratascratch.com/coding/10319-monthly-percentage-difference?code_type=3) : Given a table of purchases by date, calculate the month-over-month percentage change in revenue. The output should include the year-month date (YYYY-MM) and percentage change, rounded to the 2nd decimal point, and sorted from the beginning of the year to the end of the year.
The percentage change column will be populated from the 2nd month forward and can be calculated as ((this month's revenue - last month's revenue) / last month's revenue)*100. 

```SQL
WITH new_date_cte AS(
        SELECT *,
               date_format(created_at, '%Y-%m') as new_date
        FROM 
           sf_transactions),
revenue_cte AS(
        SELECT new_date,
               SUM(value) AS revenue
        FROM 
           new_date_cte
        GROUP BY new_date)
SELECT 
    new_date,
    round(100 * (revenue - LAG(revenue) OVER(ORDER BY new_date))/LAG(revenue) OVER(ORDER BY new_date),2) AS revenue_diff
FROM
  revenue_cte;
```
##### Answer:

![Amazone-monthly-percentage-difference-output](https://user-images.githubusercontent.com/112051343/188700178-532abb7d-e832-400a-8bf8-5587bce6e400.JPG)
_________________________________
📌 **Premium vs Freemium**

- [Question](https://platform.stratascratch.com/coding/10300-premium-vs-freemium?code_type=3): Find the total number of downloads for paying and non-paying users by date. Include only records where non-paying customers have more downloads than paying customers. The output should be sorted by earliest date first and contain 3 columns date, non-paying downloads, paying downloads.


```SQL
SELECT 
    date,
    SUM(CASE WHEN paying_customer='no' THEN downloads END) AS non_paying,
    SUM(CASE WHEN paying_customer='Yes' THEN downloads END) AS paying
FROM 
    ms_user_dimension u 
JOIN 
    ms_acc_dimension a ON u.acc_id=a.acc_id
JOIN
    ms_download_facts d ON u.user_id=d.user_id
GROUP BY 
    date
HAVING 
    non_paying > paying
ORDER BY 
    date;

```
##### Answer:

![2](https://user-images.githubusercontent.com/112051343/188704510-807f5ff5-e3ac-4762-b35c-d8027f324adf.JPG)
______________________________________
📌 **Popularity Percentage**

- [Question](https://platform.stratascratch.com/coding/10284-popularity-percentage?code_type=3): Find the popularity percentage for each user on Meta/Facebook. The popularity percentage is defined as the total number of friends the user has divided by the total number of users on the platform, then converted into a percentage by multiplying by 100.
Output each user along with their popularity percentage. Order records in ascending order by user id.
The 'user1' and 'user2' column are pairs of friends.

```SQL
WITH users AS(
        SELECT user1,user2 
        FROM facebook_friends
        UNION 
        SELECT user2,user1 
        FROM facebook_friends
        ORDER BY 1) 
SELECT user1,
100*(COUNT(user2)/(SELECT COUNT(DISTINCT user1) FROM users)) AS popularity_percent 
FROM users
GROUP BY user1
ORDER BY user1;

```
##### Answer:

![3](https://user-images.githubusercontent.com/112051343/188708134-b46716cf-675c-4b17-a576-4c4534d438bb.JPG)
__________________________________
📌 **Marketing Campaign Success [Advanced]**

- [Question](https://platform.stratascratch.com/coding/514-marketing-campaign-success-advanced?code_type=3): You have a table of in-app purchases by user. Users that make their first in-app purchase are placed in a marketing campaign where they see call-to-actions for more in-app purchases. Find the number of users that made additional in-app purchases due to the success of the marketing campaign.
The marketing campaign doesn't start until one day after the initial in-app purchase so users that only made one or multiple purchases on the first day do not count, nor do we count users that over time purchase only the products they purchased on the first day.


```SQL
WITH initial_purches_cte AS(
SELECT *,
        DENSE_RANK() OVER(PARTITION BY user_id ORDER BY created_at) rank_by_order,
        DENSE_RANK() OVER(PARTITION BY user_id, product_id ORDER BY created_at) rank_by_product
    FROM marketing_campaign)
SELECT
    COUNT(DISTINCT user_id) total_users
FROM
initial_purches_cte
WHERE rank_by_order > 1 AND rank_by_product = 1;
```
##### Answer:

![4](https://user-images.githubusercontent.com/112051343/188716853-3a222426-84d9-4a7a-a758-cd4cc8381596.JPG)
____________________________________






