Dù ngày nay đa phần dự án dùng annotation, nhưng XML mapping **vẫn rất mạnh** - nhất là khi bạn muốn:
- Giữ **tách biệt giữa logic Java và cấu trúc DB**
- Dễ **tùy biến mapping** mà không cần sửa code
- Dễ **tích hợp hệ thống cũ** hoặc **generated entity tự động**

# Múc tiêu bài này
1. Cách viết **Liquibase XML** để tạo bảng
2. Cách viết **Hibernate mapping XML** (`.hbm.xml`) tương ứng
3. Cách **cấu hình Hibernate** để đọc mapping XML
4. Cách **liên kết bảng (one-to-many, many-to-one)** trong XML

# 1. Liquibase: Tạo bảng `users` và `posts`
> `src/main/resources/db/changelog/db.changelog-1.0.xml`
```xml
<databaseChangeLog
        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
   http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd">

<!-- 1. Tạo bảng users -->
    <changeSet id="1" author="dungnd">
        <createTable tableName="users">
            <column name="id" type="bigint" autoIncrement="true">
                <constraints primaryKey="true" nullable="false"/>
            </column>
            <column name="username" type="varchar(100)">
                <constraints nullable="false" unique="true"/>
            </column>
            <column name="email" type="varchar(150)"/>
            <column name="created_at" type="timestamptz" defaultValueComputed="CURRENT_TIMESTAMP"/>
        </createTable>
    </changeSet>

<!-- 2. Bảng posts -->
    <changeSet id="2" author="dung">
        <createTable tableName="posts">
            <column name="id" type="bigint" autoIncrement="true">
                <constraints primaryKey="true" nullable="false"/>
            </column>
            <column name="user_id" type="bigint">
                <constraints nullable="false" foreignKeyName="fk_post_user" references="users(id)"/>
            </column>
            <column name="title" type="varchar(200)"/>
            <column name="content" type="text"/>
        </createTable>
    </changeSet>

</databaseChangeLog>
```
Giải thích:
- `users` là bảng cha (User)
- `posts` là bảng con (Post) - có `user_id` là khóa ngoại trỏ đến `user(id)`

# 2. Hibernate Mapping XML (`.hbm.xml`)
Giả sử bạn có 2 class Java tương ứng:
```pgsql
User.java
Post.java
```
Bạn sẽ **không dùng annotation**, mà tạo file `.hbm.xml` để Hibernate biết cách map.

## 1. User.hbm.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">

<hibernate-mapping package="com.example.model">
    <class name="id" table="users">
        <id name="id" column-"id" type="long">
            <generator class="identity">
        </id>
        <property name="username" column="username" type="string" not-null="true" unique="true" />
        <property name="email" column="email" type="string" />
        <property name="createAt" column="create_at" type="timesamp" />

        <!-- Quan hệ 1-nhiều: 1 User có nhiều Post -->
        <set name="posts" table="posts" inverse="true" lazy="true" casecade="all"/>
            <key column="user_id" />
            <one-to-many class="Post" />
        </set>

    </class>
</hibernate-mapping>
```
✅ **Giải thích:**
| Thành phần                      | Ý nghĩa                                                                    |
| ------------------------------- | -------------------------------------------------------------------------- |
| `<hibernate-mapping>`           | File gốc chứa tất cả mapping.                                              |
| `package`                       | Package Java chứa entity.                                                  |
| `<class>`                       | Đại diện cho một bảng trong DB.                                            |
| `<id>`                          | Cột khóa chính.                                                            |
| `<generator class="identity"/>` | ID tự tăng (tương ứng với `autoIncrement` trong DB).                       |
| `<property>`                    | Một cột thông thường.                                                      |
| `<set>`                         | Quan hệ **one-to-many** (1 User có nhiều Post).                            |
| `inverse="true"`                | Nghĩa là quan hệ này được quản lý ở phía `Post` (tránh lặp insert/update). |
| `<key>`                         | Cột khóa ngoại ở bảng con.                                                 |
| `<one-to-many>`                 | Class con tương ứng.                                                       |

## 2. `Post.hbm.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">

<hibernate-mapping package="com.example.model">

    <class name="Post" table="posts">

        <id name="id" column="id" type="long">
            <generator class="identity"/>
        </id>

        <property name="title" column="title" type="string"/>
        <property name="content" column="content" type="text"/>

        <!-- Quan hệ nhiều-1: nhiều Post thuộc 1 User -->
        <many-to-one name="user" class="User" column="user_id" not-null="true"/>

    </class>

</hibernate-mapping>
```
✅ **Giải thích:**
| Thành phần         | Ý nghĩa                                    |
| ------------------ | ------------------------------------------ |
| `<many-to-one>`    | Quan hệ nhiều-1, mỗi post thuộc về 1 user. |
| `class="User"`     | Tên class cha tương ứng.                   |
| `column="user_id"` | Cột khóa ngoại trong DB.                   |

# 3. Hai class Java tương ứng
`User.java`
```java
package com.example.model;

import java.time.OffsetDateTime;
import java.util.Set;

public class User {
    private Long id;
    private String username;
    private String email;
    private OffsetDateTime createdAt;
    private Set<Post> posts;

    // getters & setters
}
```
`Post.java`
```java
package com.example.model;

public class Post {
    private Long id;
    private String title;
    private String content;
    private User user;

    // getters & setters
}
```