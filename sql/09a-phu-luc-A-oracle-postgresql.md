# Phụ lục A: Execution Plan – Kế hoạch thực thi (Appendix A – Execution Plans) — Phần 1

> Nguồn: *SQL Performance Explained* – Markus Winand, Phụ lục A, trang PDF 175–189 (trang sách 165–179). Phần 1 gồm: phần mở đầu phụ lục, Oracle Database và PostgreSQL. (SQL Server và MySQL nằm ở Phần 2.)

## Mục lục

- [Phụ lục A: Execution Plan (Appendix A – Execution Plans)](#phụ-lục-a-execution-plan--kế-hoạch-thực-thi-appendix-a--execution-plans--phần-1)
  - [Nội dung của phụ lục (Contents)](#nội-dung-của-phụ-lục-contents)
- [Cơ sở dữ liệu Oracle (Oracle Database)](#cơ-sở-dữ-liệu-oracle-oracle-database)
  - [Lấy execution plan (Getting an Execution Plan) — Oracle](#lấy-execution-plan-getting-an-execution-plan--oracle)
    - [Tạo và lưu execution plan (Creating and Saving an Execution Plan)](#tạo-và-lưu-execution-plan-creating-and-saving-an-execution-plan)
    - [Hiển thị execution plan (Showing Execution Plans)](#hiển-thị-execution-plan-showing-execution-plans)
  - [Các thao tác (Operations) — Oracle](#các-thao-tác-operations--oracle)
    - [Truy cập index và bảng (Index and Table Access) — Oracle](#truy-cập-index-và-bảng-index-and-table-access--oracle)
    - [Phép join (Joins) — Oracle](#phép-join-joins--oracle)
    - [Sắp xếp và gom nhóm (Sorting and Grouping) — Oracle](#sắp-xếp-và-gom-nhóm-sorting-and-grouping--oracle)
    - [Truy vấn Top-N (Top-N Queries) — Oracle](#truy-vấn-top-n-top-n-queries--oracle)
  - [Phân biệt access predicate và filter predicate (Distinguishing Access and Filter-Predicates) — Oracle](#phân-biệt-access-predicate-và-filter-predicate-distinguishing-access-and-filter-predicates--oracle)
- [PostgreSQL (PostgreSQL)](#postgresql-postgresql)
  - [Lấy execution plan (Getting an Execution Plan) — PostgreSQL](#lấy-execution-plan-getting-an-execution-plan--postgresql)
  - [Các thao tác (Operations) — PostgreSQL](#các-thao-tác-operations--postgresql)
    - [Truy cập index và bảng (Index and Table Access) — PostgreSQL](#truy-cập-index-và-bảng-index-and-table-access--postgresql)
    - [Các thao tác join (Join Operations) — PostgreSQL](#các-thao-tác-join-join-operations--postgresql)
    - [Sắp xếp và gom nhóm (Sorting and Grouping) — PostgreSQL](#sắp-xếp-và-gom-nhóm-sorting-and-grouping--postgresql)
    - [Truy vấn Top-N (Top-N Queries) — PostgreSQL](#truy-vấn-top-n-top-n-queries--postgresql)
  - [Phân biệt access predicate và filter predicate (Distinguishing Access and Filter-Predicates) — PostgreSQL](#phân-biệt-access-predicate-và-filter-predicate-distinguishing-access-and-filter-predicates--postgresql)
- [Bảng thuật ngữ (Glossary)](#bảng-thuật-ngữ-glossary)

---

<!-- [PDF page 175] -->

Trước khi cơ sở dữ liệu có thể thực thi một câu lệnh SQL, **optimizer** (*bộ tối ưu hóa truy vấn* — thành phần của DBMS quyết định *cách* thực thi câu lệnh) phải tạo ra một **execution plan** (*kế hoạch thực thi* — chuỗi các bước cụ thể mà database sẽ làm để trả về kết quả) cho câu lệnh đó. Sau đó database sẽ thực thi kế hoạch này theo từng bước một. Ở khía cạnh này, optimizer rất giống một **trình biên dịch** (*compiler*): nó dịch mã nguồn (câu lệnh SQL) thành một chương trình có thể thực thi được (execution plan).

Execution plan là **nơi đầu tiên cần nhìn vào** khi đi tìm nguyên nhân khiến một câu lệnh chạy chậm. Các mục dưới đây giải thích cách lấy ra và cách đọc execution plan để tối ưu hiệu năng trên nhiều hệ cơ sở dữ liệu khác nhau.

> 📝 **Giải thích thêm (không có trong sách):** Hãy hình dung SQL là câu "tôi muốn *cái gì*" (khai báo – *declarative*), còn execution plan là câu trả lời của database cho "tôi sẽ làm *thế nào*" (thủ tục – *procedural*): đọc index nào, theo thứ tự nào, join bằng thuật toán gì, sắp xếp ở đâu. Cùng một câu SQL có thể có nhiều plan khác nhau tùy vào index đang có, thống kê (*statistics*) của bảng và giá trị của bind parameter. Vì thế, khi một truy vấn chậm, đừng đoán — hãy xem plan.

### Nội dung của phụ lục (Contents)

Bảng nội dung gốc của Phụ lục A (số trang là số trang in trong sách):

| Mục | Trang |
|---|---|
| **Oracle Database** | 166 |
| — Getting an Execution Plan (Lấy execution plan) | 166 |
| — Operations (Các thao tác) | 167 |
| — Distinguishing Access and Filter-Predicates (Phân biệt access và filter predicate) | 170 |
| **PostgreSQL** | 172 |
| — Getting an Execution Plan | 172 |
| — Operations | 174 |
| — Distinguishing Access and Filter-Predicates | 177 |
| **SQL Server** | 180 |
| — Getting an Execution Plan | 180 |
| — Operations | 182 |
| — Distinguishing Access and Filter-Predicates | 185 |
| **MySQL** | 188 |
| — Getting an Execution Plan | 188 |
| — Operations | 188 |
| — Distinguishing Access and Filter-Predicates | 190 |

*(Tài liệu này — Phần 1 — trình bày Oracle Database và PostgreSQL; SQL Server và MySQL nằm trong Phần 2.)*

---

<!-- [PDF page 176] -->

## Cơ sở dữ liệu Oracle (Oracle Database)

Hầu hết các môi trường phát triển (*IDE*) đều có thể hiển thị execution plan rất dễ dàng, nhưng mỗi công cụ lại định dạng plan trên màn hình theo những cách rất khác nhau. Phương pháp được mô tả trong mục này cho ra execution plan **đúng như dạng được trình bày xuyên suốt cuốn sách**, và chỉ yêu cầu Oracle Database phiên bản **9iR2 trở lên**.

### Lấy execution plan (Getting an Execution Plan) — Oracle

Việc xem một execution plan trong Oracle Database gồm hai bước:

1. `explain plan for` — lưu execution plan vào bảng `PLAN_TABLE`.
2. Định dạng và hiển thị execution plan.

#### Tạo và lưu execution plan (Creating and Saving an Execution Plan)

Để tạo một execution plan, bạn chỉ cần đặt tiền tố `explain plan for` trước câu lệnh SQL tương ứng:

```sql
EXPLAIN PLAN FOR select * from dual;
```

Bạn có thể chạy lệnh `explain plan for` trong bất kỳ môi trường phát triển nào hoặc trong SQL*Plus. Tuy nhiên, lệnh này **không hiển thị** plan mà **lưu** nó vào một bảng tên là `PLAN_TABLE`. Kể từ phiên bản **10g**, bảng này tự động có sẵn dưới dạng một **bảng tạm toàn cục** (*global temporary table* — bảng có định nghĩa dùng chung nhưng dữ liệu riêng cho từng session). Với các phiên bản trước đó, bạn phải tự tạo bảng này trong từng **schema** khi cần. Hãy nhờ quản trị viên cơ sở dữ liệu (DBA) tạo giúp, hoặc xin họ câu lệnh `create table` có sẵn trong bộ cài đặt Oracle Database:

```text
$ORACLE_HOME/rdbms/admin/utlxplan.sql
```

Bạn có thể chạy câu lệnh này trong bất kỳ schema nào bạn muốn để tạo `PLAN_TABLE` trong schema đó.

> ⚠️ **Cảnh báo (Warning)**
>
> Lệnh `explain plan for` **không nhất thiết** tạo ra cùng một execution plan như khi câu lệnh thực sự được thực thi.

> 📝 **Giải thích thêm (không có trong sách):** Một lý do điển hình: `EXPLAIN PLAN` không "nhìn" giá trị thật của bind parameter (không có *bind peeking*) và luôn coi bind là kiểu `VARCHAR2`, nên có thể chọn plan khác với lúc chạy thật. Từ Oracle 10g trở đi, để xem plan **thực sự** đã được dùng, có thể chạy câu lệnh rồi gọi `DBMS_XPLAN.DISPLAY_CURSOR`:
>
> ```sql
> -- Oracle 10g+
> select /*+ gather_plan_statistics */ count(*) from employees where subsidiary_id = :x;
> select * from table(dbms_xplan.display_cursor(null, null, 'ALLSTATS LAST'));
> ```
>
> `ALLSTATS LAST` còn hiển thị số dòng thực tế (`A-Rows`) bên cạnh số dòng ước lượng (`E-Rows`) — tương tự như `EXPLAIN ANALYZE` của PostgreSQL.

<!-- [PDF page 177] -->

#### Hiển thị execution plan (Showing Execution Plans)

Package `DBMS_XPLAN` được giới thiệu từ phiên bản 9iR2, có khả năng định dạng và hiển thị các execution plan lấy từ `PLAN_TABLE`. Ví dụ sau cho thấy cách hiển thị execution plan **gần nhất** đã được explain trong session database hiện tại:

```sql
select * from table(dbms_xplan.display);
```

Một lần nữa, nếu câu lệnh này không chạy được ngay, bạn nên nhờ DBA hỗ trợ.

Truy vấn trên sẽ hiển thị execution plan giống như trong sách:

```text
--------------------------------------------------------------
| Id | Operation         | Name | Rows | Bytes | Cost (%CPU)|.
--------------------------------------------------------------
|  0 | SELECT STATEMENT  |      |    1 |     2 |     2   (0)|.
|  1 |  TABLE ACCESS FULL| DUAL |    1 |     2 |     2   (0)|.
--------------------------------------------------------------
```

Một số cột hiển thị trong execution plan này đã bị lược bỏ trong sách để vừa khổ trang hơn (dấu chấm ở cuối mỗi dòng cho biết bảng còn các cột tiếp theo).

> 📝 **Giải thích thêm (không có trong sách):** Ý nghĩa các cột:
>
> | Cột | Ý nghĩa |
> |---|---|
> | `Id` | Số thứ tự của bước trong plan; dùng để tham chiếu từ phần *Predicate Information*. |
> | `Operation` | Tên thao tác. Thụt lề thể hiện quan hệ cha–con: thao tác thụt sâu hơn cung cấp dữ liệu cho thao tác ngay phía trên nó. |
> | `Name` | Đối tượng được truy cập (bảng, index). |
> | `Rows` | Số dòng ước lượng mà bước đó trả về (*cardinality estimate*). |
> | `Bytes` | Dung lượng dữ liệu ước lượng. |
> | `Cost (%CPU)` | Chi phí ước lượng của optimizer (đơn vị tương đối), và phần trăm chi phí dành cho CPU. |
>
> Các cột bị lược trong sách thường là `Time` (thời gian ước lượng) và, với truy vấn song song/phân vùng, các cột như `Pstart`/`Pstop`, `TQ`...

> 💡 **Ví dụ bổ sung (không có trong sách):** Quy trình đầy đủ trên Oracle với bảng mẫu `employees`:
>
> ```sql
> CREATE TABLE employees (
>   employee_id   NUMBER NOT NULL,
>   subsidiary_id NUMBER NOT NULL,
>   last_name     VARCHAR2(100),
>   CONSTRAINT employees_pk PRIMARY KEY (employee_id, subsidiary_id)
> );
>
> EXPLAIN PLAN FOR
>   SELECT last_name FROM employees
>    WHERE employee_id = 123 AND subsidiary_id = 30;
>
> SELECT * FROM TABLE(dbms_xplan.display);
> ```
>
> Plan mong đợi sẽ có dạng (rút gọn):
>
> ```text
> --------------------------------------------------------------
> | Id | Operation                   | Name         | Rows | Cost |
> --------------------------------------------------------------
> |  0 | SELECT STATEMENT            |              |    1 |    2 |
> |  1 |  TABLE ACCESS BY INDEX ROWID| EMPLOYEES    |    1 |    2 |
> |* 2 |   INDEX UNIQUE SCAN         | EMPLOYEES_PK |    1 |    1 |
> --------------------------------------------------------------
> ```
>
> Đọc từ trong ra ngoài: bước 2 (thụt sâu nhất) chạy trước — duyệt B-tree của khóa chính để tìm đúng một ROWID; bước 1 dùng ROWID đó đọc dòng trong bảng; bước 0 trả kết quả cho client. Vì điều kiện bao phủ toàn bộ khóa chính (unique), Oracle dùng `INDEX UNIQUE SCAN`.


### Các thao tác (Operations) — Oracle

#### Truy cập index và bảng (Index and Table Access) — Oracle

**INDEX UNIQUE SCAN**

`INDEX UNIQUE SCAN` chỉ thực hiện việc **duyệt cây B-tree** (*B-tree traversal* — đi từ nút gốc qua các nút nhánh xuống tới nút lá). Database dùng thao tác này khi có một **ràng buộc duy nhất** (*unique constraint*) đảm bảo rằng điều kiện tìm kiếm sẽ khớp với **không quá một** mục (entry). Xem thêm Chương 1, "Giải phẫu một index" (*Anatomy of an Index*).

**INDEX RANGE SCAN**

`INDEX RANGE SCAN` thực hiện việc duyệt B-tree **và** đi theo **chuỗi nút lá** (*leaf node chain* — danh sách liên kết hai chiều nối các nút lá của index) để tìm tất cả các mục khớp điều kiện. Xem thêm Chương 1, "Giải phẫu một index" (*Anatomy of an Index*).

Những thứ được gọi là **index filter predicate** (*vị từ lọc trên index* — điều kiện chỉ được kiểm tra trong lúc đi dọc nút lá, không giúp thu hẹp phạm vi quét) thường gây ra vấn đề hiệu năng cho `INDEX RANGE SCAN`. Mục tiếp theo giải thích cách nhận diện chúng.

<!-- [PDF page 178] -->

**INDEX FULL SCAN**

Đọc **toàn bộ** index — tất cả các dòng — **theo thứ tự của index**. Tùy vào nhiều thống kê hệ thống (*system statistics*) khác nhau, database có thể thực hiện thao tác này nếu nó cần tất cả các dòng theo thứ tự index — ví dụ do có mệnh đề `order by` tương ứng. Thay vào đó, optimizer cũng có thể dùng `INDEX FAST FULL SCAN` rồi thực hiện thêm một thao tác sắp xếp. Xem Chương 6, "Sắp xếp và gom nhóm" (*Sorting and Grouping*).

**INDEX FAST FULL SCAN**

Đọc **toàn bộ** index — tất cả các dòng — **theo thứ tự được lưu trên đĩa** (không theo thứ tự logic của index). Thao tác này thường được thực hiện **thay cho full table scan** nếu tất cả các cột cần thiết đều có sẵn trong index. Tương tự `TABLE ACCESS FULL`, `INDEX FAST FULL SCAN` có thể tận dụng các thao tác **đọc nhiều block một lúc** (*multi-block read*). Xem Chương 5, "Gom cụm dữ liệu" (*Clustering Data*).

**TABLE ACCESS BY INDEX ROWID**

Lấy một dòng từ bảng bằng **ROWID** (*địa chỉ vật lý của dòng trong bảng*) thu được từ bước tra cứu index ngay trước đó. Xem thêm Chương 1, "Giải phẫu một index" (*Anatomy of an Index*).

**TABLE ACCESS FULL**

Thao tác này còn được gọi là **full table scan** (*quét toàn bộ bảng*). Nó đọc **toàn bộ bảng** — tất cả các dòng và cột — theo đúng cách lưu trên đĩa. Mặc dù đọc nhiều block một lúc giúp tăng tốc full table scan đáng kể, nó vẫn là **một trong những thao tác tốn kém nhất**. Ngoài lượng IO cao, full table scan còn phải kiểm tra mọi dòng của bảng nên có thể tiêu tốn một lượng thời gian CPU đáng kể. Xem thêm "Full Table Scan" ở trang 13.

> 📝 **Giải thích thêm (không có trong sách):** So sánh nhanh bốn cách "đọc rộng" trong Oracle:
>
> | Thao tác | Đọc gì | Thứ tự kết quả | Kiểu đọc đĩa |
> |---|---|---|---|
> | `INDEX RANGE SCAN` | Một khoảng nút lá của index | Theo thứ tự index | Từng block (single-block) |
> | `INDEX FULL SCAN` | Toàn bộ nút lá của index | Theo thứ tự index (dùng được cho `ORDER BY`) | Từng block |
> | `INDEX FAST FULL SCAN` | Toàn bộ block của index | Không có thứ tự ý nghĩa | Nhiều block một lúc |
> | `TABLE ACCESS FULL` | Toàn bộ block của bảng | Không có thứ tự ý nghĩa | Nhiều block một lúc |

> 💡 **Ví dụ bổ sung (không có trong sách):** Với bảng `employees` có index `emp_name ON employees(last_name)` và `last_name` được khai báo `NOT NULL`:
>
> ```sql
> -- (1) Đếm số dòng: chỉ cần index, không cần thứ tự
> SELECT COUNT(*) FROM employees;
> --   -> thường thấy INDEX FAST FULL SCAN trên EMP_NAME (index nhỏ hơn bảng,
> --      đọc multi-block được)
>
> -- (2) Lấy tất cả tên theo thứ tự
> SELECT last_name FROM employees ORDER BY last_name;
> --   -> có thể là INDEX FULL SCAN (trả về sẵn theo thứ tự, không cần SORT)
> --      hoặc INDEX FAST FULL SCAN + SORT ORDER BY, tùy chi phí ước lượng
>
> -- (3) Cần cột không có trong index, không có điều kiện lọc
> SELECT * FROM employees;
> --   -> TABLE ACCESS FULL
> ```
>
> Lưu ý: nếu `last_name` cho phép `NULL`, Oracle không thể dùng index B-tree một cột để đếm `COUNT(*)` vì các dòng có `last_name IS NULL` không được lưu trong index đó.

#### Phép join (Joins) — Oracle

Nhìn chung, các thao tác join chỉ xử lý **hai bảng một lúc**. Nếu một truy vấn có nhiều phép join hơn, chúng được thực hiện **tuần tự**: đầu tiên là hai bảng, sau đó là kết quả trung gian với bảng tiếp theo. Vì vậy, trong ngữ cảnh join, từ "bảng" cũng có thể mang nghĩa "**kết quả trung gian**" (*intermediate result*).

**NESTED LOOPS JOIN**

Join hai bảng bằng cách lấy kết quả từ một bảng, rồi **với mỗi dòng** từ bảng thứ nhất, truy vấn bảng còn lại. Xem thêm "Nested Loops" ở trang 92.

<!-- [PDF page 179] -->

**HASH JOIN**

Hash join nạp các bản ghi ứng viên từ **một phía** của phép join vào một **bảng băm** (*hash table*), sau đó bảng băm này được **dò tìm** (*probe*) cho mỗi dòng từ phía bên kia của phép join. Xem thêm "Hash Join" ở trang 101.

**MERGE JOIN**

Merge join kết hợp hai danh sách đã sắp xếp **giống như khóa kéo** (*zipper* — hai hàng răng cài vào nhau lần lượt từng chiếc). Cả hai phía của phép join đều phải được **sắp xếp trước**. Xem thêm "Sort Merge" ở trang 109.

> 💡 **Ví dụ bổ sung (không có trong sách):** Hai bảng `employees` (1.000 dòng) và `sales` (1.000.000 dòng), truy vấn:
>
> ```sql
> SELECT e.last_name, s.sale_date, s.eur_value
>   FROM employees e
>   JOIN sales s ON s.employee_id = e.employee_id
>  WHERE e.last_name = 'WINAND';
> ```
>
> - Nếu có index trên `employees(last_name)` **và** `sales(employee_id)`: điều kiện lọc trả về rất ít nhân viên → optimizer thường chọn **NESTED LOOPS**: với mỗi nhân viên tên WINAND, dùng index trên `sales.employee_id` để tìm các dòng bán hàng.
> - Nếu không có index trên `sales(employee_id)`: mỗi vòng lặp sẽ phải quét toàn bộ `sales` → optimizer chuyển sang **HASH JOIN**: nạp các nhân viên tên WINAND (phía nhỏ) vào hash table, rồi quét `sales` một lần và dò hash table cho từng dòng.
> - **MERGE JOIN** thường xuất hiện khi cả hai phía đã có sẵn thứ tự theo cột join (ví dụ đọc qua index) hoặc với điều kiện join không phải bằng (`<`, `>`), nơi hash join không áp dụng được.

#### Sắp xếp và gom nhóm (Sorting and Grouping) — Oracle

**SORT ORDER BY**

Sắp xếp kết quả theo mệnh đề `order by`. Thao tác này cần **lượng bộ nhớ lớn** để **vật chất hóa** (*materialize* — lưu toàn bộ lại trước khi trả ra) kết quả trung gian (**không pipelined** — tức là không thể trả dòng đầu tiên trước khi đã đọc xong toàn bộ đầu vào). Xem thêm "Indexing Order By" ở trang 130.

**SORT ORDER BY STOPKEY**

Sắp xếp **một tập con** của kết quả theo mệnh đề `order by`. Được dùng cho **truy vấn top-N** nếu không thể thực thi theo kiểu pipelined. Xem thêm "Querying Top-N Rows" ở trang 143.

**SORT GROUP BY**

Sắp xếp tập kết quả theo các cột trong `group by`, rồi ở bước thứ hai tổng hợp (*aggregate*) kết quả đã sắp xếp. Thao tác này cần lượng bộ nhớ lớn để vật chất hóa tập kết quả trung gian (không pipelined). Xem thêm "Indexing Group By" ở trang 139.

**SORT GROUP BY NOSORT**

Tổng hợp một tập dữ liệu **đã được sắp xếp sẵn** theo mệnh đề `group by`. Thao tác này **không đệm** (*buffer*) kết quả trung gian: nó được thực thi theo kiểu **pipelined**. Xem thêm "Indexing Group By" ở trang 139.

**HASH GROUP BY**

Gom nhóm kết quả bằng một bảng băm. Thao tác này cần lượng bộ nhớ lớn để vật chất hóa tập kết quả trung gian (không pipelined). Kết quả đầu ra **không có thứ tự nào có ý nghĩa**. Xem thêm "Indexing Group By" ở trang 139.

> 📝 **Giải thích thêm (không có trong sách):** "Pipelined" nghĩa là mỗi dòng có thể được chuyển ngay cho bước kế tiếp mà không cần chờ đọc hết dữ liệu — giống một dây chuyền sản xuất. Các thao tác có chữ `NOSORT` (như `SORT GROUP BY NOSORT`) cho biết Oracle đã tận dụng được thứ tự sẵn có của index nên **không thực sự sắp xếp** dù tên thao tác vẫn có chữ "SORT".

> 💡 **Ví dụ bổ sung (không có trong sách):** Bảng `sales(sale_date DATE, product_id NUMBER, eur_value NUMBER)` có index `sales_dt_pr ON sales(sale_date, product_id)`:
>
> ```sql
> SELECT sale_date, product_id, SUM(eur_value)
>   FROM sales
>  WHERE sale_date = TRUNC(sysdate) - 1
>  GROUP BY sale_date, product_id;
> ```
>
> Index đã trả các dòng theo thứ tự `(sale_date, product_id)` → Oracle có thể dùng `SORT GROUP BY NOSORT` (pipelined, ít bộ nhớ) — dù optimizer vẫn có thể chọn `HASH GROUP BY` nếu ước lượng thấy rẻ hơn. Còn nếu `GROUP BY` theo một cột không có trong index, ví dụ `customer_id`, dữ liệu đọc ra không có sẵn thứ tự phù hợp, nên optimizer bắt buộc phải dùng `HASH GROUP BY` hoặc `SORT GROUP BY` (không pipelined).

<!-- [PDF page 180] -->

#### Truy vấn Top-N (Top-N Queries) — Oracle

Hiệu quả của truy vấn top-N phụ thuộc vào **chế độ thực thi** của các thao tác bên dưới. Chúng **rất kém hiệu quả** khi phải dừng giữa chừng các thao tác **không pipelined** như `SORT ORDER BY` (vì thao tác đó đã phải đọc và sắp xếp hết dữ liệu trước rồi).

**COUNT STOPKEY**

Dừng (*abort*) các thao tác bên dưới khi đã lấy đủ số dòng mong muốn. Xem thêm mục "Querying Top-N Rows".

**WINDOW NOSORT STOPKEY**

Dùng một **hàm cửa sổ** (*window function* — mệnh đề `over`) để dừng việc thực thi khi đã lấy đủ số dòng mong muốn. Xem thêm "Using Window Functions for Pagination" ở trang 156.

> 💡 **Ví dụ bổ sung (không có trong sách):** Lấy 10 đơn hàng mới nhất, với index `sales_dt ON sales(sale_date)`:
>
> ```sql
> -- Oracle 11g trở về trước
> SELECT * FROM (
>   SELECT * FROM sales ORDER BY sale_date DESC
> ) WHERE rownum <= 10;
>
> -- Oracle 12c+ (cú pháp chuẩn SQL:2008)
> SELECT * FROM sales ORDER BY sale_date DESC
> FETCH FIRST 10 ROWS ONLY;
> ```
>
> - Có index trên `sale_date`: plan có `COUNT STOPKEY` phía trên `INDEX FULL SCAN DESCENDING` + `TABLE ACCESS BY INDEX ROWID` → chỉ đọc khoảng 10 mục index rồi dừng (pipelined).
> - Không có index: plan là `COUNT STOPKEY` phía trên `SORT ORDER BY STOPKEY` phía trên `TABLE ACCESS FULL` → vẫn phải đọc cả bảng, chỉ tiết kiệm được bộ nhớ khi sắp xếp.
>
> Ở Oracle 12c+, `FETCH FIRST` được viết lại nội bộ bằng `ROW_NUMBER()`, nên plan thường hiện `WINDOW NOSORT STOPKEY` (có index) hoặc `WINDOW SORT PUSHED RANK` (không có index).

### Phân biệt access predicate và filter predicate (Distinguishing Access and Filter-Predicates) — Oracle

Oracle Database dùng **ba phương pháp khác nhau** để áp dụng mệnh đề `where` (các **vị từ** – *predicates*, tức các điều kiện):

**Access predicate – vị từ truy cập ("access")**
: Access predicate biểu diễn **điều kiện bắt đầu và điều kiện dừng** của việc duyệt nút lá (*leaf node traversal*).

**Index filter predicate – vị từ lọc trên index ("filter" cho các thao tác index)**
: Index filter predicate chỉ được áp dụng **trong lúc** duyệt nút lá. Chúng **không** góp phần vào điều kiện bắt đầu/dừng và **không** thu hẹp phạm vi được quét.

**Table level filter predicate – vị từ lọc ở mức bảng ("filter" cho các thao tác trên bảng)**
: Các vị từ trên những cột **không thuộc index** được đánh giá ở mức bảng. Để làm được điều đó, database phải **nạp dòng từ bảng** trước.

<!-- [PDF page 181] -->

Các execution plan được tạo bằng tiện ích `DBMS_XPLAN` (xem "Getting an Execution Plan" ở trang 166) hiển thị cách sử dụng index trong phần **"Predicate Information"** nằm bên dưới bảng execution plan:

```text
------------------------------------------------------
| Id | Operation         | Name       | Rows  | Cost |
------------------------------------------------------
|  0 | SELECT STATEMENT  |            |     1 | 1445 |
|  1 |  SORT AGGREGATE   |            |     1 |      |
|* 2 |   INDEX RANGE SCAN| SCALE_SLOW |  4485 | 1445 |
------------------------------------------------------

Predicate Information (identified by operation id):
   2 - access("SECTION"=:A AND "ID2"=:B)
       filter("ID2"=:B)
```

Số thứ tự trong phần predicate information tham chiếu tới cột **"Id"** của execution plan. Ở cột đó, database cũng hiển thị một **dấu sao** (`*`) để đánh dấu những thao tác có predicate information.

Ví dụ này, lấy từ chương "Hiệu năng và khả năng mở rộng" (*Performance and Scalability*), cho thấy một `INDEX RANGE SCAN` có **cả** access predicate **lẫn** filter predicate. Oracle Database có một **điểm kỳ quặc**: nó cũng hiển thị một số filter predicate dưới dạng access predicate — ví dụ `ID2=:B` trong execution plan ở trên.

> ❗ **Quan trọng (Important)**
>
> Nếu một điều kiện xuất hiện dưới dạng **filter predicate**, thì nó **là** filter predicate — việc nó **cũng** được hiển thị như access predicate **không quan trọng**.

Điều này có nghĩa là `INDEX RANGE SCAN` quét **toàn bộ phạm vi** thỏa điều kiện `"SECTION"=:A` và áp dụng bộ lọc `"ID2"=:B` lên **từng dòng**.

Các filter predicate ở mức bảng được hiển thị tại thao tác truy cập bảng tương ứng, chẳng hạn `TABLE ACCESS BY INDEX ROWID` hoặc `TABLE ACCESS FULL`.

> 📝 **Giải thích thêm (không có trong sách):** Tại sao `ID2` chỉ là filter? Index `SCALE_SLOW` được định nghĩa trên `(SECTION, ID1, ID2)`. Truy vấn có điều kiện trên `SECTION` và `ID2` nhưng **không có** điều kiện trên `ID1`. Trong B-tree, các mục được sắp xếp trước theo `SECTION`, rồi `ID1`, rồi mới tới `ID2`; trong cùng một `SECTION`, các giá trị `ID2` nằm rải rác theo `ID1`. Vì vậy database chỉ có thể dùng `SECTION` để xác định điểm bắt đầu/kết thúc quét, còn `ID2` phải được kiểm tra trên từng mục lá — đúng định nghĩa của index filter predicate.

> 💡 **Ví dụ bổ sung (không có trong sách):** Bài toán: làm sao để `ID2` trở thành access predicate thật sự?
>
> ```sql
> -- Cách 1: đổi thứ tự cột, đưa ID2 lên trước ID1
> CREATE INDEX scale_fast ON scale_data (section, id2, id1);
> ```
>
> Plan với index mới (dạng mong đợi):
>
> ```text
> Predicate Information (identified by operation id):
>    2 - access("SECTION"=:A AND "ID2"=:B)
> ```
>
> Lúc này **không còn** dòng `filter(...)` → cả hai điều kiện đều là access predicate, phạm vi quét thu hẹp chỉ còn những mục thỏa đồng thời `SECTION` và `ID2`. Quy tắc chung: cột so sánh bằng (`=`) nên đứng trước, cột so sánh khoảng hoặc cột không được lọc nên đứng sau trong index nhiều cột.
>
> Một ví dụ về filter **ở mức bảng**:
>
> ```text
> |* 1 |  TABLE ACCESS BY INDEX ROWID| EMPLOYEES   |
> |* 2 |   INDEX RANGE SCAN          | EMP_SUB_ID  |
>
> Predicate Information (identified by operation id):
>    1 - filter("LAST_NAME"='WINAND')
>    2 - access("SUBSIDIARY_ID"=30)
> ```
>
> Ở đây `LAST_NAME` không có trong index `EMP_SUB_ID`, nên Oracle phải đọc từng dòng của bảng (bước 1) rồi mới lọc được.

---

<!-- [PDF page 182] -->

## PostgreSQL (PostgreSQL)

Các phương pháp mô tả trong mục này áp dụng cho **PostgreSQL 8.0 trở về sau**.

### Lấy execution plan (Getting an Execution Plan) — PostgreSQL

Execution plan của PostgreSQL được lấy ra bằng cách đặt lệnh `explain` phía trước câu lệnh SQL. Tuy nhiên có **một hạn chế quan trọng**: các câu lệnh SQL có **bind parameter** (*tham số ràng buộc* — chỗ giữ giá trị như `$1`, `$2`, v.v. được truyền vào lúc thực thi) **không thể** được explain trực tiếp theo cách này — chúng cần được **prepare** (*chuẩn bị*) trước:

```sql
PREPARE stmt(int) AS SELECT $1;
```

Lưu ý rằng PostgreSQL dùng `"$n"` cho bind parameter. Tầng trừu tượng cơ sở dữ liệu (*database abstraction layer*) của bạn có thể che giấu điều này để bạn dùng dấu chấm hỏi (`?`) như quy định trong chuẩn SQL.

Việc thực thi prepared statement thì có thể được explain:

```sql
EXPLAIN EXECUTE stmt(1);
```

Cho tới PostgreSQL **9.1**, execution plan đã được tạo ngay ở lời gọi `prepare`, và do đó **không thể** xét đến các giá trị thực tế được cung cấp khi `execute`. Kể từ PostgreSQL **9.2**, việc tạo execution plan được **hoãn lại cho tới lúc thực thi**, và nhờ vậy có thể xét đến giá trị thực tế của các bind parameter.

> 📌 **Ghi chú (Note)**
>
> Các câu lệnh không có bind parameter có thể được explain trực tiếp:
>
> ```sql
> EXPLAIN SELECT 1;
> ```
>
> Trong trường hợp này, optimizer **luôn luôn** xét đến các giá trị thực tế trong quá trình lập kế hoạch truy vấn. Nếu bạn dùng PostgreSQL 9.1 trở về trước và dùng bind parameter trong chương trình, bạn cũng nên dùng `explain` **với bind parameter** để lấy được đúng execution plan giống như chương trình.

> 📝 **Giải thích thêm (không có trong sách):** Từ 9.2 trở đi, PostgreSQL có cơ chế *custom plan* vs. *generic plan*: 5 lần thực thi đầu của một prepared statement luôn được lập kế hoạch riêng theo giá trị thực (custom plan); sau đó nếu generic plan (không phụ thuộc giá trị) không đắt hơn đáng kể, PostgreSQL sẽ chuyển sang dùng lại generic plan. Từ **PostgreSQL 12**, tham số `plan_cache_mode` (`auto` / `force_custom_plan` / `force_generic_plan`) cho phép điều khiển hành vi này. Ngoài ra, từ **PostgreSQL 16** có thể xem generic plan trực tiếp mà không cần `PREPARE`: `EXPLAIN (GENERIC_PLAN) SELECT ... WHERE id = $1;`.

<!-- [PDF page 183] -->

Kết quả của lệnh explain như sau:

```text
                QUERY PLAN
------------------------------------------
 Result  (cost=0.00..0.01 rows=1 width=0)
```

Kết quả này có thông tin tương tự các execution plan của Oracle được trình bày xuyên suốt cuốn sách: tên thao tác (`"Result"`), chi phí (*cost*) liên quan, số dòng ước lượng (`rows`), và độ rộng dòng dự kiến (`width`, tính bằng byte).

Lưu ý rằng PostgreSQL hiển thị **hai giá trị cost**. Giá trị thứ nhất là chi phí **khởi động** (*startup* — chi phí trước khi trả được dòng đầu tiên), giá trị thứ hai là **tổng chi phí** thực thi nếu lấy hết tất cả các dòng. Execution plan của Oracle Database chỉ hiển thị giá trị thứ hai.

Lệnh `explain` của PostgreSQL có hai tùy chọn. Tùy chọn **`VERBOSE`** cung cấp thông tin bổ sung như tên bảng đầy đủ (*fully qualified* — kèm tên schema) — `VERBOSE` thường **không có giá trị lắm**.

Tùy chọn explain thứ hai là **`ANALYZE`**. Mặc dù được dùng rộng rãi, tôi (tác giả) **khuyên bạn đừng tạo thói quen dùng nó một cách tự động**, bởi vì nó **thực sự thực thi câu lệnh**. Điều đó phần lớn là vô hại với câu lệnh `select`, nhưng nó sẽ **thay đổi dữ liệu** của bạn khi dùng cho `insert`, `update` hoặc `delete`. Để tránh rủi ro vô tình sửa dữ liệu, bạn có thể bọc nó trong một **transaction** (*giao dịch*) và thực hiện `rollback` sau đó.

Tùy chọn `ANALYZE` thực thi câu lệnh và ghi lại **thời gian thực tế** cùng **số dòng thực tế**. Điều này rất có giá trị khi đi tìm nguyên nhân của những **ước lượng cardinality** (*cardinality estimate* — ước lượng số dòng) bị sai:

```sql
BEGIN;
EXPLAIN ANALYZE EXECUTE stmt(1);
```

```text
                   QUERY PLAN
--------------------------------------------------
 Result  (cost=0.00..0.01 rows=1 width=0)
         (actual time=0.002..0.002 rows=1 loops=1)
 Total runtime: 0.020 ms
```

```sql
ROLLBACK;
```

Lưu ý rằng plan ở trên đã được định dạng lại cho vừa trang sách. PostgreSQL thực tế in các giá trị "actual" (thực tế) **trên cùng một dòng** với các giá trị ước lượng.

<!-- [PDF page 184] -->

> ⚠️ **Cảnh báo (Warning)**
>
> `explain analyze` **thực thi** câu lệnh được explain, **kể cả khi** câu lệnh đó là `insert`, `update` hoặc `delete`.

Số dòng (`rows`) là **giá trị duy nhất** được hiển thị ở **cả hai phần** — phần ước lượng và phần thực tế. Điều đó cho phép bạn nhanh chóng phát hiện các ước lượng cardinality bị sai.

Cuối cùng nhưng không kém phần quan trọng, các prepared statement phải được **đóng lại**:

```sql
DEALLOCATE stmt;
```

> 💡 **Ví dụ bổ sung (không có trong sách):** Phát hiện ước lượng sai bằng `EXPLAIN ANALYZE` (PostgreSQL). Giả sử bảng `orders` có 1 triệu dòng, cột `status` có index, và thống kê đã cũ (chưa chạy `ANALYZE orders` sau khi nạp dữ liệu):
>
> ```sql
> BEGIN;
> EXPLAIN ANALYZE SELECT * FROM orders WHERE status = 'PENDING';
> ROLLBACK;
> ```
>
> ```text
>  Index Scan using orders_status on orders
>    (cost=0.42..8.44 rows=1 width=64)
>    (actual time=0.031..412.770 rows=250000 loops=1)
>    Index Cond: (status = 'PENDING'::text)
> ```
>
> So sánh `rows=1` (ước lượng) với `rows=250000` (thực tế): sai lệch rất lớn → optimizer chọn Index Scan vì tưởng chỉ có 1 dòng, trong khi Seq Scan hoặc Bitmap Heap Scan có thể nhanh hơn nhiều. Cách xử lý: chạy `ANALYZE orders;` để cập nhật thống kê rồi xem lại plan.
>
> Lưu ý thêm: từ PostgreSQL 9.4 dòng `Total runtime` được đổi thành `Planning time` và `Execution time`; từ PostgreSQL 9.0 có thể dùng cú pháp có ngoặc như `EXPLAIN (ANALYZE, BUFFERS) ...` để xem thêm số block đọc từ cache/đĩa.

> 📝 **Giải thích thêm (không có trong sách) — Đối chiếu với PostgreSQL hiện đại (PG 10–17): Toàn cảnh các cờ `EXPLAIN` mới và Bộ đôi công cụ Production (`pg_stat_statements` & `auto_explain`):**
>
> Khi cuốn sách này ra đời năm 2012 (PostgreSQL 9.1/9.2), tác giả chỉ giới thiệu hai tùy chọn `VERBOSE` và `ANALYZE`, đồng thời phải dùng `PREPARE` / `EXECUTE` thủ công để xem plan của câu lệnh có tham số `$1`. Trong **PostgreSQL hiện đại (PG 10–17)**, lệnh `EXPLAIN (...)` đã được bổ sung hàng loạt cờ chẩn đoán chuyên sâu:
>
> | Cờ `EXPLAIN` | Phiên bản PG | Ý nghĩa & Ứng dụng thực chiến |
> |---|---|---|
> | **`BUFFERS`** | PG 9.0+ *(mở rộng ở **PG 13+**)* | Hiển thị số trang 8KB đọc từ bộ nhớ đệm (`shared hit`), đọc từ OS cache/ổ đĩa (`shared read`), bị làm bẩn (`dirtied`), ghi ra (`written`) và tràn ra đĩa tạm (`temp read/written`). **Từ PG 13+**, `BUFFERS` hiển thị cả thống kê buffer riêng cho bước **Planning** (rất hữu ích khi bảng có hàng trăm partition hoặc index bị phình). *Lưu ý:* Từ PG 10+, chỉ cần gõ `EXPLAIN (ANALYZE, BUFFERS)` (mặc định `BUFFERS` là `off` khi chỉ gọi `ANALYZE`). Khi bật thêm `track_io_timing = on`, plan sẽ in rõ thời gian chờ ổ đĩa `I/O Timings: shared read=... ms`. |
> | **`WAL`** | **PG 13+** | *(Dùng kèm `ANALYZE`)* Đo chính xác số lượng bản ghi nhật ký (`records`), số ảnh trang đầy đủ (`fpi` — *full page images*) và tổng số `bytes` WAL sinh ra ở từng nút thực thi. Cực kỳ quan trọng để tối ưu tải ghi `INSERT`/`UPDATE`/`DELETE` và phát hiện *HOT Update* hay *Hint Bits* gây ghi WAL ngầm. |
> | **`SETTINGS`** | **PG 12+** | Tự động in ra ở cuối plan mọi tham số cấu hình (`GUC`) đang **khác với mặc định** và có ảnh hưởng đến bộ lập kế hoạch (vd: `work_mem`, `random_page_cost`, `effective_cache_size`, `enable_seqscan`). Giúp giải quyết dứt điểm câu hỏi *"Tại sao cùng câu SQL mà môi trường Dev và Production lại ra 2 plan khác nhau?"*. |
> | **`GENERIC_PLAN`** | **PG 16+** | *(Không dùng kèm `ANALYZE`)* Cho phép chạy trực tiếp `EXPLAIN (GENERIC_PLAN) SELECT * FROM sales WHERE id = $1;` với các ký hiệu `$1, $2` mà **không cần `PREPARE stmt` / `DEALLOCATE stmt`** như cách thủ công thời PG 9.1/9.2 trong sách! |
> | **`SERIALIZE`** | **PG 17+** | *(Dùng kèm `ANALYZE`, nhận `NONE` / `TEXT` / `BINARY`)* Trước PG 17, `EXPLAIN ANALYZE` không chuyển đổi dữ liệu kết quả thành gói tin gửi về client, nên **bỏ sót chi phí giải nén dữ liệu `TOAST` lớn** (như cột `JSONB`, `TEXT`, `BYTEA` dài). Cờ `SERIALIZE` ở PG 17 đo chính xác thời gian CPU và dung lượng KB tuần tự hóa đầu ra (`Serialization: time=... ms  output=... kB  format=text`). |
> | **`MEMORY`** | **PG 17+** | Báo cáo lượng bộ nhớ RAM mà chính bộ lập kế hoạch (*Query Planner*) tiêu thụ để dựng execution plan (`Planner Memory: used=... bytes allocated=... bytes`), rất hữu ích với câu truy vấn join hàng chục bảng hoặc bảng phân vùng lớn. |
> | **`FORMAT JSON`** | PG 9.0+ | Xuất plan dạng JSON đầy đủ chi tiết để dán vào các công cụ trực quan hóa (*visualizer*) hiện đại. |
>
> ---
>
> **Bộ ba công cụ bắt buộc khi tối ưu hiệu năng PostgreSQL trên Production:**
>
> 1. **`pg_stat_statements` — Rada săn tìm truy vấn chậm toàn hệ thống:**
>    - Thay vì đoán mò câu lệnh nào chậm, extension chuẩn `pg_stat_statements` gom nhóm mọi câu truy vấn (sau khi chuẩn hóa hằng số thành `$1, $2`) và thống kê tổng thời gian thực thi, số lần gọi, số block đọc từ RAM/đĩa và lượng WAL sinh ra.
>    - *(Lưu ý phiên bản:* Từ **PostgreSQL 13+**, cột `total_time` cũ được tách thành `total_plan_time` và `total_exec_time` — cần bật `pg_stat_statements.track_planning = on` nếu muốn theo dõi cả thời gian lập kế hoạch).*
> 2. **`auto_explain` — Tự động bắt "quả tang" Execution Plan thực tế lúc nửa đêm:**
>    - Khi một truy vấn chỉ chậm thoáng qua lúc 2 giờ sáng (do khóa, do cache lạnh, hoặc do bộ tham số đặc biệt gây *bad custom plan*), khi bạn chạy lại `EXPLAIN ANALYZE` vào buổi sáng thì nó đã chạy nhanh trở lại!
>    - Module `auto_explain` giải quyết triệt để bài toán này bằng cách **tự động ghi toàn bộ kết quả `EXPLAIN (ANALYZE, BUFFERS, WAL, SETTINGS)` vào file log của PostgreSQL** ngay khi có bất kỳ câu truy vấn nào chạy vượt quá ngưỡng thời gian cấu hình (ví dụ `500ms`).
> 3. **Trình trực quan hóa Execution Plan (`explain.dalibo.com` & `explain.depesz.com`):**
>    - Với các plan dài hàng trăm dòng, đọc văn bản thuần rất dễ sót nút cổ chai. Cộng đồng PostgreSQL sử dụng hai công cụ chuẩn (có thể cài offline/on-premise hoàn toàn):
>      - **`explain.dalibo.com` (PEV2 — Postgres Explain Visualizer 2):** Vẽ sơ đồ cây trực quan với thanh nhiệt thời gian thực thi (*exclusive time*), lượng buffer đọc/ghi và độ lệch ước lượng (*estimation inaccuracy*) khi bạn chạy `EXPLAIN (ANALYZE, COSTS, VERBOSE, BUFFERS, FORMAT JSON)`.
>      - **`explain.depesz.com`:** Chấp nhận trực tiếp đầu ra text của `EXPLAIN (ANALYZE, BUFFERS)`, tô màu đỏ rực những dòng có thời gian thực thi độc lập (*exclusive time*) cao nhất hoặc ước lượng số dòng (`rows`) sai lệch hàng trăm lần.

> 💡 **Ví dụ bổ sung (không có trong sách) — Thực hành trên PostgreSQL: Sử dụng các cờ `EXPLAIN` thế hệ mới (PG 12–17), `pg_stat_statements` và `auto_explain`:**
>
> **1. Xem Generic Plan trực tiếp với `$1, $2` mà không cần `PREPARE` (PostgreSQL 16+):**
>
> ```sql
> EXPLAIN (GENERIC_PLAN, SETTINGS)
> SELECT sale_id, eur_value
>   FROM sales
>  WHERE subsidiary_id = $1
>    AND sale_date >= $2;
> ```
>
> **2. Chẩn đoán toàn diện một truy vấn bằng tất cả các cờ mới trên PostgreSQL 17:**
>
> ```sql
> EXPLAIN (ANALYZE, BUFFERS, WAL, SETTINGS, SERIALIZE, MEMORY)
> SELECT employee_id, last_name, profile_json
>   FROM employees
>  WHERE subsidiary_id = 30;
> ```
>
> Kết quả đầu ra trên PostgreSQL 17:
>
> ```text
> Index Scan using emp_sub_id on employees  (cost=0.29..42.15 rows=15 width=540) (actual time=0.025..0.180 rows=15 loops=1)
>   Index Cond: (subsidiary_id = 30)
>   Buffers: shared hit=18 read=2
>   I/O Timings: shared read=0.095
> Settings: effective_cache_size = '16GB', random_page_cost = '1.1', work_mem = '64MB'
> Planning:
>   Buffers: shared hit=12
>   Memory: used=14528B  allocated=32768B
> Planning Time: 0.142 ms
> Serialization: time=0.410 ms  output=128kB  format=text
> Execution Time: 0.635 ms
> ```
> *(Nhờ cờ `SERIALIZE` của PG 17, ta phát hiện riêng bước giải nén cột `profile_json` từ bảng TOAST và đóng gói kết quả mất `0.410 ms` — chiếm hơn 60% tổng `Execution Time: 0.635 ms`!)*
>
> **3. Truy vấn `pg_stat_statements` (PostgreSQL 13–17) để tìm Top 5 câu lệnh ngốn tổng thời gian và I/O cao nhất:**
>
> ```sql
> SELECT calls,
>        ROUND(total_exec_time::numeric, 2) AS total_exec_ms,
>        ROUND(mean_exec_time::numeric, 2)  AS avg_exec_ms,
>        rows                               AS total_rows,
>        shared_blks_hit,
>        shared_blks_read,
>        ROUND(100.0 * shared_blks_hit / NULLIF(shared_blks_hit + shared_blks_read, 0), 2) AS cache_hit_pct,
>        LEFT(query, 100)                   AS query_sample
>   FROM pg_stat_statements
>  ORDER BY total_exec_time DESC
>  LIMIT 5;
> ```
>
> **4. Cấu hình `auto_explain` trong `postgresql.conf` (hoặc bật theo session để debug):**
>
> ```sql
> -- Bật ngay trong session hiện tại để tự động ghi plan của các câu lệnh chạy > 200ms:
> LOAD 'auto_explain';
> SET auto_explain.log_min_duration = '200ms';
> SET auto_explain.log_analyze      = on;
> SET auto_explain.log_buffers      = on;
> SET auto_explain.log_wal          = on;
> SET auto_explain.log_settings     = on;
> SET auto_explain.log_nested_statements = on; -- Bắt cả câu SQL nằm bên trong hàm PL/pgSQL!
> -- Lưu ý trên Production tải cao: đặt auto_explain.sample_rate = 0.05 (chỉ đo mẫu 5% số truy vấn)
> -- để tránh overhead của việc đo thời gian từng nút (timing overhead).
> ```

### Các thao tác (Operations) — PostgreSQL

#### Truy cập index và bảng (Index and Table Access) — PostgreSQL

**Seq Scan**

Thao tác `Seq Scan` quét **toàn bộ relation** (*quan hệ*, tức bảng) theo đúng cách lưu trên đĩa (giống `TABLE ACCESS FULL` của Oracle).

**Index Scan**

`Index Scan` thực hiện duyệt B-tree, đi qua các nút lá để tìm tất cả các mục khớp, **và** lấy dữ liệu tương ứng trong bảng. Nó giống như một `INDEX RANGE SCAN` theo sau bởi một thao tác `TABLE ACCESS BY INDEX ROWID` (của Oracle). Xem thêm Chương 1, "Giải phẫu một index" (*Anatomy of an Index*).

Những thứ được gọi là index filter predicate thường gây ra vấn đề hiệu năng cho `Index Scan`. Mục tiếp theo giải thích cách nhận diện chúng.

**Index Only Scan (từ PostgreSQL 9.2)**

`Index Only Scan` thực hiện duyệt B-tree và đi qua các nút lá để tìm tất cả các mục khớp. **Không cần truy cập bảng** vì index đã có đủ mọi cột để đáp ứng truy vấn (ngoại lệ: thông tin về **khả năng hiển thị MVCC** — *MVCC visibility information*, tức thông tin cho biết một phiên bản dòng có "nhìn thấy được" với transaction hiện tại hay không). Xem thêm "Index-Only Scan" ở trang 116.

> 📝 **Giải thích thêm (không có trong sách):** Do ngoại lệ MVCC nói trên, PostgreSQL dùng **visibility map** để biết trang nào của bảng chứa toàn dòng "ai cũng thấy". Với các trang đó, Index Only Scan không phải đọc bảng; với các trang khác, nó vẫn phải ghé bảng. Trong `EXPLAIN ANALYZE` bạn sẽ thấy dòng `Heap Fetches: N` — số lần vẫn phải đọc bảng. Chạy `VACUUM` thường xuyên giúp giảm con số này. Ngoài ra, từ **PostgreSQL 11** có thể thêm cột "chỉ để đọc" vào index bằng `INCLUDE` để phục vụ Index Only Scan: `CREATE INDEX ... ON sales (sale_date) INCLUDE (eur_value);`.

<!-- [PDF page 185] -->

**Bitmap Index Scan / Bitmap Heap Scan / Recheck Cond**

Bài đăng của Tom Lane trên danh sách thư (*mailing list*) về hiệu năng của PostgreSQL giải thích điều này rất rõ ràng và súc tích:

> Một Index Scan thông thường lấy **từng con trỏ tuple một** (*tuple-pointer* — con trỏ tới vị trí của dòng) từ index, và **ngay lập tức** ghé thăm tuple đó trong bảng. Một bitmap scan thì lấy **tất cả** các con trỏ tuple từ index **trong một lần**, sắp xếp chúng bằng một cấu trúc dữ liệu "bitmap" trong bộ nhớ, rồi ghé thăm các tuple của bảng **theo thứ tự vị trí vật lý** của tuple.
>
> *(Nguyên văn: "A plain Index Scan fetches one tuple-pointer at a time from the index, and immediately visits that tuple in the table. A bitmap scan fetches all the tuple-pointers from the index in one go, sorts them using an in-memory "bitmap" data structure, and then visits the table tuples in physical tuple-location order.")*
>
> — Tom Lane[^1]

> 📝 **Giải thích thêm (không có trong sách):** Trong plan, ba tên này đi cùng nhau: `Bitmap Index Scan` (đọc index, dựng bitmap) là con của `Bitmap Heap Scan` (đọc bảng theo thứ tự vật lý). `Recheck Cond` xuất hiện dưới `Bitmap Heap Scan` vì khi bitmap quá lớn so với `work_mem`, nó trở nên "lossy" (chỉ nhớ trang chứ không nhớ dòng cụ thể), nên điều kiện phải được kiểm tra lại trên từng dòng của trang. Bitmap scan còn có thể kết hợp nhiều index bằng `BitmapAnd` / `BitmapOr`, ví dụ:
>
> ```text
>  Bitmap Heap Scan on employees
>    Recheck Cond: ((subsidiary_id = 30) AND (last_name = 'WINAND'::text))
>    ->  BitmapAnd
>          ->  Bitmap Index Scan on emp_sub_id
>                Index Cond: (subsidiary_id = 30)
>          ->  Bitmap Index Scan on emp_name
>                Index Cond: (last_name = 'WINAND'::text)
> ```

#### Các thao tác join (Join Operations) — PostgreSQL

Nhìn chung, các thao tác join chỉ xử lý **hai bảng một lúc**. Nếu một truy vấn có nhiều phép join hơn, chúng được thực hiện **tuần tự**: đầu tiên là hai bảng, sau đó là kết quả trung gian với bảng tiếp theo. Vì vậy, trong ngữ cảnh join, từ "bảng" cũng có thể mang nghĩa "kết quả trung gian".

**Nested Loops**

Join hai bảng bằng cách lấy kết quả từ một bảng, rồi với mỗi dòng từ bảng thứ nhất, truy vấn bảng còn lại. Xem thêm "Nested Loops" ở trang 92.

**Hash Join / Hash**

Hash join nạp các bản ghi ứng viên từ một phía của phép join vào một bảng băm (được đánh dấu bằng **`Hash`** trong plan), sau đó bảng băm này được dò tìm cho mỗi bản ghi từ phía bên kia của phép join. Xem thêm "Hash Join" ở trang 101.

**Merge Join**

(Sort) merge join kết hợp hai danh sách đã sắp xếp giống như khóa kéo. Cả hai phía của phép join đều phải được sắp xếp trước. Xem thêm "Sort Merge" ở trang 109.

[^1]: http://archives.postgresql.org/pgsql-performance/2005-12/msg00623.php

> 💡 **Ví dụ bổ sung (không có trong sách):** Cách nhận diện các phép join trong plan PostgreSQL (dạng mong đợi, đã rút gọn cost):
>
> ```text
>  Hash Join
>    Hash Cond: (s.employee_id = e.employee_id)
>    ->  Seq Scan on sales s
>    ->  Hash
>          ->  Index Scan using emp_name on employees e
>                Index Cond: (last_name = 'WINAND'::text)
> ```
>
> Nút `Hash` là con của `Hash Join`: nó xây bảng băm từ phía `employees` (sau khi lọc theo tên — phía nhỏ). Sau đó `Seq Scan on sales` đọc từng dòng và dò vào bảng băm theo `Hash Cond`. Nếu tạo thêm index `CREATE INDEX sales_emp ON sales(employee_id);`, plan có thể chuyển thành:
>
> ```text
>  Nested Loop
>    ->  Index Scan using emp_name on employees e
>          Index Cond: (last_name = 'WINAND'::text)
>    ->  Index Scan using sales_emp on sales s
>          Index Cond: (employee_id = e.employee_id)
> ```

<!-- [PDF page 186] -->

#### Sắp xếp và gom nhóm (Sorting and Grouping) — PostgreSQL

**Sort / Sort Key**

Sắp xếp tập dữ liệu theo các cột được nêu trong `Sort Key`. Thao tác `Sort` cần lượng bộ nhớ lớn để vật chất hóa kết quả trung gian (không pipelined). Xem thêm "Indexing Order By" ở trang 130.

**GroupAggregate**

Tổng hợp một tập dữ liệu **đã sắp xếp sẵn** theo mệnh đề `group by`. Thao tác này **không đệm** lượng dữ liệu lớn (pipelined). Xem thêm "Indexing Group By" ở trang 139.

**HashAggregate**

Dùng một **bảng băm tạm** để gom nhóm các bản ghi. Thao tác `HashAggregate` không yêu cầu dữ liệu đầu vào đã được sắp xếp, nhưng thay vào đó nó dùng lượng bộ nhớ lớn để vật chất hóa kết quả trung gian (không pipelined). Kết quả đầu ra không có thứ tự nào có ý nghĩa. Xem thêm "Indexing Group By" ở trang 139.

> 💡 **Ví dụ bổ sung (không có trong sách):** Với bảng `sales(sale_date date, product_id int, eur_value numeric)` (PostgreSQL):
>
> ```sql
> EXPLAIN SELECT product_id, sum(eur_value) FROM sales GROUP BY product_id;
> ```
>
> - Không có index phù hợp: thường thấy `HashAggregate` → `Seq Scan on sales` (hoặc `GroupAggregate` → `Sort` (`Sort Key: product_id`) → `Seq Scan` nếu số nhóm quá nhiều so với `work_mem`).
> - Sau khi `CREATE INDEX sales_prod ON sales(product_id, eur_value);` và bảng đã được `VACUUM`: có thể thấy `GroupAggregate` → `Index Only Scan using sales_prod` — dữ liệu đến đúng thứ tự `product_id`, không cần `Sort`, chạy pipelined.

#### Truy vấn Top-N (Top-N Queries) — PostgreSQL

**Limit**

Dừng các thao tác bên dưới khi đã lấy đủ số dòng mong muốn. Xem thêm "Querying Top-N Rows" ở trang 143.

Hiệu quả của truy vấn top-N phụ thuộc vào chế độ thực thi của các thao tác bên dưới. Nó **rất kém hiệu quả** khi phải dừng các thao tác không pipelined như `Sort`.

**WindowAgg**

Cho biết có sử dụng **hàm cửa sổ** (*window functions*). Xem thêm "Using Window Functions for Pagination" ở trang 156.

> 🚫 **Thận trọng (Caution)**
>
> PostgreSQL **không thể** thực thi truy vấn top-N theo kiểu pipelined khi dùng hàm cửa sổ.

> 💡 **Ví dụ bổ sung (không có trong sách):** Lấy 10 dòng bán hàng mới nhất (PostgreSQL):
>
> ```sql
> -- Có index sales_dt ON sales(sale_date)
> EXPLAIN SELECT * FROM sales ORDER BY sale_date DESC LIMIT 10;
> ```
>
> ```text
>  Limit
>    ->  Index Scan Backward using sales_dt on sales
> ```
>
> `Limit` dừng sau 10 dòng, và vì Index Scan đã trả dữ liệu theo đúng thứ tự (đọc ngược index), chỉ khoảng 10 mục index được đọc → pipelined. Nếu không có index, plan sẽ là `Limit` → `Sort` (`Sort Method: top-N heapsort`) → `Seq Scan`: vẫn phải đọc hết bảng.
>
> Còn với hàm cửa sổ:
>
> ```sql
> SELECT * FROM (
>   SELECT s.*, ROW_NUMBER() OVER (ORDER BY sale_date DESC) rn FROM sales s
> ) t WHERE rn <= 10;
> ```
>
> Plan có `WindowAgg` nằm dưới một `Subquery Scan` với `Filter: (rn <= 10)`; ở các phiên bản PostgreSQL mà sách đề cập, việc lọc này không dừng được bước bên dưới — đúng như lời cảnh báo ở trên. (Ghi chú: từ **PostgreSQL 15**, optimizer đã có thể dừng sớm `WindowAgg` với điều kiện kiểu `row_number() <= N` — plan hiện `Run Condition`.)

> 📝 **Giải thích thêm (không có trong sách) — Đối chiếu với PostgreSQL hiện đại (PG 10–17): Các nút Execution Plan mới xuất hiện từ PostgreSQL 10 đến 17:**
>
> Khi đọc `EXPLAIN` trên PostgreSQL 10–17, ngoài các thao tác kinh điển mà sách liệt kê (từ thời PG 9.2), bạn sẽ thường xuyên bắt gặp 4 nhóm nút thực thi thế hệ mới cực kỳ quan trọng sau:
>
> 1. **`Incremental Sort` (Sắp xếp tăng dần — từ PostgreSQL 13+):**
>    - **Vấn đề trước PG 13:** Nếu bạn có index trên `(sale_date)` nhưng truy vấn yêu cầu `ORDER BY sale_date, eur_value`, PostgreSQL cũ buộc phải đọc toàn bộ dữ liệu rồi `Sort` lại từ đầu (không pipelined).
>    - **Từ PostgreSQL 13+:** Nút `Incremental Sort` nhận biết dữ liệu từ `Index Scan` đã được sắp xếp sẵn theo tiền tố `Presorted Key: sale_date`. Do đó, nó chỉ gom từng cụm nhỏ có cùng `sale_date` để sắp xếp cục bộ theo `eur_value` rồi đẩy kết quả ra ngay theo kiểu **pipelined**! Nhờ vậy, các truy vấn `ORDER BY a, b LIMIT 10` khi chỉ có index trên `(a)` chạy nhanh hơn hàng chục đến hàng trăm lần.
> 2. **`Memoize` (Bộ nhớ đệm vòng lặp trong — từ PostgreSQL 14+):**
>    - Xuất hiện bên dưới `Nested Loop` (giữa `Nested Loop` và `Index Scan` phía trong). Khi bảng phía ngoài có nhiều dòng lặp lại cùng một khóa join (ví dụ hàng nghìn đơn hàng cùng thuộc một `subsidiary_id`), nút `Memoize` lưu kết quả tra cứu của nhánh trong vào bảng băm trong bộ nhớ (`Cache Key`), giúp bỏ qua hoàn toàn các lần duyệt B-Tree lặp lại (`Hits: 9950  Misses: 50  Evictions: 0`).
> 3. **`Gather` / `Gather Merge` và các nút `Parallel ...` (Truy vấn song song — hoàn thiện mạnh ở PG 10–17):**
>    - PostgreSQL hiện đại có thể chia tải cho nhiều tiến trình *background worker* cùng thực hiện `Parallel Seq Scan`, `Parallel Index Scan`, `Parallel Index Only Scan`, `Parallel Bitmap Heap Scan`, `Parallel Hash Join` và `Partial HashAggregate`. Nút `Gather` (không giữ thứ tự) hoặc `Gather Merge` (giữ thứ tự đã sắp xếp) ở phía trên có nhiệm vụ thu gom kết quả từ các `Workers Planned` / `Workers Launched`.
> 4. **`Append` / `MergeAppend` kèm Partition Pruning (Cắt tỉa phân vùng — PG 11–17):**
>    - Với bảng phân vùng (*Declarative Partitioning* từ PG 10+), PostgreSQL có thể loại bỏ các phân vùng không liên quan ngay lúc lập kế hoạch hoặc lúc thực thi (hiển thị `Subplans Removed: N` trong plan), thay vì phải quét toàn bộ các bảng con như cơ chế kế thừa bảng (`INHERITS`) thời PG 9.2.

> 💡 **Ví dụ bổ sung (không có trong sách) — Thực hành trên PostgreSQL: Nhận diện `Incremental Sort` (PG 13+) và `Memoize` (PG 14+) trong Execution Plan:**
>
> ```sql
> -- Ví dụ 1: Incremental Sort (chỉ có index trên sale_date, nhưng ORDER BY sale_date DESC, eur_value DESC LIMIT 10)
> EXPLAIN (ANALYZE, BUFFERS)
> SELECT sale_id, sale_date, eur_value
>   FROM sales
>  ORDER BY sale_date DESC, eur_value DESC
>  LIMIT 10;
> ```
>
> ```text
> Limit  (cost=1.45..3.12 rows=10 width=20) (actual time=0.045..0.062 rows=10 loops=1)
>   Buffers: shared hit=18
>   ->  Incremental Sort  (cost=1.45..168420.50 rows=1000000 width=20) (actual time=0.044..0.059 rows=10 loops=1)
>         Sort Key: sale_date DESC, eur_value DESC
>         Presorted Key: sale_date
>         Full-sort Groups: 1  Sort Method: quicksort  Average Memory: 26kB  Peak Memory: 26kB
>         Buffers: shared hit=18
>         ->  Index Scan Backward using sales_dt on sales  (cost=0.42..42150.42 rows=1000000 width=20) (actual time=0.015..0.032 rows=15 loops=1)
>               Buffers: shared hit=18
> ```
> *(Nhờ `Incremental Sort` với `Presorted Key: sale_date`, PostgreSQL chỉ đọc đúng 15 dòng của ngày mới nhất (`Full-sort Groups: 1`), sắp xếp 15 dòng đó theo `eur_value DESC` bằng `26kB` RAM và trả ngay 10 dòng đầu tiên!)*
>
> ```sql
> -- Ví dụ 2: Nested Loop + Memoize (PG 14+) khi join bảng sales (nhiều dòng trùng customer_id) với customers:
> EXPLAIN (ANALYZE, BUFFERS)
> SELECT s.sale_id, c.customer_name
>   FROM sales s
>   JOIN customers c ON c.customer_id = s.customer_id
>  WHERE s.sale_date = DATE '2026-10-24';
> ```
>
> ```text
> Nested Loop  (cost=0.72..412.50 rows=1000 width=40) (actual time=0.035..1.240 rows=1000 loops=1)
>   Buffers: shared hit=215
>   ->  Index Scan using sales_dt on sales s  (cost=0.42..38.50 rows=1000 width=12) (actual time=0.018..0.290 rows=1000 loops=1)
>         Index Cond: (sale_date = '2026-10-24'::date)
>         Buffers: shared hit=15
>   ->  Memoize  (cost=0.30..0.52 rows=1 width=36) (actual time=0.000..0.001 rows=1 loops=1000)
>         Cache Key: s.customer_id
>         Cache Mode: logical
>         Hits: 950  Misses: 50  Evictions: 0  Overflows: 0  Memory Usage: 6kB
>         Buffers: shared hit=200
>         ->  Index Scan using customers_pkey on customers c  (cost=0.29..0.51 rows=1 width=36) (actual time=0.003..0.003 rows=1 loops=50)
>               Index Cond: (customer_id = s.customer_id)
>               Buffers: shared hit=200
> ```
> *(Chú ý: Dù vòng lặp ngoài có 1.000 dòng (`loops=1000`), nhờ `Memoize` trúng bộ nhớ đệm 950 lần (`Hits: 950  Misses: 50`), nhánh `Index Scan` trên `customers_pkey` chỉ phải chạy thực sự **50 lần (`loops=50`)** thay vì 1.000 lần!)*

<!-- [PDF page 187] -->

### Phân biệt access predicate và filter predicate (Distinguishing Access and Filter-Predicates) — PostgreSQL

PostgreSQL dùng **ba phương pháp khác nhau** để áp dụng mệnh đề `where` (các vị từ – predicates):

**Access Predicate – vị từ truy cập ("Index Cond")**
: Access predicate biểu diễn điều kiện bắt đầu và điều kiện dừng của việc duyệt nút lá.

**Index Filter Predicate – vị từ lọc trên index ("Index Cond")**
: Index filter predicate chỉ được áp dụng trong lúc duyệt nút lá. Chúng không góp phần vào điều kiện bắt đầu/dừng và không thu hẹp phạm vi được quét.

**Table level filter predicate – vị từ lọc ở mức bảng ("Filter")**
: Các vị từ trên những cột không thuộc index được đánh giá ở mức bảng. Để làm được điều đó, database phải nạp dòng từ **heap table** (*bảng heap* — cách PostgreSQL lưu dữ liệu bảng, không theo thứ tự nào) trước.

Execution plan của PostgreSQL **không** hiển thị riêng biệt access predicate và index filter predicate — **cả hai** đều hiện ra dưới tên **"Index Cond"**. Điều đó có nghĩa là bạn phải **so sánh execution plan với định nghĩa của index** để phân biệt access predicate với index filter predicate.

> 📌 **Ghi chú (Note)**
>
> Explain plan của PostgreSQL **không cung cấp đủ thông tin** để tìm ra các index filter predicate.

Các vị từ hiển thị là **"Filter"** thì **luôn luôn** là filter predicate ở mức bảng — **kể cả khi** chúng được hiển thị dưới một thao tác `Index Scan`.

Hãy xét ví dụ sau, vốn xuất hiện lần đầu trong chương "Hiệu năng và khả năng mở rộng" (*Performance and Scalability*):

```sql
CREATE TABLE scale_data (
   section NUMERIC NOT NULL,
   id1     NUMERIC NOT NULL,
   id2     NUMERIC NOT NULL
);
CREATE INDEX scale_data_key ON scale_data(section, id1);
```

<!-- [PDF page 188] -->

Câu `select` sau lọc trên cột `ID2`, là cột **không có** trong index:

```sql
PREPARE stmt(int) AS SELECT count(*)
                       FROM scale_data
                      WHERE section = 1
                        AND id2 = $1;
EXPLAIN EXECUTE stmt(1);
```

```text
                      QUERY PLAN
-----------------------------------------------------
Aggregate  (cost=529346.31..529346.32 rows=1 width=0)
  Output: count(*)
  -> Index Scan using scale_data_key on scale_data
     (cost=0.00..529338.83 rows=2989 width=0)
     Index Cond: (scale_data.section = 1::numeric)
     Filter: (scale_data.id2 = ($1)::numeric)
```

Vị từ trên `ID2` xuất hiện dưới dạng `"Filter"` bên dưới thao tác `Index Scan`. Đó là vì PostgreSQL thực hiện việc truy cập bảng **như một phần của** thao tác `Index Scan`. Nói cách khác, thao tác `TABLE ACCESS BY INDEX ROWID` của Oracle Database **bị ẩn bên trong** thao tác `Index Scan` của PostgreSQL. Do đó, một `Index Scan` hoàn toàn có thể lọc trên những cột **không có trong index**.

> ❗ **Quan trọng (Important)**
>
> Các vị từ `Filter` của PostgreSQL là **filter predicate ở mức bảng** — kể cả khi chúng được hiển thị dưới một `Index Scan`.

Khi ta thêm index từ chương "Performance and Scalability", ta có thể thấy **tất cả** các cột đều hiện ra dưới dạng `"Index Cond"` — **bất kể** chúng là access predicate hay filter predicate.

```sql
CREATE INDEX scale_slow
          ON scale_data (section, id1, id2);
```

Execution plan với index mới **không hiển thị bất kỳ điều kiện filter nào**:

```text
                      QUERY PLAN
------------------------------------------------------
Aggregate  (cost=14215.98..14215.99 rows=1 width=0)
  Output: count(*)
  -> Index Scan using scale_slow on scale_data
     (cost=0.00..14208.51 rows=2989 width=0)
     Index Cond: (section = 1::numeric AND id2 = ($1)::numeric)
```

<!-- [PDF page 189] -->

Xin lưu ý rằng điều kiện trên `ID2` **không thể** thu hẹp việc duyệt nút lá, bởi vì index có cột `ID1` đứng **trước** `ID2`. Điều đó có nghĩa là `Index Scan` sẽ quét **toàn bộ phạm vi** thỏa điều kiện `SECTION=1::numeric` và áp dụng bộ lọc `ID2=($1)::numeric` lên **từng dòng** thỏa mệnh đề trên `SECTION`.

> 📝 **Giải thích thêm (không có trong sách):** So sánh ba "tầng" lọc trong hai ví dụ trên:
>
> | Index | Điều kiện | Hiển thị trong plan PostgreSQL | Vai trò thực sự |
> |---|---|---|---|
> | `scale_data_key (section, id1)` | `section = 1` | `Index Cond` | Access predicate |
> | `scale_data_key (section, id1)` | `id2 = $1` | `Filter` | Table level filter (phải đọc bảng cho mọi dòng có `section = 1`) |
> | `scale_slow (section, id1, id2)` | `section = 1` | `Index Cond` | Access predicate |
> | `scale_slow (section, id1, id2)` | `id2 = $1` | `Index Cond` | **Index filter predicate** (kiểm tra trên nút lá, không thu hẹp phạm vi) |
>
> Chi phí giảm từ ~529.338 xuống ~14.208 vì với `scale_slow`, các dòng không thỏa `id2` bị loại ngay ở nút lá, không cần đọc bảng. Nhưng vẫn phải quét cả khoảng `section = 1` trên index — điều mà plan không nói ra; bạn phải tự suy luận từ định nghĩa index.

> 💡 **Ví dụ bổ sung (không có trong sách):** Bài tập tự kiểm tra trên PostgreSQL. Với bảng `scale_data` ở trên:
>
> ```sql
> CREATE INDEX scale_fast ON scale_data (section, id2, id1);
> EXPLAIN EXECUTE stmt(1);
> ```
>
> Plan vẫn chỉ hiện `Index Cond: (section = 1::numeric AND id2 = ($1)::numeric)` — **giống hệt** về hình thức với khi dùng `scale_slow`! Nhưng lần này cả hai điều kiện đều là access predicate, vì `section` và `id2` là hai cột **đầu tiên liên tiếp** của index. Dấu hiệu nhận biết duy nhất trong plan là cost/thời gian thấp hơn hẳn. Đó chính là ý của tác giả: muốn phân biệt access và index filter predicate trong PostgreSQL, phải đối chiếu với định nghĩa index.
>
> Quy tắc tự suy luận: đi lần lượt các cột của index từ trái sang phải; các cột có điều kiện `=` liên tiếp từ đầu (cộng thêm tối đa một cột có điều kiện khoảng `<`, `>`, `BETWEEN` ngay sau đó) là access predicate; điều kiện trên các cột còn lại của index là index filter predicate.
>
> (Bổ sung theo phiên bản mới: từ **PostgreSQL 18**, B-tree hỗ trợ *skip scan*, nên trong một số trường hợp — khi cột bị "bỏ qua" như `id1` có ít giá trị khác nhau — điều kiện trên cột đứng sau vẫn có thể giúp nhảy qua các phần của index. Oracle có cơ chế tương tự từ lâu với `INDEX SKIP SCAN`.)

---

## Bảng thuật ngữ (Glossary)

| Thuật ngữ (English) | Nghĩa tiếng Việt | Giải thích ngắn |
|---|---|---|
| Execution plan | Kế hoạch thực thi | Các bước cụ thể database dùng để thực thi một câu lệnh SQL. |
| Optimizer | Bộ tối ưu hóa truy vấn | Thành phần tạo ra execution plan, giống như compiler dịch SQL thành chương trình chạy được. |
| Compiler | Trình biên dịch | Phép so sánh của tác giả: SQL là mã nguồn, plan là chương trình thực thi. |
| `PLAN_TABLE` | Bảng kế hoạch | Bảng Oracle dùng để lưu kết quả của `EXPLAIN PLAN FOR`. |
| Global temporary table | Bảng tạm toàn cục | Bảng có định nghĩa dùng chung nhưng dữ liệu riêng cho từng session (Oracle). |
| `DBMS_XPLAN` | — | Package Oracle (từ 9iR2) định dạng và hiển thị execution plan. |
| B-tree traversal | Duyệt cây B-tree | Đi từ nút gốc xuống nút lá để tìm điểm bắt đầu. |
| Leaf node chain | Chuỗi nút lá | Danh sách liên kết hai chiều nối các nút lá của index. |
| ROWID | Địa chỉ dòng | Địa chỉ vật lý của một dòng trong bảng Oracle. |
| Full table scan | Quét toàn bộ bảng | Đọc tất cả dòng và cột của bảng (`TABLE ACCESS FULL`, `Seq Scan`). |
| Multi-block read | Đọc nhiều block một lúc | Đọc nhiều block liên tiếp trong một thao tác IO. |
| Intermediate result | Kết quả trung gian | Kết quả của một bước join được dùng làm đầu vào cho bước kế tiếp. |
| Nested loops join | Join vòng lặp lồng | Với mỗi dòng phía ngoài, truy vấn bảng phía trong. |
| Hash join | Join băm | Nạp một phía vào hash table, rồi dò bằng các dòng phía kia. |
| Merge join (sort merge) | Join trộn | Kết hợp hai danh sách đã sắp xếp như khóa kéo. |
| Hash table | Bảng băm | Cấu trúc dữ liệu tra cứu nhanh theo khóa. |
| Probe | Dò tìm | Tra một giá trị vào hash table. |
| Materialize | Vật chất hóa | Lưu toàn bộ kết quả trung gian trước khi xử lý tiếp. |
| Pipelined | Thực thi kiểu dây chuyền | Mỗi dòng được chuyển ngay cho bước sau, không chờ đọc hết đầu vào. |
| Top-N query | Truy vấn Top-N | Truy vấn chỉ lấy N dòng đầu tiên theo một thứ tự. |
| Window function | Hàm cửa sổ | Hàm dùng mệnh đề `over`, ví dụ `ROW_NUMBER()`. |
| Predicate | Vị từ / điều kiện | Một điều kiện trong mệnh đề `where`. |
| Access predicate | Vị từ truy cập | Điều kiện xác định điểm bắt đầu/dừng khi duyệt nút lá. |
| Index filter predicate | Vị từ lọc trên index | Điều kiện kiểm tra trên nút lá, không thu hẹp phạm vi quét. |
| Table level filter predicate | Vị từ lọc mức bảng | Điều kiện trên cột không thuộc index, cần đọc dòng từ bảng. |
| Predicate Information | Thông tin vị từ | Phần bên dưới plan của `DBMS_XPLAN` cho biết access/filter của từng bước. |
| Bind parameter | Tham số ràng buộc | Chỗ giữ giá trị (`$1`, `?`, `:a`) truyền vào khi thực thi. |
| Prepared statement | Câu lệnh đã chuẩn bị | Câu lệnh được `PREPARE` trước, thực thi bằng `EXECUTE`, đóng bằng `DEALLOCATE`. |
| Database abstraction layer | Tầng trừu tượng CSDL | Thư viện/ORM giữa ứng dụng và database, có thể che giấu cú pháp bind parameter. |
| Startup cost / Total cost | Chi phí khởi động / tổng chi phí | Hai giá trị cost của PostgreSQL: trước dòng đầu tiên / khi lấy hết các dòng. |
| Cardinality estimate | Ước lượng cardinality | Ước lượng số dòng mà một bước trả về. |
| `VERBOSE` / `ANALYZE` | — | Tùy chọn của `EXPLAIN` trong PostgreSQL: thêm chi tiết / thực thi thật và đo thời gian, số dòng. |
| Transaction / Rollback | Giao dịch / Hoàn tác | Bọc `EXPLAIN ANALYZE` trong transaction rồi rollback để không làm đổi dữ liệu. |
| Relation | Quan hệ | Thuật ngữ PostgreSQL cho bảng (và các đối tượng tương tự). |
| MVCC visibility information | Thông tin hiển thị MVCC | Thông tin cho biết một phiên bản dòng có hiển thị với transaction hiện tại không. |
| Index Only Scan | Quét chỉ trên index | Đáp ứng truy vấn chỉ từ index, không cần đọc bảng (PostgreSQL 9.2+). |
| Bitmap scan | Quét bitmap | Lấy hết con trỏ tuple từ index, sắp xếp bằng bitmap, rồi đọc bảng theo thứ tự vật lý. |
| Tuple-pointer | Con trỏ tuple | Con trỏ tới vị trí vật lý của một dòng (tuple) trong bảng PostgreSQL. |
| Heap table | Bảng heap | Cách lưu bảng không theo thứ tự của PostgreSQL. |
| `Index Cond` | Điều kiện index | Nhãn PostgreSQL dùng cho cả access predicate lẫn index filter predicate. |
| `Filter` | Bộ lọc | Nhãn PostgreSQL cho filter predicate mức bảng. |
