# 1. User-Defined Function là gì?
- Là hàm do **người dùng tự định nghĩa**.
- Có thể nhập tham số, xử lý logic, và trả về kết quả (giá trị đơn, bảng, hoặc void).
- Giúp **tái sử dụng logic, tách biệt nghiệp vụ,** và **tăng tính module**.
# 2. Cú pháp cơ bản
```sql
CREATE [OR REPLACE] FUNCTION function_name(param_name param_type, ...)
RETURNS return_type
LANGUAGE plpgsql
AS $$
BEGIN
  -- Logic
  RETURN ...;
END;
$$;
```
- `OR REPLACE`: ghi dè hàm nếu đã tồn tại.
- `RETURNS`: kiểu dữ liệu trả về (`INTEGER`, `TEXT`, `BOOLEAN`, `TABLE(...)`, `SETOF(...)`, hoặc `VOID`).
- `LANGUAGE`: thường dùng `plpgsql`, nhưng trong PostgreSQL, C, Python (qua exception).

# 3. Ví dụ cơ bản
**(a) Hàm cộng 2 số**
```sql
CREATE OR REPLACE FUNCTION add_numbers(a INT, b INT)
RETURNS INT
LANGUAGE plpgsql
AS $$
BEGIN
  RETURN a + b;
END;
$$;

-- Gọi hàm
SELECT add_numbers(10, 20);
```
**(b) Hàm trả về BOOLEAN**
```sql
CREATE FUNCTION is_even(n INT)
RETURNS BOOLEAN
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN (n % 2 = 0);
END;
$$;

SELECT is_even(4);  -- true
SELECT is_even(5);  -- false
```
**(c) Hàm trả về bảng (TABLE)**
```sql
CREATE FUNCTION get_high_salary(min_salary NUMERIC)
RETURNS TABLE(emp_id INT, emp_name TEXT, emp_salary NUMERIC)
LANGUAGE plspgsql
AS $$
BEGIN
  RETURN QUERY
  SELECT id, name, salary
  FROM employees
  WHERE salary > min_salary;
END;
$$;

-- Gọi hàm
SELECT * FROM get_high_salary(15000);
```
**(d) Hàm trả về nhiều dòng (SETOF)**
```sql
CREATE FUNCTION get_names_by_department(dep TEXT)
RETURNS SETOF TEXT
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT name FROM employees WHERE department = dep;
END;
$$;

SELECT * FROM get_names_by_department('IT');
```

# 4. Hàm với biến & logic
```sql
CREATE FUNCTION factorial(n INT)
RETURN BIGINT
LANGUAGE plpgsql
AS $$
DECLARE
  result BIGINT := 1;
  i INT;
BEGIN
  IF n < 0 THEN
    RAISE EXCEPTION 'Input must be >= 0';
  END IF;

  FOR i IN 1..n LOOP
    result := result * i;
  END LOOP;

  RETURN result;
END;
$$;

SELECT factorial(5); -- 120
```
# 5. Hàm VOID (không trả về gì)
```sql
CREATE FUNCTION log_message(msg TEXT)
RETURNS VOID
LANGUAGE plpgsql
AS $$
BEGIN
    RAISE NOTICE 'Log: %', msg;
END;
$$;

SELECT log_message('Hello world');
```
# 6. Quản lý UDF
- Xem danh sách hàm:
```sql
\df
```
- Xem chi tiết:
```sql
\df+ function_name
```
- Xóa hàm:
```sql
DROP FUNCTION function_name(params...);
```