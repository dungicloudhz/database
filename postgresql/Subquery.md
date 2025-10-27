# 1. Subquery with ANY and ALL
## 1. `= ANY (subquery)`
- Ý nghĩa: So sánh một giá trị với **bất kỳ phần tử nào** trpmg tập kết quả của subquery.
- Tương đương: **`IN`**
```sql
-- Lấy ra tất cả nhân viên có lương bằng với lương của ít nhất 1 nhân viên phòng 'IT'
SELECT e.emp_id, e.name, e.salary
FROM employees e
WHERE e.salary = ANY (
    SELECT salary
    FROM employees
    WHERE department = 'IT'
);
-- Ở đây **`= ANY(subquery)`** hoạt động giống **`IN (subquery)`**
```
## 2. `> ANY (subquery)`
- Ý nghĩa: So sánh giá trị với **ít nhất một** phần tử trong tập subquery.
- Điều kiện sẽ đúng nếu tồn tại phần tử nhỏ hơn nó.
```sql
-- Tìm nhân viên có lương cao hơn ít nhất 1 nhân viên trong phòng 'HR'
SELECT e.emp_id, e.name, e.salary
FROM employees e
WHERE e.salary > ANY (
    SELECT salary
    FROM employees
    WHERE department = 'HR'
);

-- Nghĩa là chỉ cần lương của nhân viên đó cao hơn một người nào đó trong HR là đủ.
```

## 3. `> ALL (subquery)`
- Ý nghĩa: So sánh giá trị với **tất cả** phần tử trong tập subquery.
- Điều kiện sẽ đúng nếu giá trị lớn hơn **mọi phần tử**.
```sql
-- Tìm nhân viên có lương cao hơn tất cả nhân viên trong phòng 'HR'
SELECT e.emp_id, e.name, e.salary
FROM employees e
WHERE e.salary > ALL (
    SELECT salary
    FROM employees
    WHERE department = 'HR'
);

-- Nghĩa là lương của nhân viên đó phải cao hơn mọi người trong HR (lớn nhất phòng HR).
```
## 4. `< ALL (subquery)` và `< ANY (subquery)`
Tương tụ:
- **`< ALL`**: nhỏ hơn tất cả → nghĩa là nhỏ hơn giá trị nhỏ nhất trong tập subquery.
- **`< ANY`**: nhỏ hơn ít nhất một → nghĩa là nhỏ hơn giá trị lớn nhất trong tập subquery.
```sql
-- Lương nhỏ hơn tất cả nhân viên phòng 'Finance'
SELECT *
FROM employees
WHERE salary < ALL (
    SELECT salary
    FROM employees
    WHERE department = 'Finance'
);

-- Lương nhỏ hơn ít nhất một nhân viên phòng 'Finance'
SELECT *
FROM employees
WHERE salary < ANY (
    SELECT salary
    FROM employees
    WHERE department = 'Finance'
);
```
🔑 Tóm tắt nhanh
```sql
= ANY ~ IN
> ANY → lớn hơn ít nhất một giá trị
> ALL → lớn hơn tất cả giá trị
< ANY → nhỏ hơn ít nhất một giá trị
< ALL → nhỏ hơn tất cả giá trị
```
# 2. Subquery with EXITS
```sql
SELECT ...
FROM table t
WHERE EXISTS (
    SELECT 1
    FROM another_table a
    WHERE a.col = t.col
);
```
- `EXISTS` trả về **TRUE** nếu **subquery có ít nhất một dòng kết quả**.
- `NOT EXISTS` ngược lại, trả về TRUE nếu **subquery không trả về dòng nào**.
- Thồng thường, người ta viết `SELECT 1` trong subquery (vì chỉ cần biết có dòng tồn tại hay không, không quan trọng nội dung).
### So sánh `EXISTS` và `IN`
Cùng mục đích, nhưng khác nhau cách xử lý:
- `IN`: kiểm tra xem giá trị có nằm trong tập hợp hay không.
- `EXISTS`: kiểm tra sự tồn tại của bản ghi.
```sql
-- Dùng IN
SELECT *
FROM employees
WHERE dept_id IN (SELECT dept_id FROM departments);

-- Dùng EXISTS
SELECT *
FROM employees e
WHERE EXISTS (
    SELECT 1
    FROM departments d
    WHERE d.dept_id = e.dept_id
);
```
⚡ Khi nào nên dùng?
- Nếu subquery có khả năng trả về nhiều `NULL` hoặc dữ liệu lớn → `EXISTS` thường an toàn và nhanh hơn.
- `IN` dễ viết, nhưng dễ gặp vấn đề khi có nhiều `NULL`.
# 3. Recursive CTEs (Đệ quy CTE)
```sql
WITH RECURSIVE cte_name AS (
    -- 1. Phần cơ sở (anchor member)
    SELECT ...
    FROM ...
    WHERE điều_kiện_cơ_sở

    UNION ALL

    -- 2. Phần đệ quy (recursive member)
    SELECT ...
    FROM bảng
    JOIN cte_name ON điều_kiện_đệ_quy
)
SELECT * FROM cte_name;
```
- **`Anchor member`**: định nghĩa dữ liệu bắt đầu (gốc).
- **`Recursive member`**: định nghĩa cách từ kết quả trước tạo ra kết quả mới
- PostgreSQL sẽ lặp đi lặp lại cho đến khi không còn dòng mới nào sinh ra.
```sql
WITH RECURSIVE numbers AS (
    -- Anchor: bắt đầu từ 1
    SELECT 1 AS n
    UNION ALL
    -- Recursive: cộng thêm 1 cho đến khi n < 10
    SELECT n + 1
    FROM numbers
    WHERE n < 10
)
SELECT * FROM numbers;
👉 Kết quả: 1, 2, 3, …, 10
```