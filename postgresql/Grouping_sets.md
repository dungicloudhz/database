# 1. `GROUPING SETS` là gì?
- Trong SQL bình thường, khi dùng `GROUPING BY`, bạn chỉ có thể nhóm theo **một tập hợp cột duy nhất**.
- `GROUPING SETS` cho phép bạn nhóm theo **nhiều tập hợp cột khác nhau trong cùng một câu lệnh**.
- Nó giúp bạn **tránh phải viết nhiều truy vấn UNION ALL**.
# 2. Cú pháp
```sql
select column_list, aggregate_function(...)
from table
group by grouping sets (
    (col1),
    (col2, col3),
    ()
)
```
- `(col1)`: nhóm theo col1
- `(col2, col3)`: nhóm theo col2 + col3
- `()` → nhóm toàn bộ bảng (tức là tổng cộng)
# 3. Ví dụ cơ bản
Giả sử bảng **sales**:
| region | product | amount |
| ------ | ------- | ------ |
| North  | A       | 100    |
| North  | B       | 200    |
| South  | A       | 150    |
| South  | B       | 300    |

Tính tổng doanh số theo:
- region
- product
- tổng toàn bảng
```sql
select region, product, sum(amount) as total_sales
from sales
group by grouping set (
    (region),
    (product),
    ()
);
```
Kết quả:
| region | product | total\_sales |
| ------ | ------- | ------------ |
| North  | NULL    | 300          |
| South  | NULL    | 450          |
| NULL   | A       | 250          |
| NULL   | B       | 500          |
| NULL   | NULL    | 750          |

PostgreSQL tự động làm nhiều cấp độ tổng hợp trong **một truy vấn duy nhất**.
# 4. `CUBE` và `ROLLUP` (các dạng đặc biệt của grouping sets)
## 1. `ROLLUP`
Sinh ra các tổng cộng **theo thứ tự phân cấp**.
```sql
select region, product, SUM(amount) as total_sales
from sales
group by rollup(region, product);
```
Kết quả:
- Tổng theo `(region, product)`
- Tổng theo `(region)`
- Tổng toàn bảng
## 2. `CUBE`
Sinh ra **mọi tổ hợp có thể của các cột**.
```sql
select region, product, SUM(amount) as total_sales
from sales
group by cube(region, product);
```
👉 Kết quả:
- `(region, product)`
- `(region)`
- `(product)`
- Tổng toàn bảng
# 5. Dùng `GROUPING()` để phân biệt dòng tổng hợp
Trong kết quả, khi cột bị `NULL`, ta khí biết là **dữ liệu thật** hay **tổng hợp**.
👉 PostgreSQL có gàm `GROUPING(col)` để kiểm tra.
```sql
select region, product,
    sum(amount) total_sales,
    grouping(region),
    grouping(product)
from sales
group by cube(region, product);
```
- `grouping(col) = 1` → cột này được tổng hợp (NULL là do tổng hợp).
- `grouping(col) = 0` → NULL là dữ liệu thật.
# 6. Tóm tắt
- `GROUPING SETS`: nhiều group by trong 1 câu query.
- `ROLLUP`: tạo phân cấp tổng hợp.
- `CUBE`: tạo mọi kết hợp có thể.
- `GROUPING(col)`: phân biệt NULL dữ liệu thật hay NULL tổng hợp.