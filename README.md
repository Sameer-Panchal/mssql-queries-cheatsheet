1) Delete duplicate record from table (keep one per Col_Name)
   
WITH CTE AS (
  SELECT *,
         ROW_NUMBER() OVER (PARTITION BY Col_Name ORDER BY (SELECT 0)) AS New_Rank
  FROM dbo.Table_Name
)
DELETE FROM CTE
WHERE New_Rank > 1;


2) Identify duplicate record values in a column

SELECT Col_Name, COUNT(*) AS Cnt
FROM dbo.Table_Name
GROUP BY Col_Name
HAVING COUNT(*) > 1;


3) Employees who joined in year 2020

SELECT *
FROM dbo.Employees
WHERE Join_Date >= '2020-01-01' AND Join_Date < '2021-01-01';


4) Employees who do not have a manager (self-join style check)

SELECT *
FROM dbo.Employees
WHERE Manager_ID IS NULL;


5) Department with highest number of employees

SELECT TOP (1) Department, COUNT(*) AS employee_count
FROM dbo.Employees
GROUP BY Department
ORDER BY employee_count DESC;


6) Employees having highest salary in each department

WITH ranked AS (
  SELECT Department, EmployeeID, Salary,
         DENSE_RANK() OVER (PARTITION BY Department ORDER BY Salary DESC) AS Rank_New
  FROM dbo.Employees
)
SELECT Department, EmployeeID, Salary
FROM ranked
WHERE Rank_New = 1;


7) Fetch 1st and last record from table by a chosen column

SELECT TOP (1) *
FROM dbo.Table_Name
ORDER BY SomeColumn ASC
UNION ALL
SELECT TOP (1) *
FROM dbo.Table_Name
ORDER BY SomeColumn DESC;


8) Department with lowest average salary

SELECT TOP (1) Department, AVG(Salary) AS Avg_Salary
FROM dbo.Employees
GROUP BY Department
ORDER BY Avg_Salary ASC;


9) Employees in company for more than 5 years

SELECT *
FROM dbo.Employees
WHERE Join_Date <= DATEADD(YEAR, -5, CAST(GETDATE() AS date));


10) Second largest number in a column

SELECT MAX(Col_Name) AS Second_Largest
FROM dbo.Table_Name
WHERE Col_Name < (SELECT MAX(Col_Name) FROM dbo.Table_Name);


11) Employees who do not have any subordinates

SELECT e.*
FROM dbo.Employees e
WHERE NOT EXISTS (
  SELECT 1
  FROM dbo.Employees s
  WHERE s.Manager_ID = e.Employee_ID
);


12) Check if table is empty or not

SELECT CASE
         WHEN EXISTS (SELECT 1 FROM dbo.Table_Name) THEN 'Not Empty'
         ELSE 'Empty'
       END AS Table_State;


13) Second highest salary from each department

WITH ranked AS (
  SELECT Department, EmployeeID, Salary,
         DENSE_RANK() OVER (PARTITION BY Department ORDER BY Salary DESC) AS rk
  FROM dbo.Employees
)
SELECT Department, EmployeeID, Salary
FROM ranked
WHERE rk = 2;


14) List all employees hired in the last 6 months

SELECT *
FROM dbo.Employees
WHERE HireDate >= DATEADD(MONTH, -6, CAST(GETDATE() AS date));


15) Employees whose salary is higher than their manager's salary

SELECT e.Employee_ID, e.Salary
FROM dbo.Employees e
JOIN dbo.Employees m ON e.Manager_ID = m.Employee_ID
WHERE e.Salary > m.Salary;


16) Employees in both departments 101 and 102
Assumes a junction table: dbo.EmployeeDepartments(Employee_ID, Department_ID)

SELECT ed.Employee_ID
FROM dbo.EmployeeDepartments ed
WHERE ed.Department_ID IN (101, 102)
GROUP BY ed.Employee_ID
HAVING COUNT(DISTINCT ed.Department_ID) = 2;


17) Employees with same salary (duplicate salaries)

SELECT *
FROM dbo.Employees
WHERE Salary IN (
  SELECT Salary
  FROM dbo.Employees
  GROUP BY Salary
  HAVING COUNT(*) > 1
);


18) Update salaries of employees based on their department

UPDATE e
SET e.Salary = CASE
                  WHEN e.Department_ID = 101 THEN e.Salary * 1.10
                  WHEN e.Department_ID = 102 THEN e.Salary * 1.20
                  ELSE e.Salary
               END
FROM dbo.Employees e;


19) Employees who joined in the same month and year as their manager

SELECT e.Employee_ID, e.Join_Date,
       m.Employee_ID AS Manager_ID, m.Join_Date AS Manager_Join_Date
FROM dbo.Employees e
JOIN dbo.Employees m ON e.Manager_ID = m.Employee_ID
WHERE MONTH(e.Join_Date) = MONTH(m.Join_Date)
  AND YEAR(e.Join_Date) = YEAR(m.Join_Date);


20) Retrieve employee names and salaries in a single string

SELECT CONCAT(First_Name, ' ', Last_Name, ' earns ', FORMAT(Salary, 'N2')) AS employee_info
FROM dbo.Employees;


21) Employees who belong to departments with less than 3 employees

SELECT e.Employee_Name
FROM dbo.Employees e
WHERE e.Department_ID IN (
  SELECT Department_ID
  FROM dbo.Employees
  GROUP BY Department_ID
  HAVING COUNT(*) < 3
);


22) Employees with the same first name

SELECT e.*
FROM dbo.Employees e
WHERE e.First_Name IN (
  SELECT First_Name
  FROM dbo.Employees
  GROUP BY First_Name
  HAVING COUNT(*) > 1
);


23) Delete employees who have been in the company for more than 15 years

DELETE e
FROM dbo.Employees e
WHERE e.Join_Date < DATEADD(YEAR, -15, CAST(GETDATE() AS date));


24) Top 3 highest-paid employees in each department

WITH d_employees AS (
  SELECT e.*,
         DENSE_RANK() OVER (PARTITION BY e.Department_ID ORDER BY e.Salary DESC) AS rk
  FROM dbo.Employees e
)
SELECT *
FROM d_employees
WHERE rk <= 3;


25) Employees in departments that have not hired anyone in the past 2 years

SELECT e.*
FROM dbo.Employees e
WHERE e.Department_ID IN (
  SELECT Department_ID
  FROM dbo.Employees
  GROUP BY Department_ID
  HAVING MAX(HireDate) < DATEADD(YEAR, -2, CAST(GETDATE() AS date))
);


26) Managers who have more than 5 subordinates

SELECT m.Employee_ID AS Manager_ID,
       CONCAT(m.First_Name, ' ', m.Last_Name) AS Manager_Name,
       COUNT(e.Employee_ID) AS direct_reports
FROM dbo.Employees m
JOIN dbo.Employees e ON e.Manager_ID = m.Employee_ID
GROUP BY m.Employee_ID, m.First_Name, m.Last_Name
HAVING COUNT(e.Employee_ID) > 5
ORDER BY direct_reports DESC;


27) Display employee names and hire dates as "Name - MM/DD/YYYY"

SELECT CONCAT(First_Name, ' ', Last_Name, ' - ', FORMAT(HireDate, 'MM/dd/yyyy')) AS name_hiredate
FROM dbo.Employees
WHERE HireDate IS NOT NULL
ORDER BY HireDate;


28) Display employees grouped by age brackets (20–30, 31–40, 41–50, 51–60, 61+)

WITH AgeCTE AS (
  SELECT e.Employee_ID,
         e.Employee_Name,
         DATEDIFF(year, e.Date_Of_Birth, GETDATE())
           - CASE WHEN DATEADD(year, DATEDIFF(year, e.Date_Of_Birth, GETDATE()), e.Date_Of_Birth) > GETDATE()
                  THEN 1 ELSE 0 END AS AgeYears
  FROM dbo.Employees e
)
SELECT CASE
         WHEN AgeYears BETWEEN 20 AND 30 THEN '20-30'
         WHEN AgeYears BETWEEN 31 AND 40 THEN '31-40'
         WHEN AgeYears BETWEEN 41 AND 50 THEN '41-50'
         WHEN AgeYears BETWEEN 51 AND 60 THEN '51-60'
         WHEN AgeYears >= 61 THEN '61+'
         ELSE 'Under 20'
       END AS AgeBracket,
       COUNT(*) AS EmployeeCount
FROM AgeCTE
WHERE AgeYears IS NOT NULL
GROUP BY CASE
           WHEN AgeYears BETWEEN 20 AND 30 THEN '20-30'
           WHEN AgeYears BETWEEN 31 AND 40 THEN '31-40'
           WHEN AgeYears BETWEEN 41 AND 50 THEN '41-50'
           WHEN AgeYears BETWEEN 51 AND 60 THEN '51-60'
           WHEN AgeYears >= 61 THEN '61+'
           ELSE 'Under 20'
         END
ORDER BY CASE
           WHEN EXISTS (SELECT 1 WHERE 1=0) THEN 0  placeholder, keeps ORDER BY at end
           WHEN 'Under 20' = 'Under 20' THEN 1
         END,
         CASE
           WHEN AgeBracket = '20-30' THEN 2
           WHEN AgeBracket = '31-40' THEN 3
           WHEN AgeBracket = '41-50' THEN 4
           WHEN AgeBracket = '51-60' THEN 5
           WHEN AgeBracket = '61+'   THEN 6
           ELSE 7
         END;


29) Products with strictly increasing prices (never reduced, never equal consecutively)

WITH ordered AS (
  SELECT
      Product_ID,
      Price,
      Price_Date,
      LAG(Price) OVER (PARTITION BY Product_ID ORDER BY Price_Date) AS Prev_Price
  FROM dbo.Product_Prices
)
SELECT Product_ID
FROM ordered
GROUP BY Product_ID
HAVING SUM(CASE WHEN Prev_Price IS NOT NULL AND Price <= Prev_Price THEN 1 ELSE 0 END) = 0;
