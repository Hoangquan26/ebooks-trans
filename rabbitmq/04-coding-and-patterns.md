# Chương 4: Giải quyết vấn đề với Rabbit – lập trình và các mẫu thiết kế (Solving problems with Rabbit: coding and patterns)

## Mục lục chương

- [Chương này đề cập đến](#chương-này-đề-cập-đến-this-chapter-covers)
- [Mở đầu](#mở-đầu)
- [4.1 Câu chuyện về tách rời: điều gì đẩy chúng ta đến với messaging (A decoupling story: what pushes us to messaging)](#41-câu-chuyện-về-tách-rời-điều-gì-đẩy-chúng-ta-đến-với-messaging-a-decoupling-story-what-pushes-us-to-messaging)
  - [4.1.1 Tư duy bất đồng bộ (tách yêu cầu và hành động)](#411-tư-duy-bất-đồng-bộ-tách-yêu-cầu-và-hành-động-an-asynchronous-state-of-mind-separating-requests-and-actions)
  - [4.1.2 Có khả năng mở rộng: một thế giới không cần bộ cân bằng tải](#412-có-khả-năng-mở-rộng-một-thế-giới-không-cần-bộ-cân-bằng-tải-affording-scale-a-world-without-load-balancers)
  - [4.1.3 API không tốn công sức: sao phải bó buộc vào một ngôn ngữ?](#413-api-không-tốn-công-sức-sao-phải-bó-buộc-vào-một-ngôn-ngữ-zero-effort-apis-why-be-locked-into-just-one-language)
- [4.2 Các mô hình "bắn và quên" (Fire-and-forget models)](#42-các-mô-hình-bắn-và-quên-fire-and-forget-models)
  - [4.2.1 Gửi cảnh báo (Sending alerts)](#421-gửi-cảnh-báo-sending-alerts)
  - [4.2.2 Xử lý song song (Parallel processing)](#422-xử-lý-song-song-parallel-processing)
- [4.3 Hãy nhớ đến tôi: RPC qua RabbitMQ và chờ câu trả lời (Remember me: RPC over RabbitMQ and waiting for answers)](#43-hãy-nhớ-đến-tôi-rpc-qua-rabbitmq-và-chờ-câu-trả-lời-remember-me-rpc-over-rabbitmq-and-waiting-for-answers)
  - [4.3.1 Queue riêng tư và gửi phản hồi (Private queues and sending acknowledgements)](#431-queue-riêng-tư-và-gửi-phản-hồi-private-queues-and-sending-acknowledgements)
  - [4.3.2 JSON RPC đơn giản với reply_to (Simple JSON RPC with reply_to)](#432-json-rpc-đơn-giản-với-reply_to-simple-json-rpc-with-reply_to)
- [4.4 Tóm tắt (Summary)](#44-tóm-tắt-summary)
- [Bảng thuật ngữ của chương](#bảng-thuật-ngữ-của-chương)

---

## Chương này đề cập đến (This chapter covers)

- Thiết kế ứng dụng theo hướng nhắn tin (*designing applications toward messaging*)
- Các mẫu nhắn tin (*messaging patterns*)
- Các mô hình "bắn và quên" (*fire-and-forget models*)
- RPC với RabbitMQ (*RPC with RabbitMQ*)

## Mở đầu

Đến lúc này bạn đã biết cách cài đặt, cấu hình và thậm chí vận hành Rabbit trong môi trường production (môi trường chạy thật). Đã đến lúc chúng ta bắt tay vào viết code. Nhưng trước hết, bạn cần hiểu những vấn đề mà mình đang cố giải quyết khi đưa messaging (cơ chế nhắn tin – trao đổi thông điệp giữa các thành phần phần mềm) vào ứng dụng.

Giống như rất nhiều người tình cờ khám phá ra RabbitMQ, hai tác giả "đáng yêu" của cuốn sách này ban đầu cũng không hề đi tìm một hàng đợi thông điệp (*message queue*); chúng tôi đang đi tìm cách giải quyết một bài toán **tách rời** (*decoupling* – làm cho các thành phần không phụ thuộc chặt vào nhau). Làm thế nào để lấy một tác vụ tốn nhiều thời gian ra khỏi ứng dụng đã kích hoạt nó (nhờ vậy ứng dụng đó được rảnh tay để phục vụ các yêu cầu khác)? Và làm sao để "dán" các ứng dụng viết bằng những ngôn ngữ khác nhau, như PHP và Erlang, lại với nhau để chúng hoạt động như một hệ thống duy nhất?

Nhìn qua thì đây có vẻ là hai vấn đề khác nhau, nhưng chúng có chung một cốt lõi: **tách yêu cầu (*request*) ra khỏi hành động (*action*)**. Nói cách khác, cả hai vấn đề đều đòi hỏi chuyển từ mô hình lập trình đồng bộ (*synchronous*) sang mô hình bất đồng bộ (*asynchronous*).

Thông thường, khi lập trình viên nghe đến "lập trình bất đồng bộ", họ hoặc là bỏ chạy lên núi, hoặc là reo lên: "Hay quá. Giống Node.js phải không?" Đôi khi là cả hai. Vấn đề với những cách tiếp cận lập trình bất đồng bộ thông thường là chúng mang tính "được ăn cả, ngã về không" (*all-or-nothing*): hoặc bạn viết lại toàn bộ code sao cho không chỗ nào bị chặn (*block*), hoặc bạn chỉ đang phí thời gian.

RabbitMQ cho bạn một con đường khác. Nó cho phép bạn "bắn" một yêu cầu đi để được xử lý ở nơi khác, nhờ đó ứng dụng đồng bộ của bạn có thể quay lại với công việc đang làm. Khi bắt đầu dùng messaging, bạn có thể đạt được phần lớn lợi ích của lập trình bất đồng bộ thuần túy mà không phải vứt bỏ code hay kiến thức sẵn có.

Trong chương này, chúng tôi sẽ cho bạn thấy lập trình bất đồng bộ có nghĩa là gì trong thế giới Rabbit. Cụ thể, chúng tôi sẽ chỉ cho bạn cách dùng Rabbit để giải quyết một loạt bài toán thực tế: từ xử lý ảnh (xử lý song song – *parallel processing*), cảnh báo (thông báo – *notifications*), cho đến dùng RabbitMQ để thực hiện các lời gọi thủ tục từ xa phân tán (*distributed remote procedure calls* – các API) đơn giản như ăn bánh. Chúng ta sẽ bắt đầu bằng việc học một số mô hình (*paradigm*) nhắn tin cơ bản rồi lao vào code. Cùng bắt đầu "tách rời" nào!

> 📝 **Giải thích thêm (không có trong sách):** "Đồng bộ" nghĩa là bên gọi phải **đứng chờ** cho tới khi công việc làm xong mới đi tiếp. "Bất đồng bộ" nghĩa là bên gọi **giao việc rồi đi làm việc khác ngay**, kết quả (nếu cần) sẽ được báo lại sau. Điểm hay của RabbitMQ là bạn không cần viết lại cả chương trình theo kiểu event-loop như Node.js: code PHP/Python cũ vẫn chạy tuần tự như bình thường, chỉ có điều thay vì *tự làm* tác vụ nặng thì nó *gửi một message* vào RabbitMQ (mất vài mili-giây) rồi trả lời người dùng ngay.

---

## 4.1 Câu chuyện về tách rời: điều gì đẩy chúng ta đến với messaging (A decoupling story: what pushes us to messaging)

Bạn làm điều này suốt. Bạn viết ra ứng dụng web mới nhất, tuyệt vời nhất của mình (một app đặt lịch dắt chó Chihuahua đi dạo), và quyết định rằng cách nhanh nhất là nhận đơn đặt hàng từ web rồi nhét thẳng vào cơ sở dữ liệu. Nghe rất hợp lý. Nhét một bản ghi nhỏ vào cơ sở dữ liệu thì tốn được bao nhiêu thời gian chứ? Chưa kể code lại rất đơn giản.

Vấn đề là: chuyện gì xảy ra khi bạn mở rộng ra toàn quốc và giờ phải đặt lịch cho 100.000 con Chihuahua mỗi giờ? Hoặc hay hơn nữa, bạn quyết định muốn lưu dữ liệu ở hai nơi (phải lưu trữ – *archive* – các yêu cầu đó chứ)? Đoán xem? Đã đến lúc phải xé bỏ toàn bộ đống code bạn đã debug cẩn thận. Gắn chặt (*coupling*) một ứng dụng trực tiếp vào kho lưu trữ thường là công thức dẫn đến cảnh "xé ra rồi thay mới" (*rip-and-replace*) về sau, và đó chính là chỗ messaging có thể giúp bạn.

> 💡 **Ví dụ bổ sung (không có trong sách):** Hình dung code "gắn chặt" ban đầu (giả mã Python):
>
> ```python
> def handle_order(request):
>     db.insert("orders", request.data)   # ghi thẳng vào DB, người dùng phải chờ
>     return "OK"
> ```
>
> Khi cần lưu thêm vào kho lưu trữ thứ hai, bạn phải sửa chính hàm này (thêm `archive.insert(...)`), người dùng chờ lâu hơn, và nếu kho thứ hai bị lỗi thì cả việc nhận đơn cũng lỗi. Còn với messaging:
>
> ```python
> def handle_order(request):
>     channel.basic_publish(exchange="orders", routing_key="new", body=json.dumps(request.data))
>     return "OK"   # trả lời ngay
> ```
>
> Việc ghi DB, ghi kho lưu trữ… do các consumer riêng làm. Muốn thêm nơi lưu thứ hai chỉ cần thêm một consumer mới, **không động** vào code nhận đơn.

### 4.1.1 Tư duy bất đồng bộ (tách yêu cầu và hành động) (An asynchronous state of mind (separating requests and actions))

Bạn hoạt động theo kiểu đồng bộ thường xuyên đến mức nào? Nếu bạn gọi một chiếc pizza, bạn có đứng chờ nó tới rồi mới làm việc khác không? Tất nhiên là không. Bạn xem TV, đọc sách, hoặc có thể dành chút thời gian trò chuyện chất lượng với người yêu. Hiếm khi bạn "tạm dừng cuộc sống" để chờ phản hồi cho các yêu cầu của mình. Bạn làm nhiều việc cùng lúc, nhờ đó cuộc sống của bạn có thể "mở rộng quy mô" và bạn làm được nhiều việc hơn. Ứng dụng của bạn cũng cần cách tiếp cận tương tự.

Vậy tại sao ngay từ đầu bạn lại thiết kế ứng dụng theo kiểu đồng bộ? Phần lớn là vì bạn nghĩ về **toàn bộ công việc** thay vì các **tác vụ nhỏ** cấu thành nên nó. Bạn nghĩ: "app của tôi cần đặt một lịch hẹn cho Chihuahua". Nhưng thực tế là app của bạn cần:

1. nhận một yêu cầu đặt lịch;
2. sau đó lưu yêu cầu đó vào cơ sở dữ liệu;
3. rồi báo cho người dắt chó gần nhất;
4. và cuối cùng báo cho khách hàng biết họ đã được xếp lịch.

Ngay cả khi bạn làm app đa luồng (*multithreaded*), bạn vẫn đã giới hạn nghiêm trọng tốc độ nhận đơn, vì mỗi luồng phải chờ bản ghi được lưu xong và người dắt chó được báo xong. Thay vào đó, bạn nên xem bốn bước này như thuộc về **hai ứng dụng riêng biệt**: một ứng dụng nhận yêu cầu, và một ứng dụng xử lý yêu cầu.

Mượn một phép so sánh tuyệt vời của Gregor Hohpe, ta có thể gọi đây là **mô hình Coffee Bean** (Coffee Bean & Tea Leaf là một chuỗi quán cà phê ở California).

Khi bạn gọi một ly chai latte, bạn không đứng chờ ở quầy thu ngân cho tới khi đồ uống làm xong. Thay vào đó, Coffee Bean tách thao tác **nhận đơn** ra khỏi thao tác **pha chế**. Người nhận đơn ghi lại yêu cầu của bạn (và thu tiền – "dinero" của bạn), rồi chuyển một thông điệp cho các barista (người pha chế) cho biết bạn đã gọi gì. Sau đó bạn chờ đồ uống được pha, trong khi người nhận đơn được rảnh tay để nhận đơn kế tiếp.

Phần quan trọng nhất của hoạt động kinh doanh là thu được tiền, nên bằng cách tách nhận đơn khỏi xử lý đơn, Coffee Bean đã tối đa hóa số đơn họ nhận được mỗi phút. Tương tự, nếu lượng cà phê tồn đọng chờ pha (*backlog*) tăng quá cao, họ có thể thêm barista để giảm tồn đọng mà không cần thay đổi số người nhận đơn. Bằng cách tách rời quy trình (tách yêu cầu và hành động), họ đã tăng lượng công việc hoàn thành được với cùng số nhân viên, và giúp việc mở rộng quy mô khi cần trở nên dễ dàng. Messaging làm điều tương tự cho ứng dụng của bạn.

Vậy hãy phân tích lại app Chihuahua của bạn với tư duy tách rời. Hình 4.1 cho thấy các bước để hoàn tất một đơn đặt dắt chó đi dạo.

**Hình 4.1 – Các bước để hoàn tất một đơn đặt dắt chó đi dạo**

![Hình 4.1 – trang sách gốc chứa hình](images/pages/page-087.png)

![Hình 4.1 – thành phần ảnh trích (khung bo tròn màu xanh, dùng làm nền ô trong sơ đồ)](images/04-coding-and-patterns/p087-9.png)

*(Ảnh trích `p087-9.png` chỉ là một mảnh trang trí – khung bo tròn có bóng đổ – của sơ đồ; một ảnh khác trên trang này không trích xuất được. Toàn bộ sơ đồ xem ở ảnh trang render phía trên.)*

Sơ đồ vẽ lại:

```mermaid
flowchart TD
    subgraph APP["Ứng dụng đặt dắt chó (Dog walking order app)"]
        subgraph REQ["order request – phần nhận yêu cầu"]
            A["Nhận yêu cầu đặt lịch (Receive the scheduling request)"]
            D["Xác nhận với khách hàng rằng yêu cầu đã được thực hiện thành công (Confirm to the customer that the request was successfully filled)"]
        end
        subgraph PROC["order processing – phần xử lý đơn"]
            B["Lưu yêu cầu vào cơ sở dữ liệu hoặc kho lưu trữ lâu dài khác (Store the request in a database or other permanent storage)"]
            C["Báo cho người dắt chó biết họ có khách (Alert the dog walker they've got a customer)"]
        end
    end
    A --> B --> C --> D
```

Mô tả hình: toàn bộ là **một** ứng dụng duy nhất ("Dog walking order app"). Khối xanh dương "order request" (nhận yêu cầu) bao ngoài; bên trong có khối xanh lá "order processing" (xử lý đơn). Luồng đi tuần tự từ trên xuống: Nhận yêu cầu → Lưu vào cơ sở dữ liệu → Báo người dắt chó → Xác nhận với khách. Vì tất cả nằm trong một app, khách phải chờ cả chuỗi bốn bước.

Nếu muốn tăng khả năng mở rộng (*scalability*) và tính linh hoạt của app, bạn cần tách nó thành hai ứng dụng khác nhau: **dog_walk_order** và **dog_walk_process**, như trong hình 4.2.

- `dog_walk_order` nằm trên internet và nhận các yêu cầu web để đặt lịch dắt chó. Khi nhận một yêu cầu, `dog_walk_order` tạo một message AMQP mới và publish (xuất bản/gửi) nó vào exchange `chihuahua_scheduling` trên Rabbit. Sau đó `dog_walk_order` có thể tạm để Khách hàng A chờ và đi nhận các yêu cầu khác.
- Trong khi đó, `dog_walk_process` lắng nghe một queue của Rabbit và nhận message chứa đơn đặt lịch của khách hàng A. Nó bắt tay vào tạo bản ghi cần thiết trong cơ sở dữ liệu cho đơn hàng và gửi một tin nhắn SMS cho người dắt chó chính của bạn, Gustav. Khi Gustav đã được gửi tin nhắn, `dog_walk_process` gửi một message ngược lại cho `dog_walk_order` báo rằng yêu cầu của khách hàng A đã được xử lý thành công.

Trong suốt thời gian `dog_walk_process` xử lý đơn của Khách hàng A, `dog_walk_order` đã có thể nhận thêm 100 yêu cầu dắt chó mới. Nếu chúng vẫn là một app, bạn chỉ nhận được đúng một yêu cầu trong suốt khoảng thời gian đó.

**Hình 4.2 – Tách chương trình dắt chó thành hai ứng dụng**

![Hình 4.2 – trang sách gốc chứa hình](images/pages/page-088.png)

Các mảnh ảnh trích từ hình (khung nền, hình elip RabbitMQ… – chỉ là thành phần đồ họa, không chứa chữ):

![Hình 4.2 – mảnh: khung bo tròn](images/04-coding-and-patterns/p088-22.png)
![Hình 4.2 – mảnh: khối nền ứng dụng nhận yêu cầu](images/04-coding-and-patterns/p088-61.png)
![Hình 4.2 – mảnh: khối nền ứng dụng xử lý](images/04-coding-and-patterns/p088-62.png)

Sơ đồ vẽ lại:

```mermaid
flowchart TD
    subgraph ORDER["Ứng dụng nhận yêu cầu dắt chó (Dog walking order request app)"]
        A["Nhận yêu cầu đặt lịch (Receive the scheduling request)"]
        F["Xác nhận với khách rằng yêu cầu đã thực hiện thành công (Confirm to the customer that the request was successfully filled)"]
    end
    subgraph RMQ["RabbitMQ"]
        M1["Message yêu cầu đặt lịch (Schedule request message)"]
        M2["Message xác nhận yêu cầu (Request confirmed message)"]
    end
    subgraph PROC["Ứng dụng xử lý đơn dắt chó (Dog walking order processing app)"]
        B["Lưu yêu cầu vào cơ sở dữ liệu hoặc kho lưu trữ lâu dài (Store the request in a database or other permanent storage)"]
        C["Báo người dắt chó rằng họ có khách (Alert the dog walker they've got a customer)"]
    end
    A --> M1 --> B --> C --> M2 --> F
```

Mô tả hình: giờ có **hai** ứng dụng. App nhận yêu cầu (khối xanh dương, phía trên) gửi "Schedule request message" vào RabbitMQ (hình elip vàng ở giữa). App xử lý (khối xanh lá, phía dưới) nhận message đó, lưu vào cơ sở dữ liệu, báo người dắt chó, rồi gửi "Request confirmed message" trở lại qua RabbitMQ để app nhận yêu cầu xác nhận với khách.

Bằng cách đặt RabbitMQ vào giữa hai phần của app vốn trước đây gắn chặt với nhau, bạn đã giúp app có thể liên tục nhận yêu cầu, trong khi trước đó bạn chỉ xử lý được từng yêu cầu một. Nhưng bạn cũng đã mở ra cả một thế giới lợi ích khác. Chuyện gì xảy ra nếu tải tăng lên nhiều đến mức một máy chủ xử lý đơn không còn đủ nữa?

> 💡 **Ví dụ bổ sung (không có trong sách):** Bài toán nhỏ để thấy rõ lợi ích. Giả sử nhận một yêu cầu web mất 10 ms, còn xử lý (ghi DB + gửi SMS) mất 990 ms.
>
> - **Một app đồng bộ, 1 luồng:** mỗi đơn mất 10 + 990 = 1000 ms ⇒ chỉ nhận được **1 đơn/giây**.
> - **Tách thành 2 app:** `dog_walk_order` chỉ mất 10 ms nhận đơn + khoảng 1 ms publish message ⇒ nhận được khoảng **90 đơn/giây**. Phần xử lý 990 ms diễn ra song song ở `dog_walk_process`. Đó chính là tinh thần câu "trong lúc xử lý đơn của khách A, `dog_walk_order` đã nhận thêm 100 yêu cầu".
>
> Tất nhiên nếu `dog_walk_process` chỉ xử lý 1 đơn/giây thì message sẽ dồn lại trong queue — và đó là chủ đề của mục tiếp theo.

### 4.1.2 Có khả năng mở rộng: một thế giới không cần bộ cân bằng tải (Affording scale: a world without load balancers)

Một trong những điều tuyệt vời khi dùng messaging là việc bổ sung năng lực xử lý cho ứng dụng rất đơn giản. Giả sử bạn vừa mở rộng dịch vụ sang Nhật Bản, và giờ đang nhận 1.000.000 đơn dắt chó mỗi giây. Mặc dù bộ nhận đơn ở frontend (phía giao tiếp với người dùng) thừa sức theo kịp tải, bộ xử lý đơn của bạn thì đang "gục ngã". Việc nhận đơn của một khách không còn cản bạn nhận đơn của những khách khác nữa, nhưng các khách hàng đó đang phát bực vì phải chờ bạn xác nhận lịch đặt. Bạn cần thêm bộ xử lý đơn.

Giống như ở Coffee Bean, bạn có thể thêm barista. Trong trường hợp của bạn, bạn khởi động (*spool up*) thêm các máy chủ `dog_walk_process` và gắn chúng vào queue nhận đơn. Xong! Không cần sửa một dòng code nào, bạn đã tăng gấp 10 lần năng lực xử lý bằng cách khởi động thêm 10 máy chủ `dog_walk_process` mới. Điều tuyệt nhất là RabbitMQ sẽ phân phối đều các yêu cầu giữa các máy chủ xử lý nhờ cơ chế xoay vòng (*round-robin*) tự động mà chúng ta đã nói ở mục 2.2. Không cần bộ cân bằng tải (*load balancer*) đắt tiền.

Điều đó quan trọng với mọi tổ chức, không chỉ với các startup phải tính toán từng đồng. Phần cứng cân bằng tải rất đắt, nghĩa là thông thường bạn bị giới hạn số chỗ có thể dùng nó để tách rời và mở rộng ứng dụng. Nếu thay vào đó bạn dùng AMQP và Rabbit, bạn có thể thêm tách rời và cân bằng tải ở bất kỳ đâu bạn muốn – miễn phí. Chưa kể bạn còn có thể định tuyến (*routing*) phức tạp hơn, chẳng hạn gửi một message tới nhiều hơn một đích, bên cạnh việc cân bằng tải xoay vòng.

Bộ cân bằng tải sẽ luôn có chỗ đứng ở frontend để phân phối các yêu cầu đến từ internet, nhưng nếu bạn tận dụng messaging thật nhiều, bạn có thể giảm sự phụ thuộc vào chúng bên trong tường lửa (*firewall*) và tăng đáng kể số chỗ bạn có thể tách rời ứng dụng. **Ứng dụng được tách rời là ứng dụng có khả năng mở rộng.**

> 💡 **Ví dụ bổ sung (không có trong sách):** Cơ chế round-robin hoạt động thế nào khi có nhiều consumer cùng đọc **một** queue? Giả sử queue `dog_walk_orders` có 6 message (đơn 1…6) và 3 tiến trình `dog_walk_process` (P1, P2, P3) cùng subscribe:
>
> | Message | Được giao cho |
> |---|---|
> | đơn 1 | P1 |
> | đơn 2 | P2 |
> | đơn 3 | P3 |
> | đơn 4 | P1 |
> | đơn 5 | P2 |
> | đơn 6 | P3 |
>
> Mỗi message chỉ đến **một** consumer (không bị xử lý trùng). Muốn tăng năng lực? Chỉ cần chạy thêm P4, P5… với **cùng một đoạn code**, cùng tên queue — RabbitMQ tự chia việc. (Trong thực tế người ta thường dùng thêm `basic_qos(prefetch_count=1)` để consumer chậm không bị dồn việc; phần này sách chưa đề cập ở đây.)

### 4.1.3 API không tốn công sức: sao phải bó buộc vào một ngôn ngữ? (Zero-effort APIs: why be locked into just one language?)

Chúng tôi đã bỏ qua một trong những lợi ích tuyệt nhất của việc dùng AMQP để tách rời ứng dụng: **API miễn phí**. Ngày nay ai cũng nói về các web API cho phép tích hợp chức năng của một app vào bất kỳ app nào khác. Thông thường việc này tốn chút công sức, vì cuối cùng bạn phải viết rất nhiều code để chuyển các yêu cầu HTTP đến thành các lời gọi hàm trong app. Nếu bạn viết app dùng AMQP để kết nối các phần với nhau, bạn thực sự có được một API mà không tốn thêm công sức nào – một API dùng messaging.

Giả sử bạn đã mở rộng việc kinh doanh dắt Chihuahua sang dịch vụ tắm chó. Bạn có hai app mới để hỗ trợ dịch vụ mới: `dog_wash_request` và `dog_wash_process`. Rồi bạn nảy ra một ý tưởng tuyệt vời: tặng một lần dắt chó đi dạo miễn phí với mỗi lần tắm. Vì cả app đặt lịch tắm và app đặt lịch dắt chó đều dùng AMQP, tất cả những gì bạn cần làm là cập nhật `dog_wash_request` để tạo thêm một message AMQP chứa thông tin đặt lịch dắt chó. `dog_wash_request` có thể ngay lập tức tận dụng `dog_walk_process`. Điều này có nghĩa là không phải viết lại code đặt lịch và không cần sao chép đoạn code đó vào các app tắm chó. Quan trọng không kém, không có yêu cầu nào buộc app dắt chó và app tắm chó phải được viết bằng cùng một ngôn ngữ.

Khi viết các app dắt chó, có thể bạn đã chọn Erlang là ngôn ngữ phù hợp nhất cho công việc. Nhưng trong những tháng sau đó, bạn phát hiện ra mình rất thích Clojure để xây dựng các ứng dụng có tính đồng thời cao (*high-concurrency*). Vì vậy bạn viết các app tắm chó bằng Clojure. Nếu bạn dùng giao thức giao tiếp tích hợp sẵn của Erlang để kết nối `dog_walk_request` với `dog_walk_process`, các app tắm chó sẽ rất khó nói chuyện với `dog_walk_process` vì chúng không được viết bằng Erlang. Nhưng vì AMQP **không phụ thuộc ngôn ngữ** (*language-agnostic*) và có thư viện kết nối gốc (*native language bindings*) cho hàng chục ngôn ngữ, bạn có thể dễ dàng kết nối một bộ nhận yêu cầu viết bằng Clojure với một bộ xử lý yêu cầu viết bằng Erlang thông qua Rabbit.

Dùng AMQP để kết nối các ứng dụng cho bạn sự linh hoạt để chọn đúng ngôn ngữ cho từng phần công việc, và thậm chí đổi ý sau này để kết nối thêm các ứng dụng mới viết bằng những ngôn ngữ hoàn toàn khác. RabbitMQ giúp bạn dễ dàng kết nối bất kỳ và mọi phần trong hạ tầng của mình theo bất kỳ cách nào bạn muốn.

Vì vậy, câu hỏi đầu tiên bạn nên luôn tự hỏi là: **làm sao tách các app của mình ra?** Hay đúng hơn: phần nào trong app là "người nhận đơn" và phần nào là "người xử lý đơn"? Với suy nghĩ đó, hãy cùng lao vào một số ví dụ thực tế về việc dùng Rabbit và messaging để giải quyết các vấn đề thật và trả lời những câu hỏi này.

> 💡 **Ví dụ bổ sung (không có trong sách):** "API miễn phí" trông thế nào trong thực tế? Giả sử `dog_walk_process` (viết bằng Erlang) đang consume từ exchange `chihuahua_scheduling` các message JSON dạng `{"customer": "...", "time": "..."}`. Khi đó app tắm chó viết bằng **Python** (thay cho Clojure cho dễ minh họa) chỉ cần thêm vài dòng là "gọi API" đặt lịch dắt chó:
>
> ```python
> import json, pika
>
> conn = pika.BlockingConnection(pika.ConnectionParameters("localhost"))
> ch = conn.channel()
> ch.basic_publish(exchange="chihuahua_scheduling",
>                  routing_key="",
>                  body=json.dumps({"customer": "A", "time": "2024-05-01T09:00"}))
> conn.close()
> ```
>
> "Hợp đồng API" ở đây chính là: *tên exchange + routing key + định dạng body*. Không cần viết HTTP endpoint, không cần biết `dog_walk_process` viết bằng ngôn ngữ gì hay chạy trên máy nào.

---

## 4.2 Các mô hình "bắn và quên" (Fire-and-forget models)

Khi xem xét các loại vấn đề mà messaging có thể giải quyết, một trong những lĩnh vực chính mà messaging rất phù hợp là xử lý kiểu **"bắn và quên"** (*fire-and-forget*). Dù bạn cần thêm danh bạ vào một danh sách gửi thư (*mailing list*) hay chuyển 1.000 bức ảnh thành ảnh thu nhỏ (*thumbnail*), bạn quan tâm đến việc các công việc đó được làm xong, nhưng không có lý do gì chúng phải được làm theo thời gian thực. Thực tế, bạn thường muốn tránh việc chặn (bắt chờ) người dùng đã kích hoạt các công việc đó. Chúng tôi mô tả những loại công việc này là "bắn và quên": bạn tạo công việc, đặt nó lên exchange, rồi để app của bạn quay lại với việc đang làm. Tùy theo yêu cầu, có thể bạn thậm chí không cần thông báo cho người dùng khi công việc hoàn tất.

Có hai loại công việc tổng quát phù hợp với mẫu này:

- **Xử lý theo lô (*Batch processing*)** — Công việc và các phép biến đổi cần được thực hiện trên một tập dữ liệu lớn. Có thể tổ chức thành một yêu cầu công việc duy nhất, hoặc nhiều công việc, mỗi việc xử lý một phần riêng của tập dữ liệu.
- **Thông báo (*Notifications*)** — Mô tả về một sự kiện đã xảy ra. Có thể là bất cứ thứ gì, từ một thông điệp cần ghi log, cho tới một cảnh báo thật sự cần gửi tới một chương trình khác hoặc tới quản trị viên.

Chúng tôi sẽ cho bạn xem hai ví dụ thực tế khác nhau về các app "bắn và quên" thuộc hai nhóm này. Ví dụ thứ nhất là một **framework cảnh báo** cho phép các app trong hạ tầng của bạn tạo cảnh báo gửi quản trị viên mà không phải lo cảnh báo cần đi đâu hay làm sao đưa nó tới đó. Ví dụ thứ hai là minh họa hoàn hảo cho xử lý theo lô: nhận một ảnh được tải lên và chuyển nó thành nhiều kích cỡ, định dạng ảnh khác nhau. Khi hoàn thành mục này, bạn sẽ nắm được kiểu lập trình RabbitMQ cơ bản nhất: kích hoạt công việc bằng những message không cần trả lời. Hãy bắt đầu tạo vài cảnh báo nào!

> 📝 **Giải thích thêm (không có trong sách):** Cách nhận biết một tác vụ có hợp với "bắn và quên" không: hãy hỏi *"người dùng có cần kết quả này ngay trong câu trả lời HTTP không?"*. Gửi email chào mừng, ghi log, tạo thumbnail, cộng điểm thưởng, đẩy thông báo… → **không cần** ⇒ hợp. Kiểm tra mật khẩu khi đăng nhập, tính giá đơn hàng để hiển thị → **cần ngay** ⇒ không hợp (khi đó xem mẫu RPC ở mục 4.3).

### 4.2.1 Gửi cảnh báo (Sending alerts)

Dù bạn viết loại app nào, việc nhận được thông báo khi mọi thứ trục trặc là cực kỳ quan trọng. Thông thường bạn chạy một trình giám sát dịch vụ (*service monitor*) như Nagios để báo cho bạn khi app bị sập hoặc khi các dịch vụ mà nó phụ thuộc không khả dụng. Nhưng còn việc được thông báo khi app của bạn đang nhận một lượng yêu cầu đăng nhập bất thường, tất cả đến từ một địa chỉ IP duy nhất thì sao? Hoặc có thể bạn muốn cho phép khách hàng được thông báo khi có sự kiện bất thường xảy ra với dữ liệu của họ?

Điều bạn cần là app của bạn tự tạo ra cảnh báo, nhưng điều này mở ra cả một loạt câu hỏi mới và thêm rất nhiều độ phức tạp cho app. Bạn sẽ cảnh báo về những sự kiện nào, và quan trọng hơn, cảnh báo **bằng cách nào**? SMS? IM (tin nhắn tức thời)? Dù nhìn theo cách nào, bạn cũng đang thêm rất nhiều "diện tích bề mặt" mới vào code để lỗi ẩn nấp. Ví dụ, chuyện gì xảy ra khi cổng SMS (*SMS gateway*) bị sập? Tất cả các web app cần gửi cảnh báo giờ đều phải có code xử lý lỗi để đối phó với việc máy chủ SMS không khả dụng.

Đừng lo, vì RabbitMQ đang phi ngựa tới giải cứu bạn. Điều duy nhất về cảnh báo mà **bắt buộc** phải làm trong web app là tạo ra **nội dung** của cảnh báo. Không có lý do gì web app phải biết cảnh báo nên gửi cho ai, làm sao đưa nó tới đó, hay phải làm gì khi việc gửi cảnh báo gặp trục trặc. Tất cả những gì bạn cần làm là viết một app máy chủ cảnh báo mới nhận các message cảnh báo qua Rabbit, rồi nâng cấp web app để publish các message cảnh báo này ở những chỗ thích hợp.

Bạn nên thiết kế framework cảnh báo mới này như thế nào? Đặc biệt, nên dùng loại exchange AMQP nào?

- Bạn có thể dùng một **fanout exchange** rồi tạo một queue cho mỗi kiểu truyền cảnh báo (IM, Twitter, SMS, v.v.). Ưu điểm là web app không phải biết gì về cách cảnh báo được chuyển tới người nhận cuối cùng. Nó chỉ publish rồi đi tiếp. Nhược điểm là mọi bộ truyền cảnh báo đều nhận được một bản sao, nên mỗi khi có cảnh báo bạn bị "ngập" trong một tin IM, một tin SMS và một tin nhắn trực tiếp Twitter.
- Một cách tổ chức hệ thống cảnh báo tốt hơn là tạo ba mức độ nghiêm trọng (*severity level*) cho cảnh báo: `info` (thông tin), `warning` (cảnh báo) và `critical` (nghiêm trọng). Nhưng với fanout exchange, mọi cảnh báo được publish sẽ bị gửi tới cả ba queue của ba mức độ. Thay vào đó bạn có thể tạo exchange dạng **direct exchange**, cho phép web app gắn thẻ (*tag*) message cảnh báo bằng mức độ nghiêm trọng làm routing key (khóa định tuyến).
- Nhưng chuyện gì xảy ra nếu bạn chọn **topic exchange**? Topic exchange cho phép bạn tạo các thẻ linh hoạt cho message, nhắm chúng tới nhiều queue, nhưng **chỉ** những queue cung cấp dịch vụ bạn muốn (khác với fanout exchange). Nếu dùng topic exchange cho framework cảnh báo, bạn sẽ không bị giới hạn chỉ một mức độ nghiêm trọng cho mỗi cảnh báo. Thực tế, giờ bạn có thể gắn thẻ message không chỉ bằng mức độ nghiêm trọng mà còn bằng **loại** cảnh báo.

Ví dụ, giả sử gã Joe Don Hacker đang "dội" máy chủ thống kê của bạn với 10.000 yêu cầu mỗi giây để lấy dữ liệu bản đồ về các lịch đặt dắt chó. Trong tổ chức của bạn, cảnh báo về việc này cần được gửi tới cả các quản trị viên hạ tầng (những người nhận mọi cảnh báo gắn cờ `critical`), lẫn đội phát triển API (những người nhận mọi cảnh báo gắn thẻ `rate_limiting` – giới hạn tần suất). Vì bạn đã chọn topic exchange cho framework cảnh báo, web app có thể gắn thẻ cảnh báo về hành vi mờ ám này là `critical.rate_limiting`. Xong! Message cảnh báo được RabbitMQ tự động định tuyến tới queue `critical` và queue `rate_limiting`, nhờ các binding (liên kết giữa exchange và queue) mà bạn đã tạo: `critical.*` và `*.rate_limiting`. Hình 4.3 cho thấy luồng hoạt động của hệ thống cảnh báo.

**Hình 4.3 – Luồng hoạt động của hệ thống cảnh báo**

![Hình 4.3 – trang sách gốc chứa hình](images/pages/page-092.png)

Các mảnh ảnh trích từ hình (khối nền, bóng đổ, biểu tượng thư… – chỉ là thành phần đồ họa):

![Hình 4.3 – mảnh: khối nền Alert producer](images/04-coding-and-patterns/p092-17.png)
![Hình 4.3 – mảnh: khối nền Alert consumer](images/04-coding-and-patterns/p092-19.png)
![Hình 4.3 – mảnh: biểu tượng tờ thư (email) 1](images/04-coding-and-patterns/p092-22.png)
![Hình 4.3 – mảnh: biểu tượng tờ thư (email) 2](images/04-coding-and-patterns/p092-23.png)
![Hình 4.3 – mảnh: hình elip RabbitMQ](images/04-coding-and-patterns/p092-30.png)
![Hình 4.3 – mảnh: biểu tượng queue](images/04-coding-and-patterns/p092-38.png)

Sơ đồ vẽ lại:

```mermaid
flowchart TD
    P["Alert producer (bên tạo cảnh báo)"] -- "message gắn thẻ critical.rate_limit" --> R(("RabbitMQ – exchange alerts"))
    R -- "binding critical.*" --> Q1[("queue critical")]
    R -- "binding *.rate_limit" --> Q2[("queue rate_limit")]
    subgraph C["Alert consumer (bên tiêu thụ cảnh báo)"]
        F1["critical_notify"]
        F2["rate_limit_notify"]
    end
    Q1 --> F1
    Q2 --> F2
    F1 -- email --> E1["ops.team@ourcompany.com"]
    F2 -- email --> E2["api.team@ourcompany.com"]
```

Mô tả hình: Alert producer gửi một message có thẻ `critical.rate_limit` tới RabbitMQ. Vì khớp cả hai binding `critical.*` và `*.rate_limit`, message được **sao chép** vào cả hai queue (mỗi queue đều chứa một bản `critical.rate_limit`). Trong Alert consumer, hàm `critical_notify` đọc queue `critical` và gửi email tới `ops.team@ourcompany.com`; hàm `rate_limit_notify` đọc queue `rate_limit` và gửi email tới `api.team@ourcompany.com`.

> 📝 **Giải thích thêm (không có trong sách):** Trong phần văn bản sách dùng tên thẻ `rate_limiting`, còn trong hình và code thì dùng `rate_limit` (binding `*.rate_limit`, queue `rate_limit`). Ý tưởng giống hệt nhau; khi chạy code bạn cần dùng đúng `rate_limit` cho khớp binding.

Để xây dựng framework cảnh báo này, bạn cần thư viện Pika mà bạn đã cài trong phần Hello World ở chương 2. Nếu bạn đã bỏ qua phần đó, đây là vài bước nhanh để cài Pika (giả sử bạn cũng chưa có `easy_install`):

```bash
$ wget http://peak.telecommunity.com/dist/ez_setup.py
...
(25.9 KB/s) - ez_setup.py saved [10285/10285]
$ python ez_setup.py
...
Installed /Library/Python/2.6/site-packages/setuptools-0.6...
$ easy_install pika
...
Installed /Library/Python/2.6/site-packages/pika-0.9.6-py2.6.egg
Processing dependencies for pika
Finished processing dependencies for pika
```

> 📝 **Giải thích thêm (không có trong sách):** Sách dùng Python 2.6 và Pika 0.9.6 (khoảng năm 2011). Ngày nay `easy_install` đã lỗi thời; trên Python 3 bạn cài bằng `pip install pika` (Pika 1.x). Lưu ý Pika 1.x đổi một số API: `exchange_declare(type=...)` thành `exchange_type=...`, `basic_consume(callback, queue=...)` thành `basic_consume(queue=..., on_message_callback=callback)`, `no_ack` thành `auto_ack`, và `print` phải viết dạng hàm `print(...)`. Code trong chương được giữ nguyên văn theo sách; cuối mục 4.2.1 có một ví dụ bổ sung viết lại cho Pika 1.x.

Tiếp theo, bạn cần thiết lập người dùng (*user*) và mật khẩu RabbitMQ mà các ứng dụng sẽ dùng để publish và nhận message cảnh báo. Hãy đặt tên user là `alert_user` và mật khẩu là `alertme`. Đồng thời hãy cấp cho `alert_user` quyền đọc/ghi/cấu hình (*read/write/configure*) trên vhost mặc định `/` (vhost – *virtual host*, máy chủ ảo: một không gian tách biệt bên trong RabbitMQ có exchange, queue và quyền riêng).

Từ thư mục `./sbin` trong bản cài RabbitMQ của bạn, hãy chạy:

```bash
$ ./rabbitmqctl add_user alert_user alertme
Creating user "alert_user" ...
...done.
$ ./rabbitmqctl set_permissions alert_user ".*" ".*" ".*"
Setting permissions for user "alert_user" in vhost "/" ...
...done.
```

> 📝 **Giải thích thêm (không có trong sách):** Ba tham số `".*" ".*" ".*"` của `set_permissions` lần lượt là biểu thức chính quy cho quyền **configure** (tạo/xóa exchange, queue), **write** (publish vào exchange) và **read** (consume từ queue, bind). `".*"` khớp với mọi tên, tức là cấp toàn quyền trên vhost `/`.

Xong phần chuẩn bị, bạn đã sẵn sàng làm phần quan trọng nhất của hệ thống cảnh báo: **AMQP consumer** sẽ nhận message cảnh báo và chuyển chúng tới đích. Tạo một file tên `alert_consumer.py` và đặt đoạn code trong listing sau vào đó.

**Listing 4.1 – Kết nối tới broker**

```python
import json, smtplib
import pika

if __name__ == "__main__":
    AMQP_SERVER = "localhost"          # Thiết lập broker (Broker settings)
    AMQP_USER = "alert_user"
    AMQP_PASS = "alertme"
    AMQP_VHOST = "/"
    AMQP_EXCHANGE = "alerts"

    # Thiết lập kết nối tới broker (Establish connection to broker)
    creds_broker = pika.PlainCredentials(AMQP_USER, AMQP_PASS)
    conn_params = pika.ConnectionParameters(AMQP_SERVER,
                                            virtual_host = AMQP_VHOST,
                                            credentials = creds_broker)
    conn_broker = pika.BlockingConnection(conn_params)
    channel = conn_broker.channel()
```

Chú thích bên cạnh code trong sách:
- **Broker settings** (Thiết lập broker): các hằng số `AMQP_SERVER`, `AMQP_USER`, `AMQP_PASS`, `AMQP_VHOST`, `AMQP_EXCHANGE`.
- **Establish connection to broker** (Thiết lập kết nối tới broker): tạo thông tin đăng nhập, tham số kết nối, `BlockingConnection` và mở channel.

Việc đầu tiên đoạn code này làm là import các thư viện cần thiết để consumer hoạt động, và cho Python biết phần thân chính của chương trình nằm ở đâu (`if __name__ == "__main__":`). Tiếp theo, bạn thiết lập các thông số cần để kết nối thành công tới broker (tên người dùng, mật khẩu, virtual host, v.v.). Các thông số này giả định bạn đang chạy RabbitMQ ngay trên máy phát triển của mình và dùng tên người dùng, mật khẩu vừa tạo. Cho đơn giản, hãy dùng virtual host mặc định `/`, nơi bạn sẽ tạo một exchange tên là `alerts`. Đây là chỗ "hành động thật" bắt đầu:

```python
channel.exchange_declare( exchange=AMQP_EXCHANGE,
                          type="topic",
                          auto_delete=False)
```

Bạn đang khai báo exchange `alerts` là một topic exchange nhờ tham số `type="topic"` truyền vào `channel.exchange_declare`. Tham số `auto_delete` mà bạn cũng truyền vào các khai báo exchange và queue đảm bảo chúng vẫn tồn tại khi consumer cuối cùng ngắt kết nối khỏi chúng.

> 📝 **Giải thích thêm (không có trong sách) – Đối chiếu Listing 4.1 & `exchange_declare` giữa Pika 0.9.x (Python 2) và Pika 1.x (Python 3):**
> - Trong **Listing 4.1**, cách tạo `PlainCredentials`, `ConnectionParameters` và `BlockingConnection` giữ nguyên hoàn toàn trên Pika 1.x (ngoài ra Pika 1.x bổ sung `pika.URLParameters("amqp://alert_user:alertme@localhost:5672/%2F")` rất tiện khi đọc chuỗi kết nối từ biến môi trường).
> - Tuy nhiên, ở lệnh `channel.exchange_declare(...)` ngay phía trên, tham số `type="topic"` (vốn trùng tên với hàm tích hợp sẵn `type()` của Python) đã bị **đổi tên thành `exchange_type="topic"`** trong Pika 1.0+. Nếu chạy code cũ trên Pika 1.x, bạn sẽ gặp lỗi `TypeError: exchange_declare() got an unexpected keyword argument 'type'`.
>
> ```python
> # Cũ (Pika 0.9.x trong sách):
> channel.exchange_declare(exchange=AMQP_EXCHANGE, type="topic", auto_delete=False)
>
> # Hiện đại (Pika 1.x trên Python 3):
> channel.exchange_declare(exchange=AMQP_EXCHANGE, exchange_type="topic", auto_delete=False)
> ```

Hãy nhớ rằng chúng ta đã nói về hai mẫu gắn thẻ cho cảnh báo:

- `<severity>.*` để gắn thẻ cảnh báo theo mức độ nghiêm trọng (chẳng hạn `critical`)
- `*.<type>` để gắn thẻ cảnh báo theo một loại cảnh báo cụ thể, như `rate_limiting`

*(Trong bản trích PDF, phần trong ngoặc nhọn bị mất; ý gốc là "mức độ nghiêm trọng" + `.*` và `*.` + "loại cảnh báo".)*

Điều bạn cần làm là tạo các binding hiện thực hóa các quy tắc này để message cảnh báo đi tới đúng queue bạn muốn. Với ví dụ của bạn, hãy tạo một binding định tuyến mọi message có thẻ bắt đầu bằng `critical.` tới queue `critical`. Đồng thời hãy tạo một binding khác định tuyến mọi message có thẻ kết thúc bằng `.rate_limit` tới queue `rate_limit`. Hãy tạo các queue `critical` và `rate_limit` rồi bind chúng, như trong listing sau.

**Listing 4.2 – Khai báo và bind các queue và exchange cho các topic cảnh báo**

```python
# Queue critical và binding topic critical.*
channel.queue_declare(queue="critical", auto_delete=False)
channel.queue_bind(queue="critical",
                   exchange="alerts",
                   routing_key="critical.*")

# Queue rate_limit và binding topic *.rate_limit
channel.queue_declare(queue="rate_limit", auto_delete=False)
channel.queue_bind(queue="rate_limit",
                   exchange="alerts",
                   routing_key="*.rate_limit")
```

Chú thích bên cạnh code trong sách:
- **critical queue and critical.\* topic binding** — queue `critical` và binding topic `critical.*`.
- **rate_limit queue and \*.rate_limit topic binding** — queue `rate_limit` và binding topic `*.rate_limit`.

> 📝 **Giải thích thêm (không có trong sách) – Đối chiếu Listing 4.2 (`queue_declare` & `queue_bind`) trên Pika 1.x và RabbitMQ 3.8–4.0:**
> - Trong **Pika 1.x**, tham số đầu tiên `queue` của `channel.queue_declare(queue=..., ...)` là **bắt buộc** (trong Pika 0.9.x nó có mặc định là `queue=""`). Với Listing 4.2, vì bạn đã truyền rõ `queue="critical"` và `queue="rate_limit"`, code này chạy được nguyên vẹn trên Pika 1.x. Nếu muốn tạo queue ẩn danh do server tự sinh tên (như ở mục 4.3), trên Pika 1.x bạn phải truyền tường minh `result = channel.queue_declare(queue="", exclusive=True)` và lấy tên queue qua thuộc tính **`result.method.queue`**.
> - Trong **RabbitMQ 3.8–4.0**, nếu hệ thống cảnh báo cần độ tin cậy cao trong cluster, bạn có thể khai báo Quorum Queue bằng cách truyền thêm `arguments={"x-queue-type": "quorum"}` và `durable=True`.

Bạn sẽ để ý rằng quy tắc binding tạo cho cảnh báo nghiêm trọng là `critical.*` chứ không phải `critical*`. Đó là vì RabbitMQ dùng dấu `.` làm dấu phân cách giữa các phần khác nhau của một thẻ. Nếu bạn dùng `critical*` làm quy tắc, chỉ những message có thẻ **chính xác** là `critical*` mới khớp. Điều bạn muốn là khớp `critical.mywebapp`, `critical.rate_limit`, hoặc bất cứ thứ gì bắt đầu bằng `critical.`; vì thế quy tắc binding phải là `critical.*`. Khi dùng topic exchange, điều quan trọng là phải cẩn thận thiết kế mẫu gắn thẻ, dùng `.` để phân tách những phần của thẻ mà bạn muốn so khớp riêng rẽ.

> 💡 **Ví dụ bổ sung (không có trong sách):** Bảng so khớp routing key với các binding pattern của topic exchange (`*` khớp **đúng một** từ; `#` khớp **không hoặc nhiều** từ; các từ phân tách bằng dấu `.`):
>
> | Routing key của message | `critical.*` | `*.rate_limit` | `critical*` | `#` |
> |---|---|---|---|---|
> | `critical.rate_limit` | ✅ | ✅ | ❌ | ✅ |
> | `critical.mywebapp` | ✅ | ❌ | ❌ | ✅ |
> | `info.rate_limit` | ❌ | ✅ | ❌ | ✅ |
> | `critical` | ❌ (thiếu từ thứ hai) | ❌ | ❌ | ✅ |
> | `critical.db.disk_full` | ❌ (`*` chỉ khớp 1 từ) | ❌ | ❌ | ✅ |
> | `criticalxyz.rate_limit` | ❌ | ✅ | ❌ | ✅ |
>
> Như vậy message `critical.rate_limit` của Joe Don Hacker sẽ được **nhân bản** vào cả queue `critical` lẫn `rate_limit`. Nếu muốn khớp cả `critical.db.disk_full`, bạn dùng `critical.#`.

Bạn cũng có thể truyền `durable=True` vào các khai báo queue và binding, điều này đảm bảo chúng tồn tại qua một lần khởi động lại RabbitMQ. Vì việc khởi động lại consumer sẽ tự động tạo exchange, queue và binding mà nó cần, bạn không cần lo về tính bền vững (*durability*) cho hệ thống cảnh báo. Lý do khác khiến bạn không quan tâm đến việc làm queue bền vững là vì bạn cũng sẽ không gắn cờ bền vững cho các message cảnh báo. Hệ thống của bạn có thể phải xử lý lượng cảnh báo rất lớn, nên bạn muốn đảm bảo hiệu năng cao nhất và không dùng message bền vững – loại message được ghi xuống ổ đĩa vốn tương đối chậm.

Có thể bạn đang nghĩ: "Chúng ta đã có exchange, queue và binding… vậy chỗ nào biến một message cảnh báo thành một cảnh báo thật sự?" Bạn làm điều đó bằng cách thiết lập các đăng ký (*subscription*) của consumer và khởi động vòng lặp lắng nghe, như trong listing sau.

**Listing 4.3 – Gắn các bộ xử lý cảnh báo**

```python
channel.basic_consume( critical_notify,
                       queue="critical",
                       no_ack=False,
                       consumer_tag="critical")

channel.basic_consume( rate_limit_notify,
                       queue="rate_limit",
                       no_ack=False,
                       consumer_tag="rate_limit")

print "Ready for alerts!"
channel.start_consuming()
```

Hãy mổ xẻ lời gọi `channel.basic_consume` và giải thích từng tham số:

- `critical_notify` là **callback** (hàm gọi lại). Đó là hàm sẽ được gọi khi có message đến cho subscription của bạn với queue `critical`. Thư viện Pika sẽ gọi `critical_notify` khi nhận được message trên subscription này, truyền vào channel, header của message, phần thân (*body*) message và method của message.
- `queue="critical"` chỉ định queue mà bạn muốn nhận message.
- `no_ack=False` báo cho RabbitMQ biết bạn muốn xác nhận (*acknowledge*) tường minh các message nhận được. Điều này khiến Rabbit không gửi message mới từ queue cho tới khi bạn đã xử lý xong và xác nhận message cuối cùng bạn nhận.
- `consumer_tag` là một định danh giúp nhận diện duy nhất subscription này trên channel AMQP mà bạn đã tạo bằng `channel = conn_broker.channel()`. Consumer tag là thứ bạn sẽ truyền cho RabbitMQ nếu muốn hủy subscription.

Khi đã thiết lập các subscription của consumer, bạn chỉ cần gọi `channel.start_consuming()` để consumer bắt đầu lắng nghe message. Có thể bạn đã để ý rằng các hàm callback (`critical_notify` và `rate_limit_notify`) mà bạn chỉ định cho các subscription vẫn chưa được định nghĩa. Hãy định nghĩa một trong số chúng trong listing sau.

> 📝 **Giải thích thêm (không có trong sách) – Đối chiếu Listing 4.3 (`basic_consume`), đính chính về `no_ack`/`auto_ack`, cơ chế `prefetch_count` (`basic.qos`) và `consumer_timeout` trong RabbitMQ hiện đại (3.8–4.0):**
>
> **1. Thay đổi chữ ký hàm `channel.basic_consume` trong Pika 1.x:**
> Trong Pika 0.9.x, tham số đầu tiên là `consumer_callback` và cờ xác nhận mang tên phủ định `no_ack` (rất dễ gây nhầm lẫn vì `no_ack=False` lại có nghĩa là *có cần ack*). Từ **Pika 1.0+**, thứ tự tham số đã đổi lại (`queue` đứng đầu) và `no_ack` được đổi tên thành **`auto_ack`** cho rõ nghĩa:
> ```python
> # Cũ (Pika 0.9.x + Python 2 trong Listing 4.3):
> channel.basic_consume(critical_notify, queue="critical", no_ack=False, consumer_tag="critical")
>
> # Hiện đại (Pika 1.x + Python 3):
> channel.basic_consume(queue="critical", on_message_callback=critical_notify, auto_ack=False, consumer_tag="critical")
> ```
>
> **2. Đính chính quan trọng: Chỉ bật `no_ack=False` (`auto_ack=False`) KHÔNG ngăn RabbitMQ gửi tiếp message mới!**
> Trong sách, tác giả viết: *"Điều này (`no_ack=False`) khiến Rabbit không gửi message mới từ queue cho tới khi bạn đã xử lý xong và xác nhận message cuối cùng bạn nhận"*. **Câu này chưa chính xác!**
> - Mặc định, nếu bạn chỉ đặt `auto_ack=False` mà **không** cấu hình **Consumer Prefetch (`basic.qos`)**, RabbitMQ sẽ đẩy (*push*) **không giới hạn** số lượng message có trong queue xuống bộ đệm mạng của consumer nhanh nhất có thể (tất cả đều ở trạng thái *unacknowledged / in-flight*). Một worker có thể bị ngập RAM hoặc "ôm" hàng vạn message trong khi các worker khác ngồi chơi!
> - Để giới hạn số lượng message "đang bay" (*in-flight / unacked*) tối đa mà một consumer được phép giữ cùng lúc trước khi phải gửi `basic_ack`, bạn **bắt buộc phải gọi `channel.basic_qos(prefetch_count=N)`** trước `basic_consume`:
>   ```python
>   # Giới hạn mỗi consumer chỉ nhận tối đa 10 message chưa ack cùng lúc
>   channel.basic_qos(prefetch_count=10, global_qos=False)
>   ```
> - **Ý nghĩa của `global_qos` trong RabbitMQ 3.3+ đến 4.0:** Theo đặc tả AMQP 0-9-1 gốc, `global=false` áp dụng cho cả channel và `global=true` áp dụng cho cả connection. Tuy nhiên, RabbitMQ đã định nghĩa lại cho thực tế hơn:
>   - `global_qos=False` *(mặc định)*: giới hạn `prefetch_count` áp dụng **riêng cho từng consumer mới** được đăng ký trên channel (ví dụ trong Listing 4.3 có 2 consumer là `critical` và `rate_limit` trên cùng 1 channel, mỗi consumer sẽ có hạn mức `N` message riêng).
>   - `global_qos=True`: giới hạn `prefetch_count` được **dùng chung cho toàn bộ channel**.
>
> **3. Cơ chế `consumer_timeout` (Mặc định 30 phút từ RabbitMQ 3.8.15+ / 3.12+ / 4.0):**
> Trong RabbitMQ 2.x, nếu một consumer nhận message (`auto_ack=False`) rồi bị treo hoặc xử lý một tác vụ quá lâu (ví dụ vài giờ) mà không gọi `basic_ack`, RabbitMQ sẽ kiên nhẫn chờ vô thời hạn cho tới khi kết nối TCP đứt.
> Nhưng trong **RabbitMQ hiện đại (3.12+ và 4.0)**, có một cơ chế bảo vệ broker tên là **Delivery Acknowledgement Timeout (`consumer_timeout`)**, mặc định là **30 phút (`1800000` ms)**:
> - Nếu một message đã giao cho consumer mà sau **30 phút** vẫn chưa được `basic_ack` (hoặc `basic_nack`/`basic_reject`), RabbitMQ sẽ coi consumer đó bị kẹt và **chủ động đóng luôn channel** với lỗi:
>   `PRECONDITION_FAILED - delivery acknowledgement on channel 1 timed out. Timeout value used: 1800000 ms`.
> - Khi channel bị đóng, toàn bộ các message chưa ack trên channel đó sẽ được trả lại queue (với cờ `redelivered=True`) để giao cho worker khác!
> - **Bài học thực tế:** Không giữ message chưa ack quá lâu, và không đặt `prefetch_count` quá lớn (ví dụ `prefetch_count=1000` trong khi mỗi message xử lý mất 5 giây → message thứ 1000 nằm chờ trong bộ đệm của client mất `5000s > 1800s` và sẽ làm nổ lỗi `consumer_timeout`!). Nếu thực sự có tác vụ batch chạy lâu hơn 30 phút, bạn có thể tăng giới hạn này trong `rabbitmq.conf` (`consumer_timeout = 3600000`) hoặc theo từng queue qua policy/argument `x-consumer-timeout`.

**Listing 4.4 – Bộ xử lý cảnh báo nghiêm trọng**

```python
def critical_notify(channel, method, header, body):
    """Sends CRITICAL alerts to administrators via e-mail."""

    EMAIL_RECIPS = ["ops.team@ourcompany.com",]

    message = json.loads(body)                          # Giải mã message từ JSON

    send_mail(EMAIL_RECIPS, "CRITICAL ALERT", message)  # Gửi email tới máy chủ SMTP
    print ("Sent alert via e-mail! Alert Text: %s " + \
           "Recipients: %s") % (str(message), str(EMAIL_RECIPS))

    channel.basic_ack(delivery_tag=method.delivery_tag) # Xác nhận (ack) message
```

Chú thích bên cạnh code trong sách:
- **Decode message from JSON** — Giải mã message từ JSON (`json.loads(body)`).
- **Transmit email to SMTP server** — Gửi email tới máy chủ SMTP (`send_mail(...)`).
- **Acknowledge message** — Xác nhận message (`channel.basic_ack(...)`).

*(Docstring "Sends CRITICAL alerts to administrators via e-mail." nghĩa là: "Gửi cảnh báo NGHIÊM TRỌNG tới quản trị viên qua e-mail.")*

Khi một callback của consumer được gọi, Pika truyền vào bốn tham số liên quan đến message:

- `channel` — Đối tượng channel mà bạn đang dùng để giao tiếp với Rabbit. Nếu bạn mở nhiều channel, đây sẽ là channel gắn với subscription mà message được nhận trên đó.
- `method` — Một đối tượng *method frame* mang consumer tag của subscription liên quan và delivery tag (thẻ giao nhận) của chính message đó.
- `header` — Một đối tượng biểu diễn các header của message AMQP. Chúng mang siêu dữ liệu (*metadata*) tùy chọn về message.
- `body` — Nội dung thật sự của message.

Trong `critical_notify`, việc đầu tiên cần kiểm tra là header `content_type`. Các cảnh báo của bạn sẽ được mã hóa JSON, nên bạn sẽ kiểm tra content type để chắc chắn nó là `application/json`. `content_type` là tùy chọn, nhưng nó hữu ích khi bạn muốn truyền đạt thông tin về cách mã hóa message giữa producer và consumer. Sau khi đã kiểm tra content type, bạn giải mã body của message từ JSON sang văn bản và tạo một email gửi Đội Vận hành (`ops.team@ourcompany.com`) chứa nội dung cảnh báo. Khi email cảnh báo đã gửi thành công, bạn gửi một xác nhận (*acknowledgement*) về cho RabbitMQ rằng bạn đã nhận được message. Việc xác nhận rất quan trọng vì RabbitMQ sẽ không đưa cho bạn message khác từ queue cho tới khi bạn xác nhận message cuối cùng đã nhận. Bằng cách đặt lệnh xác nhận làm thao tác **cuối cùng**, bạn đảm bảo rằng nếu consumer bị sập, RabbitMQ sẽ giao message đó cho một consumer khác.

> 📝 **Giải thích thêm (không có trong sách):** Đoạn code trong Listing 4.4 (và 4.5) thực tế **không** có dòng kiểm tra `content_type` như văn bản mô tả. Nếu muốn làm đúng như mô tả, bạn có thể thêm ở đầu hàm:
>
> ```python
>     if header.content_type != "application/json":
>         channel.basic_ack(delivery_tag=method.delivery_tag)  # hoặc basic_reject
>         return
> ```
>
> Ngoài ra, hãy chú ý thứ tự "làm việc xong rồi mới ack": nếu tiến trình chết giữa `send_mail` và `basic_ack`, message chưa được ack sẽ được RabbitMQ giao lại (có thể dẫn đến email gửi **hai lần** — đây là ngữ nghĩa "ít nhất một lần" – *at-least-once*). Nếu ack trước rồi mới gửi mail, khi sập bạn sẽ **mất** cảnh báo. Với cảnh báo, gửi trùng tốt hơn là mất.
>
> **Đối chiếu Listing 4.4 trên Python 3 + Pika 1.x (`body` là kiểu `bytes`):**
> Trong Python 2, `body` là `str` (chuỗi byte). Trong **Python 3**, `body` mà Pika 1.x truyền vào hàm callback luôn có kiểu **`bytes`** (mảng byte thô):
> - Nếu bạn nối chuỗi hoặc in trực tiếp (như `print("Alert: " + body)` ở Listing 4.13 sau này), Python 3 sẽ báo lỗi `TypeError: can only concatenate str (not "bytes") to str` — bạn cần gọi `body.decode("utf-8")`.
> - Riêng với `json.loads(body)`, từ **Python 3.6+**, hàm `json.loads()` hỗ trợ đọc trực tiếp cả `bytes` lẫn `str` (tự giải mã UTF-8), nhưng viết tường minh `json.loads(body.decode("utf-8"))` vẫn là thói quen rất tốt.

Sau khi đã giải thích hết các thành phần của consumer, hãy xem toàn bộ chương trình ghép lại trong listing sau.

**Listing 4.5 – Alert consumer — alert_consumer.py, từ đầu đến cuối**

```python
import json, smtplib
import pika


def send_mail(recipients, subject, message):
    """E-mail generator for received alerts."""
    headers = ("From: %s\r\nTo: \r\nDate: \r\n" + \
               "Subject: %s\r\n\r\n") % ("alerts@ourcompany.com",
                                         subject)

    smtp_server = smtplib.SMTP()
    smtp_server.connect("mail.ourcompany.com", 25)
    smtp_server.sendmail("alerts@ourcompany.com",
                         recipients,
                         headers + str(message))
    smtp_server.close()

# Các bộ xử lý thông báo (Notify processors)
def critical_notify(channel, method, header, body):
    """Sends CRITICAL alerts to administrators via e-mail."""

    EMAIL_RECIPS = ["ops.team@ourcompany.com",]

    message = json.loads(body)                          # Giải mã message từ JSON

    send_mail(EMAIL_RECIPS, "CRITICAL ALERT", message)  # Gửi email tới máy chủ SMTP
    print ("Sent alert via e-mail! Alert Text: %s " + \
           "Recipients: %s") % (str(message), str(EMAIL_RECIPS))

    channel.basic_ack(delivery_tag=method.delivery_tag) # Xác nhận message


def rate_limit_notify(channel, method, header, body):
    """Sends the message to the administrators via e-mail."""

    EMAIL_RECIPS = ["api.team@ourcompany.com",]

    message = json.loads(body)                          # Giải mã message từ JSON

    #(f-asc_10) Transmit e-mail to SMTP server
    send_mail(EMAIL_RECIPS, "RATE LIMIT ALERT!", message)

    print ("Sent alert via e-mail! Alert Text: %s " + \
           "Recipients: %s") % (str(message), str(EMAIL_RECIPS))

    channel.basic_ack(delivery_tag=method.delivery_tag) # Xác nhận message


if __name__ == "__main__":
    AMQP_SERVER = "localhost"                           # Thiết lập broker
    AMQP_USER = "alert_user"
    AMQP_PASS = "alertme"
    AMQP_VHOST = "/"
    AMQP_EXCHANGE = "alerts"

    # Thiết lập kết nối tới broker
    creds_broker = pika.PlainCredentials(AMQP_USER, AMQP_PASS)
    conn_params = pika.ConnectionParameters(AMQP_SERVER,
                                            virtual_host = AMQP_VHOST,
                                            credentials = creds_broker)
    conn_broker = pika.BlockingConnection(conn_params)
    channel = conn_broker.channel()

    # Khai báo exchange
    channel.exchange_declare( exchange=AMQP_EXCHANGE,
                              type="topic",
                              auto_delete=False)

    # Tạo queue và binding cho các topic
    channel.queue_declare(queue="critical", auto_delete=False)
    channel.queue_bind(queue="critical",
                       exchange="alerts",
                       routing_key="critical.*")
    channel.queue_declare(queue="rate_limit", auto_delete=False)
    channel.queue_bind(queue="rate_limit",
                       exchange="alerts",
                       routing_key="*.rate_limit")

    # Gắn các bộ xử lý cảnh báo
    channel.basic_consume( critical_notify,
                           queue="critical",
                           no_ack=False,
                           consumer_tag="critical")

    channel.basic_consume( rate_limit_notify,
                           queue="rate_limit",
                           no_ack=False,
                           consumer_tag="rate_limit")

    print "Ready for alerts!"
    channel.start_consuming()
```

Chú thích bên cạnh code trong sách (theo thứ tự xuất hiện):
- **Notify processors** — Các bộ xử lý thông báo (`critical_notify`, `rate_limit_notify`).
- **Decode message from JSON** — Giải mã message từ JSON (xuất hiện ở cả hai hàm).
- **Transmit email to SMTP server** — Gửi email tới máy chủ SMTP.
- **Acknowledge message** — Xác nhận message (ở cả hai hàm).
- **Broker settings** — Thiết lập broker.
- **Establish connection to broker** — Thiết lập kết nối tới broker.
- **Declare the exchange** — Khai báo exchange.
- **Build queues and bindings for topics** — Tạo các queue và binding cho các topic.
- **Make alert processors** — Gắn (tạo) các bộ xử lý cảnh báo.

*(Docstring: "E-mail generator for received alerts." = "Bộ tạo e-mail cho các cảnh báo nhận được"; "Sends the message to the administrators via e-mail." = "Gửi message tới quản trị viên qua e-mail". Dòng comment `#(f-asc_10) Transmit e-mail to SMTP server` là một dấu vết biên tập còn sót trong sách gốc, nghĩa "Gửi e-mail tới máy chủ SMTP".)*

> 💡 **Ví dụ bổ sung (không có trong sách) – Viết lại hoàn chỉnh Listing 4.5 (`alert_consumer.py`) cho Python 3 + Pika 1.x hiện đại:**
> ```python
> # alert_consumer_py3.py (Python 3.10+ & Pika 1.3+)
> import json
> import smtplib
> import pika
>
> def send_mail(recipients, subject, message):
>     """E-mail generator for received alerts."""
>     headers = (
>         f"From: alerts@ourcompany.com\r\n"
>         f"To: {', '.join(recipients)}\r\n"
>         f"Subject: {subject}\r\n\r\n"
>     )
>     with smtplib.SMTP("mail.ourcompany.com", 25) as smtp_server:
>         smtp_server.sendmail("alerts@ourcompany.com", recipients, headers + str(message))
>
> def critical_notify(channel, method, header, body):
>     EMAIL_RECIPS = ["ops.team@ourcompany.com"]
>     message = json.loads(body.decode("utf-8"))
>     send_mail(EMAIL_RECIPS, "CRITICAL ALERT", message)
>     print(f"Sent alert via e-mail! Alert Text: {message} Recipients: {EMAIL_RECIPS}")
>     channel.basic_ack(delivery_tag=method.delivery_tag)
>
> def rate_limit_notify(channel, method, header, body):
>     EMAIL_RECIPS = ["api.team@ourcompany.com"]
>     message = json.loads(body.decode("utf-8"))
>     send_mail(EMAIL_RECIPS, "RATE LIMIT ALERT!", message)
>     print(f"Sent alert via e-mail! Alert Text: {message} Recipients: {EMAIL_RECIPS}")
>     channel.basic_ack(delivery_tag=method.delivery_tag)
>
> if __name__ == "__main__":
>     creds_broker = pika.PlainCredentials("alert_user", "alertme")
>     conn_params = pika.ConnectionParameters(
>         "localhost", virtual_host="/", credentials=creds_broker
>     )
>     conn_broker = pika.BlockingConnection(conn_params)
>     channel = conn_broker.channel()
>
>     # Pika 1.x: dùng exchange_type thay cho type
>     channel.exchange_declare(
>         exchange="alerts", exchange_type="topic", auto_delete=False
>     )
>
>     channel.queue_declare(queue="critical", auto_delete=False)
>     channel.queue_bind(queue="critical", exchange="alerts", routing_key="critical.*")
>
>     channel.queue_declare(queue="rate_limit", auto_delete=False)
>     channel.queue_bind(queue="rate_limit", exchange="alerts", routing_key="*.rate_limit")
>
>     # Quan trọng trên RabbitMQ hiện đại: giới hạn số message unacked mỗi consumer
>     channel.basic_qos(prefetch_count=10, global_qos=False)
>
>     # Pika 1.x: queue đứng đầu, dùng on_message_callback và auto_ack
>     channel.basic_consume(
>         queue="critical",
>         on_message_callback=critical_notify,
>         auto_ack=False,
>         consumer_tag="critical",
>     )
>     channel.basic_consume(
>         queue="rate_limit",
>         on_message_callback=rate_limit_notify,
>         auto_ack=False,
>         consumer_tag="rate_limit",
>     )
>
>     print("Ready for alerts!")
>     channel.start_consuming()
> ```

Giờ bạn đã có một consumer gọn gàng, dịch các message AMQP cảnh báo thành các email cảnh báo nhắm tới những nhóm khác nhau, chỉ đơn giản bằng cách điều chỉnh thẻ của message. Việc thêm loại cảnh báo mới và phương thức truyền mới rất đơn giản. Tất cả những gì bạn cần làm là tạo một callback consumer cung cấp cách xử lý cảnh báo mới và nối nó với một queue được "nạp" message thông qua một quy tắc binding cho loại cảnh báo mới đó. Consumer của bạn sẽ chẳng hữu ích mấy nếu không có cảnh báo nào để xử lý. Vậy hãy xem cần những gì để tạo ra cảnh báo mà consumer có thể xử lý.

Mục tiêu của chúng ta khi bắt đầu mục này là làm cho việc tạo cảnh báo trở nên đơn giản, không phức tạp đối với các app hiện có. Nếu nhìn vào listing sau, bạn sẽ thấy rằng, mặc dù consumer mất khoảng 90 dòng code để xử lý một cảnh báo, bản thân cảnh báo có thể được tạo ra trong chưa đến 20 dòng.

**Listing 4.6 – Ví dụ bộ tạo cảnh báo — alert_producer.py**

```python
import json, pika
from optparse import OptionParser

# Đọc các tham số dòng lệnh (Read in command-line arguments)
opt_parser = OptionParser()
opt_parser.add_option("-r",
                      "--routing-key",
                      dest="routing_key",
                      help="Routing key for message " + \
                           " (e.g. myalert.im)")
opt_parser.add_option("-m",
                      "--message",
                      dest="message",
                      help="Message text for alert.")

args = opt_parser.parse_args()[0]

# Thiết lập kết nối tới broker (Establish connection to broker)
creds_broker = pika.PlainCredentials("alert_user", "alertme")
conn_params = pika.ConnectionParameters("localhost",
                                        virtual_host = "/",
                                        credentials = creds_broker)
conn_broker = pika.BlockingConnection(conn_params)

channel = conn_broker.channel()

# Publish message cảnh báo tới broker (Publish alert message to broker)
msg = json.dumps(args.message)
msg_props = pika.BasicProperties()
msg_props.content_type = "application/json"
msg_props.durable = False

channel.basic_publish(body=msg,
                      exchange="alerts",
                      properties=msg_props,
                      routing_key=args.routing_key)

print ("Sent message %s tagged with routing key '%s' to " + \
       "exchange '/'.") % (json.dumps(args.message),
                           args.routing_key)
```

Chú thích bên cạnh code trong sách:
- **Read in command-line arguments** — Đọc các tham số dòng lệnh (`-r/--routing-key`: routing key cho message, ví dụ `myalert.im`; `-m/--message`: nội dung văn bản của cảnh báo).
- **Establish connection to broker** — Thiết lập kết nối tới broker.
- **Publish alert message to broker** — Publish message cảnh báo tới broker.

Producer mẫu này có thể chạy từ dòng lệnh để tạo cảnh báo với bất kỳ nội dung và thẻ định tuyến nào bạn thích. Phần đầu chương trình chỉ đơn giản lấy message và routing key từ dòng lệnh. Từ đó bạn kết nối tới broker RabbitMQ y hệt cách đã làm trong alert consumer. Phần thú vị là khi bạn publish message:

```python
msg = json.dumps(args.message)
msg_props = pika.BasicProperties()
msg_props.content_type = "application/json"
msg_props.durable = False

channel.basic_publish(body=msg,
                      exchange="alerts",
                      properties=msg_props,
                      routing_key=args.routing_key)
```

Chỉ cần năm dòng code là bạn tạo được message cảnh báo và gắn cho nó routing key phù hợp (chẳng hạn `critical.mywebapp`). Sau khi mã hóa JSON nội dung cảnh báo, bạn tạo một đối tượng `BasicProperties` tên là `msg_props`. Đây là nơi bạn có thể đặt header content type tùy chọn của message AMQP, và cũng là nơi bạn làm cho message bền vững nếu muốn tính lưu trữ lâu dài (*persistency*). Cuối cùng, chỉ bằng một dòng code, bạn publish message tới exchange `alerts` với routing key phân loại kiểu cảnh báo.

Vì các message có routing key không khớp với binding nào sẽ bị loại bỏ, bạn thậm chí có thể gắn cho cảnh báo những routing key dành cho các loại cảnh báo mà bạn **chưa** hỗ trợ. Ngay khi bạn hỗ trợ các loại cảnh báo đó, mọi message cảnh báo mang routing key ấy sẽ được định tuyến tới đúng consumer.

Điểm cuối cùng cần lưu ý về consumer (*ý tác giả là producer*) là cờ `block_on_flow_control` mà bạn truyền cho `channel.basic_publish`. Cờ này bảo Pika tạm hoãn việc trả về từ `basic_publish` nếu cơ chế điều khiển luồng (*flow control*) của RabbitMQ yêu cầu ngừng publish. Khi RabbitMQ báo với Pika rằng đã có thể tiếp tục, nó mới trả về, cho phép publish tiếp. Điều này giúp producer của bạn "cư xử tử tế" với RabbitMQ: nếu Rabbit bị quá tải, nó có thể điều tiết (*throttle*) producer để làm chậm lại. Nếu bạn publish cảnh báo từ một chương trình khác không thể chấp nhận bị chặn, hãy nhớ đặt `block_on_flow_control` thành false.

> 📝 **Giải thích thêm (không có trong sách):** Đoạn code Listing 4.6 in trong sách **không** có tham số `block_on_flow_control` dù văn bản có nhắc tới (có thể là sót khi biên tập). Với Pika 0.9.x, lời gọi sẽ có dạng `channel.basic_publish(..., block_on_flow_control=True)`. Tham số này đã bị loại bỏ trong các phiên bản Pika mới; ngày nay RabbitMQ điều tiết producer bằng cơ chế *connection blocking* (khi bộ nhớ/ổ đĩa vượt ngưỡng), và `BlockingConnection` của Pika 1.x sẽ tự chặn khi bị broker block. Ngoài ra, thuộc tính `msg_props.durable = False` không phải là thuộc tính AMQP chuẩn; cách chuẩn để chọn message bền vững là `delivery_mode = 2` (bền vững) hoặc `1` (không bền vững).

> 💡 **Ví dụ bổ sung (không có trong sách) – Đối chiếu Listing 4.6 (`alert_producer.py`) sang Python 3 + Pika 1.x hiện đại:**
> Trong Listing 4.6 có 3 điểm cũ của Python 2 / Pika 0.9.x cần cập nhật khi viết trên Python 3:
> 1. Thư viện `optparse` đã bị ngừng phát triển (*deprecated*) từ Python 3.2 → thay bằng **`argparse`**.
> 2. Thuộc tính `msg_props.durable = False` hoàn toàn không có trong đặc tả AMQP 0-9-1 (trong Python 2 nó chỉ tình cờ gán thêm một biến rác vào object `BasicProperties` mà không gửi đi đâu cả!) → trong Pika 1.x, dùng **`delivery_mode=pika.DeliveryMode.Transient`** (tương đương `1`) hoặc **`pika.DeliveryMode.Persistent`** (tương đương `2`).
> 3. Tham số `body` của `basic_publish` nên truyền dạng `bytes` (`json.dumps(...).encode("utf-8")`), và luôn đóng kết nối `conn_broker.close()` ở cuối script để Pika đẩy (*flush*) hết dữ liệu ra socket TCP.
>
> ```python
> # alert_producer_py3.py (Python 3.10+ & Pika 1.3+)
> import argparse
> import json
> import pika
>
> parser = argparse.ArgumentParser(description="Send alert message to RabbitMQ")
> parser.add_argument("-r", "--routing-key", required=True, help="Routing key (e.g. critical.rate_limit)")
> parser.add_argument("-m", "--message", required=True, help="Message text for alert")
> args = parser.parse_args()
>
> creds_broker = pika.PlainCredentials("alert_user", "alertme")
> conn_params = pika.ConnectionParameters("localhost", virtual_host="/", credentials=creds_broker)
> conn_broker = pika.BlockingConnection(conn_params)
> channel = conn_broker.channel()
>
> msg_bytes = json.dumps(args.message).encode("utf-8")
> msg_props = pika.BasicProperties(
>     content_type="application/json",
>     delivery_mode=pika.DeliveryMode.Transient,  # 1 = không bền vững (RAM), 2 = bền vững (Disk)
> )
>
> channel.basic_publish(
>     exchange="alerts",
>     routing_key=args.routing_key,
>     body=msg_bytes,
>     properties=msg_props,
> )
>
> print(f"Sent message {json.dumps(args.message)} tagged with routing key '{args.routing_key}' to exchange 'alerts'.")
> conn_broker.close()
> ```

Chỉ với tổng cộng 100 dòng code, bạn đã cho các web app một cách linh hoạt và có khả năng mở rộng để phát ra cảnh báo, sau đó cảnh báo được truyền bất đồng bộ tới người nhận. Bạn cũng thấy mẫu nhắn tin "bắn và quên" có lợi thế nào khi bạn cần truyền thông tin để được xử lý nhanh nhưng không cần biết kết quả xử lý. Ví dụ, bạn có thể dễ dàng mở rộng alert consumer bằng cách thêm một bộ xử lý dùng mẫu binding `*.*` để ghi log một bản sao của mọi cảnh báo vào cơ sở dữ liệu. Nhưng cảnh báo và ghi log còn lâu mới là những ứng dụng duy nhất của mẫu nhắn tin "bắn và quên". Hãy xem một ví dụ trong đó bạn cần thực hiện xử lý nặng về CPU trên nội dung message, và RabbitMQ có thể giúp bạn chuyển việc đó thành một thao tác bất đồng bộ ra sao.

> 💡 **Ví dụ bổ sung (không có trong sách):** Chạy thử hệ thống cảnh báo. Mở terminal 1 chạy consumer, terminal 2 chạy producer:
>
> ```bash
> # Terminal 1
> $ python alert_consumer.py
> Ready for alerts!
>
> # Terminal 2
> $ python alert_producer.py -r critical.rate_limit -m "10000 req/s from 1.2.3.4"
> Sent message "10000 req/s from 1.2.3.4" tagged with routing key 'critical.rate_limit' to exchange '/'.
> ```
>
> Ở terminal 1 sẽ thấy **hai** dòng "Sent alert via e-mail!…": một từ `critical_notify` (gửi ops.team) và một từ `rate_limit_notify` (gửi api.team). Thử thêm:
>
> | Lệnh producer | Ai nhận email? |
> |---|---|
> | `-r critical.mywebapp` | chỉ ops.team |
> | `-r info.rate_limit` | chỉ api.team |
> | `-r info.mywebapp` | không ai (message bị loại bỏ vì không khớp binding) |
>
> (Muốn chạy được thật thì cần một máy chủ SMTP `mail.ourcompany.com`; khi thử nghiệm có thể thay `send_mail` bằng một hàm chỉ `print`.)

> 💡 **Ví dụ bổ sung (không có trong sách):** Bài tập "thêm loại cảnh báo mới không sửa producer": thêm queue `all_alerts` với binding `*.*` để ghi log mọi cảnh báo (đúng ý tưởng sách gợi ý ở trên). Viết theo **Pika 1.x / Python 3** cho dễ chạy ngày nay:
>
> ```python
> import pika
>
> conn = pika.BlockingConnection(pika.ConnectionParameters(
>     "localhost", credentials=pika.PlainCredentials("alert_user", "alertme")))
> ch = conn.channel()
> ch.exchange_declare(exchange="alerts", exchange_type="topic", auto_delete=False)
> ch.queue_declare(queue="all_alerts", auto_delete=False)
> ch.queue_bind(queue="all_alerts", exchange="alerts", routing_key="*.*")
>
> def log_alert(channel, method, properties, body):
>     print(f"[LOG] {method.routing_key}: {body.decode()}")
>     channel.basic_ack(delivery_tag=method.delivery_tag)
>
> ch.basic_consume(queue="all_alerts", on_message_callback=log_alert, auto_ack=False)
> print("Logging all alerts...")
> ch.start_consuming()
> ```
>
> Producer và hai consumer cũ không hề bị sửa, nhưng giờ mọi cảnh báo có dạng `x.y` đều được ghi log thêm.

### 4.2.2 Xử lý song song (Parallel processing)

Giả sử bạn bắt đầu vận hành trang web mạng xã hội của riêng mình và vừa mới triển khai một tính năng mới toanh bóng bẩy: tải ảnh lên (*picture uploads*). Mọi người muốn chia sẻ ảnh kỳ nghỉ với bạn bè và gia đình — có lẽ bạn đã thấy tính năng này ở đâu đó rồi. Ngoài ra, để cải thiện tương tác giữa những người dùng, bạn muốn thông báo cho bạn bè của họ khi một người trong danh bạ vừa tải ảnh mới lên. Một tuần sau khi phát hành tính năng mới, mấy anh bên bộ phận marketing tìm đến bàn làm việc của bạn, nhờ bạn cộng một số điểm thưởng cho người dùng — như một phần thưởng cho những bức ảnh họ tải lên nhằm khuyến khích họ tiếp tục gửi ảnh và tăng mức độ hoạt động trên trang web. Bạn đồng ý, thêm vài dòng code, và thế là móc (*hook*) hệ thống tính điểm thưởng vào quy trình tải ảnh lên. Nhìn bằng con mắt lập trình viên của bạn thì nó hơi "bẩn" một chút, nhưng nó chạy đúng như mong đợi và sếp thì hài lòng với kết quả.

Tháng sau, hóa đơn băng thông (*bandwidth bill*) gửi về và anh phụ trách vận hành (*ops*) nổi giận vì lượng băng thông sử dụng đã tăng gấp đôi. API đối ngoại cung cấp cho các client đang hiển thị ảnh kích thước đầy đủ (*full-size images*) trong khi lẽ ra chỉ nên đưa đường dẫn tới các ảnh thu nhỏ (*thumbnails*). Vậy là bạn phải làm sao cho code tải ảnh lên tạo luôn cả những ảnh thumbnail đó nữa. Làm thế nào đây? Cách dễ nhất là gắn thêm một hook nữa vào đó và chạy việc tạo thumbnail trực tiếp ngay từ controller xử lý tải ảnh lên, nhưng khoan đã… Nếu với mỗi bức ảnh tải lên bạn đều phải thực thi một thao tác thay đổi kích thước ảnh (*picture resize*), điều đó có nghĩa là các máy chủ web tuyến đầu (*frontend web servers*) sẽ bị quá tải, nên bạn không thể cứ làm thế được. Mà người dùng trang web của bạn cũng chẳng muốn ngồi đợi script xử lý ảnh chạy xong mới nhận được xác nhận là ảnh của họ đã tải lên thành công. Điều này có nghĩa là bạn cần một giải pháp thông minh hơn — một thứ cho phép bạn chạy các tác vụ **song song** (*in parallel*) và trên một máy khác với máy đang phục vụ các yêu cầu của người dùng.

Bạn có thể thấy rằng thay đổi kích thước ảnh (*resizing a picture*), cộng điểm thưởng cho người dùng (*rewarding the user*), và thông báo cho bạn bè (*notifying friends*) là ba tác vụ riêng biệt. Những tác vụ đó độc lập với nhau ở chỗ chúng không phải chờ kết quả của nhau mới chạy được; nghĩa là bạn có thể tái cấu trúc (*refactor*) code không chỉ để xử lý việc thu nhỏ ảnh một cách tách biệt, mà còn để làm những việc kia **song song** cùng lúc. Hơn nữa, nếu đạt được thiết kế như vậy, bạn có thể đối phó với các yêu cầu mới một cách dễ dàng. Cần ghi log mỗi lần tải ảnh lên ư? Bạn chỉ việc thêm một worker mới để làm việc ghi log, và cứ thế tiếp tục.

Nghe thật tuyệt, gần như trong mơ vậy, nhưng toàn bộ chuyện song song hóa (*parallelization*) này có vẻ khó thực hiện. Bạn phải viết bao nhiêu code để đạt được việc phát đa hướng thông điệp (*message multicast* – gửi một message tới nhiều người nhận cùng lúc)? Không nhiều đâu; hãy chào đón **fanout exchange**.

Như chúng tôi đã nói khi mô tả các kiểu exchange, fanout exchange sẽ đặt một bản sao của message vào tất cả các queue được bind với nó — đơn giản chỉ có vậy — và đó chính là thứ bạn cần cho module tải ảnh lên của mình. Mỗi khi người dùng tải một bức ảnh lên, thay vì làm toàn bộ công việc xử lý ngay lập tức, bạn sẽ publish một message chứa siêu thông tin (*metainformation* / *metadata*) của bức ảnh rồi để các worker bất đồng bộ làm phần còn lại một cách song song. RabbitMQ sẽ đảm bảo rằng mỗi consumer nhận được một bản sao của message. Nhiệm vụ của từng worker là xử lý message đó cho phù hợp.

Các message sẽ chứa những siêu thông tin sau đây về bức ảnh: mã định danh ảnh (*image ID*) của bức ảnh, mã người dùng (*user ID*), và đường dẫn (*path*) để tìm bức ảnh trên hệ thống file. Bạn sẽ dùng JSON làm định dạng trao đổi dữ liệu. Điều này giúp mọi việc dễ dàng hơn trong tương lai nếu bạn cần hỗ trợ nhiều ngôn ngữ lập trình cho các tác vụ khác nhau. Message của bạn sẽ trông như thế này:

```json
{
    'image_id': 123456,
    'user_id': 6543,
    'image_path': '/path/to/pic.jpg'
}
```

Hình 4.4 cho thấy bạn sẽ khai báo một exchange tên là `upload-pictures` và bind ba queue vào nó: `resize-picture`, `add-points`, và `notify-friends`. Từ thiết kế này, bạn có thể thấy rằng việc thêm một *loại* tác vụ mới, chẳng hạn như ghi log (`logs`), chỉ đơn thuần là khai báo một queue mới và bind nó vào exchange `upload-pictures`. Trọng tâm của bạn với tư cách là lập trình viên sẽ là viết code cho từng worker và logic publish; RabbitMQ sẽ lo phần còn lại.

**Hình 4.4 – Tải ảnh lên (Uploading pictures)**

![Hình 4.4 – trang sách gốc chứa hình](images/pages/page-100.png)

![Hình 4.4 – thành phần minh họa tài liệu JSON trong sơ đồ](images/04-coding-and-patterns/p100-13.png)

*(Ảnh trích `p100-13.png` là khung hình tờ giấy biểu diễn nội dung message JSON trong sơ đồ Hình 4.4; toàn bộ sơ đồ xem ở ảnh trang render phía trên và sơ đồ vẽ lại bên dưới.)*

Sơ đồ vẽ lại:

```mermaid
flowchart TD
    Upload(["Picture upload<br/>(Tải ảnh lên)"])
    Msg["image_id: 123456<br/>image_path: pic.jpg<br/>user_id: 123456"]
    Ex["upload-pictures<br/>(Fanout exchange)"]
    Q1["resize-picture"]
    Q2["notify-friends<br/>(Added later – Thêm vào sau)"]
    Q3["add-points"]
    Q4["logs<br/>(Added later – Thêm vào sau)"]

    Upload -->|"New message (Message mới)"| Msg
    Msg --> Ex
    Ex --> Q1
    Ex --> Q2
    Ex --> Q3
    Ex -.-> Q4
```

Mô tả Hình 4.4:
- Khi sự kiện **Picture upload** (Tải ảnh lên) diễn ra, hệ thống tạo một **New message** (Message mới) chứa `image_id: 123456`, `image_path: pic.jpg`, `user_id: 123456`.
- Message này được gửi tới **Fanout exchange** tên là `upload-pictures`.
- Từ exchange `upload-pictures`, bản sao của message được phát đồng thời tới tất cả các queue đang bind vào nó: `resize-picture` (thu nhỏ ảnh), `notify-friends` (báo cho bạn bè — ghi chú *Added later*: thêm vào sau), `add-points` (cộng điểm thưởng), và `logs` (ghi log — khung nét đứt, *Added later*: có thể thêm vào sau bất kỳ lúc nào mà không cần sửa code publish).

Vậy hãy bắt đầu bằng việc thêm logic publisher vào module tải ảnh lên, như trong listing sau. Ở đây ta lược bỏ phần logic lấy bức ảnh từ yêu cầu POST và lưu nó vào một nơi nào đó trên hệ thống file.

**Listing 4.7 – Publisher tải ảnh lên (Upload pictures publisher)**

```php
<?php

$channel->exchange_declare('upload-pictures',                 // (1) Khai báo exchange
    'fanout', false, true, false);

$metadata = json_encode(array(                                // (2) Mã hóa metadata của ảnh sang JSON
        'image_id' => $image_id,
        'user_id' => $user_id,
        'image_path' => $image_path
    ));

$msg = new AMQPMessage($metadata,                             // (3) Khởi tạo message AMQP
                array('content_type' => 'application/json',
                      'delivery_mode' => 2));

$channel->basic_publish($msg, 'upload-pictures');             // (4) Publish message
?>
```

Chú thích đánh số bên cạnh code trong sách:
- **(1) Declare exchange** — Khai báo exchange.
- **(2) Encode image metadata as JSON** — Mã hóa siêu dữ liệu (metadata) của ảnh sang định dạng JSON.
- **(3) Instantiate AMQP** — Khởi tạo đối tượng message AMQP (`AMQPMessage`).
- **(4) Publish message** — Publish message.

Hãy xem bạn đã làm gì ở đây. Đoạn code lấy một AMQP channel không được đưa vào vì chúng ta đã trình bày ở các ví dụ trước (trong chương 3 với thư viện `php-amqplib`). Tại **(1)**, bạn khai báo exchange `upload-pictures`, với kiểu là `fanout` và có thuộc tính bền vững (`durable` là tham số thứ tư = `true`). Sau đó tại **(2)**, bạn tạo metadata của message được mã hóa dưới dạng JSON. Các biến `$image_id`, `$user_id`, và `$image_path` đã được khởi tạo trong quá trình tải ảnh lên. Tại **(3)**, bạn tạo một thực thể (*instance*) mới của message, chỉ định `delivery_mode` là `2` để làm cho message bền vững (*persistent*). Cuối cùng tại **(4)**, bạn publish message tới exchange `upload-pictures`. Bạn không cần cung cấp routing key vì message sẽ được quạt ra (*fanned-out*) tới tất cả các queue đã bind.

> 📝 **Giải thích thêm (không có trong sách):** Trong `php-amqplib`, chữ ký của hàm `exchange_declare` là:
> `exchange_declare($exchange, $type, $passive = false, $durable = false, $auto_delete = true)`
> Do đó lệnh `$channel->exchange_declare('upload-pictures', 'fanout', false, true, false);` có nghĩa là: tên exchange `'upload-pictures'`, kiểu `'fanout'`, `passive = false`, `durable = true` (tồn tại qua lần khởi động lại broker), và `auto_delete = false` (không tự xóa khi consumer ngắt kết nối). Tương tự, `queue_declare($queue, $passive, $durable, $exclusive, $auto_delete)` ở các listing tiếp theo cũng có tham số thứ ba `true` là `durable = true`.

Tiếp theo, hãy tạo một trong các consumer — consumer dùng để cộng điểm thưởng cho người dùng sau mỗi lần tải ảnh lên. Hãy xem trong file `add-points-consumer.php` để thấy toàn bộ code hoàn chỉnh, vì listing sau đây lược bỏ những phần chúng ta đã học trước đó, như việc nạp (`include`/`require`) thư viện AMQP hay khởi tạo kết nối và channel.

**Listing 4.8 – Consumer cộng điểm (Add points consumer)**

```php
<?php

$channel->exchange_declare('upload-pictures',                 // (1) Khai báo exchange
    'fanout', false, true, false);

$channel->queue_declare('add-points',                         // (2) Khai báo queue
    false, true, false, false);

$channel->queue_bind('add-points', 'upload-pictures');        // (3) Bind queue

$consumer = function($msg){};                                 // (4) Đã lược bỏ code

$channel->basic_consume($queue,                               // (5) Bắt đầu nhận message
                        $consumer_tag,
                        false,
                        false,
                        false,
                        false,
                        $consumer);
?>
```

Chú thích đánh số bên cạnh code trong sách:
- **(1) Declare exchange** — Khai báo exchange.
- **(2) Declare queue** — Khai báo queue.
- **(3) Bind queue** — Bind queue vào exchange.
- **(4) Code omitted** — Đã lược bỏ code (phần thân hàm callback sẽ trình bày ở Listing 4.9).
- **(5) Start consuming messages** — Bắt đầu tiêu thụ (nhận) message.

Đoạn code này rất dễ hiểu. Tại **(1)**, bạn khai báo exchange giống như khi publish message *(trong sách gốc ghi nhầm là "topic exchange", thực tế trong code là `'fanout'`)*. Sau đó tại **(2)**, bạn tạo queue `add-points`, nơi message sẽ được RabbitMQ giao tới. Bạn bind queue đó tại **(3)** vào exchange với routing key rỗng (không truyền tham số thứ ba). Tại **(4)**, bạn tạm thời lược bỏ phần code của hàm callback; tại **(5)**, bạn gửi lệnh `basic_consume` để chuẩn bị cho consumer. Bạn cũng lược bỏ vòng lặp chờ (`wait`) cùng phần code dọn dẹp đóng channel và kết nối. Listing tiếp theo cho thấy hàm callback.

**Listing 4.9 – Hàm callback cộng điểm (Add points callback function)**

```php
<?php

function add_points_to_user($user_id){                        // (1) Hàm cộng điểm cho người dùng
    echo sprintf("Adding points to user: %s\n", $user_id);
}

$consumer = function($msg){                                   // (2) Callback của consumer

    if($msg->body == 'quit'){                                 // (3) Dừng việc nhận message
        $msg->delivery_info['channel']->
            basic_cancel($msg->delivery_info['consumer_tag']);
    }

    $meta = json_decode($msg->body, true);                    // (4) Giải mã metadata JSON

    add_points_to_user($meta['user_id']);                     // (5) Xử lý dữ liệu

    $msg->delivery_info['channel']->                          // Xác nhận message (Acknowledge message)
        basic_ack($msg->delivery_info['delivery_tag']);
};

?>
```

Chú thích bên cạnh code trong sách:
- **(1) Add points to user function** — Hàm cộng điểm cho người dùng.
- **(2) Consumer callback** — Hàm callback của consumer.
- **(3) Stop consuming messages** — Dừng việc tiêu thụ message.
- **(4) Decode JSON metadata** — Giải mã metadata từ JSON.
- **(5) Process data** — Xử lý dữ liệu.
- **Acknowledge message** — Xác nhận đã xử lý xong message.

Trong Listing 4.9, bạn có phần code thực sự xử lý message. Tại **(1)**, bạn thêm một hàm giả lập (*dummy function*), hiện tại chỉ in ra màn hình rằng nó đang cộng điểm cho người dùng. Trong một ứng dụng thực tế, bạn sẽ đưa vào đây logic để tăng điểm cho người dùng, chẳng hạn như trên cơ sở dữ liệu Redis. Sau đó tại **(2)**, bạn định nghĩa hàm callback của consumer. Đoạn code khéo léo tại **(3)** là một cái móc (*hook*) để dừng việc tiêu thụ message. Nếu phần thân message (`$msg->body`) bằng `'quit'`, thì bạn dừng consumer lại (bằng `basic_cancel`). Mẹo đơn giản này đảm bảo đóng channel và kết nối một cách sạch sẽ. Tiếp theo tại **(4)**, bạn truyền phần thân message vào hàm `json_decode` để lấy metadata. Bạn truyền `true` làm tham số thứ hai để đảm bảo PHP sẽ giải mã đối tượng JSON thành một mảng kết hợp (*associative array*). Tại **(5)**, bạn gọi hàm `add_points_to_user`, truyền vào tham số `user_id` mà bạn lấy được từ message đã giải mã. Cuối cùng, bạn gửi xác nhận (`basic_ack`) về cho RabbitMQ.

Hãy kiểm thử bản cài đặt này. Bạn sẽ chỉ việc sao chép code từ publisher và sửa lại phần logic tạo message để có một script kiểm thử đơn giản. Trong trường hợp này, bạn sẽ lấy ba tham số từ dòng lệnh: mã ảnh (*image ID*), mã người dùng (*user ID*), và đường dẫn ảnh (*image path*). Bạn sẽ mã hóa chúng rồi gửi qua RabbitMQ tới consumer mà bạn đã tạo trước đó. Chúng tôi sẽ không giải thích listing sau đây vì nó giống hệt những gì bạn đã thấy trong Listing 4.7.

**Listing 4.10 – Kiểm thử tải ảnh lên (Upload pictures test)**

```php
<?php
require_once('../lib/php-amqplib/amqp.inc');
require_once('../config/config.php');

$conn = new AMQPConnection(HOST, PORT, USER, PASS, VHOST);
$channel = $conn->channel();

$channel->exchange_declare('upload-pictures',
    'fanout', false, true, false);

$metadata = json_encode(array(
    'image_id' => $argv[1],
    'user_id' => $argv[2],
    'image_path' => $argv[3]
));

$msg = new AMQPMessage($metadata, array(
    'content_type' => 'application/json',
    'delivery_mode' => 2));

$channel->basic_publish($msg, 'upload-pictures');

$channel->close();
$conn->close();
?>
```

Hãy lưu đoạn code này vào một file tên là `fanout-publisher.php` và mở hai cửa sổ terminal. Trong cửa sổ thứ nhất, khởi chạy script `add-points-consumer.php`:

```bash
$ php add-points-consumer.php
```

Trong cửa sổ kia, thực thi publisher, truyền vào vài tham số bất kỳ để mô phỏng một yêu cầu:

```bash
$ php fanout-publisher.php 1 2 /path/to/pic.jpg
```

Nếu mọi thứ diễn ra suôn sẻ, bạn có thể chuyển sang cửa sổ terminal thứ nhất để thấy dòng thông báo sau:

```text
Adding points to user: 2
```

Cho đến đây thì chưa có gì ấn tượng lắm. Hãy thêm một consumer nữa để thấy fanout exchange và xử lý song song hoạt động ra sao. Đặt đoạn code trong listing sau vào file `resize-picture-consumer.php`.

**Listing 4.11 – Consumer thu nhỏ ảnh (Resize picture consumer)**

```php
<?php
require_once('../lib/php-amqplib/amqp.inc');
require_once('../config/config.php');

$conn = new AMQPConnection(HOST, PORT, USER, PASS, VHOST);
$channel = $conn->channel();

$channel->exchange_declare('upload-pictures',
    'fanout', false, true, false);

$channel->queue_declare('resize-picture',                     // (1) Khai báo queue resize-picture
    false, true, false, false);

$channel->queue_bind('resize-picture', 'upload-pictures');    // (2) Bind queue vào exchange

$consumer = function($msg){

    if($msg->body == 'quit'){
        $msg->delivery_info['channel']->
            basic_cancel($msg->delivery_info['consumer_tag']);
    }

    $meta = json_decode($msg->body, true);

    resize_picture($meta['image_id'], $meta['image_path']);   // (3) Thu nhỏ ảnh

    $msg->delivery_info['channel']->
        basic_ack($msg->delivery_info['delivery_tag']);
};

function resize_picture($image_id, $image_path){              // (4) Hàm thu nhỏ ảnh
    echo sprintf("Resizing picture: %s %s\n",
                $image_id, $image_path);
}

$channel->basic_consume($queue,
                        $consumer_tag,
                        false,
                        false,
                        false,
                        false,
                        $consumer);

while(count($channel->callbacks)) {
    $channel->wait();
}

$channel->close();
$conn->close();
?>
```

Chú thích đánh số bên cạnh code trong sách:
- **(1) Declare resize picture queue** — Khai báo queue thu nhỏ ảnh (`resize-picture`).
- **(2) Bind queue to exchange** — Bind queue vào exchange.
- **(3) Resize picture** — Gọi hàm thu nhỏ ảnh.
- **(4) Resize picture function** — Định nghĩa hàm thu nhỏ ảnh.

*(Lưu ý nhỏ trong code gốc của sách: biến `$queue` và `$consumer_tag` truyền vào `$channel->basic_consume(...)` được giả định lấy từ file cấu hình hoặc khởi tạo thành `'resize-picture'` / `'add-points'` tương ứng với queue vừa khai báo.)*

> 📝 **Giải thích thêm (không có trong sách) – Đối chiếu Listing 4.7–4.11 (`php-amqplib` cổ điển vs `php-amqplib 3.x` hiện đại và `pika 1.x` trên Python 3):**
>
> **1. Những thay đổi trong `php-amqplib 3.x` (PHP 8+):**
> - Thay `require_once('../lib/php-amqplib/amqp.inc')` và `new AMQPConnection(...)` bằng Composer autoloader và `PhpAmqpLib\Connection\AMQPStreamConnection`.
> - Thay mảng `$msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag'])` (đã deprecated) bằng các phương thức trực tiếp trên đối tượng `AMQPMessage`: **`$msg->ack()`**, **`$msg->nack()`**, và **`$msg->getChannel()->basic_cancel($msg->getConsumerTag())`**.
> - Thay vòng lặp `while (count($channel->callbacks)) { $channel->wait(); }` bằng **`$channel->consume();`** (có sẵn từ `php-amqplib 2.12+ / 3.x`).
> - Hằng số `AMQPMessage::DELIVERY_MODE_PERSISTENT` được dùng thay cho số `2` "ma thuật".
>
> **2. Bản tương đương của Listing 4.10 (Fanout Publisher) & Listing 4.11 (Resize Picture Consumer) bằng Python 3 + Pika 1.x:**
> Nếu bạn muốn chạy ví dụ xử lý ảnh song song ở mục 4.2.2 bằng **Python 3 (`pika 1.x`)** thay vì PHP:
> ```python
> # fanout_publisher_py3.py (Tương đương Listing 4.7 & 4.10)
> import json, sys, pika
>
> conn = pika.BlockingConnection(pika.ConnectionParameters("localhost"))
> ch = conn.channel()
> ch.exchange_declare(exchange="upload-pictures", exchange_type="fanout", durable=True, auto_delete=False)
>
> metadata = json.dumps({
>     "image_id": sys.argv[1] if len(sys.argv) > 1 else "1",
>     "user_id": sys.argv[2] if len(sys.argv) > 2 else "2",
>     "image_path": sys.argv[3] if len(sys.argv) > 3 else "/path/to/pic.jpg",
> }).encode("utf-8")
>
> ch.basic_publish(
>     exchange="upload-pictures",
>     routing_key="",  # Fanout exchange bỏ qua routing_key, nhưng Pika 1.x vẫn yêu cầu truyền chuỗi ""
>     body=metadata,
>     properties=pika.BasicProperties(
>         content_type="application/json",
>         delivery_mode=pika.DeliveryMode.Persistent,  # delivery_mode = 2
>     ),
> )
> conn.close()
> ```
> ```python
> # resize_picture_consumer_py3.py (Tương đương Listing 4.8, 4.9 & 4.11)
> import json, pika
>
> conn = pika.BlockingConnection(pika.ConnectionParameters("localhost"))
> ch = conn.channel()
> ch.exchange_declare(exchange="upload-pictures", exchange_type="fanout", durable=True, auto_delete=False)
> ch.queue_declare(queue="resize-picture", durable=True, auto_delete=False)
> ch.queue_bind(queue="resize-picture", exchange="upload-pictures", routing_key="")
>
> # Giới hạn mỗi worker chỉ nhận 1 tác vụ nặng về CPU một lúc (Fair dispatch)
> ch.basic_qos(prefetch_count=1)
>
> def on_message(channel, method, properties, body):
>     text = body.decode("utf-8")
>     if text == "quit":
>         channel.basic_ack(delivery_tag=method.delivery_tag)
>         channel.basic_cancel(consumer_tag=method.consumer_tag)
>         return
>     meta = json.loads(text)
>     print(f"Resizing picture: {meta['image_id']} {meta['image_path']}")
>     channel.basic_ack(delivery_tag=method.delivery_tag)
>
> ch.basic_consume(queue="resize-picture", on_message_callback=on_message, auto_ack=False)
> ch.start_consuming()
> ```

Đoạn code trong Listing 4.11 về cơ bản giống hệt Listing 4.8. Những điểm đáng chú ý nằm ở **(1)** và **(2)**, nơi bạn tạo và bind queue `resize-picture` vào exchange `upload-pictures`. Bạn có thể thấy rằng consumer này dùng chung một exchange với ví dụ trước. Như thường lệ với AMQP, message được publish tới *một* exchange rồi sau đó, tùy thuộc vào các binding, chúng có thể được định tuyến tới một hoặc nhiều queue (hoặc không tới queue nào cả).

Phần code tiếp theo rất trực diện: bên trong callback của consumer, bạn gọi hàm `resize_picture` tại **(3)**, truyền vào `image_id` và `image_path` lấy được từ metadata. Cuối cùng, hàm `resize_picture` tại **(4)** in ra một thông báo cho biết nó đang thay đổi kích thước bức ảnh. Giống như trước, trong một hệ thống thực tế, tại đây bạn sẽ viết code thực sự thay đổi kích thước ảnh.

Bây giờ, hãy mở cửa sổ thứ ba trên terminal và gõ:

```bash
$ php resize-picture-consumer.php
```

Sau đó quay lại cửa sổ nơi bạn có script publisher và chạy lại nó:

```bash
$ php fanout-publisher.php 1 2 /path/to/pic.jpg
```

Nếu mọi thứ đều ổn, bạn sẽ thấy trên mỗi cửa sổ consumer các thông báo tương ứng sau:

```text
Adding points to user: 2
```

và

```text
Resizing picture: 1 /path/to/pic.jpg
```

Dựa trên các ví dụ từ consumer cộng điểm cho người dùng, bạn có thể thấy rằng nếu tích hợp RabbitMQ vào giải pháp của mình, việc mở rộng code theo các yêu cầu mới trở nên vô cùng đơn giản. Để thêm consumer thu nhỏ ảnh, bạn chỉ cần một hàm dựa trên mã ảnh và đường dẫn ảnh, có khả năng đọc bức ảnh từ hệ thống file, thay đổi kích thước của nó (có thể dùng một công cụ nào đó như ImageMagick), rồi cập nhật bản ghi trong cơ sở dữ liệu dựa theo mã ảnh. Điều tương tự cũng áp dụng cho việc thông báo cho bạn bè của người dùng: lấy mã người dùng làm tham số, bạn có thể truy vấn danh bạ của người dùng đó từ cơ sở dữ liệu rồi gửi thông báo — có thể dưới dạng email — tới từng người bạn đó.

Điều bạn có thể rút ra từ ví dụ này là sức mạnh của việc nhắn tin với RabbitMQ nằm ở cách bạn **kết hợp các exchange và queue lại với nhau**. Nếu bạn cần một cách nào đó để lọc message, bạn có thể dùng **topic exchange** như ở mục trước. Có phải một hành động trong ứng dụng của bạn kéo theo nhiều hành động khác có thể chạy song song không? Khi đó hãy dùng **fanout exchange** *(sách gốc in nhầm là "topic exchanges", nhưng ngữ cảnh đang tổng kết mục 4.2.2 dùng fanout exchange để chạy song song nhiều tác vụ; tất nhiên bạn cũng có thể kết hợp topic exchange nếu vừa muốn lọc vừa muốn quạt ra nhiều queue)*. Nếu bạn muốn "nghe lén" (*spy*) một luồng message rồi thoát ra mà không để lại dấu vết, hãy dùng các queue ẩn danh (*anonymous queues*) được đặt chế độ tự động xóa (*autodeleted*). Một khi đã quen với việc tư duy theo các mẫu nhắn tin (*messaging patterns*), bạn sẽ bắt đầu thấy một số tác vụ lập trình có thể trở nên đơn giản đến mức nào.

Nhưng những lợi thế của thiết kế này so với thiết kế gom mọi thứ vào chung một module chưa dừng lại ở đó. Hãy tưởng tượng bây giờ việc thu nhỏ ảnh đang diễn ra quá chậm; bạn cần thêm năng lực tính toán mà lại không muốn sửa code. Chuyện đó giải quyết rất dễ: bạn có thể khởi chạy thêm nhiều tiến trình consumer (`resize-picture-consumer.php`) và RabbitMQ sẽ tự lo việc phân phối các message một cách hợp lý (chia tải vòng tròn giữa các consumer cùng lắng nghe trên queue `resize-picture`). Kể cả khi chúng nằm trên các máy khác nhau thì cũng chẳng thành vấn đề. Hãy thử tưởng tượng xem bạn sẽ mở rộng quy mô (*scale*) đoạn code ban đầu — nơi mọi thứ diễn ra tuần tự ngay trong lúc đang phục vụ yêu cầu của người dùng — bằng cách nào. Như bạn đã thấy, xử lý song song với RabbitMQ thật đơn giản.

> 💡 **Ví dụ bổ sung (không có trong sách):** Kết hợp cả **multicast (fanout)** lẫn **chia tải (work queue)** trong cùng một kiến trúc. Giả sử mỗi giây có 100 ảnh được tải lên:
> - Tác vụ `add-points` (cộng điểm vào Redis) chỉ mất 1 ms → **1 tiến trình** `add-points-consumer` xử lý dư sức 100 ảnh/giây.
> - Tác vụ `resize-picture` (dùng ImageMagick nén ảnh) mất 100 ms/ảnh → 1 tiến trình chỉ xử lý được 10 ảnh/giây. Bạn chỉ cần bật **10 tiến trình** `resize-picture-consumer` cùng lắng nghe trên đúng một queue `resize-picture` (không đổi một dòng code nào!).
>
> Khi đó:
> 1. Exchange `upload-pictures` (kiểu `fanout`) nhân bản mỗi message thành 2 bản: 1 bản vào queue `add-points`, 1 bản vào queue `resize-picture`.
> 2. Queue `add-points` giao toàn bộ 100 message/giây cho 1 worker cộng điểm.
> 3. Queue `resize-picture` chia đều (round-robin) 100 message/giây cho 10 worker xử lý ảnh (mỗi worker làm 10 ảnh/giây).
>
> Bạn vừa có **xử lý song song nhiều nghiệp vụ khác nhau** (nhờ fanout exchange -> nhiều queue), vừa có **mở rộng ngang từng nghiệp vụ nặng** (nhờ một queue -> nhiều worker)!

---

## 4.3 Hãy nhớ đến tôi: RPC qua RabbitMQ và chờ câu trả lời (Remember me: RPC over RabbitMQ and waiting for answers)

Có rất nhiều cách để thực hiện các lời gọi thủ tục từ xa (*remote procedure calls* – **RPC**): đủ mọi thứ từ UNIX RPC cho đến các REST API và SOAP. Điểm chung của tất cả các phương pháp RPC truyền thống này là mối liên kết chặt chẽ (*tight linkage*) giữa client và server. Client kết nối trực tiếp tới server, gửi một yêu cầu, rồi bị chặn (*blocks*), đứng chờ phản hồi từ server. Mô hình này có nhiều lợi ích ở chỗ bản chất điểm-nối-điểm (*point-to-point*) của nó giúp sơ đồ kết nối (*topology*) đơn giản ở quy mô nhỏ. Nhưng chính sơ đồ kết nối đơn giản đó lại hạn chế tính linh hoạt và làm tăng độ phức tạp khi đến lúc phải mở rộng quy mô (*scale up*).

Ví dụ, làm thế nào các client của bạn khám phá (*discover*) được nơi tìm thấy những server có dịch vụ chúng cần khi có nhiều server? SOAP và hầu hết các hệ thống RPC doanh nghiệp đã nghĩ ra những giao thức bổ trợ và danh bạ dịch vụ (*service directories*) phức tạp, chồng thêm độ phức tạp và các điểm có thể gây lỗi (*points of failure*), tất cả chỉ nhân danh việc có thể phục vụ các API từ nhiều máy chủ RPC mà không tạo sự phụ thuộc chặt (*tight coupling*) giữa client và server. Hơn nữa, chuyện gì xảy ra nếu máy chủ RPC mà client đang nói chuyện bị sập (*crashes*)? Client phải tự kết nối lại, và nếu máy chủ đó chết hẳn, client phải tự tìm lại một máy chủ mới cung cấp cùng dịch vụ — rồi sau khi làm xong tất cả những việc đó, client vẫn phải tự thử gọi lại (*retry*) lệnh API.

Sẽ thế nào nếu, thay vì các danh bạ phức tạp và nhiều giao thức chồng chéo, bạn có thể thực hiện RPC chỉ qua **một giao thức duy nhất**? Sẽ thế nào nếu client của bạn có thể phát ra một lời gọi API mà không cần bận tâm máy chủ nào sẽ phục vụ nó, và phải làm gì nếu máy chủ đó gặp sự cố? Sử dụng một message broker (MQ broker) để làm RPC có thể mang lại cho bạn tất cả những điều này. Khi dùng RabbitMQ cho RPC, bạn chỉ đơn giản là publish một message. Việc dùng các binding để định tuyến message đó tới đúng queue — nơi nó sẽ được máy chủ RPC tiêu thụ — là chuyện của RabbitMQ. RabbitMQ làm toàn bộ phần việc nặng nhọc: đưa message tới đúng nơi, cân bằng tải (*load balancing*) các message RPC qua nhiều máy chủ RPC, và thậm chí giao lại (*retasking*) một message RPC cho máy chủ khác khi máy chủ được giao ban đầu bị sập. Tất cả những điều này đều đạt được mà không cần các giao thức `WS-*` rắc rối, hay bất kỳ trí thông minh định tuyến nào ở phía client. Câu hỏi đặt ra là: làm sao bạn nhận lại được câu trả lời (*replies*) về cho client? Dù sao thì trải nghiệm của bạn với RabbitMQ từ đầu đến giờ vẫn là kiểu "bắn và quên" (*fire-and-forget*).

### 4.3.1 Queue riêng tư và gửi phản hồi (Private queues and sending acknowledgements)

Vì các message AMQP có tính một chiều (*unidirectional*), làm thế nào một máy chủ RPC có thể trả lời lại kết quả cho client ban đầu? Với RabbitMQ đứng ở giữa, máy chủ RPC thậm chí còn chẳng biết danh tính của client đã gọi mình, trừ phi có một mã định danh riêng của ứng dụng nằm trong phần tải (*payload*) của message. May mắn thay, đội ngũ phát triển RabbitMQ có một giải pháp rất thanh lịch: **dùng chính các message để gửi câu trả lời ngược lại**. Trên mỗi header của message AMQP đều có một trường tên là `reply_to`. Bên trong trường này, producer của message có thể chỉ định tên của queue mà nó sẽ lắng nghe để chờ câu trả lời. Máy chủ RPC nhận yêu cầu sau đó có thể kiểm tra trường `reply_to` này và tạo một message mới chứa câu trả lời với tên queue đó làm routing key.

Có thể bạn đang tự nhủ: "Mỗi lần lại phải tạo một tên queue duy nhất nghe có vẻ tốn công quá. Rồi làm sao ngăn các client khác đọc mất câu trả lời của mình?" Một lần nữa, RabbitMQ lại ra tay cứu giúp. Có thể bạn còn nhớ từ chương 1 rằng nếu bạn khai báo một queue mà không đặt tên queue, RabbitMQ sẽ tự gán một tên cho bạn. Cái tên này ngẫu nhiên lại là một tên queue duy nhất, và khi được khai báo kèm tham số `exclusive` (độc quyền), nó đảm bảo rằng **chỉ một mình bạn** mới có thể đọc từ queue đó. Tất cả những gì các RPC client của bạn phải làm là khai báo một queue tạm thời (*temporary*), độc quyền (*exclusive*), ẩn danh (*anonymous*), rồi đưa tên của queue đó vào header `reply_to` trong message RPC của chúng — thế là phía server đã có một nơi để gửi phản hồi về.

Lưu ý rằng chúng tôi không hề nói gì về việc bind queue nhận câu trả lời (*reply queue*) vào một exchange nào cả. Đó là bởi vì khi máy chủ RPC publish message trả lời của nó lên RabbitMQ mà **không chỉ định exchange** (tức là dùng exchange mặc định `""`), RabbitMQ hiểu rằng message đó nhắm tới một queue trả lời và routing key chính là tên của queue đó.

> 📝 **Giải thích thêm (không có trong sách):** Nhắc lại cơ chế từ Chương 1 và Chương 2: trong AMQP, **exchange mặc định** (*default exchange*, có tên là chuỗi rỗng `""`) là một direct exchange đặc biệt được khai báo sẵn. Mọi queue khi được tạo ra đều **tự động được bind** vào exchange mặc định `""` với routing key trùng với chính tên của queue đó (chẳng hạn `amq.gen-9ith1Dx...`). Nhờ vậy, khi RPC server gọi `basic_publish` với `exchange=""` và `routing_key=header.reply_to`, RabbitMQ sẽ chuyển thẳng message trả lời vào đúng queue riêng tư của client mà client không cần thực hiện lệnh `queue_bind` nào cả.

Nói lý thuyết thế đủ rồi; hãy xem cách bạn làm cho RPC hoạt động với RabbitMQ bằng code thực tế.

### 4.3.2 JSON RPC đơn giản với reply_to (Simple JSON RPC with reply_to)

Thứ đầu tiên bạn cần là một máy chủ RPC (*RPC server*). Trước khi đi sâu vào code, việc nhìn qua luồng hoạt động của RPC client và server trong Hình 4.5 sẽ giúp bạn dễ hình dung hơn.

Trong listing tiếp theo, bạn sẽ xây dựng một máy chủ API đơn giản hiện thực hóa lời gọi `ping`. Chức năng duy nhất của lời gọi này là nhận lệnh gọi `ping` từ client, và gửi lại một câu trả lời `Pong!` kèm theo nhãn thời gian (*timestamp*) mà client đã đính kèm trong lời gọi ban đầu.

**Hình 4.5 – Luồng hoạt động của RPC client và server (RPC client and server flow)**

![Hình 4.5 – trang sách gốc chứa hình](images/pages/page-107.png)

Các thành phần hình ảnh trích từ trang 107 của PDF:

![Hình 4.5 – thành phần hộp nền xanh (API client / API server)](images/04-coding-and-patterns/p107-11.png)
![Hình 4.5 – thành phần hình elip xanh lá (RabbitMQ)](images/04-coding-and-patterns/p107-12.png)
![Hình 4.5 – thành phần hình trụ xanh lá (Queue)](images/04-coding-and-patterns/p107-13.png)
![Hình 4.5 – thành phần biểu tượng thông điệp (Message)](images/04-coding-and-patterns/p107-16.png)

*(Bốn ảnh trích `p107-11.png`, `p107-12.png`, `p107-13.png`, `p107-16.png` là các khối đồ họa nền cấu tạo nên Hình 4.5; toàn bộ sơ đồ có chữ và mũi tên xem ở ảnh trang render phía trên và sơ đồ vẽ lại bên dưới.)*

Sơ đồ vẽ lại:

```mermaid
flowchart TD
    subgraph Client["API client"]
        RC(["reply_callback"])
    end

    MsgReq["Message yêu cầu:<br/>routing_key = 'ping'<br/>reply_to = 'amq.gen-9ith1Dx...'"]
    ReplyQ[("Reply Queue riêng tư:<br/>amq.gen-9ith1Dx...")]
    RMQ(("RabbitMQ"))
    PingQ[("Queue yêu cầu:<br/>ping")]
    MsgRep["Message trả lời:<br/>routing_key = 'amq.gen-9ith1Dx...'"]

    subgraph Server["API server"]
        AP(["api_ping<br/>(trong hình gốc ghi nhầm là rate_limit_notify)"])
    end

    Client -->|"1. Publish yêu cầu"| MsgReq
    MsgReq --> RMQ
    RMQ -->|"2. Định tuyến theo key 'ping'"| PingQ
    PingQ -->|"3. Giao yêu cầu"| AP
    AP -->|"4. Publish trả lời (exchange='')"| MsgRep
    MsgRep --> RMQ
    RMQ -->|"5. Định tuyến theo tên queue<br/>amq.gen-9ith1Dx..."| ReplyQ
    ReplyQ -->|"6. Giao trả lời"| RC
    RC -->|"7. Kết quả"| Pong["Pong!"]
```

Mô tả Hình 4.5:
1. **API client** tạo một queue trả lời ẩn danh, độc quyền (tên do RabbitMQ sinh ra, ví dụ `amq.gen-9ith1Dx...`) và gửi một message yêu cầu `ping` tới **RabbitMQ** (kèm tên queue trả lời trong header `reply_to`).
2. **RabbitMQ** định tuyến message `ping` vào queue `ping`.
3. **API server** tiêu thụ message từ queue `ping` và đưa vào hàm xử lý *(lưu ý: trong hình vẽ gốc của sách, ô trắng trong **API server** bị in nhầm nhãn là `rate_limit_notify` do tác giả copy hình từ mục 4.2.1; đúng theo code Listing 4.12 phải là hàm `api_ping`)*.
4. Sau khi xử lý xong, **API server** publish message trả lời với routing key là `amq.gen-9ith1Dx...` (lấy từ `header.reply_to`) qua **RabbitMQ**.
5. **RabbitMQ** đưa message trả lời vào đúng queue `amq.gen-9ith1Dx...` của client.
6. Hàm callback `reply_callback` bên trong **API client** nhận message từ queue này và xuất ra kết quả `Pong!`.

Trước khi chạy code của mục này, bạn cũng cần tạo người dùng `rpc_user` với mật khẩu `rpcme` và cấp quyền trên vhost `/` (tương tự như đã làm với `alert_user` ở mục 4.2.1):

```bash
$ ./rabbitmqctl add_user rpc_user rpcme
$ ./rabbitmqctl set_permissions rpc_user ".*" ".*" ".*"
```

**Listing 4.12 – Máy chủ API — rpc_server.py (API server—rpc_server.py)**

```python
import pika, json

# Thiết lập kết nối tới broker (Establish connection to broker)
creds_broker = pika.PlainCredentials("rpc_user", "rpcme")
conn_params = pika.ConnectionParameters("localhost",
                                        virtual_host = "/",
                                        credentials = creds_broker)
conn_broker = pika.BlockingConnection(conn_params)
channel = conn_broker.channel()

# Khai báo exchange và queue cho lệnh gọi ping (Declare exchange and ping call queue)
channel.exchange_declare(exchange="rpc",
                         type="direct",
                         auto_delete=False)
channel.queue_declare(queue="ping", auto_delete=False)
channel.queue_bind(queue="ping",
                   exchange="rpc",
                   routing_key="ping")

# Chờ các lời gọi RPC và trả lời (Wait for RPC calls and reply)
def api_ping(channel, method, header, body):
    """'ping' API call."""
    channel.basic_ack(delivery_tag=method.delivery_tag)
    msg_dict = json.loads(body)
    print "Received API call...replying..."
    channel.basic_publish(body="Pong!" + str(msg_dict["time"]),
                          exchange="",
                          routing_key=header.reply_to)

channel.basic_consume(api_ping,
                      queue="ping",
                      consumer_tag="ping")

print "Waiting for RPC calls..."
channel.start_consuming()
```

Chú thích bên cạnh code trong sách:
- **Establish connection to broker** — Thiết lập kết nối tới broker.
- **Declare exchange and ping call queue** — Khai báo exchange và queue cho lệnh gọi `ping`.
- **Wait for RPC calls and reply** — Chờ các lời gọi RPC và gửi câu trả lời.

Chúng ta đã học phần thiết lập và kết nối tới RabbitMQ rồi, nên hãy nhảy tới phần thú vị nơi tạo exchange và các queue để nhận các lời gọi API:

```python
channel.exchange_declare(exchange="rpc",
                         type="direct",
                         auto_delete=False)
channel.queue_declare(queue="ping", auto_delete=False)
channel.queue_bind(queue="ping",
                   exchange="rpc",
                   routing_key="ping")
```

Điều bạn vừa làm ở đây là thiết lập một direct exchange điển hình và tạo một queue cùng với binding. Đối với API này, bạn đang đi theo một mẫu thiết kế trong đó **tên của hàm RPC** chính là thứ bạn dùng làm mẫu binding (và cũng là tên queue cho các lời gọi đó). Trong trường hợp này, lời gọi API `ping` được tạo bằng cách bind queue `ping` vào exchange `rpc` với mẫu binding là `ping`. Tất cả những gì các client của bạn cần làm là đặt `ping` làm routing key và đưa các tham số của chúng vào phần thân message. Bạn cũng có thể định tuyến các yêu cầu RPC phức tạp hơn bằng cách sử dụng một topic exchange. Tiếp theo, bạn cần thiết lập đăng ký consumer (*consumer subscription*):

```python
def api_ping(channel, method, header, body):
    """'ping' API call."""
    channel.basic_ack(delivery_tag=method.delivery_tag)
    msg_dict = json.loads(body)
    print "Received API call...replying..."
    channel.basic_publish(body="Pong!" + str(msg_dict["time"]),
                          exchange="",
                          routing_key=header.reply_to)

channel.basic_consume(api_ping,
                      queue="ping",
                      consumer_tag="ping")
```

Giờ đây `api_ping` sẽ được gọi mỗi khi RabbitMQ giao một message cho bạn thông qua queue `ping`. Tất cả những điều này đều tương tự như những gì bạn đã làm từ đầu sách tới giờ. Điểm khác biệt mà bạn sẽ nhận thấy là lệnh `basic_publish` được phát ra sau khi xác nhận (*acknowledge*) message yêu cầu. Khoan đã! Làm sao bạn có thể publish một câu trả lời ngay trên chính cái channel mà bạn đang dùng để consume được? Chẳng phải trước đó chúng ta đã nói điều đó là bất khả thi sao?! Thực ra, trong trường hợp này điều đó làm được là vì thư viện Pika sẽ không bắt đầu consume trở lại cho tới khi hàm `api_ping` của bạn kết thúc (*returns*).

Điều quan trọng hơn cần tập trung vào là cấu hình của lệnh `basic_publish`. Nó sử dụng `reply_to` lấy từ `header` làm `routing_key` cho message trả lời. Ngoài ra, không giống bất kỳ lệnh publish nào khác mà bạn từng làm với RabbitMQ, ở đây **không có exchange nào** được chỉ định khi publish (`exchange=""`). Đó là hai thành phần đặc biệt duy nhất bạn cần biết để làm cho RPC hoạt động qua Rabbit: **publish câu trả lời bằng cách dùng `reply_to` làm đích đến, và publish mà không chỉ định tên exchange**.

Thế còn RPC client thì sao? Nó trông như thế nào và làm sao bạn thiết lập được queue trả lời? Hãy cùng xem listing tiếp theo.

**Listing 4.13 – API client — rpc_client.py**

```python
import time, json, pika

# Thiết lập kết nối tới broker (Establish connection to broker)
creds_broker = pika.PlainCredentials("rpc_user", "rpcme")
conn_params = pika.ConnectionParameters("localhost",
                                        virtual_host = "/",
                                        credentials = creds_broker)
conn_broker = pika.BlockingConnection(conn_params)
channel = conn_broker.channel()

# Phát lời gọi RPC và chờ câu trả lời (Issue RPC call and wait for reply)
msg = json.dumps({"client_name": "RPC Client 1.0",
                  "time" : time.time()})

result = channel.queue_declare(exclusive=True, auto_delete=True)
msg_props = pika.BasicProperties()
msg_props.reply_to=result.method.queue

channel.basic_publish(body=msg,
                      exchange="rpc",
                      properties=msg_props,
                      routing_key="ping")

print "Sent 'ping' RPC call. Waiting for reply..."

def reply_callback(channel, method, header, body):
    """Receives RPC server replies."""
    print "RPC Reply --- " + body
    channel.stop_consuming()

channel.basic_consume(reply_callback,
                      queue=result.method.queue,
                      consumer_tag=result.method.queue)

channel.start_consuming()
```

Chú thích bên cạnh code trong sách:
- **Establish connection to broker** — Thiết lập kết nối tới broker.
- **Issue RPC call and wait for reply** — Phát lời gọi RPC và chờ câu trả lời.

Trái tim của việc làm cho RPC hoạt động ở phía client nằm ở đoạn code này:

```python
result = channel.queue_declare(exclusive=True, auto_delete=True)
msg_props = pika.BasicProperties()
msg_props.reply_to=result.queue
```

*(Lưu ý nhỏ: Trong Listing 4.13 đầy đủ ở trên, tác giả viết `msg_props.reply_to=result.method.queue`, còn trong đoạn trích giải thích ngắn này tác giả viết tắt thành `result.queue`; với Pika thì `result.method.queue` là thuộc tính chính xác chứa tên queue do RabbitMQ trả về trong frame `Queue.DeclareOk`.)*

Chỉ trong ba dòng đó, bạn tạo queue trả lời của mình và gán header `reply_to` trên message bằng tên của queue mới tạo. Khi khai báo queue trả lời, hãy nhớ đặt `exclusive=True` và `auto_delete=True`. Điều này đảm bảo không ai khác có thể "chôm chỉa" (*pilfer*) các message của bạn (dù tên queue do Rabbit tạo ra vốn đã là duy nhất), và khi bạn ngắt kết nối khỏi queue sau khi nhận được câu trả lời, queue đó sẽ được Rabbit tự động xóa đi. Tất cả những gì còn lại là publish message gọi API và đăng ký hàm callback của bạn vào queue trả lời:

```python
channel.basic_publish(body=msg,
                      exchange="rpc",
                      properties=msg_props,
                      routing_key="ping")

print "Sent 'ping' RPC call. Waiting for reply..."

def reply_callback(channel, method, header, body):
    """Receives RPC server replies."""
    print "RPC Reply --- " + body
    channel.stop_consuming()

channel.basic_consume(reply_callback,
                      queue=result.method.queue,
                      consumer_tag=result.method.queue)
```

Chẳng có phép thuật huyền bí nào ở đây cả. Một khi đã thiết lập xong queue trả lời, bạn có thể consume từ nó giống như bất kỳ queue nào khác. Chỉ cần lưu ý **đừng bắt đầu consume từ queue trước khi bạn publish message gọi API**. Nếu không, channel sẽ chuyển sang chế độ consume (vòng lặp chặn `start_consuming()`) và bạn sẽ gặp lỗi (hoặc không bao giờ tới được dòng lệnh publish) khi cố gắng publish. Vậy khi chạy ứng dụng RPC của bạn thì phía client và phía server trông sẽ như thế nào?

```text
Client) Sent 'ping' RPC call. Waiting for reply...
Server) Received API call...replying...
Client) RPC Reply --- Pong! (Client Name: RPC Client 1.0)
        (RPC Call Issued Time: 1288111236.43)
```

*(Lưu ý: Đầu ra mẫu của sách hiển thị đầy đủ cả `Client Name` và `RPC Call Issued Time`, trong khi dòng lệnh ở Listing 4.12 viết gọn là `"Pong!" + str(msg_dict["time"])` — ý đồ minh họa vẫn hoàn toàn giống nhau.)*

Bạn có thể thấy rằng câu trả lời của server thực sự dựa trên lời gọi của client, bởi vì nhãn thời gian (*timestamp*) nằm trong câu trả lời của server chính là nhãn thời gian có trong phần thân message yêu cầu của client. Từ đây, bạn có thể dễ dàng mở rộng API bằng cách tạo thêm các queue và binding mới cho những phương thức API mới. Phần tuyệt nhất là chẳng có lý do gì bắt buộc một máy chủ RPC duy nhất phải trả lời tất cả các lệnh gọi API. Chẳng hạn, bạn có thể dễ dàng viết một máy chủ RPC mới thực hiện việc xử lý ảnh, và chạy nó trên một máy vật lý hoàn toàn khác với máy chủ API `ping`. Các client của bạn sẽ không hề nhận ra sự khác biệt, và bạn hoàn toàn tự do mở rộng quy mô các API của mình theo bất kỳ cách nào bạn thấy phù hợp. RabbitMQ làm nên điều kỳ diệu khi khiến tất cả hoạt động như một mạng lưới API thống nhất (*one API fabric*). Không cần giao thức đặc biệt. Không cần danh bạ dịch vụ.

> 💡 **Ví dụ bổ sung (không có trong sách):** Trong thực tế, nếu một client gửi **nhiều** yêu cầu RPC liên tiếp qua cùng một queue trả lời (thay vì mỗi yêu cầu lại tạo một queue mới), làm sao client phân biệt được câu trả lời nào ứng với yêu cầu nào? AMQP cung cấp sẵn một thuộc tính chuẩn trong `BasicProperties` tên là **`correlation_id`** (mã tương quan).
>
> Dưới đây là ví dụ RPC hoàn chỉnh viết bằng **Python 3 + Pika 1.x**, sử dụng cả `reply_to` và `correlation_id`:
>
> ```python
> # rpc_server_py3.py
> import json, pika
>
> creds = pika.PlainCredentials("rpc_user", "rpcme")
> conn = pika.BlockingConnection(pika.ConnectionParameters("localhost", credentials=creds))
> ch = conn.channel()
>
> ch.exchange_declare(exchange="rpc", exchange_type="direct", auto_delete=False)
> ch.queue_declare(queue="ping", auto_delete=False)
> ch.queue_bind(queue="ping", exchange="rpc", routing_key="ping")
>
> def api_ping(channel, method, props, body):
>     msg_dict = json.loads(body.decode())
>     reply = f"Pong! (Client Name: {msg_dict['client_name']}) (RPC Call Issued Time: {msg_dict['time']})"
>     channel.basic_publish(
>         exchange="",
>         routing_key=props.reply_to,
>         properties=pika.BasicProperties(correlation_id=props.correlation_id),
>         body=reply.encode()
>     )
>     channel.basic_ack(delivery_tag=method.delivery_tag)
>
> ch.basic_consume(queue="ping", on_message_callback=api_ping, auto_ack=False)
> print("Waiting for RPC calls...")
> ch.start_consuming()
> ```
>
> ```python
> # rpc_client_py3.py
> import json, time, uuid, pika
>
> creds = pika.PlainCredentials("rpc_user", "rpcme")
> conn = pika.BlockingConnection(pika.ConnectionParameters("localhost", credentials=creds))
> ch = conn.channel()
>
> result = ch.queue_declare(queue="", exclusive=True, auto_delete=True)
> reply_queue = result.method.queue
> corr_id = str(uuid.uuid4())
>
> msg = json.dumps({"client_name": "RPC Client 1.0", "time": time.time()})
> ch.basic_publish(
>     exchange="rpc",
>     routing_key="ping",
>     properties=pika.BasicProperties(reply_to=reply_queue, correlation_id=corr_id),
>     body=msg.encode()
> )
> print("Sent 'ping' RPC call. Waiting for reply...")
>
> def on_reply(channel, method, props, body):
>     if props.correlation_id == corr_id:
>         print("RPC Reply ---", body.decode())
>         channel.stop_consuming()
>
> ch.basic_consume(queue=reply_queue, on_message_callback=on_reply, auto_ack=True)
> ch.start_consuming()
> ```

> 📝 **Giải thích thêm (không có trong sách) – Bảng tổng hợp đối chiếu Pika 0.9.x (Python 2, Listing 4.1–4.13) vs Pika 1.x (Python 3 hiện đại):**
>
> | Thao tác AMQP | Pika 0.9.x + Python 2 (Trong sách, 2012) | Pika 1.x + Python 3 (Chuẩn hiện đại) | Lý do thay đổi / Lưu ý |
> |---|---|---|---|
> | **Khai báo Exchange** | `ch.exchange_declare(exchange="rpc", type="direct")` | `ch.exchange_declare(exchange="rpc", exchange_type="direct")` | Tránh đè lên hàm built-in `type()` của Python. |
> | **Khai báo Queue ẩn danh** | `res = ch.queue_declare(exclusive=True)` | `res = ch.queue_declare(queue="", exclusive=True)` | Trong Pika 1.x, đối số `queue` là bắt buộc; truyền `""` để broker tự sinh tên. |
> | **Lấy tên Queue vừa tạo** | `res.method.queue` *(sách có chỗ ghi nhầm `res.queue`)* | `res.method.queue` | Tên queue nằm trong AMQP method frame `Queue.DeclareOk`. |
> | **Đăng ký Consumer** | `ch.basic_consume(cb, queue="q", no_ack=False)` | `ch.basic_consume(queue="q", on_message_callback=cb, auto_ack=False)` | Đưa `queue` lên đầu; đổi tên `no_ack` (phủ định kép) thành `auto_ack` rõ nghĩa. |
> | **Mặc định của `no_ack` / `auto_ack`** | `no_ack=False` (mặc định phải `basic_ack`) | `auto_ack=False` (mặc định phải `basic_ack`) | Giữ nguyên hành vi an toàn mặc định. |
> | **Kiểu dữ liệu của `body`** | `str` (`"RPC Reply --- " + body`) | `bytes` (`"RPC Reply --- " + body.decode("utf-8")`) | Python 3 tách bạch chuỗi Unicode (`str`) và mảng byte (`bytes`). |
> | **Đánh dấu message bền vững** | `msg_props.durable = False` *(sai chuẩn)* hoặc `delivery_mode=2` | `delivery_mode=pika.DeliveryMode.Persistent` (`2`) hoặc `Transient` (`1`) | AMQP 0-9-1 chỉ định nghĩa `delivery_mode`, không có `msg_props.durable`. |
>
> ---
>
> 📝 **Giải thích thêm (không có trong sách) – Cách làm RPC hiện đại trong RabbitMQ 3.4–4.0: Tính năng Direct Reply-to (`amq.rabbitmq.reply-to`):**
>
> **1. Điểm nghẽn của cách tạo `exclusive` reply queue truyền thống (Listing 4.13):**
> Trong Listing 4.13, mỗi lần gọi RPC, client lại gọi `channel.queue_declare(exclusive=True, auto_delete=True)` để tạo một queue thật rồi xóa đi ngay sau khi nhận câu trả lời.
> - Trong một cluster RabbitMQ, việc tạo và xóa một queue buộc broker phải **ghi giao dịch đồng bộ vào cơ sở dữ liệu metadata toàn cụm (Mnesia hoặc Khepri)** trên mọi node, đồng thời khởi tạo một tiến trình Erlang cho queue đó.
> - Nếu hệ thống của bạn thực hiện **5.000 lời gọi RPC/giây**, việc khai báo và xóa 5.000 queue tạm mỗi giây sẽ gây quá tải nặng nề lên Mnesia/Khepri (*metadata churn*) và tốn thêm 1 vòng lặp mạng (*network round-trip*) cho lệnh `queue.declare`!
>
> **2. Giải pháp hiện đại: Hàng đợi ảo `amq.rabbitmq.reply-to` (Direct Reply-to):**
> Từ RabbitMQ 3.4.0 trở đi (và là chuẩn mực trên RabbitMQ 3.8–4.0), RabbitMQ cung cấp tính năng **Direct Reply-to** thông qua tên queue đặc biệt **`amq.rabbitmq.reply-to`**:
> - Client **không cần gọi `queue_declare`**!
> - Client chỉ cần đăng ký consume trực tiếp từ `queue="amq.rabbitmq.reply-to"` với **`auto_ack=True` (bắt buộc)** **trước** khi publish yêu cầu RPC.
> - Khi client publish yêu cầu với `reply_to="amq.rabbitmq.reply-to"`, bản thân RabbitMQ sẽ tự động viết lại trường `reply_to` này thành một định danh bí mật nội bộ (ví dụ `amq.rabbitmq.reply-to.g2dkAB...`) trỏ thẳng tới tiến trình Erlang của channel client!
> - RPC Server giữ nguyên 100% code cũ: nó vẫn chỉ đọc `props.reply_to` và publish câu trả lời vào `exchange=""` với `routing_key=props.reply_to`. Câu trả lời sẽ bay thẳng về bộ nhớ channel của RPC Client mà **không đi qua bất kỳ queue thật nào và không ghi bất kỳ metadata nào vào Mnesia/Khepri**!
>
> **Bảng so sánh 3 kiến trúc RPC trên RabbitMQ:**
>
> | Tiêu chí | 1. Queue tạm mỗi request (Listing 4.13) | 2. Một Queue riêng tư cố định mỗi Client + `correlation_id` | 3. **Direct Reply-to (`amq.rabbitmq.reply-to`)** *(Khuyến nghị hiện đại)* |
> |---|---|---|---|
> | **Lệnh `queue_declare`** | Gọi mỗi lần RPC (rất chậm!) | Gọi 1 lần khi khởi động client | **Không cần gọi `queue_declare`** |
> | **Tải lên Mnesia / Khepri** | Cực cao (*metadata churn*) | Thấp | **Bằng 0 (Zero metadata overhead)** |
> | **Chế độ xác nhận trả lời** | `auto_ack=True` hoặc `False` | `auto_ack=True` hoặc `False` | **Bắt buộc `auto_ack=True`** (`no-ack` mode) |
> | **Thứ tự gọi ở Client** | Declare → Publish → Consume | Declare → Consume → Publish | **Consume (`auto_ack=True`) → Publish** |

> 💡 **Ví dụ bổ sung (không có trong sách) – RPC Client hiện đại dùng Direct Reply-to (`amq.rabbitmq.reply-to`) với Python 3 + Pika 1.x:**
> Bạn có thể dùng nguyên vẹn `rpc_server_py3.py` ở trên, và viết lại `rpc_client` siêu nhanh bằng `amq.rabbitmq.reply-to` (không cần `queue_declare`) như sau:
>
> ```python
> # rpc_client_direct_reply_to.py (Python 3.10+ & Pika 1.3+ & RabbitMQ 3.8–4.0)
> import json, time, uuid, pika
>
> creds = pika.PlainCredentials("rpc_user", "rpcme")
> conn = pika.BlockingConnection(pika.ConnectionParameters("localhost", credentials=creds))
> ch = conn.channel()
>
> corr_id = str(uuid.uuid4())
>
> def on_reply(channel, method, props, body):
>     if props.correlation_id == corr_id:
>         print("Direct Reply-to RPC Reply ---", body.decode("utf-8"))
>         channel.stop_consuming()
>
> # Bước 1 (BẮT BUỘC TRƯỚC KHI PUBLISH): Consume từ 'amq.rabbitmq.reply-to' với auto_ack=True!
> # Lưu ý: basic_consume chỉ đăng ký callback trên channel chứ chưa chặn luồng (chưa gọi start_consuming)
> ch.basic_consume(
>     queue="amq.rabbitmq.reply-to",
>     on_message_callback=on_reply,
>     auto_ack=True,  # Direct Reply-to bắt buộc phải bật auto_ack=True
> )
>
> # Bước 2: Publish yêu cầu RPC với reply_to="amq.rabbitmq.reply-to"
> msg = json.dumps({"client_name": "Modern Direct-Reply Client", "time": time.time()}).encode("utf-8")
> ch.basic_publish(
>     exchange="rpc",
>     routing_key="ping",
>     properties=pika.BasicProperties(
>         reply_to="amq.rabbitmq.reply-to",
>         correlation_id=corr_id,
>         content_type="application/json",
>     ),
>     body=msg,
> )
> print("Sent 'ping' RPC call via Direct Reply-to. Waiting for reply...")
>
> # Bước 3: Bắt đầu vòng lặp chờ câu trả lời
> ch.start_consuming()
> conn.close()
> ```
> **Mẹo nhỏ về Pika:** Trong sách (đoạn giải thích Listing 4.13), tác giả dặn *"đừng gọi `basic_consume` trước khi `basic_publish`"*. Thực ra trong Pika, chỉ có hàm **`channel.start_consuming()`** mới chặn luồng để vào vòng lặp I/O, còn hàm **`channel.basic_consume(...)`** chỉ gửi frame `Basic.Consume` lên broker rồi trả về ngay! Do đó, với Direct Reply-to (`amq.rabbitmq.reply-to`), bạn hoàn toàn gọi `ch.basic_consume(...)` trước, sau đó gọi `ch.basic_publish(...)`, rồi cuối cùng mới gọi `ch.start_consuming()` như đoạn code trên!

---

## 4.4 Tóm tắt (Summary)

Trong chương này, chúng ta đã tìm hiểu những cách nền tảng để viết các ứng dụng tận dụng sức mạnh của RabbitMQ cũng như các mẫu nhắn tin (*messaging patterns*) đứng đằng sau chúng. Chúng ta đã thảo luận đủ mọi thứ: từ các mẫu "bắn và quên" (*fire-and-forget*), như gửi cảnh báo và xử lý ảnh, cho đến giao tiếp hai chiều thực thụ (*bidirectional communication*) vận hành các API RPC. Với những kiến trúc nhắn tin nền tảng này trong tay, bạn hoàn toàn tự do bắt đầu thiết kế những mẫu phức tạp của riêng mình — kết hợp các nguyên lý cơ bản lại thành những giải pháp độc đáo nhằm đạt được các mục tiêu cụ thể của bạn. Giờ đây, khi bạn đã bắt đầu đưa RabbitMQ vào trái tim trong kiến trúc ứng dụng của mình, đã đến lúc chúng ta tìm hiểu cách vận hành RabbitMQ trong những cấu hình có tính chống chịu lỗi cao (*resilient configurations*), đảm bảo nó luôn sẵn sàng bất cứ khi nào ứng dụng của bạn cần đến.

---

## Bảng thuật ngữ của chương

| Thuật ngữ tiếng Anh | Nghĩa tiếng Việt | Giải thích ngắn |
|---|---|---|
| **Asynchronous** | Bất đồng bộ | Mô hình trong đó bên gửi yêu cầu không đứng chờ hành động hoàn tất mà tiếp tục làm việc khác ngay sau khi gửi thông điệp. |
| **Anonymous queue** | Hàng đợi ẩn danh | Queue được khai báo với tên rỗng (`""`), để RabbitMQ tự sinh một tên duy nhất (thường có dạng `amq.gen-...`). |
| **Auto-delete (`auto_delete`)** | Tự động xóa | Cờ cấu hình cho phép exchange hoặc queue tự động bị xóa khi consumer cuối cùng ngắt kết nối khỏi nó. |
| **Callback function** | Hàm gọi lại (hàm callback) | Hàm do lập trình viên định nghĩa và đăng ký với thư viện AMQP để được tự động gọi mỗi khi có message mới giao tới. |
| **Consumer tag (`consumer_tag`)** | Thẻ định danh consumer | Chuỗi định danh duy nhất cho một lượt đăng ký nhận message (`basic_consume`) trên một channel; dùng khi cần hủy đăng ký (`basic_cancel`). |
| **Decoupling** | Tách rời (giảm phụ thuộc) | Thiết kế hệ thống sao cho các thành phần không phụ thuộc trực tiếp vào nhau (tách việc nhận yêu cầu khỏi việc thực thi hành động). |
| **Delivery mode (`delivery_mode`)** | Chế độ giao nhận / lưu trữ | Thuộc tính của message AMQP: `1` là không bền vững (chỉ nằm trong RAM), `2` là bền vững (*persistent* – được ghi xuống đĩa khi nằm trong durable queue). |
| **Delivery tag (`delivery_tag`)** | Thẻ giao nhận | Số thứ tự định danh một lần giao message trên channel, được consumer gửi kèm trong `basic_ack` để báo cho RabbitMQ biết đã xử lý xong message nào. |
| **Exclusive queue (`exclusive`)** | Hàng đợi độc quyền | Queue chỉ cho phép duy nhất kết nối (connection) đã tạo ra nó được phép đọc/tiêu thụ, và tự xóa khi kết nối đó đóng. |
| **Fanout exchange** | Bộ trao đổi kiểu quạt (fanout) | Loại exchange sao chép và gửi mọi message nhận được tới tất cả các queue đang được bind với nó, bỏ qua routing key. |
| **Fire-and-forget** | Bắn và quên | Mẫu nhắn tin một chiều trong đó producer chỉ gửi message đi để nơi khác xử lý mà không cần chờ hay nhận lại kết quả. |
| **Flow control** | Điều khiển luồng | Cơ chế của RabbitMQ nhằm điều tiết (tạm chặn) các producer khi broker bị quá tải bộ nhớ hoặc tài nguyên. |
| **Multicast** | Phát đa hướng | Gửi một thông điệp từ một nguồn tới nhiều đích nhận đồng thời (trong RabbitMQ được hiện thực qua fanout hoặc topic exchange). |
| **Parallel processing** | Xử lý song song | Thực thi đồng thời nhiều tác vụ độc lập nhau (hoặc chia tải cùng một loại tác vụ cho nhiều worker chạy cùng lúc). |
| **Reply queue (`reply_to`)** | Hàng đợi nhận trả lời / trường `reply_to` | Queue tạm thời do RPC client tạo ra để nhận kết quả trả về; tên của nó được đặt vào thuộc tính header `reply_to` của message yêu cầu. |
| **RPC (Remote Procedure Call)** | Lời gọi thủ tục từ xa | Mẫu giao tiếp hai chiều trong đó client gửi yêu cầu thực thi một hàm tới server ở xa và chờ nhận lại kết quả trả về. |
| **Synchronous** | Đồng bộ | Mô hình trong đó bên gọi phải đứng chờ (bị chặn – *block*) cho tới khi tác vụ hoàn thành và trả về kết quả mới đi tiếp. |
| **Topic exchange** | Bộ trao đổi theo chủ đề (topic) | Loại exchange định tuyến message dựa trên việc khớp mẫu ký tự đại diện (`*` cho một từ, `#` cho không hoặc nhiều từ, phân tách bởi dấu `.`) giữa routing key và binding key. |
