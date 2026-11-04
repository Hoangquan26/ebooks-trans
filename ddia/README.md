# Designing Data-Intensive Applications (DDIA) — Phiên bản Tiếng Việt (Đầy đủ 100% + Ví dụ bổ sung)

- **Tác giả gốc:** Martin Kleppmann (O'Reilly Media, 2017 — 491 trang PDF) — *The Big Ideas Behind Reliable, Scalable, and Maintainable Systems*
- **Nguyên tắc biên dịch & chuyển thể:**
  1. **Giữ trọn vẹn 100% nội dung sách gốc:** Đầy đủ mọi Phần (`Part I–III`), Chương (`Chapter 1–11`), mục, tiểu mục, đoạn văn, hộp ghi chú/sidebar, `Summary`, danh mục tài liệu tham khảo (`References`) cuối mỗi chương, mọi đoạn truy vấn/mã nguồn (`Example`), bảng biểu (`Bảng` / `Table`) và hình minh họa (`Hình` / `Figure` kèm ảnh trang PDF gốc + sơ đồ Mermaid / bảng phân tích tiếng Việt).
  2. **Diễn giải dễ hiểu + Bổ sung bài toán/ví dụ thực tế:** Các khái niệm cốt lõi về hệ thống dữ liệu và phân tán (B-Tree vs LSM-Tree, OLTP vs OLAP, Columnar Storage, Protobuf/Avro, Replication, Partitioning, ACID & Isolation Levels, 2PL vs SSI, Unreliable Networks/Clocks, Linearizability, Consensus Raft/Paxos/2PC, MapReduce/Spark, Stream Processing Kafka/Flink/CDC/Event Sourcing) đều được bổ sung ví dụ thực tiễn trong các khối đánh dấu rõ:
     - `> 💡 **Ví dụ bổ sung (không có trong sách):** ...`
     - `> 📝 **Giải thích thêm (không có trong sách):** ...`
  3. **Bảng thuật ngữ cuối mỗi chương:** Giúp tra cứu nhanh thuật ngữ Thiết kế Hệ thống (System Design) và Hệ thống Phân tán (Distributed Systems).

---

## Mục lục các chương

| STT | Chương | Tệp Markdown | Trang PDF gốc |
|---|---|---|---|
| 00 | **Lời mở đầu** (Bìa, Bản quyền, Mục lục, Giới thiệu về cuốn sách – *About this Book*) | [00-loi-mo-dau.md](./00-loi-mo-dau.md) | 1 – 20 |
| 01 | **Phần I & Chương 1:** Ứng dụng tin cậy, có khả năng mở rộng và dễ bảo trì (*Reliable, Scalable and Maintainable Applications*) | [01-reliable-scalable-maintainable.md](./01-reliable-scalable-maintainable.md) | 21 – 46 |
| 02 | **Chương 2:** Mô hình dữ liệu và ngôn ngữ truy vấn (*Data Models and Query Languages*) | [02-data-models-and-query-languages.md](./02-data-models-and-query-languages.md) | 47 – 88 |
| 03 | **Chương 3:** Lưu trữ và truy xuất (*Storage and Retrieval*) | [03-storage-and-retrieval.md](./03-storage-and-retrieval.md) | 89 – 128 |
| 04 | **Chương 4:** Mã hóa và sự tiến hóa dữ liệu (*Encoding and Evolution*) | [04-encoding-and-evolution.md](./04-encoding-and-evolution.md) | 129 – 162 |
| 05 | **Phần II & Chương 5:** Nhân bản dữ liệu (*Replication*) | [05-replication.md](./05-replication.md) | 163 – 212 |
| 06 | **Chương 6:** Phân vùng dữ liệu (*Partitioning*) | [06-partitioning.md](./06-partitioning.md) | 213 – 234 |
| 07 | **Chương 7:** Giao dịch (*Transactions*) | [07-transactions.md](./07-transactions.md) | 235 – 286 |
| 08 | **Chương 8:** Những rắc rối với hệ thống phân tán (*The Trouble with Distributed Systems*) | [08-trouble-with-distributed-systems.md](./08-trouble-with-distributed-systems.md) | 287 – 332 |
| 09 | **Chương 9:** Tính nhất quán và đồng thuận (*Consistency and Consensus*) | [09-consistency-and-consensus.md](./09-consistency-and-consensus.md) | 333 – 396 |
| 10 | **Phần III & Chương 10:** Xử lý theo lô (*Batch Processing*) | [10-batch-processing.md](./10-batch-processing.md) | 397 – 446 |
| 11 | **Chương 11:** Xử lý luồng (*Stream Processing*) | [11-stream-processing.md](./11-stream-processing.md) | 447 – 491 |
