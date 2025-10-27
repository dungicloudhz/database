# 1. Transaction là gì?
- **Transaction** = nhóm các câu lệnh SQL được thực thi như **một đơn vị công việc** .
- Hoặc **tất cả cùng thành công**, hoặc **tất cả cùng thất bại** (nguyên tắc all-or-nothing).
- PostgreSQL đảm bảo transaction bằng cơ chế **ACID**.
# 2. ACID trong Transaction
- **Atomicity**: tất cả câu lệnh trong transaction phải hoàn tất, nếu lỗi thì rollback hết.
- **Consistency**: DB chuyển từ trạng thái hợp lệ này sang trạng thái hợp lệ khác.
- **Isolation**: các transaction chạy song song không ảnh hưởng sai đến nhau.
- **Durability**: khi đã commit, dữ liệu được lưu vĩnh viễn (ngay cả khi mất điện).
# 3. Transaction cơ bản trong PostgreSQL
**Bắt đầu transaction**
```sql
BEGIN; -- hoặc START TRANSACTION
```
**Thực hiện nhiều câu lệnh**
```sql
INSERT INTO acounts (id, balance) VALUES (1, 1000);
UPDATE acounts SET balance = balance - 100 WHERE id = 1; 
```
**Kết thúc**
- **Lưu thay đổi (commit)**:
```sql
COMMIT;
```
- **Hủy bỏ thay đổi (rollback)**:
```sql
ROLLBACK;
```
👉 Nếu bạn không dùng `BEGIN ... COMMIT`, PostgreSQL mặc định **autocommit**: mỗi câu lệnh SQL là 1 transaction riêng.
# 4. SAVEPOINT (transaction lồng nhau)
Cho phép bạn rollback một phần thay đổi, thay vì toàn bộ.
```sql
BEGIN;

INSERT INTO accounts (id, balance) VALUES (2, 500);

SAVEPOINT sp1;

UPDATE accounts SET balance = balance - 100 WHERE id = 2;

ROLLBACK TO sp1;  -- quay lại sp1, hủy update nhưng giữ insert

COMMIT;
```
# 5. Isolation Levels (mức độ cô lập)
PostgreSQL hỗ trợ 4 mức chuẩn SQL:
- **READ UNCOMMITED** → (PostgeSQL thực tế = READ COMMITTED)
- **READ COMMITED** → thấy dữ liệu đã commit của transaction khác.
- **REPEATABLE READ** → cùng transaction thì đọc nhiều lần luôn thấy kết quả cũ.
- **SERIALIZABLE** → nghiêm ngặt nhất, mô phỏng chạy tuần tự.
Ví dụ set level:
```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```
# 6. Transaction + Concurrency (song song)
- PostgreSQL dùng MVVC (Multi-Version Concurrency Control) để quản lý transaction song song.
- Khi 2 transaction cập nhật cùng 1 row, PostgreSQL dùng **row-level lock** để tránh xung đột.
- Có thể điêu khiển lock:
```sql
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
```