Các thẻ `<set>`, `<list>`, `<bag>` trong Hibernate mapping (XML) chính là **cách Hibernate ánh xạ mối quan hệ 1-nhiều hoặc nhiều-nhiều** (collection relationship) giữa các entity.
# Giới thiệu
## 1. Tổng quan - vì sao có nhiều loại collection
Trong Java, bạn có các kiểu collection khác nhau:
- `Set` → không trùng phần từ
- `List` → có thứ tự, có thể trùng
- `Bag` → có thể trùng, không đảm bảo thử tự
- `Map` → ánh xạ key-value

Hibernate cũng **hỗ trợ tương ứng** các loại đó trong mapping XML để ánh xạ **đặc điểm dữ liệu trong DB**.
Mỗi loại collection ảnh hưởng trục tiếp đến:
- Cấu trúc bảng trung gian (join table)
- Cách Hibernate sinh SQL (insert/update/delete)
- Cách Hibernate xử lý cache và dirty-checking (so sánh thay đổi)

## 2. Ví dụ chung về mapping 1-Nhiều
Giả sử bạn có 2 entity:
- `Department` (phòng ban)
- `Employee` (nhân viên)
Quan hệ:
> Một phòng ban có nhiều nhân viên
> Một nhân viên chỉ thuộc 1 phòng ban

## 3. `<set>` - Dữ liệu duy nhất, không trùng, thứ tự không quan trọng
```xml
<class name="Department" table="department">
    <id name="id" column="id"/>
    <property name="name" column="name"/>

    <set name="employees" table="employee" inverse="true" lazy="true" cascade="all">
        <key column="department_id"/>
        <one-to-many class="Employee"/>
    </set>
</class>
```
Đặc điểm:
- Ánh xạ sang `java.util.Set`
- Không chứa phần tử trùng (dựa trên `equals()` của entity)
- Hibernate quản lý bằng cách so sánh ID để tránh duplicate
- Không cần thêm cột thứ tự trong DB

Khi nên dùng:
- Khi bạn chỉ cần danh sách các phần tử **duy nhất, thứ tự không quan trọng** 
- Phù hợp cho quan hệ `1-Nhiều` hoặc `N-Nhiều` mà bạn không cần `order`

Ưu điểm:
- Hibernate xử lý rất hiệu quả (dễ dirty-check)
- Không cần thêm cột `index` hay `order_column`

Nhược điểm:
- Không giữ được thứ tự ban đầu (vì là `Set`)
- Nếu bạn cố thêm trùng, Hibernate sẽ loại bỏ

## 4. `<list>` - Dữ liệu có thứ tự, cho phép trùng
```xml
<class name="Department" table="department">
    <id name="id" column="id"/>
    <property name="name" column="name"/>

    <list name="employees" table="employee" cascade="all" lazy="true" inverse="true">
        <key column="department_id"/>
        <list-index column="position"/>
        <one-to-many class="Employee"/>
    </list>
</class>
```
Đặc điểm:
- Ánh xạ sang `java.util.List`
- Có thể trùng phần tử
- Hibernate thêm **cột index** (ở đây là `position`) để lưu thứ tự

Khi nên dùng:
- Khi thứ tự các phần tử **quan trọng** (ví dụ: danh sách câu hỏi, bước xử lý. v.v.)
- Khi bạn cần **giữ nguyên thứ tự chèn vào**

Ưu điểm:
- Giữ được thứ tự chính xác
- Dễ hiểu khi cần trình bày dữ liệu tuần tự

Nhược điểm:
- Thêm cột `list-index` vào bảng con
- Hibernate phải kiểm tra lại thư tự mỗi lần update → chậm hơn `set`

## 5. `<bag>` - Dữ liệu có thể trùng, không thứ tự
```xml
<class name="Department" table="department">
    <id name="id" column="id"/>
    <property name="name" column="name"/>

    <bag name="employees" table="employee" cascade="all" lazy="true" inverse="true">
        <key column="department_id"/>
        <one-to-many class="Employee"/>
    </bag>
</class>
```

Đặc điểm:
- Ánh xạ sang `java.util.List` hoặc `java.util.Collection`
- Có thể trùng phần tử
- Không có cột thứ tự
- Hibernate coi đây là **unordered collection**

Khi nên dùng:
- Khi dữ liệu có thể trùng, không cần thứ tự
- Khi bạn chỉ cần lưu danh sách đơn giản (ít cần dirty-check)

Ứu điểm:
- Không cần `list-index` → SQL đơn giản hơn
- Insert nhanh hơn `list` vì không quản lý thứ tự

Nhược điểm:
- Hibernate khí kiểm tra thay đổi (dirty-check)
- Dễ bị lỗi "recreate whole bag" khi update → Hibernate xóa hết rồi insert lại

## 6. So sánh tổng thể
| Loại     | Cho phép trùng | Giữ thứ tự | SQL hiệu quả       | Dirty-check   | Khi dùng                           |
| -------- | -------------- | ---------- | ------------------ | ------------- | ---------------------------------- |
| **set**  | ❌ Không        | ❌ Không    | ✅ Tốt              | ✅ Tốt         | Dữ liệu duy nhất, không cần thứ tự |
| **list** | ✅ Có           | ✅ Có       | ⚠️ Trung bình      | ⚠️ Trung bình | Cần giữ thứ tự phần tử             |
| **bag**  | ✅ Có           | ❌ Không    | ✅ Nhanh khi insert | ❌ Kém         | Dữ liệu trùng, không cần thứ tự    |

## 7. Lời khuyên thực tế

| Trường hợp                                                     | Nên dùng                                          |
| -------------------------------------------------------------- | ------------------------------------------------- |
| Danh sách không trùng, không quan trọng thứ tự                 | `<set>`                                           |
| Danh sách có thứ tự rõ ràng (thứ tự nhập, thứ tự xử lý, v.v.)  | `<list>`                                          |
| Danh sách có thể trùng, không cần thứ tự, chỉ cần lưu đơn giản | `<bag>`                                           |
| Quan hệ nhiều-nhiều (join table)                               | `<set>` hoặc `<bag>` (tùy cần duy nhất hay không) |

# Tổng quan: Hibernate xử lý collection khi `save()`
Khi bạn `session.save(parent)`, Hibernate sẽ:
1. Lưu entity cha (nếu mới)
2. Duyệt collection (set, list, bag,..)
3. Sinh ra `INSERT`, `UPDATE`, hoặc `DELETE` cho từng phần tử con
4. Tùy theo loại collection, Hibernate **quản lý khác nhau** (dirty-check, duplicate checking, ordering...)

## 1. `<set>` - Nhanh, sạch, nhưng không trùng
Ví dụ mapping:
```xml
<set name="employees" table="employee" cascade="all" inverse="true">
    <key column="department_id"/>
    <one-to-many class="Employee"/>
</set>
```
Khi `save()`:
```java
Department dept = new Department("IT");
dept.getEmployees().add(new Employee("Alice"));
dept.getEmployees().add(new Employee("Bob"));
session.save(dept);
```
SQL Hibernate sinh ra:
```sql
insert into department (name) values ('IT');
insert into employee (name, department_id) values ('Alice', 1);
insert into employee (name, department_id) values ('Bob', 1);
```

Ưu điểm:
- Hibernate **biết chính xác phần tử nào mới/thêm/xóa**, vì `Set` dựa trên `equals()` và `hashCode()`.
- Khi update collection, Hibernate chỉ **chèn hoặc xóa những phần tử thay đổi, không xóa toàn bộ rồi thêm lại.**
- Dirty-check (so sánh thay đổi) **rất hiệu quả**.

Nhược điểm:
- Không thể chứa phần tử trùng (`equals()` giống nhau).
- Không lưu thứ tự.
- Nếu entity chưa có `equals()` hoặc `hashCode()` chuẩn → dễ bị lỗi cập nhật sai.

**Ví dụ lỗi phổ biến:**
Nếu `Employee` không override `equals()` → Hibernate nghĩ 2 bản ghi là khác nhau → xóa hết rồi thêm lại → mất hiệu năng.

## 2. `<list>` - Giữ tứ tự, nhưng chậm hơn khi update
Ví dụ mapping:
```xml
<list name="employees" table="employee" cascade="all">
    <key column="department_id"/>
    <list-index column="position"/>
    <one-to-many class="Employee"/>
</list>
```
Khi `save()`:
```java
dept.getEmployees().add(new Employee("Alice"));
dept.getEmployees().add(new Employee("Bob"));
session.save(dept);
```
SQL Hibernate sinh ra:
```sql
insert into department (name) values ('IT');
insert into employee (name, department_id, position) values ('Alice', 1, 0);
insert into employee (name, department_id, position) values ('Bob', 1, 1);
```

Ưu điểm:
- Giữ được **thứ tự phần tử** (có `list-index`).
- Cho phép **phần tử trùng**.

Nhược điểm khi update:
- Nếu bạn **thêm/xóa ở giữa danh sách**, Hibernate sẽ:
    - Cập nhật lại tất cả `list-index` sau đó
    - Có thể sinh **nhiều câu `UPDATE` không cần thiết**
    Ví dụ:
    ```java
    dept.getEmployees().add(0, new Employee("Charlie"));
    ```
    → Hibernate sinh:
    ```sql
    insert into employee (name, department_id, position) values ('Charlie', 1, 0);
    update employee set position=1 where id=1;
    update employee set position=2 where id=2;
    ```
    => Mất hiệu năng khi danh sách dài.
Lưu ý:
- Hibernate luôn phải duy trì thứ tự nên dirty-check phức tạp hơn.
- Dễ lỗi khi trộn thao tác thêm/xóa nhiều phần tử.

## 4. `<bag>` - Đơn giản nhất khi insert, nhưng cực tệ khi update
Ví dụ mapping:
```xml
<bag name="employees" table="employee" cascade="all">
    <key column="department_id"/>
    <one-to-many class="Employee"/>
</bag>
```
Khi `save()`:
```java
dept.getEmployees().add(new Employee("Alice"));
dept.getEmployees().add(new Employee("Bob"));
session.save(dept);
```
Hibernate sinh:
```sql
insert into department (name) values ('IT');
insert into employee (name, department_id) values ('Alice', 1);
insert into employee (name, department_id) values ('Bob', 1);
```
=> Nhanh và đơn giản
___
Nhưng khi `update()` collection:
```java
dept.getEmployees().remove(0);
dept.getEmployees().add(new Employee("Charlie"));
session.update(dept);
```
Hibernate KHÔNG thể biết phần tử nào bị thay đổi
→ Nên nó sẽ:
```sql
delete from employee where department_id=1;
insert into employee (name, department_id) values ('Charlie', 1);
insert into employee (name, department_id) values ('Bob', 1);
```
Hibernate xóa toàn bộ và insert lại tất cả phẩn tử!
___
Ưu điểm:
- Khi **insert ban đầu** → rất nhanh, SQL đơn giản
- Không cần `equals()` hoặc `list-index`

Nhược điểm:
- Khi update → **xóa toàn bộ và chèn lại toàn bộ**
- Không có thứ tự
- Không thể dirty-check từng phần tử
- Không hiệu quả khi dữ liệu lớn

## 5. So sánh khi `save` / `update`
| Collection | Khi Save mới                | Khi Update                     | Có thứ tự | Có phần tử trùng | Hiệu năng       |
| ---------- | --------------------------- | ------------------------------ | --------- | ---------------- | --------------- |
| **Set**    | Rất nhanh, insert chính xác | Cập nhật đúng phần tử thay đổi | ❌ Không   | ❌ Không          | ✅ Tốt           |
| **List**   | Chậm hơn (thêm cột index)   | Có thể update hàng loạt index  | ✅ Có      | ✅ Có             | ⚠️ Trung bình   |
| **Bag**    | Nhanh nhất khi insert       | Xóa toàn bộ rồi insert lại     | ❌ Không   | ✅ Có             | ❌ Tệ khi update |

## 6. Khi nào nên chọn loại save/update thường xuyên
| Tình huống thực tế                                                     | Loại nên dùng | Lý do                       |
| ---------------------------------------------------------------------- | ------------- | --------------------------- |
| Quan hệ 1-nhiều dữ liệu ít thay đổi (ví dụ: `Department` – `Employee`) | `<set>`       | Tối ưu update               |
| Danh sách có thứ tự rõ ràng (ví dụ: `Survey` – `Question`)             | `<list>`      | Giữ thứ tự câu hỏi          |
| Dữ liệu chỉ insert 1 lần, ít update (ví dụ: `Order` – `OrderItem`)     | `<bag>`       | Insert nhanh                |
| Quan hệ nhiều-nhiều (user-role, tag-post)                              | `<set>`       | Tránh trùng, update ổn định |

## 7. Tổng kết nhanh
| Loại     | Ưu điểm chính                           | Nhược điểm chính           | Dùng khi                             |
| -------- | --------------------------------------- | -------------------------- | ------------------------------------ |
| **Set**  | Dễ dirty-check, insert/update chính xác | Không trùng, không thứ tự  | Quan hệ ổn định, dữ liệu duy nhất    |
| **List** | Giữ thứ tự                              | Cập nhật index tốn chi phí | Khi thứ tự quan trọng                |
| **Bag**  | Insert nhanh                            | Update xóa toàn bộ         | Khi chỉ cần thêm dữ liệu (ít update) |

