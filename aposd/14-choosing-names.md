# CHƯƠNG 14: Chọn Tên Gọi (Chapter 14: Choosing Names)

Việc chọn tên cho biến, phương thức và các thực thể khác là một trong những khía cạnh bị đánh giá thấp nhất của thiết kế phần mềm. Tên gọi tốt là một dạng tài liệu: chúng giúp mã nguồn dễ hiểu hơn. Chúng giảm bớt nhu cầu phải có các tài liệu khác và giúp việc phát hiện lỗi trở nên dễ dàng hơn. Ngược lại, việc lựa chọn tên gọi kém làm tăng độ phức tạp của mã nguồn, đồng thời tạo ra sự mơ hồ và những hiểu lầm có thể dẫn đến lỗi (bug). Việc chọn tên là một ví dụ cho nguyên lý rằng độ phức tạp mang tính tích lũy dần. Chọn một cái tên tầm thường cho một biến cụ thể, thay vì cái tên tốt nhất có thể, có lẽ sẽ không ảnh hưởng nhiều đến độ phức tạp tổng thể của cả hệ thống. Tuy nhiên, các hệ thống phần mềm có hàng nghìn biến; việc chọn tên gọi tốt cho tất cả những biến đó sẽ tạo ra tác động đáng kể đến độ phức tạp và khả năng quản lý hệ thống.

## 14.1 Ví dụ: tên gọi tồi gây ra lỗi (Example: bad names cause bugs)

Đôi khi chỉ một biến được đặt tên kém cũng có thể gây ra những hậu quả nghiêm trọng. Lỗi khó sửa nhất mà tôi từng xử lý bắt nguồn từ một lựa chọn tên gọi tồi. Vào cuối những năm 1980 và đầu những năm 1990, tôi và các nghiên cứu sinh của mình đã tạo ra một hệ điều hành phân tán có tên là Sprite. Đến một lúc nào đó, chúng tôi nhận thấy các tệp tin thỉnh thoảng bị mất dữ liệu: một trong các khối dữ liệu (data block) đột nhiên biến thành toàn số 0, mặc dù tệp tin không hề bị người dùng chỉnh sửa. Vấn đề này không xảy ra thường xuyên, nên việc truy tìm nguyên nhân vô cùng khó khăn. Một vài nghiên cứu sinh đã cố gắng tìm lỗi, nhưng họ không đạt được tiến triển nào và cuối cùng đành bỏ cuộc. Tuy nhiên, tôi coi bất kỳ lỗi nào chưa giải quyết được là một sự xúc phạm cá nhân không thể chấp nhận, vì vậy tôi quyết định tự mình truy tìm nó.

Phải mất sáu tháng, nhưng cuối cùng tôi cũng tìm ra và sửa được lỗi. Vấn đề thực chất khá đơn giản (giống như hầu hết các lỗi khác, một khi bạn đã tìm ra chúng). Mã nguồn của hệ thống tệp tin đã sử dụng tên biến `block` cho hai mục đích khác nhau. Trong một số tình huống, `block` dùng để chỉ số hiệu khối vật lý trên đĩa (physical block number on disk); trong các tình huống khác, `block` lại dùng để chỉ số hiệu khối logic bên trong một tệp tin (logical block number within a file). Thật không may, tại một điểm trong mã nguồn có một biến `block` đang chứa số hiệu khối logic, nhưng nó lại vô tình được sử dụng trong một ngữ cảnh cần đến số hiệu khối vật lý; kết quả là một khối không liên quan trên đĩa đã bị ghi đè toàn số 0.

Trong quá trình truy tìm lỗi, nhiều người, bao gồm cả tôi, đã đọc lướt qua đoạn mã bị lỗi đó, nhưng chúng tôi không bao giờ nhận ra vấn đề. Khi thấy biến `block` được dùng như một số hiệu khối vật lý, chúng tôi mặc nhiên giả định theo phản xạ rằng nó thực sự đang giữ một số hiệu khối vật lý. Phải mất một quá trình gắn mã theo dõi (instrumentation) kéo dài — thứ cuối cùng đã chỉ ra rằng sự hỏng dữ liệu chắc chắn đang xảy ra tại một câu lệnh cụ thể — trước khi tôi có thể vượt qua được "rào cản tâm lý" do cái tên đó tạo ra và kiểm tra xem giá trị của nó thực sự đến từ đâu. Nếu các tên biến khác nhau đã được sử dụng cho các loại khối khác nhau, chẳng hạn như `fileBlock` và `diskBlock`, thì lỗi đó khó có thể xảy ra; lập trình viên hẳn đã biết rằng `fileBlock` không thể được dùng trong tình huống đó.

Đáng tiếc là hầu hết các lập trình viên không dành nhiều thời gian để suy nghĩ về tên gọi. Họ có xu hướng dùng cái tên đầu tiên nảy ra trong đầu, miễn là nó khớp ở mức độ chấp nhận được với thứ mà nó đặt tên. Ví dụ, `block` là một cái tên khớp khá sát cho cả khối vật lý trên đĩa lẫn khối logic trong tệp; nó chắc chắn không phải là một cái tên tệ hại. Dù vậy, nó đã dẫn đến một sự tiêu tốn thời gian khổng lồ để truy tìm một lỗi tinh vi. Vì thế, bạn không nên bằng lòng với những cái tên chỉ ở mức "khớp tương đối". Hãy dành thêm một chút thời gian để chọn những cái tên tuyệt vời — chính xác, không mơ hồ và trực quan. Sự chú tâm thêm này sẽ nhanh chóng mang lại lợi ích xứng đáng, và theo thời gian bạn sẽ học được cách chọn tên tốt một cách nhanh chóng.

> 💡 **Ví dụ bổ sung (không có trong sách):** Trong các ứng dụng web hoặc tài chính, lỗi tương tự `block` rất hay xảy ra với tiền tệ hoặc đơn vị thời gian. Chẳng hạn, đặt tên biến là `price` hoặc `amount` mà không rõ là đồng Việt Nam (VND), đô la Mỹ (USD) hay đơn vị nhỏ nhất (cents/xu); hoặc đặt tên `timeout` mà không rõ là giây hay mili-giây. Nếu dùng `priceInCents`, `amountUsd`, hay `timeoutMs`, `timeoutSec`, lập trình viên sẽ lập tức nhận ra nếu vô tình truyền `timeoutSec` vào một hàm đang mong đợi `timeoutMs`.

## 14.2 Tạo nên một hình ảnh (Create an image)

Khi chọn một cái tên, mục tiêu là tạo ra một hình ảnh trong tâm trí người đọc về bản chất của thứ đang được đặt tên. Một cái tên tốt truyền tải rất nhiều thông tin về việc thực thể bên dưới *là gì*, và một điều cũng quan trọng không kém, là *nó không phải là gì*. Khi cân nhắc một cái tên cụ thể, hãy tự hỏi bản thân: "Nếu ai đó nhìn thấy cái tên này một cách độc lập, mà không nhìn thấy phần khai báo, tài liệu mô tả, hay bất kỳ đoạn mã nào sử dụng nó, họ có thể đoán chính xác đến mức nào thứ mà cái tên này đề cập tới? Liệu có cái tên nào khác vẽ nên một bức tranh rõ ràng hơn không?" Tất nhiên, lượng thông tin bạn có thể nhồi nhét vào một cái tên là có giới hạn; tên gọi sẽ trở nên cồng kềnh nếu chứa quá hai hoặc ba từ. Do đó, thách thức đặt ra là tìm ra chỉ vài từ mà vẫn nắm bắt được những khía cạnh quan trọng nhất của thực thể.

Tên gọi là một dạng trừu tượng hóa (abstraction): chúng cung cấp một cách tư duy đơn giản hóa về một thực thể phức tạp hơn ở bên dưới. Giống như các dạng trừu tượng hóa khác, những cái tên tốt nhất là những cái tên tập trung sự chú ý vào điều quan trọng nhất của thực thể bên dưới trong khi lược bỏ các chi tiết kém quan trọng hơn.

> 📝 **Giải thích thêm (không có trong sách):** Hãy hình dung cái tên như một tiêu đề báo ngắn gọn: nó không thể kể toàn bộ câu chuyện, nhưng chỉ cần đọc lướt qua là người đọc phải hình dung được ngay chủ đề chính và không hiểu nhầm sang chuyện khác. Ví dụ: `user` quá chung chung, nhưng `authenticatedUser` hoặc `pendingInvite` lập tức vẽ ra bối cảnh và trạng thái của thực thể đó trong tâm trí người đọc.

## 14.3 Tên gọi phải chính xác (Names should be precise)

Tên gọi tốt có hai thuộc tính: **tính chính xác (precision)** và **tính nhất quán (consistency)**. Hãy bắt đầu với tính chính xác. Vấn đề phổ biến nhất với tên gọi là chúng quá chung chung hoặc mơ hồ; kết quả là người đọc khó nhận biết được cái tên đang đề cập đến điều gì; người đọc có thể giả định rằng cái tên đề cập đến một thứ khác với thực tế, như trong lỗi `block` ở trên. Hãy xem xét khai báo phương thức sau đây:

```cpp
/**
 * Returns the total number of indexlets this object is managing.
 */
int IndexletManager::getCount() {...}
```

Thuật ngữ "count" quá chung chung: đếm cái gì? Nếu ai đó nhìn thấy một lời gọi tới phương thức này, họ khó có thể biết nó làm gì trừ khi đọc tài liệu của nó. Một cái tên chính xác hơn như `getActiveIndexlets` hoặc `numIndexlets` sẽ tốt hơn: với một trong những cái tên này, người đọc có thể sẽ đoán được phương thức trả về gì mà không cần phải xem tài liệu của nó.

Dưới đây là một số ví dụ khác về những cái tên chưa đủ chính xác, được lấy từ các dự án của sinh viên:

Một dự án xây dựng trình soạn thảo văn bản GUI đã sử dụng tên `x` và `y` để chỉ vị trí của một ký tự trong tệp tin. Những cái tên này quá chung chung. Chúng có thể mang nhiều ý nghĩa; ví dụ, chúng cũng có thể đại diện cho tọa độ (tính bằng pixel) của một ký tự trên màn hình. Một người nhìn thấy cái tên `x` đứng một mình khó có thể nghĩ rằng nó chỉ vị trí của một ký tự trong một dòng văn bản. Mã nguồn sẽ rõ ràng hơn nếu sử dụng những cái tên như `charIndex` và `lineIndex`, phản ánh đúng các lớp trừu tượng cụ thể mà mã nguồn đang triển khai.

Một dự án trình soạn thảo khác chứa đoạn mã sau:

```java
// Blink state: true when cursor visible.
private boolean blinkStatus = true;
```

Cái tên `blinkStatus` không truyền tải đủ thông tin. Từ "status" (trạng thái) quá mơ hồ đối với một giá trị boolean: nó không đưa ra manh mối nào về việc giá trị `true` hay `false` có nghĩa là gì. Từ "blink" (nhấp nháy) cũng mơ hồ, vì nó không chỉ rõ cái gì đang nhấp nháy. Phương án thay thế sau đây tốt hơn:

```java
// Controls cursor blinking: true means the cursor is visible,
// false means the cursor is not displayed.
private boolean cursorVisible = true;
```

Cái tên `cursorVisible` truyền tải nhiều thông tin hơn; ví dụ, nó cho phép người đọc đoán được giá trị `true` nghĩa là gì (như một quy tắc chung, tên của các biến boolean luôn phải là vị ngữ / mệnh đề đúng-sai). Từ "blink" không còn nằm trong tên nữa, nên người đọc sẽ phải tham khảo tài liệu nếu muốn biết tại sao con trỏ không phải lúc nào cũng hiển thị; thông tin này kém quan trọng hơn.

Một dự án triển khai giao thức đồng thuận (consensus protocol) chứa đoạn mã sau:

```java
// Value representing that the server has not voted (yet) for
// anyone for the current election term.
private static final String VOTED_FOR_SENTINEL_VALUE = "null";
```

Tên của giá trị này chỉ ra rằng nó đặc biệt (sentinel value — giá trị đánh dấu), nhưng nó không nói rõ ý nghĩa đặc biệt đó là gì. Một cái tên cụ thể hơn như `NOT_YET_VOTED` sẽ tốt hơn.

Một biến tên là `result` đã được sử dụng trong một phương thức không có giá trị trả về. Cái tên này có nhiều vấn đề. Thứ nhất, nó tạo ra ấn tượng sai lầm rằng biến đó sẽ là giá trị trả về của phương thức. Thứ hai, nó hầu như không cung cấp thông tin gì về thứ mà nó thực sự lưu giữ, ngoại trừ việc đó là một giá trị được tính toán nào đó. Cái tên nên cung cấp thông tin về việc kết quả đó thực chất là gì, chẳng hạn như `mergedLine` hoặc `totalChars`. Trong các phương thức thực sự có giá trị trả về, thì việc sử dụng tên `result` là hợp lý. Cái tên này vẫn hơi chung chung, nhưng người đọc có thể xem tài liệu của phương thức để biết ý nghĩa của nó, và việc biết rằng giá trị đó cuối cùng sẽ trở thành giá trị trả về cũng rất hữu ích.

> 🚩 **Cờ Đỏ (Red Flag — Tên gọi mơ hồ / Vague Name):** Nếu tên của một biến hoặc phương thức đủ rộng để ám chỉ nhiều thứ khác nhau, thì nó không truyền tải được nhiều thông tin cho lập trình viên và thực thể bên dưới có nhiều khả năng bị sử dụng sai mục đích.

Giống như mọi quy tắc khác, quy tắc về việc chọn tên chính xác cũng có một vài ngoại lệ. Ví dụ, việc dùng những cái tên chung chung như `i` và `j` làm biến lặp trong vòng lặp là hoàn toàn ổn, miễn là các vòng lặp đó chỉ trải dài vài dòng mã. Nếu bạn có thể nhìn thấy toàn bộ phạm vi sử dụng của một biến, thì ý nghĩa của biến đó có lẽ sẽ hiển nhiên từ mã nguồn nên bạn không cần một cái tên dài. Ví dụ, hãy xem đoạn mã sau:

```cpp
for (i = 0; i < numLines; i++) {
      ...
}
```

Từ đoạn mã này, rõ ràng là `i` đang được dùng để lặp qua từng dòng trong một thực thể nào đó. Nếu vòng lặp trở nên quá dài đến mức bạn không thể nhìn thấy toàn bộ nó cùng một lúc, hoặc nếu ý nghĩa của biến lặp khó suy ra từ mã nguồn hơn, thì một cái tên mang tính mô tả hơn là điều cần thiết.

Cũng có khả năng một cái tên bị *quá* cụ thể, chẳng hạn như trong khai báo sau đây cho một phương thức xóa một đoạn văn bản:

```cpp
void delete(Range selection) {...}
```

Tên tham số `selection` là quá cụ thể, vì nó gợi ý rằng văn bản bị xóa luôn luôn là phần đang được chọn (bôi đen) trong giao diện người dùng. Tuy nhiên, phương thức này có thể được gọi trên bất kỳ đoạn văn bản nào, dù có đang được chọn hay không. Do đó, tên tham số nên chung chung hơn một chút, chẳng hạn như `range`.

Nếu bạn thấy khó nghĩ ra một cái tên vừa chính xác, trực quan, lại vừa không quá dài cho một biến cụ thể, thì đây là một cờ đỏ. Nó gợi ý rằng biến đó có thể không có định nghĩa hoặc mục đích rõ ràng. Khi điều này xảy ra, hãy cân nhắc các cách phân tách (factoring) thay thế. Ví dụ, có lẽ bạn đang cố dùng một biến duy nhất để đại diện cho nhiều thứ; nếu đúng vậy, việc tách phần biểu diễn thành nhiều biến khác nhau có thể mang lại một định nghĩa đơn giản hơn cho từng biến. Quá trình chọn tên gọi tốt có thể cải thiện thiết kế của bạn bằng cách phát hiện ra những điểm yếu.

> 🚩 **Cờ Đỏ (Red Flag — Khó chọn tên / Hard to Pick Name):** Nếu bạn thấy khó tìm được một cái tên đơn giản cho một biến hoặc phương thức để tạo ra hình ảnh rõ ràng về đối tượng bên dưới, đó là dấu hiệu cho thấy đối tượng bên dưới có thể không có một thiết kế mạch lạc.

> 💡 **Ví dụ bổ sung (không có trong sách):** Giả sử bạn có một biến lưu thông tin người dùng nhưng lại đặt tên là `userOrError` hoặc `dataAndStatus`. Việc cái tên phải ghép bằng từ "Or" hay "And" thường là dấu hiệu cho thấy biến hoặc hàm đó đang ôm đồm hai trách nhiệm khác nhau. Tương tự, một hàm có tên `validateAndSaveAndSendEmail()` rất khó đặt tên ngắn gọn vì bản thân hàm đó đang làm quá nhiều việc thay vì được tách thành các hàm nhỏ có mục đích rõ ràng.

## 14.4 Sử dụng tên gọi một cách nhất quán (Use names consistently)

Thuộc tính quan trọng thứ hai của tên gọi tốt là tính nhất quán. Trong bất kỳ chương trình nào cũng có những biến nhất định được sử dụng lặp đi lặp lại. Ví dụ, một hệ thống tệp tin thao tác với các số hiệu khối (block number) liên tục. Đối với mỗi mục đích sử dụng phổ biến này, hãy chọn một cái tên để dùng cho mục đích đó, và sử dụng cùng một cái tên đó ở mọi nơi. Chẳng hạn, một hệ thống tệp tin có thể luôn dùng `fileBlock` để giữ chỉ số của một khối bên trong một tệp. Đặt tên nhất quán giúp giảm tải nhận thức (cognitive load) theo cách rất giống với việc tái sử dụng một lớp (class) phổ biến: một khi người đọc đã thấy cái tên đó trong một ngữ cảnh, họ có thể tái sử dụng kiến thức của mình và ngay lập tức đưa ra các giả định khi nhìn thấy cái tên đó trong một ngữ cảnh khác.

Tính nhất quán có ba yêu cầu:
1. Thứ nhất, luôn sử dụng cái tên chung đó cho mục đích đã định;
2. Thứ hai, không bao giờ sử dụng cái tên chung đó cho bất kỳ thứ gì khác ngoài mục đích đã định;
3. Thứ ba, đảm bảo rằng mục đích đó đủ hẹp để tất cả các biến mang cái tên đó đều có cùng hành vi.

Yêu cầu thứ ba này đã bị vi phạm trong lỗi hệ thống tệp tin ở đầu chương. Hệ thống tệp tin đã dùng `block` cho các biến có hai hành vi khác nhau (khối tệp tin và khối đĩa); điều này dẫn đến một giả định sai lầm về ý nghĩa của một biến, và kết quả là gây ra lỗi.

Đôi khi bạn sẽ cần nhiều biến cùng đề cập đến một loại đối tượng chung. Ví dụ, một phương thức sao chép dữ liệu tệp tin sẽ cần hai số hiệu khối, một cho nguồn (source) và một cho đích (destination). Khi điều này xảy ra, hãy sử dụng cái tên chung cho mỗi biến nhưng thêm một tiền tố phân biệt, chẳng hạn như `srcFileBlock` và `dstFileBlock`.

Vòng lặp là một lĩnh vực khác mà việc đặt tên nhất quán có thể giúp ích. Nếu bạn sử dụng các tên như `i` và `j` cho biến vòng lặp, hãy luôn dùng `i` ở các vòng lặp ngoài cùng và `j` cho các vòng lặp lồng bên trong. Điều này cho phép người đọc đưa ra những giả định tức thì (và an toàn) về những gì đang diễn ra trong mã nguồn khi họ nhìn thấy một cái tên nhất định.

## 14.5 Một quan điểm khác: hướng dẫn phong cách của Go (A different opinion: Go style guide)

Không phải ai cũng chia sẻ quan điểm của tôi về việc đặt tên. Một số nhà phát triển của ngôn ngữ Go lập luận rằng tên gọi nên rất ngắn, thường chỉ một ký tự duy nhất. Trong một bài thuyết trình về việc chọn tên cho Go, Andrew Gerrand tuyên bố rằng "những cái tên dài che khuất những gì mã nguồn làm."<sup>1</sup> Ông đưa ra mẫu mã sau đây, sử dụng tên biến một chữ cái:

```go
func RuneCount(b []byte) int {
      i, n := 0, 0
      for i < len(b) {
            if b[i] < RuneSelf {
                  i++
            } else {
                  _, size := DecodeRune(b[i:])
                  i += size
            }
            n++
      }
      return n
}
```

và lập luận rằng nó dễ đọc hơn phiên bản sau đây, vốn sử dụng những cái tên dài hơn:

```go
func RuneCount(buffer []byte) int {
      index, count := 0, 0
      for index < len(buffer) {
            if buffer[index] < RuneSelf {
                  index++
            } else {
                  _, size := DecodeRune(buffer[index:])
                  index += size
            }
            count++
      }
      return count
}
```

Cá nhân tôi không thấy phiên bản thứ hai khó đọc hơn phiên bản thứ nhất chút nào. Nếu có khác biệt, thì cái tên `count` đưa ra một manh mối tốt hơn một chút về hành vi của biến so với `n`. Với phiên bản đầu tiên, cuối cùng tôi đã phải đọc suốt đoạn mã để cố gắng hiểu xem `n` có nghĩa là gì, trong khi tôi không cảm thấy cần phải làm vậy với phiên bản thứ hai. Tuy nhiên, nếu `n` được sử dụng nhất quán trong toàn bộ hệ thống để chỉ các biến đếm (và không chỉ bất cứ thứ gì khác), thì cái tên ngắn đó có lẽ sẽ rõ ràng đối với các lập trình viên khác.

Văn hóa của Go khuyến khích việc sử dụng cùng một cái tên ngắn cho nhiều thứ khác nhau: `ch` cho character (ký tự) hoặc channel (kênh), `d` cho data (dữ liệu), difference (hiệu số), hoặc distance (khoảng cách), v.v. Đối với tôi, những cái tên mơ hồ như thế này rất dễ dẫn đến nhầm lẫn và sai sót, hệt như trong ví dụ về biến `block`.

Nhìn chung, tôi cho rằng tính dễ đọc (readability) phải được quyết định bởi **người đọc**, chứ không phải **người viết**. Nếu bạn viết mã với các tên biến ngắn và những người đọc nó thấy dễ hiểu, thì điều đó hoàn toàn ổn. Nếu bạn bắt đầu nhận được những phàn nàn rằng mã của bạn khó hiểu (cryptic), thì bạn nên cân nhắc dùng tên dài hơn (một tìm kiếm trên Web với từ khóa "go language short names" sẽ cho thấy khá nhiều phàn nàn như vậy). Tương tự, nếu tôi bắt đầu nhận được những phàn nàn rằng tên biến dài khiến mã của tôi khó đọc hơn, thì tôi sẽ cân nhắc dùng những cái tên ngắn hơn.

Gerrand đưa ra một nhận xét mà tôi đồng tình: "Khoảng cách giữa nơi khai báo một cái tên và nơi sử dụng nó càng lớn, thì cái tên đó càng nên dài." Cuộc thảo luận trước đó về việc sử dụng biến vòng lặp tên là `i` và `j` chính là một ví dụ cho quy tắc này.

---
<sup>1</sup> https://talks.golang.org/2014/names.slide#1

## 14.6 Kết luận (Conclusion)

Những cái tên được chọn lọc kỹ lưỡng giúp mã nguồn trở nên hiển nhiên hơn; khi ai đó bắt gặp biến đó lần đầu tiên, phỏng đoán đầu tiên của họ về hành vi của nó — được đưa ra mà không cần suy nghĩ nhiều — sẽ chính xác. Chọn tên gọi tốt là một ví dụ cho tư duy đầu tư (investment mindset) đã được thảo luận trong Chương 3: nếu bạn dành thêm một chút thời gian ngay từ đầu để chọn những cái tên tốt, việc làm việc trên mã nguồn đó trong tương lai sẽ dễ dàng hơn cho bạn. Ngoài ra, bạn cũng sẽ ít có khả năng tạo ra lỗi hơn. Phát triển kỹ năng đặt tên cũng là một khoản đầu tư. Khi mới quyết định ngừng bằng lòng với những cái tên tầm thường, bạn có thể thấy việc nghĩ ra tên hay thật bực bội và tốn thời gian. Tuy nhiên, khi có thêm kinh nghiệm, bạn sẽ thấy việc đó trở nên dễ dàng hơn; cuối cùng, bạn sẽ đạt đến mức gần như không mất thêm thời gian nào để chọn được những cái tên tốt, và nhờ vậy bạn nhận được lợi ích gần như miễn phí.

## Bảng thuật ngữ chương (Chapter Glossary)

| Thuật ngữ (English) | Tiếng Việt | Giải thích ngắn gọn |
| :--- | :--- | :--- |
| **Abstraction** | Trừu tượng hóa | Cách đơn giản hóa để tư duy về một thực thể phức tạp hơn ở bên dưới bằng cách tập trung vào khía cạnh quan trọng nhất. |
| **Precision** | Tính chính xác | Thuộc tính của tên gọi giúp phản ánh đúng, cụ thể thực thể mà nó đại diện, tránh chung chung hay mơ hồ. |
| **Consistency** | Tính nhất quán | Việc luôn dùng cùng một cái tên cho cùng một mục đích/hành vi hẹp trong toàn bộ mã nguồn, và không dùng tên đó cho mục đích khác. |
| **Cognitive load** | Tải nhận thức | Lượng nỗ lực trí óc cần thiết để đọc, hiểu và ghi nhớ các thông tin khi làm việc với mã nguồn. |
| **Sentinel value** | Giá trị đánh dấu / Giá trị canh gác | Một giá trị đặc biệt được dùng để biểu thị một trạng thái ngoại lệ hoặc kết thúc (ví dụ: chưa bình chọn, kết thúc dữ liệu). |
| **Predicate** | Vị ngữ (mệnh đề đúng/sai) | Tên gọi mang tính khẳng định hoặc tính từ (như `cursorVisible`, `isValid`) giúp người đọc đoán ngay được ý nghĩa của giá trị `true`/`false`. |
| **Instrumentation** | Gắn mã theo dõi / Đo đạc | Kỹ thuật chèn thêm mã ghi log hoặc theo dõi vào chương trình để quan sát hành vi thực tế khi tìm lỗi. |
| **Investment mindset** | Tư duy đầu tư | Sẵn sàng bỏ thêm một chút công sức ban đầu để cải thiện thiết kế (bao gồm đặt tên), nhằm tiết kiệm thời gian và tránh lỗi về lâu dài. |
