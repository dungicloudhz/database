# 1. Khái niệm
- **Function**: phải có `RETURN` (trả về 1 giá trị hoặc bảng). Dùng trong `SELECT`, `WHERE`, `JOIN`...
- **Procedure**: không cần `RETURN`. Được gọi bằng lệnh `CALL`, thích hợp cho logic nghiệp vụ (transaction, logging, batch processing...).
- Procedure có thể:
    - Chứa nhiều lệnh SQL phức tạp.
    - Gọi `COMMIT` hoặc `ROLLBACK` bên trong (function thì KHÔNG được).
    - Dùng để thay thế một phần **stored procedure** trong Oracle/SQL Server.

# 2. Cú pháp tạo
```sql
CREATE [OR REPLACE] PROCEDURE procedure_name(param_name data_type,...)
LANGUAGE plpgsql
AS $$
BEGIN
    -- Logic
END;
$$;
```

# 3. Gọi Procedure
```sql
CALL procedure_name(arg1, arg2, ...);
```
