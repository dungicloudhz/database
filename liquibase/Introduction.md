# 1. Các thẻ cơ bản và ý nghĩa ngắn gọn
- `<databaseChangeLog>`: thẻ gốc chứa changeSet.
- `<changeSet id="..." author="...">`: một đơn vị migration(di cư). `id` + `author` phải là duy nhất.
- `<preConditions>`: kiểm tra điều kiện trước khi chạy (ví dụ `tableExits`, `columnExits`).
- `<createTable>`: tạo bảng.
- `<column>`: cột trong bảng.
- `<constraints>`: ràng buộc cột (`primaryKey`, `nullable`, `unique`, `foreignKeyName` trong 1 số thẻ).
- `<addForeignKeyConstraint>`: tạo foreign key giữa 2 bảng (thường tách riêng sau createTable).
- `<createIndex>`: tạo index.
- `<addUniqueConstraint>`: tạo ràng buộc duy nhất.
- `<sql>`: chạy sql tùy ý.
- `<insert>`, `<update>`, `<delete>`: thao tác dữ liệu trong changeSet.
- `<rollback>`: khai báo hành động khi rollback.
- `<include>`: include các file changelog khác (dùng trong master file).
- `onFail` trong `<preConditions>`: `HALT`, `MARK_RAN`, `CONTINUE`, `WARN`.

# 2. Bảng mapping kiểu dữ liệu phổ biến (PostgreSQL)
> Liquibase không ép bạn dùng kiểu cụ thể; dùng kiểu SQL của DB. Dưới đây là gợi ý cho PostgreSQL.

- `bigserial`/ `bigint` + sequence - cho id tự tăng (big).
- `serial`/ `integer` - id tựu tăng (nhỏ hơn).
- `bigint` hoặc `int` - số nguyên lớn.
- `integer` hoặc `int` - số nguyên