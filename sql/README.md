# SQL Performance Explained — Phiên bản Tiếng Việt (Đầy đủ 100% + Ví dụ bổ sung)

- **Tác giả gốc:** Markus Winand (2012, 207 trang) — *Everything developers need to know about SQL performance*
- **Nguyên tắc chuyển thể:**
  1. **Giữ trọn vẹn 100% nội dung sách:** mọi mục, đoạn văn, hộp Tip/Important/Note/Warning, ghi chú riêng cho từng hệ quản trị (Oracle, PostgreSQL, SQL Server, MySQL, DB2), chú thích chân trang, mọi câu SQL và execution plan (chép nguyên văn), mọi Hình (nhúng ảnh trang gốc + vẽ lại + mô tả tiếng Việt) và Ví dụ (Example).
  2. **Chỉ thêm ví dụ/giải thích**, đánh dấu rõ:
     - `> 💡 **Ví dụ bổ sung (không có trong sách):** ...`
     - `> 📝 **Giải thích thêm (không có trong sách):** ...`
     (bao gồm cả ghi chú khác biệt ở các phiên bản DB mới hơn: PostgreSQL 11+ `INCLUDE`, MySQL 8 functional/descending index & hash join, Oracle 12c `FETCH FIRST`, ...)
  3. **Bảng thuật ngữ** ở cuối mỗi phần.
- **Ghi chú:** Ảnh trang gốc nằm trong `images/pages/`. Thông tin cá nhân của người được cấp phép bản PDF đã được lược bỏ.

---

## Mục lục

| STT | Chương | Tệp Markdown | Trang PDF |
|---|---|---|---|
| 00 | **Lời mở đầu** (Bìa, License, Mục lục, Preface – *Developers Need to Index*) | [00-loi-mo-dau.md](./00-loi-mo-dau.md) | 1 – 10 |
| 01 | **Chương 1:** Giải phẫu một index (*Anatomy of an Index*) | [01-anatomy-of-an-index.md](./01-anatomy-of-an-index.md) | 11 – 18 |
| 02a | **Chương 2:** Mệnh đề WHERE — Phần 1: Toán tử bằng, Hàm (*The Equality Operator, Functions*) | [02a-where-equality-functions.md](./02a-where-equality-functions.md) | 19 – 41 |
| 02b | **Chương 2:** Mệnh đề WHERE — Phần 2: Truy vấn tham số hóa, Tìm theo khoảng, Partial index (*Parameterized Queries, Searching for Ranges, Partial Indexes*) | [02b-where-bind-ranges-partial.md](./02b-where-bind-ranges-partial.md) | 42 – 62 |
| 02c | **Chương 2:** Mệnh đề WHERE — Phần 3: NULL trong Oracle, Điều kiện bị che khuất (*NULL in the Oracle Database, Obfuscated Conditions*) | [02c-where-null-obfuscated.md](./02c-where-null-obfuscated.md) | 63 – 88 |
| 03 | **Chương 3:** Hiệu năng và khả năng mở rộng (*Performance and Scalability*) | [03-performance-scalability.md](./03-performance-scalability.md) | 89 – 100 |
| 04 | **Chương 4:** Phép join (*The Join Operation*) | [04-join-operation.md](./04-join-operation.md) | 101 – 120 |
| 05 | **Chương 5:** Gom cụm dữ liệu (*Clustering Data*) | [05-clustering-data.md](./05-clustering-data.md) | 121 – 138 |
| 06 | **Chương 6:** Sắp xếp và gom nhóm (*Sorting and Grouping*) | [06-sorting-grouping.md](./06-sorting-grouping.md) | 139 – 152 |
| 07 | **Chương 7:** Kết quả từng phần (*Partial Results*) | [07-partial-results.md](./07-partial-results.md) | 153 – 168 |
| 08 | **Chương 8:** Sửa đổi dữ liệu (*Modifying Data*) | [08-modifying-data.md](./08-modifying-data.md) | 169 – 174 |
| 09a | **Phụ lục A:** Execution Plan — Phần 1: Oracle, PostgreSQL | [09a-phu-luc-A-oracle-postgresql.md](./09a-phu-luc-A-oracle-postgresql.md) | 175 – 189 |
| 09b | **Phụ lục A:** Execution Plan — Phần 2: SQL Server, MySQL | [09b-phu-luc-A-sqlserver-mysql.md](./09b-phu-luc-A-sqlserver-mysql.md) | 190 – 202 |
