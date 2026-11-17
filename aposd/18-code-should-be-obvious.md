# CHƯƠNG 18: Mã nguồn phải rõ ràng (Chapter 18: Code Should be Obvious)

Sự mơ hồ (obscurity) là một trong hai nguyên nhân chính gây ra độ phức tạp đã được mô tả ở Mục 2.3. Sự mơ hồ xảy ra khi các thông tin quan trọng về một hệ thống không hiện ra rõ ràng đối với các lập trình viên mới. Giải pháp cho vấn đề mơ hồ là viết mã nguồn theo cách khiến nó trở nên rõ ràng (obvious); chương này thảo luận về một số yếu tố làm cho mã nguồn trở nên rõ ràng hơn hoặc kém rõ ràng hơn.

Nếu mã nguồn rõ ràng, điều đó có nghĩa là ai đó có thể đọc mã nguồn một cách nhanh chóng, không cần suy nghĩ quá nhiều, và những phỏng đoán đầu tiên của họ về hành vi hoặc ý nghĩa của mã nguồn sẽ chính xác. Nếu mã nguồn rõ ràng, người đọc không cần tốn nhiều thời gian hay công sức để thu thập toàn bộ thông tin họ cần nhằm làm việc với mã nguồn đó. Ngược lại, nếu mã nguồn không rõ ràng, người đọc phải bỏ ra rất nhiều thời gian và năng lượng để hiểu nó. Điều này không chỉ làm giảm hiệu suất của họ mà còn làm tăng khả năng hiểu sai và phát sinh lỗi (bugs). Mã nguồn rõ ràng cần ít chú thích (comments) hơn so với mã nguồn không rõ ràng.

Tính "rõ ràng" nằm trong tâm trí của người đọc: việc nhận ra mã nguồn của người khác không rõ ràng thường dễ hơn nhiều so với việc nhìn thấy vấn đề trong chính mã nguồn của mình. Vì vậy, cách tốt nhất để đánh giá mức độ rõ ràng của mã nguồn là thông qua việc duyệt mã nguồn (code reviews). Nếu ai đó đọc mã nguồn của bạn và nói rằng nó không rõ ràng, thì tức là nó không rõ ràng, bất kể với bạn nó có vẻ sáng tỏ đến đâu. Bằng cách cố gắng tìm hiểu điều gì đã làm cho mã nguồn trở nên khó hiểu, bạn sẽ học được cách viết mã nguồn tốt hơn trong tương lai.

## 18.1 Những yếu tố giúp mã nguồn rõ ràng hơn (Things that make code more obvious)

Hai trong số những kỹ thuật quan trọng nhất để làm cho mã nguồn trở nên rõ ràng đã được thảo luận ở các chương trước. Kỹ thuật đầu tiên là chọn tên gọi tốt (Chương 14). Những cái tên chính xác và giàu ý nghĩa sẽ làm rõ hành vi của mã nguồn và giảm bớt nhu cầu phải viết tài liệu. Nếu một cái tên mơ hồ hoặc đa nghĩa, người đọc sẽ phải đọc xuyên suốt mã nguồn để suy luận ra ý nghĩa của thực thể được đặt tên đó; việc này rất tốn thời gian và dễ gây ra sai sót. Kỹ thuật thứ hai là tính nhất quán (Chương 17). Nếu những việc tương tự luôn được thực hiện theo những cách tương tự nhau, thì người đọc có thể nhận ra các mẫu quen thuộc mà họ đã thấy trước đó và ngay lập tức rút ra các kết luận (an toàn) mà không cần phân tích chi tiết mã nguồn.

Dưới đây là một vài kỹ thuật tổng quát khác giúp mã nguồn trở nên rõ ràng hơn:

**Sử dụng khoảng trắng hợp lý (Judicious use of white space).** Cách định dạng mã nguồn có thể tác động lớn đến mức độ dễ hiểu của nó. Hãy xem xét đoạn tài liệu mô tả tham số dưới đây, trong đó các khoảng trắng đã bị nén lại:

```java
/**
 * ...
 * @param numThreads The number of threads that this manager should
 * spin up in order to manage ongoing connections. The MessageManager
 * spins up at least one thread for every open connection, so this
 * should be at least equal to the number of connections you expect
 * to be open at once. This should be a multiple of that number if
 * you expect to send a lot of messages in a short amount of time.
 * @param handler Used as a callback in order to handle incoming
 * messages on this MessageManager's open connections. See
 * {@code MessageHandler} and {@code handleMessage} for details.
 */
```

Rất khó để nhận ra phần tài liệu của một tham số kết thúc ở đâu và phần tiếp theo bắt đầu từ chỗ nào. Thậm chí không dễ thấy có bao nhiêu tham số hay tên của chúng là gì. Nếu thêm một chút khoảng trắng và thụt lề, cấu trúc đột nhiên trở nên mạch lạc và đoạn tài liệu dễ đọc lướt hơn hẳn:

```java
/**
 * @param numThreads
 *          The number of threads that this manager should spin up in
 *          order to manage ongoing connections. The MessageManager spins
 *          up at least one thread for every open connection, so this
 *          should be at least equal to the number of connections you
 *          expect to be open at once. This should be a multiple of that
 *          number if you expect to send a lot of messages in a short
 *          amount of time.
 * @param handler
 *          Used as a callback in order to handle incoming messages on
 *          this MessageManager's open connections. See
 *          {@code MessageHandler} and {@code handleMessage} for details.
 */
```

Các dòng trống (blank lines) cũng rất hữu ích để phân tách các khối mã lớn bên trong một phương thức, như trong ví dụ sau:

```cpp
void* Buffer::allocAux(size_t numBytes)
{
    // Làm tròn độ dài lên bội số của 8 byte để đảm bảo căn chỉnh bộ nhớ (alignment).
    uint32_t numBytes32 = (downCast<uint32_t>(numBytes) + 7) & ~0x7;
    assert(numBytes32 != 0);

    // Nếu có đủ bộ nhớ tại firstAvailable, hãy dùng vùng đó. Cấp phát ngược
    // từ trên xuống dưới, vì vùng bộ nhớ này được đảm bảo đã căn chỉnh
    // (vùng bộ nhớ ở phía dưới có thể đã được dùng cho các chunk có kích thước thay đổi).
    if (availableLength >= numBytes32) {
        availableLength -= numBytes32;
        return firstAvailable + availableLength;
    }

    // Tiếp theo, kiểm tra xem có khoảng trống dư ở cuối chunk cuối cùng không.
    if (extraAppendBytes >= numBytes32) {
        extraAppendBytes -= numBytes32;
        return lastChunk->data + lastChunk->length + extraAppendBytes;
    }

    // Phải tạo một vùng cấp phát mới; cấp phát không gian bên trong vùng đó.
    uint32_t allocatedLength;
    firstAvailable = getNewAllocation(numBytes32, &allocatedLength);
    availableLength = allocatedLength - numBytes32;
    return firstAvailable + availableLength;
}
```

Cách tiếp cận này đặc biệt hiệu quả nếu dòng đầu tiên sau mỗi dòng trống là một chú thích mô tả khối mã tiếp theo: các dòng trống giúp các chú thích trở nên nổi bật hơn.

Khoảng trắng bên trong một câu lệnh cũng giúp làm rõ cấu trúc của câu lệnh đó. Hãy so sánh hai câu lệnh sau, một câu không có khoảng trắng và một câu có khoảng trắng:

```cpp
for(int pass=1;pass>=0&&!empty;pass--) {
for (int pass = 1; pass >= 0 && !empty; pass--) {
```

**Chú thích (Comments).** Đôi khi không thể tránh khỏi việc phải viết những đoạn mã không rõ ràng. Khi điều đó xảy ra, điều quan trọng là sử dụng chú thích để bù đắp bằng cách cung cấp thông tin còn thiếu. Để làm tốt điều này, bạn phải đặt mình vào vị trí của người đọc và hình dung xem điều gì có khả năng gây bối rối cho họ, cũng như thông tin nào sẽ giải tỏa sự bối rối đó. Phần tiếp theo sẽ trình bày một vài ví dụ.

> 💡 **Ví dụ bổ sung (không có trong sách):**
> Trong thực tế, việc dùng "biến giải thích" (explaining variable) và tách điều kiện phức tạp bằng khoảng trắng cũng làm tăng tính rõ ràng đáng kể. So sánh hai đoạn mã kiểm tra quyền truy cập sau:
> ```python
> # Kém rõ ràng: nhồi nhét mọi điều kiện vào một dòng dài
> if user.is_active and not user.is_banned and (user.role == "admin" or (resource.owner_id == user.id and not resource.is_locked)):
>     allow_edit()
>
> # Rõ ràng hơn: dùng tên biến có ý nghĩa và tách khối hợp lý
> is_valid_user = user.is_active and not user.is_banned
> is_owner_editable = (resource.owner_id == user.id) and not resource.is_locked
>
> if is_valid_user and (user.role == "admin" or is_owner_editable):
>     allow_edit()
> ```
> Chỉ với vài biến trung gian đặt tên rõ ràng và một dòng trống ngăn cách, người đọc hiểu ngay logic nghiệp vụ trong một lần liếc mắt mà không cần phải phân tích biểu thức logic lồng nhau.

## 18.2 Những yếu tố làm mã nguồn kém rõ ràng (Things that make code less obvious)

Có rất nhiều yếu tố có thể làm cho mã nguồn trở nên không rõ ràng; mục này cung cấp một vài ví dụ điển hình. Một số kỹ thuật trong số này, chẳng hạn như lập trình hướng sự kiện, lại rất hữu ích trong nhiều tình huống, vì vậy bạn vẫn có thể phải sử dụng chúng. Khi đó, việc viết thêm tài liệu hướng dẫn (extra documentation) sẽ giúp giảm thiểu sự bối rối cho người đọc.

**Lập trình hướng sự kiện (Event-driven programming).** Trong lập trình hướng sự kiện, một ứng dụng phản hồi lại các tác nhân bên ngoài, chẳng hạn như khi có một gói tin mạng đến hoặc khi người dùng nhấn nút chuột. Một module sẽ chịu trách nhiệm báo cáo các sự kiện đến. Các phần khác của ứng dụng đăng ký quan tâm đến những sự kiện nhất định bằng cách yêu cầu module sự kiện gọi một hàm hoặc phương thức cụ thể khi các sự kiện đó xảy ra.

Lập trình hướng sự kiện khiến luồng điều khiển (flow of control) trở nên khó theo dõi. Các hàm xử lý sự kiện (event handler functions) không bao giờ được gọi trực tiếp; chúng được gọi gián tiếp bởi module sự kiện, thường thông qua con trỏ hàm (function pointer) hoặc interface. Ngay cả khi bạn tìm thấy điểm gọi hàm trong module sự kiện, bạn vẫn không thể biết chính xác hàm cụ thể nào sẽ được thực thi: điều này phụ thuộc vào những handler nào đã được đăng ký tại thời điểm chạy (runtime). Chính vì thế, rất khó để suy luận về mã nguồn hướng sự kiện hoặc tự thuyết phục bản thân rằng nó hoạt động đúng.

Để bù đắp cho sự mơ hồ này, hãy sử dụng chú thích giao diện (interface comment) cho mỗi hàm xử lý sự kiện để chỉ rõ khi nào nó được gọi, như trong ví dụ sau:

```cpp
/**
 * Phương thức này được gọi trong luồng điều phối (dispatch thread) bởi một
 * transport nếu một lỗi ở tầng transport ngăn không cho RPC hoàn thành.
 *
 * (Nguyên văn: This method is invoked in the dispatch thread by a transport if a
 * transport-level error prevents an RPC from completing.)
 */
void
Transport::RpcNotifier::failed() {
    ...
}
```

> 🚩 **Cờ Đỏ (Red Flag — Mã nguồn không rõ ràng / Nonobvious Code):** Nếu ý nghĩa và hành vi của mã nguồn không thể được hiểu thông qua một lần đọc lướt nhanh, đó là một cờ đỏ. Điều này thường có nghĩa là có những thông tin quan trọng không được thể hiện một cách rõ ràng ngay lập tức cho người đọc mã nguồn.

**Các cấu trúc chứa dữ liệu tổng quát (Generic containers).** Nhiều ngôn ngữ lập trình cung cấp các lớp tổng quát để nhóm hai hoặc nhiều phần tử vào một đối tượng duy nhất, chẳng hạn như `Pair` trong Java hoặc `std::pair` trong C++. Những lớp này rất hấp dẫn vì chúng giúp dễ dàng truyền nhiều đối tượng qua lại chỉ bằng một biến duy nhất. Một trong những cách dùng phổ biến nhất là trả về nhiều giá trị từ một phương thức, như trong ví dụ Java sau:

```java
return new Pair<Integer, Boolean>(currentTerm, false);
```

Đáng tiếc là các generic container lại dẫn đến mã nguồn không rõ ràng vì các phần tử được nhóm bên trong mang những cái tên chung chung làm lu mờ ý nghĩa thực sự của chúng. Trong ví dụ trên, phía gọi hàm phải tham chiếu đến hai giá trị trả về thông qua `result.getKey()` và `result.getValue()`, những cái tên hoàn toàn không cho biết manh mối gì về ý nghĩa thực của các giá trị đó. Vì vậy, tốt hơn hết là không nên dùng generic container. Nếu bạn cần một cấu trúc chứa dữ liệu, hãy định nghĩa một lớp (class) hoặc cấu trúc (structure) mới chuyên biệt cho mục đích sử dụng cụ thể đó. Khi ấy, bạn có thể dùng những cái tên giàu ý nghĩa cho các phần tử, đồng thời có thể bổ sung tài liệu mô tả ngay trong phần khai báo — điều mà generic container không thể làm được.

Ví dụ này minh họa một nguyên tắc chung: **phần mềm phải được thiết kế để dễ đọc, chứ không phải để dễ viết (software should be designed for ease of reading, not ease of writing).** Các generic container mang lại sự tiện lợi tức thời cho người viết mã, nhưng chúng lại tạo ra sự bối rối cho tất cả những người đọc mã về sau. Sẽ tốt hơn nhiều nếu người viết mã dành thêm vài phút để định nghĩa một cấu trúc chứa dữ liệu chuyên biệt, nhờ đó mã nguồn tạo ra trở nên rõ ràng hơn.

> 💡 **Ví dụ bổ sung (không có trong sách):**
> Trong Python hoặc TypeScript, tình trạng tương tự xảy ra khi lập trình viên trả về một `tuple` không có tên hoặc một `Pair`/`Record` chung chung:
> ```python
> # Kém rõ ràng: người gọi phải nhớ vị trí 0 và 1 đại diện cho gì
> def get_user_stats(user_id: int) -> tuple[int, bool]:
>     ...
>     return (login_count, is_verified)
>
> stats = get_user_stats(42)
> if stats[1] and stats[0] > 5:  # stats[1] là gì? stats[0] là gì?
>     ...
>
> # Rõ ràng: dùng dataclass hoặc NamedTuple chuyên biệt
> @dataclass(frozen=True)
> class UserStats:
>     login_count: int
>     is_verified: bool
>
> stats = get_user_stats(42)
> if stats.is_verified and stats.login_count > 5:
>     ...
> ```

**Khác biệt về kiểu giữa khai báo và cấp phát (Different types for declaration and allocation).** Hãy xem xét ví dụ Java sau:

```java
private List<Message> incomingMessageList;
...
incomingMessageList = new ArrayList<Message>();
```

Biến được khai báo với kiểu `List`, nhưng giá trị thực tế lại là một `ArrayList`. Đoạn mã này hoàn toàn hợp lệ, vì `List` là lớp cha (superclass/interface) của `ArrayList`, nhưng nó có thể đánh lừa người đọc khi họ chỉ nhìn thấy dòng khai báo mà không thấy dòng cấp phát thực tế. Kiểu thực tế có thể ảnh hưởng đến cách biến được sử dụng (ví dụ: `ArrayList` có các đặc tính về hiệu năng và độ an toàn luồng — thread-safety — khác với các lớp con khác của `List`), vì vậy tốt hơn là nên giữ sự đồng nhất giữa kiểu khi khai báo và kiểu khi cấp phát.

> 📝 **Giải thích thêm (không có trong sách):**
> Lưu ý sắc thái trong lời khuyên của Ousterhout ở đây: trong thiết kế API công khai (public interface) hoặc tham số hàm, việc khai báo kiểu trừu tượng như `List<Message>` thường được ưa chuộng để giấu chi tiết cài đặt (information hiding). Tuy nhiên, đối với một **biến thành viên nội bộ (private field)** hoặc **biến cục bộ** nơi mã nguồn bên trong lớp phụ thuộc trực tiếp vào đặc tính hiệu năng (ví dụ truy cập ngẫu nhiên $O(1)$ của `ArrayList` so với $O(n)$ của `LinkedList`, hay tính đồng bộ của `CopyOnWriteArrayList`), việc khai báo luôn kiểu cụ thể `private ArrayList<Message> incomingMessageList;` giúp người đọc nắm bắt ngay đặc tính thực sự của cấu trúc dữ liệu đó mà không phải đi tìm chỗ khởi tạo (`new ...`).

**Mã nguồn vi phạm kỳ vọng của người đọc (Code that violates reader expectations).** Hãy xem xét đoạn mã sau, vốn là chương trình chính (`main`) của một ứng dụng Java:

```java
public static void main(String[] args) {
    ...
    new RaftClient(myAddress, serverAddresses);
}
```

Hầu hết các ứng dụng đều thoát khi chương trình `main` của chúng kết thúc (return), nên người đọc nhiều khả năng sẽ mặc định rằng điều đó cũng xảy ra ở đây. Tuy nhiên, thực tế không phải vậy. Hàm khởi tạo (constructor) của `RaftClient` tạo ra các luồng (threads) bổ sung, và các luồng này vẫn tiếp tục chạy ngay cả khi luồng `main` của ứng dụng đã kết thúc. Hành vi này cần phải được ghi rõ trong chú thích giao diện (interface comment) của hàm khởi tạo `RaftClient`, nhưng bản thân hành vi đó đủ bất thường để đáng đặt thêm một chú thích ngắn ngay ở cuối hàm `main`. Chú thích đó nên chỉ rõ rằng ứng dụng sẽ tiếp tục thực thi trên các luồng khác. Mã nguồn trở nên rõ ràng nhất khi nó tuân theo các quy ước mà người đọc kỳ vọng; nếu không tuân theo, điều quan trọng là phải ghi chú lại hành vi đó để người đọc không bị bối rối.

## 18.3 Kết luận (Conclusion)

Một cách khác để suy nghĩ về tính rõ ràng (obviousness) là nhìn dưới góc độ thông tin (information). Nếu mã nguồn không rõ ràng, điều đó thường có nghĩa là có những thông tin quan trọng về mã nguồn mà người đọc chưa có: trong ví dụ `RaftClient`, người đọc có thể không biết rằng hàm khởi tạo `RaftClient` đã tạo ra các luồng mới; trong ví dụ `Pair`, người đọc có thể không biết rằng `result.getKey()` trả về số thứ tự của nhiệm kỳ hiện tại (`currentTerm`).

Để làm cho mã nguồn trở nên rõ ràng, bạn phải đảm bảo rằng người đọc luôn có đủ thông tin họ cần để hiểu nó. Bạn có thể thực hiện điều này theo ba cách:
1. **Cách tốt nhất là giảm lượng thông tin cần thiết**, bằng cách sử dụng các kỹ thuật thiết kế như trừu tượng hóa (abstraction) và loại bỏ các trường hợp đặc biệt (eliminating special cases).
2. **Thứ hai, bạn có thể tận dụng những thông tin mà người đọc đã tích lũy được từ các ngữ cảnh khác** (ví dụ: bằng cách tuân theo các quy ước và đáp ứng đúng kỳ vọng thông thường) để người đọc không phải học thêm thông tin mới cho riêng mã nguồn của bạn.
3. **Thứ ba, bạn có thể trình bày trực tiếp các thông tin quan trọng đó cho họ ngay trong mã nguồn**, sử dụng các kỹ thuật như đặt tên tốt và viết chú thích chiến lược (strategic comments).

## Bảng thuật ngữ chương (Chapter Glossary)

| Thuật ngữ (English) | Tiếng Việt | Giải thích ngắn gọn |
| :--- | :--- | :--- |
| Obscurity | Sự mơ hồ | Tình trạng thông tin quan trọng về hệ thống không hiện ra rõ ràng với người đọc mã nguồn. |
| Obvious code | Mã nguồn rõ ràng | Mã mà người đọc có thể hiểu nhanh chóng, chính xác ngay từ phỏng đoán đầu tiên mà không cần tốn nhiều công sức suy luận. |
| Code review | Duyệt mã nguồn | Quy trình người khác đọc và đánh giá mã nguồn của bạn; công cụ hiệu quả nhất để phát hiện mã nguồn thiếu rõ ràng. |
| White space | Khoảng trắng | Dấu cách, thụt lề và dòng trống dùng để phân tách cấu trúc, giúp mã nguồn và tài liệu dễ đọc lướt. |
| Event-driven programming | Lập trình hướng sự kiện | Mô hình lập trình phản hồi các sự kiện bên ngoài thông qua các hàm xử lý (handler) được gọi gián tiếp, dễ làm luồng điều khiển khó theo dõi. |
| Generic container | Cấu trúc chứa tổng quát | Các kiểu dữ liệu chung chung như `Pair`, `std::pair`, `tuple` làm mất ý nghĩa cụ thể của từng phần tử bên trong. |
| Reader expectations | Kỳ vọng của người đọc | Những giả định tự nhiên mà lập trình viên có sẵn dựa trên quy ước chung của ngôn ngữ hoặc hệ thống. |
