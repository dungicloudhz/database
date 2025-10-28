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
- `varchar(n)` - chuỗi có giới hạn.
- `text` - chuỗi dài không giới hạn.
- `boolean` - true/false.
- `timestamp without time zone` - thời gian không kèm TZ.
- `timestamp with time zone` / `timestamptz` - thời gian có TZ.
- `date` - ngày.
- `numeric(p,s)` - số thập phân chính xác (money, rate).
- `uuid` - UUID (khuyến nghị cho id phân tán).
- `json`/ `jsonb` - lưu JSON (jsonb thường tốt hơn).
- `bytea` - blob / nhị phân.

# 3. Chi tiết các tùy chọn cột thương dùng
- `autoIncrement="true"`: Liquibase sẽ tạo cột tự tăng (vendor-specific). Với Postgres, `bigserial` là an toan.
- `defaultValue="..."`/ `defaultValueNumeric="..."`/ `defaultValueComputed="CURRENT_TIMESTAMP"`: gán giá trị mặc định.
- `<constraints primaryKey="true" nullable="false" unique="true">`
- `remarks="..."` trong `<column>` (thêm comment nếu driver hỗ trợ).
- `type` viết theo SQL DB (ví dụ `varchar(100)`, `timestampz`, `jsonb`)

# 4. Cách định nghĩa quan hệ (FK) - ví dụ mẫu hoàn chỉnh
Đây là một file changelog mãu gồm 3 bảng: `health_declarations`, `health_declaration_awsers`, `health_declaration_answer_children`.
Mình sẽ flow file XML hoàn chỉnh + giải thích từng phần dưới.
```postgreSql
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
      http://www.liquibase.org/xml/ns/dbchangelog
      http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd">

    <!-- CHANGESET 1: create health_declarations -->
    <changeSet id="1-create-health-declarations" author="top_dungnd">
        <preConditions onFail="MARK_RAN">
            <not><tableExists tableName="health_declarations"/></not>
        </preConditions>

        <createTable tableName="health_declarations">
            <!-- 1) id: kiểu bigint tự tăng (Postgres dùng bigserial) -->
            <column name="id" type="bigserial">
                <constraints primaryKey="true" nullable="false"/>
            </column>

            <!-- 2) order_number: số nguyên -->
            <column name="order_number" type="integer"/>

            <!-- 3) code: varchar(50) -->
            <column name="code" type="varchar(50)">
                <constraints nullable="false" unique="true"/>
            </column>

            <!-- 4) content: text (nội dung câu hỏi) -->
            <column name="content" type="text"/>

            <!-- 5) input_type: varchar tên loại input (radio, checkbox, text...) -->
            <column name="input_type" type="varchar(50)"/>

            <!-- 6) audit timestamps -->
            <column name="created_date_time" type="timestamptz" defaultValueComputed="CURRENT_TIMESTAMP"/>
            <column name="modified_date_time" type="timestamptz"/>
        </createTable>

        <rollback>
            <dropTable tableName="health_declarations"/>
        </rollback>
    </changeSet>

    <!-- CHANGESET 2: create health_declaration_answers and FK -> health_declarations -->
    <changeSet id="2-create-health-declaration-answers" author="top_dungnd">
        <preConditions onFail="MARK_RAN">
            <not><tableExists tableName="health_declaration_answers"/></not>
            <tableExists tableName="health_declarations"/>
        </preConditions>

        <createTable tableName="health_declaration_answers">
            <column name="id" type="bigserial">
                <constraints primaryKey="true" nullable="false"/>
            </column>

            <!-- declaration_id: sẽ là FK tới health_declarations.id -->
            <column name="declaration_id" type="bigint">
                <constraints nullable="false"/>
            </column>

            <column name="default_answer" type="boolean" defaultValueBoolean="false"/>
            <column name="data_type" type="varchar(50)"/>
            <column name="format_pattern" type="text"/>
            <column name="content" type="text"/>
            <column name="detail_explanation_required" type="boolean" defaultValueBoolean="false"/>
            <column name="created_date_time" type="timestamptz" defaultValueComputed="CURRENT_TIMESTAMP"/>
            <column name="modified_date_time" type="timestamptz"/>
        </createTable>

        <!-- Tạo index trên declaration_id để tăng hiệu suất join -->
        <createIndex tableName="health_declaration_answers" indexName="idx_hda_declaration_id">
            <column name="declaration_id"/>
        </createIndex>

        <!-- Thêm FK constraint riêng (dễ quản lý khi rollback/tách changeSet) -->
        <addForeignKeyConstraint
            baseTableName="health_declaration_answers"
            baseColumnNames="declaration_id"
            constraintName="fk_hda_declaration"
            referencedTableName="health_declarations"
            referencedColumnNames="id"
            onDelete="CASCADE"
            onUpdate="NO ACTION"/>

        <rollback>
            <dropTable tableName="health_declaration_answers"/>
        </rollback>
    </changeSet>

    <!-- CHANGESET 3: create children table (junction) -->
    <changeSet id="3-create-answer-children" author="top_dungnd">
        <preConditions onFail="MARK_RAN">
            <not><tableExists tableName="health_declaration_answer_children"/></not>
            <tableExists tableName="health_declaration_answers"/>
            <tableExists tableName="health_declarations"/>
        </preConditions>

        <createTable tableName="health_declaration_answer_children">
            <column name="id" type="bigserial">
                <constraints primaryKey="true" nullable="false"/>
            </column>

            <!-- FK tới answer -->
            <column name="answer_id" type="bigint">
                <constraints nullable="false"/>
            </column>

            <!-- FK tới child declaration (ví dụ trả lời này mở thêm câu hỏi con) -->
            <column name="child_declaration_id" type="bigint">
                <constraints nullable="false"/>
            </column>

            <column name="created_date_time" type="timestamptz" defaultValueComputed="CURRENT_TIMESTAMP"/>
            <column name="modified_date_time" type="timestamptz"/>
        </createTable>

        <createIndex tableName="health_declaration_answer_children" indexName="idx_hdac_answer_id">
            <column name="answer_id"/>
        </createIndex>

        <createIndex tableName="health_declaration_answer_children" indexName="idx_hdac_child_decl_id">
            <column name="child_declaration_id"/>
        </createIndex>

        <addForeignKeyConstraint
            baseTableName="health_declaration_answer_children"
            baseColumnNames="answer_id"
            constraintName="fk_hdac_answer"
            referencedTableName="health_declaration_answers"
            referencedColumnNames="id"
            onDelete="CASCADE"/>

        <addForeignKeyConstraint
            baseTableName="health_declaration_answer_children"
            baseColumnNames="child_declaration_id"
            constraintName="fk_hdac_child_declaration"
            referencedTableName="health_declarations"
            referencedColumnNames="id"
            onDelete="RESTRICT"/>

        <rollback>
            <dropTable tableName="health_declaration_answer_children"/>
        </rollback>
    </changeSet>

</databaseChangeLog>

```
**Giải thích chi tiết phân quan trọng trong ví dụ**
- Tách mỗi bảng vào một `changeSet` riêng: dễ đọc, rollback, quản lý.
- `preConditions` dùng để tránh lỗi khi bảng đã tồn tại; `onFail="MARK_RAN"` đánh dấu changeSet là đã chạy thay vì dừng app.
- Dùng `bigserial` cho id trong PostgreSQL; nếu bạn muốn cross-venor, có thể dùng `autoIncrement="true"` và `type="bigint"`
- Tạo index cho các cột FK để cải thiện hiệu suất join.
- Tạo FK bằng `<addForeignKeyConstraint>` thay vì thêm trong `<column>` để control rõ ràng (tên constraint, onDelete,...).
- `onDelete="CASCADE"` nghĩa khi dòng parent bị xóa thì các child liên quan bị xóa theo; `RESTRICT` ngĩa cấm xóa nếu có child.

# 5. Các tình huống đặc biệt va cách xử lý
## 5.1 Nếu cần sequence riêng (Postgre)
```sql
<changeSet id="create-sequence-sample" author="me">
    <createSequence sequenceName="seq_user_id" startValue="1000"/>
    <rollback>
        <dropSequence sequenceName="seq_user_id"/>
    </rollback>
</changeSet>
```
Rồi dùng `defaultValueComputed="nextval('seq_user_id')"` cho cột nếu cần.

## 5.2 Enum / giá trị cố định
Postgre có `CREATE TYPE ... AS ENUM`. Bạn có 2 cách:
- Dùng `check constraint` (portable):
```sql
<addCheckConstraint tableName="users" constraintName="chk_user_status" checkCondition="status IN ('ACTIVE', 'INACTIVE', 'BLOCKED')" />
```
- Hoặc tạo `enum` Postgres (không portable):
```sql
<sql>CREATE TYPE user_status AS ENUM ('ACTIVE', 'INACTIVE', 'BLOCKED');</sql>
```

## 5.3 Cập nhật kiểu dữ liệu / đặt NOT NULL cho cột có dữ liệu
Nếu muốn set `NOT NULL` trên cột đã có dữ liệu, cần:
1. Update các row có NULL sang giá trị hợp lệ.
2. Sau đó dùng `<addNotNullConstraint>`.

Ví dụ:
```sql
<changeSet id="make-code-not-null" author="me">
    <update tableName="health_declarations">
        <column name="code" value="UNKNOWN"/>
        <where>code IS NULL</where>
    </update>

    <addNotNullConstraint tableName="health_declarations" columnName="code" columnDataType="varchar(50)"/>

    <rollback>
        <dropNotNullConstraint tableName="health_declarations" columnName="code" columnDataType="varchar(50)"/>
    </rollback>
</changeSet>
```

## 5.4 Data migration (insert/update) kèm rollback
Luôn cung cấp rollback nếu có thể:
```sql
<changeSet id="insert-sample" author="me">
    <insert tableName="health_declarations">
        <column name="order_number" valueNumeric="1"/>
        <column name="code" value="HD-001"/>
        <column name="content" value="Bạn có triệu chứng sốt?"/>
        <column name="input_type" value="radio"/>
    </insert>

    <rollback>
        <delete tableName="health_declarations">
            <where>code='HD-001'</where>
        </delete>
    </rollback>
</changeSet>
```

# 6. Một số thẻ / attributes nâng cao bạn sẽ rất hay dùng
- `context` trêm `<changeSet>`: chỉ chạy khi bạn Liquibbase với context tương ứng (vd `dev`, `prod`).
```sql
<changeSet id="only-prod" author="me" context="prod">
   ...
</changeSet>
```
- `lables`: phân nhãn thay vì context (cách khác để filter).
- `runOnChange="true"` trên `<changeSet>`: nếu nội dung changeSet thay đổi, Liquibase sẽ chạy lại - cẩn trọng.
- `runAlways="true"`: luôn chạy mỗi lần `update` (dùng cho migration dữ liệu mà bạn muốn chạy every deploy)
- `splitStatement="false"` trong `<sql>` nếu bạn muốn chạy nhiều câu lệnh trong 1 batch ma không bị tách.

# 7. Lời khuyên & best practive (tóm tắt)
1. Mỗi changeSet làm 1 việc logic → dễ rollback & track.
2. Không sửa changeSet đã deploy (checksum mismatch). Nếu cần sửa, tạo changeSet mới.
3. Dùng preConditions hợp lý để an toàn khi chạy trên nhiều môi trường.
4. Test migration trên dev/staging trước khi lên prod.
5. Dùng tên constraint rõ ràng: fk_<child>_<parent>; idx_<table>_<col>.
6. Đặt spring.jpa.hibernate.ddl-auto=none nếu dùng Hibernate + Liquibase.
7. Luôn commit changelog vào VCS (Git).