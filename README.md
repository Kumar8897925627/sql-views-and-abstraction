# sql-views-and-abstraction
"Implementing abstraction layers in PostgreSQL. Creating reusable Views for simplified reporting, security-driven data masking, and analyzing the limitations of Updatable Views."
# 🖼️ SQL Views: Reusable Query Layers

In this final task, I explored **Views**, which act as a saved lens through which we can look at our data. They provide a layer of abstraction between the complex database structure and the end user.



## 🛡️ Why Use Views?

1. **Simplicity:** Instead of writing a 10-line JOIN query every time, I can simply run `SELECT * FROM my_view`.
2. **Security:** I created a `public_staff_list` view that excludes sensitive salary data, allowing users to see names without seeing private financial info.
3. **Consistency:** If the underlying table structure changes, I only need to update the View definition once, and all connected dashboards will stay functional.

## ⚠️ Important Limitations
* **Performance:** Views don't store data; they run the query every time you call them. For massive datasets, "Materialized Views" are often used instead.
* **Updates:** While you can sometimes update data through a view, it becomes restricted when the view uses `JOIN`, `GROUP BY`, or `DISTINCT`.



## 🛠️ How to Test
1. Run `views_management.sql` in **pgAdmin**.
2. Notice how the `employee_directory` view makes the complicated join logic "disappear" for the end user.

   -- ==========================================================
-- PROJECT: SQL Views and Data Abstraction (Task 10)
-- CONCEPTS: CREATE VIEW, Security Masking, Updatable Views
-- ==========================================================

-- 1. CREATE A COMPLEX JOIN VIEW
-- We combine employee info with department names into one "virtual table"
CREATE OR REPLACE VIEW employee_directory AS
SELECT 
    e.emp_id,
    e.first_name || ' ' || e.last_name AS full_name,
    d.dept_name,
    d.location,
    e.hire_date
FROM employees e
JOIN departments d ON e.dept_id = d.dept_id;

-- 2. QUERY THE VIEW
-- Now we can treat the view like a real table!
SELECT * FROM employee_directory 
WHERE dept_name = 'Marketing'
ORDER BY full_name;

-- 3. SECURITY VIEW (Data Masking)
-- Create a view for HR that hides specific salary figures
CREATE OR REPLACE VIEW public_staff_list AS
SELECT 
    first_name, 
    last_name, 
    department
FROM employees; -- Notice 'salary' is excluded for security

-- 4. ATTEMPTING TO UPDATE THROUGH A VIEW
-- Some views are "Updatable," but complex joins usually are not.
-- This might fail or be restricted depending on the DB engine logic.
UPDATE employee_directory 
SET full_name = 'New Name' 
WHERE emp_id = 1; 

-- 5. REFRESHING THE VIEW
-- If you add a column to the table, you must recreate the view.
DROP VIEW IF EXISTS employee_directory;

CREATE VIEW employee_directory AS
SELECT 
    e.emp_id,
    e.first_name,
    e.last_name,
    d.dept_name,
    e.salary -- Now including salary in the updated version
FROM employees e
JOIN departments d ON e.dept_id = d.dept_id;

-- 6. FINAL CLEANUP (Optional)
-- DROP VIEW employee_directory;
