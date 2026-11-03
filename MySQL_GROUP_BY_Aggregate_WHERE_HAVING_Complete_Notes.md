# MySQL Complete Notes: GROUP BY, Aggregate Functions, WHERE and HAVING

## 1. Introduction

Database mein aksar humein sirf individual records nahi, balki data ka
summary bhi chahiye hota hai.

For example:

-   Company mein total kitne employees hain?
-   Sabhi employees ki total salary kitni hai?
-   Average salary kya hai?
-   Har department mein kitne employees hain?
-   Har department ki total salary kitni hai?
-   Kaun-se departments ki average salary ₹50,000 se zyada hai?
-   Sirf Jaipur ke employees ka department-wise count kya hai?

In questions ka answer nikalne ke liye hum Aggregate Functions, `WHERE`,
`GROUP BY` aur `HAVING` use karte hain.

------------------------------------------------------------------------

# Part 1: Practical database setup

## Step 1: Database create karein

``` sql
CREATE DATABASE company_db;
USE company_db;
```

## Step 2: Employees table create karein

``` sql
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(50),
    department VARCHAR(50),
    city VARCHAR(50),
    salary DECIMAL(10,2),
    experience INT
);
```

## Step 3: Sample data insert karein

``` sql
INSERT INTO employees
(emp_id, emp_name, department, city, salary, experience)
VALUES
(1, 'Rahul', 'IT', 'Jaipur', 60000, 5),
(2, 'Priya', 'HR', 'Delhi', 45000, 3),
(3, 'Amit', 'IT', 'Jaipur', 75000, 7),
(4, 'Neha', 'Sales', 'Mumbai', 40000, 2),
(5, 'Rohit', 'HR', 'Jaipur', 50000, 4),
(6, 'Pooja', 'IT', 'Delhi', 80000, 8),
(7, 'Karan', 'Sales', 'Jaipur', 55000, 5),
(8, 'Anjali', 'Marketing', 'Mumbai', 65000, 6),
(9, 'Vikas', 'IT', 'Mumbai', 70000, 6),
(10, 'Sneha', 'Marketing', 'Jaipur', 48000, 3),
(11, 'Manish', 'HR', 'Delhi', 52000, 5),
(12, 'Riya', 'Sales', 'Delhi', 60000, 6);
```

Check data:

``` sql
SELECT * FROM employees;
```

------------------------------------------------------------------------

# Part 2: Aggregate Functions

## 2. Aggregate Function kya hota hai?

Aggregate Functions multiple rows ke data par calculation perform karti
hain aur generally ek summarized result return karti hain.

Example:

Agar 5 employees ki salaries hain:

`30000, 40000, 50000, 60000, 70000`

To aggregate functions se hum:

-   Total salary calculate kar sakte hain.
-   Average salary calculate kar sakte hain.
-   Highest salary find kar sakte hain.
-   Lowest salary find kar sakte hain.
-   Employees ki count nikal sakte hain.

## 3. COUNT()

`COUNT()` ka use records ya non-NULL values ki counting karne ke liye
hota hai.

### Syntax

``` sql
SELECT COUNT(column_name)
FROM table_name;
```

### Example 1: Total employees count

``` sql
SELECT COUNT(*) AS total_employees
FROM employees;
```

Expected result: `12`

### Example 2: Employee IDs count

``` sql
SELECT COUNT(emp_id) AS total_employees
FROM employees;
```

### Example 3: Salary values count

``` sql
SELECT COUNT(salary) AS salary_count
FROM employees;
```

**Important difference:**

-   `COUNT(*)` table ke sabhi matching rows count karta hai, chahe kisi
    column mein NULL ho.
-   `COUNT(column_name)` us column ki sirf non-NULL values count karta
    hai.
-   `COUNT(DISTINCT column_name)` unique non-NULL values count karta
    hai.

### Example 4: Unique departments count

``` sql
SELECT COUNT(DISTINCT department) AS total_departments
FROM employees;
```

Expected result: `4`

## 4. SUM()

`SUM()` ka use numeric column ke values ka total calculate karne ke liye
hota hai.

### Syntax

``` sql
SELECT SUM(column_name)
FROM table_name;
```

### Example: Total salary

``` sql
SELECT SUM(salary) AS total_salary
FROM employees;
```

Expected result: `670000`

### Example: IT department ki total salary

``` sql
SELECT SUM(salary) AS it_total_salary
FROM employees
WHERE department = 'IT';
```

Expected result: `285000`

## 5. AVG()

`AVG()` ka use numeric values ka average calculate karne ke liye hota
hai.

### Syntax

``` sql
SELECT AVG(column_name)
FROM table_name;
```

### Example: Average employee salary

``` sql
SELECT AVG(salary) AS average_salary
FROM employees;
```

Expected result: `55833.33` (approximately)

### Example: Average experience

``` sql
SELECT AVG(experience) AS average_experience
FROM employees;
```

## 6. MIN()

`MIN()` kisi column ki minimum value return karta hai.

### Example: Minimum salary

``` sql
SELECT MIN(salary) AS minimum_salary
FROM employees;
```

Expected result: `40000`

## 7. MAX()

`MAX()` kisi column ki maximum value return karta hai.

### Example: Maximum salary

``` sql
SELECT MAX(salary) AS maximum_salary
FROM employees;
```

Expected result: `80000`

## 8. All aggregate functions together

``` sql
SELECT
    COUNT(*) AS total_employees,
    SUM(salary) AS total_salary,
    AVG(salary) AS average_salary,
    MIN(salary) AS minimum_salary,
    MAX(salary) AS maximum_salary
FROM employees;
```

### Expected output

  -----------------------------------------------------------------------------------
    total_employees   total_salary   average_salary   minimum_salary   maximum_salary
  ----------------- -------------- ---------------- ---------------- ----------------
                 12         670000         55833.33            40000            80000

  -----------------------------------------------------------------------------------

**Note:** `SUM()` aur `AVG()` NULL values ko ignore karte hain. Agar koi
matching non-NULL numeric value nahi hai, to result NULL ho sakta hai.
`COUNT()` ke rules alag hain, jaise upar explain kiya gaya hai.

------------------------------------------------------------------------

# Part 3: WHERE Clause

## 9. WHERE kya hota hai?

`WHERE` clause ka use table ke individual records ko condition ke basis
par filter karne ke liye hota hai.

Yaani hum decide karte hain ki kaun-si rows par query ka operation
perform hona chahiye.

### Syntax

``` sql
SELECT column_names
FROM table_name
WHERE condition;
```

### Example 1: IT department ke employees

``` sql
SELECT *
FROM employees
WHERE department = 'IT';
```

### Example 2: Salary 50000 se zyada

``` sql
SELECT *
FROM employees
WHERE salary > 50000;
```

### Example 3: Jaipur ke employees

``` sql
SELECT *
FROM employees
WHERE city = 'Jaipur';
```

### Example 4: Multiple conditions using AND

``` sql
SELECT *
FROM employees
WHERE department = 'IT'
AND salary > 65000;
```

Dono conditions true honi chahiye.

### Example 5: Multiple conditions using OR

``` sql
SELECT *
FROM employees
WHERE city = 'Jaipur'
OR city = 'Delhi';
```

Koi ek condition true honi chahiye.

### Example 6: IN operator

``` sql
SELECT *
FROM employees
WHERE department IN ('IT', 'HR');
```

### Example 7: BETWEEN operator

``` sql
SELECT *
FROM employees
WHERE salary BETWEEN 40000 AND 60000;
```

`BETWEEN` dono boundary values ko include karta hai.

### Example 8: LIKE operator

``` sql
SELECT *
FROM employees
WHERE emp_name LIKE 'A%';
```

A se start hone wale employees.

------------------------------------------------------------------------

# Part 4: GROUP BY Clause

## 10. GROUP BY kya hota hai?

`GROUP BY` clause ka use same value wale records ko groups mein organize
karne ke liye hota hai.

Generally, iska use aggregate functions ke saath hota hai, taaki har
group ka summary result mil sake.

### Real-time example

Maan lijiye company mein 12 employees hain aur 4 departments hain:

-   IT
-   HR
-   Sales
-   Marketing

Agar humein department-wise employees count karna hai, to pehle
employees ko department ke basis par groups mein divide karenge aur phir
har group ki counting karenge.

### Syntax

``` sql
SELECT column_name, AGGREGATE_FUNCTION(column_name)
FROM table_name
GROUP BY column_name;
```

## 11. Department-wise employee count

``` sql
SELECT
    department,
    COUNT(*) AS total_employees
FROM employees
GROUP BY department;
```

### Expected output

  department     total_employees
  ------------ -----------------
  HR                           3
  IT                           4
  Marketing                    2
  Sales                        3

**Explanation:**

-   `GROUP BY department` same department wale employees ko group karta
    hai.
-   `COUNT(*)` har department ke employees count karta hai.
-   Result mein har department ki ek row aati hai.

## 12. Department-wise total salary

``` sql
SELECT
    department,
    SUM(salary) AS total_salary
FROM employees
GROUP BY department;
```

### Expected output

  department     total_salary
  ------------ --------------
  HR                   147000
  IT                   285000
  Marketing            113000
  Sales                155000

## 13. Department-wise average salary

``` sql
SELECT
    department,
    AVG(salary) AS average_salary
FROM employees
GROUP BY department;
```

### Expected output

  department     average_salary
  ------------ ----------------
  HR                      49000
  IT                      71250
  Marketing               56500
  Sales                51666.67

## 14. Department-wise minimum and maximum salary

``` sql
SELECT
    department,
    MIN(salary) AS minimum_salary,
    MAX(salary) AS maximum_salary
FROM employees
GROUP BY department;
```

## 15. City-wise employee count

``` sql
SELECT
    city,
    COUNT(*) AS total_employees
FROM employees
GROUP BY city;
```

### Expected output

  city       total_employees
  -------- -----------------
  Delhi                    4
  Jaipur                   5
  Mumbai                   3

## 16. Multiple columns using GROUP BY

Hum ek se zyada columns ke basis par bhi grouping kar sakte hain.

### Example: Department aur city-wise employee count

``` sql
SELECT
    department,
    city,
    COUNT(*) AS total_employees
FROM employees
GROUP BY department, city;
```

**Explanation:** Pehle department aur phir us department ke andar city
ke combination ke basis par groups bante hain.

For example:

-   IT + Jaipur
-   IT + Delhi
-   IT + Mumbai
-   HR + Jaipur
-   HR + Delhi

Har unique department-city combination ka separate result milega.

## 17. GROUP BY with ORDER BY

Department-wise total salary ko descending order mein display karna:

``` sql
SELECT
    department,
    SUM(salary) AS total_salary
FROM employees
GROUP BY department
ORDER BY total_salary DESC;
```

`ORDER BY` result ko sort karta hai; ye grouping perform nahi karta.

------------------------------------------------------------------------

# Part 5: HAVING Clause

## 18. HAVING kya hota hai?

`HAVING` clause ka use grouped results ko condition ke basis par filter
karne ke liye hota hai.

Jab humein aggregate function ke result par condition lagani ho, tab
generally `HAVING` use karte hain.

### Real-time example

Maan lijiye company ke 4 departments hain. Management ko sirf wahi
departments dekhne hain jinki total salary ₹1,50,000 se zyada hai.

Pehle department-wise total salary calculate hogi, phir us grouped
result ko filter karenge.

### Syntax

``` sql
SELECT column_name, AGGREGATE_FUNCTION(column_name)
FROM table_name
GROUP BY column_name
HAVING condition;
```

## 19. Total salary greater than 150000

``` sql
SELECT
    department,
    SUM(salary) AS total_salary
FROM employees
GROUP BY department
HAVING SUM(salary) > 150000;
```

### Expected output

  department     total_salary
  ------------ --------------
  IT                   285000
  Sales                155000

**Explanation:**

-   `GROUP BY` department-wise groups banata hai.
-   `SUM(salary)` har department ki total salary calculate karta hai.
-   `HAVING` sirf un departments ko show karta hai jinki total salary
    ₹1,50,000 se zyada hai.

## 20. Average salary greater than 55000

``` sql
SELECT
    department,
    AVG(salary) AS average_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 55000;
```

Expected departments: IT aur Marketing.

## 21. Departments having more than 2 employees

``` sql
SELECT
    department,
    COUNT(*) AS total_employees
FROM employees
GROUP BY department
HAVING COUNT(*) > 2;
```

Expected departments: HR, IT aur Sales.

## 22. HAVING with multiple conditions

``` sql
SELECT
    department,
    COUNT(*) AS total_employees,
    AVG(salary) AS average_salary
FROM employees
GROUP BY department
HAVING COUNT(*) > 2
AND AVG(salary) > 50000;
```

Dono aggregate conditions true honi chahiye.

------------------------------------------------------------------------

# Part 6: WHERE vs HAVING

## 23. WHERE aur HAVING mein difference

  -----------------------------------------------------------------------
  WHERE                               HAVING
  ----------------------------------- -----------------------------------
  Individual rows filter karta hai.   Grouped results filter karta hai.

  Grouping se pehle filter hota hai.  Grouping ke baad filter hota hai.

  Aggregate functions ke result par   Aggregate function conditions ke
  directly filter karne ke liye       liye use hota hai.
  generally use nahi hota.            

  `GROUP BY` ke bina bhi use kar      Commonly `GROUP BY` ke saath use
  sakte hain.                         hota hai.
  -----------------------------------------------------------------------

### WHERE example

Sirf Jaipur ke employees select karo:

``` sql
SELECT *
FROM employees
WHERE city = 'Jaipur';
```

### HAVING example

Sirf un departments ko show karo jinki average salary ₹50,000 se zyada
hai:

``` sql
SELECT
    department,
    AVG(salary) AS average_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;
```

## 24. WHERE and HAVING together

**Question:** Jaipur ke employees mein department-wise groups banao aur
sirf un departments ko show karo jahan employees ki average salary
₹50,000 se zyada hai.

``` sql
SELECT
    department,
    COUNT(*) AS total_employees,
    AVG(salary) AS average_salary
FROM employees
WHERE city = 'Jaipur'
GROUP BY department
HAVING AVG(salary) > 50000;
```

### Query ka working

1.  `FROM employees` table se data read hota hai.
2.  `WHERE city = 'Jaipur'` sirf Jaipur ke employees select karta hai.
3.  `GROUP BY department` department-wise groups banata hai.
4.  `COUNT(*)` aur `AVG(salary)` har group ka summary nikalte hain.
5.  `HAVING AVG(salary) > 50000` groups ko filter karta hai.

Expected output:

  department     total_employees   average_salary
  ------------ ----------------- ----------------
  IT                           1            60000
  Sales                        1            55000

------------------------------------------------------------------------

# Part 7: Combined real-time examples

## 25. Question: Har department mein total employees aur total salary find karo.

``` sql
SELECT
    department,
    COUNT(*) AS employee_count,
    SUM(salary) AS total_salary
FROM employees
GROUP BY department;
```

## 26. Question: Sirf IT aur HR departments ka average salary find karo.

``` sql
SELECT
    department,
    AVG(salary) AS average_salary
FROM employees
WHERE department IN ('IT', 'HR')
GROUP BY department;
```

## 27. Question: Un departments ko show karo jahan 3 ya usse zyada employees hain.

``` sql
SELECT
    department,
    COUNT(*) AS total_employees
FROM employees
GROUP BY department
HAVING COUNT(*) >= 3;
```

## 28. Question: Jaipur mein ₹50,000 se zyada salary wale employees ka department-wise count find karo.

``` sql
SELECT
    department,
    COUNT(*) AS total_employees
FROM employees
WHERE city = 'Jaipur'
AND salary > 50000
GROUP BY department;
```

## 29. Question: Un departments ko show karo jinki maximum salary ₹70,000 se zyada hai.

``` sql
SELECT
    department,
    MAX(salary) AS maximum_salary
FROM employees
GROUP BY department
HAVING MAX(salary) > 70000;
```

## 30. Question: Department-wise total salary ko highest se lowest order mein display karo.

``` sql
SELECT
    department,
    SUM(salary) AS total_salary
FROM employees
GROUP BY department
ORDER BY total_salary DESC;
```

## 31. Question: Un cities ko show karo jahan 3 se zyada employees hain.

``` sql
SELECT
    city,
    COUNT(*) AS total_employees
FROM employees
GROUP BY city
HAVING COUNT(*) > 3;
```

## 32. Question: Delhi ke employees ki average salary find karo.

``` sql
SELECT AVG(salary) AS average_salary
FROM employees
WHERE city = 'Delhi';
```

------------------------------------------------------------------------

# Part 8: SQL Query Logical Execution Order

## 33. SQL query internally kis order mein process hoti hai?

Query likhne ka order aur logical processing order alag hote hain.

SQL query ka simplified logical processing order:

1.  `FROM` -- Table identify hoti hai.
2.  `WHERE` -- Individual rows filter hoti hain.
3.  `GROUP BY` -- Rows groups mein divide hoti hain.
4.  `HAVING` -- Groups filter hote hain.
5.  `SELECT` -- Required columns aur expressions select hote hain.
6.  `ORDER BY` -- Result sort hota hai.
7.  `LIMIT` -- Result rows ki limit apply hoti hai.

Example:

``` sql
SELECT
    department,
    AVG(salary) AS average_salary
FROM employees
WHERE city = 'Jaipur'
GROUP BY department
HAVING AVG(salary) > 50000
ORDER BY average_salary DESC
LIMIT 5;
```

Is query ka logical flow:

`FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT`

**Note:** Ye logical processing order hai. Database optimizer actual
execution ko internally optimize kar sakta hai.

------------------------------------------------------------------------

# Part 9: Important rules and common mistakes

1.  `COUNT(*)` matching rows count karta hai.
2.  `COUNT(column)` NULL values ko count nahi karta.
3.  `SUM()` aur `AVG()` numeric columns ke liye use hote hain.
4.  `MIN()` aur `MAX()` values ka minimum aur maximum nikalte hain.
5.  `GROUP BY` same values wale records ko group karta hai.
6.  `HAVING` grouped result ko filter karta hai.
7.  `WHERE` aggregate calculation se pehle rows filter karta hai.
8.  `WHERE` aur `HAVING` ek hi query mein use kar sakte hain.
9.  Aggregate function ko `WHERE` mein directly use karne ke bajaye
    `HAVING` use karein.
10. `GROUP BY` ke saath non-aggregated selected columns ko grouping
    columns mein include karna chahiye, taaki query valid aur
    predictable rahe.
11. MySQL mein `ONLY_FULL_GROUP_BY` SQL mode enabled ho sakta hai. Is
    mode mein invalid grouping queries reject hoti hain.
12. `ORDER BY` sorting karta hai; `GROUP BY` grouping karta hai. Dono ka
    purpose alag hai.

### Common mistake

Incorrect approach:

``` sql
SELECT department, AVG(salary)
FROM employees
WHERE AVG(salary) > 50000
GROUP BY department;
```

Correct approach:

``` sql
SELECT department, AVG(salary) AS average_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;
```

Reason: `WHERE` individual rows filter karta hai; average salary
calculate hone ke baad groups filter karne ke liye `HAVING` use hota
hai.

------------------------------------------------------------------------

# Part 10: Practice questions for students

## A. Aggregate Functions

1.  Total employees count karo.
2.  Total salary calculate karo.
3.  Average salary calculate karo.
4.  Minimum salary find karo.
5.  Maximum salary find karo.
6.  Unique departments count karo.
7.  Unique cities count karo.
8.  Average employee experience calculate karo.

## B. WHERE

9.  IT department ke employees display karo.
10. ₹50,000 se zyada salary wale employees display karo.
11. Jaipur ke employees display karo.
12. Delhi aur Mumbai ke employees display karo.
13. 3 se 6 years experience wale employees display karo.
14. A se start hone wale employees find karo.

## C. GROUP BY

15. Department-wise employee count find karo.
16. Department-wise total salary find karo.
17. Department-wise average salary find karo.
18. Department-wise minimum salary find karo.
19. Department-wise maximum salary find karo.
20. City-wise employee count find karo.
21. Department aur city-wise employee count find karo.

## D. HAVING

22. Sirf un departments ko display karo jahan 2 se zyada employees hain.
23. Un departments ko display karo jinki average salary ₹50,000 se zyada
    hai.
24. Un departments ko display karo jinki total salary ₹1,50,000 se zyada
    hai.
25. Un cities ko display karo jahan 3 ya usse zyada employees hain.
26. Un departments ko display karo jahan minimum salary ₹45,000 se zyada
    hai.

## E. Combined queries

27. Jaipur ke employees ka department-wise count find karo.
28. IT aur HR departments ki average salary find karo.
29. Jaipur ke employees mein un departments ko find karo jinki average
    salary ₹50,000 se zyada hai.
30. Un departments ko find karo jahan employees 2 se zyada hain aur
    average salary ₹50,000 se zyada hai.
31. Department-wise total salary ko descending order mein display karo.
32. Mumbai ke employees ki department-wise average salary find karo.

------------------------------------------------------------------------

# Quick Revision

  Concept               Simple meaning
  --------------------- -------------------------------
  Aggregate Functions   Multiple rows par calculation
  `COUNT()`             Counting
  `SUM()`               Total
  `AVG()`               Average
  `MIN()`               Minimum value
  `MAX()`               Maximum value
  `WHERE`               Individual rows filter
  `GROUP BY`            Similar rows ke groups
  `HAVING`              Groups filter
  `ORDER BY`            Sorting

## One-line memory trick

-   `WHERE` = Kaun-se records chahiye?
-   `GROUP BY` = Records ke groups kaise banane hain?
-   Aggregate Functions = Har group ka calculation kya hoga?
-   `HAVING` = Kaun-se groups chahiye?
-   `ORDER BY` = Result kis order mein chahiye?

**Remember:** `WHERE` rows ko filter karta hai, `GROUP BY` rows ko group
karta hai, Aggregate Functions summary calculate karti hain aur `HAVING`
groups ko filter karta hai.
