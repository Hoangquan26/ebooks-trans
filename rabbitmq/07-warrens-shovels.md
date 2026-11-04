# Chương 7: Warren và Shovel: chuyển đổi dự phòng và sao chép (Warrens and Shovels: failover and replication)

## Mục lục chương

- [Mở đầu chương](#mở-đầu-chương)
- [7.1 Warren: một cách khác để "gom cụm" (Warrens: another way of clustering)](#71-warren-một-cách-khác-để-gom-cụm-warrens-another-way-of-clustering)
- [7.2 Dựng cụm master/slave dựa trên bộ cân bằng tải (Setting up load balancer–based master/slave clusters)](#72-dựng-cụm-masterslave-dựa-trên-bộ-cân-bằng-tải-setting-up-load-balancerbased-masterslave-clusters)
- [7.3 Giao tiếp và sao chép đường dài (Long-distance communication and replication)](#73-giao-tiếp-và-sao-chép-đường-dài-long-distance-communication-and-replication)
  - [7.3.1 "Xúc" thỏ: giới thiệu plugin Shovel (Shoveling your Rabbits: an introduction to the Shovel plugin)](#731-xúc-thỏ-giới-thiệu-plugin-shovel-shoveling-your-rabbits-an-introduction-to-the-shovel-plugin)
  - [7.3.2 Cài đặt Shovel (Installing Shovel)](#732-cài-đặt-shovel-installing-shovel)
  - [7.3.3 Cấu hình và chạy Shovel (Configuring and running Shovel)](#733-cấu-hình-và-chạy-shovel-configuring-and-running-shovel)
- [7.4 Tóm tắt (Summary)](#74-tóm-tắt-summary)
- [Bảng thuật ngữ của chương](#bảng-thuật-ngữ-của-chương)

---

## Mở đầu chương

**Chương này bao gồm (This chapter covers):**

- Hiểu về các cặp máy chủ chủ động/dự phòng (*active/standby pairs*) — mà sách gọi là **warren**
- Tạo warren bằng bộ cân bằng tải (*load balancer*)
- Xây dựng cơ chế sao chép đường dài (*long-distance replication*) bằng **Shovel**

Cho tới giờ, mỗi khi nói về tính sẵn sàng cao (*high availability* – khả năng hệ thống vẫn phục vụ được khi có sự cố), chúng ta luôn đặt nó trong bối cảnh tính năng gom cụm (*clustering*) có sẵn của RabbitMQ. Nhưng clustering không phải là cách duy nhất để tăng khả năng chống chịu (*resiliency*) cho hạ tầng Rabbit của bạn, và tùy nhu cầu, nó cũng không phải lúc nào cũng là cách đúng.

Clustering bắt bạn phải đánh đổi: bạn được lợi ở chỗ tất cả các node hoạt động như **một khối duy nhất** để chia tải, nhưng đổi lại bạn chịu nhược điểm là **không dùng được các queue bền vững (*durable queue*) nằm trên những node đang bị sập** cho tới khi các node đó được khôi phục. Ngoài ra, clustering cũng không cho bạn thứ cần thiết để xây dựng một kiến trúc RabbitMQ trải rộng trên **nhiều hơn một trung tâm dữ liệu** (*data center*).

Vì vậy, dù thoạt nghe clustering có vẻ như một con dao đa năng Thụy Sĩ (*Swiss army knife*) giải quyết mọi vấn đề về tính sẵn sàng, bạn vẫn cần thêm vài công cụ khác trong hộp đồ nghề. Đó chính là lúc **warren** và **Shovel** xuất hiện.

Tận dụng những kiến thức bạn đã tích lũy cho tới giờ, bạn sẽ học cách xây dựng các **cặp active/standby** gồm những máy chủ RabbitMQ độc lập (*standalone*), cho phép bạn đánh đổi khả năng mở rộng (*scalability*) lấy sự linh hoạt cao hơn trong việc nhắn tin bền vững (*durable messaging*). Sau đó, bạn sẽ thấy cách dùng plugin **Shovel** để sao chép nội dung các queue trên một máy chủ Rabbit ở thành phố này, qua một khoảng cách xa, tới một máy chủ Rabbit (hoặc một cluster) ở thành phố khác. Khi học xong, bạn sẽ có một bộ đồ nghề hoàn chỉnh cho mọi tình huống cần tính sẵn sàng cao.

Tuyệt nhất là: nếu bạn đã thiết kế ứng dụng theo các kỹ thuật mới học ở chương trước (tự kết nối lại, không giả định gì về trạng thái máy chủ), bạn có thể dùng **nguyên xi, không sửa gì** các ứng dụng đó để tận dụng những mô hình (*topology*) RabbitMQ mới này. Hãy bắt đầu bằng việc tìm hiểu các cặp active/standby, hay như cách chúng tôi gọi: **warren**.

> 📝 **Giải thích thêm (không có trong sách):** "Warren" trong tiếng Anh nghĩa là *hang thỏ* — một hệ thống nhiều hang thỏ nối với nhau. Tác giả chơi chữ vì RabbitMQ có biểu tượng là con thỏ (*rabbit*). Còn "shovel" nghĩa là *cái xẻng* — plugin Shovel "xúc" tin nhắn từ máy chủ này sang máy chủ khác. Trong bản dịch, chúng tôi giữ nguyên hai từ **warren** và **Shovel** vì đây là tên gọi riêng.

---

## 7.1 Warren: một cách khác để "gom cụm" (Warrens: another way of clustering)

Ở các phiên bản RabbitMQ trước 1.8.0 có một hành vi khá "thú vị" (nói mỉa) khi một node trong cluster chứa một **durable queue** bị sập. Nếu một client **tạo lại** queue durable đó trong lúc node vẫn đang sập, thì **toàn bộ nội dung của queue cũ sẽ bị mất** khi node sập được bật lại. Node vừa được khôi phục về cơ bản sẽ nói: "Ối, queue này đã tồn tại rồi; tôi không cần bản sao của mình nữa." Khi nội dung của queue cũ là thứ quý giá, đây là một vấn đề cực lớn.

Kể từ phiên bản 1.8.0, một hành vi khác đã thay thế. Khi một node chứa durable queue bị sập, queue đó **không thể được tạo lại**. Bất kỳ client nào cố gắng khai báo lại (*redeclare*) queue đó sẽ nhận lỗi AMQP **404 NOT_FOUND**. Khi node sập được khôi phục, durable queue và nội dung của nó cũng được khôi phục theo (với điều kiện các tin nhắn đã được gửi với `delivery_mode` bằng 2, tức tin nhắn bền vững/được ghi xuống đĩa). Nhưng **cho tới khi node đó được khôi phục**, mọi tin nhắn lẽ ra được giao tới queue đó sẽ hoặc bị "rơi vào lỗ đen" (*black-holed* – bị vứt bỏ âm thầm), hoặc client sẽ nhận lỗi nếu client có bật cờ `mandatory` khi publish.

> 💡 **Ví dụ bổ sung (không có trong sách):** Giả sử bạn có cluster 3 node `rabbit@n1`, `rabbit@n2`, `rabbit@n3`. Queue durable `orders` được tạo trên `n2`. Lúc 10:00, `n2` mất điện.
> - Với RabbitMQ < 1.8.0: một app khác lúc 10:05 gọi `queue_declare("orders", durable=True)` → queue `orders` mới (rỗng) được tạo trên `n1`. Lúc 10:30 `n2` sống lại, thấy `orders` đã tồn tại → **vứt bỏ** 5.000 đơn hàng cũ đang nằm trên đĩa của nó. Mất dữ liệu!
> - Với RabbitMQ ≥ 1.8.0: lúc 10:05, lệnh `queue_declare("orders", durable=True)` trả về lỗi `404 NOT_FOUND`. Các tin nhắn publish tới exchange có binding tới `orders` trong khoảng 10:00–10:30 sẽ bị bỏ (nếu không đặt `mandatory`) hoặc bị trả lại cho producer (nếu đặt `mandatory=True`). Lúc 10:30 `n2` sống lại, 5.000 đơn cũ (gửi với `delivery_mode=2`) vẫn còn nguyên. Dữ liệu cũ an toàn, nhưng trong 30 phút đó hệ thống không nhận được đơn mới vào queue này.

Nếu ứng dụng của bạn **không thể chấp nhận rủi ro mất tin nhắn**, hoặc không chịu nổi độ trễ (*latency*) của việc liên tục publish lại tin nhắn cho tới khi queue bị sập quay trở lại, thì bạn cần thứ mà chúng tôi gọi là **warren**. Theo cách nói của chúng tôi, một **warren** là **một cặp máy chủ độc lập (standalone) kiểu active/standby, với một bộ cân bằng tải đặt phía trước để xử lý việc chuyển đổi dự phòng (*failover*)** (xem hình 7.1).

Ưu điểm của cách bố trí này là nó thực sự **không chia sẻ gì cả** (*shared-nothing*). Giữa máy chủ active (đang hoạt động) và máy chủ standby (dự phòng) không có sự phối hợp nào, nên bất kỳ sự cố nào ảnh hưởng tới máy active sẽ không tự động lan sang máy standby, và ngược lại. Sự tách biệt giữa chúng hoàn toàn tới mức bạn thậm chí có thể **chạy hai phiên bản RabbitMQ khác nhau** trên hai máy. Điều này cho phép bạn triển khai một phiên bản RabbitMQ mới lên môi trường production trong khi vẫn giữ phiên bản cũ bên cạnh để phòng hờ. Có rất nhiều lý do và tình huống mà việc có hai máy chủ RabbitMQ hoàn toàn độc lập, sẵn sàng gánh tải cho nhau, là có lợi. Dù lý do là gì, một warren có thể rất hữu ích khi tính năng clustering có sẵn của Rabbit không hoàn toàn đáp ứng được yêu cầu.

Sẽ là thiếu sót nếu chúng tôi không nhắc tới việc còn **một cách khác** để dựng warren cho tính sẵn sàng cao. Một trường phái khác lập luận: "Tôi muốn node standby của tôi có **tất cả** các tin nhắn đã nằm trong node active vào lúc nó gặp sự cố." Cách tiếp cận của chúng tôi — dùng bộ cân bằng tải và kiến trúc shared-nothing — **không** cho bạn điều đó. Thay vào đó, cách của chúng tôi cho bạn **ngay lập tức một nơi để tiếp tục publish và consume tin nhắn**, và khi bạn khôi phục node active, nó cho phép các consumer kết nối lại và "rút cạn" (*drain*) những tin nhắn đã nằm trong queue lúc node active bị sập. Bạn không mất tin nhắn nào, dù cũ hay mới, nhưng bạn phải chờ node active được khôi phục thì các tin nhắn cũ mới dùng được trở lại.

![Mảnh ảnh trích từ Hình 7.1 – biểu tượng consumer](images/07-warrens-shovels/p147-1.png)
![Mảnh ảnh trích từ Hình 7.1 – nền hình tròn (node RabbitMQ)](images/07-warrens-shovels/p147-2.png)
![Mảnh ảnh trích từ Hình 7.1 – biểu tượng thiết bị cân bằng tải](images/07-warrens-shovels/p147-3.png)

*(Ba ảnh nhỏ ở trên là các mảnh đồ họa được trích tự động từ Hình 7.1 – biểu tượng consumer, nền hình tròn của node và biểu tượng thiết bị phần cứng. Hình hoàn chỉnh xem ở ảnh trang render bên dưới.)*

**Hình 7.1 – Warren dựa trên bộ cân bằng tải (Load balancer–based warren)**

![Hình 7.1 – trang sách render (trang 147)](images/pages/page-147.png)

```mermaid
flowchart TD
    C["Consumer"] <--> LB["Bộ cân bằng tải (load balancer)"]
    LB <-->|"Node chính (Primary node)"| A
    LB -.->|"Node phụ (Secondary node) – chỉ dùng nếu node chính hỏng"| B
    subgraph A["RabbitMQ standalone server rabbit_a@localhost"]
        EA["Exchange A"]
        Q1["Queue 1"]
    end
    subgraph B["RabbitMQ standalone server rabbit_b@localhost"]
        EB["Exchange B"]
        Q2["Queue 2"]
    end
```

*Mô tả hình:* Trên cùng là một **Consumer**, nối hai chiều với **bộ cân bằng tải** (vẽ như một thiết bị phần cứng gắn rack). Bộ cân bằng tải nối bằng mũi tên liền tới **Node chính (Primary node)** — máy chủ RabbitMQ độc lập `rabbit_a@localhost` (hình tròn màu xanh lá đậm) chứa **Exchange A** và **Queue 1**. Nó cũng nối bằng mũi tên đứt nét tới **Node phụ (Secondary node – chỉ dùng nếu node chính hỏng)** — máy chủ RabbitMQ độc lập `rabbit_b@localhost` (hình tròn xanh nhạt) chứa **Exchange B** và **Queue 2**. Hai node hoàn toàn độc lập, không có đường nối nào giữa chúng.

Trường phái kia cho rằng bạn nên xây warren bằng **bộ lưu trữ dùng chung** (*shared storage*) giữa máy active và standby, với RabbitMQ **không chạy** trên node standby (xem hình 7.2). Khi máy active gặp sự cố, bạn dùng **Pacemaker**¹ để chuyển địa chỉ IP của RabbitMQ sang node standby, rồi khởi động Rabbit trên node đó để nó nhặt lại metadata (siêu dữ liệu: định nghĩa queue, exchange, binding...), nội dung và trạng thái hiện tại từ bộ lưu trữ dùng chung.

Theo ý kiến của chúng tôi, cách bố trí đó có vài vấn đề:

- **Thứ nhất**, bộ lưu trữ được dùng chung, nên nếu có một kiểu hỏng dữ liệu (*corruption*) nào đó giết chết node active, thì chính chỗ hỏng đó cũng sẽ có mặt trên node standby và ngăn RabbitMQ khởi động ở đó.
- **Thứ hai**, bạn cần chắc chắn rằng RabbitMQ ở node standby có **cùng tên node và cùng UID** (mã định danh người dùng của hệ điều hành) với RabbitMQ trên node chính. Nếu một trong hai điều này không đúng, Rabbit dự phòng sẽ không truy cập được các file trên bộ lưu trữ dùng chung và không thể khởi động.
- **Cuối cùng**, dùng cách này cho warren nghĩa là Rabbit dự phòng của bạn **thực ra không chạy**. Vì vậy có khả năng là một điều gì đó đã thay đổi trên node standby, khiến Rabbit không khởi động được đúng vào lúc bạn cần nó.

Đó là quá nhiều sự phức tạp, mà chúng tôi thì thích sự đơn giản.

Do việc hỏng dữ liệu bị nhân bản sang cả hai bên, và do không có một RabbitMQ chạy đầy đủ trên cả hai node như trong cách dùng bộ lưu trữ chung, chúng tôi ưu tiên **warren dựa trên bộ cân bằng tải**, và đó là kiểu warren chúng tôi sẽ hướng dẫn bạn. Nhưng nếu warren dùng bộ lưu trữ chung nghe hấp dẫn hơn hoặc phù hợp với trường hợp của bạn hơn, có một bài hướng dẫn tốt trên website RabbitMQ giải thích cách dựng nó: http://www.rabbitmq.com/pacemaker.html. Cuối cùng, nếu bạn đã xây dựng ứng dụng theo nguyên tắc **"không giả định gì cả"** (*assume nothing*) và luôn dựng lại "kết cấu" (*fabric* – tập hợp exchange, queue, binding mà app cần) mỗi khi app kết nối lại, thì app của bạn có thể dùng warren dựa trên bộ cân bằng tải **mà không cần thay đổi gì**.

> ¹ **Chú thích:** Pacemaker là một bộ tiện ích cluster cho Linux, xử lý việc chuyển IP (*IP failover*) giữa node active và node standby, đồng thời tự động khởi động ứng dụng được bảo vệ trên node standby khi có sự cố. Pacemaker có tại http://www.clusterlabs.org/.

![Mảnh ảnh trích từ Hình 7.2 – biểu tượng consumer](images/07-warrens-shovels/p148-1.png)
![Mảnh ảnh trích từ Hình 7.2 – nền hình tròn node 1](images/07-warrens-shovels/p148-2.png)
![Mảnh ảnh trích từ Hình 7.2 – chi tiết đồ họa](images/07-warrens-shovels/p148-3.png)
![Mảnh ảnh trích từ Hình 7.2 – nền hình tròn node 2](images/07-warrens-shovels/p148-6.png)
![Mảnh ảnh trích từ Hình 7.2 – chi tiết đồ họa](images/07-warrens-shovels/p148-7.png)
![Mảnh ảnh trích từ Hình 7.2 – biểu tượng bộ lưu trữ](images/07-warrens-shovels/p148-9.png)

*(Các ảnh nhỏ ở trên là các mảnh đồ họa được trích tự động từ Hình 7.2. Hình hoàn chỉnh xem ở ảnh trang render bên dưới.)*

**Hình 7.2 – Warren dựa trên bộ lưu trữ dùng chung (Shared storage–based warren)**

![Hình 7.2 – trang sách render (trang 148)](images/pages/page-148.png)

```mermaid
flowchart TD
    C["Consumer"] <--> VIP["IP ảo (Virtual IP): 10.1.61.1"]
    VIP <--> N1
    VIP -.-> N2
    PM["Pacemaker (điều khiển IP ảo, sở hữu bộ lưu trữ, khởi động rabbit dự phòng)"]
    subgraph N1["RabbitMQ standalone server rabbit@10.1.62.2"]
        EA1["Exchange A"]
        QA1["Queue 1"]
    end
    subgraph N2["RabbitMQ standalone server rabbit@10.1.62.3"]
        EA2["Exchange A"]
        QA2["Queue 1"]
    end
    N1 <--> S["Bộ lưu trữ dùng chung (Shared storage)"]
    N2 -.-> S
```

*Mô tả hình:* **Consumer** kết nối tới **IP ảo (Virtual IP) 10.1.61.1**. IP ảo này đang trỏ (mũi tên liền) tới node chính `rabbit@10.1.62.2`, và có thể chuyển (mũi tên đứt) sang node dự phòng `rabbit@10.1.62.3`. Cả hai node đều có **cùng** Exchange A và Queue 1 (vì cùng đọc từ một kho dữ liệu). Ở giữa là khung vàng **Pacemaker** (điều khiển IP ảo, là chủ sở hữu bộ lưu trữ, khởi động rabbit dự phòng). Phía dưới là **bộ lưu trữ dùng chung (Shared storage)**: node chính đang kết nối (mũi tên liền), node dự phòng chỉ kết nối khi cần (mũi tên đứt).

> 💡 **Ví dụ bổ sung (không có trong sách):** So sánh hai kiểu warren qua một kịch bản cụ thể. Node chính có 1.000 tin nhắn trong queue `orders` thì bị sập do hỏng file dữ liệu Mnesia.
>
> | Tiêu chí | Warren dùng load balancer (shared-nothing) | Warren dùng shared storage + Pacemaker |
> |---|---|---|
> | Sau failover, có chỗ để publish/consume ngay? | Có — `rabbit_b` đã chạy sẵn | Phải chờ Pacemaker chuyển IP và khởi động Rabbit |
> | 1.000 tin nhắn cũ | Vẫn nằm trên đĩa của `rabbit_a`, dùng lại được khi `rabbit_a` được sửa | Sẵn ngay trên node dự phòng **nếu** dữ liệu không hỏng |
> | Nếu nguyên nhân là file dữ liệu bị hỏng | `rabbit_b` không bị ảnh hưởng | Node dự phòng đọc đúng file hỏng đó → cũng không khởi động được |
> | Biết trước node dự phòng có "khỏe" không? | Có — giám sát được vì nó luôn chạy | Không — nó chỉ chạy khi cần |
>
> Đây chính là lý do tác giả chọn kiểu dùng load balancer.

> 📝 **Giải thích thêm (không có trong sách) – Kiến trúc "Warren" (2012) vs Quorum Queues & Federation trong RabbitMQ hiện đại (3.8–4.0):**
>
> Tại sao năm 2012 (thời RabbitMQ 2.7), các tác giả lại phải sáng tạo ra mô hình **"Warren"** (đặt 2 broker độc lập sau HAProxy hoặc dùng Pacemaker + ổ đĩa chung)?
> 1. **Hạn chế lịch sử của RabbitMQ 2.x:** Ở thời điểm đó, một queue thông thường chỉ nằm trên **một node duy nhất** trong cluster. Nếu node đó sập, queue bị khóa hoàn toàn (`404 NOT_FOUND`) để bảo toàn dữ liệu trên đĩa. Tính năng *Classic Mirrored Queues* (mới ra mắt ở 2.6.0) lúc bấy giờ còn non trẻ, hiệu năng đồng bộ thấp, dễ bị treo khi mạng chập chờn (*split-brain*) và đôi khi sự cố Erlang/Mnesia trên một node có thể kéo sập cả cụm.
> 2. **Nhược điểm chí mạng của "Warren" khi nhìn từ chuẩn mực hiện đại:**
>    - Với **Warren dùng Load Balancer (shared-nothing)**: Khi `rabbit_a` sập, toàn bộ các tin nhắn đang nằm trong `rabbit_a` bị **"đóng băng"** cho đến khi kỹ sư hệ thống sửa xong máy chủ `rabbit_a`. Hơn nữa, khi `rabbit_a` sống lại, HAProxy chuyển kết nối mới về `rabbit_a`, khiến các tin nhắn vừa nhận tạm trên `rabbit_b` có nguy cơ bị "bỏ quên" nếu không có consumer gắn riêng vào `rabbit_b` để rút cạn (*drain*)!
>    - Với **Warren dùng Pacemaker + Shared Storage (SAN/NFS)**: Đây hiện được đội ngũ RabbitMQ xếp vào danh sách **Anti-Pattern (không khuyến khích)** vì thời gian khởi động lại lâu (Cold Standby), ổ đĩa mạng chung (NFS/SAN) trở thành điểm chết duy nhất (*Single Point of Failure*), và file khóa Mnesia/Khepri rất dễ hỏng khi hai node tranh chấp quyền ghi (*fencing failure*).
>
> **Trong RabbitMQ hiện đại (3.8–3.13 và RabbitMQ 4.0), người ta thay thế "Warren" bằng gì?**
> - **Bên trong cùng một Trung tâm dữ liệu / Vùng đám mây (Multi-AZ trong 1 Region):** Thay vì dựng 2 node độc lập kiểu Warren, kiến trúc chuẩn công nghiệp là dựng **một Cluster 3 hoặc 5 node** và sử dụng **Quorum Queues** (`x-queue-type: quorum`) hoặc **Streams** (`x-queue-type: stream`).
>   - Quorum Queue dùng thuật toán đồng thuận **Raft**: dữ liệu luôn được ghi đồng thời lên quá bán số node ($2/3$ hoặc $3/5$ node) trước khi xác nhận cho Producer.
>   - Khi 1 node sập, các node còn lại bầu chọn Leader mới chỉ trong **vài phần nghìn giây đến dưới 1 giây**, và **toàn bộ 1.000 tin nhắn cũ lẫn tin nhắn mới đều được phục vụ ngay lập tức** mà không mất một tin nhắn nào, cũng không cần chờ node chết sống lại!
> - **Giữa hai Trung tâm dữ liệu hoặc khi cần cô lập lỗi tuyệt đối (Active/Standby跨 DC):** Nếu bạn thực sự muốn hai cụm RabbitMQ tách biệt hoàn toàn (shared-nothing) để phòng thảm họa cấp cụm hoặc nâng cấp Blue-Green, trong RabbitMQ hiện đại bạn kết hợp:
>   1. Hai cụm (mỗi cụm 3 node chạy Quorum Queue) đặt sau Load Balancer hoặc DNS.
>   2. Bật **Federation Plugin (`rabbitmq_federation`)** ở chế độ *Queue Federation* hoặc *Exchange Federation*, hoặc dùng **Dynamic Shovel**, hoặc tính năng **Warm Standby Replication** (trên VMware Tanzu RabbitMQ) để tự động đồng bộ hoặc hút cạn (*drain*) tin nhắn từ cụm Standby về cụm Active mà ứng dụng không cần lo bị "bỏ quên" tin nhắn.
>
> **Bảng đối chiếu toàn diện: Warren (Sách) vs Kiến trúc RabbitMQ hiện đại (3.8–4.0):**
>
> | Tiêu chí | Warren + HAProxy (Sách – 2012) | Warren + Pacemaker/SAN (Sách – 2012) | Cluster 3 Node + Quorum Queues (Hiện đại 3.8–4.0) | 2 Cụm độc lập + Queue Federation / Shovel (Hiện đại) |
> |---|---|---|---|---|
> | **Mục đích chính** | Có nơi publish/consume ngay khi node chính sập | Giữ lại tin nhắn cũ trên đĩa chung khi node chính sập | **HA chuẩn mực trong 1 DC / Multi-AZ** | **Disaster Recovery (DR) liên DC / Nâng cấp Blue-Green** |
> | **Tin nhắn cũ khi node chính sập** | Bị kẹt trên đĩa của node sập cho tới khi sửa xong máy | Đọc lại được sau khi Pacemaker mount đĩa & bật broker (mất 30s–vài phút) | **Sẵn sàng ngay lập tức** trên node Follower vừa lên làm Leader | Tự động được kéo sang cụm đang có Consumer nhờ Queue Federation / Shovel |
> | **Nguy cơ mất hoặc "bỏ quên" tin nhắn** | Cao nếu quên nối consumer vào node `rabbit_b` sau khi `rabbit_a` sống lại | Nguy cơ hỏng database nếu 2 node cùng mount ổ đĩa (split-brain) | **Không mất dữ liệu** (nhờ Raft Quorum + Publisher Confirms) | Không bị bỏ quên (Queue Federation tự chuyển tiếp khi có consumer) |
> | **Khuyến nghị năm 2026** | Chỉ dùng khi kết hợp thêm Queue Federation / Shovel để rút cạn tin nhắn | **Đã lỗi thời (Deprecated / Anti-pattern)** | **Khuyến nghị mặc định cho mọi hệ thống Production** | **Khuyến nghị cho hệ thống Đa vùng (Multi-Region)** |

---

## 7.2 Dựng cụm master/slave dựa trên bộ cân bằng tải (Setting up load balancer–based master/slave clusters)

Thực tế, việc dựng một warren dựa trên bộ cân bằng tải rất đơn giản và được xây dựng trên tất cả các khái niệm bạn đã nắm được. Trước tiên, bạn sẽ khởi động hai node RabbitMQ trên máy phát triển của mình và đặt tên là `rabbit_a` và `rabbit_b` để chúng không xung đột với các node cluster cục bộ mà bạn đang chạy (chạy các lệnh sau từ thư mục cài đặt Rabbit):

```bash
$ RABBITMQ_NODE_PORT=5675 RABBITMQ_NODENAME=rabbit_a \
./sbin/rabbitmq-server -detached
Activating RabbitMQ plugins ...
$ RABBITMQ_NODE_PORT=5676 RABBITMQ_NODENAME=rabbit_b \
./sbin/rabbitmq-server -detached
Activating RabbitMQ plugins ...
```

> 📝 **Giải thích thêm (không có trong sách):** Biến môi trường `RABBITMQ_NODE_PORT` chỉ định cổng AMQP mà node lắng nghe (5675 cho `rabbit_a`, 5676 cho `rabbit_b`), còn `RABBITMQ_NODENAME` đặt tên node Erlang. Tùy chọn `-detached` cho node chạy nền, trả lại dấu nhắc lệnh cho bạn. Nhờ khác cổng và khác tên, hai node có thể cùng chạy trên một máy mà không đụng nhau — và quan trọng là chúng **không** được join vào cùng một cluster, nên hoàn toàn độc lập.

Với hai node active/standby đã chạy, giờ bạn cần thiết lập một cấu hình HAProxy mới để coi `rabbit_b` là **máy chủ dự phòng** (*backup server*), để nó chỉ được dùng khi `rabbit_a` sập. Thực ra, cấu hình warren cho HAProxy (xem listing sau) trông giống đến bất ngờ với cấu hình cluster.

**Listing 7.1 – Cấu hình HAProxy cho một warren dựa trên bộ cân bằng tải**

```bash
# HAProxy Config for Local RabbitMQ SLB Warren

global
    log 127.0.0.1    local0 info
    maxconn 4096
    stats socket /tmp/haproxy_2.socket uid haproxy mode 770 level admin
    daemon

defaults
    log    global
    mode    tcp
    option    tcplog
    option    dontlognull
    retries    3
    option redispatch
    maxconn    2000
    timeout connect   5s
    timeout client 120s
    timeout server 120s

listen rabbitmq_local_cluster 127.0.0.1:5680
    mode tcp
    balance roundrobin
    server rabbit_a 127.0.0.1:5675 check inter 5000 rise 2 fall 3
    server rabbit_b 127.0.0.1:5676 backup check inter 5000 rise 2 fall 3

listen private_monitoring :8101
    mode http
    option httplog
    stats enable
    stats uri /stats
    stats refresh 5s
```

Các chú thích (callout) bên cạnh listing trong sách:

- **Khối `global`** – *Các tùy chọn ghi log (Logging options)*.
- **Khối `defaults`** – *Các thiết lập mặc định cho cân bằng tải (Load balancing defaults)*.
- **❶ Dòng `listen rabbitmq_local_cluster 127.0.0.1:5680`** – *IP frontend cho consumer và producer (Frontend IP for consumers and producers)*: địa chỉ mà các ứng dụng kết nối vào.
- **`mode tcp` / `balance roundrobin`** – *Các tùy chọn cân bằng tải (Load balancing options)*.
- **Dòng `server rabbit_a ...`** – *Node active (Active node)*.
- **❷ Dòng `server rabbit_b ... backup ...`** – *Node dự phòng (Backup node)*.
- **❸ Khối `listen private_monitoring :8101`** – *Trang thống kê (Statistics page)*.

Ở đây, socket cục bộ của HAProxy ❶ và trang thống kê ❸ đã được đổi để tránh xung đột với instance HAProxy dùng cho cluster Rabbit (ở chương trước). Tuy nhiên, thay đổi thú vị hơn là ❷: việc thêm một tùy chọn cấu hình HAProxy mới — **`backup`**. Khi bạn thêm `backup` vào chỉ thị (*directive*) `server` của một backend, bạn đang bảo HAProxy **chỉ dùng backend đó khi tất cả các server không-phải-backup đều không khả dụng**.

> 📝 **Giải thích thêm (không có trong sách):** Trong sách, phần văn bản gán ❶ cho "socket cục bộ" (dòng `stats socket /tmp/haproxy_2.socket ...`), trong khi nhãn callout ❶ cạnh listing lại chỉ vào dòng frontend `listen ... 127.0.0.1:5680`. Thực tế so với cấu hình cluster ở chương trước, cả tên file socket (`haproxy_2.socket`), cổng frontend (5680) và cổng thống kê (8101) đều được đổi để hai instance HAProxy chạy song song trên cùng một máy không giành nhau tài nguyên.
>
> Ý nghĩa các tham số trên dòng `server`: `check` = bật kiểm tra sức khỏe (*health check*); `inter 5000` = kiểm tra mỗi 5000 ms (5 giây); `rise 2` = cần 2 lần kiểm tra thành công liên tiếp để coi server là "sống" lại; `fall 3` = cần 3 lần thất bại liên tiếp để coi server là "chết". `option redispatch` cho phép HAProxy chuyển kết nối sang server khác nếu server đã chọn bị lỗi; `retries 3` = thử lại tối đa 3 lần.

Nếu bạn xem trang thống kê HAProxy của warren (xem hình 7.3) bằng cách vào http://localhost:8101/stats, bạn sẽ thấy `rabbit_b` có **màu xanh dương nhạt** thay vì màu xanh lá nhạt thông thường của một server khả dụng (như `rabbit_a`). Màu xanh dương nhạt nghĩa là `rabbit_b` khả dụng nhưng là server dự phòng (một server dự phòng **không** khả dụng sẽ có màu đỏ như mọi backend server khác).

Để thử warren vừa "ra lò", hãy khởi động consumer từ phần cluster (chương trước) bằng lệnh `python cluster_test_consumer.py localhost 5680`:

```bash
$ python cluster_test_consumer.py localhost 5680
Ready for testing!
```

Bạn đã kết nối và sẵn sàng hành động. Hãy thử bằng cách chạy `python cluster_test_producer localhost 5680`:

```text
 [producer output]
Sent cluster test message.

 [consumer output]
Ready for testing!
Received: Cluster Test!/1301612486
```

Có vẻ mọi thứ chạy trơn tru như đồng hồ. Đã đến lúc thử failover. Bạn sẽ làm hỏng `rabbit_a` bằng cách dừng ứng dụng Erlang của nó. Từ thư mục cài đặt RabbitMQ, chạy `./sbin/rabbitmqctl -n rabbit_a stop_app`. Hãy nhớ rằng bạn đã bảo HAProxy chỉ kiểm tra sức khỏe các backend server **mỗi 5 giây** (`inter 5000`) và yêu cầu một backend server phải thất bại **ba lần** (`fall 3`) trước khi bị coi là không khả dụng. Vì vậy bạn sẽ cần chờ **15 giây** trước khi các lần kết nối lại của consumer thành công. Khi HAProxy đã chuyển sang RabbitMQ dự phòng, bạn sẽ thấy như sau:

```text
 $ python cluster_test_consumer.py localhost 5680
 Ready for testing!
 Received: Cluster Test!/1301612486
 Traceback (most recent call last):
   File "cluster_test_consumer.py", line 58, in <module>
     channel.start_consuming()
   File "/Library/Python/2.6/site-packages/pika-0.9.6-py2.6.egg/
   pika/adapters/blocking_connection.py", line 293, in start_consuming
     self.transport.connection.process_data_events()
   File "/Library/Python/2.6/site-packages/pika-0.9.6-py2.6.egg/
   pika/adapters/blocking_connection.py", line 87, in process_data_events
     raise AMQPConnectionError
 AMQPConnectionError
 ...
 Ready for testing!
```

> 📝 **Giải thích thêm (không có trong sách):** Đọc output trên: consumer đang nghe thì kết nối tới `rabbit_a` bị cắt → pika ném `AMQPConnectionError` (traceback được in ra). Vì `cluster_test_consumer.py` ở chương trước được viết với vòng lặp tự kết nối lại, nó thử kết nối lại vào `localhost:5680`. Các lần đầu HAProxy vẫn cố đẩy sang `rabbit_a` (chưa đủ 3 lần health check thất bại) nên thất bại (dấu `...`). Sau khoảng 15 giây, HAProxy đánh dấu `rabbit_a` là chết và chuyển kết nối sang `rabbit_b` → consumer in lại `Ready for testing!` vì nó đã khai báo lại exchange/queue/binding trên `rabbit_b`.

![Hình 7.3 – Trang thống kê HAProxy cho warren](images/07-warrens-shovels/p150-1.png)

**Hình 7.3 – Trang thống kê HAProxy cho warren của bạn (HAProxy statistics page for your warren)**

*Mô tả hình:* Ảnh chụp màn hình trang thống kê HAProxy tại `http://localhost:8101/stats`. Trang có bảng cho frontend/backend `rabbitmq_local_cluster` với hai dòng server `rabbit_a` và `rabbit_b`, cùng các cột như Queue, Session rate, **Sessions** (Cur – hiện tại, Max, Limit, Total...), Bytes, Errors, Server (Status, Check...). `rabbit_a` hiển thị màu xanh lá nhạt (server hoạt động), còn `rabbit_b` hiển thị màu xanh dương nhạt (server dự phòng, đang khả dụng). Bên dưới là bảng cho `private_monitoring`.

Nếu bạn xem trang thống kê HAProxy (http://localhost:8101/stats), bạn sẽ thấy số **1** dưới tiêu đề **Cur** trong phần **Sessions** (của `rabbit_b`). Điều đó xác nhận rằng consumer của bạn đang kết nối tới server dự phòng. Giờ, nếu bạn publish lại bằng `python cluster_test_producer localhost 5680`, liệu tin nhắn có còn tới được consumer?

```text
  [producer output]
 Sent cluster test message.

  [consumer output]
 ...
 Ready for testing!
 Received: Cluster Test!/1301612545
```

Có đấy! Bạn đã có một warren RabbitMQ active/standby hoạt động đầy đủ, với cơ chế failover dựa trên bộ cân bằng tải. Phần tuyệt nhất là bạn có thể yên tâm rằng khi failover xảy ra, bạn không phải lo RabbitMQ không khởi động được trên node standby, **vì nó vốn đang chạy sẵn rồi!** Vì Rabbit luôn chạy trên cả node active lẫn standby, bạn có thể giám sát cả hai mọi lúc và biết ngay lập tức nếu node standby trở nên không khả dụng — trước cả khi nó được gọi tới. Đó là một điều nữa mà bạn không làm được với warren dùng bộ lưu trữ chung.

> 💡 **Ví dụ bổ sung (không có trong sách):** Bài toán tính thời gian failover. Giả sử bạn đổi cấu hình thành `check inter 2000 rise 2 fall 4`. Hỏi: sau khi `rabbit_a` chết, tối đa bao lâu HAProxy mới chuyển sang `rabbit_b`? Và khi `rabbit_a` sống lại, bao lâu thì nó nhận lại kết nối mới?
>
> **Lời giải:** Cần 4 lần kiểm tra thất bại liên tiếp, mỗi lần cách nhau 2 giây → khoảng 4 × 2 = **8 giây** (cộng thêm thời gian timeout của lần kiểm tra cuối). Khi `rabbit_a` sống lại, cần 2 lần thành công liên tiếp → khoảng 2 × 2 = **4 giây** thì `rabbit_a` được đánh dấu "UP" và vì `rabbit_b` là `backup`, các kết nối **mới** sẽ quay về `rabbit_a`. Lưu ý: các kết nối TCP **đang mở** tới `rabbit_b` không tự động bị chuyển về; consumer vẫn ở lại `rabbit_b` cho tới khi kết nối bị ngắt. Đó là lý do khi khôi phục node active, bạn cần cho consumer kết nối lại để "rút cạn" các tin nhắn cũ còn nằm trên `rabbit_a`.

> 💡 **Ví dụ bổ sung (không có trong sách) – Tự động "rút cạn" (drain) tin nhắn giữa 2 cụm Active/Standby hiện đại bằng Queue Federation:**
>
> Trong kiến trúc Warren dùng HAProxy như sách trình bày, nhược điểm lớn nhất là khi `rabbit_a` sống lại và HAProxy chuyển kết nối về `rabbit_a`, các tin nhắn còn sót lại trên `rabbit_b` (hoặc ngược lại) sẽ nằm im nếu không có consumer kết nối trực tiếp vào đó.
>
> Trong RabbitMQ hiện đại (3.8–4.0), nếu bạn triển khai 2 cụm độc lập (Active/Standby), bạn không cần bắt ứng dụng tự kết nối vào từng cụm để "rút cạn". Thay vào đó, bạn bật plugin `rabbitmq_federation` và cấu hình **Queue Federation** hai chiều (hoặc từ Standby về Active). Khi queue trên cụm Standby không còn consumer cục bộ nào nhưng queue cùng tên trên cụm Active lại có consumer, Queue Federation sẽ **tự động chuyển tiếp (drain)** toàn bộ tin nhắn tồn đọng từ Standby sang Active!
>
> Ngoài ra, các thư viện client hiện đại như `pika 1.x` (Python 3), `amqp-client 5.x` (Java), và `RabbitMQ.Client 6.x/7.x` (.NET) đều hỗ trợ truyền **danh sách nhiều endpoint** để client tự động failover từ node chính sang node dự phòng ngay cả khi không có HAProxy đứng trước:
>
> ```python
> # Python 3 + pika 1.x: Truyền danh sách endpoint (Primary trước, Backup sau)
> import pika
>
> endpoints = [
>     pika.ConnectionParameters(host="127.0.0.1", port=5675),  # Primary (rabbit_a)
>     pika.ConnectionParameters(host="127.0.0.1", port=5676),  # Backup (rabbit_b)
> ]
>
> # Pika sẽ thử kết nối vào 5675 trước; nếu 5675 sập mới chuyển sang 5676
> connection = pika.BlockingConnection(endpoints)
> ```

Với clustering và warren trong tay, bạn đã được "bảo hiểm" khi nói tới xử lý sự cố và mở rộng **bên trong một data center**. Nhưng bạn sẽ làm gì khi cần sao chép tin nhắn giữa các Rabbit ở **những data center khác nhau**? Đó là lúc bạn cần một cái **Shovel** (cái xẻng).

---

## 7.3 Giao tiếp và sao chép đường dài (Long-distance communication and replication)

Clustering của RabbitMQ rất tuyệt để mở rộng hiệu năng nhắn tin bên trong một data center, nhưng nó "gãy" ở chỗ bạn cần định tuyến tin nhắn từ một máy chủ Rabbit ở thành phố này tới một máy chủ Rabbit ở thành phố khác. Bạn có thể thử dùng clustering để bắc cầu giữa các data center ở những vị trí địa lý khác nhau (*geo-diverse*), nhưng bạn sẽ đụng phải vài trở ngại "chặn đứng cuộc chơi" (*show stoppers*):

- **Đầu tiên và quan trọng nhất**, bạn **không kiểm soát được** việc RabbitMQ chọn đặt queue của bạn lên node nào trong cluster. Vì vậy, ngay cả khi bạn có hai node cluster ở Chicago và node thứ ba ở Los Angeles, bạn cũng không có cách nào đảm bảo queue A nằm ở thành phố này còn queue C nằm ở thành phố kia.
- **Thứ hai**, framework giao tiếp OTP của Erlang **không chịu được độ trễ tốt**. Vì thế những đường truyền WAN (*wide area network* – mạng diện rộng) đắt đỏ giữa Chicago và LA sẽ gây hỗn loạn và đủ loại hành vi kỳ lạ bên trong cluster của bạn.
- **Ngoài ra**, RabbitMQ **không có chiến lược nào** để đối phó với việc phân mảnh mạng (*network partitioning* – khi mạng bị chia cắt khiến các node không thấy nhau) nếu đường WAN đó bị đứt.

Ngay lúc này có lẽ bạn đang tự hỏi: "Vậy làm sao xử lý các hạ tầng phân tán địa lý với RabbitMQ?" Câu trả lời là dùng **Shovel**. Nhưng trước khi dùng được nó, bạn cần biết một chút nền tảng về cách Shovel hoạt động.

> 📝 **Giải thích thêm (không có trong sách):** Nhận định "RabbitMQ không có chiến lược đối phó với phân mảnh mạng" đúng ở thời điểm sách được viết (RabbitMQ 2.x). Các phiên bản RabbitMQ sau này đã bổ sung các chế độ xử lý partition (`cluster_partition_handling` như `pause_minority`, `autoheal`), nhưng khuyến nghị chính thức vẫn là **không** trải một cluster qua WAN; thay vào đó dùng Shovel hoặc Federation.

### 7.3.1 "Xúc" thỏ: giới thiệu plugin Shovel (Shoveling your Rabbits: an introduction to the Shovel plugin)

**Shovel** là một plugin cho RabbitMQ cho phép bạn định nghĩa **các quan hệ sao chép** (*replication relationships*) giữa **một queue trên một máy chủ RabbitMQ** và **một exchange trên một máy chủ khác**. Ban đầu được thiết kế bởi LShift (một trong những "cha mẹ" ban đầu của Rabbit Technologies), Shovel giờ đây được đội phát triển lõi của RabbitMQ bảo trì.

Giống như hầu hết các plugin RabbitMQ, Shovel là một ứng dụng Erlang riêng, chỉ "tình cờ" được Rabbit nạp khi khởi động. **Không giống** hầu hết các plugin, Shovel không tích hợp sâu với lõi Rabbit — khi bạn định nghĩa một quan hệ sao chép giữa hai máy chủ trong Shovel, bạn chỉ định **URL đầy đủ của cả hai máy chủ**, bao gồm cả tên người dùng và mật khẩu (ví dụ `amqp://guest:guest@localhost:5675/`). Ở một khía cạnh nào đó, Shovel lẽ ra có thể được viết thành một ứng dụng Erlang độc lập thay vì một plugin RabbitMQ. Nhưng nhờ là một plugin RabbitMQ, bạn có thể tin cậy rằng RabbitMQ sẽ **tự động khởi động Shovel** cùng các quan hệ sao chép đã định nghĩa mỗi khi bạn bật Rabbit.

> 📝 **Giải thích thêm (không có trong sách):** Về bản chất, Shovel hoạt động như một cặp **consumer + producer** được đóng gói sẵn: nó là một **AMQP client** kết nối tới broker nguồn, consume tin nhắn từ một queue, rồi kết nối tới broker đích và publish lại tin nhắn đó vào một exchange. Vì nó nói chuyện bằng giao thức AMQP thông thường (chứ không phải giao thức nội bộ Erlang giữa các node cluster), nó chịu được độ trễ và mất kết nối WAN: khi mạng đứt, tin nhắn chỉ đơn giản là tích tụ trong queue nguồn chờ Shovel kết nối lại.

Có lẽ một ví dụ thực tế sẽ cho thấy Shovel có thể giúp ích thế nào. **Bác nông dân Jacques** điều hành **Avocados Supreme Limited**, một công ty trồng bơ lớn ở Nam California. Jacques gặp vấn đề: trong nhiều năm, Avocados Supreme hoạt động từ một nhà kho duy nhất ở **Goleta**, California, nhưng gần đây nhà kho đã chạy ở mức **80%** công suất và đôi khi các đơn hàng bị chậm trễ khi hết hàng tồn kho. Để khắc phục, Jacques mở thêm một nhà kho thứ hai ở cuối đường, tại **Carpinteria** (xem hình 7.4). Mục đích duy nhất của nhà kho Carpinteria là chứa thêm hàng tồn kho và giao các đơn hàng khi Goleta hết hàng.

Một trong những "viên ngọc quý" trong hoạt động của Avocados Supreme là **hệ thống xử lý đơn hàng** dùng RabbitMQ để nối website của họ với bộ phận hoàn tất đơn hàng (*fulfillment*) ở nhà kho Goleta.

![Hình 7.4 – Bản đồ topology các nhà kho](images/07-warrens-shovels/p152-1.png)

![Mảnh ảnh trích từ Hình 7.4 – vòng tròn vùng Goleta](images/07-warrens-shovels/p152-2.png)
![Mảnh ảnh trích từ Hình 7.4 – vòng tròn website/RabbitMQ #1](images/07-warrens-shovels/p152-3.png)
![Mảnh ảnh trích từ Hình 7.4 – vòng tròn vùng Carpinteria](images/07-warrens-shovels/p152-6.png)
![Mảnh ảnh trích từ Hình 7.4 – vòng tròn RabbitMQ #2](images/07-warrens-shovels/p152-7.png)

*(Ảnh lớn đầu tiên là nền bản đồ; các ảnh nhỏ là các mảnh đồ họa trích tự động từ Hình 7.4. Hình hoàn chỉnh xem ở ảnh trang render bên dưới.)*

**Hình 7.4 – Bản đồ topology các nhà kho (Warehouse map topology)**

![Hình 7.4 – trang sách render (trang 152)](images/pages/page-152.png)

*Mô tả hình:* Một bản đồ vùng bờ biển Nam California. Ở **Goleta** (chấm vàng) có một vòng tròn lớn màu vàng chứa **Website** (*Web site*) và **Warehouse RabbitMQ #1** (RabbitMQ của nhà kho số 1), có mũi tên hai chiều giữa website và RabbitMQ #1. Ở **Carpinteria** (chấm đỏ, gần Goleta) có một vòng tròn lớn màu đỏ chứa **Warehouse RabbitMQ #2** (RabbitMQ của nhà kho số 2). Hai vòng tròn được nối với địa điểm trên bản đồ bằng các đường chấm.

Giờ đây, khi nhà kho Carpinteria đã đi vào hoạt động, **Pierre** (kiến trúc sư công nghệ của Avocados Supreme) đang "rối như canh hẹ" khi tìm cách nối nhà kho mới vào để nhận những đơn hàng mà Goleta không đáp ứng được. Vì không thể dùng clustering của RabbitMQ để bắc cầu giữa hai nhà kho, Pierre đã định cập nhật website để bắt đầu publish tới cả hai máy chủ RabbitMQ ở Goleta và Carpinteria. Nhưng Pierre lo rằng điều này sẽ làm chậm việc đặt hàng trên website, vì web app giờ phải publish tới cả Goleta (nơi đặt website) lẫn Carpinteria (một chuyến đi-về dài hơn nhiều) trước khi có thể xác nhận đơn hàng cho khách (xem hình 7.5). Chưa kể Pierre giờ phải sửa web app để làm việc này, trong khi một trong những lý do anh chọn messaging chính là khả năng định tuyến tin nhắn **mà không phải thay đổi mã frontend**. Trong lúc Pierre đang than thở về các lựa chọn của mình, anh phát hiện ra Shovel.

![Mảnh ảnh trích từ Hình 7.5 – biểu tượng máy chủ website](images/07-warrens-shovels/p153-11.png)
![Mảnh ảnh trích từ Hình 7.5 – biểu tượng khách hàng](images/07-warrens-shovels/p153-15.png)

**Hình 7.5 – Xử lý đơn hàng khi không có Shovel (Order processing without Shovel)**

![Hình 7.5 – trang sách render (trang 153)](images/pages/page-153.png)

```mermaid
flowchart TD
    K["Khách hàng"] -->|"Đơn bơ (Avocado order)"| W["Website"]
    W -->|"publish (LAN, nhanh)"| G["RabbitMQ – nhà kho Goleta"]
    W -->|"publish (WAN, chậm)"| CP["RabbitMQ – nhà kho Carpinteria"]
```

*Mô tả hình:* Một khách hàng gửi **đơn bơ (Avocado order)** (tờ giấy màu cam) tới **Website**. Website phải tự gửi đơn hàng tới **hai** nơi: **RabbitMQ nhà kho Goleta** (mũi tên thẳng xuống, gần) và **RabbitMQ nhà kho Carpinteria** (mũi tên chéo, xa). Website phải chờ cả hai lần publish xong mới xác nhận được đơn.

Sau khi thử nghiệm một chút, Pierre phát hiện mình có thể dùng Shovel để tạo một **queue mới ở Goleta** đăng ký nhận tin (*subscribe* – tức được bind) từ exchange `incoming_orders` mà website publish vào. Sau đó anh có thể bảo Shovel consume các tin nhắn này và **publish lại** chúng qua đường WAN tới exchange `incoming_orders` trên RabbitMQ ở Carpinteria, nơi chúng sẽ được định tuyến tới bộ phận hoàn tất đơn hàng dự phòng (xem hình 7.6). Phần tuyệt nhất là website **không phải làm chậm việc xác nhận đơn hàng chút nào**. Nó có thể tiếp tục publish vào RabbitMQ ở Goleta nằm trên cùng mạng LAN và trả về xác nhận đơn hàng "nhanh như chớp" cho những khách hàng mê bơ. Sau đó Shovel có thể sao chép **bất đồng bộ** (*asynchronously*) các đơn hàng đó sang Carpinteria mà website không hề biết hay bị ảnh hưởng bởi độ trễ tăng thêm. Đây là kịch bản hoàn hảo cho Pierre, người được thưởng một kỳ nghỉ ở bãi biển Monterey vì sự xuất sắc của mình.

Vậy dù bạn cần sao chép tin nhắn giữa các máy chủ RabbitMQ cách nhau cả một đất nước hay chỉ cách nhau một con phố, **Shovel là giải pháp "ruột"** của bạn. Vậy làm sao cài đặt Shovel và bắt đầu phép màu sao chép tin nhắn?

![Mảnh ảnh trích từ Hình 7.6 – biểu tượng máy chủ website](images/07-warrens-shovels/p154-11.png)
![Mảnh ảnh trích từ Hình 7.6 – biểu tượng khách hàng](images/07-warrens-shovels/p154-13.png)

**Hình 7.6 – Xử lý đơn hàng có Shovel (Order processing with Shovel)**

![Hình 7.6 – trang sách render (trang 154)](images/pages/page-154.png)

```mermaid
flowchart TD
    K["Khách hàng"] -->|"Đơn bơ (Avocado order)"| W["Website"]
    W -->|"publish – xong (done)"| G["RabbitMQ – nhà kho Goleta"]
    G -.->|"Shovel (sao chép lúc rảnh – replicated at leisure)"| CP["RabbitMQ – nhà kho Carpinteria"]
```

*Mô tả hình:* Khách hàng gửi **đơn bơ** tới **Website**. Website chỉ publish vào **RabbitMQ nhà kho Goleta** và coi như **xong (done)**. Một mũi tên đứt nét có nhãn **Shovel (replicated at leisure – sao chép "thong thả", không vội)** mang đơn hàng từ RabbitMQ Goleta sang **RabbitMQ nhà kho Carpinteria**.

> 💡 **Ví dụ bổ sung (không có trong sách):** Bài toán độ trễ. Giả sử một lần publish tới RabbitMQ trên cùng LAN mất 1 ms, còn một lần publish qua WAN tới Carpinteria (có chờ xác nhận) mất 40 ms. Website nhận 50 đơn/giây vào giờ cao điểm.
> - **Không có Shovel (hình 7.5):** mỗi đơn mất ≈ 1 + 40 = 41 ms trước khi xác nhận cho khách (nếu publish tuần tự). Nếu đường WAN chập chờn, mỗi đơn có thể treo vài giây, thậm chí lỗi và khách không đặt được hàng.
> - **Có Shovel (hình 7.6):** mỗi đơn chỉ mất ≈ 1 ms. Shovel chịu phần 40 ms ở "hậu trường". Nếu WAN đứt 10 phút, 50 × 600 = 30.000 đơn sẽ nằm chờ an toàn trong queue `backup_orders` ở Goleta, và Shovel sẽ đẩy dần sang Carpinteria khi kết nối lại — khách hàng không hề hay biết.

### 7.3.2 Cài đặt Shovel (Installing Shovel)

Cài đặt Shovel rất đơn giản. Kể từ RabbitMQ 2.7.0, các plugin `rabbitmq-shovel` và `amqp_client` đã được đóng gói sẵn cùng Rabbit; bạn chỉ cần bật chúng lên (hãy đảm bảo thư mục `/etc/rabbitmq/` đã tồn tại trước khi bật plugin). Từ thư mục làm việc RabbitMQ của bạn, chạy:

```bash
$ ./sbin/rabbitmq-plugins enable amqp_client
The following plugins have been enabled:
  amqp_client
Plugin configuration has changed. Restart RabbitMQ for
changes to take effect.
$ ./sbin/rabbitmq-plugins enable rabbitmq_shovel
The following plugins have been enabled:
  erlando
  rabbitmq_shovel
Plugin configuration has changed. Restart RabbitMQ for
changes to take effect.
```

> 📝 **Giải thích thêm (không có trong sách):** Output cho biết: (1) Shovel cần plugin `amqp_client` (thư viện AMQP client viết bằng Erlang) vì như đã nói, Shovel thực chất là một AMQP client; (2) khi bật `rabbitmq_shovel`, plugin phụ thuộc `erlando` cũng được bật kèm tự động; (3) phải **khởi động lại RabbitMQ** thì thay đổi mới có hiệu lực. Danh sách plugin đã bật được lưu trong file `enabled_plugins` dưới `/etc/rabbitmq/` — đó là lý do thư mục này phải tồn tại.

Nếu bạn đang chạy một phiên bản RabbitMQ cũ hơn 2.7.0, bạn sẽ phải tự tải và cài các plugin. Để làm vậy, trước tiên hãy tải các plugin `rabbitmq-shovel` và `amqp_client` từ http://www.rabbitmq.com/plugins.html và đặt chúng vào thư mục `./plugins/` trong bản cài đặt RabbitMQ của bạn:

```bash
 $ cd ./plugins/
 $ wget http://www.rabbitmq.com/.../v2.6.1/amqp_client-2.6.1.ez
 Resolving www.rabbitmq.com... 208.91.1.36
 Connecting to www.rabbitmq.com|208.91.1.36|:80... connected.
 HTTP request sent, awaiting response... 200 OK
 Length: 154928 (151K) [application/andrew-inset]
 Saving to: `amqp_client-2.6.1.ez'
 100%[===============================>] 154,928 120K/s in 1.3s
 $ wget http://www.rabbitmq.com/.../v2.6.1/rabbitmq-shovel-2.6.1.ez
 Resolving www.rabbitmq.com... 208.91.1.36
 Connecting to www.rabbitmq.com|208.91.1.36|:80... connected.
 HTTP request sent, awaiting response... 200 OK
 Length: 42131 (41K) [application/andrew-inset]
 Saving to: `rabbitmq-shovel-2.6.1.ez'
 100%[==================================>] 42,131 112K/s in 0.4s
```

(Ghi chú: phần `...` trong URL là do sách rút gọn đường dẫn đầy đủ. File `.ez` là định dạng gói nén của ứng dụng Erlang.)

Thế là xong. À, chưa hẳn. Bạn vẫn cần cấu hình Shovel và khởi động các broker RabbitMQ của mình.

> 📝 **Giải thích thêm (không có trong sách) – Cách bật plugin Shovel trong RabbitMQ hiện đại (3.8–4.0):**
>
> Trong RabbitMQ hiện đại (3.8–3.13 và RabbitMQ 4.0), quy trình cài đặt và bật Shovel đã được đơn giản hóa đáng kể so với bản 2.6/2.7 trong sách:
> 1. **Không bao giờ phải tải file `.ez` thủ công:** Toàn bộ các plugin chính thức (`rabbitmq_shovel`, `rabbitmq_shovel_management`, `rabbitmq_federation`, v.v.) đã được đóng gói sẵn bên trong mọi bản phân phối RabbitMQ và Docker image chính thức.
> 2. **Tự động bật các thư viện phụ thuộc:** Bạn không cần chạy lệnh `enable amqp_client` riêng lẻ nữa. Khi bật `rabbitmq_shovel`, RabbitMQ sẽ tự động kích hoạt cả `amqp_client` (cho AMQP 0-9-1) và `amqp10_client` / `amqp10_common` (cho AMQP 1.0).
> 3. **Kích hoạt nóng (Online activation) – Không cần khởi động lại RabbitMQ:** Khác với RabbitMQ 2.7 báo *"Restart RabbitMQ for changes to take effect"*, từ RabbitMQ 3.x trở đi, lệnh `rabbitmq-plugins enable` sẽ nạp và chạy plugin **ngay lập tức trên node đang chạy** mà không gây gián đoạn dịch vụ! (Trừ khi bạn cố tình thêm cờ `--offline`).
> 4. **Luôn bật kèm `rabbitmq_shovel_management`:** Nếu bạn có dùng giao diện Web Management UI, hãy bật thêm `rabbitmq_shovel_management` bằng một lệnh duy nhất:
>
> ```bash
> # Bật cả Shovel lẫn giao diện quản lý Shovel trên Web UI (không cần restart broker)
> $ rabbitmq-plugins enable rabbitmq_shovel rabbitmq_shovel_management
> Enabling plugins on node rabbit@node1:
> rabbitmq_shovel
> rabbitmq_shovel_management
> The following plugins have been configured:
>   amqp10_client
>   amqp10_common
>   amqp_client
>   cowboy
>   cowlib
>   rabbitmq_management
>   rabbitmq_management_agent
>   rabbitmq_shovel
>   rabbitmq_shovel_management
>   rabbitmq_web_dispatch
> Applying plugin configuration to rabbit@node1...
> The following plugins have been enabled:
>   amqp10_client
>   amqp10_common
>   rabbitmq_shovel
>   rabbitmq_shovel_management
>
> started 4 plugins.
> ```
>
> Khi đó, trên giao diện Web Management (`http://localhost:15672/`), dưới tab **Admin** sẽ xuất hiện thêm hai mục cực kỳ tiện lợi: **Shovel Status** (giám sát trạng thái kết nối và tốc độ chuyển tin của từng Shovel) và **Shovel Management** (tạo/sửa/xóa Dynamic Shovel trực tiếp trên trình duyệt).

### 7.3.3 Cấu hình và chạy Shovel (Configuring and running Shovel)

Toàn bộ thông tin cấu hình của Shovel, từ các quan hệ sao chép cho tới thiết lập kết nối lại, đều nằm trong file **`rabbitmq.config`**. Giống như phần còn lại của file cấu hình RabbitMQ, cấu hình của Shovel được định dạng như một **tuple Erlang hợp lệ** tên là `rabbitmq_shovel`, với các chỉ thị cấu hình lồng bên trong. Vì cấu hình có thể trông khá phức tạp, hãy xem listing sau — cho thấy `rabbitmq.config` có cấu hình Shovel trông như thế nào — rồi sau đó xem xét từng phần.

**Listing 7.2 – File rabbitmq.config với cấu hình Shovel**

```erlang
[
    {mnesia, [{dump_log_write_threshold, 100}
             ]},
    {rabbit, [{vm_memory_high_watermark, 0.4}
             ]},
    {rabbitmq_shovel,
       [{shovels,
         [{avocado_order_shovel,
           [{sources, [{broker, "amqp://guest:guest@localhost:5675/"},
                        {declarations,
                           [{'queue.declare',
                                   [{queue, <<"backup_orders">>},
                                     durable]},
                                    {'exchange.declare',
                                       [{exchange, <<"incoming_orders">>},
                                        {type, <<"direct">>},
                                        durable]},
                                    {'queue.bind',
                                      [{exchange, <<"incoming_orders">>},
                                       {queue, <<"backup_orders">>},
                                       {routing_key, <<"warehouse">>}]}
                                   ]}]},
              {destinations, [{broker, "amqp://guest:guest@localhost:5676"},
                              {declarations,
                                 [{'queue.declare',
                                   [{queue, <<"warehouse_carpinteria">>},
                                        durable]},
                                  {'exchange.declare',
                                   [{exchange, <<"incoming_orders">>},
                                    {type, <<"direct">>},
                                    durable]},
                                  {'queue.bind',
                                   [{exchange, <<"incoming_orders">>},
                                    {queue, <<"warehouse_carpinteria">>},
                                    {routing_key, <<"warehouse">>}]}
                                 ]}]},
                  {queue, <<"backup_orders">>},
                  {ack_mode, no_ack},
                  {publish_properties, [{delivery_mode, 2}]},
                  {publish_fields, [{exchange, <<"incoming_orders">>},
                                {routing_key, <<"warehouse">>}]},
                  {reconnect_delay, 5}
                 ]}
                ]
              }]
           }
     ].
```

> 📝 **Giải thích thêm (không có trong sách):** Hai mục `{mnesia, ...}` và `{rabbit, ...}` ở đầu file là cấu hình RabbitMQ thông thường (không liên quan tới Shovel): `dump_log_write_threshold` điều chỉnh tần suất Mnesia (cơ sở dữ liệu nội bộ của RabbitMQ) ghi log ra đĩa; `vm_memory_high_watermark, 0.4` nghĩa là RabbitMQ sẽ chặn publisher khi dùng tới 40% RAM. Chúng được đặt ở đây để cho thấy cấu hình Shovel nằm **cùng cấp** với các mục cấu hình khác trong một danh sách Erlang duy nhất, kết thúc bằng dấu chấm `.`.

Ngay dưới chỉ thị `rabbitmq_shovel`, bạn thấy một mục con tên là **`shovels`**. Đây là một danh sách các định nghĩa shovel, trong đó **mỗi shovel định nghĩa một quan hệ sao chép giữa hai máy chủ RabbitMQ**. Trong trường hợp này, bạn chỉ định nghĩa một shovel tên là `avocado_order_shovel`. Bên trong shovel đó, bạn định nghĩa **`sources`** (nguồn) — nơi các tin nhắn cần sao chép đi ra — và **`destinations`** (đích) — nơi các tin nhắn đó sẽ đi tới. Cả `sources` và `destinations` đều chứa cùng các loại chỉ thị cấu hình:

- **`broker` hoặc `brokers`** — URL xác định máy chủ, tên người dùng, mật khẩu và vhost (*virtual host* – "máy chủ ảo", không gian tên riêng bên trong một broker) của máy chủ RabbitMQ mà bạn sẽ "xúc" tin nhắn từ đó (hoặc tới đó). Nếu nguồn hoặc đích là một **cluster** RabbitMQ, hãy dùng `brokers` và theo sau là nhiều hơn một chuỗi URL đặt trong `[]` (ví dụ `["amqp://server1...", "amqp://server2..."]`). Điều này cho phép Shovel chuyển sang một node khác của cluster nếu node chính bị hỏng.
- **`declarations`** — Danh sách các lệnh AMQP để khai báo các queue, exchange và binding cần có sẵn để shovel hoạt động.

`declarations` là phần khó hiểu nhất. Chúng được lồng bên trong một danh sách (mảng), trong đó mỗi phần tử là một **tuple Erlang** xác định lệnh AMQP cần chạy, kèm theo một danh sách tuple khác cung cấp các tham số cho lệnh AMQP đó:

```erlang
{declarations,
   [{'queue.declare',
           [{queue, <<"backup_orders">>},
             durable]},
    {'exchange.declare',
           [{exchange, <<"incoming_orders">>},
            {type, <<"direct">>},
            durable]},
    {'queue.bind',
        [{exchange, <<"incoming_orders">>},
         {queue, <<"backup_orders">>},
         {routing_key, <<"warehouse">>}]}
   ]}]},
```

Ví dụ, phần `declarations` này ra lệnh cho Shovel khai báo một queue tên là `backup_orders` và một exchange tên là `incoming_orders` (hãy nhớ quy tắc của chúng ta: **không giả định rằng bất kỳ phần nào trong "kết cấu" đã có sẵn**). Sau đó bạn bảo shovel bind `backup_orders` vào `incoming_orders` với routing key là `warehouse`.

Bạn có thể nhận thấy vài điểm "kỳ quặc" trong các chỉ thị đó:

- **Thứ nhất**, tất cả các chuỗi không chỉ được đặt trong dấu nháy kép, mà còn có **cặp dấu ngoặc nhọn kép** bao quanh: `<<"backup_orders">>`. Dấu ngoặc nhọn bảo Erlang đừng coi thông tin này là một chuỗi (*string*) mà là một kiểu dữ liệu đặc biệt gọi là **binary**. Bạn không cần hiểu binary là gì, chỉ cần biết rằng **Shovel sẽ crash ngay khi khởi động nếu bạn quên dấu ngoặc nhọn**.
- **Điểm kỳ quặc thứ hai** là tham số `durable` **không** được bọc trong dấu ngoặc nhọn `{}` như mọi tham số khác. Đó là vì `durable` không nhận giá trị. Nó hoặc có mặt hoặc không. Bạn chỉ cần dấu ngoặc nhọn `{}` khi tham số đó nhận một giá trị.

Ngoài những điểm đó ra, các lệnh AMQP và tham số của chúng hẳn trông quen thuộc với bạn, và bạn có thể chỉ định bất kỳ tham số nào bạn vẫn thường dùng trong ngôn ngữ lập trình bạn chọn. Ví dụ, dù bình thường bạn sẽ không muốn làm vậy, bạn hoàn toàn có thể định nghĩa queue `backup_orders` là `auto_delete`. Giống như `durable`, vì `auto_delete` không nhận giá trị, bạn chỉ cần thêm nó (ngăn cách bằng dấu phẩy) không có dấu ngoặc nhọn, ngay sau `durable` trong danh sách tham số của `queue.declare`.

> 💡 **Ví dụ bổ sung (không có trong sách):** Đối chiếu phần `declarations` với code Python (pika) tương đương mà bạn đã quen từ các chương trước:
>
> ```python
> # Tương đương với khối declarations phía nguồn trong Listing 7.2
> channel.queue_declare(queue="backup_orders", durable=True)
> channel.exchange_declare(exchange="incoming_orders",
>                          exchange_type="direct",  # pika 0.9.x dùng tham số type="direct"
>                          durable=True)
> channel.queue_bind(queue="backup_orders",
>                    exchange="incoming_orders",
>                    routing_key="warehouse")
> ```
>
> Và nếu muốn thêm `auto_delete` như sách nói, dòng `queue.declare` trong config sẽ thành:
>
> ```erlang
> {'queue.declare', [{queue, <<"backup_orders">>}, durable, auto_delete]}
> ```
>
> Một lỗi hay gặp: viết `{queue, "backup_orders"}` (thiếu `<< >>`) → Shovel crash khi khởi động. Hoặc viết `{durable}` hay `{durable, true}` thay vì `durable` trần → cấu hình sai định dạng.

Sau khi đã định nghĩa sources và destinations, bạn cũng cần định nghĩa một số **thiết lập chung** cho shovel. Tất cả các thiết lập này đều nhận giá trị, nên chúng được bọc trong dấu ngoặc nhọn `{}`. Các thiết lập bạn có thể định nghĩa là:

- **`queue`** — Tên của queue trên máy chủ nguồn mà Shovel sẽ lắng nghe để lấy tin nhắn cần sao chép.
- **`ack_mode`** — Shovel có nên xác nhận (*acknowledge*) việc đã nhận tin nhắn ở phía nguồn **trước khi** hoàn tất việc giao tới đích hay không.
- **`prefetch_count`** — Số lượng tin nhắn mà Shovel sẽ đệm (*buffer*) nội bộ tại một thời điểm. Bộ đệm nội bộ này là một "trạm dừng chân" giữa nguồn và đích, và **không được bảo vệ** khi Shovel gặp sự cố.
- **`publish_properties`** — Danh sách các thuộc tính (*properties*) cần đặt riêng khi publish tin nhắn tới đích. Ví dụ `{delivery_mode, 2}` sẽ đặt `delivery_mode` thành bền vững (*durable*, giá trị 2). Mặc định, Shovel sao chép các thuộc tính của tin nhắn nguồn khi publish tới đích, trừ khi một thuộc tính được ghi đè cụ thể trong `publish_properties`.
- **`publish_fields`** — Tương tự `publish_properties`, nhưng định nghĩa **exchange** để publish tin nhắn vào trên máy chủ đích và **`routing_key`** để gắn cho tin nhắn. Nếu `exchange` hoặc `routing_key` không được định nghĩa, Shovel sao chép các thiết lập bị bỏ trống đó từ tin nhắn gốc.
- **`reconnect_delay`** — Số giây cần chờ trước khi kết nối lại tới nguồn hoặc đích sau khi bị ngắt kết nối.

Vậy trong trường hợp của bạn, bạn muốn bảo Shovel consume tin nhắn từ queue `backup_orders` trên máy chủ nguồn và giao chúng tới exchange `incoming_orders` trên máy chủ đích với routing key là `warehouse`. Bạn không muốn tự động xác nhận tin nhắn (ví dụ, đặt `ack_mode` là `on_confirm` hoặc `on_publish`), nhưng bạn muốn tin nhắn được publish dưới dạng bền vững (delivery mode 2). Cuối cùng, bạn muốn Shovel chờ 5 giây trước khi kết nối lại nếu bị ngắt kết nối. Ghép tất cả lại, đây là cách nó trông trong file cấu hình:

```erlang
 {queue, <<"backup_orders">>},
 {ack_mode, no_ack},
 {publish_properties, [{delivery_mode, 2}]},
 {publish_fields, [{exchange, <<"incoming_orders">>},
                   {routing_key, <<"warehouse">>}]},
 {reconnect_delay, 5}
```

> 📝 **Giải thích thêm (không có trong sách):** Có một chỗ **không nhất quán** trong sách: văn bản nói "không muốn tự động xác nhận (ví dụ đặt `ack_mode` là `on_confirm` hoặc `on_publish`)", nhưng cấu hình lại ghi `{ack_mode, no_ack}`. Theo tài liệu Shovel, ba giá trị của `ack_mode` là:
> - `no_ack` — Shovel consume ở chế độ tự động xác nhận: broker nguồn coi tin nhắn là đã giao ngay khi gửi cho Shovel. **Nhanh nhất nhưng kém an toàn nhất**: nếu Shovel chết hoặc đích không nhận được, tin nhắn có thể mất (đặc biệt là những tin đang nằm trong bộ đệm nội bộ).
> - `on_publish` — Shovel chỉ ack ở nguồn **sau khi đã publish** tin nhắn tới đích.
> - `on_confirm` — Shovel chỉ ack ở nguồn **sau khi broker đích xác nhận** (*publisher confirm*) là đã nhận tin nhắn. **An toàn nhất**.
>
> Như vậy, nếu đúng ý "không muốn tự động ack", bạn nên dùng `{ack_mode, on_confirm}` (hoặc `on_publish`). Chúng tôi giữ nguyên `no_ack` trong code để trung thành với sách, nhưng với dữ liệu quan trọng như đơn hàng, `on_confirm` là lựa chọn hợp lý hơn.

> 📝 **Giải thích thêm (không có trong sách) – Static Shovel (trong sách) vs Dynamic Shovel trong RabbitMQ hiện đại (3.8–4.0):**
>
> Cách cấu hình Shovel trong Listing 7.2 bằng các tuple Erlang phức tạp (`<<"binary">>`, `'queue.declare'`, v.v.) bên trong file cấu hình hệ thống được gọi là **Static Shovel (Shovel tĩnh)**.
> - Ở thời điểm năm 2012 (RabbitMQ 2.7), **Static Shovel là cách duy nhất** tồn tại. Nhược điểm rất lớn của nó là: cú pháp Erlang tuple cực kỳ dễ gõ sai (quên một dấu `<< >>` hoặc dấu chấm `.` ở cuối là node từ chối khởi động), và **mỗi lần muốn thêm, sửa hoặc xóa một Shovel, bạn bắt buộc phải khởi động lại node RabbitMQ**.
> - Từ **RabbitMQ 3.3.0+ đến 3.13 và RabbitMQ 4.0**, RabbitMQ bổ sung **Dynamic Shovel (Shovel động)** — và đây hiện là **phương pháp tiêu chuẩn được dùng trong hơn 95% hệ thống thực tế**.
>
> **Bảng so sánh: Static Shovel vs Dynamic Shovel (RabbitMQ 3.8–4.0):**
>
> | Tiêu chí | Static Shovel (Listing 7.2 trong sách) | Dynamic Shovel (Chuẩn hiện đại 3.8–4.0) |
> |---|---|---|
> | **Nơi khai báo** | File `advanced.config` (hoặc `rabbitmq.config` cũ) dưới dạng Erlang tuple | **Runtime Parameters** của cụm (lưu trong database Mnesia/Khepri) |
> | **Cần khởi động lại node?** | **Bắt buộc phải restart** RabbitMQ mỗi khi thay đổi | **Không cần restart!** Có hiệu lực ngay lập tức (hot-reload) |
> | **Công cụ quản lý** | Phải SSH vào từng server sửa file cấu hình | Qua CLI (`rabbitmqctl`), HTTP REST API (`PUT /api/parameters/shovel/...`), hoặc giao diện **Web Management UI** (*Admin -> Shovel Management*) |
> | **Tính sẵn sàng trong Cluster** | Chạy cố định trên node có file cấu hình đó | **Tự động chạy trên một node trong cluster**; nếu node đó sập, một node khác trong cluster tự động tiếp quản Shovel! |
> | **Khai báo Topology (`declarations`)** | Phải tự liệt kê dài dòng từng `queue.declare`, `exchange.declare`, `queue.bind` bằng Erlang | Tự động tạo queue/exchange cơ bản dựa trên `src-queue` / `dest-queue` / `dest-exchange` (hoặc kết hợp với Definitions/Policies) |
> | **Giao thức hỗ trợ** | Chỉ AMQP 0-9-1 (ở bản 2.7) | Hỗ trợ cả **AMQP 0-9-1** lẫn **AMQP 1.0** (`"amqp091"` và `"amqp10"`), nối được với **Azure Service Bus, ActiveMQ Artemis, Amazon MQ, IBM MQ** |
> | **`ack-mode` mặc định** | Phải tự chỉ định (sách dùng `no_ack` kém an toàn) | Mặc định là **`on-confirm`** (an toàn nhất, dùng Publisher Confirms) |
>
> *Lưu ý thêm về Static Shovel hiện đại:* Từ RabbitMQ 3.7.0+, file `rabbitmq.config` đã được thay bằng cặp file `rabbitmq.conf` (định dạng `key = value` kiểu sysctl) và `advanced.config` (định dạng Erlang tuple). Nếu bạn vẫn muốn dùng Static Shovel ngày nay, đoạn cấu hình `{rabbitmq_shovel, ...}` phải đặt trong file **`/etc/rabbitmq/advanced.config`**.

> 💡 **Ví dụ bổ sung (không có trong sách) – Tạo `avocado_order_shovel` bằng Dynamic Shovel hiện đại (không cần restart):**
>
> Thay vì viết 45 dòng Erlang tuple trong `rabbitmq.config` và phải khởi động lại máy chủ như Listing 7.2, trong RabbitMQ hiện đại (3.8–4.0) bạn có thể tạo đúng Shovel đó **ngay lúc hệ thống đang chạy** bằng 1 trong 3 cách sau:
>
> **Cách 1: Dùng lệnh CLI `rabbitmqctl set_parameter`**
> ```bash
> rabbitmqctl set_parameter shovel avocado_order_shovel '{
>   "src-protocol": "amqp091",
>   "src-uri": "amqp://guest:guest@localhost:5675/%2f",
>   "src-queue": "backup_orders",
>   "dest-protocol": "amqp091",
>   "dest-uri": "amqp://guest:guest@localhost:5676/%2f",
>   "dest-exchange": "incoming_orders",
>   "dest-exchange-key": "warehouse",
>   "ack-mode": "on-confirm",
>   "src-prefetch-count": 1000,
>   "reconnect-delay": 5,
>   " dest-add-forward-headers": true
> }'
> ```
>
> **Cách 2: Dùng HTTP REST API (rất tiện cho CI/CD, Ansible, Terraform)**
> ```bash
> # %2f là ký tự "/" (tên vhost mặc định) đã được URL-encode
> curl -u admin:StrongPass123! -X PUT \
>   http://localhost:15672/api/parameters/shovel/%2f/avocado_order_shovel \
>   -H "content-type: application/json" \
>   -d '{
>     "value": {
>       "src-protocol": "amqp091",
>       "src-uri": "amqp://admin:StrongPass123!@10.0.1.10:5672/%2f",
>       "src-queue": "backup_orders",
>       "dest-protocol": "amqp091",
>       "dest-uri": "amqp://admin:StrongPass123!@10.0.2.20:5672/%2f",
>       "dest-exchange": "incoming_orders",
>       "dest-exchange-key": "warehouse",
>       "ack-mode": "on-confirm",
>       "reconnect-delay": 5
>     }
>   }'
> ```
>
> **Cách 3: Dùng giao diện Web Management UI**
> Mở trình duyệt vào `http://localhost:15672` → chọn tab **Admin** → chọn **Shovel Management** ở menu bên phải → bấm **Add a new shovel**, điền tên `avocado_order_shovel`, chọn giao thức `AMQP 0-9-1` (hoặc `AMQP 1.0`), nhập URI/queue nguồn và URI/exchange đích → bấm **Add shovel**.
>
> **Kiểm tra trạng thái Shovel đang chạy và xóa Shovel động:**
> ```bash
> # Xem trạng thái kết nối của mọi Shovel (cả Static lẫn Dynamic)
> $ rabbitmqctl shovel_status
>
> # Xóa Dynamic Shovel ngay lập tức mà không cần restart
> $ rabbitmqctl clear_parameter shovel avocado_order_shovel
> ```
>
> **Ví dụ kết nối RabbitMQ với Azure Service Bus / ActiveMQ Artemis qua giao thức AMQP 1.0:**
> Nhờ hỗ trợ `amqp10`, Dynamic Shovel hiện đại còn là công cụ cầu nối (*bridge*) tuyệt vời giữa các hệ thống message broker khác hãng:
> ```bash
> rabbitmqctl set_parameter shovel rabbit_to_azure_sb '{
>   "src-protocol": "amqp091",
>   "src-uri": "amqp://localhost:5672/%2f",
>   "src-queue": "backup_orders",
>   "dest-protocol": "amqp10",
>   "dest-uri": "amqps://RootManageSharedAccessKey:MySASKey@mybus.servicebus.windows.net:5671/?sasl=plain",
>   "dest-address": "carpinteria-orders-queue",
>   "ack-mode": "on-confirm"
> }'
> ```

Hình 7.7 cho thấy topology mà Shovel của bạn định nghĩa. Bạn đang lấy các đơn hàng từ queue `backup_orders` trên máy chủ Rabbit `localhost:5675` và publish chúng vào exchange `incoming_orders` trên máy chủ Rabbit `localhost:5676`.

![Mảnh ảnh trích từ Hình 7.7 – khung exchange](images/07-warrens-shovels/p158-28.png)
![Mảnh ảnh trích từ Hình 7.7 – khung queue](images/07-warrens-shovels/p158-36.png)
![Mảnh ảnh trích từ Hình 7.7 – nền hình tròn Goleta](images/07-warrens-shovels/p158-38.png)
![Mảnh ảnh trích từ Hình 7.7 – nền hình tròn Carpinteria](images/07-warrens-shovels/p158-39.png)
![Mảnh ảnh trích từ Hình 7.7 – khung queue](images/07-warrens-shovels/p158-41.png)
![Mảnh ảnh trích từ Hình 7.7 – khung exchange](images/07-warrens-shovels/p158-50.png)

*(Các ảnh nhỏ ở trên là các mảnh đồ họa trích tự động từ Hình 7.7. Hình hoàn chỉnh xem ở ảnh trang render bên dưới.)*

**Hình 7.7 – Topology xử lý đơn hàng bằng Shovel (Shovel order processing topology)**

![Hình 7.7 – trang sách render (trang 158)](images/pages/page-158.png)

```mermaid
flowchart LR
    O["Order 67"] --> EX1
    subgraph G["RabbitMQ – nhà kho Goleta (localhost:5675)"]
        EX1["incoming_orders (exchange)"]
        QG["warehouse_goleta (queue)"]
        QB["backup_orders (queue)"]
        EX1 -->|"Order 67"| QG
        EX1 -->|"Order 67"| QB
    end
    subgraph C["RabbitMQ – nhà kho Carpinteria (localhost:5676)"]
        EX2["incoming_orders (exchange)"]
        QC["warehouse_carpinteria (queue)"]
        EX2 -->|"Order 67"| QC
    end
    QB -->|"Shovel – Order 67"| EX2
```

*Mô tả hình:* Một **đơn hàng số 67 (Order 67)** đi vào exchange **`incoming_orders`** trên **RabbitMQ nhà kho Goleta**. Exchange này định tuyến bản sao của đơn hàng tới **hai** queue: **`warehouse_goleta`** (để nhà kho Goleta xử lý như bình thường) và **`backup_orders`** (queue dành cho Shovel). **Shovel** lấy Order 67 từ `backup_orders` và publish sang exchange **`incoming_orders`** trên **RabbitMQ nhà kho Carpinteria**, nơi nó được định tuyến vào queue **`warehouse_carpinteria`**. Kết quả: cả hai nhà kho đều có một bản của Order 67.

Giờ cấu hình Shovel của bạn đã hoàn tất, đã đến lúc khởi động các máy chủ Rabbit nguồn và đích để xem nó hoạt động thế nào. Bạn sẽ dựng hai node RabbitMQ độc lập trên máy phát triển để thử nghiệm. Vì cả máy chủ nguồn lẫn đích sẽ dùng **cùng một file `rabbitmq.config`**, bạn sẽ có **hai shovel giống hệt nhau** đang chạy: một trên nguồn và một trên đích. Bình thường bạn chỉ muốn có một shovel, nhưng vì bạn đang chạy cả hai node trên máy cục bộ, dùng một file cấu hình cho cả hai sẽ đơn giản hơn cho việc thử nghiệm.

Nếu bạn vẫn còn `rabbit_a` và `rabbit_b` đang chạy từ phần warren, hãy dừng chúng ngay, vì bạn sẽ dùng lại chúng cho ví dụ này. Sau khi đã dừng, chuyển vào thư mục cài đặt RabbitMQ và chạy `RABBITMQ_NODE_PORT=5675 RABBITMQ_NODENAME=rabbit_a ./sbin/rabbitmq-server` để khởi động node nguồn (bạn **không** dùng tùy chọn `-detached` để có thể thấy lỗi trong file cấu hình nếu có). Vì terminal đầu tiên đã bị node nguồn chiếm ở chế độ chạy tiền cảnh (*foreground*), hãy mở một terminal mới rồi chạy `RABBITMQ_NODE_PORT=5676 RABBITMQ_NODENAME=rabbit_b ./sbin/rabbitmq-server` để khởi động node đích.

```bash
# Terminal 1 – node nguồn (source)
$ RABBITMQ_NODE_PORT=5675 RABBITMQ_NODENAME=rabbit_a ./sbin/rabbitmq-server

# Terminal 2 – node đích (destination)
$ RABBITMQ_NODE_PORT=5676 RABBITMQ_NODENAME=rabbit_b ./sbin/rabbitmq-server
```

*(Ghi chú: trong bản PDF, hai lệnh này bị dính chữ thành `rabbit_a./sbin/...`; giữa tên node và `./sbin/rabbitmq-server` phải có một dấu cách như trên.)*

> 📝 **Giải thích thêm (không có trong sách):** Vì sao "hai shovel giống hệt nhau" không gây sao chép trùng lặp? Cả hai shovel (một chạy trong `rabbit_a`, một chạy trong `rabbit_b`) đều dùng cùng URL nguồn `localhost:5675` và cùng URL đích `localhost:5676`. Chúng trở thành **hai consumer trên cùng queue `backup_orders`**; RabbitMQ chia tin nhắn kiểu round-robin giữa chúng, nên mỗi đơn hàng vẫn chỉ được chuyển sang đích **một lần**. Trong môi trường thật, bạn thường chỉ đặt cấu hình Shovel trên **một** node (thường là node phía nguồn hoặc phía đích).

Với cả nguồn và đích đều đang chạy, bạn sẽ cần một consumer và một producer để thử thiết lập này. Những biến thể của consumer và producer dùng cho cluster (chương trước), được sửa lại để xử lý đơn hàng bơ, sẽ làm tốt việc này. Trước tiên hãy xem consumer, ở listing sau.

**Listing 7.3 – Consumer thử nghiệm Shovel**

```python
import sys, json, pika, time, traceback

def msg_rcvd(channel, method, header, body):
    message = json.loads(body)
    print "Received order %(ordernum)d for %(type)s." % message      # (1) In và xác nhận
    channel.basic_ack(delivery_tag=method.delivery_tag)              #     đơn hàng

if __name__ == "__main__":
    AMQP_SERVER = sys.argv[1]                                        # (2) Thiết lập broker
    AMQP_PORT = int(sys.argv[2])

    creds_broker = pika.PlainCredentials("guest", "guest")           # (3) Thiết lập kết nối
    conn_params = pika.ConnectionParameters( AMQP_SERVER,            #     tới broker
                                             port=AMQP_PORT,
                                             virtual_host="/",
                                             credentials=creds_broker)

    conn_broker = pika.BlockingConnection(conn_params)               # (4) Thiết lập kết nối
    channel = conn_broker.channel()                                  #     tới RabbitMQ

    print "Ready for orders!"
    channel.basic_consume( msg_rcvd,                                 # (5) Bắt đầu xử lý
                           queue="warehouse_carpinteria",            #     đơn hàng
                           no_ack=False,
                           consumer_tag="order_processor")
    channel.start_consuming()
```

Các chú thích (callout) trong sách:

1. **Print and acknowledge order** – In ra và xác nhận (ack) đơn hàng: hàm callback `msg_rcvd` giải mã JSON, in số đơn và loại bơ, rồi gửi `basic_ack`.
2. **Broker settings** – Thiết lập broker: lấy host và cổng từ tham số dòng lệnh.
3. **Establish broker connection settings** – Thiết lập các tham số kết nối tới broker (thông tin đăng nhập `guest/guest`, vhost `/`).
4. **Establish connection to RabbitMQ** – Mở kết nối tới RabbitMQ và tạo channel.
5. **Start processing orders** – Bắt đầu xử lý đơn hàng: đăng ký consume từ queue `warehouse_carpinteria` (ack thủ công, `no_ack=False`) với consumer tag `order_processor`.

Nó không khác mấy so với các consumer bạn đã viết ở chương 4 để xử lý cảnh báo (*alerts*). Producer của bạn (ở listing sau) còn đơn giản hơn, với thay đổi duy nhất là định dạng của đơn hàng bơ mà bạn gửi đi.

**Listing 7.4 – Producer thử nghiệm Shovel**

```python
import sys, json, pika, random

AMQP_HOST = sys.argv[1]
AMQP_PORT = int(sys.argv[2])
AVOCADO_TYPE = sys.argv[3]

creds_broker = pika.PlainCredentials("guest", "guest")             # (1) Thiết lập kết nối
conn_params = pika.ConnectionParameters(AMQP_HOST,                 #     tới broker
                                        port=AMQP_PORT,
                                        virtual_host = "/",
                                        credentials = creds_broker)

conn_broker = pika.BlockingConnection(conn_params)                 # (2) Kết nối tới RabbitMQ
channel = conn_broker.channel()                                    #     và gửi tin nhắn

msg = json.dumps({"ordernum": random.randrange(0, 100, 1),
                  "type" : AVOCADO_TYPE})
msg_props = pika.BasicProperties(content_type="application/json")

channel.basic_publish(body=msg, mandatory=True,
                      exchange="incoming_orders",
                      properties=msg_props,
                      routing_key="warehouse")

print "Sent avocado order message."
```

Các chú thích (callout) trong sách:

1. **Establish connection to broker** – Thiết lập các tham số kết nối tới broker.
2. **Connect to RabbitMQ and send message** – Kết nối tới RabbitMQ và gửi tin nhắn: tạo đơn hàng JSON với số đơn ngẫu nhiên 0–99 và loại bơ, rồi publish vào exchange `incoming_orders` với routing key `warehouse` (cờ `mandatory=True`).

> 💡 **Ví dụ bổ sung (không có trong sách) – Viết lại Listing 7.3 & Listing 7.4 trên Python 3 + `pika 1.x` hiện đại:**
>
> Trong `pika 1.x` trên Python 3, chữ ký hàm `basic_consume` đã đổi thứ tự tham số (`queue` đứng trước `on_message_callback`) và đổi `no_ack` thành `auto_ack`. Dưới đây là code Consumer và Producer chuẩn hiện đại:
>
> ```python
> # shovel_consumer_modern.py (Python 3 + pika 1.x)
> import json
> import sys
> import pika
>
> def msg_rcvd(channel, method, properties, body):
>     message = json.loads(body.decode("utf-8"))
>     print(f"Received order {message['ordernum']} for {message['type']}.")
>     channel.basic_ack(delivery_tag=method.delivery_tag)
>
> if __name__ == "__main__":
>     amqp_server = sys.argv[1]
>     amqp_port = int(sys.argv[2])
>
>     creds = pika.PlainCredentials("guest", "guest")
>     conn_params = pika.ConnectionParameters(
>         host=amqp_server, port=amqp_port, virtual_host="/", credentials=creds
>     )
>     with pika.BlockingConnection(conn_params) as conn:
>         channel = conn.channel()
>         channel.basic_qos(prefetch_count=50)
>         print("Ready for orders!")
>         channel.basic_consume(
>             queue="warehouse_carpinteria",
>             on_message_callback=msg_rcvd,
>             auto_ack=False,
>             consumer_tag="order_processor",
>         )
>         channel.start_consuming()
> ```
>
> ```python
> # shovel_producer_modern.py (Python 3 + pika 1.x)
> import json
> import random
> import sys
> import pika
>
> amqp_host = sys.argv[1]
> amqp_port = int(sys.argv[2])
> avocado_type = sys.argv[3]
>
> creds = pika.PlainCredentials("guest", "guest")
> conn_params = pika.ConnectionParameters(
>     host=amqp_host, port=amqp_port, virtual_host="/", credentials=creds
> )
>
> with pika.BlockingConnection(conn_params) as conn:
>     channel = conn.channel()
>     channel.confirm_delivery()  # Bật Publisher Confirms hiện đại
>
>     msg = json.dumps({"ordernum": random.randrange(0, 100, 1), "type": avocado_type})
>     msg_props = pika.BasicProperties(
>         content_type="application/json",
>         delivery_mode=pika.DeliveryMode.Persistent,  # Tin nhắn bền vững (2)
>     )
>
>     channel.basic_publish(
>         exchange="incoming_orders",
>         routing_key="warehouse",
>         body=msg.encode("utf-8"),
>         properties=msg_props,
>         mandatory=True,
>     )
>     print("Sent avocado order message.")
> ```

Consumer của bạn nhận hai tham số dòng lệnh: tên host đích và cổng đích. Producer nhận cùng các tham số đó nhưng thêm tham số thứ ba: **loại bơ** (hass, fuerte, v.v. — bất kỳ từ đơn nào cũng được). Bạn sẽ publish một đơn hàng bơ vào **node nguồn** bằng producer, rồi qua "phép màu" của Shovel, nhận và in đơn hàng đó từ consumer gắn trên **máy chủ đích hoàn toàn độc lập**. Hãy khởi động consumer và kết nối nó tới node đích:

```bash
$ python shovel_consumer.py localhost 5676
Ready for orders!
```

Giờ hãy gửi cho nó một đơn hàng bơ bằng cách publish vào node nguồn:

```bash
$ python shovel_producer.py localhost 5675 hass
```

Quay lại phía consumer, bạn sẽ thấy đại loại như sau:

```text
$ python shovel_consumer.py localhost 5676
Ready for orders!
Received order 66 for hass.
```

Tuyệt cú mèo (*Hot diggity*)! Bạn đã có một shovel chạy được, cho phép những gì publish trên một máy chủ RabbitMQ được consume trên một máy chủ hoàn toàn độc lập khác. Dù tốn công thiết lập hơn clustering, Shovel mở rộng tầm với của RabbitMQ bằng cách cho phép bạn tạo những topology còn vững chắc hơn, nay bao gồm cả **liên kết** (*federation* – kết nối, chia sẻ tin nhắn giữa các broker/cluster độc lập) giữa các máy chủ và cluster RabbitMQ độc lập. Trong tương lai RabbitMQ sẽ hỗ trợ federation mạnh mẽ một cách nguyên bản (*natively*), nhưng cho tới lúc đó, Shovel sẽ cung cấp cho bạn khả năng federation mà bạn cần ngay hôm nay.

> 📝 **Giải thích thêm (không có trong sách):** Lời hứa này đã thành hiện thực: các phiên bản RabbitMQ sau này có **plugin Federation** (`rabbitmq_federation`) cho phép federate exchange và queue giữa các broker. Ngoài ra Shovel cũng có thêm kiểu **dynamic shovel** — cấu hình lúc chạy qua `rabbitmqctl set_parameter shovel ...` hoặc giao diện quản lý, thay vì phải sửa `rabbitmq.config` và khởi động lại như cách **static shovel** trong chương này.

> 📝 **Giải thích thêm (không có trong sách) – Khi nào dùng Shovel, khi nào dùng Federation trong RabbitMQ hiện đại (3.8–4.0)?**
>
> Ngày nay, cả **Shovel** (`rabbitmq_shovel`) và **Federation** (`rabbitmq_federation`) đều là plugin chính thức có sẵn trong RabbitMQ để truyền tin nhắn qua mạng WAN / liên trung tâm dữ liệu. Vậy bạn nên chọn công cụ nào?
>
> | Tiêu chí | Plugin Shovel (`rabbitmq_shovel`) | Plugin Federation (`rabbitmq_federation`) |
> |---|---|---|
> | **Tư duy hoạt động** | **"Đẩy / Hút điểm-nối-điểm" (Point-to-Point Move):** Lấy tin nhắn từ một queue nguồn bất kỳ và đẩy sang exchange/queue đích | **"Liên kết định tuyến thông minh" (Smart Routing/Sharing):** Đồng bộ các binding từ Downstream lên Upstream; chỉ gửi tin nhắn qua WAN khi phía Downstream thực sự có Consumer/Binding cần tin nhắn đó |
> | **Nơi đặt cấu hình** | Có thể chạy ở bất kỳ broker nào (nguồn, đích, hoặc một broker thứ ba ở giữa) | Luôn cấu hình ở phía **Downstream** (nơi nhận tin) bằng cách khai báo `federation-upstream` + `Policy` |
> | **Hỗ trợ giao thức** | **AMQP 0-9-1 và AMQP 1.0** (nối được với Azure Service Bus, ActiveMQ Artemis, v.v.) | Chỉ hỗ trợ **AMQP 0-9-1** (giữa các cụm RabbitMQ với nhau) |
> | **Trường hợp sử dụng tốt nhất** | 1. Di chuyển (drain) dữ liệu khi nâng cấp cụm hoặc cứu hộ queue bị đầy.<br>2. Tích hợp với hệ thống AMQP 1.0 bên ngoài.<br>3. Sao chép 1-1 đơn giản, tường minh như bài toán nhà kho Goleta → Carpinteria. | 1. Phát tán sự kiện (Pub/Sub) từ 1 cụm trung tâm ra hàng chục cụm vùng (Multi-Region Exchange Federation).<br>2. Chia tải công việc liên vùng chỉ khi vùng chính quá tải (Queue Federation). |

> 💡 **Ví dụ bổ sung (không có trong sách):** Bài tập tự kiểm tra. Hãy dự đoán điều gì xảy ra trong các tình huống sau với cấu hình Listing 7.2:
> 1. **Bạn dừng `rabbit_b` (đích) rồi chạy producer 3 lần vào `localhost:5675`.** → Shovel mất kết nối tới đích, cứ mỗi 5 giây (`reconnect_delay`) thử kết nối lại. Ba đơn hàng nằm chờ trong `backup_orders` trên `rabbit_a`. Khi bạn bật lại `rabbit_b`, Shovel kết nối lại, khai báo lại `warehouse_carpinteria` (nhờ `declarations`) và đẩy 3 đơn sang; consumer in ra 3 dòng `Received order ...`. *(Lưu ý: với `no_ack` và nhiều shovel, một vài tin nhắn đã nằm trong bộ đệm của Shovel lúc mất kết nối có thể bị mất — đây là lý do nên dùng `on_confirm`.)*
> 2. **Bạn chạy producer với routing key khác, ví dụ `"warehouse_x"`.** → Exchange `incoming_orders` (kiểu direct) ở Goleta không có binding nào khớp với `backup_orders`, nên tin nhắn không vào `backup_orders` và **không** được shovel sang Carpinteria; vì `mandatory=True`, broker sẽ trả lại tin nhắn cho producer.
> 3. **Bạn xóa dòng `{routing_key, <<"warehouse">>}` trong `publish_fields`.** → Shovel giữ nguyên routing key gốc của tin nhắn (`warehouse`), nên kết quả không đổi.

---

## 7.4 Tóm tắt (Summary)

Đó là một chuyến đi dài qua tất cả các lựa chọn để giúp Rabbit chống chịu sự cố, nhưng hãy nhớ lại: khi mới bắt đầu, bạn hoàn toàn phụ thuộc vào "lòng thương xót" của sự cố máy chủ — chỉ cần một máy hỏng là Rabbit của bạn sập và hạ tầng nhắn tin bị ngắt. Giờ thì bạn không còn phải sợ ai đó vấp phải dây điện nữa.

- Nếu thứ bạn thực sự cần là **tính sẵn sàng cao tuyệt đối, không có khả năng mất tin nhắn**, bạn có thể xây một **warren** dùng hai máy chủ RabbitMQ độc lập, với một **bộ cân bằng tải** khiến chúng trông như một máy duy nhất đối với các ứng dụng của bạn.
- Nếu vì lý do nào đó bạn cần mở rộng độ tin cậy này để **bắc cầu giữa các Rabbit ở nhiều data center**, bạn đã biết cách dùng **Shovel** để cung cấp cơ chế sao chép giúp biến điều đó thành hiện thực.
- Có lẽ quan trọng nhất, bạn biết cách làm cho **ứng dụng của mình chống chịu được** khi từng Rabbit riêng lẻ gặp sự cố. Bằng cách khiến chúng **không giả định gì** về trạng thái của các máy chủ mà chúng kết nối tới và cài đặt chức năng **tự kết nối lại**, các consumer và producer của bạn có thể tận dụng bất kỳ tùy chọn dự phòng nào của RabbitMQ mà bạn đã học, và sống sót qua sự cố node mà không "lỡ nhịp" nào.

Ứng dụng và Rabbit của bạn giờ đã vững chắc, chống chịu tốt và sẵn sàng cho production! Nhưng một hạ tầng RabbitMQ vững chắc và phân tán thì có ích gì nếu nó là một "cực hình" khi quản trị? Để trả lời câu hỏi đó, tiếp theo chúng ta sẽ bước vào thế giới tuyệt vời của **plugin quản lý RabbitMQ** (*RabbitMQ management plugin*).

> 💡 **Ví dụ bổ sung (không có trong sách):** Bảng chọn nhanh giải pháp theo tình huống, tổng hợp từ chương 5–7:
>
> | Tình huống | Giải pháp phù hợp |
> |---|---|
> | Cần tăng thông lượng, chia tải trong **một** data center | Cluster RabbitMQ (+ load balancer) |
> | Không chấp nhận "khoảng trống" khi node chứa durable queue sập; cần nơi publish ngay lập tức | Warren dùng load balancer (active/standby, shared-nothing) |
> | Muốn node dự phòng có sẵn **toàn bộ** tin nhắn cũ, chấp nhận độ phức tạp | Warren dùng shared storage + Pacemaker |
> | Chuyển/sao chép tin nhắn giữa các data center, qua WAN | Shovel |

---

## Bảng thuật ngữ của chương

| Thuật ngữ (Anh) | Nghĩa tiếng Việt | Giải thích ngắn |
|---|---|---|
| High availability | Tính sẵn sàng cao | Khả năng hệ thống tiếp tục phục vụ khi một thành phần gặp sự cố |
| Resiliency | Khả năng chống chịu | Mức độ hệ thống chịu được và phục hồi sau sự cố |
| Clustering / cluster | Gom cụm / cụm | Nhiều node RabbitMQ hoạt động như một broker logic duy nhất |
| Node | Nút | Một tiến trình máy chủ RabbitMQ (một máy ảo Erlang) |
| Durable queue | Queue bền vững | Queue được giữ lại sau khi broker khởi động lại |
| Warren | Warren ("hang thỏ") | Cặp máy chủ RabbitMQ độc lập active/standby với load balancer phía trước |
| Active/standby pair | Cặp chủ động/dự phòng | Một máy phục vụ, một máy chờ sẵn để thay thế khi máy kia hỏng |
| Standalone server | Máy chủ độc lập | Node RabbitMQ không thuộc cluster nào |
| Load balancer | Bộ cân bằng tải | Thành phần phân phối kết nối tới các máy chủ phía sau (ví dụ HAProxy) |
| Failover | Chuyển đổi dự phòng | Tự động chuyển sang máy dự phòng khi máy chính hỏng |
| Shared-nothing | Không chia sẻ gì | Kiến trúc trong đó các máy không dùng chung tài nguyên hay phối hợp với nhau |
| Shared storage | Bộ lưu trữ dùng chung | Ổ đĩa/SAN được nhiều máy cùng truy cập |
| Pacemaker | Pacemaker | Bộ công cụ cluster Linux chuyển IP và khởi động dịch vụ trên node dự phòng |
| Virtual IP | IP ảo | Địa chỉ IP có thể "di chuyển" giữa các máy |
| UID | Mã định danh người dùng | Số định danh người dùng hệ điều hành, quyết định quyền truy cập file |
| Corruption | Hỏng dữ liệu | Dữ liệu trên đĩa bị lỗi, không đọc được đúng |
| Black-holed | Rơi vào lỗ đen | Tin nhắn bị vứt bỏ âm thầm, không báo lỗi |
| mandatory flag | Cờ mandatory | Cờ khi publish yêu cầu broker trả lại tin nhắn nếu không định tuyến được tới queue nào |
| delivery_mode 2 | Chế độ giao bền vững | Thuộc tính đánh dấu tin nhắn cần được ghi xuống đĩa |
| Fabric | "Kết cấu" | Tập exchange, queue, binding mà ứng dụng cần |
| backup (HAProxy) | Máy dự phòng (HAProxy) | Tùy chọn trên dòng `server`: chỉ dùng khi mọi server chính đều hỏng |
| Health check (`check inter rise fall`) | Kiểm tra sức khỏe | HAProxy định kỳ kiểm tra backend; `inter` = chu kỳ, `rise`/`fall` = số lần thành công/thất bại liên tiếp |
| Data center | Trung tâm dữ liệu | Địa điểm vật lý đặt máy chủ |
| Geo-diverse | Phân tán địa lý | Hạ tầng trải trên nhiều vị trí địa lý |
| WAN | Mạng diện rộng | Mạng kết nối các địa điểm ở xa, thường có độ trễ cao |
| LAN | Mạng cục bộ | Mạng trong cùng một địa điểm, độ trễ thấp |
| Latency | Độ trễ | Thời gian để dữ liệu đi từ điểm này tới điểm kia |
| Network partitioning | Phân mảnh mạng | Mạng bị chia cắt khiến các node không liên lạc được với nhau |
| OTP | OTP (Open Telecom Platform) | Framework/thư viện chuẩn của Erlang, dùng cho giao tiếp giữa các node |
| Shovel | Shovel ("cái xẻng") | Plugin consume tin nhắn từ queue ở broker nguồn và publish sang exchange ở broker đích |
| Replication relationship | Quan hệ sao chép | Định nghĩa nguồn → đích cho một shovel |
| Plugin | Plugin | Mô-đun mở rộng được RabbitMQ nạp khi khởi động |
| amqp_client | amqp_client | Thư viện AMQP client viết bằng Erlang, Shovel phụ thuộc vào nó |
| rabbitmq.config | rabbitmq.config | File cấu hình RabbitMQ, viết theo cú pháp Erlang |
| Tuple (Erlang) | Bộ giá trị | Cấu trúc dữ liệu Erlang dạng `{a, b, ...}` |
| Binary (Erlang) | Kiểu binary | Kiểu dữ liệu Erlang viết dạng `<<"...">>`; bắt buộc cho chuỗi trong cấu hình Shovel |
| sources / destinations | Nguồn / đích | Hai phần cấu hình broker và declarations của shovel |
| broker / brokers | Broker / danh sách broker | URL AMQP của máy chủ (danh sách nhiều URL khi là cluster, để failover) |
| declarations | Khai báo | Danh sách lệnh AMQP (queue.declare, exchange.declare, queue.bind) Shovel chạy trước khi hoạt động |
| ack_mode | Chế độ xác nhận | `no_ack`, `on_publish`, `on_confirm` — khi nào Shovel ack tin nhắn ở nguồn |
| prefetch_count | Số tin nhắn nạp trước | Số tin nhắn Shovel đệm nội bộ (không được bảo vệ khi Shovel hỏng) |
| publish_properties | Thuộc tính khi publish | Thuộc tính tin nhắn bị ghi đè khi publish tới đích |
| publish_fields | Trường khi publish | Exchange và routing key dùng khi publish tới đích |
| reconnect_delay | Độ trễ kết nối lại | Số giây chờ trước khi kết nối lại |
| Federation | Liên kết (federation) | Kết nối, chia sẻ tin nhắn giữa các broker/cluster độc lập |
| Management plugin | Plugin quản lý | Plugin cung cấp giao diện web/API quản trị RabbitMQ (chủ đề chương sau) |
