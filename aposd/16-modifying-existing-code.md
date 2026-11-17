# CHƯƠNG 16: Chỉnh sửa mã nguồn hiện có (Chapter 16: Modifying Existing Code)

Chương 1 đã mô tả cách phát triển phần mềm mang tính lặp lại và gia tăng (iterative and incremental). Một hệ thống phần mềm lớn phát triển qua một loạt các giai đoạn tiến hóa, trong đó mỗi giai đoạn lại bổ sung các tính năng mới và chỉnh sửa các mô-đun hiện có. Điều này có nghĩa là thiết kế của một hệ thống luôn không ngừng biến đổi. Ta không thể nào hình dung ra được thiết kế chuẩn xác ngay từ đầu; thiết kế của một hệ thống trưởng thành được định hình bởi các thay đổi trong quá trình tiến hóa của hệ thống nhiều hơn là bởi bất kỳ ý tưởng ban đầu nào. Các chương trước đã trình bày cách loại bỏ sự phức tạp trong quá trình thiết kế và cài đặt ban đầu; chương này sẽ thảo luận về cách ngăn chặn sự phức tạp âm thầm len lỏi vào hệ thống khi nó tiến hóa.

## 16.1 Luôn giữ tư duy chiến lược (Stay strategic)

Chương 3 đã giới thiệu sự khác biệt giữa lập trình chiến thuật (tactical programming) và lập trình chiến lược (strategic programming): trong lập trình chiến thuật, mục tiêu chính là làm cho mọi thứ chạy được một cách nhanh chóng, ngay cả khi điều đó tạo ra thêm sự phức tạp; trong lập trình chiến lược, mục tiêu quan trọng nhất là tạo ra một thiết kế hệ thống tuyệt vời. Cách tiếp cận chiến thuật rất nhanh chóng dẫn đến một thiết kế hệ thống lộn xộn. Nếu bạn muốn sở hữu một hệ thống dễ bảo trì và dễ mở rộng, thì chỉ "chạy được" là chưa đủ; bạn phải ưu tiên thiết kế và tư duy một cách chiến lược. Ý tưởng này cũng áp dụng khi bạn chỉnh sửa mã nguồn hiện có.

Đáng tiếc là khi các lập trình viên đi vào mã nguồn hiện có để thực hiện các thay đổi như sửa lỗi hoặc thêm tính năng mới, họ thường không tư duy theo hướng chiến lược. Một lối suy nghĩ điển hình là: "Thay đổi nhỏ nhất có thể mà tôi có thể thực hiện để làm được việc mình cần là gì?" Đôi khi các lập trình viên biện minh cho điều này vì họ không cảm thấy thoải mái với đoạn mã đang sửa; họ lo ngại rằng những thay đổi lớn hơn sẽ mang lại rủi ro cao hơn trong việc tạo ra lỗi mới. Tuy nhiên, điều này lại dẫn đến lập trình chiến thuật. Mỗi một thay đổi tối thiểu như vậy lại đưa thêm vào vài trường hợp đặc biệt (special cases), các phụ thuộc (dependencies), hoặc những dạng phức tạp khác. Kết quả là thiết kế hệ thống trở nên tệ đi một chút, và các vấn đề cứ thế tích tụ qua từng bước tiến hóa của hệ thống.

> 💡 **Ví dụ bổ sung (không có trong sách):**
> Giả sử hàm `calculateShippingFee(Order order)` ban đầu chỉ tính phí dựa trên cân nặng. Khi có yêu cầu mới: "Đơn hàng của khách VIP được miễn phí ship nếu tổng tiền > 500k", một lập trình viên chiến thuật sẽ chèn ngay một khối `if (order.isVip() && order.getTotal() > 500000) return 0;` vào đầu hàm. Vài tháng sau, có thêm yêu cầu "Đơn hàng nội thành vào ngày lễ giảm 50%", lại có người chèn thêm một câu `if` nữa. Cứ như vậy, hàm `calculateShippingFee` biến thành một mớ bòng bong các điều kiện chắp vá thay vì được tái cấu trúc (refactor) thành một mô hình tính giá/khuyến mãi sạch sẽ (ví dụ: áp dụng Strategy Pattern hoặc tách biệt rõ ràng logic tính phí cơ bản và chính sách giảm giá).

Nếu bạn muốn duy trì một thiết kế sạch sẽ cho hệ thống, bạn phải áp dụng cách tiếp cận chiến lược khi chỉnh sửa mã nguồn hiện có. Lý tưởng nhất là sau khi bạn hoàn thành mỗi thay đổi, hệ thống sẽ có được cấu trúc đúng như thể bạn đã thiết kế nó ngay từ đầu với thay đổi đó trong tâm trí. Để đạt được mục tiêu này, bạn phải cưỡng lại sự cám dỗ của việc vá lỗi nhanh (quick fix). Thay vào đó, hãy suy nghĩ xem thiết kế hệ thống hiện tại có còn là thiết kế tốt nhất khi xét đến thay đổi mong muốn hay không. Nếu không, hãy tái cấu trúc (refactor) hệ thống để bạn có được thiết kế tốt nhất có thể. Với cách tiếp cận này, thiết kế hệ thống sẽ được cải thiện sau mỗi lần chỉnh sửa.

Đây cũng là một ví dụ về tư duy đầu tư (investment mindset) đã được giới thiệu ở trang 15 (Chương 3): nếu bạn đầu tư thêm một chút thời gian để tái cấu trúc và cải thiện thiết kế hệ thống, bạn sẽ nhận lại một hệ thống sạch sẽ hơn. Điều này sẽ đẩy nhanh tốc độ phát triển, và bạn sẽ thu hồi lại được công sức đã bỏ ra cho việc tái cấu trúc. Ngay cả khi thay đổi cụ thể của bạn không đòi hỏi phải tái cấu trúc, bạn vẫn nên để mắt tìm kiếm những điểm chưa hoàn hảo trong thiết kế mà bạn có thể sửa luôn khi đang làm việc với đoạn mã đó. Bất cứ khi nào bạn chỉnh sửa mã nguồn, hãy cố gắng tìm cách cải thiện thiết kế hệ thống ít nhất một chút trong quá trình đó. Nếu bạn không làm cho thiết kế tốt hơn, nhiều khả năng bạn đang làm cho nó tệ đi.

Như đã thảo luận ở Chương 3, tư duy đầu tư đôi khi xung đột với thực tế của việc phát triển phần mềm thương mại. Nếu việc tái cấu trúc hệ thống "theo đúng cách" mất tới ba tháng trong khi một bản vá nhanh gọn (quick and dirty fix) chỉ mất hai giờ, bạn có thể phải chấp nhận cách làm nhanh gọn, đặc biệt nếu bạn đang chạy đua với một hạn chót (deadline) gấp rút. Hoặc, nếu việc tái cấu trúc hệ thống sẽ tạo ra sự không tương thích làm ảnh hưởng đến nhiều người và nhiều nhóm khác, thì việc tái cấu trúc đó có thể không khả thi.

Tuy nhiên, bạn nên hạn chế những sự thỏa hiệp này càng nhiều càng tốt. Hãy tự hỏi bản thân: "Đây đã là cách tốt nhất mà mình có thể làm để tạo ra một thiết kế hệ thống sạch sẽ, trong giới hạn các ràng buộc hiện tại chưa?" Biết đâu có một phương án thay thế gần sạch sẽ bằng bản tái cấu trúc 3 tháng nhưng chỉ mất vài ngày để làm? Hoặc, nếu bạn không thể chi trả thời gian cho một đợt tái cấu trúc lớn ngay lúc này, hãy đề nghị sếp phân bổ thời gian để bạn quay lại xử lý nó sau khi qua hạn chót hiện tại. Mọi tổ chức phát triển phần mềm đều nên lên kế hoạch dành một phần nhỏ trong tổng nỗ lực của mình cho việc dọn dẹp và tái cấu trúc; công việc này chắc chắn sẽ mang lại lợi ích xứng đáng về lâu dài.

> 📝 **Giải thích thêm (không có trong sách):**
> Nguyên tắc này rất gần với **"Quy tắc hướng đạo sinh" (Boy Scout Rule)** trong kỹ thuật phần mềm: *"Hãy luôn để lại khu cắm trại sạch sẽ hơn lúc bạn mới đến."* Khi vào sửa code cũ, đừng chỉ sửa cho xong việc của mình rồi đi ra; hãy dọn dẹp một tên biến tối nghĩa, tách một hàm quá dài, hoặc cập nhật lại chú thích bị lệch pha.

## 16.2 Duy trì chú thích: giữ chú thích ở gần mã nguồn (Maintaining comments: keep the comments near the code)

Khi bạn thay đổi mã nguồn hiện có, rất có khả năng những thay đổi đó sẽ làm mất hiệu lực của một số chú thích hiện có. Rất dễ xảy ra tình trạng quên cập nhật chú thích khi sửa code, dẫn đến những chú thích không còn chính xác nữa. Chú thích sai lệch gây ức chế cho người đọc, và nếu có quá nhiều chú thích như vậy, người đọc sẽ bắt đầu mất lòng tin vào toàn bộ hệ thống chú thích. May mắn thay, chỉ với một chút kỷ luật và vài quy tắc định hướng, việc giữ cho chú thích luôn cập nhật là hoàn toàn khả thi mà không tốn quá nhiều công sức. Phần này và các phần tiếp theo sẽ đưa ra một số kỹ thuật cụ thể.

Cách tốt nhất để đảm bảo chú thích được cập nhật là đặt chúng ở gần đoạn mã mà chúng mô tả, sao cho các lập trình viên sẽ nhìn thấy chúng khi thay đổi mã nguồn. Chú thích càng nằm xa đoạn mã liên quan, khả năng nó được cập nhật đúng cách càng thấp. Ví dụ, vị trí tốt nhất cho chú thích giao diện (interface comment) của một phương thức là nằm trong tệp mã nguồn, ngay bên cạnh thân của phương thức đó. Mọi thay đổi đối với phương thức đều sẽ đụng chạm đến đoạn mã này, vì vậy lập trình viên rất dễ nhìn thấy chú thích giao diện và cập nhật nó nếu cần.

Một lựa chọn khác đối với các ngôn ngữ như C và C++ (vốn tách biệt tệp mã nguồn và tệp tiêu đề - header file) là đặt chú thích giao diện ngay bên cạnh phần khai báo phương thức trong tệp `.h`. Tuy nhiên, vị trí này nằm rất xa đoạn mã cài đặt; lập trình viên sẽ không nhìn thấy những chú thích đó khi chỉnh sửa thân phương thức, và họ sẽ tốn thêm công sức để mở một tệp khác rồi tìm đúng chú thích giao diện để cập nhật. Một số người có thể lập luận rằng chú thích giao diện nên nằm trong tệp tiêu đề để người dùng có thể học cách sử dụng một trừu tượng hóa mà không cần phải nhìn vào tệp mã nguồn. Tuy nhiên, người dùng vốn dĩ không cần phải đọc cả tệp mã nguồn lẫn tệp tiêu đề; họ nên lấy thông tin từ tài liệu được biên dịch bởi các công cụ như Doxygen hay Javadoc. Ngoài ra, nhiều IDE hiện đại cũng sẽ tự động trích xuất và hiển thị tài liệu cho người dùng, chẳng hạn như hiển thị tài liệu của một phương thức khi bạn gõ tên phương thức đó. Với những công cụ như vậy, tài liệu nên được đặt ở nơi thuận tiện nhất cho các lập trình viên đang trực tiếp làm việc trên mã nguồn.

Khi viết chú thích cài đặt (implementation comments), đừng dồn tất cả chú thích cho toàn bộ phương thức lên đầu phương thức. Hãy dàn trải chúng ra, đẩy mỗi chú thích xuống phạm vi hẹp nhất (narrowest scope) bao trọn tất cả các dòng mã mà chú thích đó đề cập đến. Ví dụ, nếu một phương thức có ba giai đoạn chính, đừng viết một khối chú thích ở đầu phương thức để mô tả chi tiết cả ba giai đoạn. Thay vào đó, hãy viết một chú thích riêng cho từng giai đoạn và đặt chú thích đó ngay phía trên dòng mã đầu tiên của giai đoạn tương ứng. Mặt khác, việc có một chú thích ở đầu phần cài đặt của phương thức để mô tả chiến lược tổng thể cũng có thể rất hữu ích, chẳng hạn như thế này:

```cpp
// Chúng ta thực hiện qua ba giai đoạn:
// Giai đoạn 1: Tìm các ứng viên khả thi
// Giai đoạn 2: Chấm điểm cho từng ứng viên
// Giai đoạn 3: Chọn ứng viên tốt nhất và loại bỏ nó khỏi danh sách
```

*(Nguyên văn trong sách:)*
```cpp
//  We proceed in three phases:
//  Phase 1: Find feasible candidates
//  Phase 2: Assign each candidate a score
//  Phase 3: Choose the best, and remove it
```

Các chi tiết bổ sung có thể được ghi chú ngay phía trên đoạn mã của từng giai đoạn. Nhìn chung, chú thích nằm càng xa đoạn mã mà nó mô tả thì mức độ mô tả của nó càng nên trừu tượng hơn (điều này giúp giảm khả năng chú thích bị mất hiệu lực do các thay đổi nhỏ trong mã nguồn).

## 16.3 Chú thích thuộc về mã nguồn, không phải nhật ký commit (Comments belong in the code, not the commit log)

Một sai lầm phổ biến khi chỉnh sửa mã nguồn là ghi thông tin chi tiết về thay đổi vào thông điệp commit (commit message) của kho lưu trữ mã nguồn, nhưng lại không ghi chú điều đó vào trong chính mã nguồn. Mặc dù trong tương lai người ta vẫn có thể xem lại các thông điệp commit bằng cách rà soát nhật ký của kho lưu trữ (repository log), nhưng một lập trình viên khi cần thông tin đó hiếm khi nghĩ đến việc đi lục lại nhật ký kho lưu trữ. Ngay cả khi họ có đi tìm trong nhật ký, việc lần ra đúng thông điệp commit cần tìm cũng rất mất công và nhàm chán.

Khi viết thông điệp commit, hãy tự hỏi bản thân xem các lập trình viên trong tương lai có cần sử dụng thông tin đó hay không. Nếu có, hãy ghi chú thông tin này ngay trong mã nguồn. Một ví dụ điển hình là thông điệp commit mô tả một vấn đề tinh vi (subtle problem) đã dẫn đến thay đổi mã nguồn đó. Nếu điều này không được ghi chú trong mã nguồn, một lập trình viên khác có thể đi ngang qua sau này và hoàn tác (undo) thay đổi đó mà không nhận ra mình vừa tái tạo lại một lỗi cũ. Nếu bạn muốn đưa một bản sao của thông tin này vào thông điệp commit nữa thì cũng tốt thôi, nhưng điều quan trọng nhất là phải đưa được nó vào trong mã nguồn. Điều này minh họa cho nguyên tắc: hãy đặt tài liệu ở nơi mà lập trình viên có khả năng nhìn thấy cao nhất; nhật ký commit hiếm khi là nơi đó.

> 💡 **Ví dụ bổ sung (không có trong sách):**
> Giả sử bạn phải thêm `Thread.sleep(50)` hoặc một lệnh `flush()` tưởng chừng thừa thãi để tránh một race condition cực hiếm gặp trên hệ thống tệp của Windows. Nếu bạn chỉ giải thích điều đó trong commit message `"Fix race condition on Windows FS"`, thì một năm sau, đồng nghiệp của bạn đọc code thấy dòng `flush()` vô duyên sẽ xóa nó đi để "tối ưu hiệu năng" — và lỗi cũ quay trở lại! Thay vào đó, một dòng chú thích ngay trên dòng lệnh đó trong code sẽ cứu cả đội khỏi một lỗi khó tái hiện.

## 16.4 Duy trì chú thích: tránh lặp lại (Maintaining comments: avoid duplication)

Kỹ thuật thứ hai để giữ cho chú thích luôn được cập nhật là tránh lặp lại (avoid duplication). Nếu tài liệu bị sao chép lặp lại ở nhiều nơi, các lập trình viên sẽ gặp khó khăn hơn trong việc tìm kiếm và cập nhật tất cả các bản sao liên quan. Thay vào đó, hãy cố gắng ghi chú mỗi quyết định thiết kế đúng một lần duy nhất (document each design decision exactly once). Nếu có nhiều vị trí trong mã nguồn chịu ảnh hưởng bởi một quyết định cụ thể, đừng lặp lại phần tài liệu đó ở tất cả các vị trí. Thay vào đó, hãy tìm một vị trí duy nhất hiển nhiên nhất để đặt tài liệu. Ví dụ, giả sử có một hành vi phức tạp liên quan đến một biến, và nó ảnh hưởng đến nhiều chỗ khác nhau nơi biến đó được sử dụng. Bạn có thể ghi chú hành vi đó trong phần chú thích ngay bên cạnh dòng khai báo biến. Đây là nơi tự nhiên nhất mà các lập trình viên có khả năng kiểm tra đầu tiên nếu họ gặp khó khăn khi đọc hiểu đoạn mã sử dụng biến đó.

Nếu không có một vị trí đơn lẻ nào "hiển nhiên" để đặt một mẩu tài liệu cụ thể sao cho lập trình viên dễ dàng tìm thấy, hãy tạo một tệp `designNotes` như đã mô tả ở Mục 13.7. Hoặc, hãy chọn vị trí tốt nhất trong số các vị trí sẵn có và đặt tài liệu ở đó. Ngoài ra, hãy thêm các chú thích ngắn ở những vị trí khác để dẫn chiếu về vị trí trung tâm: `"Xem chú thích tại xyz để hiểu giải thích cho đoạn mã bên dưới."` (`"See the comment in xyz for an explanation of the code below."`). Nếu tham chiếu này trở nên lỗi thời vì chú thích gốc đã bị di chuyển hoặc xóa đi, sự bất nhất này sẽ tự bộc lộ rõ ràng vì lập trình viên sẽ không tìm thấy chú thích tại vị trí được chỉ dẫn; khi đó họ có thể dùng lịch sử quản lý phiên bản (revision control history) để xem chuyện gì đã xảy ra với chú thích gốc rồi cập nhật lại tham chiếu. Ngược lại, nếu tài liệu bị lặp lại và một vài bản sao không được cập nhật, sẽ chẳng có dấu hiệu nào báo cho lập trình viên biết rằng họ đang đọc phải thông tin cũ kỹ, lỗi thời.

Đừng ghi chú lại các quyết định thiết kế của một mô-đun này bên trong một mô-đun khác. Ví dụ, đừng đặt chú thích trước lời gọi một phương thức để giải thích những gì diễn ra bên trong phương thức được gọi. Nếu người đọc muốn biết, họ nên xem chú thích giao diện (interface comments) của phương thức đó. Các công cụ phát triển tốt thường sẽ cung cấp thông tin này một cách tự động, chẳng hạn bằng cách hiển thị chú thích giao diện của phương thức khi bạn chọn tên phương thức hoặc di chuột qua nó. Hãy cố gắng giúp lập trình viên dễ dàng tìm thấy tài liệu phù hợp, nhưng đừng làm điều đó bằng cách lặp lại tài liệu.

Nếu thông tin đã được ghi chép ở một nơi nào đó bên ngoài chương trình của bạn, đừng lặp lại tài liệu đó bên trong chương trình; chỉ cần dẫn chiếu đến tài liệu bên ngoài đó. Ví dụ, nếu bạn viết một lớp cài đặt giao thức HTTP, bạn không cần phải mô tả giao thức HTTP bên trong mã nguồn của mình. Đã có vô số nguồn tài liệu về giao thức này trên Web; bạn chỉ cần thêm một chú thích ngắn vào mã nguồn kèm theo đường dẫn URL tới một trong các nguồn đó. Một ví dụ khác là các tính năng đã được mô tả trong sách hướng dẫn sử dụng (user manual). Giả sử bạn đang viết một chương trình cài đặt một tập hợp các lệnh (commands), với mỗi phương thức chịu trách nhiệm cài đặt một lệnh. Nếu đã có sách hướng dẫn sử dụng mô tả các lệnh đó, không cần phải lặp lại thông tin này trong mã nguồn. Thay vào đó, hãy chèn một ghi chú ngắn như sau vào phần chú thích giao diện của mỗi phương thức lệnh:

```java
// Cài đặt lệnh Foo; xem sách hướng dẫn sử dụng để biết chi tiết.
// (Nguyên văn: Implements the Foo command; see the user manual for details.)
```

Điều quan trọng là người đọc có thể dễ dàng tìm thấy mọi tài liệu cần thiết để hiểu mã nguồn của bạn, nhưng điều đó không có nghĩa là bạn phải tự tay viết lại tất cả các tài liệu đó.

## 16.5 Duy trì chú thích: kiểm tra bản diff (Maintaining comments: check the diffs)

Một cách hay để đảm bảo tài liệu luôn được cập nhật là dành vài phút trước khi commit một thay đổi lên hệ thống quản lý phiên bản để rà soát lại toàn bộ các thay đổi (diffs) trong lần commit đó; hãy đảm bảo rằng mỗi thay đổi đều đã được phản ánh đầy đủ trong phần tài liệu. Việc rà soát trước khi commit (pre-commit scans) này cũng sẽ giúp phát hiện nhiều vấn đề khác, chẳng hạn như vô tình để sót mã gỡ lỗi (debugging code) trong hệ thống hoặc quên xử lý các mục `TODO`.

> 📝 **Giải thích thêm (không có trong sách):**
> Trong quy trình làm việc hiện đại với Git, việc chạy `git diff` (hoặc xem lại tab Changes trên IDE / giao diện Pull Request) trước khi tạo commit hoặc trước khi gửi yêu cầu review code là một thói quen cực kỳ giá trị. Ngoài việc kiểm tra xem comment/docstring đã khớp với logic mới hay chưa, bạn còn kịp thời phát hiện các dòng `console.log`/`print` bỏ quên, các import thừa, hoặc các thay đổi ngoài ý muốn.

## 16.6 Chú thích ở mức cao hơn sẽ dễ bảo trì hơn (Higher-level comments are easier to maintain)

Một suy nghĩ cuối cùng về việc bảo trì tài liệu: chú thích sẽ dễ bảo trì hơn nếu chúng nằm ở mức cao hơn và trừu tượng hơn so với mã nguồn. Những chú thích này không phản ánh các chi tiết vụn vặt của mã nguồn, nên chúng sẽ không bị ảnh hưởng bởi những thay đổi nhỏ trong code; chỉ những thay đổi về hành vi tổng thể mới làm ảnh hưởng đến các chú thích này. Tất nhiên, như đã thảo luận ở Chương 13, một số chú thích vẫn cần phải chi tiết và chính xác. Nhưng nhìn chung, những chú thích hữu ích nhất (những chú thích không chỉ đơn thuần lặp lại mã nguồn) cũng chính là những chú thích dễ bảo trì nhất.

---

## Bảng thuật ngữ chương (Chapter Glossary)

| Thuật ngữ (English) | Tiếng Việt | Giải thích ngắn gọn |
| :--- | :--- | :--- |
| **Strategic programming** | Lập trình chiến lược | Cách tiếp cận ưu tiên tạo ra thiết kế hệ thống sạch sẽ, tốt nhất có thể về lâu dài thay vì chỉ làm cho code chạy nhanh. |
| **Tactical programming** | Lập trình chiến thuật | Cách tiếp cận tập trung làm cho tính năng chạy hoặc sửa lỗi nhanh nhất có thể, chấp nhận chắp vá và làm tăng sự phức tạp của hệ thống. |
| **Refactoring** | Tái cấu trúc mã nguồn | Việc chỉnh sửa cấu trúc bên trong của mã nguồn để làm thiết kế sạch hơn, dễ hiểu hơn mà không làm thay đổi hành vi bên ngoài. |
| **Investment mindset** | Tư duy đầu tư | Sẵn sàng bỏ thêm một ít thời gian hiện tại để cải thiện thiết kế và dọn dẹp mã nguồn nhằm tăng tốc độ phát triển trong tương lai. |
| **Interface comment** | Chú thích giao diện | Chú thích mô tả cách sử dụng, hợp đồng (contract) và hành vi bên ngoài của một lớp hoặc phương thức. |
| **Implementation comment** | Chú thích cài đặt | Chú thích giải thích *cách thức* và *lý do* bên trong phần thân mã nguồn thực hiện công việc của nó. |
| **Narrowest scope** | Phạm vi hẹp nhất | Đặt chú thích ngay sát đoạn mã cụ thể mà nó mô tả thay vì dồn hết lên đầu hàm hoặc đặt ở tệp khác. |
| **Commit log / Commit message** | Nhật ký commit / Thông điệp commit | Lịch sử ghi chú các lần thay đổi mã nguồn trong hệ thống quản lý phiên bản (như Git); không nên dùng để thay thế chú thích trong code. |
| **Pre-commit scan (check the diffs)** | Rà soát trước khi commit (kiểm tra bản diff) | Thói quen xem lại toàn bộ thay đổi trước khi commit để đảm bảo chú thích đã được cập nhật và không để sót mã gỡ lỗi hay `TODO`. |
