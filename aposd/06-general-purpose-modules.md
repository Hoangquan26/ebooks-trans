# CHƯƠNG 6: Các Mô-đun Đa dụng Thì Sâu Hơn (Chapter 6: General-Purpose Modules are Deeper)

Một trong những quyết định phổ biến nhất mà bạn sẽ phải đối mặt khi thiết kế một mô-đun mới là nên triển khai nó theo hướng đa dụng (general-purpose) hay chuyên dụng (special-purpose). Một số người có thể lập luận rằng bạn nên chọn cách tiếp cận đa dụng, trong đó bạn xây dựng một cơ chế có thể dùng để giải quyết nhiều bài toán khác nhau, chứ không chỉ những bài toán quan trọng ở thời điểm hiện tại. Trong trường hợp đó, cơ chế mới có thể tìm thấy những công dụng không lường trước được trong tương lai, qua đó tiết kiệm thời gian. Cách tiếp cận đa dụng có vẻ nhất quán với tư duy đầu tư (investment mindset) đã thảo luận ở Chương 3, nơi bạn bỏ ra thêm một chút thời gian ngay từ đầu để tiết kiệm thời gian về sau.

Mặt khác, chúng ta biết rằng rất khó để dự đoán các nhu cầu tương lai của một hệ thống phần mềm, nên một giải pháp đa dụng có thể bao gồm những tính năng chẳng bao giờ thực sự được dùng đến. Hơn nữa, nếu bạn triển khai một thứ quá đa dụng, nó có thể không giải quyết tốt bài toán cụ thể mà bạn đang gặp phải hôm nay. Do đó, một số người có thể lập luận rằng tốt hơn hết là tập trung vào nhu cầu hiện tại, chỉ xây dựng đúng những gì bạn biết mình cần, và chuyên biệt hóa nó cho cách bạn dự định sử dụng hôm nay. Nếu bạn chọn cách tiếp cận chuyên dụng rồi sau này phát hiện thêm các nhu cầu khác, bạn luôn có thể tái cấu trúc (refactor) nó để trở nên đa dụng hơn. Cách tiếp cận chuyên dụng có vẻ nhất quán với phương pháp phát triển phần mềm tăng tiến (incremental approach).

---

## 6.1 Hãy làm cho các lớp hơi đa dụng một chút (Make classes somewhat general-purpose)

Theo kinh nghiệm của tôi, điểm cân bằng lý tưởng (sweet spot) là triển khai các mô-đun mới theo hướng **hơi đa dụng một chút (somewhat general-purpose)**. Cụm từ "hơi đa dụng một chút" có nghĩa là phần chức năng (functionality) của mô-đun nên phản ánh các nhu cầu hiện tại của bạn, nhưng giao diện (interface) của nó thì không nên như vậy. Thay vào đó, giao diện nên đủ tổng quát để hỗ trợ nhiều cách sử dụng khác nhau. Giao diện cần dễ dùng cho các nhu cầu hôm nay mà không bị trói buộc riêng vào chúng. Từ "hơi" (somewhat) rất quan trọng: đừng đi quá đà mà xây dựng một thứ quá đa dụng đến mức khó sử dụng cho chính nhu cầu hiện tại của bạn.

Lợi ích quan trọng nhất (và có lẽ gây bất ngờ nhất) của cách tiếp cận đa dụng là nó tạo ra các giao diện đơn giản hơn và sâu hơn (simpler and deeper) so với cách tiếp cận chuyên dụng. Cách tiếp cận đa dụng cũng có thể tiết kiệm thời gian cho bạn trong tương lai nếu bạn tái sử dụng lớp đó cho các mục đích khác. Tuy nhiên, ngay cả khi mô-đun chỉ được dùng cho mục đích ban đầu của nó, cách tiếp cận đa dụng vẫn tốt hơn nhờ vào sự đơn giản mà nó mang lại.

> 📝 **Giải thích thêm (không có trong sách):** Điểm mấu chốt ở đây là tách biệt giữa *những gì mô-đun làm hôm nay* (chức năng thực tế — không cần viết thừa tính năng cho tương lai xa) và *cách người gọi tương tác với mô-đun* (giao diện — nên dùng các khái niệm nền tảng, phổ quát thay vì tên gọi gắn chặt với một màn hình hay một thao tác UI cụ thể).

---

## 6.2 Ví dụ: Lưu trữ văn bản cho một trình soạn thảo (Example: storing text for an editor)

Hãy xem xét một ví dụ từ lớp học thiết kế phần mềm, trong đó sinh viên được yêu cầu xây dựng các trình soạn thảo văn bản có giao diện đồ họa (GUI text editors) đơn giản. Các trình soạn thảo này phải hiển thị được một tệp và cho phép người dùng trỏ chuột, nhấp và gõ để chỉnh sửa tệp. Chúng phải hỗ trợ nhiều khung nhìn (views) đồng thời của cùng một tệp trên các cửa sổ khác nhau; đồng thời cũng phải hỗ trợ hoàn tác (undo) và làm lại (redo) nhiều cấp cho các thay đổi đối với tệp.

Mỗi dự án của sinh viên đều bao gồm một lớp quản lý phần văn bản nền tảng (underlying text) của tệp. Các lớp văn bản này thường cung cấp các phương thức để tải tệp vào bộ nhớ, đọc và chỉnh sửa văn bản của tệp, và ghi văn bản đã sửa trở lại tệp.

Nhiều nhóm sinh viên đã triển khai các API chuyên dụng (special-purpose APIs) cho lớp văn bản. Họ biết rằng lớp này sẽ được dùng trong một trình soạn thảo tương tác, vì vậy họ nghĩ về các tính năng mà trình soạn thảo phải cung cấp và thiết kế API của lớp văn bản khớp với từng tính năng cụ thể đó. Ví dụ, nếu người dùng trình soạn thảo nhấn phím `backspace`, trình soạn thảo xóa ký tự ngay bên trái con trỏ; nếu người dùng nhấn phím `delete`, trình soạn thảo xóa ký tự ngay bên phải con trỏ. Biết vậy, một số nhóm đã tạo ra mỗi phương thức trong lớp văn bản để hỗ trợ từng tính năng cụ thể này:

```java
void backspace(Cursor cursor);
void delete(Cursor cursor);
```

Mỗi phương thức này nhận vị trí con trỏ làm đối số; một kiểu dữ liệu đặc biệt `Cursor` đại diện cho vị trí này. Trình soạn thảo cũng phải hỗ trợ vùng chọn (selection) để có thể sao chép hoặc xóa. Các sinh viên xử lý việc này bằng cách định nghĩa một lớp `Selection` và truyền một đối tượng của lớp này vào lớp văn bản khi xóa:

```java
void deleteSelection(Selection selection);
```

Các sinh viên có lẽ nghĩ rằng sẽ dễ triển khai giao diện người dùng (user interface) hơn nếu các phương thức của lớp văn bản tương ứng trực tiếp với các tính năng mà người dùng nhìn thấy. Tuy nhiên, trong thực tế, sự chuyên biệt hóa này mang lại rất ít lợi ích cho mã nguồn giao diện người dùng, đồng thời tạo ra tải nhận thức (cognitive load) cao cho các lập trình viên làm việc trên giao diện người dùng hoặc trên lớp văn bản. Lớp văn bản rốt cuộc có một số lượng lớn các phương thức nông (shallow methods), mỗi phương thức chỉ phù hợp cho một thao tác giao diện người dùng duy nhất. Nhiều phương thức, chẳng hạn như `delete`, chỉ được gọi ở đúng một nơi. Kết quả là, một lập trình viên làm việc trên giao diện người dùng phải học rất nhiều phương thức của lớp văn bản.

Cách tiếp cận này tạo ra sự rò rỉ thông tin (information leakage) giữa giao diện người dùng và lớp văn bản. Các trừu tượng hóa liên quan đến giao diện người dùng, chẳng hạn như vùng chọn (`selection`) hay phím `backspace`, bị phản ánh vào trong lớp văn bản; điều này làm tăng tải nhận thức cho các lập trình viên làm việc trên lớp văn bản. Mỗi thao tác giao diện người dùng mới lại đòi hỏi phải định nghĩa thêm một phương thức mới trong lớp văn bản, nên một lập trình viên làm việc trên giao diện người dùng rất dễ phải sửa cả lớp văn bản nữa. Một trong những mục tiêu trong thiết kế lớp là cho phép mỗi lớp được phát triển độc lập, nhưng cách tiếp cận chuyên biệt hóa đã trói buộc lớp giao diện người dùng và lớp văn bản lại với nhau.

---

## 6.3 Một API đa dụng hơn (A more general-purpose API)

Một cách tiếp cận tốt hơn là làm cho lớp văn bản trở nên tổng quát hơn (generic). API của nó chỉ nên được định nghĩa theo các đặc tính cơ bản của văn bản, mà không phản ánh các thao tác cấp cao hơn sẽ được triển khai bằng nó. Ví dụ, chỉ cần hai phương thức để chỉnh sửa văn bản:

```java
void insert(Position position, String newText);
void delete(Position start, Position end);
```

Phương thức đầu tiên chèn một chuỗi bất kỳ tại một vị trí bất kỳ trong văn bản, và phương thức thứ hai xóa tất cả các ký tự ở các vị trí lớn hơn hoặc bằng `start` nhưng nhỏ hơn `end`. API này cũng sử dụng một kiểu tổng quát hơn là `Position` thay vì `Cursor`, vốn là thứ phản ánh một giao diện người dùng cụ thể. Lớp văn bản cũng nên cung cấp các tiện ích đa dụng để thao tác với các vị trí trong văn bản, chẳng hạn như phương thức sau:

```java
Position changePosition(Position position, int numChars);
```

Phương thức này trả về một vị trí mới cách một vị trí cho trước một số ký tự nhất định. Nếu đối số `numChars` dương, vị trí mới nằm sau `position` trong tệp; nếu `numChars` âm, vị trí mới nằm trước `position`. Phương thức tự động chuyển sang dòng tiếp theo hoặc dòng trước đó khi cần thiết. Với các phương thức này, phím `delete` có thể được triển khai bằng đoạn mã sau (giả sử biến `cursor` giữ vị trí con trỏ hiện tại):

```java
text.delete(cursor, text.changePosition(cursor, 1));
```

Tương tự, phím `backspace` có thể được triển khai như sau:

```java
text.delete(text.changePosition(cursor, -1), cursor);
```

Với API văn bản đa dụng, mã nguồn để triển khai các chức năng giao diện người dùng như `delete` và `backspace` có dài hơn một chút so với cách tiếp cận ban đầu dùng API văn bản chuyên dụng. Tuy nhiên, đoạn mã mới rõ ràng hơn (more obvious) so với đoạn mã cũ. Một lập trình viên làm việc trong mô-đun giao diện người dùng có lẽ cần quan tâm chính xác những ký tự nào bị xóa bởi phím `backspace`. Với đoạn mã mới, điều này là hiển nhiên. Với đoạn mã cũ, lập trình viên phải vào lớp văn bản và đọc tài liệu và/hoặc mã nguồn của phương thức `backspace` để xác nhận hành vi. Hơn nữa, cách tiếp cận đa dụng có tổng lượng mã nguồn ít hơn so với cách tiếp cận chuyên dụng, vì nó thay thế một số lượng lớn các phương thức chuyên dụng trong lớp văn bản bằng một số lượng nhỏ hơn các phương thức đa dụng.

Một lớp văn bản được triển khai với giao diện đa dụng có tiềm năng được sử dụng cho các mục đích khác ngoài trình soạn thảo tương tác. Lấy một ví dụ: giả sử bạn đang xây dựng một ứng dụng chỉnh sửa một tệp chỉ định bằng cách thay thế tất cả các lần xuất hiện của một chuỗi cụ thể bằng một chuỗi khác. Các phương thức từ lớp văn bản chuyên dụng, chẳng hạn như `backspace` và `delete`, sẽ hầu như không có giá trị gì cho ứng dụng này. Tuy nhiên, lớp văn bản đa dụng đã có sẵn phần lớn chức năng cần thiết cho ứng dụng mới. Tất cả những gì còn thiếu là một phương thức để tìm lần xuất hiện tiếp theo của một chuỗi cho trước, chẳng hạn như:

```java
Position findNext(Position start, String string);
```

Tất nhiên, một trình soạn thảo văn bản tương tác nhiều khả năng cũng sẽ có cơ chế tìm kiếm và thay thế, trong trường hợp đó lớp văn bản hẳn đã bao gồm sẵn phương thức này rồi.

> 💡 **Ví dụ bổ sung (không có trong sách):** Hãy liên hệ với một dịch vụ gửi thông báo (Notification Service) trong hệ thống backend.
> - **Cách tiếp cận chuyên dụng (nông & rò rỉ thông tin):**
>   ```java
>   void sendOrderConfirmationEmail(Order order);
>   void sendPasswordResetEmail(User user, String token);
>   void sendInvoicePaidSms(Invoice invoice);
>   ```
>   Mỗi nghiệp vụ mới (ví dụ: nhắc gia hạn gói cước) lại phải sửa `NotificationService` để thêm hàm mới và truyền nguyên các đối tượng nghiệp vụ (`Order`, `User`, `Invoice`) vào lớp gửi thông báo.
> - **Cách tiếp cận hơi đa dụng (sâu & tách bạch):**
>   ```java
>   void send(NotificationChannel channel, Recipient recipient, TemplateId template, Map<String, Object> variables);
>   ```
>   Lúc này `NotificationService` không cần biết gì về `Order` hay `Invoice`. Nghiệp vụ đặt hàng tự chuẩn bị `template` và `variables`, còn `NotificationService` chỉ lo việc render mẫu, thử lại khi lỗi, ghi log và gửi qua kênh phù hợp. Giao diện gọn hơn, sâu hơn và dùng lại được cho mọi nghiệp vụ mới.

---

## 6.4 Tính tổng quát dẫn đến che giấu thông tin tốt hơn (Generality leads to better information hiding)

Cách tiếp cận đa dụng tạo ra sự tách biệt sạch sẽ hơn giữa lớp văn bản và lớp giao diện người dùng, dẫn đến khả năng che giấu thông tin (information hiding) tốt hơn. Lớp văn bản không cần biết về các chi tiết cụ thể của giao diện người dùng, chẳng hạn như phím `backspace` được xử lý ra sao; những chi tiết này giờ đây được đóng gói (encapsulated) trong lớp giao diện người dùng. Các tính năng giao diện người dùng mới có thể được thêm vào mà không cần tạo thêm các hàm hỗ trợ mới trong lớp văn bản. Giao diện đa dụng cũng làm giảm tải nhận thức: một lập trình viên làm việc trên giao diện người dùng chỉ cần học vài phương thức đơn giản, và chúng có thể được tái sử dụng cho nhiều mục đích khác nhau.

Phương thức `backspace` trong phiên bản ban đầu của lớp văn bản là một **trừu tượng hóa giả (false abstraction)**. Nó tỏ ra như đang che giấu thông tin về việc những ký tự nào bị xóa, nhưng mô-đun giao diện người dùng thực sự cần biết điều này; các lập trình viên giao diện người dùng nhiều khả năng phải đọc mã nguồn của phương thức `backspace` để xác nhận hành vi chính xác của nó. Việc đặt phương thức đó trong lớp văn bản chỉ khiến các lập trình viên giao diện người dùng khó lấy được thông tin họ cần hơn. Một trong những yếu tố quan trọng nhất của thiết kế phần mềm là xác định **ai cần biết điều gì, và khi nào**. Khi các chi tiết là quan trọng, tốt hơn hết là làm cho chúng tường minh và hiển nhiên nhất có thể, như trong bản triển khai đã chỉnh sửa của thao tác `backspace`. Việc che giấu thông tin này đằng sau một giao diện chỉ tạo ra sự mù mờ (obscurity).

---

## 6.5 Những câu hỏi cần tự vấn (Questions to ask yourself)

Nhận ra một thiết kế lớp đa dụng sạch sẽ thì dễ hơn nhiều so với việc tự tạo ra nó. Dưới đây là một số câu hỏi bạn có thể tự hỏi bản thân, giúp bạn tìm được sự cân bằng phù hợp giữa đa dụng và chuyên dụng cho một giao diện.

**Đâu là giao diện đơn giản nhất vẫn đáp ứng được tất cả các nhu cầu hiện tại của tôi?** Nếu bạn giảm số lượng phương thức trong một API mà không làm giảm năng lực tổng thể của nó, thì có lẽ bạn đang tạo ra các phương thức đa dụng hơn. API văn bản chuyên dụng có ít nhất ba phương thức để xóa văn bản: `backspace`, `delete`, và `deleteSelection`. API đa dụng hơn chỉ có một phương thức để xóa văn bản, nhưng phục vụ cả ba mục đích đó. Việc giảm số lượng phương thức chỉ có ý nghĩa miễn là API của từng phương thức riêng lẻ vẫn giữ được sự đơn giản; nếu bạn phải đưa vào hàng tá đối số bổ sung chỉ để giảm số lượng phương thức, thì có thể bạn không thực sự đang đơn giản hóa mọi thứ.

**Phương thức này sẽ được sử dụng trong bao nhiêu tình huống?** Nếu một phương thức được thiết kế cho một cách dùng cụ thể duy nhất, chẳng hạn như phương thức `backspace`, đó là một **cờ đỏ (red flag)** cho thấy nó có thể quá chuyên dụng. Hãy xem liệu bạn có thể thay thế nhiều phương thức chuyên dụng bằng một phương thức đa dụng duy nhất hay không.

**API này có dễ sử dụng cho các nhu cầu hiện tại của tôi không?** Câu hỏi này giúp bạn nhận ra khi nào mình đã đi quá xa trong việc làm cho một API trở nên đơn giản và đa dụng. Nếu bạn phải viết rất nhiều mã nguồn bổ sung để sử dụng một lớp cho mục đích hiện tại của mình, đó là một **cờ đỏ (red flag)** cho thấy giao diện chưa cung cấp đúng chức năng cần thiết. Ví dụ, một cách tiếp cận cho lớp văn bản là thiết kế nó xoay quanh các thao tác trên từng ký tự đơn lẻ: `insert` chèn một ký tự và `delete` xóa một ký tự. API này vừa đơn giản vừa đa dụng. Tuy nhiên, nó sẽ không thực sự dễ dùng cho một trình soạn thảo văn bản: mã nguồn ở tầng cao hơn sẽ chứa đầy các vòng lặp để chèn hoặc xóa các dải ký tự. Cách tiếp cận từng ký tự đơn lẻ cũng sẽ kém hiệu năng đối với các thao tác lớn. Vì vậy, tốt hơn hết là lớp văn bản nên hỗ trợ sẵn các thao tác trên các dải ký tự (ranges of characters).

> 📝 **Giải thích thêm (không có trong sách):** Ba câu hỏi trên tạo thành hai ranh giới kiểm soát hai thái cực sai lầm:
> - Nếu API có nhiều hàm chỉ dùng đúng một chỗ → **Quá chuyên dụng** (nông, rò rỉ thông tin).
> - Nếu API ít hàm nhưng người gọi phải viết vòng lặp hoặc truyền quá nhiều cờ/tham số phức tạp để làm việc thông thường → **Quá tổng quát / quá nguyên thủy** (khó dùng, đẩy gánh nặng lên người gọi).
> Điểm ngọt "hơi đa dụng" nằm ngay giữa hai ranh giới đó.

---

## 6.6 Kết luận (Conclusion)

Các giao diện đa dụng có nhiều ưu điểm hơn so với các giao diện chuyên dụng. Chúng có xu hướng đơn giản hơn, với ít phương thức hơn nhưng sâu hơn. Chúng cũng tạo ra sự phân tách sạch sẽ hơn giữa các lớp, trong khi các giao diện chuyên dụng thường làm rò rỉ thông tin giữa các lớp. Làm cho các mô-đun của bạn hơi đa dụng một chút là một trong những cách tốt nhất để giảm độ phức tạp tổng thể của hệ thống.

---

## Bảng thuật ngữ chương (Chapter Glossary)

| Thuật ngữ (English) | Tiếng Việt | Giải thích ngắn gọn |
|---|---|---|
| General-purpose module | Mô-đun đa dụng | Mô-đun có giao diện tổng quát, hỗ trợ nhiều mục đích sử dụng khác nhau thay vì gắn chặt vào một kịch bản cụ thể. |
| Special-purpose module | Mô-đun chuyên dụng | Mô-đun có giao diện được thiết kế riêng cho từng tính năng hoặc thao tác cụ thể của bối cảnh sử dụng hiện tại. |
| Somewhat general-purpose | Hơi đa dụng một chút | Nguyên tắc thiết kế: chức năng đáp ứng đúng nhu cầu hiện tại, nhưng giao diện đủ tổng quát để dùng linh hoạt và tách biệt lớp. |
| Shallow method | Phương thức nông | Phương thức thực hiện rất ít công việc so với độ phức tạp phải học và duy trì giao diện của nó. |
| Information leakage | Rò rỉ thông tin | Khi các chi tiết hoặc trừu tượng hóa của một mô-đun (như UI) bị lộ và phản ánh vào thiết kế của mô-đun khác (như lớp dữ liệu). |
| False abstraction | Trừu tượng hóa giả | Giao diện tưởng như che giấu chi tiết nhưng thực chất buộc người gọi phải đọc mã nguồn bên trong mới hiểu được hành vi chính xác. |
| Cognitive load | Tải nhận thức | Lượng kiến thức và nỗ lực tinh thần lập trình viên cần bỏ ra để hiểu và làm việc đúng với mã nguồn. |
| Information hiding | Che giấu thông tin | Đóng gói các chi tiết cài đặt bên trong mô-đun để các mô-đun khác không cần biết và không bị phụ thuộc vào chúng. |
