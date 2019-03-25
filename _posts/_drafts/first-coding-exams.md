---
layout: post
title: Preparing For Your First Coding Exams
author: Andrew Kulpa
---


Process:
  * Analyze a problem statement and recognize where it is ambiguous and under defined
  * Frame the problem
  * Implement your solution
  * Try to spot bugs and errors
  * Repeat this process and adapt your solution whenever appropriate
  * Keep your data structures and algorithm knowledge at your fingertips
Java:
  * TwoSum

SQL:
  * Big-Countries (where):
    * Solution:
      ``` SQL 
      SELECT name, population, area
      FROM World
      WHERE population > 25000000 OR area > 3000000
      ```
  * Not Boring Movies (Order By)
    * Solution:
      ``` SQL 
      SELECT *
      FROM cinema
      WHERE id % 2 != 0
          AND description != 'boring'
      ORDER BY rating DESC;
      ```
  * Combine-Two-Tables (left join)
  * Duplicate Emails (duplicates)
    * Solution:
      ``` SQL 
      SELECT DISTINCT Email
      FROM Person `p1`
      Inner Join Person `p2` USING(Email)
      WHERE p1.Id != p2.Id;
      ```
  * Customers who never order (group by; having; null check)
    * Solution:
      ``` SQL 
      SELECT Name 'Customers'
      FROM customers
      LEFT JOIN orders
      ON customers.Id = orders.CustomerId
      GROUP BY customers.id
      HAVING COUNT(orders.id) = 0
      ```
  * Classes More Than 5 Students (group by; having; distinct counts)
    * Solution:
      ``` SQL 
      SELECT class
      FROM courses
      GROUP BY class
      HAVING count(DISTINCT student) > 4
      ```
  * swap-salary (update when)
    * Solution:
      ``` SQL 
      UPDATE salary
      SET sex = CASE sex
          WHEN 'm' THEN 'f'
          WHEN 'f' THEN 'm'
          ELSE sex
      END
      ```
  * Employees-Earning-More-Than-Their-Managers (self join):
    * Solution:
      ``` SQL 
      SELECT e1.name 'Employee'
      FROM employee e1
      INNER JOIN employee e2 
      WHERE e1.ManagerId = e2.Id
      AND e1.salary > e2.salary 
      ```
  * Rising Temperature (self join, to_days)
    * Solution:
      ``` SQL 
      SELECT w2.Id
      FROM Weather w1, Weather w2
      WHERE TO_DAYS(w2.RecordDate) = TO_DAYS(w1.RecordDate) + 1
      AND w2.temperature > w1.temperature
      ```
  * Delete Duplicate Emails (self join, nested IN SELECT)
    * Solution:
      ``` SQL 
      DELETE p1 FROM Person p1
      WHERE p1.Id IN (
          SELECT p3.Id
          FROM (SELECT * FROM Person) AS p2
          INNER JOIN (SELECT * FROM Person) AS p3 USING (Email)
          WHERE p2.Id < p3.Id
      )
      ```
    * Solution 2:
      ``` SQL 
      DELETE p1 FROM Person p1,
          Person p2
      WHERE
          p1.Email = p2.Email AND p1.Id > p2.Id
      ```
  * Second Highest Salary (Self join, Limit, Offset, Distinct)
    * Solution:
      ``` SQL 
      SELECT
        (SELECT DISTINCT
                Salary
            FROM
                Employee
            ORDER BY Salary DESC
            LIMIT 1 OFFSET 1) AS SecondHighestSalary;
      ```

