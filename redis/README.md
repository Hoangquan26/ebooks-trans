# Redis in Action — Phiên bản Tiếng Việt (Đầy đủ 100% + Ví dụ bổ sung)

- **Tác giả gốc:** Josiah L. Carlson, lời tựa bởi Salvatore Sanfilippo (Manning Publications, 2013 — 322 trang)
- **Nguyên tắc biên dịch & chuyển thể:**
  1. **Giữ trọn vẹn 100% nội dung sách gốc:** Đầy đủ mọi chương, mục, tiểu mục, đoạn văn, hộp ghi chú (`NOTE`, `TIP`, `WARNING`, sidebar), chú thích chân trang, `Summary`, mọi đoạn mã (`Listing` bằng Python, Lua, cấu hình Redis, phiên tương tác `redis-cli` / Python REPL), bảng biểu (`Bảng`) và hình minh họa (`Hình` kèm ảnh render từ trang PDF gốc + sơ đồ Mermaid / bảng giải thích tiếng Việt).
  2. **Diễn giải dễ hiểu + Bổ sung bài toán/ví dụ thực tế:** Các khái niệm cấu trúc dữ liệu Redis (`STRING`, `LIST`, `SET`, `HASH`, `ZSET`), persistence (`RDB`, `AOF`), replication, transaction (`WATCH`/`MULTI`/`EXEC`), pipeline, distributed lock, semaphore, task queue, tìm kiếm đảo ngược (*inverted index*), mạng xã hội, tối ưu bộ nhớ (`ziplist`/`listpack`, `intset`, sharding), mở rộng quy mô và lập trình Lua đều được bổ sung ví dụ thực tiễn và ghi chú cập nhật cho **Redis 6/7/8** và **Python 3 / `redis-py` 4.x–5.x** trong các khối đánh dấu rõ:
     - `> 💡 **Ví dụ bổ sung (không có trong sách):** ...`
     - `> 📝 **Giải thích thêm (không có trong sách):** ...`
  3. **Bảng thuật ngữ cuối mỗi chương:** Giúp tra cứu nhanh thuật ngữ Redis, hệ thống phân tán và ý nghĩa kỹ thuật.

---

## Mục lục các chương

| STT | Chương | Tệp Markdown | Trang PDF gốc |
|---|---|---|---|
| 00 | **Lời mở đầu** (Bìa, Mục lục, Foreword, Preface, Acknowledgments, About this book, Cover illustration) | [00-loi-mo-dau.md](./00-loi-mo-dau.md) | 1 – 27 |
| 01 | **Phần 1 & Chương 1:** Làm quen với Redis (*Getting to know Redis*) | [01-getting-to-know-redis.md](./01-getting-to-know-redis.md) | 28 – 50 |
| 02 | **Chương 2:** Giải phẫu một ứng dụng web dùng Redis (*Anatomy of a Redis web application*) | [02-anatomy-of-a-redis-web-application.md](./02-anatomy-of-a-redis-web-application.md) | 51 – 63 |
| 03 | **Phần 2 & Chương 3:** Các lệnh trong Redis (*Commands in Redis*) | [03-commands-in-redis.md](./03-commands-in-redis.md) | 64 – 89 |
| 04 | **Chương 4:** Giữ an toàn dữ liệu và đảm bảo hiệu năng (*Keeping data safe and ensuring performance*) | [04-keeping-data-safe-and-ensuring-performance.md](./04-keeping-data-safe-and-ensuring-performance.md) | 90 – 116 |
| 05 | **Chương 5:** Sử dụng Redis để hỗ trợ ứng dụng (*Using Redis for application support*) | [05-using-redis-for-application-support.md](./05-using-redis-for-application-support.md) | 117 – 136 |
| 06 | **Chương 6:** Các thành phần ứng dụng trong Redis (*Application components in Redis*) | [06-application-components-in-redis.md](./06-application-components-in-redis.md) | 137 – 180 |
| 07 | **Chương 7:** Các ứng dụng dựa trên tìm kiếm (*Search-based applications*) | [07-search-based-applications.md](./07-search-based-applications.md) | 181 – 211 |
| 08 | **Chương 8:** Xây dựng một mạng xã hội đơn giản (*Building a simple social network*) | [08-building-a-simple-social-network.md](./08-building-a-simple-social-network.md) | 212 – 233 |
| 09 | **Phần 3 & Chương 9:** Giảm dung lượng bộ nhớ sử dụng (*Reducing memory use*) | [09-reducing-memory-use.md](./09-reducing-memory-use.md) | 234 – 254 |
| 10 | **Chương 10:** Mở rộng quy mô Redis (*Scaling Redis*) | [10-scaling-redis.md](./10-scaling-redis.md) | 255 – 275 |
| 11 | **Chương 11:** Lập trình kịch bản Redis với Lua (*Scripting Redis with Lua*) | [11-scripting-redis-with-lua.md](./11-scripting-redis-with-lua.md) | 276 – 299 |
| 12 | **Phụ lục A & B:** Cài đặt nhanh gọn (*Quick and dirty setup*) & Tài nguyên và tài liệu tham khảo khác (*Other resources and references*) | [12-phu-luc-A-B.md](./12-phu-luc-A-B.md) | 300 – 310 |
