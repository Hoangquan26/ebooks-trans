# CHƯƠNG 8: Kéo Sự Phức Tạp Xuống Dưới (Chapter 8: Pull Complexity Downwards)

Chương này giới thiệu một cách tư duy khác về cách tạo ra các lớp sâu hơn (deeper classes). Giả sử bạn đang phát triển một mô-đun mới và phát hiện ra một phần phức tạp không thể tránh khỏi. Cách nào tốt hơn: bạn nên để người dùng của mô-đun tự xoay xở với sự phức tạp đó, hay bạn nên xử lý sự phức tạp đó ngay bên trong mô-đun? Nếu sự phức tạp đó có liên quan đến chức năng mà mô-đun cung cấp, thì câu trả lời thứ hai thường là lựa chọn đúng đắn. Hầu hết các mô-đun đều có số lượng người dùng nhiều hơn số lượng nhà phát triển, vì vậy thà để nhà phát triển chịu khổ còn hơn là để người dùng chịu khổ. Là một nhà phát triển mô-đun, bạn nên nỗ lực làm cho cuộc sống của người dùng mô-đun trở nên dễ dàng nhất có thể, ngay cả khi điều đó đồng nghĩa với việc bạn phải làm thêm nhiều việc hơn. Một cách diễn đạt khác cho ý tưởng này là: một mô-đun có giao diện (interface) đơn giản quan trọng hơn là có phần cài đặt (implementation) đơn giản.

Là một nhà phát triển, rất dễ bị cám dỗ hành xử theo chiều ngược lại: giải quyết những vấn đề dễ và đẩy những vấn đề khó cho người khác. Nếu một tình huống phát sinh mà bạn không chắc chắn cách xử lý, điều dễ nhất là ném ra một ngoại lệ (exception) và để bên gọi tự giải quyết. Nếu bạn không chắc chắn nên áp dụng chính sách (policy) nào, bạn có thể định nghĩa một vài tham số cấu hình (configuration parameters) để kiểm soát chính sách đó và giao phó cho quản trị viên hệ thống tự tìm ra các giá trị tốt nhất cho chúng.

Những cách tiếp cận như thế này sẽ giúp cuộc sống của bạn dễ dàng hơn trong ngắn hạn, nhưng chúng lại khuếch đại sự phức tạp, khiến nhiều người phải đối mặt với một vấn đề thay vì chỉ một người. Ví dụ, nếu một lớp ném ra một ngoại lệ, mọi bên gọi lớp đó đều sẽ phải xử lý nó. Nếu một lớp xuất ra (export) các tham số cấu hình, mọi quản trị viên hệ thống ở mọi nơi cài đặt sẽ phải học cách thiết lập chúng.

> 📝 **Giải thích thêm (không có trong sách):**
> Khái niệm **"kéo sự phức tạp xuống dưới" (pull complexity downwards)** xuất phát từ mô hình phân tầng kiến trúc phần mềm: các mô-đun nền tảng/hạ tầng nằm ở tầng dưới, còn các mô-đun ứng dụng/giao diện người dùng nằm ở tầng trên. Khi bạn giải quyết triệt để một vấn đề ở tầng dưới (trong một mô-đun duy nhất), mọi mô-đun ở tầng trên gọi đến nó đều được hưởng lợi từ một giao diện sạch sẽ, đơn giản. Ngược lại, nếu bạn đẩy vấn đề lên tầng trên (ném ngoại lệ vô tội vạ, đòi hỏi người dùng tự cấu hình mọi thứ), sự phức tạp sẽ nhân lên theo số lượng người gọi (callers) hoặc số lượng hệ thống được triển khai.

## 8.1 Ví dụ: lớp văn bản của trình soạn thảo (Example: editor text class)

Hãy xem xét lớp quản lý văn bản của một tệp tin cho trình soạn thảo văn bản giao diện đồ họa (GUI text editor), vốn đã được thảo luận trong Chương 6 và Chương 7. Lớp này cung cấp các phương thức để đọc một tệp từ đĩa vào bộ nhớ, truy vấn và chỉnh sửa bản sao trong bộ nhớ của tệp, và ghi phiên bản đã chỉnh sửa trở lại đĩa. Khi sinh viên phải cài đặt lớp này, nhiều người trong số họ đã chọn một giao diện hướng dòng (line-oriented interface), với các phương thức để đọc, chèn và xóa nguyên cả dòng văn bản. Điều này mang lại một phần cài đặt đơn giản cho lớp, nhưng lại tạo ra sự phức tạp cho phần mềm ở tầng cao hơn. Ở tầng giao diện người dùng (user interface), các thao tác hiếm khi liên quan đến toàn bộ dòng. Ví dụ, các phím bấm khiến từng ký tự riêng lẻ được chèn vào bên trong một dòng đang tồn tại; việc sao chép hoặc xóa vùng chọn có thể làm thay đổi một phần của nhiều dòng khác nhau. Với giao diện văn bản hướng dòng, phần mềm tầng cao hơn đã phải tự tách (split) và ghép (join) các dòng để cài đặt giao diện người dùng.

Một giao diện hướng ký tự (character-oriented interface) như đã mô tả trong Mục 6.3 sẽ kéo sự phức tạp xuống dưới. Phần mềm giao diện người dùng giờ đây có thể chèn và xóa các dải văn bản tùy ý mà không cần phải tách và gộp dòng, nhờ đó trở nên đơn giản hơn. Phần cài đặt của lớp văn bản có thể trở nên phức tạp hơn: nếu bên trong nó biểu diễn văn bản dưới dạng một tập hợp các dòng, nó sẽ phải tự tách và gộp dòng để thực hiện các thao tác hướng ký tự. Cách tiếp cận này tốt hơn vì nó đóng gói (encapsulate) sự phức tạp của việc tách và gộp dòng vào bên trong lớp văn bản, từ đó làm giảm tổng thể sự phức tạp của hệ thống.

> 💡 **Ví dụ bổ sung (không có trong sách):**
> Giả sử bạn viết một thư viện đọc/ghi tệp CSV.
> - **Đẩy phức tạp lên trên:** Thư viện chỉ trả về từng dòng thô (`String`), buộc mọi nơi sử dụng phải tự tách dấu phẩy, tự xử lý trường hợp có dấu phẩy nằm trong dấu ngoặc kép (ví dụ `"Hà Nội, Việt Nam"`), và tự xử lý ký tự xuống dòng `\n` nằm bên trong một ô.
> - **Kéo phức tạp xuống dưới:** Thư viện cung cấp hàm `readRow()` trả về danh sách các ô (`List<String>`) đã được phân tích cú pháp hoàn chỉnh theo chuẩn RFC 4180. Code bên trong thư viện phức tạp hơn một chút (cần máy trạng thái để xử lý dấu ngoặc kép và ký tự thoát), nhưng hàng chục nơi gọi thư viện ở tầng nghiệp vụ chỉ việc dùng kết quả sạch sẽ mà không bao giờ gặp lỗi phân tích CSV.

## 8.2 Ví dụ: tham số cấu hình (Example: configuration parameters)

Các tham số cấu hình (configuration parameters) là một ví dụ về việc đẩy sự phức tạp lên trên thay vì kéo xuống dưới. Thay vì tự xác định một hành vi cụ thể ở bên trong, một lớp có thể xuất ra một vài tham số để điều khiển hành vi của nó, chẳng hạn như kích thước của bộ nhớ đệm (cache) hoặc số lần thử lại (retry) một yêu cầu trước khi bỏ cuộc. Người dùng của lớp khi đó phải chỉ định các giá trị thích hợp cho các tham số này. Các tham số cấu hình đã trở nên rất phổ biến trong các hệ thống ngày nay; một số hệ thống có hàng trăm tham số như vậy.

Những người ủng hộ lập luận rằng các tham số cấu hình là tốt vì chúng cho phép người dùng tinh chỉnh (tune) hệ thống cho các yêu cầu và khối lượng công việc (workloads) cụ thể của họ. Trong một số tình huống, rất khó để mã nguồn hạ tầng tầng thấp biết được chính sách nào là tốt nhất để áp dụng, trong khi người dùng lại am hiểu hơn nhiều về lĩnh vực (domain) của họ. Chẳng hạn, một người dùng có thể biết rằng một số yêu cầu mang tính khẩn cấp về thời gian hơn các yêu cầu khác, nên việc người dùng chỉ định mức độ ưu tiên cao hơn cho các yêu cầu đó là hợp lý. Trong những tình huống như thế này, các tham số cấu hình có thể mang lại hiệu năng tốt hơn trên nhiều lĩnh vực đa dạng.

Tuy nhiên, các tham số cấu hình cũng cung cấp một cái cớ dễ dàng để né tránh việc giải quyết các vấn đề quan trọng và đẩy chúng sang cho người khác. Trong nhiều trường hợp, người dùng hoặc quản trị viên rất khó hoặc không thể xác định được giá trị đúng cho các tham số. Trong những trường hợp khác, các giá trị đúng hoàn toàn có thể được xác định tự động chỉ với một chút công sức làm thêm trong phần cài đặt hệ thống.

Hãy xem xét một giao thức mạng phải đối phó với tình trạng mất gói tin (lost packets). Nếu nó gửi một yêu cầu nhưng không nhận được phản hồi trong một khoảng thời gian nhất định, nó sẽ gửi lại yêu cầu đó. Một cách để xác định khoảng thời gian chờ thử lại (retry interval) là đưa ra một tham số cấu hình. Tuy nhiên, giao thức truyền tải có thể tự tính toán một giá trị hợp lý bằng cách đo thời gian phản hồi cho các yêu cầu thành công và sau đó lấy một bội số của giá trị này làm khoảng thời gian chờ thử lại. Cách tiếp cận này kéo sự phức tạp xuống dưới và giúp người dùng không phải tự mò mẫm tìm ra khoảng thời gian thử lại phù hợp. Nó còn có thêm lợi thế là tính toán khoảng thời gian thử lại một cách linh hoạt (dynamically), nhờ đó nó sẽ tự động điều chỉnh nếu điều kiện vận hành thay đổi. Ngược lại, các tham số cấu hình rất dễ trở nên lỗi thời.

Do đó, bạn nên tránh các tham số cấu hình nhiều nhất có thể. Trước khi xuất ra một tham số cấu hình, hãy tự hỏi bản thân: "Liệu người dùng (hoặc các mô-đun tầng cao hơn) có thể xác định được một giá trị tốt hơn giá trị mà chúng ta có thể tự xác định ở đây không?" Khi bạn thực sự phải tạo ra các tham số cấu hình, hãy xem liệu bạn có thể tự động tính toán các giá trị mặc định hợp lý hay không, để người dùng chỉ cần cung cấp giá trị trong những điều kiện ngoại lệ. Lý tưởng nhất là mỗi mô-đun nên giải quyết trọn vẹn một vấn đề; các tham số cấu hình dẫn đến một giải pháp chưa hoàn chỉnh, làm tăng thêm sự phức tạp cho hệ thống.

> 💡 **Ví dụ bổ sung (không có trong sách):**
> Trong thực tế phát triển ứng dụng web hoặc microservice:
> - **Thiết kế đẩy phức tạp lên trên:** Bắt người dùng (hoặc DevOps) cấu hình tay `thread_pool_size`, `connection_timeout_ms`, `read_timeout_ms`, `max_retries`, `backoff_base_ms`, `backoff_max_ms` trong file config mà không có giá trị mặc định thông minh. Khi lưu lượng tăng gấp 3 lần hoặc mạng chậm đi, cấu hình cũ gây nghẽn cổ chai hoặc bão thử lại (retry storm).
> - **Thiết kế kéo phức tạp xuống dưới:** Hệ thống tự lấy số nhân CPU để đặt mặc định kích thước pool hợp lý, tự đo độ trễ p95/p99 để điều chỉnh timeout động (adaptive timeout) hoặc dùng thuật toán exponential backoff kèm jitter có sẵn, và chỉ cho phép ghi đè (override) cấu hình khi có nhu cầu đặc thù hiếm gặp.

## 8.3 Đi quá xa (Taking it too far)

Hãy sử dụng sự cân nhắc thận trọng khi kéo sự phức tạp xuống dưới; đây là một ý tưởng rất dễ bị làm quá mức. Một cách tiếp cận cực đoan sẽ là kéo toàn bộ chức năng của cả ứng dụng xuống vào trong một lớp duy nhất, điều này rõ ràng là vô lý. Việc kéo sự phức tạp xuống dưới có ý nghĩa nhất nếu: (a) sự phức tạp được kéo xuống có liên quan chặt chẽ đến chức năng hiện có của lớp, (b) việc kéo sự phức tạp xuống sẽ mang lại nhiều sự đơn giản hóa ở những nơi khác trong ứng dụng, và (c) việc kéo sự phức tạp xuống làm đơn giản hóa giao diện của lớp. Hãy nhớ rằng mục tiêu là giảm thiểu tổng thể sự phức tạp của hệ thống.

Chương 6 đã mô tả cách một số sinh viên định nghĩa các phương thức trong lớp văn bản phản ánh giao diện người dùng, chẳng hạn như một phương thức cài đặt chức năng của phím Backspace. Có vẻ như điều này là tốt, vì nó kéo sự phức tạp xuống dưới. Tuy nhiên, việc thêm kiến thức về giao diện người dùng vào lớp văn bản không giúp đơn giản hóa mã nguồn tầng cao hơn nhiều, và kiến thức về giao diện người dùng cũng không liên quan đến các chức năng cốt lõi của lớp văn bản. Trong trường hợp này, việc kéo sự phức tạp xuống chỉ dẫn đến rò rỉ thông tin (information leakage).

> 📝 **Giải thích thêm (không có trong sách):**
> Để kiểm tra xem bạn có đang "đi quá xa" khi kéo sự phức tạp xuống dưới hay không, hãy áp dụng 3 tiêu chí của Ousterhout như một bộ lọc:
> 1. **Tính liên quan (Cohesion):** Phần phức tạp này có thực sự thuộc về bản chất của mô-đun này không? (Xử lý tách/gộp dòng thuộc về `Text`, còn phím Backspace thuộc về `UI`).
> 2. **Tác động lan tỏa (Leverage):** Kéo nó xuống có giúp nhiều chỗ khác đơn giản hơn rõ rệt không?
> 3. **Giao diện gọn hơn (Simpler interface):** Giao diện công khai (public API) của mô-đun có trở nên tinh gọn hơn không, hay lại phình to thêm các phương thức đặc thù?

## 8.4 Kết luận (Conclusion)

Khi phát triển một mô-đun, hãy tìm kiếm cơ hội để tự nhận lấy một chút khổ cực về phần mình nhằm làm giảm bớt sự khổ cực cho người dùng của bạn.

## Bảng thuật ngữ chương (Chapter Glossary)

| Thuật ngữ (English) | Tiếng Việt | Giải thích ngắn gọn |
| :--- | :--- | :--- |
| Pull complexity downwards | Kéo sự phức tạp xuống dưới | Nguyên tắc thiết kế xử lý sự phức tạp ngay bên trong mô-đun tầng dưới để giữ cho giao diện và mã nguồn tầng trên đơn giản. |
| Configuration parameter | Tham số cấu hình | Giá trị do người dùng hoặc quản trị viên thiết lập từ bên ngoài để điều khiển hành vi của mô-đun. |
| Line-oriented interface | Giao diện hướng dòng | Giao diện thao tác văn bản theo đơn vị từng dòng nguyên vẹn. |
| Character-oriented interface | Giao diện hướng ký tự | Giao diện thao tác văn bản theo từng ký tự hoặc dải ký tự tùy ý, không phụ thuộc ranh giới dòng. |
| Retry interval | Khoảng thời gian chờ thử lại | Thời gian chờ trước khi gửi lại một yêu cầu mạng khi chưa nhận được phản hồi. |
| Information leakage | Rò rỉ thông tin | Hiện tượng một quyết định thiết kế hoặc kiến thức của tầng/mô-đun này bị lộ hoặc phụ thuộc vào tầng/mô-đun khác. |
| Encapsulate | Đóng gói | Che giấu sự phức tạp và chi tiết cài đặt bên trong ranh giới của một mô-đun hoặc lớp. |
