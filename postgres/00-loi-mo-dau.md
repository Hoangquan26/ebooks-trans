# PostgreSQL 14 Internals — Lời mở đầu & Mục lục (Front Matter & About This Book)

**Tác giả:** Egor Rogov  
**Tên sách:** *PostgreSQL 14 Internals*  
**Nhà xuất bản:** Postgres Professional  
**Nơi xuất bản:** Moscow, 2023  

---

Hình chú voi trên bìa sách là một phần trích từ bức minh họa trong cuốn *The History of Four-footed Beasts and Serpents* của Edward Topsell, xuất bản tại London năm 1658.

*PostgreSQL 14 Internals*  
Tác giả: Egor Rogov  
Dịch từ tiếng Nga sang tiếng Anh: Liudmila Mantrova  
© Postgres Professional, 2022–2023  
ISBN 978-5-6045970-4-0  
Phiên bản PDF của cuốn sách này có tại: `postgrespro.com/community/books/internals`

---

## Tóm tắt mục lục (Contents at a Glance)

* **Về cuốn sách này (About This Book)** — trang 15
* **Chương 1: Giới thiệu (Introduction)** — trang 20
* **Phần I: Cô lập và MVCC (Part I: Isolation and MVCC)** — trang 37
  * **Chương 2: Sự cô lập (Isolation)** — trang 39
  * **Chương 3: Trang và Tuple (Pages and Tuples)** — trang 62
  * **Chương 4: Ảnh chụp nhanh (Snapshots)** — trang 80
  * **Chương 5: Dọn dẹp trang và Cập nhật HOT (Page Pruning and HOT Updates)** — trang 92
  * **Chương 6: Vacuum và Autovacuum (Vacuum and Autovacuum)** — trang 102
  * **Chương 7: Đóng băng (Freezing)** — trang 123
  * **Chương 8: Xây dựng lại bảng và chỉ mục (Rebuilding Tables and Indexes)** — trang 135
* **Phần II: Bộ đệm và WAL (Part II: Buffer Cache and WAL)** — trang 145
  * **Chương 9: Bộ đệm (Buffer Cache)** — trang 147
  * **Chương 10: Nhật ký ghi trước (Write-Ahead Log)** — trang 164
  * **Chương 11: Các chế độ WAL (WAL Modes)** — trang 182
* **Phần III: Khóa (Part III: Locks)** — trang 197
  * **Chương 12: Khóa cấp quan hệ (Relation-Level Locks)** — trang 199
  * **Chương 13: Khóa cấp dòng (Row-Level Locks)** — trang 210
  * **Chương 14: Các loại khóa khác (Miscellaneous Locks)** — trang 231
  * **Chương 15: Khóa trên cấu trúc bộ nhớ (Locks on Memory Structures)** — trang 240
* **Phần IV: Thực thi truy vấn (Part IV: Query Execution)** — trang 249
  * **Chương 16: Các giai đoạn thực thi truy vấn (Query Execution Stages)** — trang 251
  * **Chương 17: Thống kê (Statistics)** — trang 271
  * **Chương 18: Phương thức truy cập bảng (Table Access Methods)** — trang 294
  * **Chương 19: Phương thức truy cập chỉ mục (Index Access Methods)** — trang 313
  * **Chương 20: Quét chỉ mục (Index Scans)** — trang 330
  * **Chương 21: Nối vòng lặp lồng nhau (Nested Loop)** — trang 350
  * **Chương 22: Băm (Hashing)** — trang 367
  * **Chương 23: Sắp xếp và Trộn (Sorting and Merging)** — trang 387
* **Phần V: Các loại chỉ mục (Part V: Types of Indexes)** — trang 409
  * **Chương 24: Hash** — trang 411
  * **Chương 25: B-tree** — trang 421
  * **Chương 26: GiST** — trang 444
  * **Chương 27: SP-GiST** — trang 472
  * **Chương 28: GIN** — trang 492
  * **Chương 29: BRIN** — trang 517
* **Kết luận (Conclusion)** — trang 540
* **Chỉ mục tra cứu (Index)** — trang 541

---

## Mục lục chi tiết (Table of Contents)

* **Về cuốn sách này (About This Book)** — 15
* **1. Giới thiệu (Introduction)** — 20
  * 1.1 Tổ chức dữ liệu (Data Organization) — 20
    * Cơ sở dữ liệu (Databases) — 20
    * Danh mục hệ thống (System Catalog) — 21
    * Lược đồ (Schemas) — 22
    * Không gian bảng (Tablespaces) — 22
    * Quan hệ (Relations) — 24
    * Tệp và nhánh tệp (Files and Forks) — 24
    * Trang (Pages) — 28
    * TOAST — 28
  * 1.2 Tiến trình và bộ nhớ (Processes and Memory) — 33
  * 1.3 Máy khách và giao thức máy khách-máy chủ (Clients and the Client-Server Protocol) — 34
* **Phần I: Cô lập và MVCC (Part I: Isolation and MVCC)** — 37
* **2. Sự cô lập (Isolation)** — 39
  * 2.1 Tính nhất quán (Consistency) — 39
  * 2.2 Các mức cô lập và bất thường trong chuẩn SQL (Isolation Levels and Anomalies in SQL Standard) — 41
    * Mất cập nhật (Lost Update) — 41
    * Đọc bẩn và Read Uncommitted (Dirty Reads and Read Uncommitted) — 42
    * Đọc không lặp lại và Read Committed (Non-Repeatable Reads and Read Committed) — 42
    * Đọc bóng ma và Repeatable Read (Phantom Reads and Repeatable Read) — 42
    * Không có bất thường và Serializable (No Anomalies and Serializable) — 43
    * Tại sao lại có những bất thường này? (Why These Anomalies?) — 43
  * 2.3 Các mức cô lập trong PostgreSQL (Isolation Levels in PostgreSQL) — 44
    * Read Committed — 45
    * Repeatable Read — 52
    * Serializable — 58
  * 2.4 Nên sử dụng mức cô lập nào? (Which Isolation Level to Use?) — 61
* **3. Trang và Tuple (Pages and Tuples)** — 62
  * 3.1 Cấu trúc trang (Page Structure) — 62
    * Phần đầu trang (Page Header) — 62
    * Vùng đặc biệt (Special Space) — 63
    * Các tuple (Tuples) — 63
    * Con trỏ mục (Item Pointers) — 63
    * Vùng trống (Free Space) — 64
  * 3.2 Bố cục phiên bản dòng (Row Version Layout) — 64
  * 3.3 Các thao tác trên tuple (Operations on Tuples) — 66
    * Chèn (Insert) — 67
    * Xác nhận (Commit) — 70
    * Xóa (Delete) — 72
    * Hủy bỏ (Abort) — 72
    * Cập nhật (Update) — 73
  * 3.4 Chỉ mục (Indexes) — 74
  * 3.5 TOAST — 74
  * 3.6 Giao dịch ảo (Virtual Transactions) — 75
  * 3.7 Giao dịch con (Subtransactions) — 76
    * Điểm lưu (Savepoints) — 76
    * Lỗi và tính nguyên tử (Errors and Atomicity) — 78
* **4. Ảnh chụp nhanh (Snapshots)** — 80
  * 4.1 Ảnh chụp nhanh là gì? (What is a Snapshot?) — 80
  * 4.2 Khả năng hiển thị của phiên bản dòng (Row Version Visibility) — 81
  * 4.3 Cấu trúc ảnh chụp nhanh (Snapshot Structure) — 82
  * 4.4 Khả năng hiển thị các thay đổi của chính giao dịch (Visibility of Transactions’ Own Changes) — 86
  * 4.5 Đường chân trời giao dịch (Transaction Horizon) — 87
  * 4.6 Ảnh chụp nhanh danh mục hệ thống (System Catalog Snapshots) — 89
  * 4.7 Xuất ảnh chụp nhanh (Exporting Snapshots) — 90
* **5. Dọn dẹp trang và Cập nhật HOT (Page Pruning and HOT Updates)** — 92
  * 5.1 Dọn dẹp trang (Page Pruning) — 92
  * 5.2 Cập nhật HOT (HOT Updates) — 95
  * 5.3 Dọn dẹp trang cho cập nhật HOT (Page Pruning for HOT Updates) — 98
  * 5.4 Tách chuỗi HOT (HOT Chain Splits) — 99
  * 5.5 Dọn dẹp trang cho chỉ mục (Page Pruning for Indexes) — 101
* **6. Vacuum và Autovacuum (Vacuum and Autovacuum)** — 102
  * 6.1 Vacuum — 102
  * 6.2 Xem lại đường chân trời cơ sở dữ liệu (Database Horizon Revisited) — 104
  * 6.3 Các pha của Vacuum (Vacuum Phases) — 107
    * Quét Heap (Heap Scan) — 107
    * Dọn dẹp chỉ mục (Index Vacuuming) — 107
    * Dọn dẹp Heap (Heap Vacuuming) — 108
    * Cắt ngắn Heap (Heap Truncation) — 108
  * 6.4 Phân tích (Analysis) — 109
  * 6.5 Tự động Vacuum và Phân tích (Automatic Vacuum and Analysis) — 109
    * Về cơ chế Autovacuum (About the Autovacuum Mechanism) — 110
    * Bảng nào cần được Vacuum? (Which Tables Need to be Vacuumed?) — 111
    * Bảng nào cần được Analyze? (Which Tables Need to Be Analyzed?) — 113
    * Autovacuum hoạt động thực tế (Autovacuum in Action) — 113
  * 6.6 Quản lý tải (Managing the Load) — 117
    * Điều tiết Vacuum (Vacuum Throttling) — 117
    * Điều tiết Autovacuum (Autovacuum Throttling) — 118
  * 6.7 Giám sát (Monitoring) — 119
    * Giám sát Vacuum (Monitoring Vacuum) — 119
    * Giám sát Autovacuum (Monitoring Autovacuum) — 121
* **7. Đóng băng (Freezing)** — 123
  * 7.1 Xoay vòng ID giao dịch (Transaction ID Wraparound) — 123
  * 7.2 Đóng băng tuple và quy tắc hiển thị (Tuple Freezing and Visibility Rules) — 124
  * 7.3 Quản lý việc đóng băng (Managing Freezing) — 127
    * Tuổi đóng băng tối thiểu (Minimal Freezing Age) — 127
    * Tuổi đóng băng tích cực (Age for Aggressive Freezing) — 129
    * Tuổi kích hoạt Autovacuum cưỡng bức (Age for Forced Autovacuum) — 131
    * Tuổi đóng băng an toàn khẩn cấp (Age for Failsafe Freezing) — 132
  * 7.4 Đóng băng thủ công (Manual Freezing) — 132
    * Đóng băng bằng Vacuum (Freezing by Vacuum) — 133
    * Đóng băng dữ liệu khi nạp ban đầu (Freezing Data at the Initial Loading) — 133
* **8. Xây dựng lại bảng và chỉ mục (Rebuilding Tables and Indexes)** — 135
  * 8.1 Vacuum toàn phần (Full Vacuuming) — 135
    * Tại sao Vacuum thông thường là chưa đủ? (Why is Routine Vacuuming not Enough?) — 135
    * Ước lượng mật độ dữ liệu (Estimating Data Density) — 136
    * Đóng băng (Freezing) — 139
  * 8.2 Các phương pháp xây dựng lại khác (Other Rebuilding Methods) — 140
    * Các lựa chọn thay thế cho Vacuum toàn phần (Alternatives to Full Vacuuming) — 140
    * Giảm thời gian gián đoạn khi xây dựng lại (Reducing Downtime During Rebuilding) — 140
  * 8.3 Các lưu ý phòng ngừa (Precautions) — 141
    * Truy vấn chỉ đọc (Read-Only Queries) — 141
    * Cập nhật dữ liệu (Data Updates) — 142
* **Phần II: Bộ đệm và WAL (Part II: Buffer Cache and WAL)** — 145
* **9. Bộ đệm (Buffer Cache)** — 147
  * 9.1 Cơ chế đệm (Caching) — 147
  * 9.2 Thiết kế bộ đệm (Buffer Cache Design) — 148
  * 9.3 Trúng đệm (Cache Hits) — 149
  * 9.4 Trượt đệm (Cache Misses) — 153
    * Tìm kiếm và loại bỏ đệm (Buffer Search and Eviction) — 154
  * 9.5 Loại bỏ hàng loạt (Bulk Eviction) — 156
  * 9.6 Chọn kích thước bộ đệm (Choosing the Buffer Cache Size) — 158
  * 9.7 Làm nóng bộ đệm (Cache Warming) — 161
  * 9.8 Bộ đệm cục bộ (Local Cache) — 163
* **10. Nhật ký ghi trước (Write-Ahead Log)** — 164
  * 10.1 Ghi nhật ký (Logging) — 164
  * 10.2 Cấu trúc WAL (WAL Structure) — 165
    * Cấu trúc logic (Logical Structure) — 165
    * Cấu trúc vật lý (Physical Structure) — 168
  * 10.3 Điểm kiểm tra (Checkpoint) — 170
  * 10.4 Khôi phục (Recovery) — 173
  * 10.5 Ghi nền (Background Writing) — 176
  * 10.6 Thiết lập WAL (WAL Setup) — 177
    * Cấu hình điểm kiểm tra (Configuring Checkpoints) — 177
    * Cấu hình ghi nền (Configuring Background Writing) — 179
    * Giám sát (Monitoring) — 179
* **11. Các chế độ WAL (WAL Modes)** — 182
  * 11.1 Hiệu năng (Performance) — 182
  * 11.2 Khả năng chịu lỗi (Fault Tolerance) — 185
    * Bộ đệm (Caching) — 186
    * Hỏng dữ liệu (Data Corruption) — 187
    * Ghi không nguyên tử (Non-Atomic Writes) — 189
  * 11.3 Các mức WAL (WAL Levels) — 191
    * Minimal — 192
    * Replica — 193
    * Logical — 196
* **Phần III: Khóa (Part III: Locks)** — 197
* **12. Khóa cấp quan hệ (Relation-Level Locks)** — 199
  * 12.1 Về khóa (About Locks) — 199
  * 12.2 Khóa hạng nặng (Heavyweight Locks) — 201
  * 12.3 Khóa trên ID giao dịch (Locks on Transaction IDs) — 202
  * 12.4 Khóa cấp quan hệ (Relation-Level Locks) — 204
  * 12.5 Hàng đợi chờ (Wait Queue) — 206
* **13. Khóa cấp dòng (Row-Level Locks)** — 210
  * 13.1 Thiết kế khóa (Lock Design) — 210
  * 13.2 Các chế độ khóa cấp dòng (Row-Level Locking Modes) — 211
    * Chế độ độc quyền (Exclusive Modes) — 211
    * Chế độ chia sẻ (Shared Modes) — 213
  * 13.3 Đa giao dịch (Multitransactions) — 213
  * 13.4 Hàng đợi chờ (Wait Queue) — 215
    * Chế độ độc quyền (Exclusive Modes) — 215
    * Chế độ chia sẻ (Shared Modes) — 221
  * 13.5 Khóa không chờ (No-Wait Locks) — 224
  * 13.6 Khóa chết (Deadlocks) — 225
    * Khóa chết do cập nhật dòng (Deadlocks by Row Updates) — 227
    * Khóa chết giữa hai câu lệnh UPDATE (Deadlocks Between Two UPDATE Statements) — 228
* **14. Các loại khóa khác (Miscellaneous Locks)** — 231
  * 14.1 Khóa phi đối tượng (Non-Object Locks) — 231
  * 14.2 Khóa mở rộng quan hệ (Relation Extension Locks) — 232
  * 14.3 Khóa trang (Page Locks) — 233
  * 14.4 Khóa tư vấn (Advisory Locks) — 234
  * 14.5 Khóa vị từ (Predicate Locks) — 235
* **15. Khóa trên cấu trúc bộ nhớ (Locks on Memory Structures)** — 240
  * 15.1 Khóa xoay (Spinlocks) — 240
  * 15.2 Khóa hạng nhẹ (Lightweight Locks) — 240
  * 15.3 Ví dụ (Examples) — 241
    * Bộ đệm (Buffer Cache) — 241
    * Bộ đệm WAL (WAL Buffers) — 242
  * 15.4 Giám sát chờ (Monitoring Waits) — 243
  * 15.5 Lấy mẫu (Sampling) — 245
* **Phần IV: Thực thi truy vấn (Part IV: Query Execution)** — 249
* **16. Các giai đoạn thực thi truy vấn (Query Execution Stages)** — 251
  * 16.1 Cơ sở dữ liệu mẫu (Demo Database) — 251
  * 16.2 Giao thức truy vấn đơn giản (Simple Query Protocol) — 252
    * Phân tích cú pháp (Parsing) — 254
    * Chuyển đổi (Transformation) — 255
    * Lập kế hoạch (Planning) — 257
    * Thực thi (Execution) — 264
  * 16.3 Giao thức truy vấn mở rộng (Extended Query Protocol) — 266
    * Chuẩn bị (Preparation) — 266
    * Ràng buộc tham số (Parameter Binding) — 267
    * Lập kế hoạch và Thực thi (Planning and Execution) — 267
    * Nhận kết quả (Getting the Results) — 270
* **17. Thống kê (Statistics)** — 271
  * 17.1 Thống kê cơ bản (Basic Statistics) — 271
  * 17.2 Giá trị NULL (NULL Values) — 274
  * 17.3 Giá trị phân biệt (Distinct Values) — 276
  * 17.4 Giá trị phổ biến nhất (Most Common Values) — 277
  * 17.5 Biểu đồ tần suất (Histogram) — 279
  * 17.6 Thống kê cho kiểu dữ liệu phi vô hướng (Statistics for Non-Scalar Data Types) — 283
  * 17.7 Độ rộng trường trung bình (Average Field Width) — 284
  * 17.8 Tương quan (Correlation) — 284
  * 17.9 Thống kê biểu thức (Expression Statistics) — 285
    * Thống kê biểu thức mở rộng (Extended Expression Statistics) — 286
    * Thống kê cho chỉ mục biểu thức (Statistics for Expression Indexes) — 287
  * 17.10 Thống kê đa biến (Multivariate Statistics) — 288
    * Phụ thuộc hàm giữa các cột (Functional Dependencies Between Columns) — 288
    * Số lượng giá trị phân biệt đa biến (Multivariate Number of Distinct Values) — 290
    * Danh sách MCV đa biến (Multivariate MCV Lists) — 291
* **18. Phương thức truy cập bảng (Table Access Methods)** — 294
  * 18.1 Engine lưu trữ có thể cắm thêm (Pluggable Storage Engines) — 294
  * 18.2 Quét tuần tự (Sequential Scans) — 296
    * Ước lượng chi phí (Cost Estimation) — 296
  * 18.3 Kế hoạch song song (Parallel Plans) — 300
  * 18.4 Quét tuần tự song song (Parallel Sequential Scans) — 301
    * Ước lượng chi phí (Cost Estimation) — 301
  * 18.5 Giới hạn thực thi song song (Parallel Execution Limitations) — 305
    * Số lượng tiến trình nền (Number of Background Workers) — 305
    * Truy vấn không thể song song hóa (Non-Parallelizable Queries) — 308
    * Truy vấn bị hạn chế song song (Parallel Restricted Queries) — 309
* **19. Phương thức truy cập chỉ mục (Index Access Methods)** — 313
  * 19.1 Chỉ mục và khả năng mở rộng (Indexes and Extensibility) — 313
  * 19.2 Lớp toán tử và họ toán tử (Operator Classes and Families) — 315
    * Lớp toán tử (Operator Classes) — 315
    * Họ toán tử (Operator Families) — 320
  * 19.3 Giao diện engine đánh chỉ mục (Indexing Engine Interface) — 322
    * Thuộc tính phương thức truy cập (Access Method Properties) — 322
    * Thuộc tính cấp chỉ mục (Index-Level Properties) — 326
    * Thuộc tính cấp cột (Column-Level Properties) — 327
* **20. Quét chỉ mục (Index Scans)** — 330
  * 20.1 Quét chỉ mục thông thường (Regular Index Scans) — 330
    * Ước lượng chi phí (Cost Estimation) — 331
    * Kịch bản tốt: Tương quan cao (Good Scenario: High Correlation) — 331
    * Kịch bản xấu: Tương quan thấp (Bad Scenario: Low Correlation) — 334
  * 20.2 Quét chỉ có chỉ mục (Index-Only Scans) — 337
    * Chỉ mục với mệnh đề Include (Indexes with the Include Clause) — 339
  * 20.3 Quét Bitmap (Bitmap Scans) — 340
    * Độ chính xác của Bitmap (Bitmap Accuracy) — 342
    * Các thao tác trên Bitmap (Operations on Bitmaps) — 343
    * Ước lượng chi phí (Cost Estimation) — 343
  * 20.4 Quét chỉ mục song song (Parallel Index Scans) — 347
  * 20.5 So sánh các phương thức truy cập khác nhau (Comparison of Various Access Methods) — 348
* **21. Nối vòng lặp lồng nhau (Nested Loop)** — 350
  * 21.1 Các loại và phương thức nối (Join Types and Methods) — 350
  * 21.2 Nối vòng lặp lồng nhau (Nested Loop Joins) — 351
    * Tích Descartes (Cartesian Product) — 352
    * Nối tham số hóa (Parameterized Joins) — 355
    * Đệm dòng (Caching Rows (Memoization)) — 359
    * Nối ngoài (Outer Joins) — 362
    * Nối phản và bán nối (Anti- and Semi-joins) — 363
    * Nối không bằng (Non-Equi-joins) — 365
    * Chế độ song song (Parallel Mode) — 366
* **22. Băm (Hashing)** — 367
  * 22.1 Nối băm (Hash Joins) — 367
    * Nối băm một lượt (One-Pass Hash Joins) — 367
    * Nối băm hai lượt (Two-Pass Hash Joins) — 372
    * Điều chỉnh động (Dynamic Adjustments) — 374
    * Nối băm trong kế hoạch song song (Hash Joins in Parallel Plans) — 377
    * Nối băm một lượt song song (Parallel One-Pass Hash Joins) — 378
    * Nối băm hai lượt song song (Parallel Two-Pass Hash Joins) — 380
    * Các biến thể (Modifications) — 382
  * 22.2 Giá trị phân biệt và Nhóm (Distinct Values and Grouping) — 384
* **23. Sắp xếp và Trộn (Sorting and Merging)** — 387
  * 23.1 Nối trộn (Merge Joins) — 387
    * Trộn các tập đã sắp xếp (Merging Sorted Sets) — 387
    * Chế độ song song (Parallel Mode) — 390
    * Các biến thể (Modifications) — 391
  * 23.2 Sắp xếp (Sorting) — 392
    * Quicksort — 393
    * Top-N Heapsort — 394
    * Sắp xếp ngoài (External Sorting) — 396
    * Sắp xếp gia tăng (Incremental Sorting) — 399
    * Chế độ song song (Parallel Mode) — 401
  * 23.3 Giá trị phân biệt và Nhóm (Distinct Values and Grouping) — 403
  * 23.4 So sánh các phương thức nối (Comparison of Join Methods) — 405
* **Phần V: Các loại chỉ mục (Part V: Types of Indexes)** — 409
* **24. Hash** — 411
  * 24.1 Tổng quan (Overview) — 411
  * 24.2 Bố cục trang (Page Layout) — 412
  * 24.3 Lớp toán tử (Operator Class) — 417
  * 24.4 Thuộc tính (Properties) — 418
    * Thuộc tính phương thức truy cập (Access Method Properties) — 419
    * Thuộc tính cấp chỉ mục (Index-Level Properties) — 419
    * Thuộc tính cấp cột (Column-Level Properties) — 420
* **25. B-tree** — 421
  * 25.1 Tổng quan (Overview) — 421
  * 25.2 Tìm kiếm và Chèn (Search and Insertions) — 422
    * Tìm kiếm theo phép bằng (Search by Equality) — 422
    * Tìm kiếm theo phép bất đẳng thức (Search by Inequality) — 423
    * Tìm kiếm theo khoảng (Search by Range) — 424
    * Chèn (Insertions) — 425
  * 25.3 Bố cục trang (Page Layout) — 426
    * Khử trùng lặp (Deduplication) — 429
    * Lưu trữ gọn các mục chỉ mục bên trong (Compact Storage of Inner Index Entries) — 431
  * 25.4 Lớp toán tử (Operator Class) — 432
    * Ngữ nghĩa so sánh (Comparison Semantics) — 432
    * Chỉ mục nhiều cột và sắp xếp (Multicolumn Indexes and Sorting) — 437
  * 25.5 Thuộc tính (Properties) — 441
    * Thuộc tính phương thức truy cập (Access Method Properties) — 441
    * Thuộc tính cấp chỉ mục (Index-Level Properties) — 442
    * Thuộc tính cấp cột (Column-Level Properties) — 443
* **26. GiST** — 444
  * 26.1 Tổng quan (Overview) — 444
  * 26.2 Cây R cho điểm (R-Trees for Points) — 445
    * Bố cục trang (Page Layout) — 447
    * Lớp toán tử (Operator Class) — 448
    * Tìm kiếm phần tử nằm trong (Search for Contained Elements) — 450
    * Tìm kiếm láng giềng gần nhất (Nearest Neighbor Search) — 452
    * Chèn (Insertion) — 456
    * Ràng buộc loại trừ (Exclusion Constraints) — 457
    * Thuộc tính (Properties) — 459
  * 26.3 Cây RD cho tìm kiếm toàn văn (RD-Trees for Full-Text Search) — 462
    * Về tìm kiếm toàn văn (About Full-Text Search) — 462
    * Đánh chỉ mục dữ liệu tsvector (Indexing tsvector Data) — 463
    * Thuộc tính (Properties) — 469
  * 26.4 Các kiểu dữ liệu khác (Other Data Types) — 470
* **27. SP-GiST** — 472
  * 27.1 Tổng quan (Overview) — 472
  * 27.2 Cây tứ phân cho điểm (Quadtrees for Points) — 473
    * Lớp toán tử (Operator Class) — 475
    * Bố cục trang (Page Layout) — 477
    * Tìm kiếm (Search) — 479
    * Chèn (Insertion) — 480
    * Thuộc tính (Properties) — 482
  * 27.3 Cây K chiều cho điểm (K-Dimensional Trees for Points) — 483
  * 27.4 Cây cơ số cho chuỗi (Radix Trees for Strings) — 485
    * Lớp toán tử (Operator Class) — 486
    * Tìm kiếm (Search) — 487
    * Chèn (Insertion) — 489
    * Thuộc tính (Properties) — 490
  * 27.5 Các kiểu dữ liệu khác (Other Data Types) — 490
* **28. GIN** — 492
  * 28.1 Tổng quan (Overview) — 492
  * 28.2 Chỉ mục cho tìm kiếm toàn văn (Index for Full-Text Search) — 493
    * Bố cục trang (Page Layout) — 494
    * Lớp toán tử (Operator Class) — 496
    * Tìm kiếm (Search) — 497
    * Từ vị thường gặp và hiếm gặp (Frequent and Rare Lexemes) — 499
    * Chèn (Insertions) — 501
    * Giới hạn kích thước tập kết quả (Limiting Result Set Size) — 503
    * Thuộc tính (Properties) — 505
    * Giới hạn của GIN và chỉ mục RUM (GIN Limitations and RUM Index) — 506
  * 28.3 Bộ ba ký tự (Trigrams) — 507
  * 28.4 Đánh chỉ mục mảng (Indexing Arrays) — 508
  * 28.5 Đánh chỉ mục JSON (Indexing JSON) — 511
    * Lớp toán tử `jsonb_ops` (jsonb_ops Operator Class) — 512
    * Lớp toán tử `jsonb_path_ops` (jsonb_path_ops Operator Class) — 514
  * 28.6 Đánh chỉ mục các kiểu dữ liệu khác (Indexing Other Data Types) — 516
* **29. BRIN** — 517
  * 29.1 Tổng quan (Overview) — 517
  * 29.2 Ví dụ (Example) — 518
  * 29.3 Bố cục trang (Page Layout) — 519
  * 29.4 Tìm kiếm (Search) — 521
  * 29.5 Cập nhật thông tin tóm tắt (Summary Information Updates) — 522
    * Chèn giá trị (Value Insertion) — 522
    * Tóm tắt khoảng (Range Summarization) — 522
  * 29.6 Lớp Minmax (Minmax Classes) — 523
    * Chọn cột để đánh chỉ mục (Choosing Columns to be Indexed) — 525
    * Kích thước khoảng và hiệu quả tìm kiếm (Range Size and Search Efficiency) — 525
    * Thuộc tính (Properties) — 528
  * 29.7 Lớp Minmax-Multi (Minmax-Multi Classes) — 530
  * 29.8 Lớp Inclusion (Inclusion Classes) — 533
  * 29.9 Lớp Bloom (Bloom Classes) — 536
* **Kết luận (Conclusion)** — 540
* **Chỉ mục tra cứu (Index)** — 541

---

# Về cuốn sách này (About This Book)

> *Sách không được viết ra để người ta tin một cách mù quáng, mà là để đem ra chất vấn và tìm hiểu.*
> — Umberto Eco, *Tên của đóa hồng* (*The Name of the Rose*)

## Cuốn sách này dành cho ai? (For Whom Is This Book?)

Cuốn sách này dành cho những ai không chấp nhận cách tiếp cận "hộp đen" (black-box) khi làm việc với cơ sở dữ liệu. Nếu bạn khao khát học hỏi, không muốn tin mặc định vào lời khuyên của chuyên gia mà muốn tự mình tìm hiểu ngọn ngành mọi thứ, hãy cùng theo dõi cuốn sách này.

Tôi giả định rằng người đọc đã từng thử sử dụng PostgreSQL và có ít nhất một số hiểu biết chung về cách nó hoạt động. Người mới bắt đầu có thể thấy nội dung hơi khó. Ví dụ, tôi sẽ không nói bất cứ điều gì về cách cài đặt máy chủ, nhập lệnh `psql` hay thiết lập các tham số cấu hình.

Tôi hy vọng cuốn sách này cũng sẽ hữu ích cho những ai đã quen thuộc với một hệ quản trị cơ sở dữ liệu khác, nhưng đang chuyển sang PostgreSQL và muốn hiểu sự khác biệt giữa chúng.

Một cuốn sách như thế này đáng lẽ đã có thể giúp tôi tiết kiệm rất nhiều thời gian từ vài năm trước. Và đó chính xác là lý do cuối cùng tôi đã viết nó.

## Cuốn sách này sẽ không cung cấp những gì (What This Book Will Not Provide)

Cuốn sách này không phải là một tập hợp các công thức giải pháp sẵn có. Bạn không thể tìm thấy các giải pháp làm sẵn cho mọi tình huống có thể xảy ra, nhưng nếu bạn hiểu được các cơ chế bên trong của một hệ thống phức tạp, bạn sẽ có thể phân tích, đánh giá một cách phản biện kinh nghiệm của người khác và tự rút ra kết luận cho riêng mình. Vì lý do đó, tôi giải thích cả những chi tiết mà thoạt đầu có vẻ như chẳng có ứng dụng thực tế nào.

Nhưng cuốn sách này cũng không phải là một tài liệu hướng dẫn từng bước (tutorial). Trong khi đi sâu vào một số lĩnh vực (những phần mà bản thân tôi hứng thú hơn), cuốn sách có thể không đề cập gì đến những lĩnh vực khác.

Cuốn sách này hoàn toàn không phải là một tài liệu tham khảo tra cứu (reference). Tôi đã cố gắng chính xác, nhưng tôi không đặt mục tiêu thay thế tài liệu chính thức (documentation), nên tôi hoàn toàn có thể bỏ qua một số chi tiết mà tôi cho là không đáng kể. Trong mọi tình huống chưa rõ ràng, hãy đọc tài liệu chính thức.

Cuốn sách này sẽ không dạy bạn cách phát triển nhân (core) PostgreSQL. Tôi không đòi hỏi kiến thức về ngôn ngữ C, vì cuốn sách này chủ yếu hướng đến các quản trị viên cơ sở dữ liệu (DBA) và lập trình viên ứng dụng. Tuy nhiên, tôi có cung cấp nhiều tham chiếu tới mã nguồn, nơi có thể cho bạn nhiều chi tiết tùy thích, thậm chí còn hơn thế nữa.

## Cuốn sách này cung cấp những gì (What This Book Does Provide)

Trong chương giới thiệu, tôi điểm qua ngắn gọn các khái niệm cơ sở dữ liệu chính làm nền tảng cho toàn bộ phần trình bày tiếp theo. Tôi không kỳ vọng bạn sẽ thu được nhiều thông tin mới từ chương này, nhưng vẫn đưa vào để hoàn thiện bức tranh tổng thể. Bên cạnh đó, phần tổng quan này có thể hữu ích cho những ai đang chuyển đổi từ các hệ quản trị cơ sở dữ liệu khác sang.

**Phần I** dành cho các vấn đề về tính nhất quán dữ liệu (data consistency) và sự cô lập (isolation). Trước tiên, tôi trình bày chúng dưới góc nhìn của người dùng (bạn sẽ biết có những mức cô lập nào và hệ quả của chúng ra sao) rồi mới đi sâu vào cơ chế bên trong. Để làm được điều đó, tôi phải giải thích các chi tiết triển khai của cơ chế kiểm soát đồng thời đa phiên bản (MVCC — *multiversion concurrency control*) và cô lập ảnh chụp nhanh (*snapshot isolation*), đồng thời đặc biệt chú ý đến việc dọn dẹp các phiên bản dòng đã lỗi thời.

**Phần II** mô tả bộ đệm (buffer cache) và WAL (nhật ký ghi trước), vốn được dùng để khôi phục tính nhất quán dữ liệu sau sự cố.

**Phần III** đi sâu vào chi tiết cấu trúc và cách sử dụng các loại khóa khác nhau: khóa hạng nhẹ (lightweight locks) cho RAM, khóa hạng nặng (heavyweight locks) cho các quan hệ (relations), và khóa cấp dòng (row-level locks).

**Phần IV** giải thích cách máy chủ lập kế hoạch và thực thi các truy vấn SQL. Tôi sẽ trình bày các phương thức truy cập dữ liệu hiện có, các phương thức nối (join methods) có thể được sử dụng, và cách các thống kê thu thập được áp dụng như thế nào.

**Phần V** mở rộng phần thảo luận về chỉ mục từ cây B-tree (đã đề cập trước đó) sang các phương thức truy cập khác. Tôi sẽ giải thích một số nguyên lý chung về khả năng mở rộng giúp phân định ranh giới giữa nhân của hệ thống đánh chỉ mục, các phương thức truy cập chỉ mục và các kiểu dữ liệu (điều này sẽ dẫn chúng ta tới khái niệm lớp toán tử — *operator classes*), sau đó đi sâu vào từng phương thức hiện có.

PostgreSQL tích hợp nhiều phần mở rộng "nội soi" (*introspective* extensions), tuy không dùng trong công việc thường nhật, nhưng cho phép chúng ta nhìn thấu vào hành vi bên trong của máy chủ. Cuốn sách này sử dụng khá nhiều phần mở rộng như vậy. Ngoài việc giúp chúng ta khám phá nội tại máy chủ, các phần mở rộng này còn có thể hỗ trợ xử lý sự cố trong các kịch bản sử dụng phức tạp.

> 💡 **Ví dụ bổ sung (không có trong sách):** Một số extension "nội soi" phổ biến mà sách sẽ dùng gồm `pageinspect` (xem trực tiếp cấu trúc trang và tuple), `pg_buffercache` (xem trạng thái các trang trong shared buffers), `pgstattuple` (kiểm tra mật độ dữ liệu và dead tuples). Bạn có thể bật thử trong một database test bằng lệnh:
> ```sql
> CREATE EXTENSION IF NOT EXISTS pageinspect;
> CREATE EXTENSION IF NOT EXISTS pg_buffercache;
> ```

## Quy ước trình bày (Conventions)

Tôi cố gắng viết cuốn sách này theo cách cho phép đọc tuần tự từng trang, từ đầu đến cuối. Nhưng khó có thể phơi bày toàn bộ sự thật cùng một lúc, vì vậy tôi phải quay lại cùng một chủ đề nhiều lần. Việc lặp đi lặp lại câu "vấn đề này sẽ được xem xét sau" hết lần này đến lần khác chắc chắn sẽ làm văn bản dài dòng hơn nhiều; đó là lý do trong những trường hợp như vậy tôi chỉ đơn giản đặt số trang ở lề sách (ví dụ: `tr. 28` — trong bản gốc là `p. 28`) để dẫn bạn đến phần thảo luận tiếp theo. Một con số tương tự trỏ ngược lại sẽ đưa bạn về trang nơi điều gì đó đã được đề cập trước đó.

Cả văn bản lẫn mọi ví dụ mã nguồn trong cuốn sách này đều áp dụng cho **PostgreSQL 14**. Bên cạnh một số đoạn văn, bạn có thể thấy số phiên bản ở lề trang (ví dụ: `v. 14`). Điều đó có nghĩa là thông tin được cung cấp có hiệu lực kể từ phiên bản PostgreSQL được chỉ định, còn tất cả các phiên bản trước đó hoặc hoàn toàn không có tính năng được mô tả, hoặc sử dụng một cách triển khai khác. Những ghi chú như vậy có thể hữu ích cho những ai chưa nâng cấp hệ thống lên bản phát hành mới nhất.

Tôi cũng sử dụng phần lề trang để hiển thị giá trị mặc định của các tham số đang thảo luận (ví dụ: `4MB`). Tên của cả tham số thông thường lẫn tham số lưu trữ đều được in nghiêng: *work_mem*.

Trong các chú thích cuối trang (footnotes), tôi cung cấp nhiều liên kết đến các nguồn thông tin khác nhau. Có khá nhiều nguồn, nhưng trước hết và quan trọng nhất, tôi liệt kê tài liệu chính thức của PostgreSQL[^1], vốn là một kho tàng tri thức. Là một phần thiết yếu của dự án, tài liệu này luôn được chính các nhà phát triển PostgreSQL cập nhật liên tục. Tuy nhiên, nguồn tham khảo gốc chắc chắn là mã nguồn[^2]. Thật đáng kinh ngạc khi bạn có thể tìm thấy biết bao câu trả lời chỉ bằng cách đọc các chú thích (comments) và lướt qua các tệp `README`, ngay cả khi bạn không biết ngôn ngữ C. Đôi khi tôi cũng dẫn chiếu đến các mục trên commitfest[^3]: bạn luôn có thể lần theo lịch sử mọi thay đổi và hiểu được logic đằng sau các quyết định của nhà phát triển nếu đọc các cuộc thảo luận liên quan trong danh sách thư (mailing list) `pgsql-hackers` (trong bản in ghi là `psql-hackers`), dù việc đó đòi hỏi phải đào bới qua hàng đống email.

Những ghi chú bên lề (side notes) có thể khiến mạch thảo luận đi chệch hướng (những điều tôi không thể kìm lòng mà không đưa vào sách) được in theo kiểu tách biệt riêng để bạn có thể dễ dàng bỏ qua nếu muốn.

Tất nhiên, cuốn sách chứa nhiều ví dụ mã lệnh, chủ yếu bằng SQL. Mã lệnh được trình bày kèm dấu nhắc `=>`; phản hồi của máy chủ sẽ theo sau nếu cần thiết:

```sql
=> SELECT now();
              now
-------------------------------
 2023-03-06 14:00:08.008545+03
(1 row)
```

Nếu bạn cẩn thận lặp lại tất cả các lệnh được cung cấp trên PostgreSQL 14, bạn sẽ nhận được kết quả giống hệt (ngoại trừ các ID giao dịch và các chi tiết không quan trọng khác). Dù sao thì mọi ví dụ mã lệnh trong cuốn sách này đều được tạo ra bởi kịch bản (script) chứa chính xác các lệnh đó.

Khi cần minh họa việc thực thi đồng thời của nhiều giao dịch, đoạn mã chạy trong một phiên làm việc (session) khác được thụt lề và đánh dấu bằng một đường kẻ dọc:

```sql
=> SHOW server_version;
 server_version
----------------
 14.7
(1 row)
```

Để tự chạy thử các lệnh như vậy (rất hữu ích cho việc tự học, cũng như bất kỳ thử nghiệm nào khác), bạn nên mở sẵn hai cửa sổ dòng lệnh `psql`.

Tên các lệnh và nhiều đối tượng cơ sở dữ liệu khác nhau (như bảng và cột, hàm hoặc phần mở rộng) được làm nổi bật trong văn bản bằng phông chữ không chân (sans-serif): `UPDATE`, `pg_class`.

Nếu một tiện ích được gọi từ hệ điều hành, nó được hiển thị với dấu nhắc kết thúc bằng `$`:

```bash
postgres$ whoami
postgres
```

Tôi sử dụng Linux, nhưng không đi vào các chi tiết kỹ thuật hệ thống phức tạp; chỉ cần có hiểu biết cơ bản về hệ điều hành này là đủ.

[^1]: `postgresql.org/docs/14/index.html`
[^2]: `git.postgresql.org/gitweb/?p=postgresql.git;a=summary`
[^3]: `commitfest.postgresql.org`

## Lời cảm ơn (Acknowledgments)

Không thể viết một cuốn sách chỉ một mình, và giờ tôi có một cơ hội tuyệt vời để cảm ơn những người tốt.

Tôi rất biết ơn Pavel Luzanov, người đã tìm thấy đúng thời điểm và đề nghị tôi bắt đầu làm một điều gì đó thực sự đáng giá.

Tôi mang ơn Postgres Professional vì cơ hội được làm việc với cuốn sách này vượt ra ngoài thời gian rảnh rỗi của mình. Nhưng đằng sau công ty là những con người thực sự, vì vậy tôi muốn bày tỏ lòng biết ơn tới Oleg Bartunov vì đã chia sẻ ý tưởng và nguồn năng lượng vô tận, và tới Ivan Panchenko vì sự hỗ trợ tận tình cùng LATEX.

Tôi muốn cảm ơn các đồng nghiệp trong nhóm đào tạo vì bầu không khí sáng tạo và những cuộc thảo luận đã định hình phạm vi và hình thức các khóa học của chúng tôi, điều này cũng được phản ánh trong cuốn sách. Xin gửi lời cảm ơn đặc biệt tới Pavel Tolmachev vì sự rà soát tỉ mỉ các bản thảo.

Nhiều chương của cuốn sách này ban đầu được xuất bản dưới dạng các bài viết trên blog Habr[^4], và tôi biết ơn độc giả vì những bình luận và phản hồi của họ. Điều đó cho thấy tầm quan trọng của công việc này, chỉ ra một số lỗ hổng trong kiến thức của tôi và giúp tôi cải thiện nội dung.

Tôi cũng muốn cảm ơn Liudmila Mantrova, người đã bỏ nhiều công sức trau chuốt ngôn từ cho cuốn sách này. Nếu bạn không bị vấp ở từng câu chữ, công lao đó thuộc về cô ấy. Ngoài ra, Liudmila còn không quản ngại dịch cuốn sách này sang tiếng Anh, điều mà tôi cũng vô cùng biết ơn.

Tôi không nêu tên cụ thể, nhưng mỗi hàm hay tính năng được nhắc đến trong cuốn sách này đều đòi hỏi nhiều năm lao động của những con người cụ thể. Tôi ngưỡng mộ các nhà phát triển PostgreSQL và rất vui khi có vinh dự được gọi nhiều người trong số họ là đồng nghiệp của mình.

[^4]: `habr.com/en/company/postgrespro/blog`

---

## Bảng thuật ngữ chương (Chapter Glossary)

| Thuật ngữ (English) | Tiếng Việt | Giải thích ngắn gọn |
| --- | --- | --- |
| Black-box approach | Cách tiếp cận hộp đen | Sử dụng hệ thống chỉ dựa vào đầu vào/đầu ra mà không hiểu cơ chế bên trong |
| Multiversion Concurrency Control (MVCC) | Kiểm soát đồng thời đa phiên bản | Cơ chế cho phép nhiều giao dịch đọc/ghi đồng thời bằng cách duy trì nhiều phiên bản của dữ liệu |
| Snapshot isolation | Cô lập ảnh chụp nhanh | Cơ chế cô lập đảm bảo mỗi giao dịch nhìn thấy một trạng thái nhất quán của dữ liệu tại một thời điểm |
| Write-Ahead Log (WAL) | Nhật ký ghi trước | Cơ chế ghi mọi thay đổi vào nhật ký trước khi áp dụng lên tệp dữ liệu để đảm bảo khả năng khôi phục |
| Buffer cache | Bộ đệm | Vùng bộ nhớ chia sẻ (shared memory) lưu giữ các trang dữ liệu được truy cập thường xuyên |
| Relation | Quan hệ | Thuật ngữ tổng quát trong PostgreSQL chỉ bảng, chỉ mục, chuỗi (sequence), khung nhìn (view)... |
| Operator class | Lớp toán tử | Cấu trúc định nghĩa cách một kiểu dữ liệu tương tác với một phương thức truy cập chỉ mục cụ thể |
| Introspective extension | Phần mở rộng nội soi | Các extension giúp quan sát trạng thái nội bộ của máy chủ PostgreSQL (như `pageinspect`, `pg_buffercache`) |
| Commitfest | Chu kỳ rà soát bản vá | Quy trình đánh giá và tích hợp các bản vá (patch) mới vào mã nguồn PostgreSQL |
