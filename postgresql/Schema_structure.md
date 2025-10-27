# 1. Schema là gì?
- Trong PostgreSQL, **schema** là một "namespace" (không gian tên) để chứa các đối tượng database:
    - bảng (**table**)
    - view
    - sequence
    - function/procedure
    - type
    - index
- Một **database** có thể chứa nhiều **schema**, và mỗi schema có thể chứa nhiều đối tượng.
👉 Nghĩ đơn giản:
- **Database** = cái ổ cứng
- **Schema** = thư mục
- **Table, view, function...** = file trong thư mục

# 2. Schema mặc định
- Khi bạn tạo 1 database mới, PostgreSQL sẽ tạo sẵn schema:
    - `Public` → schema mặc định, nếu bạn không chỉ định schema thì đối tượng sẽ được tạo ở đây.
    - `pg_catalog` → chứa hệ thống (system catalog).
    - `information_schema` → chứa metadata chuẩn SQL (danh sách tables, views, colums,...).

Ví dụ:
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name TEXT
);
```
→ Thực ra là:
```sql
CREATE TABLE public.users (
    id SERIAL PRIMARY KEY,
    name TEXT
);
```
# 3. Quản lý Schema
## 1. Tạo schema
```sql
CREATE SCHEMA sales;
```
## 2. Tạo Object trong schema
```sql
CREATE TABLE sales.orders (
    order_id SERIAL PRIMARY KEY,
    amount NUMERIC
)
```
## 3. Chỉ định schema khi truy vấn
```sql
SELECT * FROM sales.orders;
```
# 4. Search Path
PostgreSQL có khái niệm **search_path** → danh sách schema mà PostgreSQL sẽ tìm đối tượng khi bạn không ghi rõ schema.
Kiểm tra:
```sql
SHOW search_path;
```
👉 Mặc định: "public"
Thay đổi:
```sql
SET search_path TO sales, public;
```
Khi đó: PostgreSQL tìm trong `sales` trước, rồi mới tìm lới `public`.
# 5. Quản lý quyền trong schema
- Cấp quyền cho user:
```sql
GRANT USAGE ON SCHEMA sale TO user1;
GRANT CREATE ON SCHEMA sale TO user1;
```
- Xóa quyền:
```sql
REVOKE ALL ON SCHEMA sales FROM user1;
```
# 6. Xem cấu trúc schema
**Liệt kê schemas**
```sql
\dn -- trong psql
```
**Liệt kê tables trong 1 schema**
```sql
\dt sales.*
```
**Metadata trong infromation_schema**
```sql
SELECT table_schema, table_name
FROM information_schema.tables
WHERE table_schema NOT IN ('information_schema', 'pg_catalog');
```