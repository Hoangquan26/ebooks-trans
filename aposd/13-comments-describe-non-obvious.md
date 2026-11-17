# CHƯƠNG 13: Chú thích nên mô tả những điều không hiển nhiên từ mã nguồn (Chapter 13: Comments Should Describe Things that Aren’t Obvious from the Code)

Lý do viết chú thích (comment) là vì các câu lệnh trong một ngôn ngữ lập trình không thể nắm bắt được toàn bộ những thông tin quan trọng tồn tại trong tâm trí của lập trình viên tại thời điểm viết mã. Chú thích ghi lại những thông tin này để các lập trình viên đến sau có thể dễ dàng hiểu và chỉnh sửa mã nguồn. Nguyên tắc chỉ đạo cho việc viết chú thích là: **chú thích nên mô tả những điều không hiển nhiên từ mã nguồn**.

Có rất nhiều điều không hiển nhiên khi nhìn vào mã nguồn. Đôi khi đó là những chi tiết ở cấp độ thấp (low-level details). Chẳng hạn, khi một cặp chỉ số (index) mô tả một khoảng (range), sẽ không hiển nhiên rằng các phần tử tại hai chỉ số đó có nằm trong khoảng hay nằm ngoài khoảng. Đôi khi không rõ tại sao đoạn mã đó lại cần thiết, hoặc tại sao nó lại được triển khai theo một cách cụ thể. Đôi khi có những quy tắc mà lập trình viên đã tuân theo, chẳng hạn như "luôn gọi `a` trước `b`". Bạn có thể đoán ra quy tắc đó bằng cách đọc toàn bộ mã nguồn, nhưng việc này rất mất công và dễ sai sót; một chú thích có thể làm cho quy tắc đó trở nên tường minh và rõ ràng.

Một trong những lý do quan trọng nhất để viết chú thích là **sự trừu tượng (abstraction)**, vốn chứa đựng rất nhiều thông tin không hiển nhiên từ mã nguồn. Ý tưởng của một sự trừu tượng là cung cấp một cách tư duy đơn giản về một thứ gì đó, nhưng mã nguồn lại quá chi tiết đến mức rất khó để nhìn thấy sự trừu tượng chỉ bằng cách đọc mã. Chú thích có thể cung cấp một góc nhìn đơn giản hơn, ở cấp độ cao hơn ("sau khi phương thức này được gọi, lưu lượng mạng sẽ bị giới hạn ở mức `maxBandwidth` byte mỗi giây"). Ngay cả khi thông tin này có thể suy ra được bằng cách đọc mã nguồn, chúng ta cũng không muốn ép người dùng của một mô-đun phải làm điều đó: việc đọc mã rất tốn thời gian và buộc họ phải cân nhắc nhiều thông tin không cần thiết cho việc sử dụng mô-đun. Lập trình viên phải có khả năng hiểu được sự trừu tượng mà một mô-đun cung cấp mà không cần đọc bất kỳ mã nguồn nào ngoài các khai báo hiển thị ra bên ngoài (externally visible declarations). Cách duy nhất để làm điều này là bổ sung chú thích cho các khai báo đó.

Chương này thảo luận về những thông tin nào cần được mô tả trong chú thích và cách viết chú thích tốt. Như bạn sẽ thấy, các chú thích tốt thường giải thích sự việc ở một mức độ chi tiết khác với mã nguồn — chi tiết hơn trong một số tình huống và ít chi tiết hơn (trừu tượng hơn) trong những tình huống khác.

## 13.1 Lựa chọn quy ước (Pick conventions)

Bước đầu tiên trong việc viết chú thích là quyết định các quy ước cho việc chú thích, chẳng hạn như bạn sẽ chú thích những gì và định dạng bạn sẽ sử dụng. Nếu bạn đang lập trình bằng một ngôn ngữ có sẵn công cụ biên dịch tài liệu, chẳng hạn như Javadoc cho Java, Doxygen cho C++, hoặc godoc cho Go, hãy tuân theo các quy ước của những công cụ đó. Không có quy ước nào là hoàn hảo, nhưng các công cụ này mang lại đủ nhiều lợi ích để bù đắp cho những khiếm khuyết đó. Nếu bạn đang lập trình trong một môi trường chưa có quy ước sẵn có, hãy thử áp dụng quy ước từ một ngôn ngữ hoặc dự án tương tự khác; điều này sẽ giúp các lập trình viên khác dễ dàng hiểu và tuân thủ quy ước của bạn hơn.

Quy ước phục vụ hai mục đích. Thứ nhất, chúng đảm bảo tính nhất quán, giúp chú thích dễ đọc và dễ hiểu hơn. Thứ hai, chúng giúp đảm bảo rằng bạn thực sự viết chú thích. Nếu bạn không có ý tưởng rõ ràng về việc mình sẽ chú thích cái gì và như thế nào, rất dễ dẫn đến việc không viết bất kỳ chú thích nào cả.

Hầu hết các chú thích đều rơi vào một trong các loại sau:

- **Giao diện (Interface):** Một khối chú thích nằm ngay trước phần khai báo của một mô-đun như lớp (class), cấu trúc dữ liệu (data structure), hàm (function), hoặc phương thức (method). Chú thích này mô tả giao diện của mô-đun. Đối với một lớp, chú thích mô tả sự trừu tượng tổng thể mà lớp đó cung cấp. Đối với một phương thức hoặc hàm, chú thích mô tả hành vi tổng thể, các tham số (arguments) và giá trị trả về (return value) nếu có, bất kỳ tác dụng phụ (side effects) hay ngoại lệ (exceptions) nào mà nó tạo ra, và bất kỳ yêu cầu nào khác mà bên gọi (caller) phải đáp ứng trước khi gọi phương thức.
- **Thành viên của cấu trúc dữ liệu (Data structure member):** Một chú thích nằm cạnh phần khai báo của một trường (field) trong cấu trúc dữ liệu, chẳng hạn như biến thực thể (instance variable) hoặc biến tĩnh (static variable) của một lớp.
- **Chú thích triển khai (Implementation comment):** Một chú thích bên trong mã nguồn của một phương thức hoặc hàm, mô tả cách mã nguồn hoạt động ở bên trong.
- **Chú thích liên mô-đun (Cross-module comment):** Một chú thích mô tả các sự phụ thuộc vượt qua ranh giới giữa các mô-đun.

Những chú thích quan trọng nhất là hai loại đầu tiên. Mọi lớp đều nên có chú thích giao diện, mọi biến của lớp đều nên có chú thích, và mọi phương thức đều nên có chú thích giao diện. Đôi khi, khai báo của một biến hoặc phương thức quá hiển nhiên đến mức không có gì hữu ích để thêm vào chú thích (các hàm getter và setter đôi khi rơi vào trường hợp này), nhưng điều này rất hiếm; việc chú thích mọi thứ sẽ dễ dàng hơn là tốn năng lượng để băn khoăn xem có cần chú thích hay không. Các chú thích triển khai thường không cần thiết (xem Mục 13.6 bên dưới). Chú thích liên mô-đun là loại hiếm gặp nhất và khó viết nhất, nhưng khi cần thiết thì chúng lại cực kỳ quan trọng; Mục 13.7 sẽ thảo luận chi tiết hơn về chúng.

## 13.2 Đừng lặp lại mã nguồn (Don’t repeat the code)

Đáng tiếc là nhiều chú thích không mấy hữu ích. Lý do phổ biến nhất là chú thích lặp lại mã nguồn: tất cả thông tin trong chú thích đều có thể dễ dàng suy ra từ đoạn mã ngay bên cạnh nó. Dưới đây là một đoạn mã mẫu từng xuất hiện trong một bài báo nghiên cứu gần đây:

```python
ptr_copy = get_copy(obj)
# Get pointer copy
if is_unlocked(ptr_copy):
# Is obj free?
    return obj
# return current obj
if is_copy(ptr_copy):
# Already a copy?
    return obj
# return obj
thread_id = get_thread_id(ptr_copy)
if thread_id == ctx.thread_id:
# Locked by current ctx
    return ptr_copy
# Return copy
```

Không có thông tin hữu ích nào trong bất kỳ chú thích nào ở trên ngoại trừ chú thích `"Locked by"` (bị khóa bởi), vốn gợi ý một điều gì đó về luồng (thread) mà có thể không hiển nhiên từ mã nguồn. Lưu ý rằng các chú thích này ở cùng một mức độ chi tiết với mã nguồn: cứ mỗi dòng mã lại có một chú thích mô tả đúng dòng đó. Những chú thích như thế này hiếm khi hữu ích.

Dưới đây là thêm một vài ví dụ về các chú thích lặp lại mã nguồn:

```java
// Add a horizontal scroll bar
hScrollBar = new JScrollBar(JScrollBar.HORIZONTAL);
add(hScrollBar, BorderLayout.SOUTH);
// Add a vertical scroll bar
vScrollBar = new JScrollBar(JScrollBar.VERTICAL);
add(vScrollBar, BorderLayout.EAST);

// Initialize the caret-position related values
caretX     = 0;
caretY     = 0;
caretMemX  = null;
```

Không có chú thích nào trong số này mang lại giá trị. Với hai chú thích đầu tiên, mã nguồn đã đủ rõ ràng đến mức thực sự không cần chú thích; trong trường hợp thứ ba, một chú thích có thể hữu ích, nhưng chú thích hiện tại không cung cấp đủ chi tiết để giúp ích.

Sau khi viết xong một chú thích, hãy tự hỏi bản thân câu hỏi sau: **một người chưa từng thấy đoạn mã này trước đây có thể tự viết ra chú thích đó chỉ bằng cách nhìn vào đoạn mã ngay bên cạnh không?** Nếu câu trả lời là có, như trong các ví dụ trên, thì chú thích đó không làm cho mã nguồn dễ hiểu hơn chút nào. Những chú thích như thế này chính là lý do khiến một số người nghĩ rằng chú thích là vô giá trị.

Một sai lầm phổ biến khác là sử dụng lại chính những từ xuất hiện trong tên của thực thể đang được viết tài liệu:

```java
/*
 * Obtain a normalized resource name from REQ.
 */
private static String[] getNormalizedResourceNames(
           HTTPRequest req) ...

/*
 * Downcast PARAMETER to TYPE.
 */
private static Object downCastParameter(String parameter, String type)
...

/*
 * The horizontal padding of each line in the text.
 */
private static final int textHorizontalPadding = 4;
```

Những chú thích này chỉ lấy các từ từ tên phương thức hoặc tên biến, có thể thêm một vài từ từ tên tham số và kiểu dữ liệu, rồi ghép chúng thành một câu. Chẳng hạn, thứ duy nhất trong chú thích thứ hai không có trong mã nguồn là từ `"to"`! Một lần nữa, các chú thích này có thể được viết chỉ bằng cách nhìn vào phần khai báo, mà không cần bất kỳ sự hiểu biết nào về các phương thức hay biến; kết quả là chúng không có giá trị gì.

> 🚩 **Cờ Đỏ (Red Flag — Chú thích lặp lại mã nguồn / Comment Repeats Code):**
> Nếu thông tin trong một chú thích đã hiển nhiên từ đoạn mã ngay bên cạnh chú thích đó, thì chú thích đó không hữu ích. Một ví dụ của việc này là khi chú thích sử dụng chính những từ cấu thành nên tên của thứ mà nó đang mô tả.

Đồng thời, lại có những thông tin quan trọng bị thiếu khỏi các chú thích trên: chẳng hạn, `"normalized resource name"` (tên tài nguyên chuẩn hóa) là gì, và các phần tử của mảng được trả về bởi `getNormalizedResourceNames` là gì? `"downcast"` nghĩa là gì? Đơn vị của phần đệm (`padding`) là gì, và phần đệm được áp dụng cho một bên của mỗi dòng hay cả hai bên? Việc mô tả những điều này trong chú thích sẽ rất hữu ích.

Bước đầu tiên để viết chú thích tốt là **sử dụng các từ ngữ trong chú thích khác với các từ trong tên của thực thể đang được mô tả**. Hãy chọn những từ ngữ cho chú thích sao cho chúng cung cấp thêm thông tin về ý nghĩa của thực thể, thay vì chỉ lặp lại tên của nó. Ví dụ, dưới đây là một chú thích tốt hơn cho `textHorizontalPadding`:

```java
/*
 * The amount of blank space to leave on the left and
 * right sides of each line of text, in pixels.
 */
private static final int textHorizontalPadding = 4;
```
*(Ghi chú dịch nghĩa: Lượng khoảng trống để chừa lại ở bên trái và bên phải của mỗi dòng văn bản, tính bằng pixel.)*

Chú thích này cung cấp thêm thông tin không hiển nhiên từ bản thân phần khai báo, chẳng hạn như đơn vị (`pixels`) và thực tế là phần đệm áp dụng cho cả hai bên của mỗi dòng. Thay vì dùng thuật ngữ `"padding"`, chú thích giải thích rõ padding là gì, phòng trường hợp người đọc chưa quen thuộc với thuật ngữ này.

## 13.3 Chú thích cấp thấp giúp tăng độ chính xác (Lower-level comments add precision)

Bây giờ bạn đã biết những điều không nên làm, hãy cùng thảo luận xem bạn nên đưa thông tin gì vào chú thích. Chú thích bổ trợ cho mã nguồn bằng cách cung cấp thông tin ở một mức độ chi tiết khác. Một số chú thích cung cấp thông tin ở cấp độ thấp hơn, chi tiết hơn so với mã nguồn; những chú thích này tăng thêm độ chính xác bằng cách làm rõ ý nghĩa chính xác của mã nguồn. Những chú thích khác lại cung cấp thông tin ở cấp độ cao hơn, trừu tượng hơn so với mã nguồn; những chú thích này mang lại trực giác, chẳng hạn như lý do đằng sau đoạn mã, hoặc một cách tư duy đơn giản hơn và trừu tượng hơn về đoạn mã. Các chú thích ở cùng cấp độ với mã nguồn rất dễ rơi vào tình trạng lặp lại mã. Mục này sẽ thảo luận chi tiết hơn về cách tiếp cận cấp thấp, và mục tiếp theo sẽ thảo luận về cách tiếp cận cấp cao.

Độ chính xác hữu ích nhất khi chú thích cho các khai báo biến như biến thực thể của lớp, tham số của phương thức, và giá trị trả về. Tên và kiểu dữ liệu trong một khai báo biến thường không đủ chính xác. Chú thích có thể điền vào những chi tiết còn thiếu như:
- Đơn vị của biến này là gì?
- Các điều kiện biên là bao gồm (inclusive) hay không bao gồm (exclusive)?
- Nếu giá trị `null` được phép, nó hàm ý điều gì?
- Nếu một biến tham chiếu đến một tài nguyên cuối cùng phải được giải phóng hoặc đóng lại, ai là người chịu trách nhiệm giải phóng hoặc đóng nó?
- Có những tính chất nào luôn đúng đối với biến này (các bất biến - invariants), chẳng hạn như "danh sách này luôn chứa ít nhất một phần tử" không?

Một số thông tin này có thể tìm ra bằng cách kiểm tra tất cả các đoạn mã nơi biến được sử dụng. Tuy nhiên, việc đó rất tốn thời gian và dễ sai sót; chú thích ở phần khai báo phải đủ rõ ràng và đầy đủ để không cần phải làm việc đó. Khi tôi nói rằng chú thích cho một khai báo nên mô tả những điều không hiển nhiên từ mã nguồn, "mã nguồn" ở đây ám chỉ đoạn mã ngay bên cạnh chú thích (phần khai báo), chứ không phải "toàn bộ mã nguồn trong ứng dụng".

Vấn đề phổ biến nhất với chú thích cho biến là chú thích quá mơ hồ. Dưới đây là hai ví dụ về các chú thích chưa đủ chính xác:

```cpp
// Current offset in resp Buffer
uint32_t offset;
```

```java
// Contains all line-widths inside the document and
// number of appearances.
private TreeMap<Integer, Integer> lineWidths;
```

Trong ví dụ đầu tiên, không rõ từ `"current"` (hiện tại) có nghĩa là gì. Trong ví dụ thứ hai, không rõ rằng các khóa (key) trong `TreeMap` là độ rộng dòng và các giá trị (value) là số lần xuất hiện. Ngoài ra, độ rộng được đo bằng pixel hay bằng ký tự? Các chú thích được viết lại dưới đây cung cấp thêm chi tiết:

```cpp
//  Position in this buffer of the first object that hasn't
//  been returned to the client.
uint32_t offset;
```
*(Ghi chú dịch nghĩa: Vị trí trong bộ đệm này của đối tượng đầu tiên chưa được trả về cho client.)*

```java
//  Holds statistics about line lengths of the form <length, count>
//  where length is the number of characters in a line (including
//  the newline), and count is the number of lines with
//  exactly that many characters. If there are no lines with
//  a particular length, then there is no entry for that length.
private TreeMap<Integer, Integer> numLinesWithLength;
```
*(Ghi chú dịch nghĩa: Lưu giữ thống kê về độ dài dòng dưới dạng `<length, count>`, trong đó `length` là số lượng ký tự trong một dòng (bao gồm cả ký tự xuống dòng), và `count` là số lượng dòng có chính xác ngần ấy ký tự. Nếu không có dòng nào có một độ dài cụ thể, thì sẽ không có mục nhập nào cho độ dài đó.)*

Khai báo thứ hai sử dụng một cái tên dài hơn nhưng truyền tải nhiều thông tin hơn. Nó cũng đổi `"width"` (độ rộng) thành `"length"` (độ dài), vì thuật ngữ này dễ khiến người ta nghĩ đến đơn vị là ký tự thay vì pixel. Lưu ý rằng chú thích thứ hai không chỉ ghi lại chi tiết của từng mục, mà còn mô tả ý nghĩa khi một mục bị thiếu.

Khi viết tài liệu cho một biến, **hãy nghĩ bằng danh từ, không phải động từ**. Nói cách khác, hãy tập trung vào việc biến đó *đại diện cho cái gì*, chứ không phải cách nó bị thao túng ra sao. Hãy xem xét chú thích sau:

```java
/* FOLLOWER VARIABLE: indicator variable that allows the Receiver and the
 * PeriodicTasks thread to communicate about whether a heartbeat has been
 * received within the follower's election timeout window.
 * Toggled to TRUE when a valid heartbeat is received.
 * Toggled to FALSE when the election timeout window is reset.  */
private boolean receivedValidHeartbeat;
```

Tài liệu này mô tả cách biến bị sửa đổi bởi nhiều đoạn mã khác nhau trong lớp. Chú thích sẽ vừa ngắn hơn vừa hữu ích hơn nếu nó mô tả thứ mà biến đại diện thay vì phản chiếu lại cấu trúc mã nguồn:

```java
/* True means that a heartbeat has been received since the last time
 * the election timer was reset. Used for communication between the
 * Receiver and PeriodicTasks threads.  */
private boolean receivedValidHeartbeat;
```
*(Ghi chú dịch nghĩa: `True` có nghĩa là một tín hiệu nhịp tim (heartbeat) đã được nhận kể từ lần cuối bộ đếm thời gian bầu cử được đặt lại. Được sử dụng để giao tiếp giữa các luồng `Receiver` và `PeriodicTasks`.)*

Với tài liệu này, có thể dễ dàng suy ra rằng biến phải được đặt thành `true` khi nhận được heartbeat và `false` khi bộ đếm thời gian bầu cử được đặt lại.

> 💡 **Ví dụ bổ sung (không có trong sách):**
> Khi khai báo một tham số hoặc biến liên quan đến thời gian, tên biến thường không đủ để tránh nhầm lẫn đơn vị hoặc điều kiện biên:
> ```java
> // Chưa tốt: lặp lại tên biến, thiếu đơn vị và ý nghĩa giá trị đặc biệt
> // Timeout for the request.
> private int requestTimeout;
>
> // Tốt hơn: nêu rõ đơn vị, ý nghĩa giá trị 0/âm, và hành vi khi hết hạn
> // Maximum time to wait for a server response before aborting, in milliseconds.
> // A value of 0 means wait indefinitely (no timeout).
> private int requestTimeoutMs;
> ```
> Tương tự với hàm tìm kiếm theo khoảng `[startIndex, endIndex)`: chỉ rõ `startIndex` là bao gồm (inclusive) còn `endIndex` là không bao gồm (exclusive) giúp người gọi không bao giờ bị lỗi lệch một đơn vị (off-by-one error).

## 13.4 Chú thích cấp cao nâng cao trực giác (Higher-level comments enhance intuition)

Cách thứ hai mà chú thích có thể bổ trợ cho mã nguồn là mang lại trực giác (intuition). Những chú thích này được viết ở cấp độ cao hơn mã nguồn. Chúng lược bỏ các chi tiết và giúp người đọc hiểu được ý định tổng thể (overall intent) và cấu trúc của đoạn mã. Cách tiếp cận này thường được dùng cho các chú thích bên trong phương thức và cho các chú thích giao diện. Ví dụ, hãy xem xét đoạn mã sau:

```java
// If there is a LOADING readRpc using the same session
// as PKHash pointed to by assignPos, and the last PKHash
// in that readRPC is smaller than current assigning
// PKHash, then we put assigning PKHash into that readRPC.
int readActiveRpcId = RPC_ID_NOT_ASSIGNED;
for (int i = 0; i < NUM_READ_RPC; i++) {
     if (session == readRpc[i].session
                && readRpc[i].status == LOADING
                && readRpc[i].maxPos < assignPos
                && readRpc[i].numHashes < MAX_PKHASHES_PERRPC) {
         readActiveRpcId = i;
         break;
     }
}
```

Chú thích này quá chi tiết và ở cấp độ quá thấp. Một mặt, nó lặp lại một phần mã nguồn: `"if there is a LOADING readRPC"` chỉ là sự sao chép lại phép kiểm tra `readRpc[i].status == LOADING`. Mặt khác, chú thích không giải thích mục đích tổng thể của đoạn mã này, hoặc cách nó khớp vào phương thức chứa nó. Kết quả là chú thích không giúp người đọc hiểu được mã nguồn.

Dưới đây là một chú thích tốt hơn:

```java
// Try to append the current key hash onto an existing
// RPC to the desired server that hasn't been sent yet.
```
*(Ghi chú dịch nghĩa: Thử nối mã băm khóa hiện tại vào một RPC đang tồn tại hướng tới máy chủ mong muốn mà vẫn chưa được gửi đi.)*

Chú thích này không chứa bất kỳ chi tiết nào; thay vào đó, nó mô tả chức năng tổng thể của đoạn mã ở cấp độ cao hơn. Với thông tin cấp cao này, người đọc có thể giải thích được hầu hết mọi thứ xảy ra trong đoạn mã: vòng lặp hẳn là đang duyệt qua tất cả các lời gọi thủ tục từ xa (RPC) hiện có; phép kiểm tra `session` có lẽ dùng để xem một RPC cụ thể có đang hướng đến đúng máy chủ hay không; phép kiểm tra `LOADING` gợi ý rằng các RPC có thể có nhiều trạng thái, và ở một số trạng thái thì không an toàn để thêm các mã băm (hash); phép kiểm tra `MAX_PKHASHES_PERRPC` gợi ý rằng có một giới hạn về số lượng mã băm có thể gửi trong một RPC duy nhất. Thứ duy nhất không được giải thích bởi chú thích là phép kiểm tra `maxPos`. Hơn nữa, chú thích mới cung cấp một cơ sở để người đọc đánh giá đoạn mã: nó đã làm mọi thứ cần thiết để thêm mã băm khóa vào một RPC hiện có chưa? Chú thích ban đầu không mô tả ý định tổng thể của đoạn mã, nên người đọc rất khó quyết định liệu đoạn mã có đang hoạt động đúng đắn hay không.

Chú thích cấp cao khó viết hơn chú thích cấp thấp vì bạn phải suy nghĩ về mã nguồn theo một cách khác. Hãy tự hỏi mình: *Đoạn mã này đang cố gắng làm gì? Điều đơn giản nhất bạn có thể nói mà giải thích được mọi thứ trong đoạn mã là gì? Điều quan trọng nhất về đoạn mã này là gì?*

Các kỹ sư có xu hướng rất chú trọng tiểu tiết. Chúng ta yêu thích các chi tiết và rất giỏi trong việc quản lý hàng tá chi tiết; điều này rất cần thiết để trở thành một kỹ sư giỏi. Tuy nhiên, những nhà thiết kế phần mềm xuất sắc còn có thể lùi lại khỏi các chi tiết và suy nghĩ về hệ thống ở cấp độ cao hơn. Điều này có nghĩa là phải quyết định xem những khía cạnh nào của hệ thống là quan trọng nhất, đồng thời có khả năng bỏ qua các chi tiết cấp thấp để chỉ nghĩ về hệ thống dựa trên những đặc tính cơ bản nhất của nó. Đây chính là bản chất của sự trừu tượng (tìm ra một cách đơn giản để tư duy về một thực thể phức tạp), và đó cũng là điều bạn phải làm khi viết các chú thích cấp cao. Một chú thích cấp cao tốt diễn đạt một hoặc một vài ý tưởng đơn giản cung cấp một khung khái niệm (conceptual framework), chẳng hạn như `"append to an existing RPC"` (nối vào một RPC hiện có). Khi đã có khung khái niệm đó, việc nhìn ra mối liên hệ giữa các câu lệnh cụ thể với mục tiêu tổng thể trở nên dễ dàng.

Dưới đây là một đoạn mã mẫu khác có chú thích cấp cao rất tốt:

```cpp
if (numProcessedPKHashes < readRpc[i].numHashes) {
      // Some of the key hashes couldn't be looked up in
      // this request (either because they aren't stored
      // on the server, the server crashed, or there
      // wasn't enough space in the response message).
      // Mark the unprocessed hashes so they will get
      // reassigned to new RPCs.
      for (size_t p = removePos; p < insertPos; p++) {
             if (activeRpcId[p] == i) {
                    if (numProcessedPKHashes > 0) {
                          numProcessedPKHashes--;
                    } else {
                          if (p < assignPos)
                               assignPos = p;
                          activeRpcId[p] = RPC_ID_NOT_ASSIGNED;
                    }
             }
      }
}
```
*(Ghi chú dịch nghĩa: Một số mã băm khóa không thể tra cứu được trong yêu cầu này (hoặc vì chúng không được lưu trên máy chủ, máy chủ bị sập, hoặc không có đủ chỗ trong thông điệp phản hồi). Đánh dấu các mã băm chưa được xử lý để chúng được gán lại cho các RPC mới.)*

Chú thích này làm hai việc. Câu thứ hai cung cấp một mô tả trừu tượng về việc đoạn mã *làm gì*. Câu đầu tiên lại khác: nó giải thích (bằng ngôn từ cấp cao) *tại sao* đoạn mã lại được thực thi. Các chú thích có dạng "chúng ta đi đến đây như thế nào" (how we get here) rất hữu ích trong việc giúp mọi người hiểu mã nguồn. Chẳng hạn, khi viết tài liệu cho một phương thức, sẽ rất hữu ích nếu mô tả các điều kiện mà phương thức đó có nhiều khả năng được gọi nhất (đặc biệt nếu phương thức chỉ được gọi trong các tình huống bất thường).

## 13.5 Tài liệu giao diện (Interface documentation)

Một trong những vai trò quan trọng nhất của chú thích là định nghĩa các sự trừu tượng. Hãy nhớ lại từ Chương 4 rằng một sự trừu tượng là một góc nhìn đơn giản hóa về một thực thể, trong đó giữ lại các thông tin thiết yếu nhưng lược bỏ những chi tiết có thể bỏ qua một cách an toàn. Mã nguồn không phù hợp để mô tả sự trừu tượng; nó quá thấp và chứa đựng các chi tiết triển khai vốn không nên hiển thị trong sự trừu tượng. Cách duy nhất để mô tả một sự trừu tượng là bằng chú thích. Nếu bạn muốn mã nguồn trình bày được các sự trừu tượng tốt, bạn bắt buộc phải viết tài liệu cho các sự trừu tượng đó bằng chú thích.

Bước đầu tiên trong việc viết tài liệu cho các sự trừu tượng là **tách biệt chú thích giao diện khỏi chú thích triển khai**. Chú thích giao diện cung cấp thông tin mà một người cần biết để sử dụng một lớp hoặc phương thức; chúng định nghĩa sự trừu tượng. Chú thích triển khai mô tả cách một lớp hoặc phương thức hoạt động ở bên trong để hiện thực hóa sự trừu tượng đó. Việc tách biệt hai loại chú thích này rất quan trọng để người dùng của một giao diện không bị phơi bày các chi tiết triển khai. Hơn nữa, hai dạng chú thích này tốt hơn hết là phải khác nhau. Nếu chú thích giao diện cũng phải mô tả cả phần triển khai, thì lớp hoặc phương thức đó là nông (shallow). Điều này có nghĩa là hành động viết chú thích có thể cung cấp những manh mối về chất lượng của một thiết kế; Chương 15 sẽ quay lại ý tưởng này.

Chú thích giao diện cho một lớp cung cấp mô tả cấp cao về sự trừu tượng mà lớp đó mang lại, như ví dụ sau:

```java
/**
 * This class implements a simple server-side interface to the HTTP
 * protocol: by using this class, an application can receive HTTP
 * requests, process them, and return responses. Each instance of
 * this class corresponds to a particular socket used to receive
 * requests. The current implementation is single-threaded and
 * processes one request at a time.
 */
public class Http {...}
```
*(Ghi chú dịch nghĩa: Lớp này triển khai một giao diện phía máy chủ đơn giản cho giao thức HTTP: bằng cách sử dụng lớp này, một ứng dụng có thể nhận các yêu cầu HTTP, xử lý chúng và trả về phản hồi. Mỗi thực thể của lớp này tương ứng với một socket cụ thể dùng để nhận yêu cầu. Bản triển khai hiện tại chạy đơn luồng và xử lý từng yêu cầu một tại một thời điểm.)*

Chú thích này mô tả các khả năng tổng thể của lớp, không có bất kỳ chi tiết triển khai nào hay thậm chí là chi tiết cụ thể của từng phương thức riêng lẻ. Nó cũng mô tả mỗi thực thể của lớp đại diện cho cái gì. Cuối cùng, chú thích mô tả các giới hạn của lớp (nó không hỗ trợ truy cập đồng thời từ nhiều luồng), điều có thể rất quan trọng đối với các lập trình viên đang cân nhắc xem có nên sử dụng nó hay không.

Chú thích giao diện cho một phương thức bao gồm cả thông tin cấp cao cho sự trừu tượng lẫn thông tin cấp thấp để đảm bảo độ chính xác:
- Chú thích thường bắt đầu bằng một hoặc hai câu mô tả hành vi của phương thức dưới góc nhìn của bên gọi; đây là sự trừu tượng cấp cao.
- Chú thích phải mô tả từng tham số và giá trị trả về (nếu có). Các chú thích này phải rất chính xác, và phải mô tả bất kỳ ràng buộc nào trên giá trị tham số cũng như sự phụ thuộc giữa các tham số.
- Nếu phương thức có bất kỳ tác dụng phụ (side effect) nào, chúng phải được ghi lại trong chú thích giao diện. Tác dụng phụ là bất kỳ hệ quả nào của phương thức mà ảnh hưởng đến hành vi tương lai của hệ thống nhưng không phải là một phần của kết quả trả về. Ví dụ, nếu phương thức thêm một giá trị vào cấu trúc dữ liệu nội bộ để có thể truy xuất bởi các lời gọi phương thức trong tương lai, đó là một tác dụng phụ; việc ghi vào hệ thống tệp cũng là một tác dụng phụ.
- Chú thích giao diện của phương thức phải mô tả bất kỳ ngoại lệ (exception) nào có thể phát sinh từ phương thức.
- Nếu có bất kỳ tiền điều kiện (precondition) nào phải được thỏa mãn trước khi gọi phương thức, chúng phải được mô tả (chẳng hạn như một phương thức khác phải được gọi trước; hoặc đối với phương thức tìm kiếm nhị phân, danh sách được tìm kiếm phải được sắp xếp trước). Tốt nhất là nên giảm thiểu các tiền điều kiện, nhưng bất kỳ tiền điều kiện nào còn lại đều phải được ghi tài liệu rõ ràng.

Dưới đây là chú thích giao diện cho một phương thức sao chép dữ liệu ra khỏi một đối tượng `Buffer`:

```cpp
/**
 * Copy a range of bytes from a buffer to an external location.
 *
 * \param offset
 *        Index within the buffer of the first byte to copy.
 * \param length
 *        Number of bytes to copy.
 * \param dest
 *        Where to copy the bytes: must have room for at least
 *        length bytes.
 *
 * \return
 *        The return value is the actual number of bytes copied,
 *        which may be less than length if the requested range of
 *        bytes extends past the end of the buffer. 0 is returned
 *        if there is no overlap between the requested range and
 *        the actual buffer.
 */
uint32_t
Buffer::copy(uint32_t offset, uint32_t length, void* dest)
...
```

Cú pháp của chú thích này (ví dụ `\return`) tuân theo quy ước của Doxygen, một chương trình trích xuất chú thích từ mã nguồn C/C++ và biên dịch chúng thành các trang Web. Mục tiêu của chú thích là cung cấp tất cả thông tin mà một lập trình viên cần để gọi phương thức, bao gồm cả cách các trường hợp đặc biệt được xử lý (hãy để ý cách phương thức này tuân theo lời khuyên ở Chương 10 và định nghĩa để triệt tiêu các lỗi liên quan đến việc chỉ định phạm vi). Lập trình viên không cần phải đọc phần thân của phương thức để gọi nó, và chú thích giao diện cũng không cung cấp bất kỳ thông tin nào về cách phương thức được triển khai, chẳng hạn như cách nó quét các cấu trúc dữ liệu nội bộ để tìm dữ liệu mong muốn.

Để xem một ví dụ dài hơn, hãy xem xét một lớp tên là `IndexLookup`, vốn là một phần của hệ thống lưu trữ phân tán. Hệ thống lưu trữ nắm giữ một tập hợp các bảng (table), mỗi bảng chứa nhiều đối tượng (object). Ngoài ra, mỗi bảng có thể có một hoặc nhiều chỉ mục (index); mỗi chỉ mục cung cấp khả năng truy cập hiệu quả vào các đối tượng trong bảng dựa trên một trường cụ thể của đối tượng. Ví dụ, một chỉ mục có thể dùng để tra cứu đối tượng dựa trên trường `name` của chúng, và một chỉ mục khác có thể dùng để tra cứu đối tượng dựa trên trường `age`. Với các chỉ mục này, ứng dụng có thể nhanh chóng trích xuất tất cả các đối tượng có một tên cụ thể, hoặc tất cả các đối tượng có độ tuổi trong một khoảng cho trước.

Lớp `IndexLookup` cung cấp một giao diện thuận tiện để thực hiện các phép tra cứu theo chỉ mục. Dưới đây là ví dụ về cách nó có thể được sử dụng trong một ứng dụng:

```java
query = new IndexLookup(table, index, key1, key2);
while (true) {
       object = query.getNext();
       if (object == NULL) {
             break;
       }
       ... process object ...
}
```

Đầu tiên, ứng dụng khởi tạo một đối tượng kiểu `IndexLookup`, cung cấp các tham số chọn một bảng, một chỉ mục, và một khoảng trong chỉ mục đó (ví dụ, nếu chỉ mục dựa trên trường `age`, `key1` và `key2` có thể được chỉ định là 21 và 65 để chọn tất cả các đối tượng có độ tuổi nằm giữa hai giá trị này). Sau đó, ứng dụng gọi phương thức `getNext` lặp đi lặp lại. Mỗi lần gọi trả về một đối tượng nằm trong khoảng mong muốn; một khi tất cả các đối tượng khớp đã được trả về, `getNext` trả về `NULL`. Vì hệ thống lưu trữ là phân tán, phần triển khai của lớp này khá phức tạp. Các đối tượng trong một bảng có thể nằm rải rác trên nhiều máy chủ, và mỗi chỉ mục cũng có thể được phân tán trên một tập hợp các máy chủ khác nhau; mã nguồn trong lớp `IndexLookup` trước tiên phải giao tiếp với tất cả các máy chủ chỉ mục liên quan để thu thập thông tin về các đối tượng trong khoảng, sau đó nó phải giao tiếp với các máy chủ thực sự lưu trữ các đối tượng để truy xuất giá trị của chúng.

Bây giờ hãy xem xét những thông tin nào cần được đưa vào chú thích giao diện cho lớp này. Với mỗi mẩu thông tin đưa ra dưới đây, hãy tự hỏi xem một lập trình viên có cần biết thông tin đó để sử dụng lớp này hay không (câu trả lời của tôi cho các câu hỏi nằm ở cuối chương):

1. Định dạng của các thông điệp mà lớp `IndexLookup` gửi tới các máy chủ nắm giữ chỉ mục và đối tượng.
2. Hàm so sánh được sử dụng để xác định xem một đối tượng cụ thể có nằm trong khoảng mong muốn hay không (việc so sánh được thực hiện bằng số nguyên, số thực dấu phẩy động, hay chuỗi ký tự?).
3. Cấu trúc dữ liệu được sử dụng để lưu trữ các chỉ mục trên máy chủ.
4. Liệu `IndexLookup` có phát hành nhiều yêu cầu tới các máy chủ khác nhau một cách đồng thời hay không.
5. Cơ chế xử lý khi máy chủ bị sập (crash).

Dưới đây là phiên bản gốc của chú thích giao diện cho lớp `IndexLookup`; đoạn trích cũng bao gồm một vài dòng từ phần định nghĩa lớp được nhắc đến trong chú thích:

```cpp
/*
 * This class implements the client side framework for index range
 * lookups. It manages a single LookupIndexKeys RPC and multiple
 * IndexedRead RPCs. Client side just includes "IndexLookup.h" in
 * its header to use IndexLookup class. Several parameters can be set
 * in the config below:
 * - The number of concurrent indexedRead RPCs
 * - The max number of PKHashes a indexedRead RPC can hold at a time
 * - The size of the active PKHashes
 *
 * To use IndexLookup, the client creates an object of this class by
 * providing all necessary information. After construction of
 * IndexLookup, client can call getNext() function to move to next
 * available object. If getNext() returns NULL, it means we reached
 * the last object. Client can use getKey, getKeyLength, getValue,
 * and getValueLength to get object data of current object.
 */
class IndexLookup {
      ...
  private:
      /// Max number of concurrent indexedRead RPCs
      static const uint8_t NUM_READ_RPC = 10;
      /// Max number of PKHashes that can be sent in one
      /// indexedRead RPC
      static const uint32_t MAX_PKHASHES_PERRPC = 256;
      /// Max number of PKHashes that activeHashes can
      /// hold at once.
      static const size_t MAX_NUM_PK = (1 << LG_BUFFER_SIZE);
}
```

Trước khi đọc tiếp, hãy thử xem bạn có thể chỉ ra các vấn đề với chú thích này không. Dưới đây là những vấn đề mà tôi tìm thấy:
- Hầu hết đoạn văn đầu tiên liên quan đến phần triển khai, chứ không phải giao diện. Lấy một ví dụ, người dùng không cần biết tên của các lời gọi thủ tục từ xa (RPC) cụ thể được dùng để giao tiếp với máy chủ. Các tham số cấu hình được nhắc đến ở nửa sau đoạn đầu đều là các biến `private` chỉ liên quan đến người bảo trì lớp, chứ không phải người dùng của nó. Tất cả thông tin triển khai này nên được lược bỏ khỏi chú thích.
- Chú thích cũng bao gồm một vài điều hiển nhiên. Chẳng hạn, không cần phải bảo người dùng include `IndexLookup.h`: bất kỳ ai viết mã C++ đều có thể đoán được điều này là cần thiết. Ngoài ra, đoạn chữ `"by providing all necessary information"` (bằng cách cung cấp tất cả thông tin cần thiết) không nói lên điều gì cả, nên có thể lược bỏ.

Một chú thích ngắn gọn hơn cho lớp này là đủ (và đáng ưu tiên hơn):

```cpp
/*
 * This class is used by client applications to make range queries
 * using indexes. Each instance represents a single range query.
 *
 * To start a range query, a client creates an instance of this
 * class. The client can then call getNext() to retrieve the objects
 * in the desired range. For each object returned by getNext(), the
 * caller can invoke getKey(), getKeyLength(), getValue(), and
 * getValueLength() to get information about that object.
 */
```
*(Ghi chú dịch nghĩa: Lớp này được các ứng dụng client sử dụng để thực hiện các truy vấn theo khoảng bằng cách dùng chỉ mục. Mỗi thực thể đại diện cho một truy vấn khoảng đơn lẻ. Để bắt đầu một truy vấn khoảng, client tạo một thực thể của lớp này. Sau đó client có thể gọi `getNext()` để truy xuất các đối tượng trong khoảng mong muốn. Với mỗi đối tượng được trả về bởi `getNext()`, bên gọi có thể gọi `getKey()`, `getKeyLength()`, `getValue()`, và `getValueLength()` để lấy thông tin về đối tượng đó.)*

Đoạn văn cuối cùng của chú thích này không thực sự bắt buộc, vì nó chủ yếu lặp lại thông tin trong chú thích của từng phương thức riêng lẻ. Tuy nhiên, việc có các ví dụ trong tài liệu của lớp để minh họa cách các phương thức phối hợp với nhau có thể rất hữu ích, đặc biệt là đối với các lớp sâu (deep classes) có các mẫu sử dụng không hiển nhiên. Lưu ý rằng chú thích mới không đề cập đến giá trị trả về `NULL` từ `getNext`. Chú thích này không có ý định ghi lại mọi chi tiết của từng phương thức; nó chỉ cung cấp thông tin cấp cao giúp người đọc hiểu cách các phương thức phối hợp với nhau và khi nào thì gọi từng phương thức. Để biết chi tiết, người đọc có thể tham khảo chú thích giao diện của từng phương thức riêng lẻ. Chú thích này cũng không đề cập đến việc máy chủ bị sập; đó là bởi vì sự cố sập máy chủ là vô hình đối với người dùng của lớp này (hệ thống tự động khôi phục từ chúng).

> 🚩 **Cờ Đỏ (Red Flag — Tài liệu triển khai làm nhiễm bẩn giao diện / Implementation Documentation Contaminates Interface):**
> Cờ đỏ này xuất hiện khi tài liệu giao diện, chẳng hạn như tài liệu cho một phương thức, mô tả các chi tiết triển khai vốn không cần thiết để sử dụng thứ đang được viết tài liệu.

Bây giờ hãy xem xét đoạn mã sau, trình bày phiên bản đầu tiên của tài liệu cho phương thức `isReady` trong `IndexLookup`:

```cpp
/**
 * Check if the next object is RESULT_READY. This function is
 * implemented in a DCFT module, each execution of isReady() tries
 * to make small progress, and getNext() invokes isReady() in a
 * while loop, until isReady() returns true.
 *
 * isReady() is implemented in a rule-based approach. We check
 * different rules by following a particular order, and perform
 * certain actions if some rule is satisfied.
 *
 * \return
 *         True means the next Object is available. Otherwise, return
 *         false.
 */
bool IndexLookup::isReady() { ... }
```

Một lần nữa, hầu hết tài liệu này, chẳng hạn như việc nhắc đến `DCFT` và toàn bộ đoạn văn thứ hai, đều liên quan đến phần triển khai, nên nó không thuộc về nơi này; đây là một trong những lỗi phổ biến nhất trong các chú thích giao diện. Một số thông tin trong tài liệu triển khai đó là hữu ích, nhưng nó nên được đặt bên trong phương thức, nơi nó sẽ được tách biệt rõ ràng khỏi tài liệu giao diện. Thêm vào đó, câu đầu tiên của tài liệu khá khó hiểu (`RESULT_READY` nghĩa là gì?) và một số thông tin quan trọng bị thiếu. Cuối cùng, không cần thiết phải mô tả phần triển khai của `getNext` ở đây. Dưới đây là một phiên bản tốt hơn của chú thích:

```cpp
/*
 * Indicates whether an indexed read has made enough progress for
 * getNext to return immediately without blocking. In addition, this
 * method does most of the real work for indexed reads, so it must
 * be invoked (either directly, or indirectly by calling getNext) in
 * order for the indexed read to make progress.
 *
 * \return
 *         True means that the next invocation of getNext will not block
 *         (at least one object is available to return, or the end of the
 *         lookup has been reached); false means getNext may block.
 */
```
*(Ghi chú dịch nghĩa: Cho biết liệu một thao tác đọc theo chỉ mục đã tiến triển đủ để `getNext` trả về ngay lập tức mà không bị chặn (block) hay chưa. Ngoài ra, phương thức này thực hiện phần lớn công việc thực sự cho các thao tác đọc theo chỉ mục, vì vậy nó phải được gọi (trực tiếp, hoặc gián tiếp bằng cách gọi `getNext`) để thao tác đọc theo chỉ mục có thể tiến triển. Giá trị trả về `True` nghĩa là lần gọi `getNext` tiếp theo sẽ không bị chặn (có ít nhất một đối tượng sẵn sàng để trả về, hoặc đã đi đến cuối phép tra cứu); `false` nghĩa là `getNext` có thể bị chặn.)*

Phiên bản chú thích này cung cấp thông tin chính xác hơn về ý nghĩa của `"ready"` (sẵn sàng), đồng thời cung cấp thông tin quan trọng rằng phương thức này cuối cùng phải được gọi nếu muốn việc truy xuất theo chỉ mục tiến triển về phía trước.

## 13.6 Chú thích triển khai: cái gì và tại sao, không phải như thế nào (Implementation comments: what and why, not how)

Chú thích triển khai là những chú thích xuất hiện bên trong các phương thức để giúp người đọc hiểu cách chúng hoạt động ở bên trong. Hầu hết các phương thức đủ ngắn và đơn giản đến mức không cần bất kỳ chú thích triển khai nào: chỉ cần dựa vào mã nguồn và chú thích giao diện là đã dễ dàng hình dung ra cách phương thức hoạt động.

Mục tiêu chính của chú thích triển khai là giúp người đọc hiểu **đoạn mã đang làm gì** (chứ không phải *làm như thế nào*). Một khi người đọc đã biết đoạn mã đang cố gắng làm gì, việc hiểu cách đoạn mã hoạt động thường rất dễ dàng. Đối với các phương thức ngắn, đoạn mã chỉ làm một việc duy nhất, vốn đã được mô tả trong chú thích giao diện của nó, nên không cần chú thích triển khai. Các phương thức dài hơn có nhiều khối mã làm những việc khác nhau như một phần của nhiệm vụ tổng thể của phương thức. Hãy thêm một chú thích trước mỗi khối chính để cung cấp một mô tả cấp cao (trừu tượng hơn) về việc khối đó làm. Dưới đây là một ví dụ:

```cpp
// Phase 1: Scan active RPCs to see if any have completed.
```
*(Ghi chú dịch nghĩa: Giai đoạn 1: Quét các RPC đang hoạt động để xem có cái nào đã hoàn thành chưa.)*

Đối với các vòng lặp, việc có một chú thích trước vòng lặp mô tả những gì xảy ra trong mỗi lần lặp sẽ rất hữu ích:

```cpp
// Each iteration of the following loop extracts one request from
// the request message, increments the corresponding object, and
// appends a response to the response message.
```
*(Ghi chú dịch nghĩa: Mỗi lần lặp của vòng lặp sau trích xuất một yêu cầu từ thông điệp yêu cầu, tăng giá trị của đối tượng tương ứng, và nối một phản hồi vào thông điệp phản hồi.)*

Lưu ý cách chú thích này mô tả vòng lặp ở một cấp độ trừu tượng và trực quan hơn; nó không đi vào bất kỳ chi tiết nào về cách trích xuất một yêu cầu từ thông điệp yêu cầu hay cách đối tượng được tăng giá trị. Chú thích cho vòng lặp chỉ cần thiết đối với những vòng lặp dài hoặc phức tạp, nơi không hiển nhiên rằng vòng lặp đang làm gì; nhiều vòng lặp đủ ngắn và đơn giản để hành vi của chúng đã tự hiển nhiên rồi.

Ngoài việc mô tả đoạn mã đang làm gì, chú thích triển khai còn hữu ích để giải thích **tại sao**. Nếu có những khía cạnh lắt léo trong mã nguồn mà sẽ không hiển nhiên khi đọc, bạn nên viết tài liệu cho chúng. Ví dụ, nếu một bản sửa lỗi (bug fix) đòi hỏi phải thêm một đoạn mã mà mục đích của nó không hoàn toàn hiển nhiên, hãy thêm một chú thích mô tả lý do tại sao đoạn mã đó lại cần thiết. Đối với các bản sửa lỗi đã có một báo cáo lỗi được viết kỹ lưỡng mô tả vấn đề, chú thích có thể dẫn chiếu đến mã vấn đề trong cơ sở dữ liệu theo dõi lỗi thay vì lặp lại toàn bộ chi tiết của nó (`"Fixes RAM-436, related to device driver crashes in Linux 2.4.x"`). Các lập trình viên có thể tra cứu trong cơ sở dữ liệu lỗi để biết thêm chi tiết (đây là một ví dụ về việc tránh trùng lặp trong chú thích, sẽ được thảo luận ở Chương 16).

Đối với các phương thức dài hơn, việc viết chú thích cho một vài biến cục bộ quan trọng nhất có thể rất hữu ích. Tuy nhiên, hầu hết các biến cục bộ không cần tài liệu nếu chúng có tên gọi tốt. Nếu tất cả các lần sử dụng của một biến đều nằm gọn trong phạm vi vài dòng gần nhau, thường rất dễ hiểu mục đích của biến mà không cần chú thích. Trong trường hợp này, việc để người đọc tự đọc mã để hiểu ý nghĩa của biến là hoàn toàn ổn. Tuy nhiên, nếu biến được sử dụng trải dài trên một đoạn mã lớn, thì bạn nên cân nhắc thêm một chú thích để mô tả biến đó. Khi viết tài liệu cho biến, hãy tập trung vào việc biến đó *đại diện cho cái gì*, chứ không phải cách nó bị thao túng trong mã nguồn.

## 13.7 Các quyết định thiết kế liên mô-đun (Cross-module design decisions)

Trong một thế giới hoàn hảo, mọi quyết định thiết kế quan trọng đều được đóng gói (encapsulate) bên trong một lớp duy nhất. Đáng tiếc là các hệ thống thực tế chắc chắn sẽ nảy sinh những quyết định thiết kế ảnh hưởng đến nhiều lớp. Chẳng hạn, thiết kế của một giao thức mạng sẽ ảnh hưởng đến cả bên gửi và bên nhận, và những thứ này có thể được triển khai ở những nơi khác nhau. Các quyết định liên mô-đun thường phức tạp và tinh vi, và chúng là nguyên nhân gây ra nhiều lỗi (bug), vì vậy việc có tài liệu tốt cho chúng là cực kỳ quan trọng.

Thách thức lớn nhất với tài liệu liên mô-đun là **tìm ra một nơi để đặt nó sao cho các lập trình viên sẽ tự nhiên tìm thấy**. Đôi khi có một nơi trung tâm hiển nhiên để đặt loại tài liệu này. Chẳng hạn, hệ thống lưu trữ RAMCloud định nghĩa một giá trị `Status`, được trả về bởi mỗi yêu cầu để báo hiệu thành công hay thất bại. Việc thêm một `Status` cho một điều kiện lỗi mới đòi hỏi phải chỉnh sửa nhiều tệp khác nhau (một tệp ánh xạ giá trị `Status` sang các ngoại lệ, một tệp khác cung cấp thông điệp dễ đọc cho con người ứng với từng `Status`, v.v.). May mắn thay, có một nơi hiển nhiên mà các lập trình viên chắc chắn phải tìm đến khi thêm một giá trị trạng thái mới, đó là phần khai báo của enum `Status`. Chúng tôi đã tận dụng điều này bằng cách thêm chú thích ngay trong enum đó để chỉ ra tất cả những nơi khác cũng phải được chỉnh sửa:

```cpp
typedef enum Status {
      STATUS_OK = 0,
      STATUS_UNKNOWN_TABLET                = 1,
      STATUS_WRONG_VERSION                 = 2,
      ...
      STATUS_INDEX_DOESNT_EXIST            = 29,
      STATUS_INVALID_PARAMETER             = 30,
      STATUS_MAX_VALUE                     = 30,
      // Note: if you add a new status value you must make the following
      // additional updates:
      // (1)  Modify STATUS_MAX_VALUE to have a value equal to the
      //      largest defined status value, and make sure its definition
      //      is the last one in the list. STATUS_MAX_VALUE is used
      //      primarily for testing.
      // (2)  Add new entries in the tables "messages" and "symbols" in
      //      Status.cc.
      // (3)  Add a new exception class to ClientException.h
      // (4)  Add a new "case" to ClientException::throwException to map
      //      from the status value to a status-specific ClientException
      //      subclass.
      // (5)  In the Java bindings, add a static class for the exception
      //      to ClientException.java
      // (6)  Add a case for the status of the exception to throw the
      //      exception in ClientException.java
      // (7)  Add the exception to the Status enum in Status.java, making
      //      sure the status is in the correct position corresponding to
      //      its status code.
}
```
*(Ghi chú dịch nghĩa: Chú ý: nếu bạn thêm một giá trị `status` mới, bạn phải thực hiện các cập nhật bổ sung sau: (1) Sửa `STATUS_MAX_VALUE` bằng giá trị lớn nhất đã định nghĩa và đảm bảo nó nằm cuối danh sách; (2) Thêm mục mới vào bảng `messages` và `symbols` trong `Status.cc`; (3) Thêm lớp ngoại lệ mới vào `ClientException.h`; (4) Thêm nhánh `case` vào `ClientException::throwException`; (5)–(7) Cập nhật tương ứng cho các lớp ngoại lệ và `Status` enum trong Java bindings.)*

Các giá trị trạng thái mới sẽ được thêm vào cuối danh sách hiện có, vì vậy các chú thích cũng được đặt ở cuối, nơi chúng có khả năng được nhìn thấy cao nhất.

Đáng tiếc là trong nhiều trường hợp, không có một nơi trung tâm hiển nhiên nào để đặt tài liệu liên mô-đun. Một ví dụ từ hệ thống lưu trữ RAMCloud là đoạn mã xử lý các máy chủ "zombie" — những máy chủ mà hệ thống tin rằng đã bị sập, nhưng thực tế vẫn đang chạy. Việc vô hiệu hóa các máy chủ zombie đòi hỏi mã nguồn ở nhiều mô-đun khác nhau, và các đoạn mã này đều phụ thuộc lẫn nhau. Không có đoạn mã nào trong số đó là một nơi trung tâm hiển nhiên để đặt tài liệu. Một khả năng là sao chép từng phần của tài liệu vào mỗi vị trí phụ thuộc vào nó. Tuy nhiên, cách này rất cồng kềnh, và rất khó giữ cho tài liệu như vậy luôn được cập nhật khi hệ thống tiến hóa. Một cách khác là đặt tài liệu tại một trong những nơi cần nó, nhưng trong trường hợp này các lập trình viên khó có thể nhìn thấy tài liệu hoặc biết phải tìm nó ở đâu.

Gần đây tôi đã thử nghiệm một cách tiếp cận trong đó các vấn đề liên mô-đun được viết thành tài liệu trong một tệp trung tâm có tên là `designNotes`. Tệp này được chia thành các mục được gắn nhãn rõ ràng, mỗi mục cho một chủ đề lớn. Ví dụ, đây là một đoạn trích từ tệp đó:

```text
...
Zombies
-------
A zombie is a server that is considered dead by the rest of the
cluster; any data stored on the server has been recovered and will
be managed by other servers. However, if a zombie is not actually
dead (e.g., it was just disconnected from the other servers for a
while) two forms of inconsistency can arise:
* A zombie server must not serve read requests once replacement servers
have taken over; otherwise it may return stale data that does not
reflect writes accepted by the replacement servers.
* The zombie server must not accept write requests once replacement
servers have begun replaying its log during recovery; if it does,
these writes may be lost (the new values may not be stored on the
replacement servers and thus will not be returned by reads).
RAMCloud uses two techniques to neutralize zombies. First,
...
```
*(Ghi chú dịch nghĩa: Máy chủ zombie là máy chủ bị phần còn lại của cụm coi là đã chết; mọi dữ liệu lưu trên đó đã được khôi phục và sẽ do các máy chủ khác quản lý. Tuy nhiên, nếu một zombie thực ra chưa chết (ví dụ chỉ bị mất kết nối tạm thời), hai dạng bất nhất quán có thể xảy ra: (1) Không được phục vụ yêu cầu đọc sau khi máy chủ thay thế tiếp quản, nếu không sẽ trả về dữ liệu cũ; (2) Không được chấp nhận yêu cầu ghi sau khi máy chủ thay thế bắt đầu phát lại nhật ký khôi phục, nếu không các lượt ghi này có thể bị mất.)*

Sau đó, trong bất kỳ đoạn mã nào liên quan đến một trong những vấn đề này sẽ có một chú thích ngắn dẫn chiếu đến tệp `designNotes`:

```cpp
// See "Zombies" in designNotes.
```

Với cách tiếp cận này, chỉ có một bản sao duy nhất của tài liệu và các lập trình viên tương đối dễ dàng tìm thấy nó khi cần. Tuy nhiên, cách này có nhược điểm là tài liệu không nằm gần bất kỳ đoạn mã nào phụ thuộc vào nó, vì vậy có thể khó giữ cho nó luôn cập nhật khi hệ thống phát triển.

> 📝 **Giải thích thêm (không có trong sách):**
> Trong thực tế phát triển phần mềm hiện đại, mô hình `designNotes` của Ousterhout thường được hiện thực hóa dưới dạng **ADR (Architecture Decision Records)** nằm ngay trong kho mã nguồn (ví dụ thư mục `docs/adr/`), kết hợp với các chú thích ngắn trỏ về mã ADR cụ thể tại các điểm chạm trong mã nguồn (ví dụ: `// See ADR-014 for why idempotency keys expire after 24h`). Nhờ nằm chung trong Git repository, tài liệu tiến hóa cùng với pull request và dễ được kiểm tra trong quá trình code review.

## 13.8 Kết luận (Conclusion)

Mục tiêu của chú thích là đảm bảo cấu trúc và hành vi của hệ thống trở nên hiển nhiên đối với người đọc, để họ có thể nhanh chóng tìm thấy thông tin mình cần và thực hiện các chỉnh sửa cho hệ thống với sự tự tin rằng chúng sẽ hoạt động đúng. Một phần thông tin này có thể được biểu diễn trong mã nguồn theo cách tự nó đã hiển nhiên với người đọc, nhưng vẫn có một lượng đáng kể thông tin không thể dễ dàng suy ra từ mã nguồn. Chú thích sẽ lấp đầy phần thông tin đó.

Khi tuân theo quy tắc rằng chú thích nên mô tả những điều không hiển nhiên từ mã nguồn, từ **"hiển nhiên" được xét dưới góc nhìn của một người lần đầu tiên đọc mã nguồn của bạn (chứ không phải bạn)**. Khi viết chú thích, hãy thử đặt mình vào tâm thế của người đọc và tự hỏi những điều cốt lõi nào mà anh ấy hoặc cô ấy sẽ cần biết. Nếu mã nguồn của bạn đang được đánh giá (code review) và một người đánh giá bảo với bạn rằng một chỗ nào đó không hiển nhiên, đừng tranh cãi với họ; **nếu một người đọc thấy nó không hiển nhiên, thì tức là nó không hiển nhiên**. Thay vì tranh cãi, hãy cố gắng hiểu xem họ thấy bối rối ở điểm nào và xem bạn có thể làm rõ điều đó không, bằng chú thích tốt hơn hoặc bằng mã nguồn tốt hơn.

## 13.9 Giải đáp các câu hỏi ở Mục 13.5 (Answers to questions from Section 13.5)

Một lập trình viên có cần biết từng mẩu thông tin sau đây để sử dụng lớp `IndexLookup` không?

1. **Định dạng của các thông điệp mà lớp `IndexLookup` gửi tới các máy chủ nắm giữ chỉ mục và đối tượng.**  
   **Không:** đây là một chi tiết triển khai cần được ẩn bên trong lớp.
2. **Hàm so sánh được sử dụng để xác định xem một đối tượng cụ thể có nằm trong khoảng mong muốn hay không (việc so sánh được thực hiện bằng số nguyên, số thực dấu phẩy động, hay chuỗi ký tự?).**  
   **Có:** người dùng của lớp cần biết thông tin này.
3. **Cấu trúc dữ liệu được sử dụng để lưu trữ các chỉ mục trên máy chủ.**  
   **Không:** thông tin này nên được đóng gói trên các máy chủ; thậm chí bản thân phần triển khai của `IndexLookup` cũng không cần phải biết điều này.
4. **Liệu `IndexLookup` có phát hành nhiều yêu cầu tới các máy chủ khác nhau một cách đồng thời hay không.**  
   **Có thể:** nếu `IndexLookup` sử dụng các kỹ thuật đặc biệt để cải thiện hiệu năng, thì tài liệu nên cung cấp một số thông tin cấp cao về điều này, vì người dùng có thể quan tâm đến hiệu năng.
5. **Cơ chế xử lý khi máy chủ bị sập.**  
   **Không:** RAMCloud tự động khôi phục từ các sự cố sập máy chủ, nên các sự cố sập không hiển thị ra với phần mềm ở tầng ứng dụng; do đó, không cần đề cập đến sự cố sập trong tài liệu giao diện cho `IndexLookup`. Nếu các sự cố sập bị phản ánh lên tầng ứng dụng, thì tài liệu giao diện sẽ cần mô tả cách chúng biểu hiện ra bên ngoài (nhưng không mô tả chi tiết về cách cơ chế khôi phục hoạt động).

## Bảng thuật ngữ chương (Chapter Glossary)

| Thuật ngữ (English) | Tiếng Việt | Giải thích ngắn gọn |
| :--- | :--- | :--- |
| **Interface comment** | Chú thích giao diện | Khối chú thích trước khai báo lớp/hàm mô tả sự trừu tượng, tham số, giá trị trả về, tác dụng phụ, ngoại lệ và tiền điều kiện. |
| **Implementation comment** | Chú thích triển khai | Chú thích bên trong thân phương thức giải thích *cái gì* (what) và *tại sao* (why) đoạn mã làm vậy, không giải thích *như thế nào* (how). |
| **Data structure member comment** | Chú thích thành viên cấu trúc dữ liệu | Chú thích bên cạnh biến thực thể hoặc biến tĩnh để làm rõ đơn vị, bất biến, và ý nghĩa của biến (tư duy bằng danh từ). |
| **Cross-module comment** | Chú thích liên mô-đun | Chú thích ghi lại các quyết định thiết kế và sự phụ thuộc trải dài qua nhiều mô-đun khác nhau. |
| **Side effect** | Tác dụng phụ | Bất kỳ hệ quả nào của phương thức ảnh hưởng đến trạng thái hoặc hành vi tương lai của hệ thống ngoài giá trị trả về. |
| **Precondition** | Tiền điều kiện | Điều kiện mà bên gọi phải đảm bảo thỏa mãn trước khi gọi phương thức. |
| **Invariant** | Bất biến | Tính chất luôn luôn đúng đối với một biến hoặc cấu trúc dữ liệu trong suốt vòng đời của nó. |
| **Shallow class / method** | Lớp / phương thức nông | Lớp hoặc phương thức có giao diện phức tạp so với phần triển khai, khiến chú thích giao diện buộc phải lộ chi tiết triển khai. |
