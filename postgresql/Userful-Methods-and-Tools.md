# 1. Mục đích của EXPLAIN
- Cho bạn biết PostgreSQL **sẽ thực thi** query như thế nào(query plan).
- Giúp hiểu: có dùng **index** không, có quét toàn bảng (**Seq Scan**) không, cos join kiểu gì (Nested Loop/ Hash Join/ Merge Join), chi phí dự đoán thế nào.
```sql
EXPLAIN <query>;
```
Ví dụ:
```sql
EXPLAIN SELECT * FROM employees WHERE salary > 5000;
```
Kết quả có dạng:
```pgsql
Seq Scan on employees  (cost=0.00..35.50 rows=10 width=100)
  Filter: (salary > 5000)
```
🔍 Ý nghĩa:
- **Seq Scan** → PostgreSQL quét toàn bộ bảng (chậm nếu bảng lớn).
- **cost=0.00..35.50** → chi phí ước tính (thấp hơn thì tốt hơn).
- **rows=10** → ước tinsh có 10 dòng kết quả.
- **width=100** → độ rộng trung bình (byte) của mỗi dòng.

# 2. EXPLAIN ANALYZE
- Không chỉ cho **plan dự đoán**, mà còn **thực thi thật** và báo thời gian thực tế.
- Cú pháp:
```sql
EXPLAIN ANALYZE <query>;
```
Ví dụ:
```sql
EXPLAIN ANALYZE SELECT * FROM employees WHERE salary > 5000; 
```
Kết quả như sau:
```sql
Seq Scan on employees  (cost=0.00..35.50 rows=10 width=100) (actual time=0.050..0.300 rows=12 loops=1)
  Filter: (salary > 5000)
  Rows Removed by Filter: 88
Planning Time: 0.120 ms
Execution Time: 0.350 ms
```
🔍 Ý nghĩa thêm:
- **actual time=0,050..0.300** → thời gian thực tế (bắt đầu → kết thúc).
- **row=12** → số dòng thực tế (khác với rows=10 dự đoán).
- **Rows Removed by filter=88** → số bản ghi bị loại.
- **Execute Time=0.350 ms** → thời gian tổng.
# 3. Ví dụ Index
Nếu bạn thêm index:
```sql
CREATE INDEX idx_salary on employees(salary);
```
Rồi chạy lại:
```sql
EXPLAIN ANALYZE SELECT * FROM employees WHERE salary > 5000;
```
Bạn sẽ thấy: 
```sql
Index Scan using idx_salary on employees  
(cost=0.15..8.37 rows=12 width=100) (actual time=0.020..0.040 rows=12 loops=1)
```
# 5. Một số option hữu ích
- **`EXPLAIN (ANALYZE, BUFFERS)`** → xem info về I/O.
- **`EXPLAIN (ANALYZE, VERBOSE)`** → in chi tiết hơn.
- **`EXPLAIN (FORMAT [JSON | TEXT | YAML | XML])`** → xuất ra JSON (dùng cho tool phân tích).
Ví dụ:
```sql
EXPLAIN (ANALYZE, BUFFERS, FORMAT [JSON | TEXT | YAML | XML])
SELECT * FROM employees WHERE salary > 5000
```
# 6. TRUNCATE là gì?
- Dùng để **xóa toàn bộ dữ liệu trong bảng** rất nhanh.
- Khác với `DELETE`:
  - `DELETE` xóa từng dòng → **chậm hơn** với bảng lớn.
  - `TRUNCATE` bỏ qua từng dòng, chỉ **reset lại dữ liệu bảng** → **nhanh hơn nhiều**.
Cú pháp:
```sql
TRUNCATE [TABLE] table_name [,...]
  [ RESTART IDENTITY | CONTINUE IDENTITY ]
  [ CASCADE | RESTRICT ]
```
# 7. Các tùy chọn quan trọng
(1) `RESTART IDENTITY` / `CONTINUE IDENTITY`

- **RESTART IDENTITY** → reset lại các cột `SERIAL`/`IDENTITY` về giá trị bắt đầu.
- **CONTINUE IDENTITY** (mặc định) → giữ nguyên sequence.
Ví dụ:
```sql
-- Tạo bảng có ID tự tăng
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name TEXT
)
INSERT INTO users (name) VALUES ('A'), ('B'), ('C')

TRUNCATE users RESTART IDENTITY;
```
Xóa hết dữ liệu + `id` sẽ quay về 1.

(2) `CASCADE`/`RESTRICT` 

- **CASECADE** → nếu bảng có quan hệ `FOREIGN KEY`, thì **tự động TRUNCATE** các bảng liên quan.
- **RESTRICT** (mặc định) → báo lỗi nếu có quan hệ.
Ví dụ:
```sql
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INT REFERENCES users(id)
);

-- Thử truncate
TRUNCATE users RESTRICT;
-- ERROR: cannot truncate a table referenced in foreign key constraint

TRUNCATE users CASECADE;
-- users và order đều bị truncate
```
# 8. Một số ví dụ
```sql
-- Truncate 1 bảng
TRUNCATE users;

-- Truncate nhiều bảng một lúc
TRUNCATE users, orders;

-- Truncate + reset ID về 1
TRUNCATE users RESTART IDENTITY;

-- Truncate với CASCADE (xóa cả bảng liên quan)
TRUNCATE users CASCADE;
```

# 9. COPY là gì?
- `COPY` cho phép **nhật (import)** và **xuất (export)** dữ liệu giữa **bảng PostgreSQL** và **file/STDIN/STDOUT**.
- Rất nhanh, tối ưu hơn `INSERT` khi xử lý **hàng triệu bản ghi**.
```sql
COPY table_name [(column_list)]
FROM 'file_path' | STDIN
WITH (FORMAT format_option [,...]);

COPY table_name [(column_list)]
TO 'file_path' | STDOUT
WITH (FORMAT format_option [,...]);
```
# 10. Export (Xuất dữ liệu)
```sql
COPY employees TO '/tmp/employees.csv' WITH (FORMAT csv, HEADER true);
```
Giải thích:
- `TO` → export dữ liệu.
- `FORMAT csv` → xuất ra định dạng CSV.
- `HEADER true` → thêm dòng tiêu đề (cột).

# 11. Import (Nhập dữ liệu)
Ví dụ: Import dữ liệu từ CSV vào bảng.
```sql
COPY employees FROM '/tmp/employees.csv' WITH (FORMAT csv, HEADER true);
```
Giải thích:
- `FROM` → import dữ liệu vào bảng.
- `HEADER true` → bỏ qua dòng đầu tiên (tên cột).

# 12. Kiểu dữ liệu mảng (Array Types)
Trong PostgreSQL, bạn có thể khai báo **mảng 1 chiều, 2 chiều, n chiều** của hầu hết mọi kiểu dữ liệu (int, text, uuid, json,...)
Ví dụ:
```sql
-- Mảng số nguyên
CREATE TABLE t1 (nums INTEGER[]);

-- Mảng văn bản
CREATE TABLE t2 (tags TEXT[]);

-- Mảng 2 chiều
CREATE TABLE t3 (matrix INTEGER[][]);
```
Chèn dữ liệu:
```sql
INSERT INTO nums[1] FROM t1; -- Lấy phần tử đầu tiên
INSERT INTO tags[1] FROM t2; -- Lấy phần tử thứ 2
INSERT INTO matrix[2][3] FROM t3; -- hàng 2, cột 3
```
Cập nhật phần tử:
```sql
UPDATE t1 SET nums[2] = 99 WHERE nums[1] = 1;
```

# 13. Các hàm & toán tử quan trọng với Array
(1) Kiểm tra phần tử
```sql
-- ANY: có ít nhất 1 phần tử thỏa mãn
SELECT * FROM t1 WHERE 3 = ANY(nums);

-- ALL: tất cả phần tử thỏa mãn
SELECT * FROM t1 WHERE @> ARRAY[2, 3];
```
(2) Toán tử array
| Toán tử / Hàm | Ý nghĩa                   | Ví dụ                        |
| ------------- | ------------------------- | ---------------------------- |
| `=`           | So sánh mảng              | `'{1,2}' = '{1,2}'`          |
| `@>`          | Mảng A chứa mảng B        | `ARRAY[1,2,3] @> ARRAY[2]` ✔ |
| `<@`          | Mảng A nằm trong mảng B   | `ARRAY[2] <@ ARRAY[1,2,3]` ✔ |
| `&&`          | Hai mảng có phần tử chung | `ARRAY[1,2] && ARRAY[2,3]` ✔ |

# 14. Hàm làm việc với mảng
```sql
-- array_length(mảng, chiều) → độ dài
SELECT array_length(ARRAY[10,20,30], 1); -- 3

-- array_append(mảng, giá trị) → thêm cuối
SELECT array_append(ARRAY[1,2], 3); -- {1,2,3}

-- array_prepend(giá trị, mảng) → thêm đầu
SELECT array_prepend(0, ARRAY[1,2]); -- {0,1,2}

-- array_cat(m1, m2) → nối mảng
SELECT array_cat(ARRAY[1,2], ARRAY[3,4]); -- {1,2,3,4}

-- unnest(mảng) → tách mảng thành nhiều dòng
SELECT unnest(ARRAY['pg', 'sql', 'array']);
-- Kết quả: 'pg' / 'sql' / 'array'

-- array_agg(cột) → gom dữ liệu thành mảng
select dept, array_agg(name)
FROM employees
GROUP BY dept;

-- string_to_array(text, delimiter)
SELECT string_to_array('a,b,c', ','); -- {a,b,c}

-- array_to_string(mảng, delimiter)
SELECT array_to_string(ARRAY['a','b','c'], '-'); -- 'a-b-c'
```

# 15. Kiểu dữ liệu JSON tron PostgreSQL
PostgreSQL hỗ trợ 2 loại:
- `json`: lưu trữ dưới dạng text, giữ nguyên format khi insert.
- `jsonb`: (binary JSON) → nén, sắp xếp key, tối ưu tìm kiếm & index → **nên dùng**.
```sql
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  data JSONB
)

INSERT INTO products (data) VALUES
('{"name": "Laptop", "price": 1200, "tags": ["tech","computer"]}');
```
# 15. Toán tử làm việc với JSON
| Toán tử | Ý nghĩa                                | Ví dụ                           |
| ------- | -------------------------------------- | ------------------------------- |
| `->`    | Truy xuất **giá trị JSON** (dạng JSON) | `data->'name'` → `"Laptop"`     |
| `->>`   | Truy xuất **giá trị text**             | `data->>'name'` → `Laptop`      |
| `#>`    | Truy xuất JSON theo **đường dẫn**      | `data#>'{tags,0}'` → `"tech"`   |
| `#>>`   | Truy xuất text theo đường dẫn          | `data#>>'{tags,0}'` → `tech`    |
| `@>`    | Kiểm tra JSON chứa JSON con            | `data @> '{"price":1200}'`      |
| `?`     | Kiểm tra key tồn tại                   | `data ? 'price'`                |
| `?&`    | Tất cả key đều có                      | `data ?& array['name','price']` |

# 16. Các hàm JSON cơ bản
(1) Trích xuất dữ liệu
```sql
-- json_object_keys: liệt kê các key
SELECT json_object_keys('{"a":1,"b":2,"c":3}');

-- json_array_elements: tách mảng thành dòng
SELECT json_array_elements('["pg","sql","json"]');

-- json_each: key-value pairs
SELECT * FROM json_each('{"name":"Laptop","price":1200}');
```

(2) Chuyển đổi JSOn
```sql
-- row_to_json: row → JSON
SELECT row_to_json(r) 
FROM (SELECT 1 AS id, 'Book' AS name) r;

-- json_build_object: tạo JSON từ key-value
SELECT json_build_object('name','Phone','price',500);

-- json_build_array: tạo JSON array
SELECT json_build_array('pg','sql','json');

-- jsonb_set: update giá trị trong JSONB
SELECT jsonb_set('{"name":"Laptop","price":1000}'::jsonb,
                 '{price}', '1200');
```
(3) Gom dữ liệu thành JSON
```sql
-- json_agg: group → JSON array
SELECT json_agg(name) 
FROM employees;

-- json_object_agg: group → JSON object (key-value)
SELECT json_object_agg(id, name) 
FROM employees;
```
