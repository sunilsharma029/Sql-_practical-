# MySQL Complete Notes: Window Functions and Date & Time Functions

**Database:** MySQL 8.0+\
**Approach:** What, Why, When, How + real-time examples\
**Practice database:** Employee Management System

> Important: MySQL window functions are available in MySQL 8.0 and
> later. Examples below use MySQL syntax.

------------------------------------------------------------------------

# Part 1: Setup -- Employee Management Database

``` sql
CREATE DATABASE sql_advanced_db;
USE sql_advanced_db;

CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(50),
    department VARCHAR(50),
    city VARCHAR(50),
    salary DECIMAL(10,2),
    joining_date DATE
);

INSERT INTO employees
(emp_id, emp_name, department, city, salary, joining_date)
VALUES
(1, 'Rahul', 'IT', 'Jaipur', 60000, '2021-01-15'),
(2, 'Priya', 'HR', 'Delhi', 45000, '2022-03-10'),
(3, 'Amit', 'IT', 'Jaipur', 75000, '2020-06-20'),
(4, 'Neha', 'Sales', 'Mumbai', 40000, '2023-02-05'),
(5, 'Rohit', 'HR', 'Jaipur', 50000, '2021-11-18'),
(6, 'Pooja', 'IT', 'Delhi', 80000, '2019-08-12'),
(7, 'Karan', 'Sales', 'Jaipur', 55000, '2022-07-25'),
(8, 'Anjali', 'Marketing', 'Mumbai', 65000, '2020-04-14'),
(9, 'Vikas', 'IT', 'Mumbai', 70000, '2021-09-01'),
(10, 'Sneha', 'Marketing', 'Jaipur', 48000, '2023-05-16'),
(11, 'Manish', 'HR', 'Delhi', 52000, '2020-12-08'),
(12, 'Riya', 'Sales', 'Delhi', 60000, '2021-06-30');

SELECT * FROM employees;
```

# Part 2: Window Functions

## 1. Window Function kya hota hai?

Window Function ek SQL function hai jo related rows ke ek group (window)
par calculation perform karta hai, lekin har individual row ko result
mein retain karta hai.

Simple words mein:

-   Aggregate function with `GROUP BY` generally multiple rows ko
    summarize karke ek row per group deta hai.
-   Window function calculation karta hai, lekin original rows ko hide
    ya collapse nahi karta.

### Example: GROUP BY vs Window Function

**GROUP BY:**

``` sql
SELECT
    department,
    AVG(salary) AS average_salary
FROM employees
GROUP BY department;
```

Ismein har department ki ek row milegi.

**Window Function:**

``` sql
SELECT
    emp_name,
    department,
    salary,
    AVG(salary) OVER (
        PARTITION BY department
    ) AS department_average
FROM employees;
```

Ismein har employee ki row rahegi aur saath mein uske department ki
average salary bhi show hogi.

## 2. Window Function ki need kyu padi?

Maan lijiye HR manager ko ye information chahiye:

-   Har employee ki salary.
-   Uske department ki average salary.
-   Department mein employee ki salary rank.
-   Previous employee ki salary.
-   Next employee ki salary.
-   Running total salary.

Sirf `GROUP BY` se in sabhi details ko ek hi result mein retain karna
difficult hai. Window Functions individual rows aur group-level
calculations ko saath display karne mein help karti hain.

## 3. Window Function ka syntax

``` sql
function_name(expression)
OVER (
    PARTITION BY column_name
    ORDER BY column_name
    ROWS BETWEEN ... AND ...
)
```

Har part ka meaning:

  -----------------------------------------------------------------------
  Part                                Meaning
  ----------------------------------- -----------------------------------
  Function                            Kaunsi calculation karni hai

  `OVER()`                            Window define karta hai

  `PARTITION BY`                      Rows ko groups mein divide karta
                                      hai

  `ORDER BY`                          Window ke andar rows ka order
                                      decide karta hai

  Frame (`ROWS`/`RANGE`)              Current row ke relative kaun-si
                                      rows calculation mein include hongi
  -----------------------------------------------------------------------

`PARTITION BY`, `ORDER BY` aur frame har query mein required nahi hote.
Ye function aur requirement par depend karta hai.

## 4. PARTITION BY kya hai?

`PARTITION BY` rows ko groups mein divide karta hai. Har group ke liye
window calculation independently hoti hai.

Example: Department-wise average salary, lekin har employee ki row
retain karni hai.

``` sql
SELECT
    emp_name,
    department,
    salary,
    AVG(salary) OVER (
        PARTITION BY department
    ) AS department_avg_salary
FROM employees;
```

**Explanation:**

-   IT employees ka separate partition.
-   HR employees ka separate partition.
-   Sales employees ka separate partition.
-   Marketing employees ka separate partition.
-   Har partition ki average salary us department ki har row ke saath
    show hogi.

## 5. ORDER BY inside OVER()

`ORDER BY` window ke andar rows ka logical order decide karta hai.

Example: Salary ke basis par employee ranking.

``` sql
SELECT
    emp_name,
    salary,
    ROW_NUMBER() OVER (
        ORDER BY salary DESC
    ) AS row_num
FROM employees;
```

Highest salary wale employee ko row number 1 milega.

**Important:** `ORDER BY` inside `OVER()` window calculation ka order
set karta hai. Final output ko sort karne ke liye outer query mein alag
`ORDER BY` likhein.

------------------------------------------------------------------------

# Part 3: Important Window Functions

## 6. ROW_NUMBER()

### What?

`ROW_NUMBER()` har row ko unique sequential number assign karta hai.

### Why / When?

-   Employees ko serial number dena.
-   Har department mein employee numbering.
-   Duplicate records mein se ek row select karna.
-   Top-N records nikalna.

### Example 1: Overall row numbering

``` sql
SELECT
    emp_name,
    department,
    salary,
    ROW_NUMBER() OVER (
        ORDER BY salary DESC
    ) AS row_num
FROM employees;
```

Har row ko unique number milega. Equal salary hone par unke order ko
deterministic banane ke liye additional column, jaise `emp_id`, add
karna useful hai.

### Example 2: Department-wise row numbering

``` sql
SELECT
    emp_name,
    department,
    salary,
    ROW_NUMBER() OVER (
        PARTITION BY department
        ORDER BY salary DESC, emp_id
    ) AS department_row_num
FROM employees;
```

Har department mein numbering 1 se restart hogi.

## 7. RANK()

### What?

`RANK()` rows ko rank assign karta hai. Equal values ko same rank milti
hai aur next rank skip hoti hai.

### Example

``` sql
SELECT
    emp_name,
    salary,
    RANK() OVER (
        ORDER BY salary DESC
    ) AS salary_rank
FROM employees;
```

Agar salaries 80000, 75000, 75000, 60000 hon, ranks hongi:

    Salary   Rank
  -------- ------
     80000      1
     75000      2
     75000      2
     60000      4

## 8. DENSE_RANK()

### What?

`DENSE_RANK()` bhi equal values ko same rank deta hai, lekin next rank
skip nahi karta.

Example:

``` sql
SELECT
    emp_name,
    salary,
    DENSE_RANK() OVER (
        ORDER BY salary DESC
    ) AS dense_salary_rank
FROM employees;
```

Agar salaries 80000, 75000, 75000, 60000 hon:

    Salary   Dense Rank
  -------- ------------
     80000            1
     75000            2
     75000            2
     60000            3

### ROW_NUMBER vs RANK vs DENSE_RANK

  Function         Equal values            Next rank
  ---------------- ----------------------- -----------
  `ROW_NUMBER()`   Different row numbers   No gap
  `RANK()`         Same rank               Gap
  `DENSE_RANK()`   Same rank               No gap

## 9. Department-wise salary ranking

``` sql
SELECT
    emp_name,
    department,
    salary,
    RANK() OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) AS department_rank
FROM employees;
```

Har department mein highest salary ko rank 1 milegi.

## 10. LAG()

### What?

`LAG()` current row se pehle wali row ka value access karne ke liye use
hota hai, according to the window ordering.

### Why / When?

-   Previous month ki sales compare karna.
-   Previous day's stock compare karna.
-   Previous employee record ki value compare karna.
-   Salary history compare karna.

### Syntax

``` sql
LAG(expression, offset, default_value)
OVER (
    PARTITION BY column_name
    ORDER BY column_name
)
```

`offset` default 1 hota hai. `default_value` optional hai.

### Example

``` sql
SELECT
    emp_name,
    salary,
    LAG(salary) OVER (
        ORDER BY emp_id
    ) AS previous_salary
FROM employees;
```

First row ke paas previous row nahi hoti, isliye result NULL aayega.

### Example: Current vs previous salary difference

``` sql
SELECT
    emp_name,
    salary,
    LAG(salary) OVER (
        ORDER BY emp_id
    ) AS previous_salary,
    salary - LAG(salary) OVER (
        ORDER BY emp_id
    ) AS salary_difference
FROM employees;
```

Ye example employee ID ke order mein adjacent rows compare karta hai; ye
kisi employee ki actual historical salary change report nahi hai.

## 11. LEAD()

### What?

`LEAD()` current row ke baad wali row ka value access karta hai.

### Example

``` sql
SELECT
    emp_name,
    salary,
    LEAD(salary) OVER (
        ORDER BY emp_id
    ) AS next_salary
FROM employees;
```

Last row ke paas next row nahi hoti, isliye result NULL aayega.

### LAG vs LEAD

  LAG                        LEAD
  -------------------------- ------------------------
  Previous row ka value      Next row ka value
  Peeche ki row dekhta hai   Aage ki row dekhta hai

## 12. FIRST_VALUE()

Window frame ke according first row ka expression value return karta
hai.

Example: Har department ki highest salary employee ka salary value har
row ke saath show karna.

``` sql
SELECT
    emp_name,
    department,
    salary,
    FIRST_VALUE(salary) OVER (
        PARTITION BY department
        ORDER BY salary DESC, emp_id
    ) AS highest_salary_in_department
FROM employees;
```

## 13. LAST_VALUE()

Window frame ke according last row ka expression value return karta hai.

``` sql
SELECT
    emp_name,
    department,
    salary,
    LAST_VALUE(salary) OVER (
        PARTITION BY department
        ORDER BY salary DESC, emp_id
        ROWS BETWEEN UNBOUNDED PRECEDING
        AND UNBOUNDED FOLLOWING
    ) AS lowest_salary_in_department
FROM employees;
```

**Important:** `LAST_VALUE()` ke result par window frame ka effect hota
hai. Agar poore partition ki last value chahiye, to
`UNBOUNDED FOLLOWING` specify karna useful hai.

## 14. Running total using SUM() OVER()

Window Function ke roop mein aggregate functions bhi use kar sakte hain.

Example: Employee ID order mein running salary total.

``` sql
SELECT
    emp_id,
    emp_name,
    salary,
    SUM(salary) OVER (
        ORDER BY emp_id
        ROWS BETWEEN UNBOUNDED PRECEDING
        AND CURRENT ROW
    ) AS running_total
FROM employees;
```

Har row ka result us row tak ki cumulative salary hogi.

### Department-wise running total

``` sql
SELECT
    emp_name,
    department,
    salary,
    SUM(salary) OVER (
        PARTITION BY department
        ORDER BY emp_id
        ROWS BETWEEN UNBOUNDED PRECEDING
        AND CURRENT ROW
    ) AS department_running_total
FROM employees;
```

## 15. Department-wise count, average and maximum using window functions

``` sql
SELECT
    emp_name,
    department,
    salary,
    COUNT(*) OVER (
        PARTITION BY department
    ) AS department_employee_count,
    AVG(salary) OVER (
        PARTITION BY department
    ) AS department_average_salary,
    MAX(salary) OVER (
        PARTITION BY department
    ) AS department_max_salary
FROM employees;
```

Ismein har employee ki row ke saath department-level summary bhi milegi.

## 16. Top 2 highest-paid employees from each department

Window Function ka ek important real-world use-case.

``` sql
WITH ranked_employees AS (
    SELECT
        emp_id,
        emp_name,
        department,
        salary,
        ROW_NUMBER() OVER (
            PARTITION BY department
            ORDER BY salary DESC, emp_id
        ) AS rn
    FROM employees
)
SELECT *
FROM ranked_employees
WHERE rn <= 2;
```

**Explanation:**

1.  CTE mein har department ke employees ko salary ke basis par number
    diya.
2.  Outer query mein sirf row number 1 aur 2 select kiye.
3.  Har department ke maximum 2 employees result mein aaye.

Agar ties ke saath top 2 salary positions chahiye, to `ROW_NUMBER()` ki
jagah `DENSE_RANK()` use karein.

## 17. Window frame kya hota hai?

Window frame define karta hai ki current row ke relative kaun-si rows
calculation mein include hongi.

Example:

``` sql
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

Meaning: - `UNBOUNDED PRECEDING` = Partition ki first row se start. -
`CURRENT ROW` = Current row tak calculate.

Common frame keywords:

  Keyword                 Meaning
  ----------------------- -----------------------------
  `UNBOUNDED PRECEDING`   Partition ki first row
  `n PRECEDING`           Current row se n rows pehle
  `CURRENT ROW`           Current row
  `n FOLLOWING`           Current row se n rows aage
  `UNBOUNDED FOLLOWING`   Partition ki last row

Example: Current employee aur usse pehle ke 2 employees ka moving
average:

``` sql
SELECT
    emp_id,
    emp_name,
    salary,
    AVG(salary) OVER (
        ORDER BY emp_id
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_average
FROM employees;
```

------------------------------------------------------------------------

# Part 4: Date and Time Functions

## 18. Date and Time Functions kya hoti hain?

MySQL Date and Time Functions ka use date aur time values ko store,
display, compare, calculate aur format karne ke liye hota hai.

### Real-time use-cases

-   Employee ne company kab join ki?
-   Employee ko company mein kitne saal ho gaye?
-   Order kis date ko place hua?
-   Last 7 days ke orders kitne hain?
-   Current date aur time kya hai?
-   Delivery date se kitne din baaki hain?
-   Month-wise sales report kaise banayein?

## 19. Important MySQL date/time data types

  -----------------------------------------------------------------------
  Data type                           Use
  ----------------------------------- -----------------------------------
  `DATE`                              Date only: `2026-11-02`

  `TIME`                              Time/duration: `14:30:00`

  `DATETIME`                          Date and time:
                                      `2026-11-02 14:30:00`

  `TIMESTAMP`                         Date and time, with
                                      time-zone-related conversion
                                      behaviour

  `YEAR`                              Year value
  -----------------------------------------------------------------------

**DATE vs DATETIME vs TIMESTAMP:**

-   `DATE` mein time store nahi hota.
-   `DATETIME` date aur time store karta hai, aur MySQL session time
    zone conversion automatically apply nahi karta.
-   `TIMESTAMP` values ko UTC ke roop mein store karke session time zone
    ke according retrieve/display karta hai.

------------------------------------------------------------------------

# Part 5: Current Date and Time Functions

## 20. NOW()

Current date aur time return karta hai.

``` sql
SELECT NOW();
```

Example output format:

`2026-11-02 14:30:00`

Actual output current server/session time par depend karega.

## 21. CURDATE()

Current date return karta hai.

``` sql
SELECT CURDATE();
```

Example:

`2026-11-02`

## 22. CURRENT_DATE()

Current date return karta hai.

``` sql
SELECT CURRENT_DATE();
```

`CURDATE()` aur `CURRENT_DATE()` equivalent forms hain.

## 23. CURTIME()

Current time return karta hai.

``` sql
SELECT CURTIME();
```

Example:

`14:30:00`

## 24. CURRENT_TIME()

Current time return karta hai.

``` sql
SELECT CURRENT_TIME();
```

## 25. CURRENT_TIMESTAMP()

Current date aur time return karta hai.

``` sql
SELECT CURRENT_TIMESTAMP();
```

`NOW()` aur `CURRENT_TIMESTAMP()` current statement ka date-time return
karte hain.

------------------------------------------------------------------------

# Part 6: Extracting Date Parts

## 26. YEAR()

Date se year extract karta hai.

``` sql
SELECT
    emp_name,
    joining_date,
    YEAR(joining_date) AS joining_year
FROM employees;
```

## 27. MONTH()

Date se month number (1--12) return karta hai.

``` sql
SELECT
    emp_name,
    MONTH(joining_date) AS joining_month
FROM employees;
```

## 28. MONTHNAME()

Month ka full name return karta hai.

``` sql
SELECT
    emp_name,
    MONTHNAME(joining_date) AS month_name
FROM employees;
```

## 29. DAY()

Month ka day number return karta hai.

``` sql
SELECT
    emp_name,
    DAY(joining_date) AS joining_day
FROM employees;
```

## 30. DAYNAME()

Weekday ka name return karta hai.

``` sql
SELECT
    emp_name,
    DAYNAME(joining_date) AS weekday_name
FROM employees;
```

## 31. DAYOFWEEK()

Weekday ka number return karta hai:

-   Sunday = 1
-   Monday = 2
-   ...
-   Saturday = 7

``` sql
SELECT DAYOFWEEK('2026-11-02');
```

## 32. DAYOFMONTH()

Month ka day number return karta hai (1--31).

``` sql
SELECT DAYOFMONTH(joining_date)
FROM employees;
```

## 33. DAYOFYEAR()

Year ka day number return karta hai (1--366).

``` sql
SELECT DAYOFYEAR('2026-11-02');
```

## 34. WEEK()

Date ka week number return karta hai. Week numbering mode ke according
vary kar sakti hai.

``` sql
SELECT WEEK('2026-11-02');
```

## 35. QUARTER()

Year ka quarter number return karta hai:

-   Q1 = January--March
-   Q2 = April--June
-   Q3 = July--September
-   Q4 = October--December

``` sql
SELECT QUARTER('2026-11-02');
```

Result: `4`

## 36. EXTRACT()

Date/time value ka specific part extract karta hai.

``` sql
SELECT
    EXTRACT(YEAR FROM joining_date) AS join_year,
    EXTRACT(MONTH FROM joining_date) AS join_month,
    EXTRACT(DAY FROM joining_date) AS join_day
FROM employees;
```

------------------------------------------------------------------------

# Part 7: Date Calculation Functions

## 37. DATE_ADD()

Date mein days, months, years ya other intervals add karta hai.

### 10 days add

``` sql
SELECT DATE_ADD('2026-11-02', INTERVAL 10 DAY);
```

### 2 months add

``` sql
SELECT DATE_ADD('2026-11-02', INTERVAL 2 MONTH);
```

### 1 year add

``` sql
SELECT DATE_ADD('2026-11-02', INTERVAL 1 YEAR);
```

### Employee joining date se 1 year baad

``` sql
SELECT
    emp_name,
    joining_date,
    DATE_ADD(joining_date, INTERVAL 1 YEAR) AS first_anniversary
FROM employees;
```

## 38. DATE_SUB()

Date se interval subtract karta hai.

``` sql
SELECT DATE_SUB('2026-11-02', INTERVAL 7 DAY);
```

### Last 7 days ka date cutoff

``` sql
SELECT DATE_SUB(CURDATE(), INTERVAL 7 DAY);
```

## 39. DATEDIFF()

Do dates ke beech days ka difference return karta hai.

``` sql
SELECT DATEDIFF('2026-11-02', '2026-10-25');
```

Result: `8`

**Important:** `DATEDIFF(date1, date2)` sirf date parts use karta hai,
time part ignore karta hai. Result `date1 - date2` ke days hota hai.

### Employee ko join kiye kitne din hue?

``` sql
SELECT
    emp_name,
    joining_date,
    DATEDIFF(CURDATE(), joining_date) AS days_in_company
FROM employees;
```

## 40. TIMESTAMPDIFF()

Do date/time values ke beech difference specified unit mein return karta
hai.

### Completed years

``` sql
SELECT
    emp_name,
    joining_date,
    TIMESTAMPDIFF(YEAR, joining_date, CURDATE()) AS completed_years
FROM employees;
```

### Difference in months

``` sql
SELECT TIMESTAMPDIFF(
    MONTH,
    '2025-01-01',
    '2026-11-02'
) AS total_months;
```

### Difference in hours

``` sql
SELECT TIMESTAMPDIFF(
    HOUR,
    '2026-11-01 10:00:00',
    '2026-11-02 14:00:00'
) AS total_hours;
```

**DATEDIFF vs TIMESTAMPDIFF:**

-   `DATEDIFF()` dates ke beech days ka difference deta hai.
-   `TIMESTAMPDIFF()` selected unit mein difference deta hai, jaise
    YEAR, MONTH, DAY, HOUR, MINUTE, SECOND.

------------------------------------------------------------------------

# Part 8: Date Formatting and Conversion

## 41. DATE_FORMAT()

Date ko desired display format mein convert karta hai.

``` sql
SELECT
    emp_name,
    DATE_FORMAT(joining_date, '%d-%m-%Y') AS formatted_date
FROM employees;
```

Example:

`15-01-2021`

### Common format specifiers

  Specifier   Meaning                    Example
  ----------- -------------------------- ----------
  `%Y`        Four-digit year            2026
  `%y`        Two-digit year             26
  `%m`        Month number               11
  `%M`        Full month name            November
  `%b`        Short month name           Nov
  `%d`        Day with leading zero      02
  `%e`        Day without leading zero   2
  `%W`        Full weekday name          Monday
  `%a`        Short weekday name         Mon
  `%H`        Hour (00--23)              14
  `%h`        Hour (01--12)              02
  `%i`        Minutes                    30
  `%s`        Seconds                    00
  `%p`        AM/PM                      PM

### Example: Human-readable date and time

``` sql
SELECT DATE_FORMAT(
    NOW(),
    '%W, %d %M %Y - %h:%i %p'
) AS readable_datetime;
```

## 42. STR_TO_DATE()

String ko specified format ke according DATE/DATETIME value mein parse
karta hai.

``` sql
SELECT STR_TO_DATE('02-11-2026', '%d-%m-%Y');
```

### Example

``` sql
SELECT STR_TO_DATE(
    '02/11/2026 02:30 PM',
    '%d/%m/%Y %h:%i %p'
);
```

`DATE_FORMAT()` display formatting ke liye hai; `STR_TO_DATE()` string
input ko date/time value mein parse karta hai.

## 43. TIME_FORMAT()

Time value ko specified format mein display karta hai.

``` sql
SELECT TIME_FORMAT('14:30:45', '%h:%i %p');
```

## 44. TIME_TO_SEC()

Time value ko seconds mein convert karta hai.

``` sql
SELECT TIME_TO_SEC('01:30:00');
```

Result: `5400`

## 45. SEC_TO_TIME()

Seconds ko time format mein convert karta hai.

``` sql
SELECT SEC_TO_TIME(5400);
```

Result: `01:30:00`

------------------------------------------------------------------------

# Part 9: Useful Date/Time Functions

## 46. LAST_DAY()

Month ki last date return karta hai.

``` sql
SELECT LAST_DAY('2026-11-02');
```

Result: `2026-11-30`

## 47. MAKEDATE()

Year aur day-of-year se date create karta hai.

``` sql
SELECT MAKEDATE(2026, 100);
```

## 48. MAKETIME()

Hour, minute aur second se time create karta hai.

``` sql
SELECT MAKETIME(14, 30, 45);
```

Result: `14:30:45`

## 49. UNIX_TIMESTAMP()

Date/time ko Unix timestamp (seconds since Unix epoch) mein convert
karta hai.

``` sql
SELECT UNIX_TIMESTAMP();
```

## 50. FROM_UNIXTIME()

Unix timestamp ko date/time representation mein convert karta hai.

``` sql
SELECT FROM_UNIXTIME(1760000000);
```

Output session time zone par depend kar sakta hai.

------------------------------------------------------------------------

# Part 10: Real-time Date/Time Examples

## 51. Current month mein join hue employees

``` sql
SELECT *
FROM employees
WHERE YEAR(joining_date) = YEAR(CURDATE())
AND MONTH(joining_date) = MONTH(CURDATE());
```

## 52. 2021 mein join hue employees

``` sql
SELECT *
FROM employees
WHERE YEAR(joining_date) = 2021;
```

## 53. Last 30 days mein join hue employees

``` sql
SELECT *
FROM employees
WHERE joining_date >= DATE_SUB(CURDATE(), INTERVAL 30 DAY)
AND joining_date < DATE_ADD(CURDATE(), INTERVAL 1 DAY);
```

## 54. Joining date ke basis par employees ko sort karna

``` sql
SELECT *
FROM employees
ORDER BY joining_date DESC;
```

## 55. Month-wise employee joining count

``` sql
SELECT
    YEAR(joining_date) AS join_year,
    MONTH(joining_date) AS join_month,
    COUNT(*) AS total_employees
FROM employees
GROUP BY YEAR(joining_date), MONTH(joining_date)
ORDER BY join_year, join_month;
```

## 56. Weekday-wise employee joining count

``` sql
SELECT
    DAYNAME(joining_date) AS joining_day,
    COUNT(*) AS total_employees
FROM employees
GROUP BY DAYOFWEEK(joining_date), DAYNAME(joining_date)
ORDER BY DAYOFWEEK(joining_date);
```

## 57. Employees with more than 5 completed years in company

``` sql
SELECT
    emp_name,
    joining_date,
    TIMESTAMPDIFF(YEAR, joining_date, CURDATE()) AS completed_years
FROM employees
WHERE TIMESTAMPDIFF(YEAR, joining_date, CURDATE()) > 5;
```

## 58. Upcoming first anniversary date

``` sql
SELECT
    emp_name,
    joining_date,
    DATE_ADD(joining_date, INTERVAL 1 YEAR) AS first_anniversary
FROM employees;
```

This calculates the first anniversary after the original joining date,
not the next upcoming annual anniversary for employees who joined many
years ago.

------------------------------------------------------------------------

# Part 11: Window Functions + Date Functions Together

## 59. Employees ko joining date ke basis par department-wise rank karna

``` sql
SELECT
    emp_name,
    department,
    joining_date,
    ROW_NUMBER() OVER (
        PARTITION BY department
        ORDER BY joining_date
    ) AS joining_order
FROM employees;
```

## 60. Month-wise joining count aur running total

``` sql
WITH monthly_joinings AS (
    SELECT
        DATE_FORMAT(joining_date, '%Y-%m') AS join_month,
        COUNT(*) AS total_joinings
    FROM employees
    GROUP BY DATE_FORMAT(joining_date, '%Y-%m')
)
SELECT
    join_month,
    total_joinings,
    SUM(total_joinings) OVER (
        ORDER BY join_month
        ROWS BETWEEN UNBOUNDED PRECEDING
        AND CURRENT ROW
    ) AS cumulative_joinings
FROM monthly_joinings;
```

## 61. Previous employee joining date

``` sql
SELECT
    emp_name,
    joining_date,
    LAG(joining_date) OVER (
        ORDER BY joining_date
    ) AS previous_joining_date
FROM employees;
```

------------------------------------------------------------------------

# Part 12: Important Differences and Rules

## 62. GROUP BY vs PARTITION BY

  -----------------------------------------------------------------------
  GROUP BY                            PARTITION BY in a window
  ----------------------------------- -----------------------------------
  Rows ko groups mein summarize karta Calculation ke liye rows ke
  hai.                                partitions banata hai.

  Usually one result row per group.   Original individual rows retain
                                      hoti hain.

  Aggregate functions ke saath        Window functions ke saath use hota
  common.                             hai.
  -----------------------------------------------------------------------

## 63. Important window function rules

1.  Window functions ke liye MySQL 8.0+ use karein.
2.  Window functions `OVER()` clause ke saath use hoti hain.
3.  `PARTITION BY` optional hai.
4.  `ORDER BY` window ke andar ranking aur sequence-based calculations
    ke liye important hai.
5.  Window function result ko filter karne ke liye CTE ya derived table
    use kar sakte hain.
6.  Ranking mein ties handle karne ke liye `ROW_NUMBER()`, `RANK()` aur
    `DENSE_RANK()` ka difference samjhein.
7.  `LAST_VALUE()` ke saath window frame carefully specify karein.
8.  Running totals ke liye explicit `ROWS` frame likhna clear aur
    predictable hota hai.

## 64. Important date/time rules

1.  MySQL date literal ka standard format `YYYY-MM-DD` hai.
2.  `NOW()` current date aur time return karta hai.
3.  `CURDATE()` sirf current date return karta hai.
4.  `CURTIME()` sirf current time return karta hai.
5.  `DATEDIFF()` time part ignore karta hai.
6.  `TIMESTAMPDIFF()` selected unit mein difference return karta hai.
7.  `DATE_FORMAT()` display formatting ke liye hai.
8.  `STR_TO_DATE()` string ko date/time mein parse karta hai.
9.  `TIMESTAMP` aur `DATETIME` ka time-zone behaviour alag hai.
10. Date filtering mein indexed column ko direct range condition dena
    aksar function-based filtering se better hota hai.

### Better date range filtering

Instead of:

``` sql
SELECT *
FROM employees
WHERE YEAR(joining_date) = 2021;
```

Prefer:

``` sql
SELECT *
FROM employees
WHERE joining_date >= '2021-01-01'
AND joining_date < '2022-01-01';
```

Is tarah indexed `joining_date` column ka range access use karna easier
ho sakta hai.

------------------------------------------------------------------------

# Part 13: Student Practice Questions

## A. Window Functions

1.  All employees ko salary ke descending order mein `ROW_NUMBER()`
    assign karein.
2.  Har department mein employees ko salary ke basis par rank karein.
3.  `RANK()` aur `DENSE_RANK()` ka output compare karein.
4.  Har department ki average salary har employee ke saath show karein.
5.  Har department ka highest salary value `FIRST_VALUE()` se show
    karein.
6.  Har department ka lowest salary value `LAST_VALUE()` se show karein.
7.  Employee ID order mein previous salary `LAG()` se show karein.
8.  Employee ID order mein next salary `LEAD()` se show karein.
9.  Department-wise running salary total calculate karein.
10. Har department ke top 2 employees find karein.
11. Department-wise employee count window function se calculate karein.
12. Employee ID order mein 3-row moving average calculate karein.

## B. Date and Time Functions

13. Current date display karein.
14. Current time display karein.
15. Current date and time display karein.
16. Employees ki joining year extract karein.
17. Employees ki joining month ka name display karein.
18. Joining date ka weekday display karein.
19. Joining date se ab tak total days calculate karein.
20. Employees ke completed service years calculate karein.
21. Joining date mein 6 months add karein.
22. Joining date se 30 days subtract karein.
23. Date ko `DD-MM-YYYY` format mein display karein.
24. String `25-12-2026` ko date mein convert karein.
25. Month-wise employee joining count calculate karein.
26. Weekday-wise joining count calculate karein.
27. Last 30 days mein join hue employees find karein.
28. Har employee ki first anniversary date calculate karein.

## C. Combined Questions

29. Har department mein earliest joining wale employee ko rank 1 dein.
30. Month-wise joining count aur cumulative joining count show karein.
31. Department-wise joining date ke basis par previous employee ki
    joining date show karein.
32. Har department ke top 2 highest-paid employees ke naam, salary aur
    joining date show karein.

------------------------------------------------------------------------

# Quick Revision

## Window Functions

  Function           Purpose
  ------------------ --------------------------
  `ROW_NUMBER()`     Unique sequential number
  `RANK()`           Ranking with gaps
  `DENSE_RANK()`     Ranking without gaps
  `LAG()`            Previous row value
  `LEAD()`           Next row value
  `FIRST_VALUE()`    First value in frame
  `LAST_VALUE()`     Last value in frame
  `SUM() OVER()`     Running/cumulative sum
  `AVG() OVER()`     Window average
  `COUNT() OVER()`   Window count

## Date and Time Functions

  Function            Purpose
  ------------------- -----------------------------
  `NOW()`             Current date and time
  `CURDATE()`         Current date
  `CURTIME()`         Current time
  `YEAR()`            Extract year
  `MONTH()`           Extract month
  `DAY()`             Extract day
  `DAYNAME()`         Weekday name
  `MONTHNAME()`       Month name
  `DATE_ADD()`        Add interval
  `DATE_SUB()`        Subtract interval
  `DATEDIFF()`        Difference in days
  `TIMESTAMPDIFF()`   Difference in selected unit
  `DATE_FORMAT()`     Format date/time
  `STR_TO_DATE()`     Parse string into date/time
  `LAST_DAY()`        Last date of month
  `EXTRACT()`         Extract a date/time part

**Remember:**

-   Window Function = Calculation + Original rows retained.
-   `PARTITION BY` = Window ke groups.
-   `ORDER BY` = Window ke andar sequence.
-   `LAG` = Previous row.
-   `LEAD` = Next row.
-   Date Functions = Date/time ko extract, calculate, compare aur format
    karna.
