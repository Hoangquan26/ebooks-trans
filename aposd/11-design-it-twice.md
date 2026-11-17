# CHƯƠNG 11: Thiết kế hai lần (Chapter 11: Design it Twice)

Thiết kế phần mềm là một việc khó, vì vậy rất khó có khả năng những suy nghĩ đầu tiên của bạn về cách cấu trúc một mô-đun hoặc một hệ thống sẽ tạo ra thiết kế tốt nhất. Bạn sẽ đạt được kết quả tốt hơn nhiều nếu cân nhắc nhiều phương án khác nhau cho mỗi quyết định thiết kế quan trọng: hãy **thiết kế hai lần** (*design it twice*).

Giả sử bạn đang thiết kế lớp (*class*) sẽ quản lý văn bản của một tệp cho một trình soạn thảo văn bản có giao diện đồ họa (GUI). Bước đầu tiên là xác định giao diện (*interface*) mà lớp này sẽ cung cấp cho phần còn lại của trình soạn thảo; thay vì chọn ngay ý tưởng đầu tiên nảy ra trong đầu, hãy cân nhắc một vài khả năng. Một lựa chọn là giao diện hướng dòng (*line-oriented interface*), với các thao tác chèn, sửa và xóa toàn bộ các dòng văn bản. Một lựa chọn khác là giao diện dựa trên việc chèn và xóa từng ký tự riêng lẻ (*character-oriented*). Lựa chọn thứ ba là giao diện hướng chuỗi (*string-oriented interface*), thao tác trên các vùng ký tự bất kỳ có thể vắt qua ranh giới giữa các dòng. Ở giai đoạn này, bạn chưa cần chốt mọi tính năng của từng phương án; chỉ cần phác thảo một vài phương thức quan trọng nhất là đủ.

> 💡 **Ví dụ bổ sung (không có trong sách):** Để hình dung rõ hơn ba phương án giao diện cho lớp quản lý văn bản (`TextBuffer`), bạn có thể phác thảo nhanh như sau:
>
> ```java
> // Phương án 1: Hướng dòng (Line-oriented)
> interface LineTextBuffer {
>     String getLine(int lineNum);
>     void insertLine(int lineNum, String text);
>     void replaceLine(int lineNum, String text);
>     void deleteLine(int lineNum);
> }
>
> // Phương án 2: Hướng ký tự (Character-oriented)
> interface CharTextBuffer {
>     char getChar(int index);
>     void insertChar(int index, char c);
>     void deleteChar(int index);
> }
>
> // Phương án 3: Hướng chuỗi / vùng (String/Range-oriented)
> interface RangeTextBuffer {
>     String getText(Position start, Position end);
>     void insertText(Position pos, String text);
>     void deleteText(Position start, Position end);
> }
> ```
>
> Chỉ cần phác thảo vài dòng như vậy, ta đã thấy ngay sự khác biệt về độ tiện dụng đối với mã gọi ở tầng trên.

Hãy cố gắng chọn những hướng tiếp cận khác biệt hoàn toàn với nhau; bạn sẽ học được nhiều hơn theo cách đó. Ngay cả khi bạn tin chắc rằng chỉ có một hướng tiếp cận hợp lý duy nhất, hãy cứ cân nhắc một thiết kế thứ hai, bất kể bạn nghĩ nó tệ đến mức nào. Việc suy nghĩ về những điểm yếu của thiết kế đó và đối chiếu chúng với các đặc điểm của những thiết kế khác sẽ mang lại rất nhiều bài học bổ ích.

Sau khi đã phác thảo sơ bộ thiết kế cho các phương án, hãy lập một danh sách ưu và nhược điểm của từng phương án. Yếu tố quan trọng nhất cần cân nhắc đối với một giao diện là **mức độ dễ sử dụng đối với phần mềm ở tầng cao hơn**. Trong ví dụ trên, cả giao diện hướng dòng lẫn giao diện hướng ký tự đều đòi hỏi phần mềm sử dụng lớp văn bản phải làm thêm việc. Giao diện hướng dòng sẽ buộc phần mềm ở tầng cao hơn phải tự tách và nối các dòng trong những thao tác trên một phần dòng hoặc nhiều dòng, chẳng hạn như cắt và dán vùng chọn (*selection*). Giao diện hướng ký tự sẽ buộc phải dùng các vòng lặp để thực hiện những thao tác sửa đổi nhiều hơn một ký tự. Ngoài ra, cũng đáng để cân nhắc các yếu tố khác:

* Một phương án có giao diện đơn giản hơn phương án khác không? Trong ví dụ về văn bản, tất cả các giao diện văn bản đều tương đối đơn giản.
* Một giao diện có mang tính tổng quát hơn (*more general-purpose*) giao diện khác không?
* Một giao diện có cho phép cài đặt hiệu quả hơn (*more efficient implementation*) giao diện khác không? Trong ví dụ về văn bản, hướng tiếp cận theo từng ký tự nhiều khả năng sẽ chậm hơn đáng kể so với các hướng khác, vì nó đòi hỏi một lời gọi riêng biệt vào mô-đun văn bản cho mỗi ký tự.

Sau khi đã so sánh các thiết kế thay thế, bạn sẽ ở vào vị trí tốt hơn để xác định thiết kế tối ưu. Lựa chọn tốt nhất có thể là một trong các phương án ban đầu, hoặc bạn có thể nhận ra rằng mình có thể kết hợp các điểm mạnh của nhiều phương án thành một thiết kế mới tốt hơn bất kỳ lựa chọn ban đầu nào.

Đôi khi không có phương án nào thực sự hấp dẫn; khi điều này xảy ra, hãy xem liệu bạn có thể nghĩ ra thêm các phương án khác hay không. Hãy dùng chính những vấn đề bạn đã phát hiện ở các phương án ban đầu để dẫn dắt cho (các) thiết kế mới. Nếu khi thiết kế lớp văn bản mà bạn chỉ cân nhắc hai hướng tiếp cận hướng dòng và hướng ký tự, bạn có thể nhận thấy rằng mỗi phương án đều gượng gạo vì nó đòi hỏi phần mềm ở tầng cao hơn phải tự thực hiện thêm các thao tác xử lý văn bản. Đó là một **cờ đỏ** (*red flag*): nếu đã có một lớp quản lý văn bản, thì chính nó phải đảm nhận toàn bộ việc thao tác văn bản. Để loại bỏ các thao tác xử lý văn bản dư thừa ở tầng trên, giao diện văn bản cần khớp sát hơn với các thao tác đang diễn ra ở phần mềm tầng cao hơn. Những thao tác này không phải lúc nào cũng tương ứng với từng ký tự đơn lẻ hay từng dòng đơn lẻ. Mạch suy luận này sẽ dẫn bạn tới một API hướng vùng (*range-oriented API*) cho văn bản, giúp loại bỏ vấn đề của các thiết kế trước đó.

> 🚩 **Cờ Đỏ (Red Flag — Lớp không tự đảm nhận hết phần việc của mình):** Khi một giao diện buộc phần mềm gọi ở tầng cao hơn phải tự thực hiện thêm các thao tác thuộc về trách nhiệm cốt lõi của mô-đun đó (ví dụ: tầng trên phải tự tách/nối dòng hay tự lặp qua từng ký tự khi làm việc với lớp quản lý văn bản), đó là dấu hiệu cho thấy giao diện chưa khớp với nhu cầu thực tế và cần được thiết kế lại.

> 📝 **Giải thích thêm (không có trong sách):** Quy trình "thiết kế hai lần" có thể tóm tắt qua sơ đồ dưới đây:
>
> ```mermaid
> flowchart TD
>     A["Xác định bài toán / quyết định thiết kế lớn"] --> B["Phác thảo 2-3 phương án khác biệt rõ rệt"]
>     B --> C["Liệt kê ưu và nhược điểm của từng phương án"]
>     C --> D{"Đã có phương án đủ tốt?"}
>     D -- "Có" --> E["Chọn phương án tốt nhất"]
>     D -- "Kết hợp được" --> F["Tổng hợp điểm mạnh thành thiết kế mới"]
>     D -- "Chưa phương án nào ổn" --> G["Dùng các nhược điểm vừa tìm ra để nghĩ phương án mới"]
>     G --> C
> ```

Nguyên lý thiết kế hai lần có thể được áp dụng ở nhiều cấp độ trong một hệ thống. Đối với một mô-đun, trước tiên bạn có thể áp dụng cách tiếp cận này để chọn giao diện, như đã mô tả ở trên. Sau đó, bạn có thể áp dụng nó một lần nữa khi thiết kế phần cài đặt (*implementation*): đối với lớp văn bản, bạn có thể cân nhắc các cách cài đặt như danh sách liên kết các dòng (*linked list of lines*), các khối ký tự có kích thước cố định (*fixed-size blocks of characters*), hoặc một "bộ đệm khoảng trống" (*gap buffer*). Mục tiêu đối với phần cài đặt sẽ khác với phần giao diện: đối với phần cài đặt, những điều quan trọng nhất là tính đơn giản (*simplicity*) và hiệu năng (*performance*). Việc khám phá nhiều thiết kế ở các cấp độ cao hơn trong hệ thống cũng rất hữu ích, chẳng hạn như khi chọn các tính năng cho giao diện người dùng, hoặc khi phân rã một hệ thống thành các mô-đun lớn. Trong mọi trường hợp, việc xác định hướng tiếp cận tốt nhất sẽ dễ dàng hơn nếu bạn có thể so sánh một vài phương án thay thế.

> 📝 **Giải thích thêm (không có trong sách):** **Bộ đệm khoảng trống (*gap buffer*)** là một cấu trúc dữ liệu kinh điển dùng trong các trình soạn thảo văn bản (như Emacs). Toàn bộ văn bản được lưu trong một mảng ký tự lớn liên tiếp, nhưng có một "khoảng trống" (vùng chưa dùng) nằm ngay tại vị trí con trỏ soạn thảo. Nhờ vậy, việc gõ hoặc xóa ký tự liên tiếp tại vị trí con trỏ diễn ra cực nhanh ($O(1)$) vì chỉ cần nới rộng hoặc thu hẹp khoảng trống ngay tại chỗ mà không phải dịch chuyển phần còn lại của tệp.

Việc thiết kế hai lần không cần phải tốn nhiều thời gian phụ trội. Đối với một mô-đun nhỏ hơn như một lớp (*class*), bạn có thể chỉ cần không quá một hoặc hai giờ để cân nhắc các phương án. Đây là một khoảng thời gian rất nhỏ so với nhiều ngày hoặc nhiều tuần bạn sẽ dành ra để cài đặt lớp đó. Những thử nghiệm thiết kế ban đầu nhiều khả năng sẽ mang lại một thiết kế tốt hơn đáng kể, hoàn toàn bù đắp xứng đáng cho thời gian bạn bỏ ra để thiết kế hai lần. Đối với các mô-đun lớn hơn, bạn sẽ dành nhiều thời gian hơn cho việc khám phá thiết kế ban đầu, nhưng thời gian cài đặt cũng sẽ lâu hơn, và lợi ích của một thiết kế tốt hơn cũng sẽ cao hơn.

Tôi nhận thấy rằng nguyên lý thiết kế hai lần đôi khi rất khó được những người cực kỳ thông minh đón nhận. Khi lớn lên, những người thông minh nhận ra rằng ý tưởng nhanh đầu tiên của họ về bất kỳ vấn đề nào cũng đủ để đạt điểm cao; chẳng cần phải cân nhắc khả năng thứ hai hay thứ ba làm gì. Điều này khiến họ dễ hình thành thói quen làm việc xấu. Tuy nhiên, khi những người này trưởng thành hơn, họ được thăng tiến vào những môi trường có những bài toán ngày càng khó hơn. Cuối cùng, ai rồi cũng chạm tới một ngưỡng mà những ý tưởng đầu tiên không còn đủ tốt nữa; nếu bạn muốn đạt được kết quả thực sự xuất sắc, bạn phải cân nhắc khả năng thứ hai, hoặc có lẽ là thứ ba, bất kể bạn thông minh đến đâu. Việc thiết kế các hệ thống phần mềm lớn thuộc vào nhóm này: không một ai đủ giỏi để làm đúng ngay trong lần thử đầu tiên.

Thật không may, tôi thường thấy những người thông minh cứ khăng khăng cài đặt ý tưởng đầu tiên nảy ra trong đầu, và điều này khiến họ không phát huy hết tiềm năng thực sự của mình (đồng thời cũng khiến việc cộng tác với họ trở nên đầy bực bội). Có lẽ trong tiềm thức họ tin rằng "người thông minh là người làm đúng ngay từ lần đầu", nên nếu phải thử nhiều thiết kế thì hóa ra mình chẳng thông minh chút nào. Sự thật không phải như vậy. Không phải là bạn không thông minh; mà là vì các bài toán thực sự rất khó! Hơn thế nữa, đó lại là một điều tốt: làm việc trên một bài toán khó buộc bạn phải suy nghĩ thấu đáo sẽ thú vị hơn nhiều so với một bài toán dễ đến mức bạn chẳng cần phải động não chút nào.

Cách tiếp cận thiết kế hai lần không chỉ cải thiện các thiết kế của bạn, mà còn cải thiện chính kỹ năng thiết kế của bạn. Quá trình nghĩ ra và so sánh nhiều hướng tiếp cận sẽ dạy cho bạn về những yếu tố làm cho một thiết kế trở nên tốt hơn hay tệ hơn. Theo thời gian, điều này sẽ giúp bạn dễ dàng loại bỏ các thiết kế tồi và nhắm trúng những thiết kế thực sự xuất sắc.

---

## Bảng thuật ngữ chương (Chapter Glossary)

| Thuật ngữ (English) | Tiếng Việt | Giải thích ngắn gọn |
| :--- | :--- | :--- |
| Design it twice | Thiết kế hai lần | Nguyên lý cân nhắc và so sánh ít nhất 2–3 phương án thiết kế khác biệt trước khi chọn phương án tốt nhất. |
| Line-oriented interface | Giao diện hướng dòng | Giao diện thao tác theo đơn vị từng dòng văn bản hoàn chỉnh. |
| Character-oriented interface | Giao diện hướng ký tự | Giao diện thao tác dựa trên việc đọc, chèn, xóa từng ký tự riêng lẻ. |
| String-oriented / Range-oriented API | Giao diện hướng chuỗi / hướng vùng | Giao diện thao tác trên các vùng ký tự bất kỳ, có thể vắt qua nhiều dòng. |
| Higher-level software | Phần mềm ở tầng cao hơn | Các mô-đun hoặc mã nguồn phía trên gọi và sử dụng giao diện của lớp/mô-đun đang thiết kế. |
| General-purpose | Mang tính tổng quát | Khả năng phục vụ nhiều mục đích hoặc tình huống sử dụng khác nhau thay vì chỉ một trường hợp hẹp. |
| Implementation | Phần cài đặt / hiện thực hóa | Cấu trúc dữ liệu và thuật toán bên trong để thực thi giao diện đã định nghĩa. |
| Gap buffer | Bộ đệm khoảng trống | Cấu trúc dữ liệu mảng có một khoảng trống tại vị trí con trỏ để tối ưu việc chèn/xóa văn bản. |
| Red flag | Cờ đỏ | Dấu hiệu cảnh báo một vấn đề hoặc sai sót tiềm ẩn trong thiết kế phần mềm. |
