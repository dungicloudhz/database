# 1. Khái niệm
**Trigger** là một đoạn code (function) được PostgreSQL **tự động gọi** khi có một sự kiện xảy ra trên bảng hoặc view:
- `INSERT`
- `UPDATE`
- `DELETE`
- `TRUNCATE`
- Với View → dùng `INSTEAD OF` để trigger
Trigger thường dùng cho:
- Ghi log thay đổi dữ liệu
- Kiểm tra, ràng buộc nghiệp vụ
- Đồng bộ dữ liệu giữa các bảng
- Thực hiện tự động (audit, tính toán, ...)

# 2. Cấu trúc trigger trong PostgreSQL
Muốn tạo **trigger**, bạn cần **function** trước(loại đặc biệt gọi là trigger function).
**Bước 1: Tạo Trigger Function**
```sql
CREATE OR REPLACE FUNCTION trigger_function_name()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    -- Logic xử lý
    RETURN NEW; -- hoặc RETURN OLD
END;
$$;
```
- `NEW`: bản ghi mới (dùng trong `INSERT`, `UPDATE`)
- `OLD`: bản ghi cũ (dùng trong `UPDATE`, `DELETE`)
**Bước 2: Gắn Trigger vào bảng**
```sql
CREATE TRIGGER trigger_name
AFTER INSERT OR UPDATE OR DELETE ON table_name
FOR EACH ROW
EXECUTE FUNCTION trigger_function_name();
```

# 3. Các loại Trigger
- **Thời điểm (Timing)**:
    - `BEFORE`: chạy trước khi SQL thực thi
    - `AFTER`: chạy sau khi SQL thực thi
    - `INSTEAD OF`: dùng cho view thay vì bảng
- **Phạm vi (Granularity)**:
    - `FOR EACH ROW`: chạy cho từng dòng bị tác động
    - `FOR EACH STATEMENT`: chạy một lần cho cả câu lệnh

# 4. Ví dụ minh họa
**Ví dụ 1: Ghi log khi INSERT**
```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name TEXT,
    salary NUMERIC
);

CREATE TABLE employees_log (
    log_id SERIAL PRIMARY KEY,
    emp_id INT,
    action TEXT,
    log_time TIMESTAMP DEFAULT now()
);

-- Trigger Function
CREATE OR REPLACE FUNCTION log_employee_insert()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    INSERT INTO employees_log(emp_id, action)
    VALUES (NEW.id, 'INSERT');
    RETURN NEW;
END;
$$;

-- Trigger
CREATE TRIGGER trg_employee_insert
AFTER INSERT ON employees
FOR EACH ROW
EXECUTE FUNCTION log_employee_insert();
```
👉 Khi `INSERT` nhân viên mới → PostgeSQL tự động ghi vào `employees_log`.
**VÍ dụ 2: Kiểm tra ràng buộc trước khi UPDATE**
```sql
CREATE OR REPLACE FUNCTION check_salary()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    IF NEW.salary < 0 THEN
        RAISE EXCEPTION 'Salary cannot be negative';
    END IF;
    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_check_salary
BEFORE UPDATE ON employees
FOR EACH ROW
EXECUTE FUNCTION check_salary();
```
**Ví dụ 3: INSTEAD OF trigger cho View**
```sql
CREATE VIEW employee_names AS
SELECT id, name FROM employees;

-- Trigger Function
CREATE OR REPLACE FUNCTION insert_employee_name()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    INSERT INTO employees(name, salary)
    VALUES (NEW.name, 1000);  -- mặc định salary
    RETURN NEW;
END;
$$;

-- Trigger
CREATE TRIGGER trg_insert_employee_name
INSTEAD OF INSERT ON employee_names
FOR EACH ROW
EXECUTE FUNCTION insert_employee_name();
```
👉 Khi bạn `INSERT` vào view `employee_names`, thực chất dữ liệu ssex vào bảng `employees`.
# 5. Các biến đặc biệt trong Trigger
- `NEW` → bản ghi mới (INSERT/UPDATE)
- `OLD` → bản ghi cũ (UPDATE/DELETE)
- `TG_OP` → tên thao tác (`INSERT`, `UPDATE`, `DELETE`, `TRUNCATE`)
- `TG_TABLE_NAME` → tên bảng gốc

# 6. Quản lý Trigger
- Liệt kê trigger của bảng:
```sql
SELECT tgname, tgtype::interger, tgrelid::regclass
FROM pg_trigger
WHERE NOT tgisinternal;
```
- Xóa trigger:
```sql
DROP TRIGGER trg_employee_insert ON employees;
```