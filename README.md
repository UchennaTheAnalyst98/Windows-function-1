# Windows-function-1
# First 15 task using windows function in MySQL.



# DATABASE CREATION

CREATE DATABASE IF NOT EXISTS db;
USE db;

# TABLE CREATIONS

CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(50)
);

INSERT INTO departments 
VALUES	(1, 'Engineering'),
		(2, 'Human Resources'),
		(3, 'Finance'),
		(4, 'Sales'),
		(5, 'Marketing');
        

CREATE TABLE jobs (
    job_id INT PRIMARY KEY,
    job_title VARCHAR(50),
    min_salary INT,
    max_salary INT
);





INSERT INTO jobs 
VALUES	(1, 'Software Engineer', 50000, 120000),
		(2, 'HR Manager', 40000, 90000),
		(3, 'Accountant', 45000, 85000),
		(4, 'Sales Executive', 30000, 70000),
		(5, 'Marketing Analyst', 40000, 80000);


CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    department_id INT,
    job_id INT,
    hire_date DATE,
    gender ENUM('M', 'F'),
    FOREIGN KEY (department_id) REFERENCES departments(department_id),
    FOREIGN KEY (job_id) REFERENCES jobs(job_id)
);




INSERT INTO employees VALUES
(101, 'John', 'Sam', 1, 1, '2018-05-20', 'M'),
(102, 'Jane', 'Gideon', 1, 1, '2019-03-10', 'F'),
(103, 'Sam', 'Brown', 2, 2, '2020-11-12', 'M'),
(104, 'Emily', 'Davis', 3, 3, '2017-01-15', 'F'),
(105, 'James', 'Wilson', 4, 4, '2021-07-30', 'M'),
(106, 'Grace', 'Lee', 5, 5, '2022-04-18', 'F'),
(107, 'Peter', 'Clark', 1, 1, '2016-02-25', 'M'),
(108, 'Olivia', 'Moore', 3, 3, '2020-08-09', 'F'),
(109, 'Lucas', 'Scott', 4, 4, '2023-01-05', 'M'),
(110, 'Sophia', 'Adams', 5, 5, '2022-09-01', 'F');


CREATE TABLE salaries (
    salary_id INT PRIMARY KEY,
    employee_id INT,
    salary INT,
    from_date DATE,
    to_date DATE,
    FOREIGN KEY (employee_id) REFERENCES employees(employee_id)
);




INSERT INTO salaries 
VALUES	(1, 101, 70000, '2022-01-01', '2022-12-31'),
		(2, 101, 75000, '2023-01-01', '2023-12-31'),
		(3, 102, 68000, '2023-01-01', '2023-12-31'),
		(4, 103, 60000, '2023-01-01', '2023-12-31'),
		(5, 104, 65000, '2022-01-01', '2022-12-31'),
		(6, 104, 67000, '2023-01-01', '2023-12-31'),
		(7, 105, 40000, '2023-01-01', '2023-12-31'),
		(8, 106, 45000, '2023-01-01', '2023-12-31'),
		(9, 107, 90000, '2023-01-01', '2023-12-31'),
		(10, 108, 62000, '2023-01-01', '2023-12-31'),
		(11, 109, 37000, '2023-01-01', '2023-12-31'),
		(12, 110, 47000, '2023-01-01', '2023-12-31');




CREATE TABLE performance_reviews (
    review_id INT PRIMARY KEY,
    employee_id INT,
    review_score INT, -- from 1 to 5
    review_date DATE,
    FOREIGN KEY (employee_id) REFERENCES employees(employee_id)
);

INSERT INTO performance_reviews 
VALUES	(1, 101, 4, '2023-12-01'),
		(2, 102, 5, '2023-12-01'),
		(3, 103, 3, '2023-12-01'),
		(4, 104, 4, '2023-12-01'),
		(5, 105, 2, '2023-12-01'),
		(6, 106, 3, '2023-12-01'),
		(7, 107, 5, '2023-12-01'),
		(8, 108, 4, '2023-12-01'),
		(9, 109, 2, '2023-12-01'),
		(10, 110, 3, '2023-12-01');

# TASKS 


# Row Number for all employees
SELECT *,
ROW_NUMBER() OVER () `row_number`
FROM employees;


# Row Number Partitioned by Department
SELECT *,
ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY hire_date) row_num
FROM employees;


# Rank employees by salary
SELECT 
	e.employee_id,
    s.salary,
RANK() OVER (ORDER BY salary DESC) `rank`
FROM employees e
INNER JOIN salaries s
	ON e.employee_id = s.employee_id;



# Dense Rank employees by salary
SELECT 
	e.employee_id,
    s.salary,
    e.first_name,
DENSE_RANK() OVER (ORDER BY salary DESC) `rank`
FROM employees e
INNER JOIN salaries s
	ON e.employee_id = s.employee_id;

#  NTILE to divide salaries into 4 quartiles
SELECT *,
NTILE(4) OVER (ORDER BY salary DESC) quartile
FROM employees e
JOIN salaries s
	ON e.employee_id = s.employee_id;
    
# Find cumulative salary by department
SELECT *,
SUM(s.salary) OVER (PARTITION BY e.department_id ORDER BY salary) cummulative_salary
FROM employees e
JOIN salaries s
	ON e.employee_id = s.employee_id;

    

# Average salary across all employees
SELECT employee_id, salary,
       AVG(salary) OVER () AS avg_salary
FROM salaries;

# Average salary within department
SELECT 
	e.department_id,
    s.salary,
AVG(s.salary) OVER (PARTITION BY e.department_id) average
FROM employees e
JOIN salaries s
	ON e.employee_id = s.employee_id;
    
# Salary difference from department average
SELECT 
	e.department_id,
    s.salary,
s.salary - AVG(s.salary) OVER (PARTITION BY e.department_id) average
FROM employees e
JOIN salaries s
	ON e.employee_id = s.employee_id;
    
    

# Running total of salary
SELECT *,
SUM(salary) OVER (ORDER BY salary) running_total
FROM salaries;


# Find previous salary (LAG)
SELECT *,
LAG(salary) OVER(ORDER BY salary) previous_salary
FROM salaries;


# Find next salary (LAG)
SELECT *,
LEAD(salary) OVER (ORDER BY salary) next_salary
FROM salaries;


# Find salary change between years (LAG with from_date)
SELECT *,
salary - LAG(salary) OVER (PARTITION BY from_date ORDER BY salary) salary_change
FROM salaries;



# Max salary per department
SELECT 
	e.department_id,
    s.salary,
MAX(salary) OVER (PARTITION BY e.department_Id ORDER BY s.salary DESC) max_sal_per_dept
FROM employees e
JOIN salaries s
	ON e.employee_id = s.employee_id;


# Count of employees per department
SELECT *,
COUNT(*) OVER (PARTITION BY department_id) count_dept
FROM employees;


