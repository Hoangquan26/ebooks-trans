# Chương 2: Mệnh đề WHERE (Chapter 2 – The Where Clause) — Phần 3

> Phần này tương ứng với các trang PDF 63–88 của sách *SQL Performance Explained* (Markus Winand). Nội dung gồm hai mục lớn: cách Oracle xử lý `NULL` (đặc biệt là trong index) và các "điều kiện bị che khuất" (*obfuscated conditions*) — những cách viết mệnh đề `WHERE` khiến index không được dùng đúng cách.

## Mục lục

- [NULL trong Oracle Database (NULL in the Oracle Database)](#null-trong-oracle-database-null-in-the-oracle-database)
  - [Đánh index cho NULL (Indexing NULL)](#đánh-index-cho-null-indexing-null)
  - [Ràng buộc NOT NULL (NOT NULL Constraints)](#ràng-buộc-not-null-not-null-constraints)
  - [Giả lập partial index (Emulating Partial Indexes)](#giả-lập-partial-index-emulating-partial-indexes)
- [Các điều kiện bị che khuất (Obfuscated Conditions)](#các-điều-kiện-bị-che-khuất-obfuscated-conditions)
  - [Kiểu ngày tháng (Date Types)](#kiểu-ngày-tháng-date-types)
  - [Chuỗi chứa số (Numeric Strings)](#chuỗi-chứa-số-numeric-strings)
  - [Kết hợp nhiều cột (Combining Columns)](#kết-hợp-nhiều-cột-combining-columns)
  - [Logic "thông minh" (Smart Logic)](#logic-thông-minh-smart-logic)
  - [Phép toán (Math)](#phép-toán-math)
- [Bảng thuật ngữ (Glossary)](#bảng-thuật-ngữ-glossary)

---

## NULL trong Oracle Database (NULL in the Oracle Database)

`NULL` của SQL thường xuyên gây bối rối. Mặc dù ý tưởng cơ bản của `NULL` — dùng để biểu diễn dữ liệu bị thiếu (*missing data*) — khá đơn giản, nó lại có một số điểm kỳ quặc. Ví dụ, bạn phải dùng `IS NULL` thay vì `= NULL`. Hơn nữa, Oracle Database còn có thêm những điểm kỳ quặc riêng về `NULL`: một mặt là vì nó không phải lúc nào cũng xử lý `NULL` đúng như chuẩn SQL yêu cầu, mặt khác là vì nó có một cách xử lý `NULL` rất "đặc biệt" trong index.

Chuẩn SQL không định nghĩa `NULL` là một *giá trị* mà là một **chỗ giữ chỗ** (*placeholder*) cho một giá trị bị thiếu hoặc chưa biết. Do đó, không có giá trị nào có thể *là* `NULL`. Thế nhưng Oracle Database lại coi một chuỗi rỗng (*empty string*) là `NULL`:

```sql
   SELECT     '0 IS NULL???' AS "what is NULL?" FROM dual
    WHERE      0 IS NULL
UNION ALL
   SELECT    '0 is not null' FROM dual
    WHERE     0 IS NOT NULL
UNION ALL
   SELECT ''''' IS NULL???'  FROM dual
    WHERE    '' IS NULL
UNION ALL
   SELECT ''''' is not null' FROM dual
    WHERE    '' IS NOT NULL;
```

```text
what is NULL?
--------------
0 is not null
'' IS NULL???
```

Kết quả cho thấy: số `0` không phải `NULL` (đúng như mong đợi), nhưng chuỗi rỗng `''` lại **là** `NULL` trong Oracle.

Để làm mọi chuyện rối thêm, thậm chí còn có trường hợp Oracle Database coi `NULL` là chuỗi rỗng:

```sql
SELECT dummy
     , dummy || ''
     , dummy || NULL
  FROM dual;
```

```text
D D D
- - -
X X X
```

Nối (*concatenate*) cột `DUMMY` (luôn chứa `'X'`) với `NULL` lẽ ra phải trả về `NULL`. Nhưng Oracle lại trả về `'X'` — tức là nó đối xử với `NULL` như một chuỗi rỗng.

Khái niệm `NULL` được dùng trong rất nhiều ngôn ngữ lập trình. Dù bạn nhìn vào đâu, một chuỗi rỗng không bao giờ là `NULL`… ngoại trừ trong Oracle Database. Thực tế, trong Oracle **không thể** lưu một chuỗi rỗng vào một trường `VARCHAR2`. Nếu bạn cố làm vậy, Oracle Database sẽ lưu `NULL`.

Điểm kỳ quặc này không chỉ lạ lùng; nó còn nguy hiểm. Và sự kỳ quặc về `NULL` của Oracle Database không dừng lại ở đó — nó còn tiếp tục với cả việc đánh index.

> 📝 **Giải thích thêm (không có trong sách):** Hệ quả thực tế của việc "chuỗi rỗng = NULL" trong Oracle:
> - `WHERE ten_cot = ''` **không bao giờ** trả về dòng nào (vì so sánh với `NULL` cho kết quả UNKNOWN). Phải viết `WHERE ten_cot IS NULL`.
> - `LENGTH('')` trả về `NULL` chứ không phải `0`.
> - Khi chuyển ứng dụng từ PostgreSQL/SQL Server/MySQL (nơi `'' <> NULL`) sang Oracle hoặc ngược lại, các truy vấn kiểu `WHERE middle_name = ''` hay ràng buộc `NOT NULL` trên cột có thể chứa chuỗi rỗng sẽ hành xử khác hẳn. Ví dụ trong PostgreSQL, `INSERT` chuỗi rỗng vào cột `NOT NULL` là hợp lệ; trong Oracle thì bị lỗi `ORA-01400: cannot insert NULL`.

> 💡 **Ví dụ bổ sung (không có trong sách):** So sánh nhanh cùng một biểu thức trên các hệ quản trị:
>
> | Biểu thức | Oracle | PostgreSQL | SQL Server | MySQL |
> |---|---|---|---|---|
> | `'' IS NULL` | TRUE | FALSE | FALSE | FALSE |
> | `'X' \|\| NULL` | `'X'` | `NULL` | (dùng `'X' + NULL` → `NULL`; `CONCAT('X', NULL)` → `'X'`) | `\|\|` mặc định là OR logic; `CONCAT('X', NULL)` → `NULL` |
> | `LENGTH('')` | `NULL` | `0` | `LEN('')` = `0` | `0` |
>
> Bảng trên cho thấy hành vi của Oracle là ngoại lệ so với các DB khác.

### Đánh index cho NULL (Indexing NULL)

Oracle Database **không đưa một dòng vào index nếu tất cả các cột được index của dòng đó đều là `NULL`**. Điều đó có nghĩa là mọi index trong Oracle đều là một **partial index** (index một phần — index chỉ chứa một tập con các dòng của bảng), giống như có một mệnh đề `where`:

```sql
CREATE INDEX idx
          ON tbl (A, B, C, ...)
       WHERE A IS NOT NULL
          OR B IS NOT NULL
          OR C IS NOT NULL
             ...;
```

(Đây là cú pháp minh họa ý nghĩa; Oracle không hỗ trợ mệnh đề `WHERE` trong `CREATE INDEX`.)

Hãy xét index `EMP_DOB`. Nó chỉ có một cột: `DATE_OF_BIRTH`. Một dòng không có giá trị `DATE_OF_BIRTH` sẽ không được thêm vào index này.

```sql
INSERT INTO employees ( subsidiary_id, employee_id
                      , first_name   , last_name
                      , phone_number)
               VALUES ( ?, ?, ?, ?, ? );
```

Câu lệnh insert trên không gán giá trị cho `DATE_OF_BIRTH` nên nó nhận giá trị mặc định là `NULL` — do đó, bản ghi này không được thêm vào index `EMP_DOB`. Hệ quả là index không thể hỗ trợ một truy vấn tìm các bản ghi có `DATE_OF_BIRTH IS NULL`:

```sql
SELECT first_name, last_name
  FROM employees
 WHERE date_of_birth IS NULL;
```

```text
----------------------------------------------------
| Id | Operation         | Name      | Rows | Cost |
----------------------------------------------------
|  0 | SELECT STATEMENT  |           |    1 |  477 |
|* 1 |  TABLE ACCESS FULL| EMPLOYEES |    1 |  477 |
----------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------
   1 - filter("DATE_OF_BIRTH" IS NULL)
```

Execution plan (kế hoạch thực thi) cho thấy database phải đọc toàn bộ bảng (`TABLE ACCESS FULL`), vì index `EMP_DOB` không hề chứa những dòng có `DATE_OF_BIRTH` là `NULL`.

Tuy nhiên, bản ghi **vẫn được** đưa vào một index ghép (*concatenated index* — index trên nhiều cột) nếu ít nhất một cột của index không phải `NULL`:

```sql
CREATE INDEX demo_null
          ON employees (subsidiary_id, date_of_birth);
```

Dòng vừa tạo ở trên được thêm vào index này vì `SUBSIDIARY_ID` không phải `NULL`. Do đó, index này có thể hỗ trợ truy vấn tìm tất cả nhân viên của một chi nhánh (*subsidiary*) cụ thể mà không có giá trị `DATE_OF_BIRTH`:

```sql
SELECT first_name, last_name
  FROM employees
 WHERE subsidiary_id = ?
   AND date_of_birth IS NULL;
```

```text
--------------------------------------------------------------
| Id | Operation                   | Name      | Rows | Cost |
--------------------------------------------------------------
|  0 | SELECT STATEMENT            |           |    1 |    2 |
|  1 |  TABLE ACCESS BY INDEX ROWID| EMPLOYEES |    1 |    2 |
|* 2 |   INDEX RANGE SCAN          | DEMO_NULL |    1 |    1 |
--------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------
   2 - access("SUBSIDIARY_ID"=TO_NUMBER(?)
          AND "DATE_OF_BIRTH" IS NULL)
```

Hãy lưu ý rằng index bao phủ toàn bộ mệnh đề where; tất cả các bộ lọc đều được dùng làm **access predicate** (vị từ truy cập — điều kiện dùng để xác định điểm bắt đầu và kết thúc của việc duyệt index) trong quá trình `INDEX RANGE SCAN`.

Ta có thể mở rộng ý tưởng này cho truy vấn ban đầu — tìm tất cả bản ghi có `DATE_OF_BIRTH IS NULL`. Để làm vậy, cột `DATE_OF_BIRTH` phải là cột ngoài cùng bên trái (*leftmost column*) của index để có thể được dùng làm access predicate. Mặc dù bản thân truy vấn không cần cột index thứ hai, ta vẫn thêm một cột khác **không bao giờ có thể là `NULL`** để đảm bảo index chứa tất cả các dòng. Ta có thể dùng bất kỳ cột nào có ràng buộc `NOT NULL`, chẳng hạn `SUBSIDIARY_ID`, cho mục đích đó.

Một cách khác: ta có thể dùng một biểu thức hằng (*constant expression*) không bao giờ là `NULL`. Điều đó đảm bảo index chứa tất cả các dòng — kể cả khi `DATE_OF_BIRTH` là `NULL`.

```sql
DROP   INDEX emp_dob;
CREATE INDEX emp_dob ON employees (date_of_birth, '1');
```

Về mặt kỹ thuật, index này là một **function-based index** (index dựa trên hàm/biểu thức). Ví dụ này cũng bác bỏ lời đồn (*myth*) rằng Oracle Database không thể đánh index cho `NULL`.

> **Mẹo (Tip)**
>
> Hãy thêm một cột không thể là `NULL` vào index để đánh index cho `NULL` giống như mọi giá trị khác.

> 💡 **Ví dụ bổ sung (không có trong sách):** Giả sử bảng `orders` (Oracle) có 5 triệu dòng, trong đó chỉ khoảng 2.000 đơn hàng chưa được giao (`shipped_date IS NULL`):
>
> | order_id | customer_id | shipped_date |
> |---|---|---|
> | 1 | 10 | 2024-01-03 |
> | 2 | 11 | NULL |
> | 3 | 10 | 2024-01-05 |
> | … | … | … |
>
> ```sql
> -- Index một cột: KHÔNG chứa các dòng shipped_date IS NULL
> CREATE INDEX ord_shipped ON orders (shipped_date);
>
> -- Truy vấn này sẽ phải TABLE ACCESS FULL trên 5 triệu dòng:
> SELECT order_id FROM orders WHERE shipped_date IS NULL;
>
> -- Sửa: thêm hằng số để mọi dòng đều có trong index
> DROP INDEX ord_shipped;
> CREATE INDEX ord_shipped ON orders (shipped_date, 0);
> ```
>
> Sau khi sửa, Oracle có thể dùng `INDEX RANGE SCAN` với `access("SHIPPED_DATE" IS NULL)` và chỉ đọc khoảng 2.000 mục index. Lưu ý: index vẫn dùng được cho các truy vấn kiểu `shipped_date = ?` hay `shipped_date > ?` như trước.
>
> **Các DB khác:** PostgreSQL, SQL Server, MySQL (InnoDB) và DB2 đều **lưu `NULL` trong B-tree index**, nên `WHERE shipped_date IS NULL` có thể dùng index một cột bình thường; không cần mẹo này.

### Ràng buộc NOT NULL (NOT NULL Constraints)

Để đánh index cho một điều kiện `IS NULL` trong Oracle Database, index phải có một cột **không bao giờ có thể là `NULL`**.

Tuy vậy, chỉ việc "hiện tại không có mục `NULL` nào" là chưa đủ. Database phải **chắc chắn** rằng *không bao giờ có thể* có mục `NULL`; nếu không, database phải giả định rằng bảng có những dòng không nằm trong index.

Index sau đây chỉ hỗ trợ được truy vấn nếu cột `LAST_NAME` có ràng buộc `NOT NULL`:

```sql
DROP INDEX emp_dob;
CREATE INDEX emp_dob_name
          ON employees (date_of_birth, last_name);
```

```sql
SELECT *
  FROM employees
 WHERE date_of_birth IS NULL;
```

```text
---------------------------------------------------------------
|Id |Operation                   | Name         | Rows | Cost |
---------------------------------------------------------------
| 0 |SELECT STATEMENT            |              |    1 |    3 |
| 1 | TABLE ACCESS BY INDEX ROWID| EMPLOYEES    |    1 |    3 |
|*2 |  INDEX RANGE SCAN          | EMP_DOB_NAME |    1 |    2 |
---------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------
   2 - access("DATE_OF_BIRTH" IS NULL)
```

Gỡ bỏ ràng buộc `NOT NULL` sẽ khiến index không dùng được cho truy vấn này nữa:

```sql
ALTER TABLE employees MODIFY last_name NULL;
```

```sql
SELECT *
  FROM employees
 WHERE date_of_birth IS NULL;
```

```text
----------------------------------------------------
| Id | Operation         | Name      | Rows | Cost |
----------------------------------------------------
|  0 | SELECT STATEMENT  |           |    1 |  477 |
|* 1 |  TABLE ACCESS FULL| EMPLOYEES |    1 |  477 |
----------------------------------------------------
```

> **Mẹo (Tip)**
>
> Thiếu một ràng buộc `NOT NULL` có thể ngăn việc dùng index trong Oracle Database — đặc biệt là với các truy vấn `count(*)`.

> 📝 **Giải thích thêm (không có trong sách):** Vì sao lại "đặc biệt là `count(*)`"? Với `SELECT count(*) FROM employees`, Oracle có thể đếm số mục trong một index nhỏ (`INDEX FAST FULL SCAN`) thay vì đọc cả bảng — nhưng chỉ khi index đó **chắc chắn** chứa mọi dòng, tức là có ít nhất một cột được khai báo `NOT NULL` (ví dụ index của khóa chính). Nếu mọi index đều chỉ trên các cột cho phép `NULL`, Oracle buộc phải `TABLE ACCESS FULL` để đếm, dù thực tế dữ liệu không có `NULL` nào.

Ngoài ràng buộc `NOT NULL`, database cũng biết rằng các biểu thức hằng như ở mục trước không thể trở thành `NULL`.

Tuy nhiên, một index trên **hàm do người dùng định nghĩa** (*user-defined function*) thì không áp đặt ràng buộc `NOT NULL` lên biểu thức được index:

```sql
CREATE OR REPLACE FUNCTION blackbox(id IN NUMBER) RETURN NUMBER
DETERMINISTIC
IS BEGIN
   RETURN id;
END;
```

```sql
DROP INDEX emp_dob_name;
CREATE INDEX emp_dob_bb
    ON employees (date_of_birth, blackbox(employee_id));
```

```sql
SELECT *
  FROM employees
 WHERE date_of_birth IS NULL;
```

```text
----------------------------------------------------
| Id | Operation         | Name      | Rows | Cost |
----------------------------------------------------
|  0 | SELECT STATEMENT  |           |    1 |  477 |
|* 1 |  TABLE ACCESS FULL| EMPLOYEES |    1 |  477 |
----------------------------------------------------
```

Tên hàm `BLACKBOX` ("hộp đen") nhấn mạnh rằng optimizer (bộ tối ưu hóa truy vấn) hoàn toàn không biết hàm này làm gì. *Chúng ta* nhìn vào thì thấy hàm chỉ trả lại nguyên giá trị đầu vào, nhưng đối với database, nó chỉ là một hàm trả về một số. Thuộc tính `NOT NULL` của tham số đã bị mất. Mặc dù index chắc chắn phải chứa tất cả các dòng (vì `EMPLOYEE_ID` là khóa chính, không bao giờ `NULL`), database không biết điều đó nên không thể dùng index cho truy vấn.

Nếu bạn biết rằng hàm không bao giờ trả về `NULL`, như trong ví dụ này, bạn có thể sửa truy vấn để thể hiện điều đó:

```sql
SELECT *
  FROM employees
 WHERE date_of_birth IS NULL
   AND blackbox(employee_id) IS NOT NULL;
```

```text
-------------------------------------------------------------
|Id |Operation                   | Name       | Rows | Cost |
-------------------------------------------------------------
| 0 |SELECT STATEMENT            |            |    1 |    3 |
| 1 | TABLE ACCESS BY INDEX ROWID| EMPLOYEES  |    1 |    3 |
|*2 |  INDEX RANGE SCAN          | EMP_DOB_BB |    1 |    2 |
-------------------------------------------------------------
```

Điều kiện bổ sung trong mệnh đề where luôn đúng và vì vậy không làm thay đổi kết quả. Tuy nhiên, Oracle Database nhận ra rằng bạn chỉ truy vấn những dòng mà theo định nghĩa chắc chắn phải nằm trong index (vì biểu thức thứ hai của index không `NULL`, nên dòng chắc chắn có trong index).

Đáng tiếc là không có cách nào để "đánh dấu" một hàm là không bao giờ trả về `NULL`, nhưng bạn có thể chuyển lời gọi hàm vào một **cột ảo** (*virtual column*, có từ Oracle 11g) và đặt ràng buộc `NOT NULL` lên cột này.

```sql
ALTER TABLE employees ADD bb_expression
      GENERATED ALWAYS AS (blackbox(employee_id)) NOT NULL;
DROP   INDEX emp_dob_bb;
CREATE INDEX emp_dob_bb
    ON employees (date_of_birth, bb_expression);
```

```sql
SELECT *
  FROM employees
 WHERE date_of_birth IS NULL;
```

```text
-------------------------------------------------------------
|Id |Operation                   | Name       | Rows | Cost |
-------------------------------------------------------------
| 0 |SELECT STATEMENT            |            |    1 |    3 |
| 1 | TABLE ACCESS BY INDEX ROWID| EMPLOYEES  |    1 |    3 |
|*2 |  INDEX RANGE SCAN          | EMP_DOB_BB |    1 |    2 |
-------------------------------------------------------------
```

Oracle Database biết rằng một số hàm có sẵn (*internal functions*) chỉ trả về `NULL` nếu đầu vào là `NULL`.

```sql
DROP INDEX emp_dob_bb;
CREATE INDEX emp_dob_upname
    ON employees (date_of_birth, upper(last_name));
```

```sql
SELECT *
  FROM employees
 WHERE date_of_birth IS NULL;
```

```text
----------------------------------------------------------
|Id |Operation                   | Name           | Cost |
----------------------------------------------------------
| 0 |SELECT STATEMENT            |                |    3 |
| 1 | TABLE ACCESS BY INDEX ROWID| EMPLOYEES      |    3 |
|*2 |  INDEX RANGE SCAN          | EMP_DOB_UPNAME |    2 |
----------------------------------------------------------
```

Hàm `UPPER` giữ nguyên thuộc tính `NOT NULL` của cột `LAST_NAME`. Tuy nhiên, gỡ bỏ ràng buộc thì index lại không dùng được:

```sql
ALTER TABLE employees MODIFY last_name NULL;
```

```sql
SELECT *
  FROM employees
 WHERE date_of_birth IS NULL;
```

```text
----------------------------------------------------
| Id | Operation         | Name      | Rows | Cost |
----------------------------------------------------
|  0 | SELECT STATEMENT  |           |    1 |  477 |
|* 1 |  TABLE ACCESS FULL| EMPLOYEES |    1 |  477 |
----------------------------------------------------
```

> 💡 **Ví dụ bổ sung (không có trong sách):** Tóm tắt các trường hợp đã gặp, cho truy vấn `WHERE date_of_birth IS NULL` trên Oracle:
>
> | Index | Cột/biểu thức thứ hai có "chắc chắn NOT NULL"? | Dùng index được? |
> |---|---|---|
> | `(date_of_birth)` | Không có cột thứ hai | ❌ `TABLE ACCESS FULL` |
> | `(date_of_birth, '1')` | Có — hằng số | ✅ |
> | `(date_of_birth, last_name)` với `last_name NOT NULL` | Có — ràng buộc | ✅ |
> | `(date_of_birth, last_name)` với `last_name` cho phép NULL | Không | ❌ |
> | `(date_of_birth, blackbox(employee_id))` | Không — hàm tự viết là hộp đen | ❌ (✅ nếu thêm `AND blackbox(employee_id) IS NOT NULL`) |
> | `(date_of_birth, bb_expression)` — cột ảo `NOT NULL` | Có — ràng buộc trên cột ảo | ✅ |
> | `(date_of_birth, upper(last_name))` với `last_name NOT NULL` | Có — `UPPER` giữ tính NOT NULL | ✅ |
> | `(date_of_birth, upper(last_name))` với `last_name` cho phép NULL | Không | ❌ |
>
> Quy tắc chung: optimizer chỉ dùng index cho `IS NULL` khi nó **chứng minh được** rằng index chứa mọi dòng thỏa điều kiện.

### Giả lập partial index (Emulating Partial Indexes)

Cách xử lý `NULL` kỳ lạ trong index của Oracle Database có thể được tận dụng để giả lập **partial index**. Để làm vậy, ta chỉ cần dùng `NULL` cho những dòng không cần được index.

Để minh họa, ta giả lập partial index sau (cú pháp này có trong PostgreSQL, Oracle không hỗ trợ):

```sql
CREATE INDEX messages_todo
          ON messages (receiver)
       WHERE processed = 'N'
```

Đầu tiên, ta cần một hàm chỉ trả về giá trị `RECEIVER` nếu giá trị `PROCESSED` là `'N'`.

```sql
CREATE OR REPLACE
FUNCTION pi_processed(processed CHAR, receiver NUMBER)
RETURN NUMBER
DETERMINISTIC
AS BEGIN
   IF processed IN ('N') THEN
      RETURN receiver;
   ELSE
      RETURN NULL;
   END IF;
END;
/
```

Hàm phải là **deterministic** (tất định — cùng đầu vào luôn cho cùng kết quả) thì mới dùng được trong định nghĩa index.

Giờ ta có thể tạo một index chỉ chứa các dòng có `PROCESSED='N'`.

```sql
CREATE INDEX messages_todo
          ON messages (pi_processed(processed, receiver));
```

Với các dòng đã xử lý, hàm trả về `NULL` — và vì đó là cột duy nhất của index, Oracle không đưa những dòng đó vào index.

Để dùng được index, bạn phải dùng **đúng biểu thức đã được index** trong truy vấn:

```sql
SELECT message
  FROM messages
 WHERE pi_processed(processed, receiver) = ?
```

```text
----------------------------------------------------------
|Id | Operation                   | Name          | Cost |
----------------------------------------------------------
| 0 | SELECT STATEMENT            |               | 5330 |
| 1 |  TABLE ACCESS BY INDEX ROWID| MESSAGES      | 5330 |
|*2 |   INDEX RANGE SCAN          | MESSAGES_TODO | 5303 |
----------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------
   2 - access("PI_PROCESSED"("PROCESSED","RECEIVER")=:X)
```

> **Partial index, phần II (Partial Indexes, Part II)**
>
> Kể từ phiên bản 11g, có một cách thứ hai — cũng "đáng sợ" không kém — để giả lập partial index trong Oracle Database: dùng một **index partition bị cố tình làm hỏng** (*intentionally broken index partition*, tức partition được đặt ở trạng thái `UNUSABLE`) kết hợp với tham số `SKIP_UNUSABLE_INDEX`.

> 💡 **Ví dụ bổ sung (không có trong sách):** Một cách viết gọn hơn mà không cần hàm PL/SQL riêng (Oracle):
>
> ```sql
> CREATE INDEX messages_todo
>     ON messages (CASE WHEN processed = 'N' THEN receiver END);
>
> SELECT message
>   FROM messages
>  WHERE CASE WHEN processed = 'N' THEN receiver END = :receiver;
> ```
>
> Biểu thức `CASE` trả về `NULL` khi `processed <> 'N'`, nên các dòng đó không vào index. Giả sử bảng có 10 triệu tin nhắn nhưng chỉ 5.000 tin chưa xử lý: index chỉ có khoảng 5.000 mục — nhỏ, nhanh, và rẻ khi `INSERT`/`UPDATE`. Điều kiện trong truy vấn phải viết **giống hệt** biểu thức trong index.
>
> **Các DB khác:** PostgreSQL hỗ trợ partial index trực tiếp (`CREATE INDEX ... WHERE processed = 'N'`); SQL Server (2008+) có *filtered index* (`CREATE INDEX ... WHERE processed = 'N'`). Oracle 12c+ có *partial indexing* cho bảng phân vùng (`INDEXING OFF` ở mức partition), nhưng đó là theo partition chứ không theo điều kiện tùy ý. MySQL không có partial index.

> 📝 **Giải thích thêm (không có trong sách) — Đối chiếu với PostgreSQL hiện đại (PG 10–17): Cách PostgreSQL lưu `NULL` trong B-Tree Index, Nén trùng lặp `NULL` (PG 13+) và `NULLS NOT DISTINCT` (PG 15+):**
>
> Toàn bộ phần đầu Chương 2c xoáy sâu vào hành vi "đặc dị" của Oracle (coi chuỗi rỗng `''` là `NULL` và không lưu các dòng toàn `NULL` vào B-Tree index). Khi làm việc trên **PostgreSQL**, kiến trúc sư cơ sở dữ liệu cần nắm vững 4 khác biệt cốt lõi sau:
>
> 1. **PostgreSQL phân biệt rạch ròi `''` (chuỗi rỗng) và `NULL`:** Tuân thủ nghiêm ngặt chuẩn SQL, trong PostgreSQL `'' IS NULL` luôn là `FALSE` và `'X' || NULL` luôn là `NULL` (nếu muốn nối bỏ qua `NULL`, hãy dùng hàm `concat('X', NULL)` hoặc `concat_ws()`).
> 2. **PostgreSQL LUÔN lưu giá trị `NULL` vào B-Tree Index:** Khác hoàn toàn với Oracle, một B-Tree index đơn cột `CREATE INDEX emp_dob ON employees (date_of_birth)` trong PostgreSQL chứa **tất cả các dòng của bảng**, kể cả các dòng có `date_of_birth IS NULL`. Do đó:
>    - Truy vấn `WHERE date_of_birth IS NULL` dùng trực tiếp **`Index Scan`** hoặc **`Index Only Scan`** trên index đơn cột cực nhanh (chỉ đọc đúng các mục `NULL`), **tuyệt đối không cần** mẹo thêm cột hằng số `'1'` hay cột `NOT NULL` giả tạo như trên Oracle!
>    - Mặc định trong B-Tree của PostgreSQL, các mục `NULL` được xếp ở **cuối cùng** khi sắp xếp tăng dần (`ASC NULLS LAST`) và ở **đầu tiên** khi sắp xếp giảm dần (`DESC NULLS FIRST`). Bạn có thể tùy chỉnh bằng `CREATE INDEX ... ON tbl (col ASC NULLS FIRST)` để khớp với truy vấn `ORDER BY col ASC NULLS FIRST`.
>    - Nếu bạn **không muốn** lưu các dòng `NULL` vào index để tiết kiệm dung lượng (giống hành vi ngầm định của Oracle), trong PostgreSQL bạn chỉ cần viết tường minh một Partial Index: `CREATE INDEX ... ON tbl (col) WHERE col IS NOT NULL;`.
> 3. **Nén trùng lặp B-Tree (*B-Tree Deduplication* — ra mắt từ PostgreSQL 13+):** Trước PostgreSQL 13, nếu một cột có hàng triệu dòng `NULL` và bạn tạo B-Tree index trên cột đó, mỗi dòng `NULL` tốn một index tuple riêng (16 bytes). Từ **PostgreSQL 13+**, cơ chế *B-Tree Deduplication* tự động gộp hàng nghìn `TID` có cùng giá trị `NULL` vào một *posting list* duy nhất, giảm kích thước index chứa nhiều `NULL` tới **70%–80%**!
> 4. **Đột phá `NULLS NOT DISTINCT` (ra mắt từ PostgreSQL 15+):**
>    - Theo chuẩn SQL truyền thống (và PostgreSQL 9.x → 14), trong ràng buộc `UNIQUE (email, deleted_at)`, hai giá trị `NULL` được coi là khác nhau (`NULLS DISTINCT`), dẫn đến việc người dùng có thể chèn hàng chục dòng trùng `(email = 'a@b.com', deleted_at = NULL)`. Trước PG 15, bạn buộc phải lách bằng cách tạo 2 Partial Unique Index rời rạc.
>    - Từ **PostgreSQL 15+**, PostgreSQL hỗ trợ chuẩn SQL:2023 với cú pháp **`UNIQUE NULLS NOT DISTINCT (col1, col2)`** (hoặc `CREATE UNIQUE INDEX ... ON tbl (col1, col2) NULLS NOT DISTINCT`). Khi bật tùy chọn này, PostgreSQL coi hai giá trị `NULL` là **bằng nhau** khi kiểm tra tính duy nhất — chỉ cho phép tối đa một dòng `NULL`!

> 💡 **Ví dụ bổ sung (không có trong sách) — Thực hành trên PostgreSQL: Truy vấn `IS NULL` bằng `Index Scan` và sử dụng `NULLS NOT DISTINCT` (PostgreSQL 15+):**
>
> ```sql
> -- Tạo bảng người dùng với ràng buộc UNIQUE NULLS NOT DISTINCT (PostgreSQL 15+)
> -- Đảm bảo mỗi email chỉ có tối đa 1 tài khoản chưa bị xóa (deleted_at IS NULL)
> CREATE TABLE app_users (
>     user_id    BIGSERIAL PRIMARY KEY,
>     email      TEXT NOT NULL,
>     phone      TEXT,
>     deleted_at TIMESTAMPTZ,
>     -- Cú pháp mới từ PostgreSQL 15+: Coi (email, NULL) trùng với (email, NULL)
>     CONSTRAINT uq_app_users_email_active
>         UNIQUE NULLS NOT DISTINCT (email, deleted_at)
> );
>
> -- Tạo B-Tree index đơn cột bình thường trên cột phone (cho phép NULL)
> CREATE INDEX idx_app_users_phone ON app_users (phone);
>
> -- Nạp 200.000 người dùng: 199.950 người đã có số điện thoại, chỉ 50 người có phone IS NULL
> INSERT INTO app_users (email, phone, deleted_at)
> SELECT 'user' || g || '@example.com',
>        CASE WHEN g <= 50 THEN NULL ELSE '090' || lpad(g::text, 7, '0') END,
>        NULL
>   FROM generate_series(1, 200000) AS g;
>
> VACUUM ANALYZE app_users;
>
> -- 1. Kiểm chứng Index Scan trực tiếp cho điều kiện IS NULL (không cần thêm cột hằng số như Oracle!)
> EXPLAIN (ANALYZE, BUFFERS)
> SELECT user_id, email FROM app_users WHERE phone IS NULL;
> ```
>
> Kế hoạch thực thi thực tế trên PostgreSQL:
>
> ```text
>                                                            QUERY PLAN
> ---------------------------------------------------------------------------------------------------------------------------------
>  Index Scan using idx_app_users_phone on app_users  (cost=0.42..4.94 rows=50 width=29) (actual time=0.018..0.032 rows=50 loops=1)
>    Index Cond: (phone IS NULL)
>    Buffers: shared hit=4
>  Planning Time: 0.088 ms
>  Execution Time: 0.049 ms
> ```
>
> ```sql
> -- 2. Kiểm chứng ràng buộc UNIQUE NULLS NOT DISTINCT (PostgreSQL 15+):
> -- Thử chèn thêm một tài khoản đang hoạt động (deleted_at = NULL) trùng email 'user1@example.com':
> INSERT INTO app_users (email, phone, deleted_at)
> VALUES ('user1@example.com', '0988888888', NULL);
> -- Lập tức bị chặn bởi lỗi:
> -- ERROR:  duplicate key value violates unique constraint "uq_app_users_email_active"
> -- DETAIL:  Key (email, deleted_at)=(user1@example.com, null) already exists.
> ```

---

## Các điều kiện bị che khuất (Obfuscated Conditions)

Các phần sau đây trình bày một số cách phổ biến làm "che khuất" (*obfuscate*) điều kiện. **Điều kiện bị che khuất** (*obfuscated conditions*) là những mệnh đề where được diễn đạt theo cách ngăn cản việc dùng index đúng cách. Mục này là một bộ sưu tập các **anti-pattern** (mẫu sai lầm điển hình) mà mọi lập trình viên nên biết và tránh.

### Kiểu ngày tháng (Date Types)

Phần lớn các kiểu che khuất liên quan đến kiểu `DATE`. Oracle Database đặc biệt dễ mắc lỗi ở khía cạnh này vì nó chỉ có một kiểu `DATE`, và kiểu này **luôn bao gồm cả thành phần thời gian** (giờ, phút, giây).

Việc dùng hàm `TRUNC` để loại bỏ thành phần thời gian đã trở thành thói quen phổ biến. Thực ra, nó không *loại bỏ* thời gian mà đặt thời gian về nửa đêm (00:00:00), vì Oracle Database không có kiểu ngày thuần túy (*pure DATE type*). Để bỏ qua thành phần thời gian khi tìm kiếm, bạn có thể dùng hàm `TRUNC` ở cả hai vế của phép so sánh — ví dụ để tìm doanh số bán hàng của ngày hôm qua:

```sql
SELECT ...
  FROM sales
 WHERE TRUNC(sale_date) = TRUNC(sysdate - INTERVAL '1' DAY)
```

Đây là một câu lệnh hoàn toàn hợp lệ và đúng, nhưng nó không thể tận dụng đúng cách một index trên `SALE_DATE`. Lý do giống như đã giải thích trong "Tìm kiếm không phân biệt hoa thường bằng UPPER hoặc LOWER" (*Case-Insensitive Search Using UPPER or LOWER*) ở trang 24: `TRUNC(sale_date)` là một thứ hoàn toàn khác với `SALE_DATE` — đối với database, hàm là những hộp đen.

Có một giải pháp khá đơn giản cho vấn đề này: một function-based index.

```sql
CREATE INDEX index_name
          ON table_name (TRUNC(sale_date))
```

Nhưng khi đó bạn phải **luôn luôn** dùng `TRUNC(date_column)` trong mệnh đề where. Nếu bạn dùng không nhất quán — lúc có, lúc không có `TRUNC` — thì bạn sẽ cần tới hai index!

Vấn đề này cũng xảy ra với các database có kiểu ngày thuần túy nếu bạn tìm kiếm theo một khoảng thời gian dài hơn, như trong truy vấn MySQL sau:

```sql
SELECT ...
  FROM sales
 WHERE DATE_FORMAT(sale_date, "%Y-%M")
     = DATE_FORMAT(now()    , "%Y-%M')
```

Truy vấn dùng một định dạng ngày chỉ gồm năm và tháng: một lần nữa, đây là một truy vấn hoàn toàn đúng nhưng mắc cùng vấn đề như trên. Tuy nhiên, giải pháp ở trên không áp dụng được ở đây vì MySQL không có function-based index.

> 📝 **Giải thích thêm (không có trong sách):** Nhận định "MySQL không có function-based index" đúng ở thời điểm sách viết. Từ **MySQL 8.0.13**, MySQL hỗ trợ *functional key parts*, ví dụ `CREATE INDEX idx ON sales ((DATE_FORMAT(sale_date, '%Y-%m')));`. Ngoài ra MySQL 5.7+ có thể index một *generated column*. Dù vậy, lời khuyên dùng điều kiện khoảng (range) tường minh bên dưới vẫn là giải pháp tốt nhất và tổng quát nhất. (Ghi chú nhỏ: trong câu lệnh gốc của sách, chuỗi `"%Y-%M')` có dấu nháy mở và đóng không khớp — đây là lỗi đánh máy trong sách, được chép nguyên văn.)

Giải pháp thay thế là dùng một **điều kiện khoảng tường minh** (*explicit range condition*). Đây là một giải pháp tổng quát, chạy được trên mọi database:

```sql
SELECT ...
  FROM sales
 WHERE sale_date BETWEEN quarter_begin(?)
                     AND quarter_end(?)
```

Nếu bạn đã làm bài tập về nhà, có lẽ bạn sẽ nhận ra mẫu này từ bài tập tìm tất cả nhân viên 42 tuổi.

Một index thông thường (*straight index*) trên `SALE_DATE` là đủ để tối ưu truy vấn này. Các hàm `QUARTER_BEGIN` và `QUARTER_END` tính các ngày biên (ngày đầu và cuối quý). Phép tính có thể hơi phức tạp một chút vì toán tử between luôn **bao gồm cả giá trị biên**. Vì vậy, hàm `QUARTER_END` phải trả về một mốc thời gian (*time stamp*) ngay trước ngày đầu tiên của quý tiếp theo nếu `SALE_DATE` có thành phần thời gian. Logic này có thể được giấu bên trong hàm.

Các ví dụ ở những trang sau trình bày cách cài đặt các hàm `QUARTER_BEGIN` và `QUARTER_END` cho nhiều database khác nhau.

#### MySQL

```sql
CREATE FUNCTION quarter_begin(dt DATETIME)
RETURNS DATETIME DETERMINISTIC
RETURN CONVERT
       (
         CONCAT
         ( CONVERT(YEAR(dt),CHAR(4))
         , '-'
         , CONVERT(QUARTER(dt)*3-2,CHAR(2))
         , '-01'
         )
       , datetime
       );

CREATE FUNCTION quarter_end(dt DATETIME)
RETURNS DATETIME DETERMINISTIC
RETURN DATE_ADD
       ( DATE_ADD ( quarter_begin(dt), INTERVAL 3 MONTH )
       , INTERVAL -1 MICROSECOND);
```

#### Oracle Database

```sql
CREATE FUNCTION quarter_begin(dt IN DATE)
RETURN DATE
AS
BEGIN
   RETURN TRUNC(dt, 'Q');
END;
/

CREATE FUNCTION quarter_end(dt IN DATE)
RETURN DATE
AS
BEGIN
   -- the Oracle DATE type has seconds resolution
   -- subtract one second from the first
   -- day of the following quarter
   RETURN TRUNC(ADD_MONTHS(dt, +3), 'Q')
        - (1/(24*60*60));
END;
/
```

(Chú thích trong code: *kiểu DATE của Oracle có độ phân giải đến giây — trừ đi một giây từ ngày đầu tiên của quý tiếp theo.*)

#### PostgreSQL

```sql
CREATE FUNCTION quarter_begin(dt timestamp with time zone)
RETURNS timestamp with time zone AS $$
BEGIN
    RETURN date_trunc('quarter', dt);
END;
$$ LANGUAGE plpgsql;

CREATE FUNCTION quarter_end(dt timestamp with time zone)
RETURNS timestamp with time zone AS $$
BEGIN
   RETURN   date_trunc('quarter', dt)
          + interval '3 month'
          - interval '1 microsecond';
END;
$$ LANGUAGE plpgsql;
```

#### SQL Server

```sql
CREATE FUNCTION quarter_begin (@dt DATETIME )
RETURNS DATETIME
BEGIN
  RETURN DATEADD (qq, DATEDIFF (qq, 0, @dt), 0)
END
GO

CREATE FUNCTION quarter_end (@dt DATETIME )
RETURNS DATETIME
BEGIN
  RETURN DATEADD
         ( ms
         , -3
         , DATEADD(mm, 3, dbo.quarter_begin(@dt))
         );
END
GO
```

(Ở SQL Server, kiểu `DATETIME` có độ chính xác khoảng 3,33 mili giây, nên hàm trừ đi 3 ms để có mốc thời gian cuối cùng biểu diễn được trước quý sau.)

Bạn có thể dùng các hàm phụ trợ tương tự cho các khoảng thời gian khác — hầu hết sẽ ít phức tạp hơn các ví dụ trên, đặc biệt khi dùng điều kiện lớn hơn hoặc bằng (`>=`) và nhỏ hơn (`<`) thay cho toán tử between. Tất nhiên, bạn cũng có thể tính các ngày biên trong ứng dụng nếu muốn.

> **Mẹo (Tip)**
>
> Hãy viết các truy vấn cho các khoảng thời gian liên tục dưới dạng điều kiện khoảng tường minh. Hãy làm vậy ngay cả với một ngày duy nhất — ví dụ, với Oracle Database:
>
> ```sql
>     sale_date >= TRUNC(sysdate)
> AND sale_date <  TRUNC(sysdate + INTERVAL '1' DAY)
> ```

Một kiểu che khuất phổ biến khác là so sánh ngày dưới dạng chuỗi, như trong ví dụ PostgreSQL sau:

```sql
SELECT ...
  FROM sales
 WHERE TO_CHAR(sale_Date, 'YYYY-MM-DD') = '1970-01-01'
```

Vấn đề, một lần nữa, là việc chuyển đổi cột ngày (`DATE_COLUMN`, ở đây là `sale_date`). Những điều kiện như vậy thường được viết ra vì niềm tin rằng bạn không thể truyền cho database các kiểu dữ liệu khác ngoài số và chuỗi. Tuy nhiên, **bind parameter** (tham số ràng buộc — giá trị được truyền tách riêng khỏi câu SQL qua dấu `?` hoặc `:ten`) hỗ trợ mọi kiểu dữ liệu. Nghĩa là, ví dụ, bạn có thể dùng một đối tượng `java.util.Date` làm bind parameter. Đây lại là một lợi ích nữa của bind parameter.

Nếu bạn không làm được như vậy, bạn chỉ cần chuyển đổi **giá trị tìm kiếm** thay vì cột của bảng:

```sql
SELECT ...
  FROM sales
 WHERE sale_date = TO_DATE('1970-01-01', 'YYYY-MM-DD')
```

Truy vấn này có thể dùng một index thông thường trên `SALE_DATE`. Hơn nữa, nó chỉ chuyển đổi chuỗi đầu vào **một lần**. Câu lệnh trước đó phải chuyển đổi tất cả các ngày được lưu trong bảng trước khi có thể so sánh chúng với giá trị tìm kiếm.

Dù bạn thay đổi theo cách nào — dùng bind parameter hay chuyển đổi vế bên kia của phép so sánh — bạn đều dễ dàng gây ra lỗi nếu `SALE_DATE` có thành phần thời gian. Trong trường hợp đó, bạn phải dùng một điều kiện khoảng tường minh:

```sql
SELECT ...
  FROM sales
 WHERE sale_date >= TO_DATE('1970-01-01', 'YYYY-MM-DD')
   AND sale_date <  TO_DATE('1970-01-01', 'YYYY-MM-DD')
                  + INTERVAL '1' DAY
```

Hãy luôn cân nhắc dùng điều kiện khoảng tường minh khi so sánh ngày.

> **LIKE trên kiểu ngày (LIKE on Date Types)**
>
> Kiểu che khuất sau đây đặc biệt khó nhận ra:
>
> ```sql
> sale_date LIKE SYSDATE
> ```
>
> Thoạt nhìn, nó không giống một sự che khuất vì không dùng hàm nào cả.
>
> Tuy nhiên, toán tử `LIKE` bắt buộc phải so sánh chuỗi. Tùy database, điều đó có thể gây lỗi hoặc gây ra một **chuyển đổi kiểu ngầm định** (*implicit type conversion*) ở cả hai vế. Phần "Predicate Information" của execution plan cho thấy Oracle Database làm gì:
>
> ```text
> filter( INTERNAL_FUNCTION(SALE_DATE)
>    LIKE TO_CHAR(SYSDATE@!))
> ```
>
> Hàm `INTERNAL_FUNCTION` chuyển đổi kiểu của cột `SALE_DATE`. Như một tác dụng phụ, nó cũng ngăn việc dùng một index thông thường trên `DATE_COLUMN` (tức cột ngày, ở đây là `SALE_DATE`) — giống hệt như mọi hàm khác.

> 💡 **Ví dụ bổ sung (không có trong sách):** Bảng `sales` (Oracle) có index `sales_dt ON sales (sale_date)`:
>
> | sale_id | sale_date | amount |
> |---|---|---|
> | 1 | 2024-03-31 23:59:58 | 100 |
> | 2 | 2024-04-01 00:00:00 | 250 |
> | 3 | 2024-04-15 14:30:00 | 80 |
> | 4 | 2024-06-30 23:59:59 | 120 |
> | 5 | 2024-07-01 00:00:00 | 60 |
>
> Bài toán: tổng doanh số quý 2/2024.
>
> ```sql
> -- ❌ Che khuất: hàm trên cột → không dùng được index sales_dt
> SELECT SUM(amount) FROM sales
>  WHERE TO_CHAR(sale_date, 'YYYY-Q') = '2024-2';
>
> -- ✅ Điều kiện khoảng tường minh dạng >= và < (không cần trừ 1 giây)
> SELECT SUM(amount) FROM sales
>  WHERE sale_date >= DATE '2024-04-01'
>    AND sale_date <  DATE '2024-07-01';
> ```
>
> Câu thứ hai dùng `INDEX RANGE SCAN` trên `sales_dt` với `access("SALE_DATE">=... AND "SALE_DATE"<...)` và trả về đúng các dòng 2, 3, 4 (tổng 450). Dòng 1 (`23:59:58` ngày 31/3) và dòng 5 (`00:00:00` ngày 1/7) bị loại đúng như mong muốn. Cách `>=`/`<` (khoảng nửa mở) không phụ thuộc vào độ chính xác của kiểu thời gian — bạn không cần biết phải trừ 1 giây (Oracle `DATE`), 1 micro giây (PostgreSQL, Oracle `TIMESTAMP(6)`) hay 3 mili giây (SQL Server `DATETIME`).
>
> Tương tự, trong PostgreSQL thay vì `WHERE sale_date::date = '2024-04-15'` (ép kiểu cột) hãy viết `WHERE sale_date >= '2024-04-15' AND sale_date < '2024-04-16'`.

> 📝 **Giải thích thêm (không có trong sách) — Đối chiếu với PostgreSQL hiện đại (PG 10–17): Phân vùng bảng tự động (*Declarative Partitioning*) và Cơ chế cắt tỉa phân vùng (*Partition Pruning*) cho dữ liệu ngày tháng:**
>
> Khi bàn về lọc theo ngày tháng/quý/năm trên các bảng dữ liệu lớn như `sales`, việc viết **điều kiện khoảng tường minh** (`WHERE sale_date >= ... AND sale_date < ...`) trong PostgreSQL hiện đại không chỉ quyết định việc có dùng được B-Tree index hay không, mà còn quyết định sống còn đối với **Phân vùng bảng (*Table Partitioning*)**:
>
> - **Thời sách viết (PostgreSQL 9.1/9.2 — năm 2012):** PostgreSQL chưa có cú pháp phân vùng khai báo. Để chia bảng `sales` theo tháng/quý, lập trình viên phải tự tạo các bảng con kế thừa (`CREATE TABLE sales_2012_q1 () INHERITS (sales)`), tự viết hàm trigger PL/pgSQL gắn vào sự kiện `BEFORE INSERT` để định tuyến từng dòng về bảng con, và bật `constraint_exclusion = partition`. Cơ chế cũ này rất chậm khi có hàng trăm phân vùng và hoàn toàn **không thể cắt tỉa phân vùng lúc chạy (*runtime pruning*)** nếu điều kiện ngày tháng dùng tham số động hoặc biểu thức như `now()`.
> - **Cuộc cách mạng *Declarative Partitioning* trong PostgreSQL 10 → 17:**
>   1. **Cú pháp phân vùng khai báo gốc (ra mắt từ PostgreSQL 10+):** Hỗ trợ `PARTITION BY RANGE (sale_date)` và `PARTITION BY LIST (...)` ngay trong lệnh `CREATE TABLE`, tự động định tuyến dòng khi `INSERT` ở tầng C engine mà không cần trigger.
>   2. **Cắt tỉa phân vùng tĩnh & động (*Static & Run-time Partition Pruning* — PostgreSQL 11+):** Tham số `enable_partition_pruning = on` (mặc định từ PG 11) thay thế cơ chế `constraint_exclusion` cũ kỹ. PostgreSQL hiện đại có thể loại bỏ các phân vùng không liên quan ở **3 giai đoạn**:
>      - *Giai đoạn lập kế hoạch (Planning time):* Khi so sánh với hằng số (`WHERE sale_date >= '2024-04-01'`).
>      - *Giai đoạn khởi tạo thực thi (Executor initialization):* Khi so sánh với hàm ổn định (`now()`, `CURRENT_DATE`) hoặc tham số Prepared Statement (`$1, $2` trong Generic Plan)!
>      - *Giai đoạn đang chạy (Execution time):* Khi giá trị ngày tháng đến từ một nhánh con của `Nested Loop` hoặc `Merge Append`.
>   3. **Hệ sinh thái hoàn chỉnh (PG 11 → 17):** Hỗ trợ `PARTITION BY HASH` và phân vùng `DEFAULT` (PG 11), tự động lan truyền `CREATE INDEX` và `FOREIGN KEY` từ bảng mẹ xuống mọi phân vùng con (PG 11–12), tự động di chuyển dòng sang phân vùng khác khi `UPDATE sale_date` (PG 11), hỗ trợ `DETACH PARTITION ... CONCURRENTLY` không khóa truy vấn đọc (PG 14), tăng tốc lập kế hoạch trên hàng nghìn phân vùng (PG 12–17), và kết hợp hoàn hảo với extension **`pg_partman`** để tự động tạo phân vùng mới / gỡ phân vùng lịch sử.
>   4. **Cảnh báo về "Điều kiện bị che khuất" trên bảng phân vùng:** Nếu bạn viết `WHERE date_trunc('month', sale_date) = '2024-04-01'` hoặc `WHERE sale_date::date = '2024-04-15'` trên bảng được phân vùng theo `PARTITION BY RANGE (sale_date)`, PostgreSQL **sẽ mất hoàn toàn khả năng Partition Pruning** và buộc phải quét qua **tất cả các phân vùng của mọi năm**! Ngược lại, dùng điều kiện khoảng tường minh `sale_date >= ... AND sale_date < ...` giúp PostgreSQL vừa cắt bỏ 99% phân vùng lịch sử, vừa dùng `Index Scan` trên đúng 1 phân vùng duy nhất.

> 💡 **Ví dụ bổ sung (không có trong sách) — Thực hành trên PostgreSQL: So sánh điều kiện bị che khuất vs. Điều kiện khoảng tường minh trên bảng `PARTITION BY RANGE`:**
>
> ```sql
> -- Tạo bảng sales phân vùng theo quý bằng Declarative Partitioning (PostgreSQL 10+)
> CREATE TABLE sales_partitioned (
>     sale_id     BIGINT GENERATED ALWAYS AS IDENTITY,
>     sale_date   TIMESTAMPTZ NOT NULL,
>     customer_id INT NOT NULL,
>     amount      NUMERIC(10,2) NOT NULL,
>     PRIMARY KEY (sale_id, sale_date)
> ) PARTITION BY RANGE (sale_date);
>
> -- Tạo các phân vùng cho 4 quý năm 2024
> CREATE TABLE sales_2024_q1 PARTITION OF sales_partitioned
>     FOR VALUES FROM ('2024-01-01 00:00:00+00') TO ('2024-04-01 00:00:00+00');
> CREATE TABLE sales_2024_q2 PARTITION OF sales_partitioned
>     FOR VALUES FROM ('2024-04-01 00:00:00+00') TO ('2024-07-01 00:00:00+00');
> CREATE TABLE sales_2024_q3 PARTITION OF sales_partitioned
>     FOR VALUES FROM ('2024-07-01 00:00:00+00') TO ('2024-10-01 00:00:00+00');
> CREATE TABLE sales_2024_q4 PARTITION OF sales_partitioned
>     FOR VALUES FROM ('2024-10-01 00:00:00+00') TO ('2025-01-01 00:00:00+00');
>
> -- Tạo index trên bảng mẹ (PG 11+ tự động tạo index tương ứng trên tất cả các phân vùng con!)
> CREATE INDEX idx_sales_part_date ON sales_partitioned (sale_date);
>
> -- Nạp 100.000 dòng trải đều trong năm 2024
> INSERT INTO sales_partitioned (sale_date, customer_id, amount)
> SELECT TIMESTAMPTZ '2024-01-01 00:00:00+00' + (g * INTERVAL '5 minutes'),
>        (g % 1000) + 1,
>        (random() * 500)::numeric(10,2)
>   FROM generate_series(1, 100000) AS g;
>
> ANALYZE sales_partitioned;
>
> -- ❌ CÁCH SAI (Điều kiện bị che khuất): Bọc hàm date_trunc quanh cột phân vùng sale_date
> -- Hậu quả: Vừa hỏng Partition Pruning (phải quét cả 4 phân vùng Q1..Q4), vừa hỏng B-Tree Index!
> EXPLAIN (ANALYZE, BUFFERS)
> SELECT sum(amount) FROM sales_partitioned
>  WHERE date_trunc('day', sale_date AT TIME ZONE 'UTC') = TIMESTAMP '2024-05-15';
> ```
>
> Kế hoạch thực thi khi bị che khuất (quét toàn bộ **cả 4 phân vùng**):
>
> ```text
>                                                             QUERY PLAN
> ----------------------------------------------------------------------------------------------------------------------------------
>  Aggregate  (cost=2685.00..2685.01 rows=1 width=32) (actual time=18.412..18.414 rows=1 loops=1)
>    Buffers: shared hit=834
>    ->  Append  (cost=0.00..2683.75 rows=500 width=6) (actual time=3.910..18.350 rows=288 loops=1)
>          ->  Seq Scan on sales_2024_q1 s1  (cost=0.00..670.31 rows=130 width=6) (actual time=3.905..3.905 rows=0 loops=1)
>                Filter: (date_trunc('day'::text, (sale_date AT TIME ZONE 'UTC'::text)) = '2024-05-15 00:00:00'::timestamp)
>          ->  Seq Scan on sales_2024_q2 s2  (cost=0.00..670.31 rows=130 width=6) (actual time=1.812..5.120 rows=288 loops=1)
>                Filter: (date_trunc('day'::text, (sale_date AT TIME ZONE 'UTC'::text)) = '2024-05-15 00:00:00'::timestamp)
>          ->  Seq Scan on sales_2024_q3 s3  (cost=0.00..670.31 rows=130 width=6) (actual time=4.610..4.610 rows=0 loops=1)
>                Filter: (date_trunc('day'::text, (sale_date AT TIME ZONE 'UTC'::text)) = '2024-05-15 00:00:00'::timestamp)
>          ->  Seq Scan on sales_2024_q4 s4  (cost=0.00..670.31 rows=110 width=6) (actual time=4.680..4.680 rows=0 loops=1)
>                Filter: (date_trunc('day'::text, (sale_date AT TIME ZONE 'UTC'::text)) = '2024-05-15 00:00:00'::timestamp)
> ```
>
> ```sql
> -- ✅ CÁCH ĐÚNG (Điều kiện khoảng tường minh):
> -- Hiệu quả kép: Partition Pruning cắt bỏ ngay Q1, Q3, Q4; đồng thời dùng Index Scan trên riêng Q2!
> EXPLAIN (ANALYZE, BUFFERS)
> SELECT sum(amount) FROM sales_partitioned
>  WHERE sale_date >= TIMESTAMPTZ '2024-05-15 00:00:00+00'
>    AND sale_date <  TIMESTAMPTZ '2024-05-16 00:00:00+00';
> ```
>
> Kế hoạch thực thi khi viết khoảng tường minh (nhanh hơn **hơn 200 lần**, chỉ tốn `shared hit=6`):
>
> ```text
>                                                                     QUERY PLAN
> --------------------------------------------------------------------------------------------------------------------------------------------------
>  Aggregate  (cost=12.87..12.88 rows=1 width=32) (actual time=0.074..0.075 rows=1 loops=1)
>    Buffers: shared hit=6
>    ->  Index Scan using sales_2024_q2_sale_date_idx on sales_2024_q2  (cost=0.29..12.15 rows=288 width=6) (actual time=0.019..0.048 rows=288 loops=1)
>          Index Cond: ((sale_date >= '2024-05-15 00:00:00+00'::timestamptz) AND (sale_date < '2024-05-16 00:00:00+00'::timestamptz))
>          Buffers: shared hit=6
>  Planning Time: 0.165 ms
>  Execution Time: 0.095 ms
> ```

### Chuỗi chứa số (Numeric Strings)

**Chuỗi chứa số** (*numeric strings*) là những con số được lưu trong các cột kiểu văn bản. Mặc dù đây là một thói quen rất tệ, nó không tự động khiến index trở nên vô dụng nếu bạn **nhất quán** xử lý nó như chuỗi:

```sql
SELECT ...
  FROM ...
 WHERE numeric_string = '42'
```

Dĩ nhiên câu lệnh này có thể dùng index trên `NUMERIC_STRING`. Tuy nhiên, nếu bạn so sánh nó với một **số**, database sẽ không thể dùng điều kiện này làm access predicate nữa.

```sql
SELECT ...
  FROM ...
 WHERE numeric_string = 42
```

Hãy để ý là thiếu dấu nháy. Mặc dù một số database báo lỗi (ví dụ PostgreSQL), nhiều database chỉ đơn giản thêm vào một phép chuyển đổi kiểu ngầm định.

```sql
SELECT ...
  FROM ...
 WHERE TO_NUMBER(numeric_string) = 42
```

Đây là cùng một vấn đề như trước. Index trên `NUMERIC_STRING` không thể được dùng vì lời gọi hàm. Giải pháp cũng giống như trước: đừng chuyển đổi cột của bảng, thay vào đó hãy chuyển đổi giá trị tìm kiếm.

```sql
SELECT ...
  FROM ...
 WHERE numeric_string = TO_CHAR(42)
```

Có thể bạn thắc mắc: tại sao database không tự động làm theo cách này? Đó là vì chuyển một chuỗi thành số **luôn cho một kết quả duy nhất, không mơ hồ**. Chiều ngược lại thì không như vậy. Một con số, khi được định dạng thành văn bản, có thể chứa khoảng trắng, dấu câu và số 0 ở đầu. Một giá trị duy nhất có thể được viết theo nhiều cách:

```text
42
042
0042
00042
...
```

Database không thể biết định dạng số được dùng trong cột `NUMERIC_STRING`, nên nó làm theo chiều ngược lại: database chuyển chuỗi thành số — đây là một phép biến đổi không mơ hồ.

Hàm `TO_CHAR` chỉ trả về **một** cách biểu diễn chuỗi của con số. Vì vậy nó chỉ khớp với chuỗi đầu tiên trong danh sách trên (`'42'`). Nếu ta dùng `TO_NUMBER`, nó khớp với tất cả. Điều đó có nghĩa là giữa hai cách viết không chỉ có khác biệt về hiệu năng mà còn có **khác biệt về ngữ nghĩa** (kết quả trả về có thể khác nhau)!

Dùng chuỗi chứa số nói chung là phiền phức: quan trọng nhất, nó gây vấn đề hiệu năng do chuyển đổi ngầm định, và còn tạo ra nguy cơ gặp lỗi chuyển đổi do các số không hợp lệ. Ngay cả truy vấn tầm thường nhất, không dùng hàm nào trong mệnh đề where, cũng có thể bị dừng giữa chừng với lỗi chuyển đổi nếu trong bảng chỉ cần có một giá trị số không hợp lệ.

> **Mẹo (Tip)**
>
> Hãy dùng kiểu số để lưu số.

Lưu ý rằng vấn đề này không tồn tại theo chiều ngược lại:

```sql
SELECT ...
  FROM ...
 WHERE numeric_number = '42'
```

Database sẽ nhất quán chuyển chuỗi thành số. Nó không áp dụng hàm lên cột (có thể đang được index): vì vậy một index thông thường sẽ hoạt động. Tuy nhiên, vẫn có thể tự tay chuyển đổi sai chiều:

```sql
SELECT ...
  FROM ...
 WHERE TO_CHAR(numeric_number) = '42'
```

> 💡 **Ví dụ bổ sung (không có trong sách):** Bảng `customers` lưu số điện thoại/mã khách hàng ở cột `cust_code VARCHAR2(10)` có index `cust_code_ix`:
>
> | cust_id | cust_code |
> |---|---|
> | 1 | `'42'` |
> | 2 | `'042'` |
> | 3 | `'A17'` |
>
> ```sql
> -- (1) Oracle: chuyển đổi ngầm → TO_NUMBER("CUST_CODE")=42 → TABLE ACCESS FULL
> --     và còn bị lỗi ORA-01722: invalid number khi gặp dòng 'A17'!
> SELECT * FROM customers WHERE cust_code = 42;
>
> -- (2) Đúng kiểu: INDEX RANGE SCAN trên cust_code_ix, chỉ trả về dòng 1
> SELECT * FROM customers WHERE cust_code = '42';
> ```
>
> Câu (1) — nếu không có dòng `'A17'` — sẽ trả về **cả dòng 1 và dòng 2** (vì `'042'` → 42), còn câu (2) chỉ trả về dòng 1. Đây chính là "khác biệt về ngữ nghĩa" mà tác giả nhắc tới.
>
> Hành vi theo DB: PostgreSQL báo lỗi `operator does not exist: character varying = integer`; SQL Server chuyển cột chuỗi thành số (do `int` có độ ưu tiên kiểu cao hơn) → mất khả năng *index seek* và có thể lỗi chuyển đổi; MySQL chuyển cả hai vế sang số dạng thực → không dùng được index trên cột chuỗi.

### Kết hợp nhiều cột (Combining Columns)

Phần này nói về một kiểu che khuất phổ biến ảnh hưởng tới concatenated index (index ghép nhiều cột).

Ví dụ đầu tiên lại là về kiểu ngày và giờ, nhưng theo chiều ngược lại. Truy vấn MySQL sau kết hợp một cột ngày và một cột giờ để áp dụng một bộ lọc khoảng trên cả hai:

```sql
SELECT ...
  FROM ...
 WHERE ADDTIME(date_column, time_column)
     > DATE_ADD(now(), INTERVAL -1 DAY)
```

Nó chọn tất cả các bản ghi trong 24 giờ qua. Truy vấn không thể dùng đúng cách một concatenated index trên (`DATE_COLUMN`, `TIME_COLUMN`) vì việc tìm kiếm không thực hiện trên các cột được index mà trên **dữ liệu dẫn xuất** (*derived data*).

Bạn có thể tránh vấn đề này bằng cách dùng một kiểu dữ liệu có cả thành phần ngày và giờ (ví dụ `DATETIME` của MySQL). Khi đó bạn có thể dùng cột này mà không cần gọi hàm:

```sql
SELECT ...
  FROM ...
 WHERE datetime_column
     > DATE_ADD(now(), INTERVAL -1 DAY)
```

Đáng tiếc là khi gặp vấn đề này, ta thường không thể thay đổi bảng.

Lựa chọn tiếp theo là function-based index nếu database hỗ trợ — dù cách này có tất cả những nhược điểm đã bàn trước đó. Khi dùng MySQL, function-based index dù sao cũng không phải là một lựa chọn (xem ghi chú về MySQL 8.0.13 ở trên).

Vẫn có thể viết truy vấn sao cho database dùng được một concatenated index trên `DATE_COLUMN, TIME_COLUMN` với access predicate — ít nhất là một phần. Để làm vậy, ta thêm một điều kiện bổ sung trên `DATE_COLUMN`.

```sql
 WHERE ADDTIME(date_column, time_column)
     > DATE_ADD(now(), INTERVAL -1 DAY)
   AND date_column
    >= DATE(DATE_ADD(now(), INTERVAL -1 DAY))
```

Điều kiện mới hoàn toàn dư thừa (*redundant*) về mặt kết quả, nhưng nó là một bộ lọc thẳng trên `DATE_COLUMN` có thể được dùng làm access predicate. Mặc dù kỹ thuật này không hoàn hảo, nó thường là một phép xấp xỉ đủ tốt.

> **Mẹo (Tip)**
>
> Hãy dùng một điều kiện dư thừa trên cột quan trọng nhất (*most significant column*) khi một điều kiện khoảng kết hợp nhiều cột.
>
> Với PostgreSQL, nên dùng cú pháp **row values** (giá trị hàng) được mô tả ở trang 151.

> 📝 **Giải thích thêm (không có trong sách):** Cú pháp row values trong PostgreSQL cho phép so sánh "bộ" nhiều cột theo thứ tự từ điển và dùng trực tiếp index ghép, ví dụ:
>
> ```sql
> -- PostgreSQL
> SELECT ...
>   FROM t
>  WHERE (date_column, time_column) > (CURRENT_DATE - 1, LOCALTIME);
> ```
>
> Điều kiện này tương đương `date_column > d OR (date_column = d AND time_column > t)` và PostgreSQL dùng nó làm access predicate trên index `(date_column, time_column)`. Sách sẽ trình bày chi tiết ở phần phân trang (*Paging Through Results*).

Bạn cũng có thể dùng kỹ thuật này khi lưu ngày và giờ trong các cột văn bản, nhưng bạn phải dùng các định dạng ngày giờ cho ra **thứ tự theo thời gian khi được sắp xếp theo thứ tự từ điển** (*lexically*) — ví dụ định dạng do ISO 8601 đề xuất (`YYYY-MM-DD HH:MM:SS`). Ví dụ sau dùng hàm `TO_CHAR` của Oracle Database cho mục đích đó:

```sql
SELECT ...
  FROM ...
 WHERE date_string || time_string
     > TO_CHAR(sysdate - 1, 'YYYY-MM-DD HH24:MI:SS')
   AND date_string
    >= TO_CHAR(sysdate - 1, 'YYYY-MM-DD')
```

Chúng ta sẽ lại gặp vấn đề áp dụng điều kiện khoảng trên nhiều cột trong phần có tên "Phân trang kết quả" (*Paging Through Results*). Ở đó ta cũng sẽ dùng cùng phương pháp xấp xỉ này để giảm nhẹ vấn đề.

Đôi khi ta gặp trường hợp ngược lại: ta **muốn** cố tình che khuất một điều kiện để nó không còn được dùng làm access predicate nữa. Ta đã xem xét vấn đề đó khi bàn về ảnh hưởng của bind parameter lên điều kiện `LIKE`. Hãy xét ví dụ sau:

```sql
SELECT last_name, first_name, employee_id
  FROM employees
 WHERE subsidiary_id = ?
   AND last_name LIKE ?
```

Giả sử có một index trên `SUBSIDIARY_ID` và một index khác trên `LAST_NAME`, index nào tốt hơn cho truy vấn này?

Nếu không biết vị trí của ký tự đại diện (*wildcard*, ví dụ `%`) trong chuỗi tìm kiếm, ta không thể đưa ra câu trả lời có căn cứ. Optimizer không có lựa chọn nào khác ngoài "đoán". Nếu bạn biết rằng chuỗi tìm kiếm luôn có wildcard ở đầu (như `'%son'`), bạn có thể cố tình che khuất điều kiện `LIKE` để optimizer không còn xem xét index trên `LAST_NAME` nữa.

```sql
SELECT last_name, first_name, employee_id
  FROM employees
 WHERE subsidiary_id = ?
   AND last_name || '' LIKE ?
```

Chỉ cần nối thêm một chuỗi rỗng vào cột `LAST_NAME` là đủ. Tuy nhiên, đây là lựa chọn cuối cùng. Chỉ làm điều này khi thực sự cần thiết.

> 💡 **Ví dụ bổ sung (không có trong sách):** Bảng `events` (MySQL 5.x) lưu ngày và giờ tách riêng, có index `ev_dt (ev_date, ev_time)`, chứa dữ liệu 3 năm (~1 triệu dòng/ngày):
>
> | ev_id | ev_date | ev_time |
> |---|---|---|
> | 1 | 2024-05-09 | 08:00:00 |
> | 2 | 2024-05-09 | 21:30:00 |
> | 3 | 2024-05-10 | 07:15:00 |
>
> Giả sử bây giờ là `2024-05-10 09:00:00`.
>
> ```sql
> -- ❌ Chỉ có điều kiện trên dữ liệu dẫn xuất: duyệt toàn bộ ~1 tỷ dòng
> SELECT * FROM events
>  WHERE ADDTIME(ev_date, ev_time) > NOW() - INTERVAL 1 DAY;
>
> -- ✅ Thêm điều kiện dư thừa trên cột đứng đầu index
> SELECT * FROM events
>  WHERE ADDTIME(ev_date, ev_time) > NOW() - INTERVAL 1 DAY
>    AND ev_date >= DATE(NOW() - INTERVAL 1 DAY);   -- ev_date >= '2024-05-09'
> ```
>
> Với câu thứ hai, MySQL dùng `range` access trên `ev_dt` để chỉ đọc các dòng từ `2024-05-09` trở đi (khoảng 2 ngày dữ liệu ≈ 2 triệu dòng), rồi mới lọc chính xác bằng `ADDTIME(...)`. Dòng 1 (`08:00` ngày 9) bị loại ở bước lọc, dòng 2 và 3 được trả về. Phần "thừa" phải đọc chỉ là phần đầu của ngày hôm qua — đó là ý nghĩa của "xấp xỉ đủ tốt".
>
> Ở trường hợp ngược lại (cố tình che khuất), một cách khác thường dùng với số là `numeric_col + 0`, như sẽ thấy ở mục *Math* bên dưới. Trên Oracle, cách "chính thống" hơn là dùng hint, ví dụ `/*+ NO_INDEX(employees emp_name) */`, hoặc trên SQL Server là `WITH (INDEX(...))` để ép index mong muốn.

### Logic "thông minh" (Smart Logic)

Một trong những tính năng then chốt của các database SQL là hỗ trợ **truy vấn ad-hoc** (*ad-hoc queries* — truy vấn tùy ý, không định trước): các truy vấn mới có thể được thực thi bất cứ lúc nào. Điều này chỉ khả thi vì bộ tối ưu hóa truy vấn (*query optimizer*, hay *query planner*) làm việc lúc chạy (*at runtime*); nó phân tích từng câu lệnh khi nhận được và lập ngay một execution plan hợp lý. Chi phí phát sinh do tối ưu hóa lúc chạy có thể được giảm thiểu bằng bind parameter.

Ý chính của phần ôn lại này là: database được tối ưu cho **SQL động** (*dynamic SQL*) — vì vậy hãy dùng nó nếu bạn cần.

Tuy vậy, có một thói quen được dùng rộng rãi là tránh SQL động để dùng **SQL tĩnh** (*static SQL*) — thường là vì lời đồn "SQL động thì chậm". Thói quen này gây hại nhiều hơn lợi nếu database dùng một **bộ nhớ đệm execution plan dùng chung** (*shared execution plan cache*) như DB2, Oracle Database hay SQL Server.

Để minh họa, hãy hình dung một ứng dụng truy vấn bảng `EMPLOYEES`. Ứng dụng cho phép tìm kiếm theo mã chi nhánh (*subsidiary id*), mã nhân viên (*employee id*) và họ (*last name*, không phân biệt hoa thường) theo bất kỳ tổ hợp nào. Vẫn có thể viết một truy vấn duy nhất bao phủ mọi trường hợp bằng cách dùng logic "thông minh".

```sql
SELECT first_name, last_name, subsidiary_id, employee_id
  FROM employees
 WHERE ( subsidiary_id    = :sub_id OR :sub_id IS NULL )
   AND ( employee_id      = :emp_id OR :emp_id IS NULL )
   AND ( UPPER(last_name) = :name   OR :name   IS NULL )
```

Truy vấn dùng các bind variable có tên (*named bind variables*) để dễ đọc hơn. Tất cả các biểu thức lọc khả dĩ đều được viết cứng (tĩnh) trong câu lệnh. Mỗi khi không cần một bộ lọc nào đó, bạn chỉ cần truyền `NULL` thay cho giá trị tìm kiếm: nó sẽ vô hiệu hóa điều kiện đó thông qua logic `OR`.

Đây là một câu lệnh SQL hoàn toàn hợp lý. Việc dùng `NULL` thậm chí còn phù hợp với định nghĩa của nó theo **logic ba giá trị** (*three-valued logic*: TRUE / FALSE / UNKNOWN) của SQL. Tuy nhiên, đây là **một trong những anti-pattern hiệu năng tệ nhất**.

Database không thể tối ưu execution plan cho một bộ lọc cụ thể vì bất kỳ bộ lọc nào cũng có thể bị vô hiệu hóa lúc chạy. Database cần chuẩn bị cho trường hợp xấu nhất — khi tất cả các bộ lọc đều bị vô hiệu hóa:

```text
----------------------------------------------------
| Id | Operation         | Name      | Rows | Cost |
----------------------------------------------------
|  0 | SELECT STATEMENT  |           |    2 |  478 |
|* 1 |  TABLE ACCESS FULL| EMPLOYEES |    2 |  478 |
----------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------
1 - filter((:NAME   IS NULL OR UPPER("LAST_NAME")=:NAME)
       AND (:EMP_ID IS NULL OR "EMPLOYEE_ID"=:EMP_ID)
       AND (:SUB_ID IS NULL OR "SUBSIDIARY_ID"=:SUB_ID))
```

Hệ quả là database dùng full table scan (quét toàn bộ bảng) ngay cả khi có index cho từng cột.

Không phải database không thể "giải" được logic "thông minh" này. Nó tạo ra execution plan **chung chung** (*generic*) là do việc dùng bind parameter, để plan có thể được lưu vào cache và tái sử dụng với các giá trị khác về sau. Nếu ta không dùng bind parameter mà viết các giá trị thực vào câu lệnh SQL, optimizer sẽ chọn đúng index cho bộ lọc đang có hiệu lực:

```sql
SELECT first_name, last_name, subsidiary_id, employee_id
  FROM employees
 WHERE( subsidiary_id    = NULL     OR NULL IS NULL )
   AND( employee_id      = NULL     OR NULL IS NULL )
   AND( UPPER(last_name) = 'WINAND' OR 'WINAND' IS NULL )
```

```text
---------------------------------------------------------------
|Id | Operation                   | Name        | Rows | Cost |
---------------------------------------------------------------
| 0 | SELECT STATEMENT            |             |    1 |    2 |
| 1 |  TABLE ACCESS BY INDEX ROWID| EMPLOYEES   |    1 |    2 |
|*2 |   INDEX RANGE SCAN          | EMP_UP_NAME |    1 |    1 |
---------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------
  2 - access(UPPER("LAST_NAME")='WINAND')
```

Tuy nhiên, đây **không phải** giải pháp. Nó chỉ chứng minh rằng database có thể giải được các điều kiện này.

> ⚠️ **Cảnh báo (Warning)**
>
> Dùng giá trị literal (giá trị viết thẳng vào câu SQL) khiến ứng dụng của bạn dễ bị tấn công **SQL injection** và có thể gây vấn đề hiệu năng do chi phí tối ưu hóa tăng lên.

Giải pháp hiển nhiên cho các truy vấn động là SQL động. Theo **nguyên tắc KISS**³ (*Keep It Simple, Stupid* — "hãy giữ cho mọi thứ đơn giản"), chỉ cần nói với database thứ bạn cần ngay lúc này — và không gì khác.

```sql
SELECT first_name, last_name, subsidiary_id, employee_id
  FROM employees
 WHERE UPPER(last_name) = :name
```

Lưu ý rằng truy vấn **vẫn dùng bind parameter**.

> **Mẹo (Tip)**
>
> Hãy dùng SQL động nếu bạn cần mệnh đề where động.
>
> Vẫn phải dùng bind parameter khi sinh SQL động — nếu không, lời đồn "SQL động thì chậm" sẽ trở thành sự thật.

Vấn đề mô tả trong phần này rất phổ biến. Tất cả các database dùng shared execution plan cache đều có một tính năng để đối phó với nó — thường lại kéo theo những vấn đề và lỗi mới.

³ http://en.wikipedia.org/wiki/KISS_principle

#### MySQL

MySQL không gặp vấn đề cụ thể này vì nó hoàn toàn không có execution plan cache. Một yêu cầu tính năng (*feature request*) từ năm 2009 thảo luận về tác động của việc cache execution plan. Có vẻ như optimizer của MySQL đủ đơn giản để việc cache execution plan không đáng công.

#### Oracle Database

Oracle Database dùng một shared execution plan cache ("SQL area") và hoàn toàn chịu ảnh hưởng của vấn đề mô tả trong phần này.

Oracle giới thiệu cái gọi là **bind peeking** ("nhìn trộm" giá trị bind) từ phiên bản 9i. Bind peeking cho phép optimizer dùng các giá trị bind thực tế của **lần thực thi đầu tiên** khi lập execution plan. Vấn đề của cách tiếp cận này là hành vi **không tất định** (*nondeterministic*): các giá trị của lần thực thi đầu tiên ảnh hưởng tới mọi lần thực thi. Execution plan có thể thay đổi mỗi khi database khởi động lại hoặc — khó lường trước hơn — khi plan trong cache hết hạn và optimizer tạo lại nó bằng các giá trị khác ở lần câu lệnh được thực thi tiếp theo.

Phiên bản 11g giới thiệu **adaptive cursor sharing** (chia sẻ cursor thích ứng) để cải thiện tình hình hơn nữa. Tính năng này cho phép database cache **nhiều** execution plan cho cùng một câu lệnh SQL. Hơn nữa, optimizer "nhìn trộm" các bind parameter và lưu **độ chọn lọc ước tính** (*estimated selectivity*) của chúng cùng với execution plan. Khi truy cập cache lần sau, độ chọn lọc của các giá trị bind hiện tại phải nằm trong các khoảng độ chọn lọc của một execution plan đã cache thì plan đó mới được tái sử dụng. Nếu không, optimizer tạo một execution plan mới và so sánh nó với các execution plan đã cache cho truy vấn này. Nếu đã có một execution plan như vậy, database thay thế nó bằng một execution plan mới cũng bao phủ cả ước tính độ chọn lọc của các giá trị bind hiện tại. Nếu chưa có, nó cache một biến thể execution plan mới cho truy vấn này — tất nhiên là kèm theo các ước tính độ chọn lọc.

#### PostgreSQL

Query plan cache của PostgreSQL chỉ hoạt động cho các câu lệnh đang mở (*open statements*) — tức là chừng nào bạn còn giữ `PreparedStatement` mở. Vấn đề mô tả ở trên chỉ xảy ra khi tái sử dụng một statement handle. Lưu ý rằng JDBC driver của PostgreSQL chỉ bật cache sau lần thực thi thứ năm.

> 📝 **Giải thích thêm (không có trong sách):** Từ PostgreSQL 9.2, với prepared statement phía server, PostgreSQL dùng *custom plan* (lập plan theo giá trị thực) cho 5 lần thực thi đầu, sau đó mới cân nhắc chuyển sang *generic plan* nếu chi phí ước tính không tệ hơn đáng kể. Từ PostgreSQL 12 có tham số `plan_cache_mode` (`auto` / `force_custom_plan` / `force_generic_plan`) để kiểm soát hành vi này.

#### SQL Server

SQL Server dùng cái gọi là **parameter sniffing** ("đánh hơi" tham số). Parameter sniffing cho phép optimizer dùng các giá trị bind thực tế của lần thực thi đầu tiên trong quá trình phân tích (*parsing*). Vấn đề của cách tiếp cận này là hành vi không tất định: các giá trị của lần thực thi đầu tiên ảnh hưởng tới mọi lần thực thi. Execution plan có thể thay đổi mỗi khi database khởi động lại hoặc — khó lường trước hơn — khi plan trong cache hết hạn và optimizer tạo lại nó bằng các giá trị khác ở lần câu lệnh được thực thi tiếp theo.

SQL Server 2005 bổ sung các **query hint** (gợi ý truy vấn) mới để kiểm soát tốt hơn parameter sniffing và việc biên dịch lại (*recompiling*). Query hint `RECOMPILE` bỏ qua plan cache cho một câu lệnh được chọn. `OPTIMIZE FOR` cho phép chỉ định các giá trị tham số thực tế chỉ dùng cho việc tối ưu hóa. Cuối cùng, bạn có thể cung cấp cả một execution plan hoàn chỉnh bằng hint `USE PLAN`.

Bản cài đặt ban đầu của hint `OPTION(RECOMPILE)` có lỗi khiến nó không xét tới tất cả các bind variable. Bản cài đặt mới ra mắt cùng SQL Server 2008 lại có một lỗi khác, khiến tình hình rất rối rắm. Erland Sommarskog⁴ đã tổng hợp mọi thông tin liên quan, bao quát tất cả các phiên bản SQL Server.

⁴ http://www.sommarskog.se/dyn-search-2008.html

Mặc dù các phương pháp **heuristic** (dựa trên kinh nghiệm/ước đoán) có thể cải thiện vấn đề "logic thông minh" ở một mức độ nào đó, thực ra chúng được tạo ra để giải quyết các vấn đề của bind parameter liên quan tới **histogram** của cột (thống kê phân bố giá trị) và các biểu thức `LIKE`.

Phương pháp đáng tin cậy nhất để có được execution plan tốt nhất là **tránh các bộ lọc không cần thiết trong câu lệnh SQL**.

> 💡 **Ví dụ bổ sung (không có trong sách):** Sinh SQL động **có bind parameter** trong Java (JDBC), thay cho truy vấn "smart logic":
>
> ```java
> StringBuilder sql = new StringBuilder(
>     "SELECT first_name, last_name, subsidiary_id, employee_id"
>   + "  FROM employees WHERE 1=1");
> List<Object> params = new ArrayList<>();
> if (subId != null) { sql.append(" AND subsidiary_id = ?");    params.add(subId); }
> if (empId != null) { sql.append(" AND employee_id = ?");      params.add(empId); }
> if (name  != null) { sql.append(" AND UPPER(last_name) = ?"); params.add(name.toUpperCase()); }
>
> PreparedStatement ps = con.prepareStatement(sql.toString());
> for (int i = 0; i < params.size(); i++) ps.setObject(i + 1, params.get(i));
> ```
>
> Với 3 bộ lọc tùy chọn, có tối đa 2³ = 8 biến thể câu SQL khác nhau. Mỗi biến thể có execution plan riêng, được cache riêng và luôn tối ưu: chỉ lọc theo tên → `INDEX RANGE SCAN` trên `EMP_UP_NAME`; theo `employee_id` + `subsidiary_id` → dùng index khóa chính; v.v. Số lượng câu lệnh nhỏ và cố định, nên vẫn tận dụng tốt plan cache. (`1=1` chỉ là mẹo để luôn nối được `AND`; optimizer loại bỏ nó ngay.)
>
> Trên SQL Server, nếu buộc phải giữ truy vấn "smart logic", có thể thêm `OPTION (RECOMPILE)` ở cuối để optimizer lập plan theo giá trị thực ở mỗi lần chạy (đánh đổi bằng chi phí biên dịch mỗi lần) — trên các bản hiện đại (SQL Server 2008 SP1 CU5 / 2008 R2 trở lên) hint này đã xử lý đúng mẫu `(@p IS NULL OR col = @p)`.

> 📝 **Giải thích thêm (không có trong sách) — Đối chiếu với PostgreSQL hiện đại (PG 10–17): Cơ chế tối ưu hóa điều kiện `OR` bằng `BitmapOr` (`Bitmap Index Scan` + `Bitmap Heap Scan`) và xử lý "Smart Logic":**
>
> Khi ứng dụng có các điều kiện lọc kết hợp bằng toán tử `OR` hoặc tìm kiếm động trên nhiều cột, PostgreSQL hiện đại xử lý theo hai cơ chế rất quan trọng mà lập trình viên cần hiểu sâu:
>
> 1. **Cách PostgreSQL xử lý "Smart Logic" `(col = $1 OR $1 IS NULL)` với Custom Plan vs. Generic Plan:**
>    - Nếu câu truy vấn chạy dưới dạng câu lệnh thông thường (không qua Prepared Statement) hoặc chạy bằng **Custom Plan** (5 lần đầu của Prepared Statement, hoặc khi bật `SET plan_cache_mode = force_custom_plan` từ **PostgreSQL 12+**), bộ tối ưu hóa của PostgreSQL sẽ thực hiện **rút gọn hằng số (*constant folding*)** ngay trước khi lập kế hoạch: với `$1 = NULL`, biểu thức `(col = NULL OR NULL IS NULL)` được rút gọn thành `TRUE` và bị loại bỏ hoàn toàn; với `$1 = 42`, biểu thức `(col = 42 OR 42 IS NULL)` được rút gọn thành `col = 42` và **dùng được `Index Scan` bình thường**!
>    - Tuy nhiên, ngay khi Prepared Statement chuyển sang **Generic Plan** (từ lần chạy thứ 6 ở chế độ `auto`), planner không biết `$1` có phải là `NULL` hay không nên không thể rút gọn biểu thức `OR`, dẫn đến **`Seq Scan` toàn bảng** giống hệt Oracle trong sách. Vì vậy, sinh **SQL động kèm bind parameter** (hoặc bật `plan_cache_mode = force_custom_plan` nếu bắt buộc dùng câu SQL tĩnh trong hàm PL/pgSQL) vẫn là quy tắc vàng.
> 2. **Cách PostgreSQL tối ưu điều kiện `OR` thật sự giữa nhiều cột (`WHERE subsidiary_id = $1 OR employee_id = $2`) bằng `BitmapOr`:**
>    - Một lần duyệt B-Tree đơn lẻ (`Index Scan`) không thể nhảy cóc qua hai index trên hai cột khác nhau nối với nhau bằng `OR`. Thay vì phải quét toàn bộ bảng (`Seq Scan`), PostgreSQL sử dụng kiến trúc **quét hai pha bằng Bitmap trong RAM**:
>      - **Pha 1 (`Bitmap Index Scan` + `BitmapOr` / `BitmapAnd`):** PostgreSQL duyệt từng B-Tree index độc lập (`idx_sub_id`, `idx_emp_id`). Thay vì đọc bảng ngay, mỗi `Bitmap Index Scan` tạo ra một mảng bit trong bộ nhớ (được cấp phát tối đa bằng tham số **`work_mem`**) đánh dấu các vị trí dòng (`TID`) thỏa mãn điều kiện. Sau đó nút **`BitmapOr`** thực hiện phép `OR` bitwise cực nhanh giữa hai bitmap (đồng thời tự động loại bỏ các dòng trùng lặp nếu một dòng thỏa cả hai vế của `OR`!).
>      - **Pha 2 (`Bitmap Heap Scan`):** PostgreSQL sắp xếp các trang bảng (*heap pages*) có bit bật theo đúng thứ tự vật lý tăng dần trên đĩa rồi mới đọc bảng tuần tự, biến I/O ngẫu nhiên (*random I/O*) thành I/O tuần tự (*sequential I/O*) và tận dụng cơ chế đọc trước (*read-ahead / prefetching* với `effective_io_concurrency`, được cải tiến mạnh mẽ qua API `ReadStream` trong **PostgreSQL 17**).
>    - **Lưu ý quan trọng về `Heap Blocks: exact` vs. `lossy` trong `EXPLAIN (ANALYZE)`:**
>      - Khi bitmap nằm gọn trong `work_mem`, bitmap lưu chính xác từng dòng (`TID`), và `EXPLAIN (ANALYZE)` hiển thị **`Heap Blocks: exact=N`**.
>      - Nếu kết quả khớp quá lớn làm bitmap vượt quá giới hạn `work_mem`, PostgreSQL tự động chuyển một phần bitmap sang chế độ **`lossy`** (chỉ nhớ số hiệu trang 8 KB chứ không nhớ từng dòng trong trang). Khi đó `EXPLAIN (ANALYZE)` sẽ hiện **`Heap Blocks: exact=N lossy=M`** kèm theo dòng **`Rows Removed by Index Recheck: K`** — vì ở các trang `lossy`, `Bitmap Heap Scan` phải đọc mọi dòng trong trang đó rồi kiểm tra lại điều kiện `Recheck Cond`. Nếu bạn thấy `lossy` xuất hiện thường xuyên trong `EXPLAIN (ANALYZE)`, hãy tăng `work_mem` cho phiên hoặc truy vấn đó!

> 💡 **Ví dụ bổ sung (không có trong sách) — Thực hành trên PostgreSQL: Phân tích `BitmapOr` (`Bitmap Index Scan` + `Bitmap Heap Scan`) trong `EXPLAIN (ANALYZE, BUFFERS)`:**
>
> ```sql
> -- Giả sử bảng employees có 2 B-Tree index độc lập trên subsidiary_id và upper(last_name)
> CREATE INDEX IF NOT EXISTS idx_emp_sub_id ON employees (subsidiary_id);
> CREATE INDEX IF NOT EXISTS idx_emp_up_lname ON employees (UPPER(last_name));
>
> -- Truy vấn dùng điều kiện OR trên hai cột khác nhau:
> EXPLAIN (ANALYZE, BUFFERS)
> SELECT employee_id, first_name, last_name, subsidiary_id
>   FROM employees
>  WHERE subsidiary_id = 20
>     OR UPPER(last_name) = 'WINAND';
> ```
>
> Kế hoạch thực thi thực tế trên PostgreSQL:
>
> ```text
>                                                             QUERY PLAN
> -----------------------------------------------------------------------------------------------------------------------------------
>  Bitmap Heap Scan on employees  (cost=8.86..124.50 rows=100 width=28) (actual time=0.045..0.182 rows=100 loops=1)
>    Recheck Cond: ((subsidiary_id = 20) OR (upper((last_name)::text) = 'WINAND'::text))
>    Heap Blocks: exact=85
>    Buffers: shared hit=91
>    ->  BitmapOr  (cost=8.86..8.86 rows=100 width=0) (actual time=0.034..0.035 rows=0 loops=1)
>          Buffers: shared hit=6
>          ->  Bitmap Index Scan on idx_emp_sub_id  (cost=0.00..4.40 rows=99 width=0) (actual time=0.022..0.022 rows=99 loops=1)
>                Index Cond: (subsidiary_id = 20)
>                Buffers: shared hit=3
>          ->  Bitmap Index Scan on idx_emp_up_lname  (cost=0.00..4.41 rows=1 width=0) (actual time=0.010..0.011 rows=1 loops=1)
>                Index Cond: (upper((last_name)::text) = 'WINAND'::text)
>                Buffers: shared hit=3
>  Planning Time: 0.140 ms
>  Execution Time: 0.215 ms
> ```
>
> **Cách đọc thông tin từ `EXPLAIN (ANALYZE, BUFFERS)` trên:**
> - Nhánh `Bitmap Index Scan on idx_emp_sub_id` tìm thấy 99 dòng (tốn 3 buffers); nhánh `Bitmap Index Scan on idx_emp_up_lname` tìm thấy 1 dòng (tốn 3 buffers).
> - Nút `BitmapOr` gộp 2 bitmap lại trong RAM, và `Bitmap Heap Scan` đọc đúng **85 trang bảng (`Heap Blocks: exact=85`, không có trang `lossy` nào)** theo thứ tự vật lý tăng dần, hoàn tất toàn bộ câu truy vấn `OR` chỉ trong `0.215 ms`!

### Phép toán (Math)

Còn một nhóm che khuất nữa, rất "thông minh", ngăn cản việc dùng index đúng cách. Thay vì dùng biểu thức logic, nó dùng **phép tính**.

Hãy xét câu lệnh sau. Nó có thể dùng index trên `NUMERIC_NUMBER` không?

```sql
SELECT numeric_number
  FROM table_name
 WHERE numeric_number - 1000 > ?
```

Tương tự, câu lệnh sau có thể dùng một index trên `A` và `B` không — thứ tự cột do bạn chọn?

```sql
SELECT a, b
  FROM table_name
 WHERE 3*a + 5 = b
```

Hãy nhìn các câu hỏi này từ một góc độ khác: nếu bạn đang phát triển một database SQL, bạn có thêm vào một **bộ giải phương trình** (*equation solver*) không? Hầu hết các nhà cung cấp database đều nói "Không!" — và vì vậy, cả hai ví dụ trên đều không dùng index.

Bạn thậm chí có thể dùng phép toán để cố tình che khuất một điều kiện — như ta đã làm trước đó với tìm kiếm toàn văn bằng `LIKE`. Ví dụ, chỉ cần cộng thêm 0 là đủ:

```sql
SELECT numeric_number
  FROM table_name
 WHERE numeric_number + 0 = ?
```

Tuy vậy, ta vẫn có thể đánh index cho các biểu thức này bằng function-based index nếu ta dùng phép tính một cách khéo léo và biến đổi mệnh đề where giống như biến đổi một phương trình:

```sql
SELECT a, b
  FROM table_name
 WHERE 3*a - b = -5
```

Ta chỉ việc chuyển các tham chiếu tới bảng (các cột) sang một vế và các hằng số sang vế còn lại. Khi đó ta có thể tạo một function-based index cho vế trái của phương trình:

```sql
CREATE INDEX math ON table_name (3*a - b)
```

> 💡 **Ví dụ bổ sung (không có trong sách):** Với câu hỏi đầu tiên (`numeric_number - 1000 > ?`), không cần function-based index — chỉ cần tự "giải phương trình" bằng cách chuyển hằng số sang vế bên kia:
>
> ```sql
> -- ❌ Cột nằm trong phép tính → không dùng được index trên numeric_number
> SELECT numeric_number FROM table_name WHERE numeric_number - 1000 > ?;
>
> -- ✅ Tương đương về toán học, cột đứng một mình → INDEX RANGE SCAN
> SELECT numeric_number FROM table_name WHERE numeric_number > ? + 1000;
> ```
>
> Ví dụ cụ thể: bảng `products(price)` có index `prod_price`. Tìm sản phẩm mà giá sau khi giảm 10% vẫn trên 90:
>
> ```sql
> WHERE price * 0.9 > 90      -- ❌ che khuất
> WHERE price > 90 / 0.9      -- ✅ = price > 100, dùng được prod_price
> ```
>
> Với câu hỏi thứ hai (`3*a + 5 = b`), không thể đưa về dạng "một cột = hằng số", nên function-based index `(3*a - b)` như sách là cách đúng. Cú pháp theo DB:
> - Oracle: `CREATE INDEX math ON table_name (3*a - b);`
> - PostgreSQL: biểu thức phải nằm trong hai lớp ngoặc đơn: `CREATE INDEX math ON table_name ((3*a - b));`
> - MySQL 8.0.13+: `CREATE INDEX math ON table_name ((3*a - b));`
> - SQL Server: tạo computed column `ALTER TABLE table_name ADD ab_expr AS (3*a - b);` rồi `CREATE INDEX math ON table_name (ab_expr);` — optimizer có thể tự khớp biểu thức `3*a - b` trong `WHERE` với computed column.
>
> Truy vấn phải viết đúng dạng `WHERE 3*a - b = -5` để khớp với biểu thức được index.

---

## Bảng thuật ngữ (Glossary)

| Thuật ngữ tiếng Anh | Nghĩa tiếng Việt | Giải thích ngắn |
|---|---|---|
| NULL | Giá trị rỗng / chỗ trống | Chỗ giữ chỗ cho giá trị bị thiếu hoặc chưa biết; không phải là một giá trị. Oracle coi chuỗi rỗng `''` là `NULL`. |
| Empty string | Chuỗi rỗng | Chuỗi có độ dài 0; trong Oracle không thể lưu vào `VARCHAR2` (bị đổi thành `NULL`). |
| Three-valued logic | Logic ba giá trị | Logic của SQL gồm TRUE, FALSE và UNKNOWN (kết quả khi so sánh với `NULL`). |
| Partial index | Index một phần | Index chỉ chứa một tập con các dòng (theo điều kiện). Mọi index Oracle ngầm là partial index (bỏ dòng có mọi cột index là `NULL`). |
| Filtered index | Index có lọc | Tên gọi partial index trong SQL Server. |
| Concatenated index | Index ghép | Index trên nhiều cột. |
| Function-based index | Index dựa trên hàm/biểu thức | Index trên kết quả một biểu thức, ví dụ `TRUNC(sale_date)` hoặc `3*a - b`. |
| Constant expression | Biểu thức hằng | Biểu thức không phụ thuộc dữ liệu, ví dụ `'1'`; không bao giờ `NULL`. |
| NOT NULL constraint | Ràng buộc NOT NULL | Ràng buộc đảm bảo cột không bao giờ chứa `NULL`; giúp optimizer biết index chứa mọi dòng. |
| User-defined function | Hàm do người dùng định nghĩa | Hàm tự viết (PL/SQL...); với optimizer là "hộp đen". |
| Deterministic | Tất định | Hàm luôn trả về cùng kết quả với cùng đầu vào; bắt buộc để dùng trong index. |
| Virtual column | Cột ảo | Cột được tính từ biểu thức (`GENERATED ALWAYS AS`), có từ Oracle 11g; có thể đặt ràng buộc lên nó. |
| Access predicate | Vị từ truy cập | Điều kiện dùng để xác định phạm vi duyệt index (điểm đầu/điểm cuối). |
| Filter predicate | Vị từ lọc | Điều kiện chỉ dùng để loại bớt dòng sau khi đã đọc. |
| INDEX RANGE SCAN | Quét theo khoảng trên index | Thao tác duyệt một đoạn của index. |
| TABLE ACCESS FULL / Full table scan | Quét toàn bộ bảng | Đọc mọi dòng của bảng. |
| Obfuscated condition | Điều kiện bị che khuất | Mệnh đề where được viết sao cho index không dùng được đúng cách. |
| Anti-pattern | Mẫu sai lầm điển hình | Cách làm phổ biến nhưng có hại. |
| Explicit range condition | Điều kiện khoảng tường minh | Ví dụ `col >= x AND col < y`; cột đứng một mình, dùng được index. |
| Implicit type conversion | Chuyển đổi kiểu ngầm định | Database tự thêm hàm chuyển kiểu (ví dụ `TO_NUMBER`, `INTERNAL_FUNCTION`), thường làm mất khả năng dùng index. |
| Numeric string | Chuỗi chứa số | Số được lưu trong cột kiểu văn bản. |
| Derived data | Dữ liệu dẫn xuất | Giá trị tính ra từ các cột (ví dụ `ADDTIME(date_column, time_column)`). |
| Most significant column | Cột quan trọng nhất | Cột đứng đầu trong thứ tự sắp xếp (ví dụ cột ngày so với cột giờ). |
| Row values | Giá trị hàng | Cú pháp so sánh nhiều cột như một bộ, ví dụ `(a, b) > (x, y)`. |
| ISO 8601 | Chuẩn ISO 8601 | Định dạng ngày giờ `YYYY-MM-DD HH:MM:SS`, sắp xếp từ điển trùng thứ tự thời gian. |
| Wildcard | Ký tự đại diện | `%` hoặc `_` trong `LIKE`. |
| Ad-hoc query | Truy vấn tùy ý | Truy vấn mới có thể chạy bất cứ lúc nào. |
| Dynamic SQL / Static SQL | SQL động / SQL tĩnh | SQL được sinh lúc chạy theo nhu cầu / SQL viết cố định sẵn. |
| Smart logic | Logic "thông minh" | Mẫu `(col = :p OR :p IS NULL)` để gộp mọi bộ lọc vào một câu tĩnh — anti-pattern hiệu năng. |
| Bind parameter / bind variable | Tham số ràng buộc | Giá trị truyền tách khỏi câu SQL (`?`, `:name`); chống SQL injection, cho phép tái dùng plan. |
| Shared execution plan cache | Bộ nhớ đệm execution plan dùng chung | Nơi lưu plan để tái sử dụng (Oracle "SQL area", SQL Server, DB2). |
| Bind peeking | "Nhìn trộm" giá trị bind | Oracle 9i+: dùng giá trị bind của lần chạy đầu để lập plan. |
| Adaptive cursor sharing | Chia sẻ cursor thích ứng | Oracle 11g+: cache nhiều plan cho một câu lệnh theo khoảng độ chọn lọc. |
| Parameter sniffing | "Đánh hơi" tham số | Tương đương bind peeking trong SQL Server. |
| Query hint (`RECOMPILE`, `OPTIMIZE FOR`, `USE PLAN`) | Gợi ý truy vấn | Chỉ thị cho optimizer của SQL Server. |
| Selectivity | Độ chọn lọc | Tỷ lệ dòng thỏa điều kiện. |
| Histogram | Biểu đồ tần suất | Thống kê phân bố giá trị của cột. |
| Nondeterministic | Không tất định | Hành vi phụ thuộc vào yếu tố ngẫu nhiên/thứ tự (ví dụ lần chạy đầu tiên). |
| SQL injection | Tấn công chèn SQL | Kẻ tấn công chèn mã SQL qua giá trị literal ghép vào câu lệnh. |
| KISS principle | Nguyên tắc KISS | "Keep It Simple, Stupid" — giữ mọi thứ đơn giản. |
| Equation solver | Bộ giải phương trình | Thứ mà database không có — nên cột phải đứng riêng một vế. |
