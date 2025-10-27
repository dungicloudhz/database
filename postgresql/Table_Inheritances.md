# 1. Table Inheritance là gì?
- PostgreSQL cho phép một **table kế thừa (inherit)** từ một table khác.
- Tương tự như kế thừa trong OOP: bảng con nhận toàn bộ cột và ràng buộc từ bảng cha, đồng thời có thể thêm cột riêng.
- Khi bạn query bảng cha, mặc định **sẽ trả về dữ liệu của bảng cha và các bảng con**.

# 2. Ví dụ cơ bản
```sql
-- Tạo bảng cha
CREATE TABLE persons (
  id SERIAL PRIMARY KEY,
  name TEXT,
  birthdate DATE
);

-- Tạo bảng con kế thừa
CREATE TABLE employees (
  salary NUMERIC,
  hire_date DATE
) INHERITS (persons);

-- Insert vào bảng con
INSERT INTO employees (name, birthdate, salary, hire_date)
VALUES ('Alice','1990-01-01', 1000, '2020-01-01');

-- Insert vào bảng cha
INSERT INTO persons (name, birthdate) VALUES ('Bob','1985-05-05');
```
- Truy vấn:
```sql
SELECT * FROM persons;
-- => sẽ trả về cả Bob (persons) và Alice (employees)

SELECT * FROM ONLY persons;
-- => chỉ trả về Bob, không lấy dữ liệu từ bảng con
```

# 3. Thêm nhiều cấp kế thừa
```sql
CREATE TABLE managers (
  department TEXT
) INHERITS (employees);

INSERT INTO managers (name, birthdate, salary, hire_date, department)
VALUES ('Charlie','1980-03-03',2000,'2015-01-01','IT');

SELECT * FROM persons; 
-- => Trả về Bob, Alice, Charlie
```

# 4. Ràng buộc (Constraints) và Inheritance
- **Primary key, Unique, Foreign Key** Không tự động kế thừa. **Lý do**: vì mỗi bảng con trong PostgreSQL vẫn là một bảng độc lập về mặt vật lý.
- Nếu muốn, bạn phải định nghĩa lại trong bảng con.
```sql
-- Bảng cha có constraint
CREATE TABLE parent (
  id SERIAL PRIMARY KEY,
  value INT CHECK (value > 0)
);

-- Bảng con
CREATE TABLE child (
  extra TEXT
) INHERITS (parent);

-- CHECK được kế thừa
INSERT INTO child (value, extra) VALUES (-5,'bad'); 
-- ❌ lỗi, vì CHECK (value > 0) được kế thừa
```

# 5. Partitioning bằng Inheritance
Trước PostgreSQL 10, **Table Inheritance** được dùng như cơ chế **Partition table** thủ công.
Ví dụ chia bảng `sales` theo năm:
```sql
CREATE TABLE sales (
  id SERIAL,
  amount NUMERIC,
  sale_date DATE
);

-- Partition cho năm 2024
CREATE TABLE sales_2024 (
  CHECK (sale_date >= DATE '2024-01-01' AND sale_date < DATE '2025-01-01')
) INHERITS (sales);

-- Partition cho năm 2025
CREATE TABLE sales_2025 (
  CHECK (sale_date >= DATE '2025-01-01' AND sale_date < DATE '2026-01-01')
) INHERITS (sales);

-- Insert dữ liệu vào partition đúng năm
INSERT INTO sales_2024 (amount, sale_date) VALUES (100,'2024-05-01');
INSERT INTO sales_2025 (amount, sale_date) VALUES (200,'2025-06-01');

-- Query cha sẽ gom cả partition
SELECT * FROM sales;
```
> Ngày nay PostgreSQL có **Declarative Partitioning** (PostgeSQL 10+), tiện hơn nhiều. Inheritance ít dùng cho partition nữa, nhưng vẫn hữu ích cho **mô hình hóa dữ liệu hướng đối tượng**.

# 6. Lưu ý khi dùng Table Inheritance
- Không tự động kế thừa PK/FK/Unique → cần định nghĩa lại nếu cần.
- Index cũng không tự động kế thừa.
- Query bảng cha lấy luôn dữ liệu con (nếu không dùng `ONLY`).
- Dùng tốt cho mô hình hóa dữ liệu phức tạp hoặc tình huống "kiểu dữ liệu chung + chuyên biệt".

✅ **Tóm tắt ngắn gọn**
- `INHERITS` cho phép bảng con nhận cột và constraint từ bảng cha.
- `SELECT * FROM parent;` → lấy cả cha + con.
- `SELECT * FROM ONLY parent;` → chỉ lấy cha.
- Dùng để mô hình hóa dữ liệu OOP hoặc partitioning (trước PostgreSQL 10).
