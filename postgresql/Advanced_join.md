# 1. `JOIN ... ON ...` cơ bản
Trước khi đi vào `USING` và `NATURAL JOIN`, nhắc lại:
```sql
select *
from employees e
join departments d on e.dept_id = d.dept_id;
-- Ghép 2 bảng theo điều kiện cột
```
# 2. `JOIN ... USING (...)`
- Dùng khi **tên cột join giống nhau** ở cả hai bảng.
- PostgreSQL sẽ tự hiểu `ON table1.column = table2.column`.
- Kết quả chỉ giữ **1 cột duy nhất** thay vì 2 bản sao.
```sql
select * 
from employees
join departments using (dept_id);
--- Tương đương với:
select * 
from employees e
join department d on e.dept_id = d.dept_id
```
📌 Điểm khác biệt:
- `ON`: kết quả có cả `e.dept_id` và `d.dept_id`.
- `USING`: kết quả chỉ có **một cột** `dept_id`.
# 3. `NATURAL JOIN`
- Là join **tự động** dựa trên tất cả cột **cùng tên** ở 2 bảng.
- Bạn không cần viết điều kiện join.
- PostgreSQL sẽ join theo tất cả cột trùng tên, và giữ **1 cột duy nhất**.
```sql
select *
from employees
natural join departments
-- PostgreSQL sẽ tự hiểu join bằng dept_id, vì đó là cột chung.
```
# 4. So sánh `USING` và `NATURAL JOIN`
|                    | `JOIN ... USING (...)`     | `NATURAL JOIN`                                                |
| ------------------ | -------------------------- | ------------------------------------------------------------- |
| **Điều kiện join** | Bạn chỉ định rõ cột nào    | PostgreSQL tự chọn tất cả cột trùng tên                       |
| **An toàn**        | Rõ ràng, dễ kiểm soát      | Có thể gây lỗi nếu 2 bảng có nhiều cột trùng tên ngoài ý muốn |
| **Kết quả**        | Giữ 1 bản sao của cột join | Giữ 1 bản sao cho mỗi cột trùng tên                           |
# 5. Khi nào dùng?
- **Dùng** `USING` khi bị chắc chắn chỉ join theo 1 (hoặc vài) cột trùng tên → **an toàn hơn**.
- **Tránh** `NATURAL JOIN` trong hệ thống lớn/phức tạp vì dễ sinh bug khi schema thay đổi (ví dụ thêm một cột trùng tên bất ngờ).
- **Dùng** `NATURAL JOIN` chỉ khi bạn làm truy vấn nhanh, bảng nhỏ hoặc học SQL ngắn gọn.