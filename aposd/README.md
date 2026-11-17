# A Philosophy of Software Design (APOSD) — Phiên bản Tiếng Việt (Đầy đủ 100% + Ví dụ bổ sung)

- **Tác giả gốc:** John Ousterhout (Yaknyam Press, 2018 — 188 trang PDF) — *A Philosophy of Software Design*
- **Nguyên tắc biên dịch & chuyển thể:**
  1. **Giữ trọn vẹn 100% nội dung sách gốc:** Đầy đủ từ trang bìa, bản quyền, mục lục, lời tựa (*Preface*), toàn bộ 21 chương (*Chapter 1–21*), các mục và tiểu mục đánh số, đoạn văn, ví dụ mã nguồn, **Cờ Đỏ (Red Flags)**, bảng biểu và hình minh họa (**Hình / Figure** kèm ảnh trích xuất từ PDF gốc + sơ đồ Mermaid minh họa khi cần), phần tổng kết nguyên tắc thiết kế (*Summary of Design Principles*), tổng hợp cờ đỏ (*Summary of Red Flags*), và thông tin tác giả (*About the Author*).
  2. **Diễn giải dễ hiểu + Bổ sung ví dụ thực tế:** Các tư tưởng cốt lõi về kiểm soát độ phức tạp (*complexity*), module sâu (*deep modules*), che giấu thông tin (*information hiding*), thiết kế tổng quát (*general-purpose modules*), tách tầng trừu tượng (*different layer, different abstraction*), đẩy độ phức tạp xuống dưới (*pull complexity downwards*), thiết kế lỗi ra khỏi sự tồn tại (*define errors out of existence*), viết comment trước (*write the comments first*), đặt tên chuẩn xác (*choosing names*), và tối ưu hiệu năng (*designing for performance*) đều được minh họa thêm bằng các khối đánh dấu rõ:
     - `> 💡 **Ví dụ bổ sung (không có trong sách):** ...`
     - `> 📝 **Giải thích thêm (không có trong sách):** ...`
  3. **Bảng thuật ngữ cuối mỗi chương:** Giúp tra cứu nhanh thuật ngữ Thiết kế Phần mềm (*Software Design*) song ngữ Anh–Việt.

---

## Mục lục các chương

| STT | Chương | Tệp Markdown | Trang PDF gốc |
|---|---|---|---|
| 00 | **Lời mở đầu** (Bìa, Bản quyền, Mục lục, Lời tựa – *Preface*) | [00-loi-mo-dau.md](./00-loi-mo-dau.md) | 1 – 12 |
| 01 | **Chương 1:** Giới thiệu (*Introduction*) | [01-introduction.md](./01-introduction.md) | 13 – 17 |
| 02 | **Chương 2:** Bản chất của Độ phức tạp (*The Nature of Complexity*) | [02-the-nature-of-complexity.md](./02-the-nature-of-complexity.md) | 18 – 24 |
| 03 | **Chương 3:** Mã nguồn chạy được thôi là chưa đủ (*Working Code Isn't Enough*) | [03-working-code-isnt-enough.md](./03-working-code-isnt-enough.md) | 25 – 30 |
| 04 | **Chương 4:** Các Module nên có chiều sâu (*Modules Should Be Deep*) | [04-modules-should-be-deep.md](./04-modules-should-be-deep.md) | 31 – 39 |
| 05 | **Chương 5:** Che giấu thông tin (và Rò rỉ thông tin) (*Information Hiding (and Leakage)*) | [05-information-hiding.md](./05-information-hiding.md) | 40 – 49 |
| 06 | **Chương 6:** Các Module đa dụng có chiều sâu hơn (*General-Purpose Modules are Deeper*) | [06-general-purpose-modules.md](./06-general-purpose-modules.md) | 50 – 55 |
| 07 | **Chương 7:** Tầng khác nhau, Trừu tượng khác nhau (*Different Layer, Different Abstraction*) | [07-different-layer-different-abstraction.md](./07-different-layer-different-abstraction.md) | 56 – 64 |
| 08 | **Chương 8:** Kéo độ phức tạp xuống phía dưới (*Pull Complexity Downwards*) | [08-pull-complexity-downwards.md](./08-pull-complexity-downwards.md) | 65 – 68 |
| 09 | **Chương 9:** Gộp chung hay Tách riêng? (*Better Together Or Better Apart?*) | [09-better-together-or-better-apart.md](./09-better-together-or-better-apart.md) | 69 – 83 |
| 10 | **Chương 10:** Thiết kế để loại bỏ lỗi ngay từ định nghĩa (*Define Errors Out Of Existence*) | [10-define-errors-out-of-existence.md](./10-define-errors-out-of-existence.md) | 84 – 98 |
| 11 | **Chương 11:** Thiết kế hai lần (*Design it Twice*) | [11-design-it-twice.md](./11-design-it-twice.md) | 99 – 101 |
| 12 | **Chương 12:** Tại sao phải viết Comment? Bốn lời bào chữa (*Why Write Comments? The Four Excuses*) | [12-why-write-comments.md](./12-why-write-comments.md) | 102 – 106 |
| 13 | **Chương 13:** Comment nên mô tả những điều không hiển nhiên từ mã nguồn (*Comments Should Describe Things that Aren't Obvious from the Code*) | [13-comments-describe-non-obvious.md](./13-comments-describe-non-obvious.md) | 107 – 128 |
| 14 | **Chương 14:** Chọn tên gọi (*Choosing Names*) | [14-choosing-names.md](./14-choosing-names.md) | 129 – 136 |
| 15 | **Chương 15:** Hãy viết Comment trước (*Write The Comments First*) | [15-write-the-comments-first.md](./15-write-the-comments-first.md) | 137 – 141 |
| 16 | **Chương 16:** Chỉnh sửa mã nguồn hiện có (*Modifying Existing Code*) | [16-modifying-existing-code.md](./16-modifying-existing-code.md) | 142 – 147 |
| 17 | **Chương 17:** Tính nhất quán (*Consistency*) | [17-consistency.md](./17-consistency.md) | 148 – 151 |
| 18 | **Chương 18:** Mã nguồn phải hiển nhiên dễ hiểu (*Code Should be Obvious*) | [18-code-should-be-obvious.md](./18-code-should-be-obvious.md) | 152 – 158 |
| 19 | **Chương 19:** Các xu hướng phần mềm (*Software Trends*) | [19-software-trends.md](./19-software-trends.md) | 159 – 165 |
| 20 | **Chương 20:** Thiết kế hướng hiệu năng (*Designing for Performance*) | [20-designing-for-performance.md](./20-designing-for-performance.md) | 166 – 175 |
| 21 | **Chương 21 & Tổng kết:** Kết luận, Tổng hợp Nguyên tắc Thiết kế, Tổng hợp Cờ Đỏ, Chỉ mục & Tác giả (*Conclusion & Summaries*) | [21-conclusion-and-summaries.md](./21-conclusion-and-summaries.md) | 176 – 188 |
