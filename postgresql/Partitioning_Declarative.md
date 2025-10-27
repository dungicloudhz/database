# 1. Declarative Partitioning là gì?
- Là cơ chế **chia một bảng lớn (parent)** thành nhiều bảng con nhỏ hơn **(partitions)**, dựa theo giá trị của một cột hoặc biểu thức.
- Giúp tăng hiệu năng:
  - Truy vấn nhanh hơn (scan ít dữ liệu hơn).
  - Quản lý dễ hơn (drop/attach từng partition).
- Giống như **partition table trong Oracle / SQL Server**.

# 2. Các loại Partitioning
PostgreSQL hỗ trợ 4 loại:
1. **RANGE**
  - Chi theo khoảng giá trị.
  - Ví dụ: dữ  liệu ngày, tháng, năm.
2. **LIST**
  - Chia theo danh sách giá trị cụ thể.
  - Ví dụ: chia theo quốc gia, trạng thái.
3. **HASH**
  - Chia đều dữ liệu theo hàm băm (hash function).
  - Dùng khi dữ liệu không có thứ tự tự nhiên.
4. **DEFAULT**
  - Parition mặc định cho dữ liệu không khớp với các dữ liệu khác.

# 3. Ví dụ:
**(a) RANGE Partitioning**
```sql
CREATE TABLE sales (
  id SERIAL,
  amount NUMERIC,
  sale_date DATE
) PARTITION BY RANGE (sale_date);

CREATE TABLE sales_2024 PARTITION OF sales
  FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');

CREATE TABLE sales_2025 PARTITION OF sales
  FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');

INSERT INTO sales (amount, sale_date) VALUES (100,'2024-05-01');
INSERT INTO sales (amount, sale_date) VALUES (200,'2025-07-01');

SELECT * FROM sales;
-- PostgreSQL tự động chọn partition đúng

```
**(b) LIST Partitioning**
```sql
CREATE TABLE customers (
  id SERIAL,
  name TEXT,
  country TEXT
) PARTITION BY LIST (country);

CREATE TABLE customers_us PARTITION OF customers FOR VALUES IN ('US');
CREATE TABLE customers_vn PARTITION OF customers FOR VALUES IN ('VN');
CREATE TABLE customers_other PARTITION OF customers DEFAULT;

INSERT INTO customers (name,country) VALUES ('Alice','US');
INSERT INTO customers (name,country) VALUES ('Bob','VN');
INSERT INTO customers (name,country) VALUES ('Charlie','JP');
```
**(c) HASH Partitioning**
```sql
CREATE TABLE logs (
  id SERIAL,
  message TEXT
) PARTITION BY HASH (id);

CREATE TABLE logs_p0 PARTITION OF logs FOR VALUES WITH (MODULUS 4, REMAINDER 0);
CREATE TABLE logs_p1 PARTITION OF logs FOR VALUES WITH (MODULUS 4, REMAINDER 1);
CREATE TABLE logs_p2 PARTITION OF logs FOR VALUES WITH (MODULUS 4, REMAINDER 2);
CREATE TABLE logs_p3 PARTITION OF logs FOR VALUES WITH (MODULUS 4, REMAINDER 3);
```
# 4. Ưu điểm so với Table Inheritance
- Tự động route dữ liệu vào partition (không cần trigger).
- Constraints & Indexes có thể khai báo parent (tự áp dụng cho partition).
- `PRIMARY KEY`, `UNIQUE`, `FOREIGN KEY` có thể hỗ trợ (một phần).
- Tối ưu truy vấn bằng **partition pruning** (PostgreSQL chỉ quét partition liên quan).

# 5. Một số hạn chế
- Không update giá trị của một partition key (phải delete + insert lại).
- `UNIQUE` chỉ hoạt động trong 1 partition, không áp dụng toàn bộ trù khi key bao hồm cả partition key.
- Không dùng partition key từ biểu thức phức tạp (chỉ cột hoặc biểu thức đơn giản).

✅ **Tóm tắt**
- **Declarative Partitioning** = cách chính thức của PostgreSQL để chia bảng từ phiên bản 10+.
- Hỗ trợ **RANGE**, **LIST**, **HASH**, **DEFAULT**.
- Ưu điểm: đơn giản, hiệu quả, giống Oracle/SQL Server.
- Hạn chế: một số ràng buộc không áp dụng xuyên partition.
