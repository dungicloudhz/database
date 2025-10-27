# 1. Window function là gì?
- Giống như hàm tổng hợp (aggregate functions: `SUM`, `AVG`, `COUNT`...) nhưng **không gộp dòng**.
- Thay vào đó, nó tính toán dựa trên **một "của sổ" (window)** là tập các dòng liên quan đến dòng hiện tại.
- Giữ nguyên tất cả các dòng trong kết quả.
Syntax:
```sql
function_name(...) OVER (
    [PARTITION BY ...]
    [ORDER BY ...]
    [ROWS BETWEEN ..]
)
```
# 2. Các thành phần trong `OVER()`
- **PARTITION BY**: chia dữ liệu thành nhóm (giống `GROUP BY`, nhưng không làm mất dòng).
- **ORDER BY**: sắp xếp trong mỗi nhóm để áp dụng hàm thứ tự (`ROW_NUMBER`, `RANK`...).
- **ROW BETWEEN**: chỉ định phạm vi dòng trong của sổ (frame).
# 3. Ví dụ thực tế
```sql
CREATE TABLE sales (
    id SERIAL PRIMARY KEY,
    emp_name VARCHAR(50),
    department VARCHAR(50),
    amount NUMERIC
);

INSERT INTO sales (emp_name, department, amount) VALUES
('Alice', 'IT', 1000),
('Bob', 'IT', 2000),
('Charlie', 'IT', 1500),
('David', 'HR', 800),
('Emma', 'HR', 1200),
('Frank', 'Finance', 3000),
('Grace', 'Finance', 2500);
```
## 1. `ROW_NUMBER()`
Đánh số thứ tự từng dòng trong mỗi `department` (theo `amount` giảm dần).
```sql
select emp_name, department, amount,
    ROW_NUMBER() ORVER(PARTITION BY department ORDER BY amount DESC) AS row_num
from sales;
-- Kết quả: nhân viên có doanh số cao nhất trong mỗi phòng ban sẽ có row_num = 1.
```
## 2. `RANK()` và `DENSE_RANK()`
```sql
select emp_name, department, amount,
    RANK() OVER (PARTITION BY department ORDER BY amount DESC) as rank,
    DENSE_RANK() OVER (PARTITION BY department ORDER BY amount DESC) as dense_rank
from sales;
```
- `RANK()`: nếu có tie (2 ngoài bằng điểm), nhảy bậc (1, 2, 2, 4).
- `DENSE_RANK()`: không nhảy bậc (1, 2, 2, 3).
## 3. Hàm tổng hợp `OVER`
Tính tổng doanh số từng phòng ban, nhưng vẫn giữ chi tiết từng nhân viên.
```sql
select emp_name, department, amount,
    SUM(amount) OVER (PARTITION BY department) as dept_total
from sales;
-- Không cần group by, mà vẫn thấy chi tiết từng nhân viên.
```
## 4. Hàm `AVG()`, `MAX()`, `MIN()` trong của sổ
```sql
select emp_name, department, amount,
    AVG(amount) OVER (PARTITION BY department) as dept_avg,
    MAX(amount) OVER (PARTITION BY department) as dept_sum
from sales;
```
## 5. `LAG()` và `LEAD()`
Lấy giá trị dòng trước (`LAG`) hoặc dòng sau (`LEAD`) trong cùng partition.
```sql
select emp_name. department, amount,
    LAG(amount) OVER (PARTITION BY department ORDER BY amount) AS prev_amount,
    LEAD(amount) OVER (PARTITION BY department ORDER BY amount) AS prev_amount
from sales;
-- Dùng để tín chênh lệch giữ các dòng liên tiếp.
```
## 6. `ROWS BETWEEN`
Ví dụ: tính tổng doanh số của 2 dòng trước + dòng hiện tại trong mỗi phòng ban.
```sql
select emp_name, department, amount,
    SUM(amount) OVER (
        PARTITION BY department
        ORDER BY amount
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) as rolling_sum
from sales;
```
# 4. Ứng dụng thực tế
- Phân tích top-N (Ví dụ: top 3 nhân viên theo doanh số trong từng phòng ban).
- Tính moving average, rolling sum.
- So sánh dữ liệu dòng trước/dòng sau.
- Ranking, phân nhóm dữ liệu.
# 5. Tóm tắt
- **`Window functions`** = aggregate functions + giữ nguyên dòng.
- Các hàm thường dùng:
    - **Ranking**: `ROW_NUMBER`, `RANK`, `DENSE_RANK`
    - **Aggregate trong window**: `SUM`, `AVG`, `MAX`, `MIN`
    - **Navigation**: `LAG`, `LEAD`, `FIRST_VALUE`, `LAST_VALUE`