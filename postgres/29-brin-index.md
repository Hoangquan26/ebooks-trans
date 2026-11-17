# CHƯƠNG 29: BRIN (CHAPTER 29: BRIN)

## 29.1 Tổng quan (29.1 Overview)

Khác với các loại chỉ mục khác được tối ưu để tìm kiếm nhanh các hàng cần thiết, **BRIN**[^1] được thiết kế để *loại bỏ* (filter out) các hàng không cần thiết. Phương thức truy cập (access method) này được tạo ra chủ yếu cho các bảng rất lớn từ vài terabyte trở lên, do đó kích thước chỉ mục nhỏ gọn được ưu tiên hơn độ chính xác khi tìm kiếm.

Để tăng tốc độ tìm kiếm, toàn bộ bảng được chia thành các phạm vi (range), vì vậy mới có tên gọi: **Block Range Index** (Chỉ mục phạm vi khối). Mỗi phạm vi bao gồm nhiều trang (page). Chỉ mục không lưu trữ các TID (Tuple ID), mà chỉ lưu thông tin tóm tắt (summary) về dữ liệu của từng phạm vi. Đối với các kiểu dữ liệu có thứ tự (ordinal data types), thông tin tóm tắt trong trường hợp đơn giản nhất là giá trị nhỏ nhất (min) và giá trị lớn nhất (max), nhưng các lớp toán tử (operator class) khác nhau có thể thu thập các thông tin khác nhau về các giá trị trong một phạm vi.

Số lượng trang trong một phạm vi được xác định tại thời điểm tạo chỉ mục dựa trên giá trị của tham số lưu trữ `pages_per_range` (mặc định là `128`).

Nếu một điều kiện truy vấn tham chiếu đến cột được đánh chỉ mục, tất cả các phạm vi được đảm bảo không có kết quả khớp sẽ bị bỏ qua. Các trang của tất cả các phạm vi còn lại được chỉ mục trả về dưới dạng một bitmap mất mát dữ liệu (lossy bitmap); tất cả các hàng trong các trang này đều phải được kiểm tra lại (recheck).

Do đó, BRIN hoạt động tốt đối với các cột có giá trị mang tính cục bộ hóa (localized values) — tức là các cột mà những giá trị được lưu trữ gần nhau có các thuộc tính thông tin tóm tắt tương đồng nhau. Đối với các kiểu dữ liệu có thứ tự, điều này có nghĩa là các giá trị phải được lưu trữ theo thứ tự tăng dần hoặc giảm dần, tức là có độ tương quan (correlation) cao giữa vị trí vật lý của chúng trên đĩa và thứ tự logic được xác định bởi các phép toán lớn hơn (`>`) và nhỏ hơn (`<`). Đối với các loại thông tin tóm tắt khác, khái niệm "thuộc tính tương đồng" có thể khác đi.

Sẽ không sai khi coi BRIN là một công cụ tăng tốc cho quá trình quét tuần tự heap (sequential heap scan) hơn là một chỉ mục theo nghĩa thông thường. Nó có thể được xem như một giải pháp thay thế cho phân vùng bảng (partitioning), trong đó mỗi phạm vi đóng vai trò như một phân vùng ảo (virtual partition).

[^1]: `postgresql.org/docs/14/brin.html`, `backend/access/brin/README`

> 📝 **Giải thích thêm (không có trong sách):**
> Tại sao chỉ mục BRIN lại nhỏ hơn B-tree hàng trăm đến hàng nghìn lần?
> B-tree phải lưu trữ một mục chỉ mục (index entry) cho *mỗi hàng* trong bảng. Nếu bảng có 100 triệu hàng, B-tree cũng có 100 triệu mục. Ngược lại, với `pages_per_range = 128` (mỗi trang 8 kB, tức 1 MB dữ liệu bảng cho mỗi phạm vi), BRIN chỉ lưu trữ **1 mục tóm tắt cho mỗi 1 MB dữ liệu** (khoảng vài nghìn hàng). Nhờ vậy, chỉ mục cho một bảng 1 TB có thể chỉ tốn vài chục MB!

---

## 29.2 Ví dụ minh họa (29.2 Example)

Cơ sở dữ liệu demo của chúng ta không có bảng nào đủ lớn cho BRIN, nhưng chúng ta có thể hình dung rằng các báo cáo phân tích đòi hỏi phải có một bảng phi chuẩn hóa (denormalized table) chứa thông tin tóm tắt về tất cả các chuyến bay đi và đến của một sân bay cụ thể, chi tiết đến từng ghế đã được đặt. Dữ liệu cho mỗi sân bay được cập nhật hàng ngày, ngay khi đồng hồ điểm nửa đêm theo múi giờ tương ứng. Dữ liệu đã thêm vào không bao giờ bị cập nhật hay xóa.

Bảng có cấu trúc như sau:

```sql
CREATE TABLE flights_bi(
  airport_code char(3),
  airport_coord point,         -- tọa độ sân bay
  airport_utc_offset interval, -- múi giờ
  flight_no char(6),
  flight_type text,            -- chuyến đi (departure) hoặc chuyến đến (arrival)
  scheduled_time timestamptz,
  actual_time timestamptz,
  aircraft_code char(3),
  seat_no varchar(4),
  fare_conditions varchar(10), -- hạng vé
  passenger_id varchar(20),
  passenger_name text
);
```

Việc nạp dữ liệu có thể được mô phỏng bằng các vòng lặp lồng nhau:[^2] vòng lặp ngoài tương ứng với các ngày (cơ sở dữ liệu demo lưu dữ liệu trong một năm), trong khi vòng lặp trong dựa trên các múi giờ. Kết quả là dữ liệu được nạp vào ít nhất sẽ được sắp xếp tương đối theo thời gian và sân bay, mặc dù nó không được sắp xếp một cách tường minh bên trong vòng lặp.

Tôi sẽ nạp một bản sao có sẵn của cơ sở dữ liệu, chiếm khoảng 4 GB và chứa khoảng 30 triệu hàng:[^3]

```bash
postgres$ pg_restore -d demo -c flights_bi.dump
```

```sql
=> ANALYZE flights_bi;
=> SELECT count(*) FROM flights_bi;
  count
----------
 30517076
(1 row)

=> SELECT pg_size_pretty(pg_total_relation_size('flights_bi'));
 pg_size_pretty
----------------
 4129 MB
(1 row)
```

[^2]: `edu.postgrespro.ru/internals-14/flights_bi.sql`
[^3]: `edu.postgrespro.ru/internals-14/flights_bi.dump`

---

## 29.3 Bố cục trang (29.3 Page Layout)

Chúng ta khó có thể gọi đây là một bảng lớn, nhưng lượng dữ liệu này sẽ đủ để minh họa cách BRIN hoạt động. Tôi sẽ tạo trước một chỉ mục:

```sql
=> CREATE INDEX ON flights_bi USING brin(scheduled_time);
=> SELECT pg_size_pretty(pg_total_relation_size(
  'flights_bi_scheduled_time_idx'
));
 pg_size_pretty
----------------
 184 kB
(1 row)
```

Nó chiếm rất ít không gian với các thiết lập mặc định.

Một chỉ mục B-tree lớn hơn gấp một nghìn lần, ngay cả khi tính năng chống trùng lặp dữ liệu (data deduplication, từ phiên bản 13) được bật. Đúng là hiệu quả của nó cũng cao hơn nhiều, nhưng dung lượng bổ sung đó có thể trở thành một sự xa xỉ không thể đáp ứng đối với các bảng thực sự lớn.

```sql
=> CREATE INDEX flights_bi_btree_idx ON flights_bi(scheduled_time);
=> SELECT pg_size_pretty(pg_total_relation_size(
  'flights_bi_btree_idx'
));
 pg_size_pretty
----------------
 210 MB
(1 row)

=> DROP INDEX flights_bi_btree_idx;
```

Trang số 0 của chỉ mục BRIN được gọi là **trang siêu dữ liệu (metapage)**; nó lưu giữ thông tin về cấu trúc của chỉ mục.

Ở một khoảng cách (offset) nhất định tính từ siêu dữ liệu là các trang chứa thông tin tóm tắt (summary information). Mỗi mục chỉ mục (index entry) trong một trang như vậy chứa bản tóm tắt của một phạm vi khối (block range) cụ thể.

Không gian giữa metapage và thông tin tóm tắt được dành cho **bản đồ phạm vi (range map)**, đôi khi còn được gọi là **bản đồ đảo ngược (reverse map)** (do đó có tên viết tắt phổ biến là `revmap`). Thực chất đây là một mảng các con trỏ trỏ tới các hàng chỉ mục tương ứng; số thứ tự chỉ mục trong mảng này tương ứng với số thứ tự của phạm vi.

```mermaid
flowchart LR
    M["Trang 0: Metapage<br/>(pagesperrange, lastrevmappage)"]
    R["Các trang Revmap<br/>(Mảng con trỏ theo số phạm vi)"]
    S["Các trang Tóm tắt (Summary)<br/>(Ví dụ: 1..10, 11..20, 21..30, ...)"]

    M --> R
    R -- "Phạm vi 0 -> (page, offset)" --> S
    R -- "Phạm vi 1 -> (page, offset)" --> S
```

Khi bảng mở rộng, kích thước của bản đồ phạm vi cũng tăng theo. Nếu bản đồ không còn vừa với các trang đã cấp phát, nó sẽ chiếm lấy trang tiếp theo, và tất cả các mục chỉ mục trước đó nằm trong trang này sẽ được chuyển sang các trang khác. Vì một trang có thể chứa rất nhiều con trỏ nên các lần di chuyển như vậy khá hiếm khi xảy ra.

Các trang chỉ mục BRIN có thể được hiển thị bằng extension `pageinspect` như thường lệ. Siêu dữ liệu bao gồm kích thước của phạm vi và số lượng trang được dành riêng cho bản đồ phạm vi:

```sql
=> SELECT pagesperrange, lastrevmappage
FROM brin_metapage_info(get_raw_page(
  'flights_bi_scheduled_time_idx', 0
));
 pagesperrange | lastrevmappage
---------------+----------------
           128 |              4
(1 row)
```

Ở đây, bản đồ phạm vi chiếm bốn trang, từ trang thứ nhất đến trang thứ tư. Chúng ta có thể xem các con trỏ trỏ tới các mục chỉ mục chứa dữ liệu tóm tắt:

```sql
=> SELECT *
FROM brin_revmap_data(get_raw_page(
  'flights_bi_scheduled_time_idx', 1
));
  pages
----------
 (6,197)
 (6,198)
 (6,199)
...
 (6,195)
 (6,196)
(1360 rows)
```

Nếu phạm vi chưa được tóm tắt (summarized), con trỏ trong bản đồ phạm vi sẽ là `NULL`.

Và dưới đây là các bản tóm tắt cho một vài phạm vi:

```sql
=> SELECT itemoffset, blknum, value
FROM brin_page_items(
  get_raw_page('flights_bi_scheduled_time_idx', 6),
  'flights_bi_scheduled_time_idx'
)
ORDER BY blknum
LIMIT 3 \gx
-[ RECORD 1 ]-------------------------------------------------------
itemoffset | 197
blknum     | 0
value      | {2016-08-15 02:45:00+03 .. 2016-08-15 16:20:00+03}
-[ RECORD 2 ]-------------------------------------------------------
itemoffset | 198
blknum     | 128
value      | {2016-08-15 05:50:00+03 .. 2016-08-15 18:55:00+03}
-[ RECORD 3 ]-------------------------------------------------------
itemoffset | 199
blknum     | 256
value      | {2016-08-15 07:15:00+03 .. 2016-08-15 18:50:00+03}
-[ RECORD 4 ]-------------------------------------------------------
itemoffset | 200
blknum     | 384
value      | {2016-08-15 07:55:00+03 .. 2016-08-15 20:20:00+03}
```

---

## 29.4 Tìm kiếm (29.4 Search)

Nếu một điều kiện truy vấn được hỗ trợ bởi chỉ mục BRIN,[^4] bộ thực thi (executor) sẽ quét bản đồ phạm vi và thông tin tóm tắt cho từng phạm vi. Nếu dữ liệu trong một phạm vi có khả năng khớp với khóa tìm kiếm, tất cả các trang thuộc phạm vi này sẽ được thêm vào bitmap. Vì BRIN không lưu giữ các TID của từng tuple riêng lẻ nên bitmap luôn ở trạng thái mất mát dữ liệu (lossy).

Việc đối chiếu dữ liệu với khóa tìm kiếm được thực hiện bởi **hàm nhất quán (consistency function)**, hàm này diễn giải thông tin tóm tắt của phạm vi. Các phạm vi chưa được tóm tắt (non-summarized ranges) luôn được thêm vào bitmap.

Bitmap nhận được sau đó được dùng để quét bảng theo cách thông thường (Bitmap Heap Scan). Điều quan trọng cần đề cập là việc đọc các trang heap diễn ra tuần tự, hết phạm vi khối này đến phạm vi khối khác, và kỹ thuật đọc trước (prefetching) được áp dụng.

[^4]: `backend/access/brin/brin.c`, hàm `bringetbitmap`

---

## 29.5 Cập nhật thông tin tóm tắt (29.5 Summary Information Updates)

### Chèn giá trị (Value Insertion)

Khi một tuple mới được thêm vào một trang heap, thông tin tóm tắt trong phạm vi chỉ mục tương ứng sẽ được cập nhật.[^5] Số thứ tự phạm vi được tính dựa trên số thứ tự trang bằng các phép toán số học đơn giản, và thông tin tóm tắt sau đó được định vị thông qua bản đồ phạm vi.

Để xác định xem thông tin tóm tắt hiện tại có cần mở rộng hay không, **hàm bổ sung (addition function)** được sử dụng. Nếu cần mở rộng và trang chỉ mục còn đủ không gian trống, việc cập nhật sẽ được thực hiện tại chỗ (in-place, không cần thêm một mục chỉ mục mới).

Giả sử chúng ta vừa thêm một tuple có giá trị `42` vào trang số `13`. Số thứ tự phạm vi được tính bằng phép chia lấy phần nguyên số thứ tự trang cho kích thước phạm vi. Giả sử kích thước phạm vi bằng `4` trang, ta có số thứ tự phạm vi là `3`; vì việc đánh số phạm vi bắt đầu từ 0 nên ta lấy con trỏ thứ tư trong bản đồ phạm vi. Giá trị nhỏ nhất trong phạm vi này là `31`, giá trị lớn nhất là `40`. Giá trị mới thêm vào nằm ngoài các giới hạn này, do đó giá trị lớn nhất được tăng lên thành `42` (`31 .. 42`).

Nếu không thể cập nhật tại chỗ, một mục mới sẽ được thêm vào, và bản đồ phạm vi sẽ được sửa đổi để trỏ tới mục mới đó.

[^5]: `backend/access/brin/brin.c`, hàm `brininsert`

### Tóm tắt phạm vi (Range Summarization)

Mọi điều vừa nêu ở trên áp dụng cho kịch bản khi một tuple mới xuất hiện trong một phạm vi đã được tóm tắt. Khi chỉ mục đang được xây dựng, tất cả các phạm vi hiện có đều được tóm tắt, nhưng khi bảng phát triển lớn lên, các trang mới có thể rơi ra ngoài các phạm vi này.

Nếu chỉ mục được tạo với tham số lưu trữ `autosummarize` được bật (mặc định là `off`), phạm vi mới sẽ được tóm tắt ngay lập tức. Nhưng trong các kho dữ liệu (data warehouse), nơi các hàng thường được thêm vào theo từng lô lớn (large batches) chứ không phải từng hàng một, chế độ này có thể làm chậm đáng kể quá trình chèn dữ liệu.

Theo mặc định, các phạm vi mới không được tóm tắt ngay lập tức. Điều này không ảnh hưởng đến tính đúng đắn của chỉ mục vì các phạm vi chưa có thông tin tóm tắt luôn được đưa vào quét. Việc tóm tắt được thực hiện bất đồng bộ, hoặc trong quá trình vacuum bảng, hoặc khi được khởi chạy thủ công bằng cách gọi hàm `brin_summarize_new_values` (hoặc hàm `brin_summarize_range` để xử lý một phạm vi đơn lẻ).

Quá trình tóm tắt phạm vi[^6] không khóa bảng đối với các thao tác cập nhật. Khi bắt đầu quá trình này, một mục giữ chỗ (placeholder entry) được chèn vào chỉ mục cho phạm vi đó. Nếu dữ liệu trong phạm vi bị thay đổi trong khi phạm vi đang được quét, mục giữ chỗ sẽ được cập nhật thông tin tóm tắt về các thay đổi này. Sau đó, **hàm hợp nhất (union function)** sẽ gộp dữ liệu này với thông tin tóm tắt của phạm vi tương ứng.

Về mặt lý thuyết, thông tin tóm tắt đôi khi có thể thu hẹp lại sau khi một số hàng bị xóa. Nhưng trong khi các chỉ mục GiST có thể phân phối lại dữ liệu sau khi tách trang (page split), thì thông tin tóm tắt của chỉ mục BRIN không bao giờ thu hẹp mà chỉ có thể mở rộng thêm. Việc thu hẹp thường không cần thiết ở đây vì kho dữ liệu thường chỉ dùng để nối thêm (append) dữ liệu mới. Bạn có thể xóa thủ công thông tin tóm tắt bằng cách gọi hàm `brin_desummarize_range` để phạm vi này được tóm tắt lại, nhưng không có manh mối nào cho biết phạm vi nào sẽ được hưởng lợi từ việc đó.

Do vậy, BRIN chủ yếu nhắm đến các bảng có kích thước rất lớn, ít có thao tác cập nhật mà chủ yếu chỉ thêm các hàng mới vào cuối tệp, hoặc hoàn toàn không bị cập nhật. Nó được sử dụng chủ yếu trong các kho dữ liệu để xây dựng các báo cáo phân tích.

[^6]: `backend/access/brin/brin.c`, hàm `summarize_range`

> 💡 **Ví dụ bổ sung (không có trong sách):**
> Sau khi nạp hàng loạt 10 triệu dòng mới vào bảng log có chỉ mục BRIN (với `autosummarize = off`), bạn có thể chủ động tóm tắt các phạm vi mới ngay lập tức mà không cần đợi `autovacuum`:
> ```sql
> SELECT brin_summarize_new_values('flights_bi_scheduled_time_idx');
> ```
> Hàm này trả về số lượng phạm vi mới vừa được tóm tắt.

---

## 29.6 Các lớp Minmax (29.6 Minmax Classes)

Đối với các kiểu dữ liệu cho phép so sánh giá trị, thông tin tóm tắt bao gồm ít nhất giá trị lớn nhất và giá trị nhỏ nhất. Các lớp toán tử tương ứng chứa từ `minmax` trong tên của chúng:[^7]

```sql
=> SELECT opcname
FROM pg_am am
JOIN pg_opclass opc ON opcmethod = am.oid
WHERE amname = 'brin'
  AND opcname LIKE '%minmax_ops'
ORDER BY opcname;
         opcname
--------------------------
 bit_minmax_ops
 bpchar_minmax_ops
 bytea_minmax_ops
...
 timetz_minmax_ops
 uuid_minmax_ops
 varbit_minmax_ops
(26 rows)
```

Dưới đây là các hàm hỗ trợ (support functions) của các lớp toán tử này:

```sql
=> SELECT amprocnum, amproc::regproc
FROM pg_am am
JOIN pg_opclass opc ON opcmethod = am.oid
JOIN pg_amproc amop ON amprocfamily = opcfamily
WHERE amname = 'brin'
  AND opcname = 'numeric_minmax_ops'
ORDER BY amprocnum;
 amprocnum |        amproc
-----------+-----------------------
         1 | brin_minmax_opcinfo
         2 | brin_minmax_add_value
         3 | brin_minmax_consistent
         4 | brin_minmax_union
(4 rows)
```

Hàm đầu tiên trả về siêu dữ liệu của lớp toán tử, và tất cả các hàm còn lại đều đã được mô tả: chúng chèn giá trị mới, kiểm tra tính nhất quán và thực hiện các phép hợp nhất.

Lớp `minmax` bao gồm các toán tử so sánh tương tự như chúng ta đã thấy ở B-tree:

```sql
=> SELECT amopopr::regoperator, oprcode::regproc, amopstrategy
FROM pg_am am
JOIN pg_opclass opc ON opcmethod = am.oid
JOIN pg_amop amop ON amopfamily = opcfamily
JOIN pg_operator opr ON opr.oid = amopopr
WHERE amname = 'brin'
  AND opcname = 'numeric_minmax_ops'
ORDER BY amopstrategy;
       amopopr       |  oprcode   | amopstrategy
---------------------+------------+--------------
 <(numeric,numeric)  | numeric_lt |            1
 <=(numeric,numeric) | numeric_le |            2
 =(numeric,numeric)  | numeric_eq |            3
 >=(numeric,numeric) | numeric_ge |            4
 >(numeric,numeric)  | numeric_gt |            5
(5 rows)
```

[^7]: `backend/access/brin/brin_minmax.c`

### Chọn cột để đánh chỉ mục (Choosing Columns to be Indexed)

Những cột nào thì hợp lý để đánh chỉ mục bằng lớp toán tử này? Như đã đề cập trước đó, các chỉ mục như vậy hoạt động tốt nếu vị trí vật lý của các hàng có sự tương quan (correlation) với thứ tự logic của các giá trị.

Hãy kiểm tra điều này cho ví dụ trên:

```sql
=> SELECT attname, correlation, n_distinct
FROM pg_stats
WHERE tablename = 'flights_bi'
ORDER BY correlation DESC NULLS LAST;
      attname       |  correlation   |  n_distinct
--------------------+----------------+--------------
 scheduled_time     |      0.9999949 |        25926
 actual_time        |      0.9999948 |        34469
 fare_conditions    |      0.7976897 |            3
 flight_type        |      0.4981733 |            2
 airport_utc_offset |      0.4440067 |           11
 aircraft_code      |     0.19249801 |            8
 airport_code       |    0.061483838 |          104
 seat_no            |   0.0024594965 |          461
 flight_no          |   0.0020146023 |          710
 passenger_id       | -0.00046121294 | 2.610987e+06
 passenger_name     |   -0.012388787 |         8618
 airport_coord      |                |            0
(12 rows)
```

Dữ liệu được sắp xếp theo thời gian (cả thời gian dự kiến và thời gian thực tế; hầu như không có sự khác biệt nào): các mục mới được thêm vào theo thứ tự thời gian, và vì dữ liệu không bị cập nhật hay xóa nên tất cả các hàng đều đi vào nhánh chính (main fork) của bảng một cách tuần tự, hết hàng này đến hàng khác.

Các cột `fare_conditions`, `flight_type` và `airport_utc_offset` có độ tương quan tương đối cao, nhưng chúng lưu trữ quá ít giá trị phân biệt.

Độ tương quan ở các cột khác quá thấp nên việc đánh chỉ mục chúng bằng lớp toán tử `minmax` không mang lại lợi ích gì.

### Kích thước phạm vi và hiệu quả tìm kiếm (Range Size and Search Efficiency)

Một kích thước phạm vi phù hợp có thể được xác định dựa trên số lượng trang dùng để lưu trữ các giá trị cụ thể.

Hãy xem xét cột `scheduled_time` và lấy thông tin về tất cả các chuyến bay được thực hiện trong 24 giờ. Trước tiên, chúng ta phải tìm hiểu xem dữ liệu liên quan đến khoảng thời gian này chiếm bao nhiêu trang bảng.

Để có được con số này, chúng ta có thể tận dụng thực tế là một TID bao gồm số thứ tự trang và vị trí bù (offset). Tiếc là không có hàm tích hợp sẵn nào để tách TID thành hai thành phần này, vì vậy chúng ta sẽ phải tự viết một hàm hơi cồng kềnh để ép kiểu thông qua biểu diễn văn bản:

```sql
=> CREATE FUNCTION tid2page(t tid) RETURNS integer
LANGUAGE sql
RETURN (t::text::point)[0]::integer;
```

Bây giờ chúng ta có thể xem các ngày được phân bố trong bảng như thế nào:

```sql
=> SELECT min(numblk), round(avg(numblk)) avg, max(numblk)
FROM (
  SELECT count(distinct tid2page(ctid)) numblk
  FROM flights_bi
  GROUP BY scheduled_time::date
) t;
 min  | avg  | max
------+------+------
 1192 | 1447 | 1512
(1 row)
```

Như có thể nhận thấy, phân phối dữ liệu không hoàn toàn đồng đều. Với kích thước phạm vi tiêu chuẩn là 128 trang, mỗi ngày sẽ chiếm từ 9 đến 12 phạm vi. Khi truy xuất dữ liệu cho một ngày cụ thể, quá trình quét chỉ mục sẽ trả về cả những hàng thực sự cần thiết lẫn một số hàng thuộc về các ngày khác vô tình rơi vào cùng các phạm vi đó. Kích thước phạm vi càng lớn, số lượng giá trị biên dư thừa phải đọc càng nhiều; chúng ta có thể thay đổi số lượng này bằng cách giảm hoặc tăng kích thước phạm vi.

Hãy thử nghiệm một truy vấn cho một ngày cụ thể (tôi đã tạo sẵn chỉ mục với các thiết lập mặc định). Để đơn giản, tôi sẽ tắt thực thi song song:

```sql
=> SET max_parallel_workers_per_gather = 0;
=> \set d '2016-08-15 02:45:00+03'
=> EXPLAIN (analyze, buffers, costs off, timing off, summary off)
SELECT * FROM flights_bi
WHERE scheduled_time >= :'d'::timestamptz
  AND scheduled_time < :'d'::timestamptz + interval '1 day';
                              QUERY PLAN
---------------------------------------------------------------------
 Bitmap Heap Scan on flights_bi (actual rows=81964 loops=1)
   Recheck Cond: ((scheduled_time >= '2016-08-15 02:45:00+03'::ti...
   Rows Removed by Index Recheck: 11606
   Heap Blocks: lossy=1536
   Buffers: shared hit=1561
   -> Bitmap Index Scan on flights_bi_scheduled_time_idx
        (actual rows=15360 loops=1)
        Index Cond: ((scheduled_time >= '2016-08-15 02:45:00+03'::...
        Buffers: shared hit=25
Planning:
  Buffers: shared hit=1
(11 rows)
```

Chúng ta có thể định nghĩa **hệ số hiệu quả (efficiency factor)** của một chỉ mục BRIN đối với một truy vấn cụ thể là tỷ số giữa số trang được bỏ qua trong lần quét chỉ mục và tổng số trang trong bảng. Nếu hệ số hiệu quả bằng 0, việc truy cập qua chỉ mục suy biến thành quét tuần tự (chưa tính đến chi phí phụ trội). Hệ số hiệu quả càng cao thì số trang phải đọc càng ít. Nhưng vì một số trang chứa dữ liệu cần trả về và không thể bỏ qua, hệ số hiệu quả luôn nhỏ hơn 1.

Trong trường hợp cụ thể này, hệ số hiệu quả là:

$$\frac{528417 - 1561}{528417} \approx 0.997$$

trong đó `528,417` là tổng số trang trong bảng.

Tuy nhiên, chúng ta không thể rút ra bất kỳ kết luận có ý nghĩa nào chỉ dựa trên một giá trị đơn lẻ. Ngay cả khi có dữ liệu đồng đều và độ tương quan lý tưởng, hiệu quả vẫn sẽ thay đổi vì ít nhất là ranh giới các phạm vi sẽ không khớp với ranh giới các trang. Chúng ta chỉ có thể có bức tranh toàn cảnh nếu coi hệ số hiệu quả là một biến ngẫu nhiên và phân tích phân phối của nó.

Đối với ví dụ của chúng ta, ta có thể chọn tất cả các ngày khác nhau trong năm, kiểm tra kế hoạch thực thi cho từng giá trị và tính toán thống kê dựa trên tập mẫu này. Chúng ta có thể dễ dàng tự động hóa quá trình này vì lệnh `EXPLAIN` có thể trả về kết quả ở định dạng `JSON`, rất thuận tiện để phân tích cú pháp (parse). Tôi sẽ không đưa toàn bộ mã nguồn vào đây, nhưng đoạn mã sau chứa tất cả các chi tiết quan trọng:

```sql
=> DO $$
DECLARE
  plan jsonb;
BEGIN
  EXECUTE
    'EXPLAIN (analyze, buffers, timing off, costs off, format json)
     SELECT * FROM flights_bi
     WHERE scheduled_time >= $1
       AND scheduled_time < $1 + interval ''1 day'''
  USING '2016-08-15 02:45:00+03'::timestamptz
  INTO plan;
  RAISE NOTICE 'shared hit=%, read=%',
    plan -> 0 -> 'Plan' ->> 'Shared Hit Blocks',
    plan -> 0 -> 'Plan' ->> 'Shared Read Blocks';
END;
$$;
NOTICE:  shared hit=1561, read=0
DO
```

Kết quả có thể được hiển thị trực quan dưới dạng biểu đồ hộp (box plot), còn gọi là "biểu đồ hộp và râu" (box-and-whiskers). Các "râu" (whiskers) biểu thị tứ phân vị thứ nhất và thứ tư (tức là râu bên phải chứa 25% giá trị lớn nhất, trong khi râu bên trái chứa 25% giá trị nhỏ nhất). Bản thân chiếc hộp chứa 50% giá trị còn lại và có đánh dấu giá trị trung vị (median). Quan trọng hơn, cách biểu diễn cô đọng này cho phép chúng ta so sánh trực quan các kết quả khác nhau.

Biểu đồ phân phối hệ số hiệu quả cho thấy với kích thước phạm vi mặc định (`128 pages/range`, `184 kB`) và hai kích thước khác nhỏ hơn 4 lần (`32 pages/range`, `529 kB`) và lớn hơn 4 lần (`512 pages/range`, `72 kB`): như chúng ta kỳ vọng, độ chính xác tìm kiếm và hiệu quả đều cao ngay cả đối với các phạm vi khá lớn (hệ số hiệu quả đều nằm trong khoảng `0.990` đến `1.000`, tiệm cận đường nét đứt biểu diễn giá trị trung bình của hệ số hiệu quả tối đa có thể cho truy vấn này, giả sử một ngày chiếm khoảng $\frac{1}{365}$ bảng).

Lưu ý rằng sự gia tăng hiệu quả phải trả giá bằng việc tăng kích thước chỉ mục. BRIN khá linh hoạt trong việc cho phép bạn tìm ra sự cân bằng giữa hai yếu tố này.

### Thuộc tính (Properties)

Các thuộc tính của BRIN được mã hóa cứng (hardwired) và không phụ thuộc vào các lớp toán tử.

#### Thuộc tính phương thức truy cập (Access Method Properties)

```sql
=> SELECT a.amname, p.name, pg_indexam_has_property(a.oid, p.name)
FROM pg_am a, unnest(array[
  'can_order', 'can_unique', 'can_multi_col',
  'can_exclude', 'can_include'
]) p(name)
WHERE a.amname = 'brin';
 amname |     name      | pg_indexam_has_property
--------+---------------+-------------------------
 brin   | can_order     | f
 brin   | can_unique    | f
 brin   | can_multi_col | t
 brin   | can_exclude   | f
 brin   | can_include   | f
(5 rows)
```

Rõ ràng, cả thuộc tính sắp xếp lẫn tính duy nhất đều không được hỗ trợ. Vì chỉ mục BRIN luôn trả về một bitmap nên các ràng buộc loại trừ (exclusion constraints) cũng không được hỗ trợ. Các cột bổ sung `INCLUDE` cũng không có ý nghĩa gì, vì ngay cả các khóa đánh chỉ mục (indexing keys) cũng không được lưu trữ trong chỉ mục BRIN.

Tuy nhiên, chúng ta có thể tạo một chỉ mục BRIN nhiều cột (multicolumn BRIN index). Trong trường hợp này, thông tin tóm tắt cho mỗi cột được thu thập và lưu trữ trong một mục chỉ mục riêng biệt, nhưng chúng vẫn dùng chung một bản đồ phạm vi. Một chỉ mục như vậy hữu ích nếu cùng một kích thước phạm vi có thể áp dụng cho tất cả các cột được đánh chỉ mục.

Ngoài ra, chúng ta có thể tạo các chỉ mục BRIN riêng biệt cho nhiều cột và tận dụng việc các bitmap có thể được gộp lại với nhau (thông qua nút `BitmapAnd`). Ví dụ:

```sql
=> CREATE INDEX ON flights_bi USING brin(airport_utc_offset);
=> EXPLAIN (analyze, costs off, timing off, summary off)
SELECT *
FROM flights_bi
WHERE scheduled_time >= :'d'::timestamptz
  AND scheduled_time < :'d'::timestamptz + interval '1 day'
  AND airport_utc_offset = '08:00:00';
                              QUERY PLAN
---------------------------------------------------------------------
 Bitmap Heap Scan on flights_bi (actual rows=1658 loops=1)
   Recheck Cond: ((scheduled_time >= '2016-08-15 02:45:00+03'::ti...
   Rows Removed by Index Recheck: 14077
   Heap Blocks: lossy=256
   -> BitmapAnd (actual rows=0 loops=1)
        -> Bitmap Index Scan on flights_bi_scheduled_time_idx (act...
             Index Cond: ((scheduled_time >= '2016-08-15 02:45:00+0...
        -> Bitmap Index Scan on flights_bi_airport_utc_offset_idx ...
             Index Cond: (airport_utc_offset = '08:00:00'::interval)
(9 rows)
```

#### Thuộc tính cấp chỉ mục (Index-Level Properties)

```sql
=> SELECT p.name, pg_index_has_property(
  'flights_bi_scheduled_time_idx', p.name
)
FROM unnest(array[
  'clusterable', 'index_scan', 'bitmap_scan', 'backward_scan'
]) p(name);
     name      | pg_index_has_property
---------------+-----------------------
 clusterable   | f
 index_scan    | f
 bitmap_scan   | t
 backward_scan | f
(4 rows)
```

Rõ ràng, quét bitmap (bitmap scan) là kiểu truy cập duy nhất được hỗ trợ.

Việc thiếu khả năng phân cụm (`clusterable = f`) thoạt nhìn có vẻ khó hiểu. Vì BRIN nhạy cảm với thứ tự vật lý của các hàng, thật hợp lý khi cho rằng nó nên hỗ trợ sắp xếp lại (reordering) để tối đa hóa hiệu quả. Nhưng việc phân cụm (`CLUSTER`) các bảng lớn dù sao cũng là một sự xa xỉ, nếu tính đến toàn bộ quá trình xử lý và dung lượng đĩa bổ sung cần thiết để xây dựng lại bảng. Bên cạnh đó, như ví dụ về bảng `flights_bi` cho thấy, một số dạng sắp xếp trong kho dữ liệu có thể xuất hiện một cách tự nhiên.

#### Thuộc tính cấp cột (Column-Level Properties)

```sql
=> SELECT p.name, pg_index_column_has_property(
  'flights_bi_scheduled_time_idx', 1, p.name
)
FROM unnest(array[
  'orderable', 'distance_orderable', 'returnable',
  'search_array', 'search_nulls'
]) p(name);
        name        | pg_index_column_has_property
--------------------+------------------------------
 orderable          | f
 distance_orderable | f
 returnable         | f
 search_array       | f
 search_nulls       | t
(5 rows)
```

Thuộc tính cấp cột duy nhất khả dụng là hỗ trợ `NULL`. Để theo dõi các giá trị `NULL` trong một phạm vi, thông tin tóm tắt cung cấp một thuộc tính riêng biệt:

```sql
=> SELECT hasnulls, allnulls, value
FROM brin_page_items(
  get_raw_page('flights_bi_airport_utc_offset_idx', 6),
  'flights_bi_airport_utc_offset_idx'
)
WHERE itemoffset = 1;
 hasnulls | allnulls |         value
----------+----------+------------------------
 f        | f        | {03:00:00 .. 03:00:00}
(1 row)
```

---

## 29.7 Các lớp Minmax-Multi (29.7 Minmax-Multi Classes)

*(Từ phiên bản 14)*

Độ tương quan đã được thiết lập có thể dễ dàng bị phá vỡ bởi các thao tác cập nhật dữ liệu. Nguyên nhân không nằm ở việc sửa đổi thực tế một giá trị cụ thể mà nằm ở chính thiết kế MVCC: phiên bản cũ của một hàng có thể bị đánh dấu xóa ở một trang, trong khi phiên bản mới của nó có thể được chèn vào bất kỳ vị trí nào đang còn trống, do đó thứ tự hàng ban đầu không thể được bảo toàn.

Để giảm thiểu phần nào hiệu ứng này, chúng ta có thể giảm giá trị của tham số lưu trữ `fillfactor` nhằm để lại nhiều không gian hơn trong trang cho các cập nhật trong tương lai. Nhưng liệu có đáng để làm phình to kích thước của một bảng vốn đã khổng lồ hay không? Bên cạnh đó, các thao tác xóa dù sao cũng sẽ giải phóng một số không gian trong các trang hiện có, từ đó tạo ra những "cái bẫy" cho các tuple mới mà lẽ ra sẽ đi vào cuối tệp.

Tình huống như vậy có thể dễ dàng được mô phỏng. Hãy xóa ngẫu nhiên 0,1% số hàng và chạy `VACUUM` trên bảng để dọn trống một ít không gian cho các tuple mới:

```sql
=> WITH t AS (
  SELECT ctid
  FROM flights_bi TABLESAMPLE BERNOULLI(0.1) REPEATABLE(0)
)
DELETE FROM flights_bi
WHERE ctid IN (SELECT ctid FROM t);
DELETE 30180

=> VACUUM flights_bi;
```

Bây giờ hãy thêm một số dữ liệu cho một ngày mới ở một trong các múi giờ. Tôi sẽ chỉ đơn giản sao chép dữ liệu của ngày hôm trước:

```sql
=> INSERT INTO flights_bi
SELECT airport_code, airport_coord, airport_utc_offset,
       flight_no, flight_type, scheduled_time + interval '1 day',
       actual_time + interval '1 day', aircraft_code, seat_no,
       fare_conditions, passenger_id, passenger_name
FROM flights_bi
WHERE date_trunc('day', scheduled_time) = '2017-08-15'
  AND airport_utc_offset = '03:00:00';
INSERT 0 40532
```

Thao tác xóa vừa thực hiện đã đủ để giải phóng một ít không gian ở tất cả hoặc gần như tất cả các phạm vi. Khi lọt vào các trang nằm đâu đó ở giữa tệp, các tuple mới đã tự động làm phình to các phạm vi. Ví dụ, thông tin tóm tắt liên quan đến phạm vi đầu tiên trước đây chỉ bao phủ chưa đầy một ngày, nhưng giờ đây nó bao trùm cả một năm:

```sql
=> SELECT value
FROM brin_page_items(
  get_raw_page('flights_bi_scheduled_time_idx', 6),
  'flights_bi_scheduled_time_idx'
)
WHERE blknum = 0;
                        value
------------------------------------------------------
 {2016-08-15 02:45:00+03 .. 2017-08-16 09:35:00+03}
(1 row)
```

Ngày được chỉ định trong truy vấn càng nhỏ, càng nhiều phạm vi phải bị quét. Biểu đồ hiệu quả cho thấy mức độ thảm họa của tình huống này: với lớp `minmax` thông thường (`128 pages/range`, `248 kB`), hệ số hiệu quả tụt xuống gần như trải dài từ 0 đến 1 (trung bình thấp hơn rất nhiều so với trước đó).

Để giải quyết vấn đề này, chúng ta phải làm cho thông tin tóm tắt tinh vi hơn một chút: thay vì một khoảng liên tục duy nhất, chúng ta phải lưu trữ **nhiều khoảng nhỏ hơn** sao cho khi gộp lại chúng bao phủ toàn bộ các giá trị. Khi đó, một trong các khoảng có thể bao phủ tập dữ liệu chính, trong khi các khoảng còn lại sẽ xử lý các giá trị ngoại lai (outliers) thỉnh thoảng xuất hiện.

Chức năng này được cung cấp bởi các lớp toán tử **minmax-multi**:[^8]

```sql
=> SELECT opcname
FROM pg_am am
JOIN pg_opclass opc ON opcmethod = am.oid
WHERE amname = 'brin'
  AND opcname LIKE '%minmax_multi_ops'
ORDER BY opcname;
            opcname
-------------------------------
 date_minmax_multi_ops
 float4_minmax_multi_ops
 float8_minmax_multi_ops
 inet_minmax_multi_ops
...
 time_minmax_multi_ops
 timestamp_minmax_multi_ops
 timestamptz_minmax_multi_ops
 timetz_minmax_multi_ops
 uuid_minmax_multi_ops
(19 rows)
```

So với các lớp toán tử `minmax`, các lớp `minmax-multi` có thêm một hàm hỗ trợ dùng để tính khoảng cách giữa các giá trị; nó được dùng để xác định độ dài khoảng, thứ mà lớp toán tử cố gắng thu nhỏ:

```sql
=> SELECT amprocnum, amproc::regproc
FROM pg_am am
JOIN pg_opclass opc ON opcmethod = am.oid
JOIN pg_amproc amop ON amprocfamily = opcfamily
WHERE amname = 'brin'
  AND opcname = 'numeric_minmax_multi_ops'
ORDER BY amprocnum;
 amprocnum |               amproc
-----------+-------------------------------------
         1 | brin_minmax_multi_opcinfo
         2 | brin_minmax_multi_add_value
         3 | brin_minmax_multi_consistent
         4 | brin_minmax_multi_union
         5 | brin_minmax_multi_options
        11 | brin_minmax_multi_distance_numeric
(6 rows)
```

Các toán tử của những lớp này hoàn toàn giống với các toán tử của lớp `minmax`.

Các lớp `minmax-multi` có thể nhận tham số `values_per_range` (mặc định là `32`), xác định số lượng tối đa các giá trị tóm tắt được phép trong mỗi phạm vi. Một giá trị tóm tắt được biểu diễn bởi hai con số (một khoảng), trong khi một điểm riêng lẻ chỉ cần một con số. Nếu không đủ chỗ cho các giá trị, một số khoảng gần nhau nhất sẽ được gộp/thu gọn lại.[^9]

Hãy xây dựng một chỉ mục `minmax-multi` thay thế cho chỉ mục hiện có. Chúng ta sẽ giới hạn số lượng giá trị tối đa trên mỗi phạm vi là `16`:

```sql
=> DROP INDEX flights_bi_scheduled_time_idx;
=> CREATE INDEX ON flights_bi USING brin(
  scheduled_time timestamptz_minmax_multi_ops(
    values_per_range = 16
  )
);
```

Biểu đồ so sánh cho thấy chỉ mục mới (`minmax-multi`, `656 kB`) đưa hệ số hiệu quả quay trở lại mức ban đầu (gần `1.000`), trong khi `minmax` (`184 kB`) bị suy giảm nghiêm trọng. Đúng như dự đoán, điều này dẫn đến sự gia tăng kích thước chỉ mục.

[^8]: `backend/access/brin/brin_minmax_multi.c`
[^9]: `backend/access/brin/brin_minmax_multi.c`, hàm `reduce_expanded_ranges`

---

## 29.8 Các lớp Inclusion (29.8 Inclusion Classes)

Sự khác biệt giữa các lớp toán tử `minmax` và `inclusion` gần giống với sự khác biệt giữa B-tree và GiST: nhóm sau được thiết kế cho các kiểu dữ liệu không hỗ trợ các phép toán so sánh (`<`, `>`), mặc dù sự sắp xếp gần nhau về mặt vật lý của các giá trị vẫn có ý nghĩa đối với chúng. Thông tin tóm tắt cho một phạm vi cụ thể do các lớp toán tử `inclusion` cung cấp được biểu diễn bằng **hộp bao (bounding box)** của các giá trị trong phạm vi này.

Dưới đây là các lớp toán tử đó; số lượng của chúng không nhiều:

```sql
=> SELECT opcname
FROM pg_am am
JOIN pg_opclass opc ON opcmethod = am.oid
WHERE amname = 'brin'
  AND opcname LIKE '%inclusion_ops'
ORDER BY opcname;
       opcname
---------------------
 box_inclusion_ops
 inet_inclusion_ops
 range_inclusion_ops
(3 rows)
```

Danh sách các hàm hỗ trợ được mở rộng thêm một hàm bắt buộc dùng để hợp nhất hai giá trị, cùng với một số hàm tùy chọn khác:

```sql
=> SELECT amprocnum, amproc::regproc
FROM pg_am am
JOIN pg_opclass opc ON opcmethod = am.oid
JOIN pg_amproc amop ON amprocfamily = opcfamily
WHERE amname = 'brin'
  AND opcname = 'box_inclusion_ops'
ORDER BY amprocnum;
 amprocnum |           amproc
-----------+----------------------------
         1 | brin_inclusion_opcinfo
         2 | brin_inclusion_add_value
         3 | brin_inclusion_consistent
         4 | brin_inclusion_union
        11 | bound_box
        13 | box_contain
(6 rows)
```

Khi làm việc với các giá trị có thể so sánh được, chúng ta dựa vào độ tương quan (`correlation`) của chúng; nhưng đối với các kiểu dữ liệu khác, không có thống kê nào như vậy được thu thập,[^10] nên rất khó dự đoán hiệu quả của một chỉ mục BRIN dựa trên `inclusion`.

Tệ hơn nữa, độ tương quan ảnh hưởng lớn đến việc ước lượng chi phí (cost estimation) của quá trình quét chỉ mục. Nếu thống kê này không khả dụng, nó sẽ bị coi là bằng `0`.[^11] Do đó, bộ lập kế hoạch (planner) không có cách nào phân biệt giữa các chỉ mục `inclusion` chính xác và các chỉ mục bị mờ nhạt (fuzzy), nên thông thường nó sẽ tránh sử dụng chúng hoàn toàn. (PostGIS từ phiên bản 3.1.0 đã thu thập thống kê về độ tương quan của dữ liệu không gian.)

Trong trường hợp cụ thể này, chúng ta có thể suy đoán rằng việc xây dựng một chỉ mục trên tọa độ sân bay là hợp lý, vì kinh độ phải có sự tương quan với múi giờ.

Không giống như các vị từ GiST, thông tin tóm tắt của BRIN có cùng kiểu dữ liệu với dữ liệu được đánh chỉ mục; do đó, không dễ để xây dựng chỉ mục cho kiểu điểm (`point`). Nhưng chúng ta có thể tạo một chỉ mục trên biểu thức (expression index) bằng cách chuyển đổi các điểm thành các hình chữ nhật giả (dummy rectangles):

```sql
=> CREATE INDEX ON flights_bi USING brin(box(airport_coord))
WITH (pages_per_range = 8);
=> SELECT pg_size_pretty(pg_total_relation_size(
  'flights_bi_box_idx'
));
 pg_size_pretty
----------------
 3816 kB
(1 row)
```

Một chỉ mục được xây dựng trên múi giờ với cùng kích thước phạm vi chiếm dung lượng xấp xỉ nhau (`3288 kB`).

Các toán tử có trong lớp này tương tự như các toán tử GiST. Ví dụ, một chỉ mục BRIN có thể được dùng để tăng tốc tìm kiếm các điểm nằm trong một khu vực nhất định:

```sql
=> SELECT airport_code, airport_name
FROM airports
WHERE box(coordinates) <@ box '135,45,140,50';
 airport_code |       airport_name
--------------+-------------------------
 KHV          | Khabarovsk-Novy Airport
(1 row)
```

Nhưng như đã đề cập trước đó, bộ lập kế hoạch từ chối sử dụng quét chỉ mục trừ khi chúng ta tắt quét tuần tự:

```sql
=> EXPLAIN (costs off)
SELECT *
FROM flights_bi
WHERE box(airport_coord) <@ box '135,45,140,50';
                         QUERY PLAN
------------------------------------------------------------
 Seq Scan on flights_bi
   Filter: (box(airport_coord) <@ '(140,50),(135,45)'::box)
(2 rows)

=> SET enable_seqscan = off;
=> EXPLAIN (analyze, costs off, timing off, summary off)
SELECT *
FROM flights_bi
WHERE box(airport_coord) <@ box '135,45,140,50';
                              QUERY PLAN
---------------------------------------------------------------------
 Bitmap Heap Scan on flights_bi (actual rows=511414 loops=1)
   Recheck Cond: (box(airport_coord) <@ '(140,50),(135,45)'::box)
   Rows Removed by Index Recheck: 630756
   Heap Blocks: lossy=19656
   -> Bitmap Index Scan on flights_bi_box_idx (actual rows=196560...
        Index Cond: (box(airport_coord) <@ '(140,50),(135,45)'::box)
(6 rows)

=> RESET enable_seqscan;
```

[^10]: `backend/commands/analyze.c`, hàm `compute_scalar_stats`
[^11]: `backend/utils/adt/selfuncs.c`, hàm `brincostestimate`

---

## 29.9 Các lớp Bloom (29.9 Bloom Classes)

*(Từ phiên bản 14)*

Các lớp toán tử dựa trên **bộ lọc Bloom (Bloom filter)** cho phép sử dụng BRIN cho bất kỳ kiểu dữ liệu nào hỗ trợ phép toán bằng (`=`) và có định nghĩa hàm băm (hash function). Chúng cũng có thể được áp dụng cho các kiểu dữ liệu có thứ tự thông thường nếu các giá trị được cục bộ hóa trong từng phạm vi riêng biệt nhưng vị trí vật lý của chúng không có sự tương quan với thứ tự logic.

Tên của các lớp toán tử này chứa từ `bloom`:[^12]

```sql
=> SELECT opcname
FROM pg_am am
JOIN pg_opclass opc ON opcmethod = am.oid
WHERE amname = 'brin'
  AND opcname LIKE '%bloom_ops'
ORDER BY opcname;
         opcname
-------------------------
 bpchar_bloom_ops
 bytea_bloom_ops
 char_bloom_ops
...
 timestamptz_bloom_ops
 timetz_bloom_ops
 uuid_bloom_ops
(24 rows)
```

Bộ lọc Bloom cổ điển là một cấu trúc dữ liệu cho phép bạn kiểm tra nhanh xem một phần tử có thuộc về một tập hợp hay không. Bộ lọc này rất nhỏ gọn, nhưng nó cho phép **dương tính giả (false positives)**: một tập hợp có thể bị cho là chứa nhiều phần tử hơn thực tế. Nhưng điều quan trọng hơn là **âm tính giả (false negatives) hoàn toàn bị loại trừ**: bộ lọc không bao giờ kết luận rằng một phần tử không có mặt trong tập hợp nếu thực tế nó có ở đó.

Bộ lọc là một mảng gồm $m$ bit (còn gọi là *chữ ký* — signature), ban đầu được điền toàn số `0`. Chúng ta chọn $k$ hàm băm khác nhau để ánh xạ bất kỳ phần tử nào của tập hợp tới $k$ bit trong chữ ký. Khi một phần tử được thêm vào tập hợp, mỗi bit tương ứng trong chữ ký được đặt thành `1`. Do đó, nếu tất cả các bit tương ứng với một phần tử đều bằng `1`, phần tử đó *có thể* có mặt trong tập hợp; nếu có ít nhất một bit bằng `0`, phần tử đó *chắc chắn* vắng mặt.

Trong trường hợp của chỉ mục BRIN, bộ lọc xử lý tập hợp các giá trị của cột được đánh chỉ mục thuộc về một phạm vi cụ thể; thông tin tóm tắt cho phạm vi này được biểu diễn bởi bộ lọc Bloom đã xây dựng.

> 📝 **Giải thích thêm (không có trong sách):**
> Extension `bloom`[^13] cung cấp phương thức truy cập chỉ mục riêng dựa trên bộ lọc Bloom. Nó xây dựng một bộ lọc cho *mỗi hàng* trong bảng và xử lý tập hợp các giá trị cột của mỗi hàng. Một chỉ mục như vậy được thiết kế để đánh chỉ mục nhiều cột cùng lúc và có thể dùng trong các truy vấn ngẫu hứng (adhoc queries), khi chưa biết trước những cột nào sẽ được tham chiếu trong điều kiện lọc. Một chỉ mục BRIN cũng có thể được xây dựng trên nhiều cột, nhưng thông tin tóm tắt của nó sẽ chứa nhiều bộ lọc Bloom độc lập cho từng cột đó.

Độ chính xác của bộ lọc Bloom phụ thuộc vào độ dài chữ ký. Về mặt lý thuyết, số bit chữ ký tối ưu có thể được ước tính theo công thức:

$$m = \frac{-n \log_2 p}{\ln 2}$$

trong đó $n$ là số phần tử trong tập hợp và $p$ là xác suất dương tính giả.

Hai thiết lập này có thể được điều chỉnh bằng các tham số lớp toán tử tương ứng:

- `n_distinct_per_range` (mặc định `-0.1`): xác định số lượng phần tử trong một tập hợp; trong trường hợp này, đó là số lượng giá trị phân biệt trong một phạm vi của cột được đánh chỉ mục. Giá trị tham số này được diễn giải giống như thống kê về các giá trị phân biệt: các giá trị âm biểu thị *tỷ lệ* (fraction) số hàng trong phạm vi chứ không phải số lượng tuyệt đối.
- `false_positive_rate` (mặc định `0.01`): xác định xác suất dương tính giả. Một giá trị gần bằng `0` có nghĩa là quá trình quét chỉ mục gần như chắc chắn sẽ bỏ qua các phạm vi không có giá trị cần tìm. Nhưng nó không đảm bảo tìm kiếm chính xác tuyệt đối, vì các phạm vi được quét vẫn sẽ chứa thêm các hàng thừa không khớp với truy vấn. Hành vi như vậy là do độ rộng của phạm vi và vị trí vật lý của dữ liệu chứ không phải do bản thân thuộc tính của bộ lọc.

Danh sách các hàm hỗ trợ được mở rộng thêm một hàm băm:

```sql
=> SELECT amprocnum, amproc::regproc
FROM pg_am am
JOIN pg_opclass opc ON opcmethod = am.oid
JOIN pg_amproc amop ON amprocfamily = opcfamily
WHERE amname = 'brin'
  AND opcname = 'numeric_bloom_ops'
ORDER BY amprocnum;
 amprocnum |        amproc
-----------+-----------------------
         1 | brin_bloom_opcinfo
         2 | brin_bloom_add_value
         3 | brin_bloom_consistent
         4 | brin_bloom_union
         5 | brin_bloom_options
        11 | hash_numeric
(6 rows)
```

Vì bộ lọc Bloom dựa trên phép băm nên chỉ có toán tử bằng (`=`) được hỗ trợ:

```sql
=> SELECT amopopr::regoperator, oprcode::regproc, amopstrategy
FROM pg_am am
JOIN pg_opclass opc ON opcmethod = am.oid
JOIN pg_amop amop ON amopfamily = opcfamily
JOIN pg_operator opr ON opr.oid = amopopr
WHERE amname = 'brin'
  AND opcname = 'numeric_bloom_ops'
ORDER BY amopstrategy;
       amopopr       |  oprcode   | amopstrategy
---------------------+------------+--------------
 =(numeric,numeric)  | numeric_eq |            1
(1 row)
```

Hãy lấy cột `flight_no` lưu trữ số hiệu chuyến bay; cột này có độ tương quan gần bằng 0, nên nó vô dụng đối với lớp toán tử phạm vi thông thường. Chúng ta sẽ giữ nguyên thiết lập dương tính giả mặc định; còn đối với số lượng giá trị phân biệt trong một phạm vi, có thể dễ dàng tính toán được. Ví dụ, đối với một phạm vi 8 trang, ta sẽ nhận được giá trị sau:

```sql
=> SELECT max(nd)
FROM (
  SELECT count(distinct flight_no) nd
  FROM flights_bi
  GROUP BY tid2page(ctid) / 8
) t;
 max
-----
  22
(1 row)
```

Đối với các phạm vi nhỏ hơn, con số này thậm chí sẽ còn thấp hơn (nhưng trong mọi trường hợp, lớp toán tử không cho phép các giá trị nhỏ hơn `16`).

Chúng ta chỉ cần tạo một chỉ mục và kiểm tra kế hoạch thực thi:

```sql
=> CREATE INDEX ON flights_bi USING brin(
  flight_no bpchar_bloom_ops(n_distinct_per_range = 22)
)
WITH (pages_per_range = 8);
=> EXPLAIN (analyze, costs off, timing off, summary off)
SELECT *
FROM flights_bi
WHERE flight_no = 'PG0001';
                              QUERY PLAN
---------------------------------------------------------------------
 Bitmap Heap Scan on flights_bi (actual rows=5192 loops=1)
   Recheck Cond: (flight_no = 'PG0001'::bpchar)
   Rows Removed by Index Recheck: 122894
   Heap Blocks: lossy=2168
   -> Bitmap Index Scan on flights_bi_flight_no_idx (actual rows=...
        Index Cond: (flight_no = 'PG0001'::bpchar)
(6 rows)

=> RESET max_parallel_workers_per_gather;
```

Biểu đồ hiệu quả cho thấy đối với một số số hiệu chuyến bay (được biểu diễn bằng các điểm ngoại lai riêng lẻ không nằm trong các râu), chỉ mục không hoạt động tốt lắm, nhưng nhìn chung hiệu quả tổng thể là khá cao (hệ số hiệu quả trung vị đạt khoảng `0.95` trở lên cho các cấu hình `2 pages/range` ~ `14,8 MB`, `4 pages/range` ~ `7,4 MB`, và `8 pages/range` ~ `3,7 MB`).

[^12]: `backend/access/brin/brin_bloom.c`
[^13]: `postgresql.org/docs/14/bloom.html`

> 💡 **Ví dụ bổ sung (không có trong sách):**
> Một ứng dụng điển hình của BRIN Bloom trong thực tế là đánh chỉ mục cột `tenant_id` hoặc `device_id` (kiểu `uuid`) trong bảng dữ liệu chuỗi thời gian (time-series) rất lớn, nơi dữ liệu được nạp theo từng lô của từng khách hàng hoặc thiết bị tại mỗi thời điểm nhưng bản thân các `uuid` không có thứ tự tăng dần:
> ```sql
> CREATE INDEX events_tenant_brin_bloom_idx
>   ON events USING brin (tenant_id uuid_bloom_ops(n_distinct_per_range = 32, false_positive_rate = 0.005))
>   WITH (pages_per_range = 32);
> ```
> Chỉ mục này vẫn giữ kích thước cực nhỏ so với B-tree trên cột `uuid`, trong khi vẫn cho phép bỏ qua phần lớn các phạm vi khối không chứa `tenant_id` cần tìm.

---

# LỜI KẾT (Conclusion)

Vậy là hành trình của chúng ta đã đi đến hồi kết. Tôi hy vọng bạn thấy cuốn sách này hữu ích — hoặc ít nhất là thú vị — và đã học được điều gì đó mới mẻ từ nó (bản thân tôi cũng đã học được rất nhiều trong quá trình viết sách).

Phần lớn thông tin được đề cập có khả năng vẫn sẽ còn phù hợp trong một thời gian khá dài, nhưng một số chi tiết chắc chắn sẽ thay đổi rất nhanh. Tôi tin rằng giá trị lớn nhất của cuốn sách này không phải là một tập hợp các dữ kiện cụ thể, mà là cách tiếp cận để khám phá hệ thống mà tôi đã trình bày.

Không nên mặc nhiên coi cuốn sách này hay tài liệu chính thức là chân lý tuyệt đối. Hãy suy ngẫm, thực nghiệm, tự mình kiểm chứng mọi sự thật: PostgreSQL cung cấp tất cả các công cụ bạn cần cho việc đó, và tôi đã cố gắng chỉ ra cách sử dụng chúng. Việc này thường dễ gần như đặt câu hỏi trên diễn đàn hay tìm kiếm câu trả lời trên Google, nhưng chắc chắn đáng tin cậy và hữu ích hơn nhiều.

Cũng vì lý do đó, tôi muốn khuyến khích bạn nhìn sâu vào mã nguồn. Đừng để sự phức tạp của nó làm bạn chùn bước: hãy cứ thử nghiệm. Mã nguồn mở là một lợi thế tuyệt vời, vì vậy hãy tận dụng cơ hội này.

Tôi rất vui khi nhận được phản hồi của bạn; bạn có thể gửi bình luận và góp ý tới `edu@postgrespro.ru`. Tôi dự định sẽ cập nhật cuốn sách thường xuyên, vì vậy những đóng góp đó thực sự có thể giúp tôi cải thiện các ấn bản trong tương lai. Phiên bản trực tuyến mới nhất của cuốn sách được cung cấp miễn phí tại `postgrespro.com/community/books/internals`.

Chúc bạn may mắn!

---

## Tóm tắt mục lục tra cứu (Index Summary Note)

*(Ghi chú: Phần **Index** nguyên bản từ trang 541–548 trong sách là bảng chỉ mục tra cứu từ khóa theo thứ tự bảng chữ cái A–X kèm số trang trong ấn bản tiếng Anh, không chứa nội dung văn bản kỹ thuật mới. Các thuật ngữ và khái niệm liên quan đến Chương 29 như `autosummarize`, `Bloom filter`, `BRIN`, `efficiency factor`, `pages_per_range`, `n_distinct_per_range`, `false_positive_rate`, `values_per_range`, `pageinspect`, `correlation`, `fillfactor` đều đã được trình bày và giải thích đầy đủ trong nội dung chương này.)*

---

## Bảng thuật ngữ chương (Chapter Glossary)

| Thuật ngữ (English) | Tiếng Việt | Giải thích ngắn gọn |
|---|---|---|
| Block Range Index (BRIN) | Chỉ mục phạm vi khối | Loại chỉ mục lưu thông tin tóm tắt cho từng nhóm trang liên tiếp nhằm loại bỏ nhanh các phạm vi không khớp điều kiện lọc. |
| Block range | Phạm vi khối | Tập hợp các trang vật lý liên tiếp trong bảng (mặc định 128 trang) tương ứng với một mục tóm tắt trong BRIN. |
| Range map / Reverse map (`revmap`) | Bản đồ phạm vi / Bản đồ đảo ngược | Cấu trúc mảng các con trỏ nằm sau metapage, ánh xạ số thứ tự phạm vi tới vị trí mục tóm tắt của nó. |
| Metapage | Trang siêu dữ liệu | Trang số 0 của chỉ mục BRIN, lưu kích thước phạm vi (`pagesperrange`) và trang cuối của `revmap`. |
| Lossy bitmap | Bitmap mất mát dữ liệu | Bitmap chỉ đánh dấu ở mức trang (không phải từng dòng), buộc bộ thực thi phải kiểm tra lại (`Recheck Cond`) mọi hàng trong trang. |
| Consistency function | Hàm nhất quán | Hàm hỗ trợ của lớp toán tử dùng để kiểm tra xem thông tin tóm tắt của một phạm vi có khả năng khớp khóa tìm kiếm hay không. |
| Correlation | Độ tương quan | Mức độ tương đồng giữa thứ tự vật lý của các hàng trên đĩa và thứ tự logic của giá trị cột (`pg_stats.correlation`). |
| Efficiency factor | Hệ số hiệu quả | Tỷ lệ số trang được bỏ qua khi quét chỉ mục BRIN trên tổng số trang của bảng. |
| Minmax operator class | Lớp toán tử Minmax | Lớp toán tử lưu giá trị nhỏ nhất và lớn nhất trong mỗi phạm vi, phù hợp cho dữ liệu có thứ tự và độ tương quan cao. |
| Minmax-multi operator class | Lớp toán tử Minmax-Multi | Phiên bản mở rộng (từ PostgreSQL 14) lưu nhiều khoảng nhỏ thay vì một khoảng đơn lẻ để chống suy giảm hiệu quả khi có giá trị ngoại lai. |
| Inclusion operator class | Lớp toán tử Inclusion | Lớp toán tử lưu hộp bao (bounding box) cho các kiểu dữ liệu không có thứ tự tuyến tính như `box`, `inet`, `range`. |
| Bloom operator class | Lớp toán tử Bloom | Lớp toán tử dùng bộ lọc Bloom trên từng phạm vi để hỗ trợ truy vấn bằng (`=`) khi dữ liệu cục bộ hóa nhưng không có tương quan thứ tự. |
| False positive / False negative | Dương tính giả / Âm tính giả | Bộ lọc Bloom có thể báo nhầm là có mặt (dương tính giả) nhưng không bao giờ bỏ sót giá trị thực sự có mặt (không có âm tính giả). |
| Summarization (`autosummarize`) | Tóm tắt phạm vi | Quá trình quét các trang mới của một phạm vi để tạo bản tóm tắt; mặc định chạy bất đồng bộ qua `VACUUM` hoặc gọi hàm thủ công. |
