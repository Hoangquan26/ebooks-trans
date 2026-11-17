# CHƯƠNG 15: Viết bình luận trước (Chapter 15: Write The Comments First)

*(Sử dụng bình luận như một phần của quá trình thiết kế — Use Comments As Part Of The Design Process)*

Nhiều lập trình viên trì hoãn việc viết tài liệu cho đến tận cuối quy trình phát triển, sau khi đã hoàn tất việc viết mã và kiểm thử đơn vị (unit testing). Đây là một trong những cách chắc chắn nhất để tạo ra tài liệu kém chất lượng. Thời điểm tốt nhất để viết bình luận (comments) là ngay từ đầu quy trình, song song với lúc bạn viết mã. Việc viết bình luận trước biến tài liệu thành một phần của quá trình thiết kế. Cách làm này không chỉ tạo ra tài liệu tốt hơn mà còn mang lại những thiết kế tốt hơn và khiến việc viết tài liệu trở nên thú vị hơn.

---

## 15.1 Bình luận viết muộn là bình luận tệ (Delayed comments are bad comments)

Hầu như mọi lập trình viên tôi từng gặp đều trì hoãn việc viết bình luận. Khi được hỏi tại sao không viết tài liệu sớm hơn, họ trả lời rằng mã nguồn vẫn đang thay đổi. Họ nói nếu viết tài liệu sớm, họ sẽ phải viết lại khi mã nguồn thay đổi; tốt hơn là nên đợi cho đến khi mã nguồn ổn định. Tuy nhiên, tôi nghi ngờ còn có một lý do khác, đó là họ xem việc viết tài liệu như một công việc nặng nhọc nhàm chán; do đó, họ trì hoãn nó càng lâu càng tốt.

Thật không may, cách tiếp cận này gây ra một số hệ quả tiêu cực. Thứ nhất, trì hoãn viết tài liệu thường đồng nghĩa với việc nó sẽ không bao giờ được viết. Một khi đã bắt đầu trì hoãn, bạn rất dễ trì hoãn thêm một chút nữa; suy cho cùng, vài tuần nữa mã nguồn sẽ còn ổn định hơn nữa mà. Đến lúc mã nguồn chắc chắn đã ổn định, khối lượng mã đã trở nên khổng lồ, khiến nhiệm vụ viết tài liệu trở nên đồ sộ và thậm chí càng kém hấp dẫn hơn. Không bao giờ có một thời điểm thuận tiện để dừng lại vài ngày chỉ để điền tất cả những bình luận còn thiếu, và thật dễ để tự biện minh rằng điều tốt nhất cho dự án là tiếp tục đi sửa lỗi (bug) hoặc viết tính năng mới tiếp theo. Điều này sẽ tạo ra ngày càng nhiều mã nguồn không có tài liệu.

Ngay cả khi bạn có đủ kỷ luật bản thân để quay lại viết bình luận (và đừng tự lừa dối mình: có lẽ bạn không có đâu), thì các bình luận đó cũng sẽ không tốt lắm. Đến thời điểm này trong quy trình, tâm trí bạn đã rời khỏi phần việc đó rồi. Trong đầu bạn, đoạn mã này đã xong; bạn đang háo hức chuyển sang dự án tiếp theo. Bạn biết viết bình luận là việc đúng đắn nên làm, nhưng nó chẳng vui chút nào. Bạn chỉ muốn làm cho xong càng nhanh càng tốt. Vì vậy, bạn lướt nhanh qua đoạn mã, thêm vào vừa đủ bình luận để trông có vẻ tươm tất. Lúc này, đã một khoảng thời gian trôi qua kể từ khi bạn thiết kế đoạn mã, nên ký ức về quá trình thiết kế đang trở nên mờ nhạt. Bạn nhìn vào mã nguồn trong khi viết bình luận, nên bình luận chỉ lặp lại những gì mã nguồn đã viết. Ngay cả khi bạn cố gắng tái dựng lại các ý tưởng thiết kế không hiển nhiên từ mã nguồn, vẫn sẽ có những thứ bạn không nhớ nổi. Do đó, các bình luận sẽ thiếu đi một số điều quan trọng nhất mà chúng lẽ ra phải mô tả.

---

## 15.2 Viết bình luận trước (Write the comments first)

Tôi sử dụng một cách tiếp cận khác để viết bình luận, trong đó tôi viết bình luận ngay từ những bước đầu tiên:

- Đối với một lớp (class) mới, tôi bắt đầu bằng việc viết bình luận giao diện cho lớp (class interface comment).
- Tiếp theo, tôi viết bình luận giao diện (interface comments) và chữ ký hàm (signatures) cho các phương thức công khai (public methods) quan trọng nhất, nhưng tôi để trống phần thân phương thức (method bodies).
- Tôi lặp đi lặp lại một chút để tinh chỉnh các bình luận này cho đến khi cấu trúc cơ bản có cảm giác ổn thỏa.
- Tại thời điểm này, tôi viết khai báo và bình luận cho các biến thực thể (instance variables) quan trọng nhất của lớp.
- Cuối cùng, tôi điền vào phần thân của các phương thức, thêm các bình luận triển khai (implementation comments) khi cần thiết.

Trong khi viết phần thân phương thức, tôi thường phát hiện ra nhu cầu cần thêm các phương thức và biến thực thể bổ sung. Với mỗi phương thức mới, tôi viết bình luận giao diện trước khi viết phần thân của phương thức; với các biến thực thể, tôi điền bình luận ngay tại thời điểm viết khai báo biến. Khi mã nguồn hoàn thành, các bình luận cũng hoàn thành. Không bao giờ có tình trạng tồn đọng (backlog) bình luận chưa viết.

Cách tiếp cận viết bình luận trước mang lại ba lợi ích. Thứ nhất, nó tạo ra các bình luận tốt hơn. Nếu bạn viết bình luận ngay trong lúc đang thiết kế lớp, các vấn đề thiết kế cốt lõi vẫn còn tươi mới trong tâm trí bạn, nên rất dễ ghi chép lại chúng. Việc viết bình luận giao diện cho mỗi phương thức trước phần thân của nó cũng tốt hơn, vì bạn có thể tập trung vào tính trừu tượng (abstraction) và giao diện của phương thức mà không bị phân tâm bởi phần triển khai. Trong quá trình viết mã và kiểm thử, bạn sẽ nhận ra và sửa chữa các vấn đề trong bình luận. Kết quả là, các bình luận ngày càng được cải thiện trong suốt quá trình phát triển.

> 💡 **Ví dụ bổ sung (không có trong sách):** Quy trình viết bình luận trước khi thiết kế một lớp `TokenBucketRateLimiter` (bộ giới hạn tốc độ yêu cầu theo thuật toán xô token).
>
> **Bước 1 & 2:** Viết bình luận giao diện của lớp, chữ ký và bình luận giao diện của các phương thức công khai chính (chưa có thân hàm):
> ```java
> /**
>  * Giới hạn tần suất thực hiện thao tác theo thuật toán Token Bucket.
>  * Mỗi thực thể quản lý một xô (bucket) độc lập với sức chứa tối đa
>  * và tốc độ nạp lại token không đổi theo thời gian thực.
>  * Lớp này an toàn với đa luồng (thread-safe).
>  */
> public class TokenBucketRateLimiter {
>     /**
>      * Tạo bộ giới hạn tốc độ mới với xô ban đầu đầy token.
>      *
>      * @param capacity      Số token tối đa xô có thể chứa (phải > 0).
>      * @param refillPerSec  Số token được nạp thêm mỗi giây (phải > 0).
>      */
>     public TokenBucketRateLimiter(int capacity, double refillPerSec) {
>     }
>
>     /**
>      * Thử lấy số lượng token yêu cầu từ xô nếu đủ; không chặn (non-blocking).
>      *
>      * @param permits Số token cần lấy (phải >= 0).
>      * @return true nếu lấy đủ token và đã trừ khỏi xô; false nếu không đủ
>      *         (trạng thái xô không bị trừ token trong trường hợp này).
>      */
>     public boolean tryAcquire(int permits) {
>         return false;
>     }
> }
> ```
> **Bước 3 & 4:** Tinh chỉnh giao diện (ví dụ: nhận ra cần nêu rõ hành vi khi `permits > capacity` hay tính thread-safety), rồi khai báo các biến thực thể cùng bình luận mô tả bất biến (invariant) của chúng:
> ```java
>     /** Sức chứa tối đa của xô; luôn > 0. */
>     private final int capacity;
>     /** Tốc độ nạp token tính bằng token/nanosecond để tránh sai số khi đổi đơn vị. */
>     private final double refillPerNanos;
>     /** Số token hiện có tại thời điểm lastRefillNanos; luôn nằm trong [0, capacity]. */
>     private double availableTokens;
>     /** Mốc thời gian (theo System.nanoTime) của lần cập nhật xô gần nhất. */
>     private long lastRefillNanos;
> ```
> **Bước 5:** Lúc này mới điền phần thân hàm và bổ sung bình luận triển khai cho những chỗ không hiển nhiên (như lý do dùng `System.nanoTime()` thay vì `System.currentTimeMillis()` để tránh đồng hồ hệ thống nhảy lùi). Khi hàm chạy xong, tài liệu cũng đã hoàn chỉnh và chính xác.

---

## 15.3 Bình luận là một công cụ thiết kế (Comments are a design tool)

Lợi ích thứ hai, và cũng là lợi ích quan trọng nhất, của việc viết bình luận ngay từ đầu là nó cải thiện thiết kế hệ thống. Bình luận cung cấp cách duy nhất để nắm bắt trọn vẹn các trừu tượng hóa (abstractions), và các trừu tượng hóa tốt là nền tảng của một thiết kế hệ thống tốt. Nếu bạn viết các bình luận mô tả những trừu tượng hóa ngay từ đầu, bạn có thể xem xét lại và tinh chỉnh chúng trước khi viết mã triển khai. Để viết được một bình luận tốt, bạn phải xác định được bản chất cốt lõi của một biến hoặc một đoạn mã: những khía cạnh quan trọng nhất của thứ này là gì? Việc làm điều này sớm trong quá trình thiết kế rất quan trọng; nếu không, bạn chỉ đang chắp vá mã nguồn (hacking code) mà thôi.

Bình luận đóng vai trò như một chú chim hoàng yến trong mỏ than (canary in the coal mine) để cảnh báo về độ phức tạp. Nếu một phương thức hoặc biến đòi hỏi một bình luận dài dòng, đó là một cờ đỏ (red flag) cho thấy bạn chưa có một trừu tượng hóa tốt. Hãy nhớ lại từ Chương 4 rằng các lớp nên sâu (deep): những lớp tốt nhất có giao diện rất đơn giản nhưng lại triển khai những chức năng mạnh mẽ. Cách tốt nhất để đánh giá độ phức tạp của một giao diện là thông qua các bình luận mô tả nó. Nếu bình luận giao diện cho một phương thức cung cấp đầy đủ thông tin cần thiết để sử dụng phương thức đó mà vẫn ngắn gọn và đơn giản, điều đó chứng tỏ phương thức có một giao diện đơn giản. Ngược lại, nếu không có cách nào để mô tả trọn vẹn một phương thức mà không cần đến một bình luận dài dòng và phức tạp, thì phương thức đó có một giao diện phức tạp. Bạn có thể so sánh bình luận giao diện của một phương thức với phần triển khai của nó để cảm nhận xem phương thức đó sâu đến mức nào: nếu bình luận giao diện phải mô tả tất cả các tính năng chính của phần triển khai, thì phương thức đó bị nông (shallow). Ý tưởng tương tự cũng áp dụng cho biến: nếu phải tốn một bình luận dài để mô tả đầy đủ một biến, đó là một cờ đỏ gợi ý rằng bạn có thể chưa chọn đúng cách phân rã biến (variable decomposition). Nhìn chung, hành động viết bình luận cho phép bạn đánh giá các quyết định thiết kế của mình từ sớm, nhờ đó bạn có thể phát hiện và khắc phục các vấn đề.

> 🚩 **Cờ Đỏ (Red Flag — Khó mô tả / Hard to Describe):** Bình luận mô tả một phương thức hoặc một biến nên đơn giản nhưng vẫn đầy đủ. Nếu bạn thấy khó khăn khi viết một bình luận như vậy, đó là dấu hiệu cho thấy có thể đang có vấn đề trong thiết kế của thứ bạn đang mô tả.

Tất nhiên, bình luận chỉ là một thước đo tốt cho độ phức tạp nếu chúng đầy đủ và rõ ràng. Nếu bạn viết một bình luận giao diện cho phương thức nhưng lại không cung cấp đủ toàn bộ thông tin cần thiết để gọi phương thức đó, hoặc viết một bình luận tối nghĩa đến mức khó hiểu, thì bình luận đó không mang lại một thước đo tốt về độ sâu của phương thức.

> 💡 **Ví dụ bổ sung (không có trong sách):** So sánh hai thiết kế thông qua bình luận giao diện của phương thức:
>
> - **Thiết kế nông / khó mô tả (Cờ đỏ):**
>   ```python
>   def sync_user_data(
>       user_id: int,
>       fetch_remote: bool,
>       overwrite_local: bool,
>       retry_count: int,
>       out_errors: list[str],
>       skip_if_locked: bool,
>   ) -> int:
>       """
>       Đồng bộ dữ liệu người dùng. Nếu fetch_remote là True thì tải từ API;
>       nếu overwrite_local là True thì ghi đè bản cục bộ, ngược lại chỉ
>       cập nhật các trường rỗng. Thử lại tối đa retry_count lần khi gặp lỗi
>       mạng, ghi thông báo lỗi vào out_errors. Nếu skip_if_locked là True
>       và bản ghi đang bị khóa thì bỏ qua và trả về -1; trả về 0 nếu thành
>       công, 1 nếu lỗi một phần, 2 nếu thất bại hoàn toàn...
>       """
>   ```
>   Chỉ cần cố gắng viết bình luận đầy đủ cho hàm này là ta thấy ngay giao diện quá phức tạp, phơi bày hàng loạt chi tiết triển khai và mã lỗi rắc rối.
>
> - **Thiết kế sâu / dễ mô tả:**
>   ```python
>   def sync_user(user_id: int) -> SyncResult:
>       """
>       Đồng bộ hồ sơ người dùng với máy chủ từ xa và trả về kết quả đồng bộ.
>       Tự động xử lý thử lại khi lỗi tạm thời và bảo toàn dữ liệu cục bộ
>       chưa được đẩy lên.
>       """
>   ```
>   Bình luận ngắn gọn nhưng đầy đủ, cho thấy lớp/hàm đã giấu được độ phức tạp vào bên trong.

---

## 15.4 Bình luận viết sớm là bình luận thú vị (Early comments are fun comments)

Lợi ích thứ ba và cũng là lợi ích cuối cùng của việc viết bình luận sớm là nó khiến việc viết bình luận trở nên thú vị hơn. Với tôi, một trong những phần thú vị nhất của lập trình là giai đoạn thiết kế ban đầu cho một lớp mới, nơi tôi đang phác thảo các trừu tượng hóa và cấu trúc cho lớp đó. Hầu hết các bình luận của tôi được viết trong giai đoạn này, và bình luận chính là cách tôi ghi lại cũng như kiểm chứng chất lượng các quyết định thiết kế của mình. Tôi luôn tìm kiếm một thiết kế có thể được diễn đạt đầy đủ và rõ ràng bằng ít từ ngữ nhất. Bình luận càng đơn giản, tôi càng cảm thấy hài lòng về thiết kế của mình, nên việc tìm ra được những bình luận đơn giản là một nguồn tự hào. Nếu bạn đang lập trình chiến lược (strategic programming), nơi mục tiêu chính của bạn là một thiết kế tuyệt vời chứ không chỉ là viết ra đoạn mã chạy được, thì việc viết bình luận phải là một việc thú vị, bởi vì đó chính là cách bạn nhận diện ra những thiết kế tốt nhất.

---

## 15.5 Bình luận viết sớm có tốn kém không? (Are early comments expensive?)

Bây giờ hãy cùng xem xét lại lập luận ủng hộ việc trì hoãn bình luận, cho rằng làm vậy sẽ tránh được chi phí phải sửa lại bình luận khi mã nguồn tiến hóa. Một phép tính nhẩm đơn giản (back-of-the-envelope calculation) sẽ cho thấy cách làm đó chẳng tiết kiệm được bao nhiêu. Đầu tiên, hãy ước tính tổng tỷ lệ thời gian phát triển mà bạn dành để gõ mã nguồn và bình luận cộng lại, bao gồm cả thời gian chỉnh sửa mã và bình luận; khó có khả năng con số này vượt quá khoảng 10% tổng thời gian phát triển. Ngay cả khi một nửa tổng số dòng mã của bạn là bình luận, việc viết bình luận có lẽ cũng không chiếm quá khoảng 5% tổng thời gian phát triển. Việc trì hoãn viết bình luận đến cuối cùng sẽ chỉ tiết kiệm được một phần nhỏ của con số 5% đó, tức là chẳng đáng kể là bao.

Việc viết bình luận trước đồng nghĩa với việc các trừu tượng hóa sẽ ổn định hơn trước khi bạn bắt đầu viết mã. Điều này có thể sẽ giúp tiết kiệm thời gian trong quá trình viết mã. Ngược lại, nếu bạn viết mã trước, các trừu tượng hóa nhiều khả năng sẽ thay đổi dần trong lúc bạn viết mã, điều này sẽ đòi hỏi phải sửa đổi mã nguồn nhiều hơn so với cách tiếp cận viết bình luận trước. Khi bạn cân nhắc tất cả các yếu tố này, hoàn toàn có khả năng việc viết bình luận trước thậm chí còn nhanh hơn xét trên tổng thể.

> 📝 **Giải thích thêm (không có trong sách):** Trong thực tế phát triển phần mềm, phần lớn thời gian của lập trình viên không nằm ở việc gõ phím, mà nằm ở việc đọc hiểu yêu cầu, thiết kế giải pháp, gỡ lỗi (debugging), kiểm thử và rà soát mã (code review). Nếu 90% thời gian là suy nghĩ và gỡ lỗi, 10% là gõ phím (5% gõ code + 5% gõ comment), thì việc "tiết kiệm" bằng cách né tránh viết lại vài dòng comment khi thiết kế thay đổi chỉ tiết kiệm được khoảng 1–2% tổng thời gian — trong khi cái giá phải trả nếu thiết kế sai từ đầu (do không viết comment để kiểm chứng trừu tượng hóa) có thể tốn thêm 20–30% thời gian đập đi viết lại code.

---

## 15.6 Kết luận (Conclusion)

Nếu bạn chưa từng thử viết bình luận trước, hãy thử một lần xem sao. Hãy kiên trì thực hiện đủ lâu để làm quen với nó. Sau đó, hãy suy ngẫm xem cách làm này ảnh hưởng như thế nào đến chất lượng bình luận của bạn, chất lượng thiết kế của bạn, và niềm vui chung của bạn trong công việc phát triển phần mềm. Sau khi thử cách này một thời gian, hãy cho tôi biết trải nghiệm của bạn có giống với trải nghiệm của tôi không, và tại sao lại có hoặc không.

---

## Bảng thuật ngữ chương (Chapter Glossary)

| Thuật ngữ (English) | Tiếng Việt | Giải thích ngắn gọn |
| --- | --- | --- |
| Write the comments first | Viết bình luận trước | Phương pháp viết bình luận giao diện của lớp, phương thức và biến ngay trong giai đoạn thiết kế trước khi viết phần thân mã nguồn. |
| Class interface comment | Bình luận giao diện lớp | Bình luận ở đầu lớp mô tả khái niệm trừu tượng, trách nhiệm và cách sử dụng lớp đó. |
| Interface comment | Bình luận giao diện | Bình luận mô tả những gì người gọi cần biết để sử dụng một phương thức hoặc lớp mà không cần đọc phần triển khai. |
| Implementation comment | Bình luận triển khai | Bình luận bên trong thân phương thức giải thích *cách thức* và *lý do* mã nguồn hoạt động như vậy. |
| Instance variable | Biến thực thể | Biến thuộc về một đối tượng (instance) của lớp, lưu giữ trạng thái của đối tượng đó. |
| Canary in the coal mine | Chim hoàng yến trong mỏ than | Hình ảnh ẩn dụ chỉ dấu hiệu cảnh báo sớm về nguy cơ tiềm ẩn (ở đây là bình luận dài dòng cảnh báo độ phức tạp thiết kế). |
| Hard to Describe (Red Flag) | Cờ Đỏ: Khó mô tả | Dấu hiệu cảnh báo khi một phương thức hoặc biến không thể được mô tả một cách vừa ngắn gọn vừa đầy đủ. |
| Variable decomposition | Phân rã biến | Cách chia nhỏ và tổ chức trạng thái dữ liệu thành các biến riêng biệt trong một lớp hoặc phương thức. |
| Strategic programming | Lập trình chiến lược | Phong cách lập trình ưu tiên đầu tư thời gian cho thiết kế tốt và cấu trúc sạch dài hạn thay vì chỉ làm cho mã chạy tạm được. |
| Back-of-the-envelope calculation | Phép tính nhẩm ước lượng | Phép tính nhanh, đơn giản dựa trên các con số ước lượng hợp lý để đánh giá độ lớn của chi phí hoặc hiệu quả. |
