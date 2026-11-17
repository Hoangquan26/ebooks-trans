# CHƯƠNG 17: Tính nhất quán (Chapter 17: Consistency)

Tính nhất quán là một công cụ mạnh mẽ giúp giảm độ phức tạp của hệ thống và làm cho hành vi của nó trở nên rõ ràng hơn. Nếu một hệ thống nhất quán, điều đó có nghĩa là những thứ giống nhau được thực hiện theo những cách giống nhau, và những thứ khác nhau được thực hiện theo những cách khác nhau. Tính nhất quán tạo ra *đòn bẩy nhận thức (cognitive leverage)*: một khi bạn đã học được cách một việc nào đó được thực hiện ở một nơi, bạn có thể sử dụng kiến thức đó để hiểu ngay lập tức các nơi khác cũng sử dụng cùng cách tiếp cận đó. Nếu một hệ thống không được triển khai theo cách nhất quán, các lập trình viên phải tìm hiểu từng tình huống một cách riêng lẻ. Điều này sẽ tốn nhiều thời gian hơn.

Tính nhất quán giúp giảm thiểu sai sót. Nếu một hệ thống không nhất quán, hai tình huống có thể trông giống hệt nhau trong khi thực tế chúng lại khác nhau. Một lập trình viên có thể nhìn thấy một khuôn mẫu quen thuộc và đưa ra những giả định sai lầm dựa trên những lần gặp khuôn mẫu đó trước đây. Ngược lại, nếu hệ thống nhất quán, những giả định được đưa ra dựa trên các tình huống trông quen thuộc sẽ an toàn. Tính nhất quán cho phép các lập trình viên làm việc nhanh hơn với ít sai sót hơn.

## 17.1 Ví dụ về tính nhất quán (Examples of consistency)

Tính nhất quán có thể được áp dụng ở nhiều cấp độ khác nhau trong một hệ thống; dưới đây là một vài ví dụ.

**Tên gọi (Names).** Chương 14 đã thảo luận về lợi ích của việc sử dụng tên gọi một cách nhất quán.

**Phong cách viết mã (Coding style).** Ngày nay, việc các tổ chức phát triển phần mềm có các *hướng dẫn phong cách (style guides)* nhằm giới hạn cấu trúc chương trình vượt ra ngoài các quy tắc do trình biên dịch áp đặt đã trở nên phổ biến. Các hướng dẫn phong cách hiện đại đề cập đến nhiều vấn đề, chẳng hạn như thụt lề (indentation), vị trí đặt dấu ngoặc nhọn (curly-brace placement), thứ tự khai báo, đặt tên, viết chú thích (commenting), và hạn chế đối với các tính năng ngôn ngữ bị coi là nguy hiểm. Các quy tắc phong cách giúp mã nguồn dễ đọc hơn và có thể giảm bớt một số loại lỗi nhất định.

**Giao diện (Interfaces).** Một giao diện có nhiều bản triển khai (implementations) là một ví dụ khác về tính nhất quán. Một khi bạn hiểu một bản triển khai của giao diện đó, bất kỳ bản triển khai nào khác cũng trở nên dễ hiểu hơn vì bạn đã biết trước những tính năng mà nó phải cung cấp.

**Mẫu thiết kế (Design patterns).** Mẫu thiết kế là những giải pháp được chấp nhận rộng rãi cho một số vấn đề phổ biến, chẳng hạn như cách tiếp cận *Model-View-Controller (MVC)* trong thiết kế giao diện người dùng. Nếu bạn có thể sử dụng một mẫu thiết kế hiện có để giải quyết vấn đề, việc triển khai sẽ diễn ra nhanh hơn, có nhiều khả năng hoạt động đúng hơn, và mã nguồn của bạn sẽ rõ ràng hơn đối với người đọc. Các mẫu thiết kế sẽ được thảo luận chi tiết hơn trong Mục 19.5.

**Bất biến (Invariants).** *Bất biến* là một thuộc tính của một biến hoặc một cấu trúc dữ liệu mà luôn luôn đúng. Ví dụ, một cấu trúc dữ liệu lưu trữ các dòng văn bản có thể áp đặt một bất biến rằng mỗi dòng đều được kết thúc bằng một ký tự xuống dòng (newline character). Các bất biến làm giảm số lượng các trường hợp đặc biệt phải xem xét trong mã nguồn và giúp việc suy luận về hành vi của mã nguồn trở nên dễ dàng hơn.

> 💡 **Ví dụ bổ sung (không có trong sách):**
> Giả sử trong một hệ thống thương mại điện tử, tất cả các hàm truy vấn thực thể theo ID đều tuân theo cùng một quy ước: trả về `Optional<T>` (hoặc ném ngoại lệ `NotFoundException` nhất quán). Nếu `getUserById(id)` trả về `null` khi không tìm thấy, còn `getOrderById(id)` lại ném `OrderNotFoundException`, và `getProductById(id)` trả về một đối tượng rỗng, thì lập trình viên sẽ rất dễ quên kiểm tra `null` hoặc quên bắt ngoại lệ khi gọi một hàm mới. Khi tất cả cùng tuân theo một quy ước giao diện và xử lý lỗi nhất quán, kiến thức về một hàm có thể áp dụng ngay cho các hàm còn lại.

## 17.2 Đảm bảo tính nhất quán (Ensuring consistency)

Tính nhất quán rất khó duy trì, đặc biệt là khi có nhiều người cùng làm việc trên một dự án trong thời gian dài. Mọi người ở nhóm này có thể không biết về các quy ước đã được thiết lập ở nhóm khác. Người mới vào chưa biết các quy tắc, nên họ vô tình vi phạm các quy ước hiện có và tạo ra những quy ước mới xung đột với những cái cũ. Dưới đây là một vài lời khuyên để thiết lập và duy trì tính nhất quán:

**Ghi thành tài liệu (Document).** Hãy tạo một tài liệu liệt kê những quy ước tổng thể quan trọng nhất, chẳng hạn như hướng dẫn phong cách viết mã. Đặt tài liệu đó ở nơi mà các lập trình viên dễ nhìn thấy, ví dụ như một vị trí nổi bật trên Wiki của dự án. Khuyến khích những người mới tham gia nhóm đọc tài liệu này, và khuyến khích những người hiện tại thỉnh thoảng xem lại. Một số hướng dẫn phong cách từ nhiều tổ chức khác nhau đã được công bố trên Web; hãy cân nhắc bắt đầu từ một trong số đó.

Đối với những quy ước mang tính cục bộ hơn, chẳng hạn như các bất biến, hãy tìm một vị trí thích hợp ngay trong mã nguồn để ghi chú lại chúng. Nếu bạn không viết các quy ước ra, rất ít khả năng người khác sẽ tuân theo chúng.

**Thực thi tự động (Enforce).** Ngay cả khi có tài liệu tốt, các lập trình viên vẫn khó có thể nhớ hết tất cả các quy ước. Cách tốt nhất để thực thi các quy ước là viết một công cụ kiểm tra các vi phạm, và đảm bảo rằng mã nguồn không thể được commit vào kho lưu trữ (repository) trừ khi nó vượt qua được trình kiểm tra đó. Các trình kiểm tra tự động hoạt động đặc biệt hiệu quả đối với các quy ước cú pháp ở mức thấp.

Một trong những dự án gần đây của tôi từng gặp vấn đề với các ký tự kết thúc dòng (line termination characters). Một số lập trình viên làm việc trên Unix, nơi các dòng được kết thúc bằng ký tự xuống dòng (`\n` — newline); những người khác làm việc trên Windows, nơi các dòng thường được kết thúc bằng ký tự về đầu dòng theo sau bởi ký tự xuống dòng (`\r\n` — carriage-return followed by newline). Nếu một lập trình viên trên hệ thống này thực hiện một chỉnh sửa nhỏ đối với một tệp trước đó được chỉnh sửa trên hệ thống kia, trình soạn thảo đôi khi sẽ thay thế toàn bộ các ký tự kết thúc dòng bằng ký tự phù hợp với hệ thống của họ. Điều này tạo ra cảm giác như mọi dòng trong tệp đều đã bị sửa đổi, khiến việc theo dõi các thay đổi thực sự có ý nghĩa trở nên rất khó khăn. Chúng tôi đã thiết lập một quy ước rằng các tệp chỉ được chứa ký tự xuống dòng (`\n`), nhưng rất khó để đảm bảo mọi công cụ mà mọi lập trình viên sử dụng đều tuân thủ quy ước đó. Mỗi khi có một lập trình viên mới tham gia dự án, chúng tôi lại gặp một loạt vấn đề về ký tự kết thúc dòng trong thời gian người đó làm quen với quy ước.

Cuối cùng, chúng tôi đã giải quyết vấn đề này bằng cách viết một đoạn script ngắn được chạy tự động trước khi các thay đổi được commit vào kho mã nguồn. Đoạn script kiểm tra tất cả các tệp đã được chỉnh sửa và hủy bỏ lệnh commit nếu bất kỳ tệp nào chứa ký tự về đầu dòng (carriage return). Script này cũng có thể được chạy thủ công để sửa chữa các tệp bị hỏng bằng cách thay thế chuỗi carriage-return/newline bằng newline. Điều này ngay lập tức loại bỏ hoàn toàn vấn đề, đồng thời giúp đào tạo các lập trình viên mới.

*Đánh giá mã nguồn (code reviews)* cũng mang lại một cơ hội khác để thực thi các quy ước và giáo dục các lập trình viên mới về các quy ước đó. Người đánh giá mã nguồn càng kỹ tính (nit-picky) bao nhiêu, mọi người trong nhóm sẽ học được các quy ước nhanh bấy nhiêu, và mã nguồn sẽ càng sạch đẹp hơn.

**Nhập gia tùy tục (When in Rome ...).** Quy ước quan trọng nhất trong tất cả là mọi lập trình viên nên tuân theo câu ngạn ngữ cổ: *"Nhập gia tùy tục" (When in Rome, do as the Romans do).* Khi làm việc trong một tệp mới, hãy quan sát xung quanh để xem mã nguồn hiện có được cấu trúc như thế nào. Tất cả các biến và phương thức `public` có được khai báo trước các biến và phương thức `private` không? Các phương thức có được sắp xếp theo thứ tự bảng chữ cái không? Tên biến sử dụng kiểu *"lạc đà" (camel case)*, như `firstServerName`, hay *"rắn bò" (snake case)*, như `first_server_name`? Khi bạn thấy bất kỳ điều gì có vẻ giống một quy ước, hãy tuân theo nó. Khi đưa ra một quyết định thiết kế, hãy tự hỏi liệu một quyết định tương tự có khả năng đã được thực hiện ở đâu đó khác trong dự án hay chưa; nếu có, hãy tìm một ví dụ hiện có và sử dụng cùng cách tiếp cận đó trong mã nguồn mới của bạn.

**Đừng thay đổi các quy ước hiện có (Don't change existing conventions).** Hãy kìm nén sự thôi thúc muốn "cải tiến" các quy ước hiện có. Việc có một "ý tưởng hay hơn" không phải là cái cớ chính đáng để tạo ra sự thiếu nhất quán. Ý tưởng mới của bạn thực sự có thể tốt hơn, nhưng giá trị của sự nhất quán so với sự thiếu nhất quán hầu như luôn lớn hơn giá trị chênh lệch giữa cách tiếp cận này với cách tiếp cận kia. Trước khi đưa ra một hành vi không nhất quán, hãy tự hỏi mình hai câu hỏi. Thứ nhất, bạn có thông tin mới quan trọng nào biện minh cho cách tiếp cận của mình mà trước đây chưa có khi quy ước cũ được thiết lập không? Thứ hai, cách tiếp cận mới có tốt hơn nhiều đến mức đáng để bỏ thời gian cập nhật lại toàn bộ các chỗ sử dụng cũ hay không? Nếu tổ chức của bạn đồng ý rằng câu trả lời cho cả hai câu hỏi đều là "có", thì hãy tiến hành nâng cấp; và khi bạn làm xong, không nên còn bất kỳ dấu vết nào của quy ước cũ. Tuy nhiên, bạn vẫn đối mặt với rủi ro là các lập trình viên khác không biết về quy ước mới, nên họ có thể đưa cách tiếp cận cũ quay trở lại trong tương lai. Nhìn chung, việc xem xét lại các quy ước đã được thiết lập từ lâu hiếm khi là một cách sử dụng thời gian hiệu quả của lập trình viên.

> 💡 **Ví dụ bổ sung (không có trong sách):**
> Trong các dự án hiện đại, việc thực thi tự động thường được thiết lập qua:
> - **Linter / Formatter** (như `eslint`, `prettier`, `ruff`, `gofmt`, `checkstyle`) để kiểm tra định dạng, quy tắc đặt tên và các mẫu mã nguy hiểm.
> - **Pre-commit hooks** (như `husky`, `pre-commit`) hoặc **CI/CD checks** để chặn mọi Pull Request vi phạm quy ước hoặc chứa lỗi định dạng trước khi được hợp nhất vào nhánh chính.
> - **`.editorconfig`** để thống nhất ký tự kết thúc dòng (`lf`) và kích thước thụt lề trên mọi IDE khác nhau.

## 17.3 Đi quá xa (Taking it too far)

Tính nhất quán không chỉ có nghĩa là những thứ giống nhau nên được làm theo những cách giống nhau, mà còn có nghĩa là *những thứ khác nhau nên được làm theo những cách khác nhau*. Nếu bạn trở nên quá mức cuồng tín về tính nhất quán và cố ép những thứ vốn khác nhau vào cùng một cách tiếp cận — chẳng hạn như dùng cùng một tên biến cho những thứ thực sự khác nhau, hoặc áp dụng một mẫu thiết kế hiện có cho một tác vụ không phù hợp với mẫu đó — bạn sẽ tạo ra sự phức tạp và nhầm lẫn. Tính nhất quán chỉ mang lại lợi ích khi các lập trình viên có thể tin tưởng rằng *"nếu nó trông giống một cái x, thì nó thực sự là một cái x."*

> 📝 **Giải thích thêm (không có trong sách):**
> Đây là lý do tại sao việc áp dụng rập khuôn một mẫu thiết kế (ví dụ: ép mọi logic đơn giản phải đi qua đầy đủ các lớp Controller → Service → Repository → Factory dù không cần thiết) hoặc đặt tên giống nhau cho các khái niệm có ngữ nghĩa khác nhau (ví dụ: dùng `userId` cho cả ID nội bộ kiểu số nguyên lẫn mã định danh công khai kiểu UUID ở các dịch vụ khác nhau) lại gây hại. Sự giống nhau giả tạo đánh lừa người đọc và làm tăng nguy cơ lỗi ngầm.

## 17.4 Kết luận (Conclusion)

Tính nhất quán là một ví dụ khác của *tư duy đầu tư (investment mindset)*. Sẽ tốn thêm một chút công sức để đảm bảo tính nhất quán: công sức để thống nhất các quy ước, công sức để tạo các trình kiểm tra tự động, công sức để tìm kiếm các tình huống tương tự nhằm mô phỏng theo trong mã nguồn mới, và công sức trong các buổi đánh giá mã nguồn để hướng dẫn cả nhóm. Phần thưởng cho khoản đầu tư này là mã nguồn của bạn sẽ trở nên rõ ràng hơn. Các lập trình viên sẽ có thể hiểu hành vi của mã nguồn nhanh hơn và chính xác hơn, và điều này sẽ cho phép họ làm việc nhanh hơn với ít lỗi hơn.

---

## Bảng thuật ngữ chương (Chapter Glossary)

| Thuật ngữ (English) | Tiếng Việt | Giải thích ngắn gọn |
| :--- | :--- | :--- |
| **Consistency** | Tính nhất quán | Nguyên tắc những thứ giống nhau được làm theo cách giống nhau, và những thứ khác nhau được làm theo cách khác nhau. |
| **Cognitive leverage** | Đòn bẩy nhận thức | Khả năng tận dụng kiến thức đã học ở một nơi để hiểu ngay lập tức các phần khác của hệ thống. |
| **Coding style / Style guide** | Phong cách viết mã / Hướng dẫn phong cách | Bộ quy tắc chuẩn hóa hình thức mã nguồn (thụt lề, ngoặc nhọn, đặt tên, chú thích...). |
| **Invariant** | Bất biến | Thuộc tính của biến hoặc cấu trúc dữ liệu luôn luôn đúng, giúp giảm các trường hợp đặc biệt cần xử lý. |
| **Design pattern** | Mẫu thiết kế | Giải pháp chuẩn được công nhận rộng rãi cho các vấn đề thiết kế phần mềm phổ biến. |
| **Line termination character** | Ký tự kết thúc dòng | Ký tự đánh dấu hết dòng trong tệp văn bản (ví dụ `\n` trên Unix và `\r\n` trên Windows). |
| **Code review** | Đánh giá mã nguồn | Quá trình đồng nghiệp kiểm tra mã nguồn trước khi hợp nhất nhằm phát hiện lỗi và duy trì quy ước chung. |
| **Investment mindset** | Tư duy đầu tư | Quan điểm sẵn sàng bỏ thêm công sức ban đầu để đạt được mã nguồn rõ ràng, ít lỗi và dễ bảo trì về sau. |
