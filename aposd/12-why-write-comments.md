# CHƯƠNG 12: Tại sao phải viết chú thích? Bốn lời bào chữa (Chapter 12: Why Write Comments? The Four Excuses)

Tài liệu ngay trong mã nguồn (*in-code documentation*) đóng một vai trò tối quan trọng trong thiết kế phần mềm. Chú thích (*comments*) là yếu tố thiết yếu giúp các lập trình viên hiểu hệ thống và làm việc hiệu quả, nhưng vai trò của chú thích không chỉ dừng lại ở đó. Tài liệu cũng đóng vai trò quan trọng trong sự trừu tượng hóa (*abstraction*); nếu không có chú thích, bạn không thể che giấu sự phức tạp. Cuối cùng, quá trình viết chú thích, nếu được thực hiện đúng cách, sẽ thực sự cải thiện thiết kế của hệ thống. Ngược lại, một thiết kế phần mềm tốt sẽ mất đi phần lớn giá trị nếu tài liệu của nó nghèo nàn.

Đáng tiếc là quan điểm này không được tất cả mọi người đồng tình. Một tỷ lệ đáng kể mã nguồn sản xuất (*production code*) hầu như không có bất kỳ chú thích nào. Nhiều lập trình viên cho rằng viết chú thích là lãng phí thời gian; những người khác nhìn thấy giá trị của chú thích, nhưng chẳng hiểu sao không bao giờ sắp xếp được thời gian để viết chúng. May mắn thay, nhiều nhóm phát triển nhận ra giá trị của tài liệu, và có cảm giác như số lượng các nhóm như vậy đang tăng dần lên. Tuy nhiên, ngay cả trong những nhóm khuyến khích viết tài liệu, chú thích thường bị coi là một công việc nhàm chán, cực nhọc (*drudge work*) và nhiều lập trình viên không biết cách viết chúng, nên tài liệu tạo ra thường chỉ ở mức trung bình. Tài liệu kém chất lượng tạo ra một lực cản khổng lồ và không cần thiết đối với quá trình phát triển phần mềm.

Trong chương này, tôi sẽ thảo luận về những lời bào chữa mà các lập trình viên dùng để né tránh việc viết chú thích, và những lý do tại sao chú thích thực sự quan trọng. Chương 13 sau đó sẽ mô tả cách viết những chú thích tốt, và vài chương tiếp theo nữa sẽ bàn về các vấn đề liên quan như cách đặt tên biến và cách sử dụng tài liệu để cải thiện thiết kế hệ thống. Tôi hy vọng những chương này sẽ thuyết phục bạn về ba điều: chú thích tốt có thể tạo ra sự khác biệt lớn đối với chất lượng tổng thể của phần mềm; việc viết chú thích tốt không hề khó; và (điều này có thể khó tin) viết chú thích thực sự có thể mang lại niềm vui.

Khi các lập trình viên không viết chú thích, họ thường biện minh cho hành vi của mình bằng một hoặc nhiều lời bào chữa sau đây:

- “Mã nguồn tốt tự giải thích chính nó.”
- “Tôi không có thời gian để viết chú thích.”
- “Chú thích bị lỗi thời và trở nên gây hiểu nhầm.”
- “Những chú thích tôi từng thấy đều vô giá trị; viết làm gì cho mất công?”

Trong các phần dưới đây, tôi sẽ lần lượt giải quyết từng lời bào chữa này.

## 12.1 Mã nguồn tốt tự giải thích chính nó (Good code is self-documenting)

Một số người tin rằng nếu mã nguồn được viết tốt, nó sẽ hiển nhiên đến mức không cần đến bất kỳ chú thích nào. Đây là một huyền thoại ngọt ngào, giống như tin đồn rằng kem tốt cho sức khỏe vậy: chúng ta rất muốn tin vào điều đó! Đáng tiếc là nó đơn giản không phải sự thật. Chắc chắn có những điều bạn có thể làm khi viết mã để giảm bớt nhu cầu về chú thích, chẳng hạn như chọn tên biến tốt (xem Chương 14). Tuy nhiên, vẫn có một lượng lớn thông tin thiết kế không thể biểu diễn được trong mã nguồn. Ví dụ, chỉ một phần nhỏ giao diện (*interface*) của một lớp, như chữ ký các phương thức (*method signatures*) của nó, có thể được đặc tả một cách hình thức trong mã nguồn. Những khía cạnh phi hình thức của một giao diện, chẳng hạn như mô tả ở mức cao về việc mỗi phương thức làm gì hoặc ý nghĩa của kết quả trả về, chỉ có thể được mô tả bằng chú thích. Có rất nhiều ví dụ khác về những thứ không thể diễn tả trong mã nguồn, chẳng hạn như lý do đằng sau một quyết định thiết kế cụ thể (*rationale*), hoặc các điều kiện mà tại đó việc gọi một phương thức cụ thể là hợp lý.

Một số lập trình viên lập luận rằng nếu người khác muốn biết một phương thức làm gì, họ chỉ cần đọc mã nguồn của phương thức đó: điều này sẽ chính xác hơn bất kỳ chú thích nào. Có khả năng người đọc có thể suy luận ra giao diện trừu tượng của phương thức bằng cách đọc mã của nó, nhưng việc đó sẽ rất tốn thời gian và đau đớn. Ngoài ra, nếu bạn viết mã với kỳ vọng rằng người dùng sẽ đọc phần cài đặt (*implementation*) của phương thức, bạn sẽ cố gắng làm cho mỗi phương thức ngắn nhất có thể để dễ đọc. Nếu phương thức làm bất cứ điều gì không tầm thường, bạn sẽ chia nhỏ nó thành nhiều phương thức con. Điều này sẽ dẫn đến một số lượng lớn các phương thức nông (*shallow methods*). Hơn nữa, việc đó thực sự không làm cho mã nguồn dễ đọc hơn: để hiểu hành vi của phương thức ở tầng cao nhất, người đọc có lẽ sẽ phải hiểu hành vi của tất cả các phương thức lồng nhau bên trong. Đối với các hệ thống lớn, việc yêu cầu người dùng đọc mã nguồn để tìm hiểu hành vi là hoàn toàn không thực tế.

Hơn thế nữa, chú thích là nền tảng của sự trừu tượng hóa. Hãy nhớ lại từ Chương 4 rằng mục tiêu của sự trừu tượng hóa là che giấu sự phức tạp: một sự trừu tượng hóa (*abstraction*) là một góc nhìn đơn giản hóa về một thực thể, giữ lại những thông tin thiết yếu nhưng lược bỏ các chi tiết có thể bỏ qua một cách an toàn. Nếu người dùng bắt buộc phải đọc mã nguồn của một phương thức để sử dụng nó, thì không hề có sự trừu tượng hóa nào cả: toàn bộ sự phức tạp của phương thức đều bị phơi bày.

Nếu không có chú thích, sự trừu tượng hóa duy nhất của một phương thức chỉ là phần khai báo (*declaration*) của nó, thứ chỉ định tên phương thức cũng như tên và kiểu dữ liệu của các tham số và kết quả trả về. Phần khai báo thiếu quá nhiều thông tin thiết yếu để có thể tự mình cung cấp một sự trừu tượng hóa hữu ích. Ví dụ, một phương thức để trích xuất một chuỗi con (*substring*) có thể có hai tham số, `start` và `end`, chỉ định khoảng các ký tự cần trích xuất. Chỉ từ phần khai báo, không thể biết được chuỗi con trích xuất có bao gồm ký tự tại vị trí `end` hay không, hoặc điều gì sẽ xảy ra nếu `start > end`. Chú thích cho phép chúng ta nắm bắt những thông tin bổ sung mà người gọi (*callers*) cần, qua đó hoàn thiện góc nhìn đơn giản hóa trong khi vẫn che giấu các chi tiết cài đặt. Một điều cũng rất quan trọng là chú thích được viết bằng ngôn ngữ tự nhiên của con người như tiếng Anh; điều này khiến chúng kém chính xác hơn mã nguồn, nhưng lại mang đến sức mạnh biểu đạt lớn hơn, nhờ đó chúng ta có thể tạo ra những mô tả đơn giản và trực quan. Nếu bạn muốn sử dụng sự trừu tượng hóa để che giấu sự phức tạp, chú thích là thứ thiết yếu.

> 💡 **Ví dụ bổ sung (không có trong sách):**
> Hãy xem xét hai khai báo hàm Java sau đây không có chú thích:
> ```java
> public List<Order> getOrders(LocalDate start, LocalDate end, boolean includeCancelled);
> ```
> Chỉ nhìn vào chữ ký hàm (dù tên hàm và tên biến rất rõ ràng), người gọi vẫn không biết:
> 1. Khoảng thời gian `[start, end]` là đóng (bao gồm cả ngày `end`) hay nửa mở `[start, end)`?
> 2. Múi giờ nào được dùng để đối chiếu `LocalDate` (UTC hay múi giờ của người dùng)?
> 3. Nếu `start` hoặc `end` là `null` thì sao? Ném ra ngoại lệ hay coi như không giới hạn?
> 4. Danh sách trả về có được sắp xếp không, và theo thứ tự nào?
>
> Một vài dòng chú thích giao diện ngắn gọn sẽ giải đáp toàn bộ những câu hỏi đó mà không bắt đồng nghiệp phải mở mã nguồn (và các câu truy vấn SQL bên dưới) ra đọc.

## 12.2 Tôi không có thời gian để viết chú thích (I don’t have time to write comments)

Thật dễ nảy sinh tâm lý xếp việc viết chú thích ở mức ưu tiên thấp hơn các nhiệm vụ phát triển khác. Khi đứng trước lựa chọn giữa việc thêm một tính năng mới và viết tài liệu cho một tính năng hiện có, việc chọn tính năng mới có vẻ rất hợp lý. Tuy nhiên, các dự án phần mềm hầu như luôn chịu áp lực về thời gian, và sẽ luôn có những thứ có vẻ ưu tiên cao hơn việc viết chú thích. Do đó, nếu bạn cho phép tài liệu bị hạ mức ưu tiên, bạn sẽ kết thúc với việc chẳng có chút tài liệu nào.

Luận điểm phản bác lại lời bào chữa này chính là tư duy đầu tư (*investment mindset*) đã được thảo luận ở trang 15. Nếu bạn muốn có một cấu trúc phần mềm sạch sẽ, thứ sẽ cho phép bạn làm việc hiệu quả về lâu dài, thì bạn phải dành thêm một chút thời gian ngay từ đầu để tạo ra cấu trúc đó. Những chú thích tốt tạo ra sự khác biệt khổng lồ đối với khả năng bảo trì (*maintainability*) của phần mềm, vì vậy công sức bỏ ra cho chúng sẽ nhanh chóng được đền đáp.

Hơn nữa, việc viết chú thích không nhất thiết phải tốn nhiều thời gian. Hãy tự hỏi bản thân xem bạn dành bao nhiêu phần trăm thời gian phát triển để gõ mã nguồn (tách biệt với việc thiết kế, biên dịch, kiểm thử, v.v.), giả sử không tính bất kỳ chú thích nào; tôi nghi ngờ câu trả lời sẽ vượt quá 10%. Bây giờ hãy giả sử rằng bạn dành thời gian gõ chú thích nhiều ngang bằng với thời gian gõ mã nguồn; đây hẳn là một giới hạn trên an toàn. Với những giả định này, việc viết những chú thích tốt sẽ không làm tăng thêm quá 10% thời gian phát triển của bạn. Lợi ích của việc có tài liệu tốt sẽ nhanh chóng bù đắp chi phí này.

Ngoài ra, nhiều chú thích quan trọng nhất lại là những chú thích liên quan đến sự trừu tượng hóa, chẳng hạn như tài liệu ở tầng cao nhất cho các lớp và phương thức. Chương 15 sẽ lập luận rằng những chú thích này nên được viết như một phần của quá trình thiết kế, và chính hành động viết tài liệu đóng vai trò là một công cụ thiết kế quan trọng giúp cải thiện thiết kế tổng thể. Những chú thích này tự hoàn vốn ngay lập tức.

> 📝 **Giải thích thêm (không có trong sách):**
> Phép tính của Ousterhout rất thực tế: thời gian lập trình viên thực sự gõ phím để viết code chỉ chiếm khoảng ~10% tổng thời gian làm việc (90% còn lại là đọc code cũ, suy nghĩ thiết kế, gỡ lỗi, viết và chạy test, họp hành, review code). Do đó, ngay cả khi bạn tốn gấp đôi thời gian gõ phím để viết thêm comment, tổng thời gian phát triển cũng chỉ tăng thêm tối đa 10% — trong khi thời gian đọc hiểu và bảo trì về sau có thể giảm đi hàng chục phần trăm.

## 12.3 Chú thích bị lỗi thời và trở nên gây hiểu nhầm (Comments get out of date and become misleading)

Đôi khi chú thích thực sự bị lỗi thời, nhưng trong thực tế điều này không nhất thiết phải trở thành một vấn đề lớn. Việc giữ cho tài liệu luôn cập nhật không đòi hỏi một nỗ lực khổng lồ. Những thay đổi lớn đối với tài liệu chỉ cần thiết khi có những thay đổi lớn đối với mã nguồn, và bản thân việc thay đổi mã nguồn sẽ tốn nhiều thời gian hơn việc thay đổi tài liệu. Chương 16 thảo luận về cách tổ chức tài liệu sao cho việc cập nhật nó sau khi sửa đổi mã nguồn trở nên dễ dàng nhất có thể (những ý tưởng cốt lõi là tránh lặp lại tài liệu và đặt tài liệu gần với đoạn mã tương ứng). Hoạt động đánh giá mã (*code reviews*) cũng cung cấp một cơ chế tuyệt vời để phát hiện và sửa chữa những chú thích đã cũ (*stale comments*).

> 💡 **Ví dụ bổ sung (không có trong sách):**
> Để tránh comment bị lỗi thời khi code thay đổi:
> - **Không lặp lại thông tin giữa nhiều nơi:** Nếu một quy tắc nghiệp vụ được mô tả ở cả đầu lớp, đầu hàm, và bên trong hàm, khi sửa code bạn rất dễ quên sửa 1 trong 3 chỗ. Hãy đặt mô tả hành vi chung ở chữ ký hàm/lớp, còn chi tiết cài đặt ở ngay bên cạnh dòng code xử lý.
> - **Xem việc cập nhật comment là tiêu chí bắt buộc trong Code Review:** Khi review một Pull Request sửa logic của hàm `calculateDiscount`, người review nên kiểm tra luôn phần docstring/comment của hàm đó xem còn khớp với hành vi mới hay không.

## 12.4 Tất cả những chú thích tôi từng thấy đều vô giá trị (All the comments I have seen are worthless)

Trong bốn lời bào chữa, đây có lẽ là lời bào chữa có cơ sở nhất. Mọi lập trình viên phần mềm đều từng thấy những chú thích không cung cấp bất kỳ thông tin hữu ích nào, và phần lớn tài liệu hiện có giỏi lắm cũng chỉ ở mức tạm được. May mắn thay, vấn đề này có thể giải quyết được; việc viết tài liệu vững chắc không hề khó, một khi bạn biết cách. Các chương tiếp theo sẽ vạch ra một khung hướng dẫn (*framework*) về cách viết tài liệu tốt và duy trì nó theo thời gian.

> 💡 **Ví dụ bổ sung (không có trong sách):**
> Một kiểu chú thích "vô giá trị" điển hình thường gặp là lặp lại y hệt những gì mã nguồn đã nói:
> ```python
> # Tăng biến đếm i thêm 1
> i += 1
> 
> # Khởi tạo danh sách người dùng rỗng
> users = []
> ```
> Khi lập trình viên chỉ từng gặp loại chú thích này, họ dễ kết luận rằng chú thích chỉ làm rác màn hình. Nhưng vấn đề không nằm ở việc viết chú thích, mà nằm ở việc *viết sai nội dung* cần chú thích (Chương 13 sẽ đi sâu vào vấn đề này).

## 12.5 Lợi ích của những chú thích được viết tốt (Benefits of well-written comments)

Bây giờ, sau khi đã thảo luận (và hy vọng là đã bác bỏ) các lập luận chống lại việc viết chú thích, hãy cùng xem xét những lợi ích mà bạn sẽ nhận được từ những chú thích tốt. Ý tưởng tổng thể đằng sau chú thích là nắm bắt những thông tin từng nằm trong tâm trí của người thiết kế nhưng không thể biểu diễn được trong mã nguồn. Thông tin này trải dài từ những chi tiết ở mức thấp, chẳng hạn như một điểm kỳ quặc của phần cứng (*hardware quirk*) dẫn đến một đoạn mã đặc biệt lắt léo, cho đến những khái niệm ở mức cao như lý do tồn tại (*rationale*) của một lớp. Khi các lập trình viên khác tiếp quản sau này để thực hiện các sửa đổi, những chú thích đó sẽ cho phép họ làm việc nhanh chóng và chính xác hơn. Nếu không có tài liệu, các lập trình viên tương lai sẽ phải tự suy luận lại hoặc đoán mò kiến thức ban đầu của người thiết kế; việc này sẽ tốn thêm thời gian, và có nguy cơ phát sinh lỗi (*bugs*) nếu lập trình viên mới hiểu sai ý định của người thiết kế ban đầu. Chú thích có giá trị ngay cả khi chính người thiết kế ban đầu là người thực hiện các thay đổi: nếu đã hơn vài tuần kể từ lần cuối bạn làm việc trên một đoạn mã, bạn sẽ quên mất nhiều chi tiết của thiết kế ban đầu.

Chương 2 đã mô tả ba cách mà sự phức tạp biểu hiện trong các hệ thống phần mềm:

- **Khuếch đại thay đổi (*Change amplification*):** một thay đổi có vẻ đơn giản lại đòi hỏi phải sửa đổi mã nguồn ở nhiều nơi.
- **Gánh nặng nhận thức (*Cognitive load*):** để thực hiện một thay đổi, lập trình viên phải tích lũy một lượng lớn thông tin.
- **Những điều chưa biết mà ta không hề hay biết (*Unknown unknowns*):** không rõ đoạn mã nào cần được sửa đổi, hoặc những thông tin nào phải được xem xét để thực hiện những sửa đổi đó.

Tài liệu tốt giúp giải quyết hai vấn đề cuối trong số này. Tài liệu có thể làm giảm gánh nặng nhận thức bằng cách cung cấp cho lập trình viên những thông tin họ cần để thực hiện thay đổi và giúp lập trình viên dễ dàng bỏ qua những thông tin không liên quan. Nếu không có tài liệu đầy đủ, lập trình viên có thể phải đọc một lượng lớn mã nguồn để tái dựng lại những gì từng nằm trong đầu người thiết kế. Tài liệu cũng có thể giảm thiểu những điều *unknown unknowns* bằng cách làm rõ cấu trúc của hệ thống, nhờ đó trở nên rõ ràng rằng thông tin nào và đoạn mã nào là liên quan cho bất kỳ thay đổi cụ thể nào.

Chương 2 đã chỉ ra rằng hai nguyên nhân chính của sự phức tạp là sự phụ thuộc (*dependencies*) và sự mù mờ (*obscurity*). Tài liệu tốt có thể làm rõ các sự phụ thuộc, và nó lấp đầy những khoảng trống để xóa bỏ sự mù mờ.

Vài chương tiếp theo sẽ chỉ cho bạn cách viết tài liệu tốt. Chúng cũng sẽ thảo luận cách tích hợp việc viết tài liệu vào quy trình thiết kế sao cho nó cải thiện thiết kế phần mềm của bạn.

## Bảng thuật ngữ chương (Chapter Glossary)

| Thuật ngữ (English) | Tiếng Việt | Giải thích ngắn gọn |
| :--- | :--- | :--- |
| In-code documentation | Tài liệu ngay trong mã nguồn | Chú thích, docstring được viết trực tiếp bên trong tệp mã nguồn để giải thích giao diện, thiết kế và các chi tiết ẩn. |
| Self-documenting code | Mã nguồn tự giải thích | Quan niệm cho rằng mã viết rõ ràng, đặt tên tốt thì không cần chú thích (tác giả coi đây là một huyền thoại). |
| Abstraction | Sự trừu tượng hóa | Góc nhìn đơn giản hóa về một thực thể, giữ lại thông tin thiết yếu và che giấu chi tiết cài đặt phức tạp. |
| Method signature / Declaration | Chữ ký phương thức / Khai báo | Phần định nghĩa tên phương thức, tham số và kiểu trả về, không bao gồm phần thân cài đặt. |
| Shallow methods | Các phương thức nông | Các phương thức quá nhỏ, tách vụn chỉ để dễ đọc, dẫn đến việc người dùng phải đọc nhiều phương thức lồng nhau để hiểu hành vi. |
| Investment mindset | Tư duy đầu tư | Quan điểm chấp nhận bỏ thêm một chút thời gian ban đầu cho thiết kế và tài liệu để làm việc nhanh và ít lỗi hơn về lâu dài. |
| Stale comments | Chú thích cũ / lỗi thời | Chú thích không còn phản ánh đúng hành vi hiện tại của mã nguồn sau khi mã đã bị sửa đổi. |
| Change amplification | Khuếch đại thay đổi | Hiện tượng một thay đổi nhỏ đòi hỏi phải sửa đổi mã ở nhiều vị trí khác nhau. |
| Cognitive load | Gánh nặng nhận thức | Lượng thông tin và kiến thức mà lập trình viên phải ghi nhớ, tích lũy trong đầu để có thể sửa đổi mã an toàn. |
| Unknown unknowns | Những điều chưa biết mà không hay biết | Tình trạng không biết mình cần phải biết gì hoặc phải sửa đoạn mã nào khi thực hiện thay đổi. |
| Obscurity | Sự mù mờ | Nguyên nhân gây phức tạp khi thông tin quan trọng không hiển nhiên hoặc khó tìm thấy trong mã nguồn. |
