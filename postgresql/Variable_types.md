# 1. UUID
- Sinh UUID ngẫu nhiên:
```sql
select gen_random_uuid();
```
# 2. Array
| Hàm / Toán tử             | Mô tả                         |
| ------------------------- | ----------------------------- |
|`array_sort(arr)`|Sắp xếp mảng|
|`array_append(arr, val)`|Thêm phần tử vào cuối mảng|
|`array_prepend(val, arr)`|Thêm phần tử vào đầu|
|`array_remove(arr, val)`|Xóa tất cả các phần tử bằng `val`|
|`unnest(arr)`|Trải mảng thành nhiều dòng|
|`array_length(arr, dim)`|Độ dài mảng (theo chiều dim)|
|`arr1 @> arr2`|Mảng bên trái chứa mảng bên phải|
|`arr1 <@ arr2`|Mảng bên trái được chứa trong mảng bên phải|
|`arr1 && arr2`|Hai mảng có phần tử chung|
|`val = ANY(arr)`|Kiểm tra val cáo trong arr|

Ví dụ:
```sql
-- Tạo bảng với mảng
CREATE TABLE products (
  id SERIAL,
  name TEXT,
  tags TEXT[]
);

INSERT INTO products (name, tags) VALUES
('Laptop', ARRAY['electronics','office']),
('Chair', ARRAY['furniture','wood']),
('Phone', ARRAY['electronics','mobile']);

-- Lấy sản phẩm có tag 'electronics'
SELECT * FROM products WHERE 'electronics' = ANY(tags);

-- Có tất cả ['electronics','office']
SELECT * FROM products WHERE tags @> ARRAY['electronics','office'];

-- Có ít nhất 1 phần tử chung
SELECT * FROM products WHERE tags && ARRAY['mobile','wood'];

-- Trải tags thành dòng
SELECT name, unnest(tags) FROM products;

-- Thêm 1 tag
UPDATE products SET tags = array_append(tags, 'discount') WHERE name='Laptop';
```
# 3. JSON / JSONB
| Hàm / Toán tử             | Mô tả                         |
| ------------------------- | ----------------------------- |
|`->`|Lấy JSON object/array|
|`->>`|Lấy text từ JSON|
|`jsonb_set(jsonb, '{path}', 'value')`|Update giá trị|
|`jsonb_insert(jsonb, '{path}', 'value')`|Insert giá trị|
|`- 'key'`|Xóa key|
|`?`|Có key hay không|
|`@>`|JSON bên trái chứa JSON bên phải|
|`jsonb_each()`|Trả về key-value cặp|
|`jsonb_array_elements()`|Trả array JSON thành nhiều dòng|

Ví dụ:
```sql
-- Bảng JSONB
CREATE TABLE orders (
  id SERIAL,
  data JSONB
);

INSERT INTO orders (data) VALUES
('{"item":"Laptop","price":1200,"details":{"brand":"Dell","color":"black"}}'),
('{"item":"Phone","price":800,"details":{"brand":"Apple","color":"white"}}');

-- Lấy giá trị JSON
SELECT data->>'item' AS item, data->'details'->>'brand' AS brand FROM orders;

-- Lọc order có giá >= 1000
SELECT * FROM orders WHERE (data->>'price')::int >= 1000;

-- Update field JSON
UPDATE orders SET data = jsonb_set(data,'{details,color}','"silver"') WHERE id=1;

-- Kiểm tra key
SELECT * FROM orders WHERE data ? 'price';
```

# 4. HSTORE
| Hàm / Toán tử             | Mô tả                         |
| ------------------------- | ----------------------------- |
|`->`|Lấy value theo key|
|`- 'key'`|Xóa key|
|`akeys(hstore)`|Trả về danh sách key|
|`avals(hstore)`|Trả về danh sách value|
|`hstore_to_json(hstore)`|Convert sang JSON|

Ví dụ:
```sql
CREATE EXTENSION IF NOT EXISTS hstore;

CREATE TABLE profiles (
  id SERIAL,
  info HSTORE
);

INSERT INTO profiles (info) VALUES
('name=>"Alice", email=>"alice@mail.com"'),
('name=>"Bob", phone=>"12345"');

-- Lấy key
SELECT info->'email' AS email FROM profiles;

-- Lấy danh sách key, value
SELECT akeys(info), avals(info) FROM profiles;

-- Thêm trường mới
UPDATE profiles SET info = info || 'address=>"Hanoi"' WHERE id=1;
```
# 5. ENUM
- So sánh trực tiếp (`=`, `<`, `>`)
- Convert sang text:
```sql
select state::text from people;
```
- Liệt kê giá trị enum:
```sql
select enum_range(NULL::mood);
```
Ví dụ:
```sql
CREATE TYPE mood AS ENUM ('happy','sad','ok');

CREATE TABLE people (
  id SERIAL,
  name TEXT,
  state mood
);

INSERT INTO people (name, state) VALUES ('Alice','happy'), ('Bob','sad');

-- So sánh enum
SELECT * FROM people WHERE state = 'happy';

-- Liệt kê toàn bộ enum
SELECT enum_range(NULL::mood);

```
# 6. Range types
| Toán tử        | Mô tả                         |
| -------------- | ----------------------------- |
| `@>`           | Range chứa giá trị/Range khác |
| `<@`           | Giá trị nằm trong range       |
| `&&`           | Hai range chồng lấn           |
| `lower(range)` | Giá trị đầu                   |
| `upper(range)` | Giá trị cuối                  |

Ví dụ:
```sql
-- Bảng với daterange
CREATE TABLE reservations (
  id SERIAL,
  room TEXT,
  during DATERANGE
);

INSERT INTO reservations (room,during) VALUES
('A', daterange('2025-09-01','2025-09-10')),
('A', daterange('2025-09-15','2025-09-20'));

-- Kiểm tra ngày có trong range
SELECT * FROM reservations WHERE during @> DATE '2025-09-05';

-- Kiểm tra overlap
SELECT * FROM reservations WHERE during && daterange('2025-09-08','2025-09-12');
```
 # 7. Network types (`inet`, `cidr`, `macaddr`)
 | Hàm / Toán tử   | Mô tả                |
| --------------- | -------------------- |
| `<<=`           | Địa chỉ thuộc subnet |
| `>>=`           | Subnet chứa địa chỉ  |
| `host(inet)`    | Trả về IP            |
| `masklen(inet)` | Trả về prefix length |
| `family(inet)`  | IPv4 hay IPv6        |

Ví dụ:
```sql
CREATE TABLE devices (
  id SERIAL,
  ip inet
);

INSERT INTO devices (ip) VALUES
('192.168.1.10'),
('192.168.1.50'),
('10.0.0.5');

-- IP có trong subnet
SELECT * FROM devices WHERE ip <<= '192.168.1.0/24';

-- Trả về host và masklen
SELECT host(ip), masklen(ip), family(ip) FROM devices;
```
# 8. Geometric types
| Hàm                | Mô tả         |
| ------------------ | ------------- |
| `area(polygon)`    | Diện tích     |
| `center(circle)`   | Tâm hình tròn |
| `diameter(circle)` | Đường kính    |
| `point(x,y)`       | Tạo điểm      |
| `circle(point,r)`  | Tạo hình tròn |

Ví dụ:
```sql
-- Tạo bảng
CREATE TABLE shapes (
  id SERIAL,
  rect box,
  circle circle
);

INSERT INTO shapes (rect, circle) VALUES
(box '((0,0),(5,5))', circle '((2,2),3)');

-- Diện tích hình chữ nhật
SELECT area(rect) FROM shapes;

-- Tâm hình tròn
SELECT center(circle), diameter(circle) FROM shapes;
```

# 9. Full-text Search (`tsvector`, `tsquery`)
| Hàm / Toán tử            | Mô tả                        |
| ------------------------ | ---------------------------- |
|`to_tsvector(text)`|Convert text -> tsvector|
|`to_tsquery(text)`|Convert string -> tsquery|
|`@@`|So khớp tsvector với tsquery|
|`ts_rank(vector, query)`|Ranking kết quả|
|`plainto_tsquery(text)`|Convert plain text -> tsquery|

Ví dụ:
```sql
CREATE TABLE documents (
  id SERIAL,
  content TEXT
);

INSERT INTO documents (content) VALUES
('PostgreSQL is a powerful database'),
('I love working with JSON and arrays'),
('Text search is very useful in PostgreSQL');

-- Tìm kiếm
SELECT * FROM documents
WHERE to_tsvector('english',content) @@ to_tsquery('postgresql & database');

-- Ranking
SELECT content, ts_rank(to_tsvector(content), to_tsquery('postgresql')) AS rank
FROM documents
ORDER BY rank DESC;
```

✅ Tóm tắt

Các kiểu dữ liệu đặc biệt có hàm/toán tử riêng để làm việc hiệu quả:
- **Array**: `ANY`, `@>`, `&&`, `unnest`, `array_append`
- **JSONB**: `->`, `->>`, `jsonb_set`, `jsonb_each`, `@>`
- **HSTORE**: `->`, `||`, `akeys`, `avals`
- **ENUM**: `enum_range`
- **Range**: `@>`, `&&`, `lower()`, `upper()`
- **Network**: `<<=`, `host()`, `masklen()`
- **Geometric**: `area()`, `center()`, `point()`
- **Full-text**: `to_tsvector`, `to_tsquery`, `@@`, `ts_rank`