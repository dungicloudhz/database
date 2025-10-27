# 1. Các phép toán tập hợp trong PostgreSQL
PostgreSQL hỗ trợ 4 phép chính:
1. `UNION`
2. `UNION ALL`
3. `INTERSECT`
4. `EXPECT`
# 2. Quy tắc chung khi dùng set operations
- Các truy vấn kết hợp ** phải có số cột giống nhau**.
- Kiểu dữ liệu ở các cột tương ứng phải **tương thích** (ví dụ: INT với INT, TEXT với VARCHAR).
- Có thể thêm `ORDER BY` **ở cuối toàn bộ truy vấn** (không đặt bên trong).
# 3. `UNION`
Kết hợp kết quả từ hai truy vấn và **loại bọ trùng lặp** (giống toán học u).
```sql
select dept_id from employees
union
select dept_id from departments;
-- Kết quả: danh sách dept_id có trong employees hoặc departments, không lặp.
```
# 4. `UNION ALL`
Giữ nguyên tất cả kết quả, **bao gồm cả trùng lặp**.
```sql
select dept_id from employees
union all
select dept_id from departments;
-- Kết quả: có thể có nhiều bản sao cùng dept_id.
-- Nhanh hơn UNION, vì không phải loại bỏ trùng lặp.
```
# 5. `INTERSECT`
Lấy **phần giao** giữa 2 tập kết quả (giống toán học ∩)
```sql
select dept_id from employees
intersect
select dept_id from departments;
-- Kết quả: chỉ lấy dept_id xuất hiện ở cả employees và departments
```
# 6. `EXCEPT`
Lấy **phần khác biệt** (giống toán học A - B).
```sql
select dept_id from employees
except
select dept_id from departments;
-- Kết quả: các dept_id có trong employees nhưng không có trong departments.
-- Thứ tự quan trọng: A EXCEPT B != B EXCEPT A 
```