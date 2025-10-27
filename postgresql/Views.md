# 1. View là gì?
- **View** = bảng ảo được tạo ra từ một câu lệnh `SELECT`.
- Không lưu dữ liệu thực tế (trừ khi là **materialized view**).
- Giúp:
  - Ẩn logic phức tạp trong query.
  - Bảo mật (người dùng chỉ truy cập view, không truy cập bảng gốc).
  - Tái sử dụng code SQL.

# 2. Tạo view
Cú pháp:
```sql
CREATE VIEW view_name AS
SELECT ...
FROM ...
WHERE ...;
--- Ví dụ:
-- Tạo bảng mẫu
CREATE TABLE employees (
  id SERIAL PRIMARY KEY,
  name TEXT,
  department TEXT,
  salary NUMERIC
);

INSERT INTO employees (name, department, salary) VALUES
('Alice','IT',1000),
('Bob','HR',1200),
('Charlie','IT',1500),
('David','Sales',2000);

-- Tạo view cho nhân viên ID
CREATE VIEW it_employees AS
SELECT id, name, salary
FROM employees
WHERE department = 'IT';

-- Truy vấn View
SELECT * FROM id_employees
```
# 3. Cập nhật qua View
- Có thể `INSERT/UPDATE/DELETE` qua vierw **nếu view đơn giản** (một bảng, không group by, không aggregate).
```sql
INSERT INTO it_employees (name, salary) VALUES ('Eve', 1800)
```
👉 PostgeSQL tự hiểu là insert vào bảng `employees` với `department = 'IT'`.
Nếu view phức tạp → cần dùng **INSTEAD OF triggers**.
- `INSTEAD OF` **Triggers**
- **View thường không update được** nếu view phức tạp (join, group by...).
- PostgreSQL hỗ trợ `INSTEAD OF trigger`:
  - Khi người dùng `INSERT/UPDATE/DELETE` vào view → trigger chạy **thay vì** thao tác trực tiếp trên view.
  - Dùng để map thao tác về bảng gốc.
  ```sql
  -- View join
  CREATE VIEW emp_view AS
  SELECT e.id, e.name, d.dept_name
  FROM employees e
  JOIN departments d ON e.department = d.id;

    -- Hàm trigger
  CREATE OR REPLACE FUNCTION emp_view_insert()
  RETURNS TRIGGER AS $$
  BEGIN
    -- Khi insert vào view → insert vào employees
    INSERT INTO employees (name, department)
    VALUES (NEW.name, (SELECT id FROM departments WHERE dept_name = NEW.dept_name));
    RETURN NULL; -- vì view không lưu trực tiếp
  END;
  $$ LANGUAGE plpgsql;

  -- Gắn trigger
  CREATE TRIGGER trg_emp_view_insert
  INSTEAD OF INSERT ON emp_view
  FOR EACH ROW EXECUTE FUNCTION emp_view_insert();

  -- Sử dụng
  INSERT INTO emp_view (name, dept_name) VALUES ('Alice', 'IT');
  SELECT * FROM emp_view;
  INSTEAD OF triggers
  ```
  → PostgreSQL tự động **redirect** insert về bảng `employees`.
# 4. Xóa hoặc thay đổi View
```sql
DROP VIEW it_employees;

-- hoặc thay thế bằng câu lệnh khác
CREATE OR REPLACE VIEW it_employees AS
SELECT name, salary
FROM employees
WHERE department = 'IT' AND salary > 12000;
```
# 5. Materialized View
- **Materialized View** (bảng thật, lưu dữ liệu, cần refresh) = view **có lưu dữ liệu** → tăng tốc truy vấn.
- Cần `REFRESH` khi dữ liệu gốc thay đổi.
- Có thể đánh index vào **Materialized View**.
- PostgreSQL hỗ trợ: `REFRESH MATERIALIZED VIEW CONCURRENTLY` → cho phép query trong khi refresh (cần unique index).
```sql
CREATE MATERIALIZED VIEW high_salary AS
SELECT * FROM employees WHERE salary > 1500;

-- Cập nhật lại dữ liệu
REFRESH MATERIALIZED VIEW high_salary;

SELECT * FROM high_salary;
```
- `REFRESH MATERIALIZED VIEW CONCURRENTLY`:
  - Bình thường khi `REFRESH MATERIALIZED VIEW`, PostgreSQL sễ **lock view** → không cho query trong lúc refresh.
  - Với `CONCURRENTLY` → cho phép **chưa refresh vừa query view** (dữ liệu cũ vẫn dùng được cho đến khi xong).
  - Điều kiện: Materialized View phải có **unique index** để PostgreSQL xác định dòng thay đổi.
# 6. Recursive View
- Tạo view bằng cách sử dụng CTE Recursive
```sql
CREATE RECURSIVE VIEW employee_hierarchy AS
WITH RECURSIVE emp_cte AS (
    -- Anchor: nhân viên không có sếp
    SELECT id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive: tìm nhân viên cấp dưới
    SELECT e.id, e.name, e.manager_id, c.level + 1
    FROM employees e
    JOIN emp_cte c ON e.manager_id = c.id
)
SELECT * FROM emp_cte;
```