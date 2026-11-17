# CHƯƠNG 21: Kết luận (Chapter 21: Conclusion)

Cuốn sách này nói về một điều duy nhất: **sự phức tạp (complexity)**. Đối phó với sự phức tạp là thách thức quan trọng nhất trong thiết kế phần mềm. Chính sự phức tạp là thứ khiến các hệ thống trở nên khó xây dựng và bảo trì, đồng thời thường khiến chúng chạy chậm hơn. Xuyên suốt cuốn sách, tôi đã cố gắng mô tả các nguyên nhân gốc rễ dẫn đến sự phức tạp, chẳng hạn như các mối phụ thuộc (dependencies) và sự tối nghĩa (obscurity). Tôi đã thảo luận về các **cờ đỏ (red flags)** giúp bạn nhận diện sự phức tạp không cần thiết, chẳng hạn như rò rỉ thông tin (information leakage), các điều kiện lỗi dư thừa (unneeded error conditions), hay những cái tên quá chung chung. Tôi đã trình bày một số ý tưởng tổng quát mà bạn có thể sử dụng để tạo ra các hệ thống phần mềm đơn giản hơn, chẳng hạn như phấn đấu xây dựng các lớp (classes) vừa **sâu (deep)** vừa **tổng quát (generic)**, **định nghĩa lỗi sao cho không còn tồn tại (defining errors out of existence)**, và tách biệt tài liệu giao diện (interface documentation) khỏi tài liệu triển khai (implementation documentation). Và cuối cùng, tôi đã thảo luận về **tư duy đầu tư (investment mindset)** cần thiết để tạo ra những thiết kế đơn giản.

Mặt trái của tất cả những gợi ý này là chúng tạo ra thêm công việc ở những giai đoạn đầu của dự án. Hơn nữa, nếu bạn chưa quen với việc suy nghĩ về các vấn đề thiết kế, bạn sẽ còn chậm lại nhiều hơn nữa trong quá trình học các kỹ thuật thiết kế tốt. Nếu điều duy nhất quan trọng đối với bạn là làm cho đoạn mã hiện tại chạy được càng nhanh càng tốt, thì việc suy nghĩ về thiết kế sẽ giống như một công việc cực nhọc cản trở mục tiêu thực sự của bạn.

Mặt khác, nếu thiết kế tốt là một mục tiêu quan trọng đối với bạn, thì những ý tưởng trong cuốn sách này sẽ giúp việc lập trình trở nên thú vị hơn. Thiết kế là một câu đố hấp dẫn: làm thế nào để giải quyết một vấn đề cụ thể với cấu trúc đơn giản nhất có thể? Thật thú vị khi khám phá các cách tiếp cận khác nhau, và thật tuyệt vời khi tìm ra một giải pháp vừa đơn giản vừa mạnh mẽ. Một thiết kế sạch sẽ, đơn giản và hiển nhiên (obvious) là một điều tuyệt đẹp.

Hơn nữa, những khoản đầu tư bạn bỏ ra cho thiết kế tốt sẽ nhanh chóng mang lại trái ngọt. Các module mà bạn định nghĩa cẩn thận ngay từ đầu dự án sẽ tiết kiệm thời gian cho bạn về sau khi bạn tái sử dụng chúng hết lần này đến lần khác. Tài liệu rõ ràng mà bạn viết cách đây sáu tháng sẽ tiết kiệm thời gian khi bạn quay lại mã nguồn để thêm một tính năng mới. Thời gian bạn dành ra để trau dồi kỹ năng thiết kế cũng sẽ tự bù đắp cho chính nó: khi kỹ năng và kinh nghiệm của bạn tăng lên, bạn sẽ thấy mình có thể tạo ra các thiết kế tốt ngày càng nhanh hơn. Thiết kế tốt thực sự không tốn nhiều thời gian hơn thiết kế làm ẩu cho xong (quick-and-dirty design), một khi bạn đã biết cách làm.

Phần thưởng cho việc trở thành một nhà thiết kế giỏi là bạn được dành một phần lớn hơn thời gian của mình trong giai đoạn thiết kế — vốn là giai đoạn rất thú vị. Những nhà thiết kế kém dành phần lớn thời gian để săn lỗi (chasing bugs) trong những đoạn mã phức tạp và mong manh (brittle). Nếu bạn cải thiện kỹ năng thiết kế của mình, bạn không chỉ tạo ra phần mềm chất lượng cao hơn một cách nhanh chóng hơn, mà bản thân quá trình phát triển phần mềm cũng sẽ trở nên đáng tận hưởng hơn.

> 💡 **Ví dụ bổ sung (không có trong sách):** Hãy hình dung hai nhóm phát triển cùng xây dựng một dịch vụ thanh toán. Nhóm A viết mã kiểu "chạy được là xong" (tactical): mỗi cổng thanh toán mới (VNPay, MoMo, Stripe) lại sao chép logic kiểm tra chữ ký, thử lại (retry), và ghi log vào từng controller riêng lẻ. Sau 6 tháng, chỉ một thay đổi nhỏ về chính sách retry buộc họ phải sửa 12 nơi và phát sinh 4 lỗi hồi quy (regression bugs). Nhóm B dành thêm 2 ngày đầu để thiết kế một abstraction `PaymentGateway` sâu, ẩn hoàn toàn cơ chế retry và chuẩn hóa lỗi vào bên trong tầng lõi. Sau 6 tháng, khi thêm cổng thanh toán thứ tư, họ chỉ mất nửa ngày để triển khai một adapter nhỏ gọn và hầu như không phát sinh lỗi nào. Khoản đầu tư ban đầu đã sinh lời gấp nhiều lần.

---

## Tóm tắt các Nguyên lý Thiết kế (Summary of Design Principles)

Dưới đây là những nguyên lý thiết kế phần mềm quan trọng nhất được thảo luận trong cuốn sách này:

1. **Sự phức tạp mang tính tích lũy: bạn phải chăm chút từng chi tiết nhỏ** *(Complexity is incremental: you have to sweat the small stuff — xem trang 11).*
2. **Mã chạy được thôi là chưa đủ** *(Working code isn’t enough — xem trang 14).*
3. **Hãy thực hiện những khoản đầu tư nhỏ liên tục để cải thiện thiết kế hệ thống** *(Make continual small investments to improve system design — xem trang 15).*
4. **Các module nên sâu** *(Modules should be deep — xem trang 22).*
5. **Giao diện nên được thiết kế để giúp cách sử dụng phổ biến nhất trở nên đơn giản nhất có thể** *(Interfaces should be designed to make the most common usage as simple as possible — xem trang 27).*
6. **Việc một module có giao diện đơn giản quan trọng hơn việc nó có phần triển khai đơn giản** *(It’s more important for a module to have a simple interface than a simple implementation — xem các trang 55, 71).*
7. **Các module tổng quát thường sâu hơn** *(General-purpose modules are deeper — xem trang 39).*
8. **Tách biệt mã tổng quát và mã chuyên biệt** *(Separate general-purpose and special-purpose code — xem trang 62).*
9. **Các tầng khác nhau nên có các trừu tượng hóa khác nhau** *(Different layers should have different abstractions — xem trang 45).*
10. **Kéo sự phức tạp xuống dưới** *(Pull complexity downward — xem trang 55).*
11. **Định nghĩa lỗi (và các trường hợp đặc biệt) sao cho chúng không còn tồn tại** *(Define errors (and special cases) out of existence — xem trang 79).*
12. **Hãy thiết kế hai lần** *(Design it twice — xem trang 91).*
13. **Chú thích (comments) nên mô tả những điều không hiển nhiên từ mã nguồn** *(Comments should describe things that are not obvious from the code — xem trang 101).*
14. **Phần mềm nên được thiết kế để dễ đọc, chứ không phải để dễ viết** *(Software should be designed for ease of reading, not ease of writing — xem trang 149).*
15. **Các bước tăng trưởng của quá trình phát triển phần mềm nên là các trừu tượng hóa (abstractions), chứ không phải các tính năng (features)** *(The increments of software development should be abstractions, not features — xem trang 154).*

---

## Tóm tắt các Cờ Đỏ (Summary of Red Flags)

Dưới đây là một số cờ đỏ quan trọng nhất được thảo luận trong cuốn sách này. Sự xuất hiện của bất kỳ triệu chứng nào trong số này trong một hệ thống đều là dấu hiệu cho thấy thiết kế của hệ thống đang có vấn đề:

> 🚩 **Cờ Đỏ (Red Flag — Module nông / Shallow Module):** Giao diện của một lớp hoặc phương thức không đơn giản hơn bao nhiêu so với phần triển khai của nó *(xem các trang 25, 110)*.

> 🚩 **Cờ Đỏ (Red Flag — Rò rỉ thông tin / Information Leakage):** Một quyết định thiết kế bị phản ánh ở nhiều module khác nhau *(xem trang 31)*.

> 🚩 **Cờ Đỏ (Red Flag — Phân rã theo trình tự thời gian / Temporal Decomposition):** Cấu trúc mã nguồn dựa trên thứ tự thực thi của các thao tác, chứ không dựa trên việc che giấu thông tin *(information hiding — xem trang 32)*.

> 🚩 **Cờ Đỏ (Red Flag — Phơi bày quá mức / Overexposure):** Một API buộc bên gọi (callers) phải biết đến các tính năng hiếm khi dùng chỉ để sử dụng các tính năng thường dùng *(xem trang 36)*.

> 🚩 **Cờ Đỏ (Red Flag — Phương thức truyền qua / Pass-Through Method):** Một phương thức hầu như không làm gì ngoài việc truyền các tham số của nó sang một phương thức khác có chữ ký (signature) tương tự *(xem trang 46)*.

> 🚩 **Cờ Đỏ (Red Flag — Lặp lại / Repetition):** Một đoạn mã không tầm thường bị lặp đi lặp lại hết lần này đến lần khác *(xem trang 62)*.

> 🚩 **Cờ Đỏ (Red Flag — Trộn lẫn chuyên biệt và tổng quát / Special-General Mixture):** Mã chuyên biệt (special-purpose code) không được tách bạch rõ ràng khỏi mã tổng quát (general-purpose code) *(xem trang 65)*.

> 🚩 **Cờ Đỏ (Red Flag — Các phương thức dính liền / Conjoined Methods):** Hai phương thức có quá nhiều mối phụ thuộc lẫn nhau đến mức khó có thể hiểu phần triển khai của phương thức này mà không hiểu phần triển khai của phương thức kia *(xem trang 72)*.

> 🚩 **Cờ Đỏ (Red Flag — Chú thích lặp lại mã nguồn / Comment Repeats Code):** Toàn bộ thông tin trong một dòng chú thích đều hiển nhiên ngay lập tức từ đoạn mã nằm ngay bên cạnh nó *(xem trang 104)*.

> 🚩 **Cờ Đỏ (Red Flag — Tài liệu triển khai làm nhiễm bẩn giao diện / Implementation Documentation Contaminates Interface):** Một chú thích giao diện (interface comment) mô tả các chi tiết triển khai mà người dùng của thành phần đang được tài liệu hóa không cần biết *(xem trang 114)*.

> 🚩 **Cờ Đỏ (Red Flag — Tên mơ hồ / Vague Name):** Tên của một biến hoặc phương thức quá thiếu chính xác đến nỗi nó không truyền đạt được nhiều thông tin hữu ích *(xem trang 123)*.

> 🚩 **Cờ Đỏ (Red Flag — Khó đặt tên / Hard to Pick Name):** Rất khó để nghĩ ra một cái tên vừa chính xác vừa trực quan cho một thực thể *(xem trang 125)*.

> 🚩 **Cờ Đỏ (Red Flag — Khó mô tả / Hard to Describe):** Để mô tả đầy đủ, phần tài liệu cho một biến hoặc phương thức buộc phải viết rất dài *(xem trang 131)*.

> 🚩 **Cờ Đỏ (Red Flag — Mã không hiển nhiên / Nonobvious Code):** Hành vi hoặc ý nghĩa của một đoạn mã không thể được hiểu một cách dễ dàng *(xem trang 148)*.

---

## Về Tác giả (About the Author)

**John Ousterhout** là Giáo sư Khoa học Máy tính (chức danh *Bosack Lerner Professor of Computer Science*) tại Đại học Stanford. Ông là người sáng tạo ra ngôn ngữ kịch bản **Tcl** và cũng nổi tiếng với các công trình nghiên cứu về hệ điều hành phân tán (distributed operating systems) và hệ thống lưu trữ (storage systems). Ousterhout nhận bằng Cử nhân (BS) ngành Vật lý tại Đại học Yale và bằng Tiến sĩ (PhD) ngành Khoa học Máy tính tại Đại học Carnegie Mellon. Ông là thành viên của Viện Hàn lâm Kỹ thuật Quốc gia Hoa Kỳ (National Academy of Engineering) và đã nhận được nhiều giải thưởng danh giá, bao gồm Giải thưởng Hệ thống Phần mềm ACM (*ACM Software System Award*), Giải thưởng *ACM Grace Murray Hopper*, Giải thưởng Nhà nghiên cứu Trẻ của Tổng thống thuộc Quỹ Khoa học Quốc gia (*National Science Foundation Presidential Young Investigator Award*), và Giải thưởng Giảng dạy Xuất sắc của Đại học U.C. Berkeley (*U.C. Berkeley Distinguished Teaching Award*).

---

## Bảng tra cứu thuật ngữ Anh–Việt (Index / Reference Note)

> 📝 **Giải thích thêm (không có trong sách):** Trong bản gốc tiếng Anh, phần **Index** (các trang 178–184) liệt kê chỉ mục thuật ngữ và ví dụ theo thứ tự bảng chữ cái kèm số trang của sách gốc. Để thuận tiện cho người đọc bản dịch tiếng Việt khi đối chiếu khái niệm giữa các chương hoặc tìm lại thuật ngữ gốc, dưới đây là bảng tra cứu tổng hợp các mục chính từ Index của sách kèm nghĩa tiếng Việt tương ứng và số trang tham chiếu trong bản gốc.

| Thuật ngữ gốc (English Index) | Thuật ngữ tiếng Việt | Trang trong sách gốc |
| :--- | :--- | :--- |
| abstraction | trừu tượng hóa | 21 |
| aggregating exceptions | gom nhóm ngoại lệ | 82 |
| agile development | phát triển linh hoạt (Agile) | 2, 153 |
| change amplification | khuếch đại thay đổi | 7, 99 |
| classitis | hội chứng chia nhỏ lớp quá mức | 26 |
| coding style | phong cách viết mã | 141 |
| cognitive load | gánh nặng nhận thức | 7, 43, 99 |
| comments (as design tool, benefits, canary in the coal mine, conventions, duplication, for intuition, for precision, implementation, interface, near code, obsolete, procrastination, repeating code, role in abstraction, worthless, writing before code) | chú thích (công cụ thiết kế, lợi ích, dấu hiệu cảnh báo sớm, quy ước, trùng lặp, tạo trực giác, tạo độ chính xác, triển khai, giao diện, gần mã nguồn, lỗi thời, trì hoãn, lặp lại mã, vai trò trong trừu tượng hóa, vô giá trị, viết trước mã) | 98, 101–107, 110, 116, 129, 131, 137–138 |
| complexity (causes, definition, incremental nature, pulling downwards, symptoms) | sự phức tạp (nguyên nhân, định nghĩa, tính tích lũy, kéo xuống dưới, triệu chứng) | 5, 7, 9, 11, 55, 82, 161 |
| composition | hợp thành (composition) | 152 |
| configuration parameters | tham số cấu hình | 56 |
| conjoined methods | các phương thức dính liền | 71 |
| consistency | tính nhất quán | 141, 146 |
| context object | đối tượng ngữ cảnh | 51 |
| cross-module design decisions | quyết định thiết kế xuyên module | 117 |
| decorator | mẫu Decorator | 49 |
| deep module | module sâu | 22 |
| defaults | giá trị mặc định | 36 |
| dependency | mối phụ thuộc | 9 |
| design it twice | thiết kế hai lần | 91 |
| design patterns | mẫu thiết kế (design patterns) | 142, 156 |
| `designNotes` file | tệp `designNotes` | 118, 139 |
| dispatcher | bộ điều phối (dispatcher) | 47 |
| do the right thing | tự làm điều đúng đắn | 36 |
| event-driven programming | lập trình hướng sự kiện | 148 |
| exception (aggregation, masking) | ngoại lệ (gom nhóm, che giấu) | 75, 81–82 |
| false abstraction | trừu tượng hóa giả tạo | 22, 43 |
| general-purpose class / code | lớp / mã tổng quát | 40, 62, 66–67 |
| generic containers | cấu trúc chứa tổng quát | 149 |
| getter / setter | phương thức getter / setter | 156 |
| global variable | biến toàn cục | 51 |
| implementation / implementation documentation | phần triển khai / tài liệu triển khai | 19, 50, 116 |
| implementation inheritance / interface inheritance | kế thừa triển khai / kế thừa giao diện | 151–152 |
| incremental development | phát triển tăng trưởng | 2, 39 |
| information hiding / information leakage | che giấu thông tin / rò rỉ thông tin | 29–30 |
| inheritance | kế thừa | 151 |
| interface (formal parts, informal parts, documentation, comment) | giao diện (phần chính thức, phần phi chính thức, tài liệu, chú thích) | 19–21, 50, 110 |
| invariants | bất biến (invariants) | 142 |
| investment mindset | tư duy đầu tư | 15, 128, 136, 144 |
| long method | phương thức dài | 70 |
| masking exceptions | che giấu ngoại lệ | 81 |
| modular design / module | thiết kế module / module | 2, 19–20 |
| names (consistency, generic, how to choose, making code obvious, precise, short names in Go) | đặt tên (nhất quán, chung chung, cách chọn, làm mã hiển nhiên, chính xác, tên ngắn trong Go) | 121, 123, 126, 141, 146 |
| object-oriented programming | lập trình hướng đối tượng | 151 |
| obscurity / obvious code | sự tối nghĩa / mã hiển nhiên | 9–10, 145 |
| pass-through method / pass-through variable | phương thức truyền qua / biến truyền qua | 46, 50 |
| performance, designing for (disk I/O, flash storage, garbage collection, dynamic memory allocation, micro-benchmark, network communication, nonvolatile memory) | thiết kế hướng hiệu năng (I/O đĩa, lưu trữ flash, thu gom rác, cấp phát bộ nhớ động, micro-benchmark, giao tiếp mạng, bộ nhớ bất biến) | 159–160 |
| private variables | biến riêng tư (private) | 30 |
| self-documenting code | mã tự tài liệu hóa | 96 |
| shallow module / small classes | module nông / các lớp nhỏ | 25–26 |
| special-purpose code | mã chuyên biệt | 62, 67 |
| specification, formal | đặc tả chính thức | 21 |
| strategic programming / tactical programming / tactical tornado | lập trình chiến lược / lập trình chiến thuật / cơn lốc chiến thuật | 13–14, 135, 153 |
| temporal decomposition | phân rã theo trình tự thời gian | 31 |
| test-driven development (TDD) / tests (unit, integration, system) | phát triển hướng kiểm thử / kiểm thử (đơn vị, tích hợp, hệ thống) | 154–155 |
| `try` block | khối `try` | 77 |
| unknown unknowns | những điều chưa biết mà ta không biết | 8, 99 |
| waterfall model | mô hình thác nước (waterfall) | 2 |
| white space | khoảng trắng | 146 |

### Các ví dụ minh họa tiêu biểu trong sách (Examples Index)

- **Editor text class / selection & cursor / undo / fence for undo / non-existent selection** *(Lớp văn bản trình soạn thảo / vùng chọn & con trỏ / hoàn tác / mốc rào hoàn tác / vùng chọn không tồn tại)*: trang 40, 50, 56, 65, 67, 69, 87, 91.
- **HTTP server / HTTP parameters / HTTP response / URL encoding** *(Máy chủ HTTP / tham số HTTP / phản hồi HTTP / mã hóa URL)*: trang 32, 34, 36, 60.
- **Java I/O / Unix I/O / file descriptor / Java substring / file data loss / file deletion** *(Java I/O / Unix I/O / bộ mô tả tệp / cắt chuỗi Java / mất dữ liệu tệp / xóa tệp)*: trang 23, 26, 49, 61, 79–80, 121.
- **RAMCloud Buffer / RAMCloud error promotion / RAMCloud Status / IndexLookup** *(Bộ đệm RAMCloud / nâng cấp lỗi RAMCloud / trạng thái RAMCloud / IndexLookup)*: trang 85, 112, 117, 163.
- **Linked list / configuration parameters / missing parameter / NFS server crash / out of memory / Tcl unset / Web site colors** *(Danh sách liên kết / tham số cấu hình / thiếu tham số / sập máy chủ NFS / hết bộ nhớ / lệnh `unset` trong Tcl / màu sắc trang web)*: trang 7, 25, 56, 78, 81–82, 86.
- **Các tổ chức & nhân vật được nhắc tới**: David Parnas (trang 29), Google (trang 17), Facebook (trang 17), VMware (trang 17), ngôn ngữ Go (trang 126).
