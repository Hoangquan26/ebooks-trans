# PostgreSQL 14 Internals — Phiên bản Tiếng Việt (Đầy đủ 100% + Ví dụ bổ sung)

- **Tác giả gốc:** Egor Rogov (Postgres Professional, 2023 — 548 trang PDF) — *PostgreSQL 14 Internals*
- **Nguyên tắc biên dịch & chuyển thể:**
  1. **Giữ trọn vẹn 100% nội dung sách gốc:** Đầy đủ từ trang bìa, lời tựa, toàn bộ 5 Phần (*Part I–V*), 29 chương chuyên sâu về kiến trúc bên trong PostgreSQL 14 (MVCC, Isolation, Pages & Tuples, Snapshots, HOT Updates, Vacuum/Autovacuum, Freezing, Buffer Cache, WAL, Relation/Row/Memory Locks, Query Planner/Executor, Access Methods, Hash/B-tree/GiST/SP-GiST/GIN/BRIN Indexes), Kết luận và Chỉ mục.
  2. **Diễn giải dễ hiểu + Bổ sung thực chiến:** Mỗi cơ chế lõi của PostgreSQL đều được minh họa thêm bằng các khối đánh dấu rõ:
     - `> 💡 **Ví dụ bổ sung (không có trong sách):** ...`
     - `> 📝 **Giải thích thêm (không có trong sách):** ...`
  3. **Bảng thuật ngữ cuối mỗi chương:** Giúp tra cứu nhanh thuật ngữ Nội bộ Cơ sở dữ liệu (*Database Internals*) song ngữ Anh–Việt.

---

## Mục lục các chương

| STT | Phần / Chương | Tệp Markdown | Trang PDF gốc |
|---|---|---|---|
| 00 | **Lời mở đầu** (Bìa, Mục lục, Giới thiệu về cuốn sách – *About This Book*) | [00-loi-mo-dau.md](./00-loi-mo-dau.md) | 1 – 19 |
| 01 | **Giới thiệu** (*Introduction*: Data Organization, Processes & Memory, Client-Server Protocol) | [01-introduction.md](./01-introduction.md) | 20 – 36 |
| 02 | **Phần I & Chương 2:** Cô lập giao dịch (*Part I: Isolation and MVCC — Isolation*) | [02-isolation.md](./02-isolation.md) | 37 – 61 |
| 03 | **Chương 3:** Trang và Bộ dữ liệu (*Pages and Tuples*) | [03-pages-and-tuples.md](./03-pages-and-tuples.md) | 62 – 79 |
| 04 | **Chương 4:** Ảnh chụp nhanh (*Snapshots*) | [04-snapshots.md](./04-snapshots.md) | 80 – 91 |
| 05 | **Chương 5:** Cắt tỉa trang và Cập nhật HOT (*Page Pruning and HOT Updates*) | [05-page-pruning-hot-updates.md](./05-page-pruning-hot-updates.md) | 92 – 101 |
| 06 | **Chương 6:** Vacuum và Autovacuum (*Vacuum and Autovacuum*) | [06-vacuum-and-autovacuum.md](./06-vacuum-and-autovacuum.md) | 102 – 122 |
| 07 | **Chương 7:** Đóng băng giao dịch (*Freezing*) | [07-freezing.md](./07-freezing.md) | 123 – 134 |
| 08 | **Chương 8:** Tái xây dựng Bảng và Chỉ mục (*Rebuilding Tables and Indexes*) | [08-rebuilding-tables-and-indexes.md](./08-rebuilding-tables-and-indexes.md) | 135 – 144 |
| 09 | **Phần II & Chương 9:** Bộ nhớ đệm (*Part II: Buffer Cache and WAL — Buffer Cache*) | [09-buffer-cache.md](./09-buffer-cache.md) | 145 – 163 |
| 10 | **Chương 10:** Nhật ký ghi trước (*Write-Ahead Log — WAL*) | [10-write-ahead-log.md](./10-write-ahead-log.md) | 164 – 181 |
| 11 | **Chương 11:** Các chế độ WAL (*WAL Modes*) | [11-wal-modes.md](./11-wal-modes.md) | 182 – 196 |
| 12 | **Phần III & Chương 12:** Khóa cấp Quan hệ (*Part III: Locks — Relation-Level Locks*) | [12-relation-level-locks.md](./12-relation-level-locks.md) | 197 – 209 |
| 13 | **Chương 13:** Khóa cấp Dòng (*Row-Level Locks*) | [13-row-level-locks.md](./13-row-level-locks.md) | 210 – 230 |
| 14 | **Chương 14:** Các loại khóa khác (*Miscellaneous Locks*) | [14-miscellaneous-locks.md](./14-miscellaneous-locks.md) | 231 – 239 |
| 15 | **Chương 15:** Khóa trên cấu trúc bộ nhớ (*Locks on Memory Structures*) | [15-locks-on-memory-structures.md](./15-locks-on-memory-structures.md) | 240 – 248 |
| 16 | **Phần IV & Chương 16:** Các giai đoạn thực thi truy vấn (*Part IV: Query Execution — Query Execution Stages*) | [16-query-execution-stages.md](./16-query-execution-stages.md) | 249 – 270 |
| 17 | **Chương 17:** Thống kê (*Statistics*) | [17-statistics.md](./17-statistics.md) | 271 – 293 |
| 18 | **Chương 18:** Phương thức truy cập Bảng (*Table Access Methods*) | [18-table-access-methods.md](./18-table-access-methods.md) | 294 – 312 |
| 19 | **Chương 19:** Phương thức truy cập Chỉ mục (*Index Access Methods*) | [19-index-access-methods.md](./19-index-access-methods.md) | 313 – 329 |
| 20 | **Chương 20:** Quét Chỉ mục (*Index Scans*) | [20-index-scans.md](./20-index-scans.md) | 330 – 349 |
| 21 | **Chương 21:** Vòng lặp lồng nhau (*Nested Loop*) | [21-nested-loop.md](./21-nested-loop.md) | 350 – 366 |
| 22 | **Chương 22:** Băm (*Hashing*) | [22-hashing.md](./22-hashing.md) | 367 – 386 |
| 23 | **Chương 23:** Sắp xếp và Trộn (*Sorting and Merging*) | [23-sorting-and-merging.md](./23-sorting-and-merging.md) | 387 – 408 |
| 24 | **Phần V & Chương 24:** Chỉ mục Hash (*Part V: Types of Indexes — Hash*) | [24-hash-index.md](./24-hash-index.md) | 409 – 420 |
| 25 | **Chương 25:** Chỉ mục B-tree (*B-tree*) | [25-btree-index.md](./25-btree-index.md) | 421 – 443 |
| 26 | **Chương 26:** Chỉ mục GiST (*GiST*) | [26-gist-index.md](./26-gist-index.md) | 444 – 471 |
| 27 | **Chương 27:** Chỉ mục SP-GiST (*SP-GiST*) | [27-sp-gist-index.md](./27-sp-gist-index.md) | 472 – 491 |
| 28 | **Chương 28:** Chỉ mục GIN (*GIN*) | [28-gin-index.md](./28-gin-index.md) | 492 – 516 |
| 29 | **Chương 29 & Kết luận:** Chỉ mục BRIN, Kết luận & Chỉ mục tra cứu (*BRIN, Conclusion & Index*) | [29-brin-index.md](./29-brin-index.md) | 517 – 548 |
