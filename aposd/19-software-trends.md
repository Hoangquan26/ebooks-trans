# CHƯƠNG 19: Các xu hướng trong phát triển phần mềm (Chapter 19: Software Trends)

Như một cách để minh họa các nguyên lý đã thảo luận trong cuốn sách này, chương này xem xét một số xu hướng và mẫu hình (pattern) đã trở nên phổ biến trong phát triển phần mềm trong vài thập kỷ qua. Đối với mỗi xu hướng, tôi sẽ mô tả cách xu hướng đó liên hệ với các nguyên lý trong cuốn sách này và sử dụng các nguyên lý đó để đánh giá xem liệu xu hướng đó có mang lại đòn bẩy chống lại sự phức tạp của phần mềm (software complexity) hay không.

## 19.1 Lập trình hướng đối tượng và kế thừa (Object-oriented programming and inheritance)

Lập trình hướng đối tượng (Object-oriented programming – OOP) là một trong những ý tưởng mới quan trọng nhất trong phát triển phần mềm trong suốt 30–40 năm qua. Nó đưa ra các khái niệm như lớp (class), kế thừa (inheritance), phương thức riêng tư (private method), và biến thực thể (instance variable). Nếu được sử dụng cẩn thận, những cơ chế này có thể giúp tạo ra các thiết kế phần mềm tốt hơn. Ví dụ, các phương thức và biến riêng tư có thể được sử dụng để đảm bảo việc ẩn giấu thông tin (information hiding): không đoạn mã nào bên ngoài một lớp có thể gọi các phương thức riêng tư hoặc truy cập các biến riêng tư, vì vậy không thể có bất kỳ sự phụ thuộc bên ngoài nào vào chúng.

Một trong những yếu tố cốt lõi của lập trình hướng đối tượng là tính kế thừa (inheritance). Kế thừa có hai dạng, mang lại những hệ quả khác nhau đối với sự phức tạp của phần mềm. Dạng kế thừa thứ nhất là *kế thừa giao diện* (interface inheritance), trong đó một lớp cha (parent class) định nghĩa chữ ký (signature) cho một hoặc nhiều phương thức, nhưng không cài đặt các phương thức đó. Mỗi lớp con (subclass) phải tự cài đặt các chữ ký đó, nhưng các lớp con khác nhau có thể cài đặt cùng một phương thức theo những cách khác nhau. Ví dụ, giao diện có thể định nghĩa các phương thức để thực hiện nhập/xuất (I/O); một lớp con có thể cài đặt các thao tác I/O cho tệp tin trên đĩa (disk file), và một lớp con khác có thể cài đặt chính những thao tác đó cho socket mạng (network socket).

Kế thừa giao diện mang lại đòn bẩy chống lại sự phức tạp bằng cách tái sử dụng cùng một giao diện cho nhiều mục đích. Nó cho phép kiến thức tích lũy được khi giải quyết một bài toán (chẳng hạn như cách sử dụng giao diện I/O để đọc và ghi tệp trên đĩa) được dùng để giải quyết các bài toán khác (chẳng hạn như giao tiếp qua socket mạng). Một cách nghĩ khác về điều này là xét theo chiều sâu (depth): một giao diện càng có nhiều cách cài đặt khác nhau, giao diện đó càng trở nên sâu sắc hơn. Để một giao diện có thể có nhiều cách cài đặt, nó phải nắm bắt được những đặc trưng thiết yếu của tất cả các cách cài đặt bên dưới đồng thời tránh xa những chi tiết khác biệt giữa các cách cài đặt đó; ý niệm này chính là cốt lõi của sự trừu tượng hóa (abstraction).

Dạng kế thừa thứ hai là *kế thừa cài đặt* (implementation inheritance). Ở dạng này, một lớp cha không chỉ định nghĩa chữ ký cho một hoặc nhiều phương thức, mà còn cung cấp các bản cài đặt mặc định. Các lớp con có thể chọn kế thừa bản cài đặt của lớp cha cho một phương thức hoặc ghi đè (override) nó bằng cách định nghĩa một phương thức mới có cùng chữ ký. Nếu không có kế thừa cài đặt, cùng một bản cài đặt phương thức có thể phải bị sao chép ở nhiều lớp con, điều này sẽ tạo ra sự phụ thuộc giữa các lớp con đó (mọi chỉnh sửa sẽ phải được lặp lại ở tất cả các bản sao của phương thức). Do đó, kế thừa cài đặt làm giảm lượng mã nguồn cần phải sửa đổi khi hệ thống tiến hóa; nói cách khác, nó làm giảm vấn đề khuếch đại thay đổi (change amplification) đã được mô tả trong Chương 2.

Tuy nhiên, kế thừa cài đặt tạo ra sự phụ thuộc giữa lớp cha và từng lớp con của nó. Các biến thực thể của lớp cha thường được truy cập bởi cả lớp cha lẫn các lớp con; điều này dẫn đến rò rỉ thông tin (information leakage) giữa các lớp trong hệ thống phân cấp kế thừa (inheritance hierarchy) và khiến việc sửa đổi một lớp trong hệ thống phân cấp trở nên khó khăn nếu không xem xét các lớp khác. Ví dụ, một lập trình viên khi thực hiện thay đổi ở lớp cha có thể cần phải kiểm tra tất cả các lớp con để đảm bảo các thay đổi đó không làm hỏng bất cứ thứ gì. Tương tự, nếu một lớp con ghi đè một phương thức ở lớp cha, lập trình viên của lớp con có thể cần phải xem xét bản cài đặt bên trong lớp cha. Trong trường hợp xấu nhất, lập trình viên sẽ cần nắm rõ toàn bộ hệ thống phân cấp lớp bên dưới lớp cha để có thể thực hiện thay đổi đối với bất kỳ lớp nào. Các hệ thống phân cấp lớp sử dụng kế thừa cài đặt một cách tràn lan thường có độ phức tạp rất cao.

Vì vậy, kế thừa cài đặt cần được sử dụng một cách thận trọng. Trước khi sử dụng kế thừa cài đặt, hãy cân nhắc xem một cách tiếp cận dựa trên *kết hợp* (composition) có thể mang lại những lợi ích tương tự hay không. Chẳng hạn, có thể sử dụng các lớp trợ giúp (helper class) nhỏ để cài đặt phần chức năng dùng chung. Thay vì kế thừa các hàm từ một lớp cha, các lớp ban đầu có thể tự xây dựng dựa trên các tính năng của những lớp trợ giúp đó.

Nếu không có giải pháp thay thế khả thi nào cho kế thừa cài đặt, hãy cố gắng tách biệt trạng thái do lớp cha quản lý khỏi trạng thái do các lớp con quản lý. Một cách để làm điều này là để một số biến thực thể nhất định được quản lý hoàn toàn bởi các phương thức trong lớp cha, và các lớp con chỉ sử dụng chúng ở chế độ chỉ đọc (read-only) hoặc thông qua các phương thức khác trong lớp cha. Cách làm này áp dụng khái niệm ẩn giấu thông tin ngay bên trong hệ thống phân cấp lớp để giảm thiểu sự phụ thuộc.

Mặc dù các cơ chế mà lập trình hướng đối tượng cung cấp có thể hỗ trợ việc triển khai các thiết kế sạch sẽ, nhưng bản thân chúng không tự động đảm bảo một thiết kế tốt. Ví dụ, nếu các lớp quá nông (shallow), hoặc có giao diện phức tạp, hoặc cho phép truy cập từ bên ngoài vào trạng thái nội bộ của chúng, thì chúng vẫn sẽ dẫn đến độ phức tạp cao.

> 💡 **Ví dụ bổ sung (không có trong sách):** Minh họa sự khác biệt giữa *kế thừa cài đặt* (dễ gây rò rỉ thông tin) và *kết hợp* (composition) cùng *kế thừa giao diện*:
>
> ```java
> // Cách dễ gây phức tạp: Kế thừa cài đặt
> class BaseCache {
>     protected Map<String, Object> store = new HashMap<>(); // Lớp con truy cập trực tiếp -> rò rỉ thông tin
>     protected int hitCount = 0;
>
>     public Object get(String key) {
>         if (store.containsKey(key)) {
>             hitCount++;
>             return store.get(key);
>         }
>         return null;
>     }
> }
>
> // Cách sạch sẽ hơn: Kết hợp (Composition) + Kế thừa giao diện (Interface Inheritance)
> interface Cache {
>     Object get(String key);
>     void put(String key, Object value);
> }
>
> class MetricsTracker { // Lớp trợ giúp nhỏ quản lý thống kê
>     private int hitCount = 0;
>     public void recordHit() { hitCount++; }
>     public int getHitCount() { return hitCount; }
> }
>
> class InMemoryCache implements Cache {
>     private final Map<String, Object> store = new HashMap<>();
>     private final MetricsTracker metrics = new MetricsTracker();
>
>     @Override
>     public Object get(String key) {
>         Object val = store.get(key);
>         if (val != null) metrics.recordHit();
>         return val;
>     }
>
>     @Override
>     public void put(String key, Object value) {
>         store.put(key, value);
>     }
> }
> ```
> Khi dùng kết hợp, `InMemoryCache` không phụ thuộc vào cấu trúc nội bộ của `MetricsTracker`, và giao diện `Cache` có thể có nhiều cài đặt khác nhau (ví dụ `RedisCache`, `DiskCache`) mà người dùng không cần biết chi tiết bên dưới.

## 19.2 Phát triển phần mềm linh hoạt – Agile (Agile development)

Phát triển phần mềm linh hoạt (Agile development) là một phương pháp tiếp cận phát triển phần mềm xuất hiện vào cuối thập niên 1990 từ tập hợp các ý tưởng về cách làm cho việc phát triển phần mềm trở nên gọn nhẹ hơn, linh hoạt hơn và mang tính gia tăng (incremental); nó được định nghĩa chính thức trong một cuộc họp của các chuyên gia thực hành vào năm 2001. Phát triển Agile chủ yếu bàn về *quy trình* phát triển phần mềm (tổ chức đội ngũ, quản lý tiến độ, vai trò của kiểm thử đơn vị, tương tác với khách hàng, v.v.) hơn là về *thiết kế* phần mềm. Tuy nhiên, nó vẫn có liên hệ với một số nguyên lý thiết kế trong cuốn sách này.

Một trong những yếu tố quan trọng nhất của phát triển Agile là quan niệm rằng việc phát triển nên mang tính gia tăng (incremental) và lặp lại (iterative). Trong cách tiếp cận Agile, một hệ thống phần mềm được phát triển qua một chuỗi các vòng lặp (iteration), mỗi vòng lặp bổ sung và đánh giá một vài tính năng mới; mỗi vòng lặp bao gồm thiết kế, kiểm thử và phản hồi từ khách hàng. Nhìn chung, điều này tương đồng với cách tiếp cận gia tăng được ủng hộ trong cuốn sách này. Như đã đề cập ở Chương 1, không thể hình dung một hệ thống phức tạp đủ rõ ngay từ đầu dự án để xác định ra thiết kế tốt nhất. Cách tốt nhất để đạt được một thiết kế tốt là phát triển hệ thống theo từng bước tăng dần, trong đó mỗi bước bổ sung thêm một vài trừu tượng (abstraction) mới và tái cấu trúc (refactor) các trừu tượng hiện có dựa trên kinh nghiệm thực tế. Điều này tương đồng với cách tiếp cận phát triển Agile.

Một trong những rủi ro của phát triển Agile là nó có thể dẫn đến *lập trình chiến thuật* (tactical programming). Phát triển Agile có xu hướng hướng sự tập trung của lập trình viên vào các *tính năng* (feature), chứ không phải các *trừu tượng* (abstraction), và nó khuyến khích lập trình viên trì hoãn các quyết định thiết kế để tạo ra phần mềm chạy được càng sớm càng tốt. Ví dụ, một số người thực hành Agile lập luận rằng bạn không nên cài đặt các cơ chế tổng quát (general-purpose) ngay lập tức; hãy cài đặt một cơ chế chuyên biệt tối giản để bắt đầu, rồi tái cấu trúc thành thứ gì đó tổng quát hơn sau này, một khi bạn biết chắc rằng nó thực sự cần thiết. Mặc dù những lập luận này có phần hợp lý ở một mức độ nhất định, nhưng chúng lại đi ngược lại cách tiếp cận mang tính *đầu tư* (investment approach), và khuyến khích một phong cách lập trình thiên về chiến thuật hơn. Điều này có thể dẫn đến sự tích tụ nhanh chóng của độ phức tạp.

Phát triển theo từng bước tăng dần nhìn chung là một ý tưởng hay, nhưng các đơn vị tăng dần của quá trình phát triển phải là các *trừu tượng*, chứ không phải các *tính năng*. Việc gác lại mọi suy nghĩ về một trừu tượng cụ thể cho đến khi có một tính năng thực sự cần đến nó là hoàn toàn ổn. Nhưng một khi bạn đã cần đến trừu tượng đó, hãy đầu tư thời gian để thiết kế nó một cách sạch sẽ; hãy làm theo lời khuyên ở Chương 6 và khiến nó mang tính tổng quát ở mức độ vừa phải (somewhat general-purpose).

> 📝 **Giải thích thêm (không có trong sách):** Điểm mấu chốt Ousterhout muốn nhấn mạnh là sự khác biệt giữa *trì hoãn đúng lúc* và *thiết kế cẩu thả*. Trong Agile, câu thần chú "YAGNI" (*You Aren't Gonna Need It* – Bạn sẽ không cần nó đâu) thường bị hiểu nhầm thành "cứ viết đại cho xong tính năng trước, thiết kế tính sau". Ousterhout đồng ý rằng không nên xây sẵn những trừu tượng chưa ai dùng tới, nhưng **ngay khi một trừu tượng đã xuất hiện trong phạm vi công việc hiện tại**, bạn phải dành thời gian thiết kế nó sạch sẽ và đủ tổng quát thay vì vá víu một giải pháp tạm bợ.

## 19.3 Kiểm thử đơn vị (Unit tests)

Trước đây, các lập trình viên hiếm khi tự viết kiểm thử (test). Nếu có viết kiểm thử, chúng thường được thực hiện bởi một đội ngũ Đảm bảo Chất lượng (QA) riêng biệt. Tuy nhiên, một trong những nguyên lý cốt lõi của phát triển Agile là việc kiểm thử phải được tích hợp chặt chẽ với quá trình phát triển, và lập trình viên phải tự viết kiểm thử cho mã nguồn của chính mình. Thực hành này hiện đã trở nên phổ biến rộng rãi. Các bài kiểm thử thường được chia thành hai loại: *kiểm thử đơn vị* (unit test) và *kiểm thử hệ thống* (system test). Kiểm thử đơn vị là loại được lập trình viên viết nhiều nhất. Chúng nhỏ gọn và tập trung: mỗi bài kiểm thử thường xác nhận một đoạn mã nhỏ bên trong một phương thức đơn lẻ. Kiểm thử đơn vị có thể chạy độc lập mà không cần thiết lập môi trường sản xuất (production environment) cho hệ thống. Kiểm thử đơn vị thường được chạy kết hợp với công cụ đo độ bao phủ kiểm thử (test coverage tool) để đảm bảo mọi dòng mã trong ứng dụng đều được kiểm thử. Bất cứ khi nào lập trình viên viết mã mới hoặc sửa đổi mã hiện có, họ có trách nhiệm cập nhật các bài kiểm thử đơn vị để duy trì độ bao phủ phù hợp.

Loại kiểm thử thứ hai bao gồm kiểm thử hệ thống (đôi khi được gọi là *kiểm thử tích hợp* – integration test), nhằm đảm bảo các phần khác nhau của một ứng dụng đều phối hợp hoạt động đúng đắn với nhau. Chúng thường liên quan đến việc chạy toàn bộ ứng dụng trong một môi trường giống môi trường sản xuất. Kiểm thử hệ thống thường có khả năng được viết bởi một đội ngũ QA hoặc đội kiểm thử riêng biệt.

Các bài kiểm thử, đặc biệt là kiểm thử đơn vị, đóng một vai trò quan trọng trong thiết kế phần mềm vì chúng tạo điều kiện thuận lợi cho việc *tái cấu trúc* (refactoring). Nếu không có một bộ kiểm thử (test suite), việc thực hiện những thay đổi cấu trúc lớn đối với một hệ thống là rất nguy hiểm. Không có cách nào dễ dàng để tìm ra lỗi, vì vậy nhiều khả năng lỗi sẽ không bị phát hiện cho đến khi mã mới được triển khai, lúc đó chi phí tìm và sửa lỗi sẽ đắt đỏ hơn rất nhiều. Hệ quả là các lập trình viên thường né tránh việc tái cấu trúc trong những hệ thống không có bộ kiểm thử tốt; họ cố gắng giảm thiểu số lượng thay đổi mã cho mỗi tính năng mới hoặc mỗi lần sửa lỗi, điều này đồng nghĩa với việc sự phức tạp ngày càng tích tụ và các sai lầm thiết kế không bao giờ được khắc phục.

Với một bộ kiểm thử tốt, lập trình viên có thể tự tin hơn khi tái cấu trúc bởi vì bộ kiểm thử sẽ phát hiện hầu hết các lỗi vô tình được tạo ra. Điều này khuyến khích lập trình viên thực hiện các cải tiến về mặt cấu trúc cho hệ thống, dẫn đến một thiết kế tốt hơn. Kiểm thử đơn vị đặc biệt có giá trị: chúng mang lại mức độ bao phủ mã cao hơn so với kiểm thử hệ thống, nên chúng có nhiều khả năng phát hiện ra mọi lỗi tiềm ẩn hơn.

Ví dụ, trong quá trình phát triển ngôn ngữ kịch bản Tcl, chúng tôi đã quyết định cải thiện hiệu năng bằng cách thay thế trình thông dịch (interpreter) của Tcl bằng một trình biên dịch mã byte (byte-code compiler). Đây là một thay đổi khổng lồ ảnh hưởng đến hầu hết mọi phần trong nhân lõi của Tcl. May mắn thay, Tcl có một bộ kiểm thử đơn vị xuất sắc, và chúng tôi đã chạy bộ kiểm thử đó trên bộ máy mã byte mới. Các bài kiểm thử hiện có hiệu quả đến mức trong việc phát hiện lỗi ở bộ máy mới, chỉ có duy nhất một lỗi phát sinh sau bản phát hành alpha của trình biên dịch mã byte.

## 19.4 Phát triển hướng kiểm thử – TDD (Test-driven development)

Phát triển hướng kiểm thử (Test-driven development – TDD) là một phương pháp phát triển phần mềm trong đó lập trình viên viết các bài kiểm thử đơn vị *trước* khi viết mã nguồn. Khi tạo một lớp mới, lập trình viên trước tiên viết các bài kiểm thử đơn vị cho lớp đó dựa trên hành vi kỳ vọng của nó. Không có bài kiểm thử nào vượt qua (pass), vì chưa có mã nguồn nào cho lớp đó cả. Sau đó, lập trình viên xử lý từng bài kiểm thử một, viết vừa đủ mã nguồn để bài kiểm thử đó vượt qua. Khi tất cả các bài kiểm thử đều vượt qua, lớp đó được coi là hoàn thành.

Mặc dù tôi là một người ủng hộ mạnh mẽ kiểm thử đơn vị, nhưng tôi không phải là người hâm mộ phát triển hướng kiểm thử (TDD). Vấn đề với phát triển hướng kiểm thử là nó hướng sự chú ý vào việc làm cho các *tính năng cụ thể* hoạt động, thay vì tìm kiếm *thiết kế tốt nhất*. Đây thuần túy là lập trình chiến thuật với đầy đủ các nhược điểm của nó. Phát triển hướng kiểm thử mang tính gia tăng quá mức (too incremental): tại bất kỳ thời điểm nào, lập trình viên rất dễ bị cám dỗ chỉ viết chắp vá thêm tính năng tiếp theo để làm cho bài kiểm thử tiếp theo vượt qua. Không có một thời điểm rõ ràng nào dành cho việc thiết kế, vì vậy rất dễ kết thúc với một mớ hỗn độn.

Như đã đề cập ở Mục 19.2, đơn vị của quá trình phát triển phải là các *trừu tượng*, không phải các *tính năng*. Một khi bạn phát hiện ra nhu cầu cho một trừu tượng, đừng tạo ra trừu tượng đó từng mảnh ghép một theo thời gian; hãy thiết kế toàn bộ nó cùng một lúc (hoặc ít nhất đủ để cung cấp một tập hợp các hàm cốt lõi tương đối toàn diện). Cách làm này có nhiều khả năng tạo ra một thiết kế sạch sẽ nơi các mảnh ghép ăn khớp tốt với nhau.

Có một trường hợp mà việc viết kiểm thử trước thực sự hợp lý: đó là khi *sửa lỗi* (fixing bugs). Trước khi sửa một lỗi, hãy viết một bài kiểm thử đơn vị bị thất bại (fail) chính vì lỗi đó. Sau đó mới sửa lỗi và đảm bảo rằng bài kiểm thử đơn vị đó giờ đã vượt qua. Đây là cách tốt nhất để chắc chắn rằng bạn thực sự đã sửa được lỗi. Nếu bạn sửa lỗi trước rồi mới viết bài kiểm thử, có khả năng bài kiểm thử đơn vị mới thực chất không hề kích hoạt đúng lỗi đó, và trong trường hợp ấy nó sẽ không cho bạn biết liệu bạn đã thực sự giải quyết được vấn đề hay chưa.

> 💡 **Ví dụ bổ sung (không có trong sách):** Quy trình sửa lỗi viết kiểm thử trước (Bug-fix Test-First):
>
> 1. Người dùng báo lỗi: Hàm `calculateDiscount(price, percent)` trả về số âm khi `percent = 120`.
> 2. **Bước 1 (Viết test trước):** Viết unit test `testDiscountNeverExceeds100Percent()` kiểm tra `calculateDiscount(100, 120) == 0`. Chạy test -> **FAIL** (tái hiện được lỗi).
> 3. **Bước 2 (Sửa lỗi):** Chặn giá trị `percent` trong khoảng `[0, 100]` (hoặc ném ngoại lệ hợp lệ tùy thiết kế).
> 4. **Bước 3 (Chạy lại test):** Chạy `testDiscountNeverExceeds100Percent()` -> **PASS**. Nhờ vậy bạn chắc chắn bài test thực sự kiểm tra đúng nguyên nhân lỗi và ngăn lỗi tái phát (regression) trong tương lai.

## 19.5 Các mẫu thiết kế (Design patterns)

Một mẫu thiết kế (design pattern) là một cách tiếp cận thường được sử dụng để giải quyết một loại bài toán cụ thể, chẳng hạn như bộ lặp (iterator) hoặc người quan sát (observer). Khái niệm mẫu thiết kế được phổ biến rộng rãi qua cuốn sách *Design Patterns: Elements of Reusable Object-Oriented Software* của Gamma, Helm, Johnson và Vlissides, và các mẫu thiết kế ngày nay được sử dụng phổ biến trong phát triển phần mềm hướng đối tượng.

Các mẫu thiết kế đại diện cho một phương án thay thế cho việc tự thiết kế: thay vì thiết kế một cơ chế mới từ đầu, chỉ cần áp dụng một mẫu thiết kế đã được biết đến rộng rãi. Phần lớn đây là một điều tốt: các mẫu thiết kế ra đời vì chúng giải quyết những bài toán phổ biến, và vì chúng được đồng thuận rộng rãi là mang lại những giải pháp sạch sẽ. Nếu một mẫu thiết kế hoạt động tốt trong một tình huống cụ thể, có lẽ bạn sẽ khó tự nghĩ ra một cách tiếp cận khác tốt hơn.

Rủi ro lớn nhất của các mẫu thiết kế là việc *áp dụng quá mức* (over-application). Không phải bài toán nào cũng có thể được giải quyết sạch sẽ bằng một mẫu thiết kế có sẵn; đừng cố ép một bài toán vào khuôn khổ của một mẫu thiết kế khi mà một cách tiếp cận tùy chỉnh sẽ sạch sẽ hơn. Việc sử dụng các mẫu thiết kế không tự động cải thiện một hệ thống phần mềm; nó chỉ làm được điều đó nếu các mẫu thiết kế thực sự phù hợp. Giống như nhiều ý tưởng khác trong thiết kế phần mềm, việc các mẫu thiết kế là tốt không nhất thiết có nghĩa là *càng nhiều* mẫu thiết kế thì *càng tốt*.

## 19.6 Getter và Setter (Getters and setters)

Trong cộng đồng lập trình Java, các phương thức getter và setter là một mẫu thiết kế phổ biến. Một getter và một setter được gắn với một biến thực thể của một lớp. Chúng có tên dạng `getFoo` và `setFoo`, trong đó `Foo` là tên của biến. Phương thức getter trả về giá trị hiện tại của biến, còn phương thức setter sửa đổi giá trị đó.

Getter và setter không phải là thứ bắt buộc về mặt kỹ thuật, bởi vì các biến thực thể hoàn toàn có thể được khai báo là công khai (`public`). Lập luận ủng hộ getter và setter là chúng cho phép thực hiện thêm các chức năng bổ sung trong khi lấy và gán giá trị, chẳng hạn như cập nhật các giá trị liên quan khi một biến thay đổi, thông báo cho các đối tượng lắng nghe (listener) về các thay đổi, hoặc thực thi các ràng buộc (constraint) lên giá trị. Ngay cả khi những tính năng này chưa cần thiết ở thời điểm ban đầu, chúng vẫn có thể được bổ sung sau này mà không làm thay đổi giao diện của lớp.

Mặc dù việc sử dụng getter và setter có thể hợp lý nếu bạn *bắt buộc* phải phơi bày các biến thực thể ra ngoài, nhưng ngay từ đầu tốt hơn hết là **không nên phơi bày các biến thực thể**. Việc phơi bày biến thực thể có nghĩa là một phần trong bản cài đặt của lớp bị lộ ra bên ngoài, điều này vi phạm nguyên lý ẩn giấu thông tin và làm tăng độ phức tạp của giao diện lớp. Getter và setter là các phương thức nông (shallow method — thường chỉ dài một dòng), nên chúng làm tăng thêm sự rườm rà (clutter) cho giao diện của lớp mà không cung cấp nhiều chức năng. Tốt hơn hết là tránh sử dụng getter và setter (hoặc bất kỳ hình thức phơi bày dữ liệu cài đặt nào) càng nhiều càng tốt.

Một trong những rủi ro của việc thiết lập một mẫu thiết kế là các lập trình viên mặc định cho rằng mẫu đó tốt và cố gắng sử dụng nó nhiều nhất có thể. Điều này đã dẫn đến tình trạng lạm dụng getter và setter trong Java.

> 💡 **Ví dụ bổ sung (không có trong sách):** Thay vì phơi bày trạng thái nội bộ qua getter/setter (mô hình lớp dữ liệu thụ động), hãy thiết kế các phương thức mang ngữ nghĩa nghiệp vụ sâu sắc hơn:
>
> ```java
> // Tránh: Lớp nông lạm dụng getter/setter, đẩy logic ra bên ngoài
> class BankAccount {
>     private double balance;
>     public double getBalance() { return balance; }
>     public void setBalance(double balance) { this.balance = balance; }
> }
> // Người gọi phải tự kiểm tra và tính toán:
> if (account.getBalance() >= amount) {
>     account.setBalance(account.getBalance() - amount);
> }
>
> // Nên: Ẩn giấu biến thực thể, cung cấp thao tác có chiều sâu
> class BankAccount {
>     private double balance;
>     public boolean withdraw(double amount) {
>         if (amount <= 0 || amount > balance) {
>             return false;
>         }
>         balance -= amount;
>         return true;
>     }
> }
> ```
> Với `withdraw(amount)`, lớp `BankAccount` tự bảo vệ tính toàn vẹn của `balance`, giảm rò rỉ thông tin và làm giao diện rõ ràng hơn.

## 19.7 Kết luận (Conclusion)

Bất cứ khi nào bạn bắt gặp một đề xuất cho một mô hình phát triển phần mềm mới, hãy chất vấn nó dưới góc độ của sự phức tạp: *liệu đề xuất đó có thực sự giúp giảm thiểu sự phức tạp trong các hệ thống phần mềm lớn hay không?* Nhiều đề xuất nghe có vẻ rất hay trên bề mặt, nhưng nếu bạn nhìn sâu hơn, bạn sẽ thấy một số trong chúng thực chất làm cho sự phức tạp trở nên tồi tệ hơn, chứ không phải tốt hơn.

## Bảng thuật ngữ chương (Chapter Glossary)

| Thuật ngữ (English) | Tiếng Việt | Giải thích ngắn gọn |
| :--- | :--- | :--- |
| Object-oriented programming (OOP) | Lập trình hướng đối tượng | Mô hình lập trình tổ chức phần mềm xoay quanh các lớp, đối tượng, đóng gói và kế thừa. |
| Interface inheritance | Kế thừa giao diện | Lớp cha chỉ định nghĩa chữ ký phương thức, các lớp con tự cài đặt theo nhiều cách khác nhau. |
| Implementation inheritance | Kế thừa cài đặt | Lớp cha cung cấp sẵn mã cài đặt mặc định để các lớp con dùng lại hoặc ghi đè. |
| Composition | Kết hợp | Xây dựng chức năng bằng cách chứa và gọi các lớp/đối tượng trợ giúp thay vì kế thừa từ lớp cha. |
| Information leakage | Rò rỉ thông tin | Chi tiết cài đặt nội bộ bị lộ hoặc chia sẻ giữa các lớp, tạo ra sự phụ thuộc chặt chẽ. |
| Agile development | Phát triển phần mềm linh hoạt (Agile) | Quy trình phát triển gia tăng, lặp lại, chú trọng phản hồi khách hàng và bàn giao phần mềm sớm. |
| Tactical programming | Lập trình chiến thuật | Viết mã nhanh để chạy được tính năng trước mắt mà ít đầu tư vào thiết kế trừu tượng dài hạn. |
| Unit test | Kiểm thử đơn vị | Bài kiểm thử nhỏ, chạy độc lập để xác nhận hành vi của một phương thức hoặc thành phần nhỏ. |
| System / Integration test | Kiểm thử hệ thống / tích hợp | Bài kiểm thử chạy toàn bộ hoặc nhiều phần của ứng dụng cùng nhau để kiểm tra sự phối hợp. |
| Refactoring | Tái cấu trúc mã nguồn | Cải thiện cấu trúc thiết kế bên trong của mã nguồn mà không làm thay đổi hành vi bên ngoài. |
| Test-driven development (TDD) | Phát triển hướng kiểm thử | Phương pháp viết bài kiểm thử đơn vị trước rồi mới viết vừa đủ mã nguồn để vượt qua từng bài kiểm thử. |
| Design pattern | Mẫu thiết kế | Giải pháp tái sử dụng đã được kiểm chứng cho một bài toán thiết kế phần mềm phổ biến. |
| Getter and setter | Phương thức lấy và gán giá trị | Các phương thức nông dùng để đọc (`getFoo`) và sửa đổi (`setFoo`) trực tiếp biến thực thể của lớp. |
