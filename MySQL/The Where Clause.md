### The equality operator
* **Primary key**
Giả sử chúng ta có bảng `Employees`  như sau:
``` SQL
CREATE TABLE employees (
   employee_id   NUMBER         NOT NULL,
   first_name    VARCHAR2(1000) NOT NULL,
   last_name     VARCHAR2(1000) NOT NULL,
   date_of_birth DATE           NOT NULL,
   phone_number  VARCHAR2(1000) NOT NULL,
   CONSTRAINT employees_pk PRIMARY KEY (employee_id)
)
```
DB sẽ tự động tạo index cho primary key mặc dù không có statement `create index`.

Query sau sẽ lấy ra employee name:
``` SQL
SELECT first_name, last_name
  FROM employees
 WHERE employee_id = 123
```
Với câu query này thì mệnh đề `where` sẽ không thể match nhiều rows vì primary key constraint (ràng buộc) đảm bảo các employee_id là duy nhất. DB sẽ không cần follow theo index leaf nodes mà chỉ cần duyệt qua index tree.

Chúng ta có thể dùng execution plan (explaination plan) để xem:
```
+----+-----------+-------+---------+---------+------+-------+
| id | table     | type  | key     | key_len | rows | Extra |
+----+-----------+-------+---------+---------+------+-------+
|  1 | employees | const | PRIMARY | 5       |    1 |       |
+----+-----------+-------+---------+---------+------+-------+
Type const is MySQL’s equivalent of Oracle’s INDEX UNIQUE SCAN.
```
`INDEX UNIQUE SCAN` chỉ duyệt qua index tree. Nó sẽ tối ưu được sự tăng trưởng của index để tìm kiếm nhanh chóng và không phụ thuộc vào lượng dữ liệu của bảng (table size).

Sau đó, DB sẽ phải thêm một bước để lấy dữ liệu (First_name và last_name) từ bảng dữ liệu: `TABLE ACCESS BY INDEX ROWID`. Ở bước này có thể tạo ra nút thắt cổ chai cho việc tối ưu (performance bottleneck) nhưng sẽ không có rủi ro trong việc kết nối với INDEX UNIQUE SCAN. Thao tác này sẽ chỉ tạo ra entry nên nó sẽ chỉ có thể trigger 1 lần truy xuất dữ liệu với bảng.
* **Concatenated Indexes**
Mặc dù DB tự động tạo index cho primary key nhưng vẫn có một vài trường hợp chúng ta cần tự tạo index nếu index cho nhiều cột. Trong trường hợp này DB sẽ tạo index trên tất cả primary key, thứ tự của các cột sẽ ảnh hưởng rất lớn đến việc sử dụng index nên hãy cẩn trọng trong việc sắp xếp.

Giả sử chúng ta cần lưu thêm employee của công ty khác vào bảo employee như vậy sẽ không đảm bảo primary unique giữa 2 công ty. Chúng ta sẽ thêm một primary key nữa đó là subsidiary id. Chúng ta sẽ tạo index như sau:
``` SQL
CREATE UNIQUE INDEX employees_pk
    ON employees (employee_id, subsidiary_id)
```
Khi chúng ta truy vấn một employee cụ thể như sau:
``` SQL
SELECT first_name, last_name
  FROM employees
 WHERE employee_id   = 123
   AND subsidiary_id = 30
```
ở trường hợp này thì DB có thể dùng `INDEX UNIQUE SCAN` mà không cần quan tâm tới bao nhiêu cột được index. Nhưng điều gì sẽ xảy ra nếu sử dụng 1 key colums, ví dụ như khi tìm kiếm tất cả employee của một subsidiary ?
``` SQL
SELECT first_name, last_name
  FROM employees
 WHERE subsidiary_id = 20
```
Khi dùng explain chúng ta có thể nhận thấy rằng DB không dùng index mà thay vào đó là `TABLE ACCESS FULL`. Nó sẽ đọc toàn bộ bảng và đánh giá từng row với mệnh đề where. Thời gian thực thi sẽ phụ thuộc vào dữ liệu của bảng.

Concatenated index cũng là một B-tree index, lưu trữ indexed data ở trong một sorted list. DB sẽ cân nhắc mỗi cột dựa vào vị trí của nó trong định nghĩa index để sắp xếp index entries. Cột đầu tiên là primary sort criterion và cột thứ 2 sẽ quyết định order chỉ khi cả 2 entries có cùng value với cột đầu tiên.

**NOTE**: Một concatenated index có nghĩa là 1 index ở nhiều cột. (A concatenated index is one index across multiple columns.)

Thứ tự index của 2 cột vì vậy mà cũng giống như thứ tự của cuốn danh bạ điện thoại: đầu tiên sẽ sắp xếp bằng tên đệm, sau đó là tên. Điều này đồng nghĩa với việc nó sẽ không hỗ trợ việc tìm kiếm với duy chỉ có cột thứ 2, cũng giống như tìm kiếm số điện thoại theo tên.