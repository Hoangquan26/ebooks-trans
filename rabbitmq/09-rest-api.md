# Chương 9: Điều khiển Rabbit bằng REST API (Controlling Rabbit with the REST API)

## Mục lục chương

- [Mở đầu chương](#mở-đầu-chương)
- [9.1 Bạn có thể làm gì với RabbitMQ REST API? (What can you do with the RabbitMQ REST API?)](#91-bạn-có-thể-làm-gì-với-rabbitmq-rest-api-what-can-you-do-with-the-rabbitmq-rest-api)
- [9.2 Cấp quyền truy cập cho client (Granting your clients access)](#92-cấp-quyền-truy-cập-cho-client-granting-your-clients-access)
- [9.3 Truy cập số liệu thống kê (Accessing statistics)](#93-truy-cập-số-liệu-thống-kê-accessing-statistics)
- [9.4 Tự động hóa việc cấp phát vhost và user (Automating vhost and user provisioning)](#94-tự-động-hóa-việc-cấp-phát-vhost-và-user-automating-vhost-and-user-provisioning)
- [9.5 Tóm tắt (Summary)](#95-tóm-tắt-summary)
- [Bảng thuật ngữ của chương](#bảng-thuật-ngữ-của-chương)

---

## Mở đầu chương

**Chương này đề cập đến (This chapter covers):**

- Những giới hạn và khả năng của Rabbit REST API
- Quản lý quyền (*permissions*) của Rabbit từ code
- Truy cập số liệu thống kê (*statistics*) và các bộ đếm (*counters*) về message
- Tự động hóa việc tạo user và virtual host

Cho đến thời điểm này, khả năng cấu hình server RabbitMQ từ ứng dụng hay từ script của bạn vẫn còn khá hạn chế. Đúng là bạn có thể viết code chạy lệnh `rabbitmqctl` rồi cố gắng "cào" (*scrape*) kết quả in ra để lấy thông tin. Nhưng đó là một giải pháp rất mong manh (*brittle*): nó gần như chắc chắn sẽ hỏng bất cứ khi nào các anh chàng ở nhóm phát triển Rabbit quyết định thay đổi định dạng output của `rabbitmqctl`.

Thực tế là cả `rabbitmqctl` lẫn giao diện web quản lý (*management web UI*) đều được thiết kế để tương tác với "thứ gì đó có nhịp tim" — tức là với **con người**. Vậy bạn sẽ làm gì khi muốn tự động hóa việc triển khai (*deployment*) các server RabbitMQ bằng những công cụ như Chef, Puppet, hay thậm chí CFEngine? Và còn nhu cầu cơ bản hơn nữa là **giám sát** (*monitor*) RabbitMQ thì sao? Làm thế nào để viết các script kiểm tra sức khỏe (*health check*) để để mắt tới "lũ thỏ" của bạn, nếu không có một "cửa sổ lập trình được" nhìn vào hoạt động bên trong của RabbitMQ?

> 📝 **Giải thích thêm (không có trong sách):** Chef, Puppet và CFEngine là các công cụ *quản lý cấu hình* (*configuration management*). Bạn viết "công thức" (recipe/manifest) mô tả trạng thái mong muốn của server (cài gói gì, tạo user nào, file cấu hình ra sao…), và công cụ sẽ tự động đưa hàng chục, hàng trăm server về đúng trạng thái đó. Những công cụ này cần một cách gọi "máy nói chuyện với máy" ổn định — đó chính là lý do REST API quan trọng.

Nếu bạn đọc cuốn sách này theo thứ tự, hẳn bạn đã quen với plugin quản lý RabbitMQ (*RabbitMQ Management plugin*) — xem chương 8 để biết cách cài đặt. Plugin này cung cấp khả năng truy cập qua web, cho phép bạn quản lý và điều khiển một server RabbitMQ ngay từ trình duyệt. Thông qua giao diện web đi kèm, quản trị viên có thể làm mọi thứ: từ tạo user và vhost cho đến xem thống kê queue và cấu hình tổng thể. Điều tuyệt nhất là: khi cài Management plugin, bạn không chỉ có giao diện web, mà còn được tặng **miễn phí** một web API theo phong cách RESTful. API này cung cấp cho ứng dụng và script của bạn **đầy đủ chức năng** giống hệt như giao diện web hoặc `rabbitmqctl`.

Khi Rabbit HQ (đội ngũ phát triển RabbitMQ) phát hành Management plugin, họ không chỉ cho lập trình viên một giao diện web thân thiện với con người, mà còn kèm theo một API quản lý dạng web theo kiểu RESTful. API này là một cách đơn giản, **trung lập về ngôn ngữ** (*language-neutral*) và **không cần Erlang** (*Erlang-free*) để cấu hình và theo dõi các node Rabbit đang chạy Management plugin.

Phần "không cần Erlang" đặc biệt quan trọng. Dù Erlang là nền tảng giúp RabbitMQ có tính phân tán, khả năng mở rộng và ổn định, bạn có thể muốn điều khiển Rabbit từ những hệ thống **không cài Erlang**. May mắn thay, management API của RabbitMQ giao tiếp bằng HTTP, nên bạn có thể nói chuyện với nó từ bất kỳ ngôn ngữ lập trình hay ngôn ngữ script nào có thư viện HTTP client.

> **GHI CHÚ (NOTE)**
>
> Dành cho những ai chưa quen thuật ngữ này: REST là viết tắt của **Representational State Transfer** (tạm dịch: "chuyển giao trạng thái biểu diễn"). Nó mô tả một quy ước cho các API dựa trên HTTP, trong đó **đối tượng (item) bạn đang thay đổi** và **trạng thái của hành động lên đối tượng đó** được mã hóa ngay trong URL.
>
> Ví dụ, bạn có thể có một API *không* RESTful chỉ dùng một URL duy nhất như `http://my-api.com/calls`, rồi nhét toàn bộ chi tiết về đối tượng và hành động vào phần thân (*body*) của request. Nhưng nếu bạn muốn khai thác dữ liệu (*data mining*) từ log của API server, tất cả những gì bạn thấy chỉ là một danh sách các request tới `/calls`. Chẳng giúp ích gì mấy.
>
> Khi dùng quy ước RESTful cho API, bạn sẽ có URL kiểu `http://my-api.com/item` (trong đó `item` là tên đối tượng bạn đang thao tác), rồi dùng các động từ HTTP chuẩn (*HTTP verbs*) như `POST`, `PUT` và `DELETE` để tạo, sửa hoặc xóa đối tượng. Lúc này log của bạn đầy thông tin hữu ích, cho biết chính xác đối tượng nào đang bị thao tác và hành động (động từ) nào được thực hiện trên nó. Để tìm hiểu thêm về REST, xem http://en.wikipedia.org/wiki/Representational_State_Transfer.

> 💡 **Ví dụ bổ sung (không có trong sách):** So sánh hai kiểu log của API server khi bạn tạo rồi xóa queue `orders`:
>
> | Kiểu API | Dòng log thứ nhất | Dòng log thứ hai |
> |---|---|---|
> | Không RESTful | `POST /calls` (body: `{"action":"create_queue","name":"orders"}`) | `POST /calls` (body: `{"action":"delete_queue","name":"orders"}`) |
> | RESTful | `PUT /api/queues/%2F/orders` | `DELETE /api/queues/%2F/orders` |
>
> Nhìn vào log RESTful, chỉ cần đọc dòng request là bạn biết ngay "ai làm gì với cái gì", không cần mở body.

Trước khi lao vào viết chương trình tương tác với management API, chúng ta cần nắm qua những gì API này cho phép (và *không* cho phép) bạn làm. Khi đã nắm vững điều đó, bạn sẽ sẵn sàng tìm hiểu cách tạo thông tin xác thực (*credentials*) cho các API client, rồi dùng quyền truy cập đó để xem số liệu thống kê nội bộ của RabbitMQ và thực hiện các thay đổi trên server Rabbit, như thêm user và virtual host. Hãy bắt đầu và xem management API có gì để mời chào nhé!

---

## 9.1 Bạn có thể làm gì với RabbitMQ REST API? (What can you do with the RabbitMQ REST API?)

Điều đầu tiên cần hiểu về API này là nó **hoàn toàn RESTful**, nên tên của đối tượng bạn đang thao tác luôn nằm trong URL. Ví dụ, nếu bạn muốn xem thống kê của queue tên `branches` trong virtual host `oak`, bạn sẽ dựng URL sau và gửi nó tới server dưới dạng một HTTP request `GET`:

```
http://localhost:55672/api/queues/oak/branches
```

(¹ Chú thích chân trang: Với các ví dụ trong chương này, chúng tôi giả định RabbitMQ đang chạy trên `localhost` và bạn chưa thay đổi cổng lắng nghe mặc định `55672` của Management plugin.)

> 📝 **Giải thích thêm (không có trong sách):** Cổng `55672` là cổng mặc định của Management plugin ở các phiên bản RabbitMQ thời điểm sách được viết (2.x). Từ RabbitMQ 3.0 trở đi (bao gồm RabbitMQ 3.8–3.13 và RabbitMQ 4.0), có 3 thay đổi nền tảng mà bạn cần lưu ý khi chạy mọi ví dụ trong chương này:
>
> | Thành phần | RabbitMQ 2.x (trong sách) | RabbitMQ 3.x / 4.0 (hiện đại) |
> |---|---|---|
> | **Cổng HTTP REST API** | `55672` (`http://localhost:55672/api/...`) | **`15672`** (`http://localhost:15672/api/...`) — hoặc `15671` cho HTTPS |
> | **HTTP Web Server nội bộ** | `MochiWeb/1.1 WebMachine/1.7` | **`Cowboy`** (hiệu năng cao hơn, hỗ trợ HTTP/2) |
> | **Tài khoản `guest:guest`** | Truy cập được từ mọi IP qua mạng | **Chỉ đăng nhập được từ `localhost`** (`loopback_users.guest = true` mặc định từ RabbitMQ 3.3+). Gọi REST API từ máy khác bắt buộc phải tạo user riêng! |
> | **Định dạng thời gian (`idle_since`)** | `"2011-7-516:55:1"` (thiếu số 0 đệm và dấu cách) | Chuẩn ISO-8601 / RFC 3339: `"2026-03-31 14:22:05"` hoặc `"2026-03-31T14:22:05.123+00:00"` |
> | **Thống kê hàng đợi nội bộ** | Chỉ có `backing_queue_status` (Classic Queue) | Phụ thuộc loại queue: Classic Queue v2 (CQv2), Quorum Queue (`raft` metrics: `leader`, `members`, `open_files`), hoặc Stream |

Nếu bạn chạy request này bằng cURL, bạn sẽ thấy kết quả đại loại như sau:

```bash
$ curl -i -u guest:guest http://localhost:55672/api/queues/oak/branches
HTTP/1.1 200 OK
Server: MochiWeb/1.1 WebMachine/1.7 (participate in the frantic)
Date: Tue, 05 Jul 2011 22:55:25 GMT
Content-Type: application/json
Content-Length: 739
Cache-Control: no-cache

{"memory":9136,"messages":0,"consumer_details":[],
"idle_since":"2011-7-516:55:1","exclusive_consumer_pid":"",
"exclusive_consumer_tag":"","messages_ready":0,
"messages_unacknowledged":0,"messages":0,"consumers":0,
"backing_queue_status":{"q1":0,"q2":0,
 ...
```

> **GHI CHÚ (NOTE)**
>
> cURL là một chương trình miễn phí cho phép bạn gửi và nhận HTTP request thủ công từ dòng lệnh. Nó có sẵn trong hầu hết các bản phân phối Linux/UNIX và cũng có thể tải trực tiếp tại http://curl.haxx.se/download.html.

> 📝 **Giải thích thêm (không có trong sách):** Ý nghĩa các tham số cURL ở trên:
> - `-i`: in cả phần header của HTTP response (nhờ vậy bạn thấy dòng `HTTP/1.1 200 OK`, `Content-Type`, …).
> - `-u guest:guest`: gửi username `guest` và password `guest` theo kiểu HTTP Basic authentication.
> - Không có `-X` thì cURL mặc định dùng động từ `GET`.

Chà, ngoài mấy dòng HTTP header khá dễ đọc, cái mớ hỗn độn kia nghĩa là gì vậy? Chính các header sẽ cho bạn manh mối. Nhìn vào header `Content-Type`, bạn thấy response được mã hóa dưới dạng `application/json`. Dành cho ai chưa biết: JSON là viết tắt của **JavaScript Object Notation**, một lựa chọn thay thế cho XML để mã hóa dữ liệu (xem http://en.wikipedia.org/wiki/JSON để tìm hiểu sâu về JSON). JSON cho phép mã hóa dữ liệu bằng những cấu trúc quen thuộc như bảng băm (*hash table*), mảng (*array*), chuỗi (*string*) và số nguyên (*integer*) — những thứ mà các ngôn ngữ lập trình vốn đã hiểu (và quan trọng hơn, những thứ mà *bạn* vốn đã hiểu).

Nếu bạn biết JavaScript, hẳn bạn đã đoán ra API đang trả về toàn bộ dữ liệu về queue dưới dạng một hash table. Ví dụ, phần tử `memory` của hash table cho bạn biết queue hiện đang tiêu tốn bao nhiêu RAM (tính bằng byte). Khi làm việc với management API, mọi lời gọi sẽ trả về **một trong hai thứ**:

- một **body rỗng** (với các hành động tạo hoặc xóa đối tượng), hoặc
- một **hash table JSON** chứa dữ liệu bạn yêu cầu (với các hành động liệt kê hoặc hiển thị đối tượng).

> 💡 **Ví dụ bổ sung (không có trong sách):** Nếu định dạng lại phần JSON ở trên cho dễ nhìn (thụt lề), ta có:
>
> ```json
> {
>   "memory": 9136,
>   "messages": 0,
>   "consumer_details": [],
>   "idle_since": "2011-7-516:55:1",
>   "exclusive_consumer_pid": "",
>   "exclusive_consumer_tag": "",
>   "messages_ready": 0,
>   "messages_unacknowledged": 0,
>   "consumers": 0,
>   "backing_queue_status": { "q1": 0, "q2": 0 }
> }
> ```
>
> Đọc như một "từ điển": khóa `memory` có giá trị `9136` (byte), `consumers` là `0` (chưa có consumer nào), `messages_ready` là `0` (không có message nào đang chờ giao)… Mẹo: thêm `| python -m json.tool` vào cuối lệnh cURL (bỏ `-i`) để JSON được in đẹp như trên.

Quan trọng không kém dữ liệu mà một API request trả về là **động từ HTTP** bạn dùng để gửi request; ở trên bạn đã dùng `GET`. RabbitMQ API hiểu `GET` là "cho tôi xem toàn bộ chi tiết/thiết lập của đối tượng nằm tại `/api/queues/oak/branches`". Ngoài `GET`, bạn còn có thể dùng `POST`, `PUT` hoặc `DELETE` khi nói chuyện với API:

- `POST` và `PUT` dùng để **tạo** đối tượng.
- `DELETE` làm đúng như tên gọi — nó **xóa** đối tượng.

`POST` và `PUT` **không thể dùng thay thế cho nhau**: một số lời gọi API yêu cầu `POST` để tạo đối tượng được tham chiếu trong URL, trong khi số khác lại dùng `PUT` (xem đường dẫn tham khảo ở cuối mục để có danh sách chi tiết chỗ nào dùng `PUT`, chỗ nào dùng `POST`).

Giả sử trong một phút rằng queue `branches` chưa tồn tại, và bạn muốn tạo nó bằng API. Chỉ cần dùng lại URL như trước nhưng đổi động từ thành `PUT`, bạn có thể biến request từ "trả về chi tiết queue" thành "tạo queue":

```bash
$ curl -i -u guest:guest -X PUT -H "Content-Type: application/json" \
    -d '{"auto_delete": false, "durable": false}' \
    http://localhost:55672/api/queues/oak/branches
HTTP/1.1 204 No Content
Server: MochiWeb/1.1 WebMachine/1.7 (participate in the frantic)
Date: Tue, 05 Jul 2011 23:24:46 GMT
Content-Type: application/json
Content-Length: 0
```

Được rồi, thật ra bạn không *chỉ* đổi động từ thành `PUT`. Bạn còn thêm hash table JSON `'{"auto_delete": false, "durable": false}'` vào body, và thêm header `Content-Type` để API biết dữ liệu trong body được mã hóa JSON. Body của request yêu cầu API **tắt** cờ `auto_delete` và cờ `durable` khi tạo queue `branches`. Các cờ này có tác dụng y hệt như cờ `auto_delete` và `durable` khi tạo queue trong các ví dụ Python ở những chương trước. Thực tế, giống hệt như các ví dụ AMQP bằng Python, API **bắt buộc** phải chỉ định các cờ này khi tạo queue. Nếu bạn để body rỗng, bạn sẽ nhận về một lỗi API.

Dù bạn đang tạo queue bằng API hay thiết lập quyền, **bất cứ khi nào dùng động từ `PUT` hoặc `POST`**, hãy mã hóa các tham số của hàm bạn đang gọi thành một hash table JSON đặt trong body.

> 📝 **Giải thích thêm (không có trong sách):** Mã trạng thái `204 No Content` nghĩa là "thành công, nhưng không có nội dung gì để trả về" — đúng với quy tắc ở trên: các hành động tạo/xóa trả về body rỗng (`Content-Length: 0`). Ý nghĩa tham số cURL mới: `-X PUT` chọn động từ `PUT`; `-H "..."` thêm một header; `-d '...'` đặt dữ liệu vào body request.
>
> Trong RabbitMQ hiện đại (3.8–3.13 và RabbitMQ 4.0), khi tạo queue qua REST API cho môi trường production, bạn cũng truyền thêm đối tượng `"arguments"` trong JSON body để chọn loại queue hiện đại như **Quorum Queue** (`"x-queue-type": "quorum"`, bắt buộc `"durable": true`) hoặc **Stream** (`"x-queue-type": "stream"`), kèm theo các giới hạn an toàn như `"x-max-length"`:
>
> ```bash
> curl -i -u admin:secret -X PUT -H "Content-Type: application/json" \
>     -d '{"durable": true, "auto_delete": false, "arguments": {"x-queue-type": "quorum", "x-delivery-limit": 5}}' \
>     http://localhost:15672/api/queues/oak/branches
> ```

> 💡 **Ví dụ bổ sung (không có trong sách):** Vòng đời đầy đủ của một queue chỉ bằng cURL (cùng một URL, khác động từ):
>
> ```bash
> # 1. Tạo queue "branches" trong vhost "oak" (bền vững, không tự xóa)
> curl -i -u guest:guest -X PUT -H "Content-Type: application/json" \
>     -d '{"auto_delete": false, "durable": true}' \
>     http://localhost:55672/api/queues/oak/branches
>
> # 2. Xem chi tiết queue
> curl -u guest:guest http://localhost:55672/api/queues/oak/branches
>
> # 3. Xóa queue
> curl -i -u guest:guest -X DELETE http://localhost:55672/api/queues/oak/branches
> ```
>
> Bước 1 và 3 trả về `204 No Content`; bước 2 trả về `200 OK` kèm hash table JSON. Nếu ở bước 2 queue không tồn tại, API trả về mã lỗi `404 Not Found`.

Vậy là bạn có thể tạo queue và xem thống kê của chúng, nhưng còn làm được gì nữa? Dưới đây là một vài thứ khác mà API cho phép script của bạn làm:

- Xem danh sách các kết nối (*connections*) hiện tại và chi tiết của chúng — `/api/connections`
- Tải xuống hoặc tải lên **toàn bộ cấu hình** của RabbitMQ broker, bao gồm queue, exchange và binding — `/api/all-configuration`
- Liệt kê tất cả các node trong một cluster (và số liệu thống kê của chúng) — `/api/nodes`
- Tạo hoặc xem một user RabbitMQ — `/api/users/<user>`
- Xem hoặc tạo một virtual host — `/api/vhosts/<vhost>`
- Thiết lập quyền cho một user — `/api/permissions/<vhost>/<user>`

Vì management API (và plugin) luôn được cải tiến liên tục, đó mới chỉ là một danh sách nhỏ các chức năng mà API cung cấp. Bạn luôn có thể xem danh sách mới nhất (và đầy đủ nhất) các lời gọi API cùng các động từ HTTP mà chúng hỗ trợ bằng cách mở http://localhost:55672/api trong trình duyệt. Giải thích thế là đủ rồi! Hãy tạo vài thông tin xác thực để bạn có thể bắt đầu viết script dùng API nào!

> 💡 **Ví dụ bổ sung (không có trong sách):** Một số lệnh cURL thử nhanh các endpoint vừa liệt kê:
>
> ```bash
> # Liệt kê tất cả kết nối đang mở
> curl -u guest:guest http://localhost:55672/api/connections
>
> # Sao lưu toàn bộ cấu hình broker ra file
> curl -u guest:guest http://localhost:55672/api/all-configuration > rabbit-backup.json
>
> # Tạo vhost "oak" (PUT, body rỗng)
> curl -i -u guest:guest -X PUT -H "Content-Type: application/json" \
>     http://localhost:55672/api/vhosts/oak
>
> # Cấp quyền đầy đủ cho user "guest" trên vhost "oak"
> curl -i -u guest:guest -X PUT -H "Content-Type: application/json" \
>     -d '{"configure": ".*", "write": ".*", "read": ".*"}' \
>     http://localhost:55672/api/permissions/oak/guest
> ```
>
> (Ở RabbitMQ 3.x/4.0, endpoint `/api/all-configuration` được đổi tên thành `/api/definitions`.)

> 📝 **Giải thích thêm (không có trong sách) — Các Endpoint REST API mới trong RabbitMQ 3.8–4.0 và Lưu ý Hiệu năng Production:**
>
> So với thời điểm năm 2012 (RabbitMQ 2.x), REST API của **RabbitMQ 3.8–3.13 và RabbitMQ 4.0** (truy cập tài liệu tương tác tại `http://localhost:15672/api/index.html`) đã bổ sung hàng loạt endpoint quản trị vận hành cực kỳ quan trọng:
>
> | Nhóm chức năng | Endpoint trên RabbitMQ 3.8–4.0 | Công dụng thực tế |
> |---|---|---|
> | **Xuất/Nhập cấu hình (Definitions)** | `GET / POST /api/definitions`<br>`GET / POST /api/definitions/<vhost>` | Thay thế `/api/all-configuration` cũ; cho phép xuất/nhập schema của toàn cluster hoặc chỉ riêng từng vhost. |
> | **Quản lý Policy & Operator Policy** | `PUT /api/policies/<vhost>/<name>`<br>`PUT /api/operator-policies/<vhost>/<name>` | Áp dụng động cấu hình TTL, `max-length`, Dead Letter Exchange cho hàng loạt queue khớp regex. **Operator Policy** giúp đội vận hành (Ops) ép buộc ngưỡng trần (như `max-length: 100000`) mà đội phát triển (Dev) không thể ghi đè! |
> | **Giới hạn bảo vệ Vhost & User** | `PUT /api/vhost-limits/<vhost>/<type>`<br>`PUT /api/user-limits/<user>/<type>` | Giới hạn số kết nối tối đa (`max-connections`) và số queue tối đa (`max-queues`) trên một vhost hoặc một user để chống một ứng dụng bị lỗi làm cạn kiệt file descriptors của cả cụm. |
> | **Cân bằng lại Quorum Queue Leader** | `POST /api/rebalance/queues` | Sau khi khởi động lại từng node (rolling upgrade), các Raft leader của Quorum Queue thường dồn về các node cuối. Gọi endpoint này sẽ tự động phân bổ đều lại leader trên khắp các node trong cluster! |
> | **Đóng cưỡng bức kết nối** | `DELETE /api/connections/<name>` | Cho phép script tự động ngắt kết nối của các client bị treo hoặc vi phạm bảo mật (kèm header `X-Reason`). |
> | **Health Checks chuyên dụng** | `GET /api/health/checks/alarms`<br>`GET /api/health/checks/local-alarms` | Bộ kiểm tra sức khỏe thế hệ mới (xem chi tiết ở Chương 10). |
>
> ⚠️ **Lưu ý hiệu năng sống còn trên Production khi gọi `GET /api/queues` hoặc `GET /api/connections`:**
> Trên các cụm RabbitMQ lớn có hàng chục nghìn queue hoặc kết nối, việc một script giám sát hoặc dashboard gọi `GET /api/queues` (không tham số) liên tục mỗi vài giây là một **anti-pattern nguy hiểm**. Node nhận request sẽ phải thu thập toàn bộ metrics chi tiết của hàng vạn đối tượng và serialize một cục JSON khổng lồ, gây nghẽn bộ thu thập thống kê (*management stats collector*) và làm tăng vọt RAM/CPU.
> Trong RabbitMQ hiện đại, bạn **luôn nên**:
> 1. **Sử dụng phân trang (Pagination) và lọc phía server**:
>    `GET /api/queues?page=1&page_size=100&name=%5Eorders&use_regex=true`
> 2. **Tắt thống kê chi tiết (`?disable_stats=true`)** nếu script chỉ cần lấy danh sách tên/cấu hình queue hoặc chỉ cần số lượng tổng (`&enable_queue_totals=true`):
>    `GET /api/queues?disable_stats=true&enable_queue_totals=true`

> 💡 **Ví dụ bổ sung (không có trong sách):** Quản lý Policy, Vhost Limits, Rebalance Queues và truy vấn phân trang trên RabbitMQ 3.8–4.0 bằng cURL (cổng `15672`):
>
> ```bash
> # 1. Tạo Operator Policy ép mọi queue trong vhost "/" không được vượt quá 500.000 message
> curl -i -u admin:secret -X PUT -H "Content-Type: application/json" \
>     -d '{"pattern": ".*", "definition": {"max-length": 500000, "overflow": "drop-head"}, "priority": 1, "apply-to": "queues"}' \
>     http://localhost:15672/api/operator-policies/%2F/protect-memory-cap
>
> # 2. Đặt giới hạn tối đa 500 kết nối và 200 queue cho vhost "oak"
> curl -i -u admin:secret -X PUT -H "Content-Type: application/json" \
>     -d '{"value": 500}' http://localhost:15672/api/vhost-limits/oak/max-connections
> curl -i -u admin:secret -X PUT -H "Content-Type: application/json" \
>     -d '{"value": 200}' http://localhost:15672/api/vhost-limits/oak/max-queues
>
> # 3. Cân bằng lại (rebalance) các queue master/leader trên toàn cluster
> curl -i -u admin:secret -X POST http://localhost:15672/api/rebalance/queues
>
> # 4. Lấy danh sách queue an toàn trên production: phân trang 50 mục/trang + tắt stats nặng
> curl -s -u admin:secret \
>     "http://localhost:15672/api/queues?page=1&page_size=50&disable_stats=true&enable_queue_totals=true"
> ```

---

## 9.2 Cấp quyền truy cập cho client (Granting your clients access)

Có thể bạn đã để ý rằng cả hai API request bạn gửi bằng cURL đều có tham số `-u guest:guest`. Giống như bất kỳ ứng dụng nào truy cập RabbitMQ qua AMQP, các script truy cập management API cũng dùng username và password RabbitMQ bình thường để xác thực và được cấp quyền truy cập (thông tin này được gửi bằng cơ chế xác thực cơ bản của HTTP — *HTTP Basic authentication*).

Ví dụ, nếu bạn có một script giám sát chỉ nên được phép kiểm tra thống kê queue, bạn có thể tạo một user Rabbit mới có quyền trên virtual host như sau:

```
Read:   .*
Write:  (để trống)
Config: (để trống)
```

Điều này cho phép script (và bất kỳ ai biết username và password Rabbit của script) **chỉ giám sát** các queue, chứ không thể publish message vào chúng hay thay đổi cấu hình của chúng.

> 📝 **Giải thích thêm (không có trong sách):** Như đã học ở chương trước, quyền trong RabbitMQ gồm ba loại, mỗi loại là một biểu thức chính quy (*regex*) so khớp với tên tài nguyên (queue/exchange):
> - **Configure (Config)**: tạo/xóa/thay đổi tài nguyên.
> - **Write**: publish message vào tài nguyên.
> - **Read**: đọc/consume message từ tài nguyên.
>
> `.*` khớp với mọi tên; chuỗi rỗng `""` không khớp với tên nào — tức là "không có quyền".

Vậy làm sao để tạo một username dùng cho truy cập API? Đơn giản: tạo user bằng `rabbitmqctl` và đặt thuộc tính `admin` thành `true`. Hãy tạo một user tên `monitor` mà các script của bạn có thể dùng để giám sát thống kê trong virtual host mặc định (`/`) (nhưng không được ghi hay thay đổi gì). Từ thư mục RabbitMQ của bạn, chạy:

```bash
$ sbin/rabbitmqctl add_user monitor monitorMe
Creating user "monitor" ...
...done.
$ sbin/rabbitmqctl set_admin monitor
Setting administrative status for user "monitor" ...
...done.
$ sbin/rabbitmqctl set_permissions -p / monitor "" "" ".*"
Setting permissions for user "monitor" in vhost "/" ...
...done.
```

Chỉ có vậy thôi! Phần quan trọng là lệnh `set_admin`. Nếu không bật cờ admin cho username, user đó sẽ **không được phép truy cập API**, bất kể đã được thiết lập quyền gì. Lệnh cuối cùng (`set_permissions`) cấp cho user `monitor` **không** có quyền configure hay write, và có **toàn quyền read** trong virtual host mặc định (`/`). Có user `monitor` trong tay rồi, bạn đã sẵn sàng viết script API đầu tiên và bắt đầu xem thống kê queue.

> 📝 **Giải thích thêm (không có trong sách):** Thứ tự ba tham số cuối của `set_permissions` là **configure, write, read**. Vì vậy `"" "" ".*"` = không configure, không write, read tất cả.
>
> Lưu ý quan trọng về phiên bản: lệnh `set_admin` chỉ có ở RabbitMQ 2.x. Từ RabbitMQ 3.0 trở đi (và xuyên suốt RabbitMQ 3.8–4.0), cờ nhị phân `admin` (`true`/`false`) đã bị loại bỏ và thay bằng hệ thống **User Tags** phân quyền mịn hơn rất nhiều. Bạn gán tag bằng lệnh `rabbitmqctl set_user_tags <user> <tag1> [tag2 ...]`:
>
> | User Tag (RabbitMQ 3.x / 4.0) | Quyền hạn trên Management UI & REST API | Trường hợp sử dụng chuẩn |
> |---|---|---|
> | *(Không có tag nào)* | **Không** thể đăng nhập Management UI hay gọi REST API (chỉ kết nối được qua AMQP/MQTT/STOMP). | Tài khoản của các microservice sản xuất/tiêu thụ message thông thường. |
> | **`management`** | Đăng nhập được UI/REST API; chỉ xem được các vhost mà user đó có quyền, xem/đóng kết nối của chính mình, xem queue/exchange trong vhost được cấp quyền. | Lập trình viên cần tự kiểm tra vhost của dự án mình mà không thấy vhost của đội khác. |
> | **`policymaker`** | Gồm toàn bộ quyền của `management` + quyền xem/tạo/xóa **Policies** và **Parameters** trên các vhost mà user có quyền. | Trưởng nhóm kỹ thuật / CI-CD pipeline cần cấu hình policy cho vhost của ứng dụng. |
> | **`monitoring`** | Gồm toàn bộ quyền của `management` + xem được **mọi** vhost, mọi kết nối/channel, và các chỉ số cấp độ node/cluster (`/api/nodes`, `/api/overview`), nhưng không sửa/xóa được user hay vhost. | Tài khoản chuyên dụng cho script giám sát, Prometheus exporter, Datadog/Nagios agent. |
> | **`administrator`** | Toàn quyền tối cao (tạo/xóa user, vhost, permissions, đóng mọi kết nối, thay đổi toàn bộ cluster). | Quản trị viên hệ thống hoặc công cụ Infrastructure-as-Code (Terraform, Ansible). |
>
> Vì vậy, với user `monitor` chỉ dùng để đọc thống kê trên RabbitMQ hiện đại, bạn tuyệt đối không nên cấp `administrator`, mà chỉ cần cấp tag `monitoring`:
>
> ```bash
> rabbitmqctl add_user monitor monitorMe
> rabbitmqctl set_user_tags monitor monitoring
> rabbitmqctl set_permissions -p / monitor "" "" ".*"
> ```
>
> Ngoài ra, trong **RabbitMQ 3.8–4.0**, bên cạnh HTTP Basic Auth (`-u user:pass`), Management REST API còn hỗ trợ xác thực hiện đại qua **OAuth 2.0 (JWT Bearer Token)** với plugin `rabbitmq_auth_backend_oauth2` — cho phép script gửi header `Authorization: Bearer <access_token>` mà không cần lưu mật khẩu tĩnh!

> 💡 **Ví dụ bổ sung (không có trong sách):** *Bài toán:* Bạn cần ba tài khoản cho ba script khác nhau trên vhost `/`: (a) script giám sát, (b) script chỉ đẩy log vào exchange `logs`, (c) script triển khai được phép khai báo mọi queue/exchange. Quyền nên đặt thế nào?
>
> | Script | Configure | Write | Read |
> |---|---|---|---|
> | (a) giám sát | `""` | `""` | `.*` |
> | (b) đẩy log | `""` | `^logs$` | `""` |
> | (c) triển khai | `.*` | `.*` | `.*` |
>
> Nguyên tắc: chỉ cấp **vừa đủ** quyền cho việc script cần làm (*principle of least privilege*). Nếu mật khẩu của script (a) bị lộ, kẻ xấu cũng chỉ xem được số liệu chứ không phá được hệ thống.

---

## 9.3 Truy cập số liệu thống kê (Accessing statistics)

Hằng ngày có rất nhiều lúc bạn cần xem có bao nhiêu message đang nằm trong một queue cụ thể. Đôi khi là để gỡ lỗi (*debug*) một ứng dụng mới đang viết; lúc khác là để giám sát trong môi trường production tỷ lệ giữa:

- các message **đang chờ được consume**, và
- các message **đã được giao cho consumer nhưng vẫn chưa được xác nhận** (*unacknowledged*).

(Con số thứ hai có thể là một chỉ số hữu ích để phát hiện những message đang làm consumer của bạn bị crash.)

Bạn có thể dùng `rabbitmqctl` để liệt kê tổng số message trong các queue, nhưng cách này có hai nhược điểm lớn:

1. Bạn chỉ có thể chạy `rabbitmqctl` từ những máy tính có cài Erlang **và** có cùng Erlang cookie với server RabbitMQ.
2. `rabbitmqctl` chỉ cho bạn thấy **tổng số** message trong queue. Nó không phân biệt giữa message đã giao và đang chờ xác nhận với message vẫn đang chờ được consume.

> 📝 **Giải thích thêm (không có trong sách):** Erlang cookie là một chuỗi bí mật (thường nằm trong file `~/.erlang.cookie`). Hai node Erlang chỉ nói chuyện được với nhau nếu có cùng cookie. Vì `rabbitmqctl` thực chất là một node Erlang nhỏ kết nối tới node RabbitMQ, nên máy chạy `rabbitmqctl` phải có Erlang và cookie trùng khớp — rất phiền khi bạn có nhiều máy.

> 💡 **Ví dụ bổ sung (không có trong sách):** Vì sao cần tách "Ready" và "Unacknowledged"? Giả sử queue `orders` có tổng 1.000 message:
> - Trường hợp A: Ready = 1.000, Unack'd = 0 → consumer **không theo kịp** hoặc **không có consumer nào** đang chạy. Cần thêm consumer.
> - Trường hợp B: Ready = 0, Unack'd = 1.000 → consumer đã nhận hết message nhưng **không ack** — có thể chúng bị treo, crash giữa chừng, hoặc quên gọi `basic_ack`. Thêm consumer cũng không giải quyết được!
>
> `rabbitmqctl` (thời đó) chỉ báo "1.000" cho cả hai trường hợp, nên bạn không chẩn đoán được.

Nghe như một công việc hoàn hảo cho Rabbit management API! Bạn sẽ dùng các thư viện có sẵn của Python là `httplib` và `json` để giao tiếp với Rabbit và mã hóa/giải mã request và response. Vì bạn muốn chạy script truy vấn thống kê từ dòng lệnh, bạn cần bắt đầu bằng việc phân tích (*parse*) các tham số dòng lệnh, như trong listing sau.

**Listing 9.1 – queue_stats.py — lấy các thiết lập ban đầu**

```python
import sys, json, httplib, urllib, base64

# (B) Kiểm tra số lượng tham số
if len(sys.argv) < 6:
    print "USAGE: queue_stats.py server_name:port auth_user " + \
          "auth_pass VHOST QUEUE_NAME"
    sys.exit(1)

# (C) Gán các tham số vào những biến có tên dễ nhớ
server, port = sys.argv[1].split(":")
username = sys.argv[2]
password = sys.argv[3]
vhost = sys.argv[4]
queue_name = sys.argv[5]
```

Chú thích trong listing:
- **(B) Validate argument count** – Kiểm tra số lượng tham số.
- **(C) Assign arguments to memorable variables** – Gán các tham số vào biến có tên dễ nhớ.

Tiện ích này nhận năm tham số (không kể tên chương trình):

1. tên server/cổng (theo dạng `host:port`),
2. username để xác thực API,
3. password để xác thực API,
4. tên virtual host chứa queue,
5. tên queue mà bạn muốn xem thống kê.

Sau khi kiểm tra **(B)** rằng có đủ số tham số tối thiểu bắt buộc, bạn gán chúng **(C)** vào những biến dễ nhớ hơn, dựa trên vị trí của từng tham số trong danh sách. Chỗ duy nhất hơi lạ là `server, port = sys.argv[1].split(":")`. Vì bạn truyền hostname và cổng của server Rabbit trong **một** tham số duy nhất, ngăn cách bởi dấu `:` (`localhost:55672`), bạn tách tham số đó thành hai phần hostname và cổng riêng biệt. Lệnh `split` cắt tham số tại dấu `:` và trả về một mảng chứa các phần đã tách (ví dụ, `"localhost:55672"` trở thành `["localhost", "55672"]`). Các phần đã tách sau đó được gán vào biến `server` và `port`.

> 📝 **Giải thích thêm (không có trong sách):** Code trong chương này viết cho **Python 2** (câu lệnh `print` không có ngoặc, module `httplib`, `urllib.quote`). Trong Python 3, các module tương ứng là `http.client` và `urllib.parse.quote`, `print` là hàm, và `base64.b64encode` nhận/trả về `bytes`. Ở cuối mục này có một phiên bản Python 3 để bạn tham khảo.

Khi đã phân tích xong tham số, bạn sẵn sàng dựng request. Mọi thao tác API về queue đều nằm dưới đường dẫn `/api/queues`. Vì vậy, nếu muốn truy cập một queue cụ thể, bạn chỉ cần nối dài đường dẫn để chỉ ra virtual host chứa queue đó và tên của queue: `/api/queues/<vhost>/<queue_name>`. Listing sau là một ví dụ.

**Listing 9.2 – queue_stats.py — dựng request**

```python
# (B) Dựng đường dẫn API (escape tên vhost và tên queue trước)
vhost = urllib.quote(vhost, safe='')
queue_name = urllib.quote(queue_name, safe='')
path = "/api/queues/%s/%s" % (vhost, queue_name)

# Đặt phương thức (động từ HTTP) của request
method = "GET"
```

Chú thích trong listing:
- **(B) Build API path** – Dựng đường dẫn API.
- **Set request method** – Đặt phương thức của request.

Có thể bạn để ý rằng bạn đã escape **(B)** (mã hóa ký tự đặc biệt) cả tên virtual host lẫn tên queue trước khi đưa chúng vào đường dẫn của request. Nếu không quote virtual host, thì việc chỉ định virtual host mặc định (`/`) sẽ gây ra lỗi API, vì server coi `/` là ký tự **phân cách đường dẫn**. Nhờ escape tên virtual host trước, đường dẫn request `/api/queues///test_queue` trở thành `/api/queues/%2F/test_queue` và API server có thể hiểu được. Cuối cùng, bạn đặt phương thức HTTP là `GET` để server biết bạn muốn **lấy chi tiết** về queue chứ không phải tạo nó.

> 💡 **Ví dụ bổ sung (không có trong sách):** Thử trong trình thông dịch Python 2 để thấy tác dụng của `urllib.quote(..., safe='')`:
>
> ```python
> >>> import urllib
> >>> urllib.quote("/", safe='')
> '%2F'
> >>> urllib.quote("my queue/v2", safe='')
> 'my%20queue%2Fv2'
> >>> urllib.quote("/")          # mặc định safe='/', nên '/' KHÔNG bị escape!
> '/'
> ```
>
> Tham số `safe=''` rất quan trọng: mặc định `urllib.quote` coi `/` là ký tự "an toàn" và giữ nguyên nó — đúng thứ ta *không* muốn ở đây. Tương tự, tên queue chứa dấu cách hay dấu `/` cũng sẽ phá vỡ đường dẫn nếu không escape.

Khi đã đặt xong đường dẫn và phương thức, bạn sẵn sàng bắn request tới server, như trong listing sau.

**Listing 9.3 – queue_stats.py — gửi API request**

```python
# (B) Kết nối tới API server
conn = httplib.HTTPConnection(server, port)

# (C) Mã hóa Base64 thông tin xác thực
credentials = base64.b64encode("%s:%s" % (username, password))

# (D) Đặt các header
headers = {"Content-Type" : "application/json",
           "Authorization" : "Basic " + credentials}

# (E) Gửi request
conn.request(method, path, "", headers)

# (F) Nhận response
response = conn.getresponse()
if response.status > 299:
    print "Error executing API call (%d): %s" % (response.status,
                                                 response.read())
    sys.exit(2)
```

Chú thích trong listing:
- **(B) Connect to API server** – Kết nối tới API server.
- **(C) Base64 credentials** – Mã hóa Base64 thông tin xác thực.
- **(D) Set headers** – Đặt các header.
- **(E) Send request** – Gửi request.
- **(F) Receive response** – Nhận response.

Đầu tiên script thiết lập một kết nối HTTP **(B)** với API server. Tại thời điểm này, server đang chờ API request và thông tin xác thực. Vì Rabbit API dùng HTTP Basic authentication, bạn cần đóng gói **(C)** username và password thành một chuỗi duy nhất theo định dạng `username:password` rồi mã hóa chuỗi đó bằng Base64.²

(² Chú thích chân trang: Base64 là một cách mã hóa văn bản và dữ liệu nhị phân để chúng có thể được biểu diễn chỉ bằng các ký tự ASCII. Đặc tả HTTP Basic authentication yêu cầu chuỗi `username:password` phải được mã hóa Base64 trước khi gửi đi. Để biết thêm về Base64, xem http://en.wikipedia.org/wiki/Base64.)

Tiếp theo, bạn cần tạo một dictionary (hash table) chứa các HTTP header **(D)** cho request. Một trong các header đó là `Authorization`, chứa thông tin xác thực đã mã hóa Base64, được nối vào sau chuỗi ASCII `"Basic"`. Header còn lại là `Content-Type`, và nó cũng quan trọng không kém, vì nó cho API server biết body request của bạn (nếu có) sẽ được mã hóa dạng JSON. Vì tiện ích của bạn chỉ gửi request `GET`, body request sẽ luôn rỗng, nhưng đặt `Content-Type` vẫn là thói quen tốt để bạn không quên khi nó *thật sự* quan trọng (với request `PUT` hoặc `POST`).

Cuối cùng, bạn gửi **(E)** request đã chuẩn bị tới API server và nhận response **(F)** về. Nếu có bất kỳ vấn đề nào khi thực hiện request, API server sẽ đặt mã trạng thái HTTP (*HTTP response code*) thành giá trị `4xx` hoặc `5xx`. Vì bất kỳ mã nào cao hơn 299 đều là mã response không chấp nhận được, bạn kiểm tra rằng mã thấp hơn 299 **(F)**, và nếu không phải, thoát chương trình và báo lỗi cho người dùng. Nếu không có lỗi, bạn sẵn sàng phân tích response và hiển thị chi tiết queue cho người dùng, như trong listing sau.

> 💡 **Ví dụ bổ sung (không có trong sách):** Tính tay header `Authorization` cho user `monitor` / mật khẩu `monitorMe`:
>
> ```python
> >>> import base64
> >>> base64.b64encode("monitor:monitorMe")
> 'bW9uaXRvcjptb25pdG9yTWU='
> ```
>
> Vậy header gửi đi là `Authorization: Basic bW9uaXRvcjptb25pdG9yTWU=`. Lưu ý: Base64 **không phải mã hóa bảo mật** — ai bắt được gói tin cũng giải mã ngược lại được ngay. Vì vậy trong môi trường thật nên dùng HTTPS cho management API.
>
> Bảng ý nghĩa nhanh các mã trạng thái hay gặp khi gọi API: `200 OK` (thành công, có dữ liệu), `204 No Content` (thành công, không có dữ liệu), `401 Unauthorized` (sai user/mật khẩu hoặc user không có quyền dùng API), `404 Not Found` (vhost/queue/user không tồn tại), `500 Internal Server Error` (lỗi phía server).

**Listing 9.4 – queue_stats.py — phân tích và hiển thị thống kê queue**

```python
# (B) Giải mã response JSON
payload = json.loads(response.read())

# (C) Hiển thị thống kê queue
print "\tMemory Used (bytes): " + str(payload["memory"])
print "\tConsumer Count: " + str(payload["consumers"])
print "\tMessages:"
print "\t\tUnack'd: " + str(payload["messages_unacknowledged"])
print "\t\tReady: " + str(payload["messages_ready"])
print "\t\tTotal: " + str(payload["messages"])
sys.exit(0)
```

Chú thích trong listing:
- **(B) Decode response** – Giải mã response.
- **(C) Display queue statistics** – Hiển thị thống kê queue.

Vì API server luôn trả thông tin về dưới dạng hash table JSON, trước hết bạn phải giải mã JSON **(B)** cho response. Việc này chuyển hash table được mã hóa JSON thành kiểu hash table thực sự mà ngôn ngữ lập trình của bạn hỗ trợ. Với Python, điều đó có nghĩa là response được chuyển thành một **dictionary** Python. Cái hay của việc response được mã hóa JSON là: một khi giải mã xong, bạn có thể truy cập các trường trong `resp_payload` giống hệt như với bất kỳ dictionary (hash table) nào khác. Ví dụ, lấy lượng bộ nhớ mà queue đang dùng chỉ đơn giản là truy cập **(C)** phần tử `memory` của dictionary `resp_payload`.³

(³ Chú thích chân trang: Trong dictionary `resp_payload` còn nhiều số liệu thống kê hơn nhiều so với những gì chúng tôi hiển thị. Bên trong còn có thông tin về bao nhiêu message là bền vững (*persistent*), bao nhiêu message đang chờ xác nhận, tốc độ trung bình message vào (*ingress*) và ra (*egress*) khỏi queue, và còn nhiều nữa. Chỉ cần thêm `print repr(resp_payload)` trước `sys.exit(0)` trong code của listing để xem tất cả các trường khác nhau.)

> 📝 **Giải thích thêm (không có trong sách):** Trong Listing 9.4, biến chứa dữ liệu đã giải mã có tên là `payload`, còn phần giải thích và chú thích chân trang của sách lại gọi nó là `resp_payload`. Đây là cùng một thứ (sự không nhất quán nằm ở bản gốc). Nếu bạn làm theo chú thích chân trang số 3 với code của Listing 9.4, hãy viết `print repr(payload)`.

Vậy khi chạy tiện ích thì trông nó thế nào?

```bash
$ python queue_stats.py localhost:55672 guest guest / test
'test' Queue Stats
-----------------
    Memory Used (bytes): 9104
    Consumer Count: 3
    Messages:
        Unack'd: 3
        Ready: 4
        Total: 7
```

> 📝 **Giải thích thêm (không có trong sách):** Hai dòng tiêu đề `'test' Queue Stats` và `-----------------` trong output không được in bởi các đoạn code trích trong Listing 9.1–9.4; chúng đến từ bản đầy đủ của script trong mã nguồn đi kèm sách. Nếu muốn có chúng, bạn có thể thêm `print "'%s' Queue Stats" % queue_name` và `print "-----------------"` ngay trước dòng in `Memory Used`. (Lưu ý `queue_name` lúc này đã bị escape; với tên đơn giản như `test` thì không khác gì.)

Từ output bạn có thể thấy queue `test` trong virtual host mặc định (`/`) đang tiêu tốn 9104 byte bộ nhớ, có ba consumer gắn vào nó, và chứa tổng cộng bảy message. Không chỉ biết queue chứa bảy message, bạn còn thấy được rằng ba trong số đó đã được giao cho consumer và đang chờ các consumer đó xác nhận (ack), còn bốn message kia đang chờ được giao cho consumer rảnh tiếp theo. Tất cả những thông tin này nằm trong tầm tay bạn chỉ với chưa đến 60 dòng code — và bạn có thể truy cập nó từ **bất cứ đâu** trong hạ tầng của mình!

Đó chính là sức mạnh của Rabbit management API. Nó cho phép bạn "soi" vào bên trong (*introspect*) và điều khiển lũ thỏ của mình từ bất cứ nơi nào có kết nối mạng. Nhưng nói nhiều về "điều khiển" RabbitMQ như vậy, mà đến giờ chúng ta mới chỉ **đọc** số liệu thống kê. Hãy thật sự thay đổi cấu hình server RabbitMQ của bạn nào.

> 💡 **Ví dụ bổ sung (không có trong sách):** *Bài toán:* Biến `queue_stats.py` thành một health check cho hệ thống giám sát (như Nagios): cảnh báo nếu số message Ready vượt ngưỡng. Phiên bản Python 3, dùng thư viện chuẩn:
>
> ```python
> # queue_check.py  (Python 3)
> import sys, json, base64, http.client, urllib.parse
>
> server, port = sys.argv[1].split(":")
> user, pwd, vhost, queue, max_ready = sys.argv[2:7]
>
> path = "/api/queues/%s/%s" % (urllib.parse.quote(vhost, safe=''),
>                               urllib.parse.quote(queue, safe=''))
> cred = base64.b64encode(("%s:%s" % (user, pwd)).encode()).decode()
>
> conn = http.client.HTTPConnection(server, int(port))
> conn.request("GET", path, headers={"Content-Type": "application/json",
>                                    "Authorization": "Basic " + cred})
> resp = conn.getresponse()
> if resp.status > 299:
>     print("UNKNOWN - API error %d" % resp.status)
>     sys.exit(3)
>
> stats = json.loads(resp.read())
> ready = stats["messages_ready"]
> if ready > int(max_ready):
>     print("CRITICAL - %d messages ready (> %s)" % (ready, max_ready))
>     sys.exit(2)
> print("OK - %d messages ready" % ready)
> sys.exit(0)
> ```
>
> Chạy: `python3 queue_check.py localhost:15672 monitor monitorMe / test 100`. Mã thoát 0/2/3 tương ứng OK/CRITICAL/UNKNOWN theo quy ước của Nagios. Chương sau của sách sẽ đi sâu hơn vào chủ đề giám sát.

> 📝 **Giải thích thêm (không có trong sách) — Viết Client gọi RabbitMQ REST API bằng Python 3 hiện đại (`requests` / `httpx` và `argparse`):**
>
> Các đoạn script trong sách (và các chương tiếp theo) được viết vào năm 2012 trên **Python 2**, sử dụng các thư viện mức thấp như `httplib` / `urllib2` (phải tự ghép chuỗi `username:password` và mã hóa `base64` thủ công, tự đọc raw stream rồi gọi `json.loads()`) cùng với `sys.argv` hoặc `optparse` (đã ngừng phát triển từ Python 3.2).
>
> Khi làm việc với RabbitMQ 3.8–4.0 trên **Python 3.10+**, các kỹ sư hệ thống hiện đại sử dụng **`requests`** (hoặc **`httpx`** nếu cần bất đồng bộ `async/await` và HTTP/2) kết hợp với **`argparse`**:
>
> | Thao tác | Python 2 trong sách (`httplib` / `urllib2` / `optparse`) | Python 3 hiện đại (`requests` / `httpx` + `argparse`) |
> |---|---|---|
> | **Phân tích tham số dòng lệnh** | Cắt tay `sys.argv[1].split(":")` hoặc dùng `optparse.OptionParser` | Dùng `argparse.ArgumentParser()` (tự động kiểm tra kiểu dữ liệu, giá trị mặc định và sinh `--help`). |
> | **HTTP Basic Auth** | Tự nối `"%s:%s" % (user, pass)` rồi gọi `base64.b64encode()` và gán header `"Authorization": "Basic ..."` | Chỉ cần truyền **`auth=(username, password)`** — thư viện tự động mã hóa Base64 chuẩn xác! |
> | **Gửi & Giải mã JSON** | `json.dumps(data)` + đặt tay header `Content-Type: application/json`; nhận về phải `json.loads(response.read())` | Truyền **`json=data`** khi `PUT`/`POST` và gọi **`resp.json()`** khi đọc kết quả. |
> | **Kiểm tra lỗi HTTP & Timeout** | Không có timeout mặc định (dễ treo vĩnh viễn!), tự kiểm tra `if response.status > 299:` | Luôn đặt **`timeout=5.0`** và gọi `resp.raise_for_status()`. |
> | **Tái sử dụng kết nối TCP** | Mở kết nối mới mỗi lần chạy | Dùng `requests.Session()` hoặc `httpx.Client()` để giữ HTTP Keep-Alive (Connection Pooling) khi gọi nhiều endpoint liên tiếp. |

> 💡 **Ví dụ bổ sung (không có trong sách):** Phiên bản `queue_stats_modern.py` viết lại hoàn chỉnh cho **Python 3 + RabbitMQ 3.x/4.0 (cổng `15672`)** sử dụng `argparse` và `requests`:
>
> ```python
> #!/usr/bin/env python3
> """queue_stats_modern.py — Truy vấn thống kê queue trên RabbitMQ 3.8–4.0."""
> import sys
> import urllib.parse
> import argparse
> import requests
>
> def main() -> None:
>     parser = argparse.ArgumentParser(description="Xem thống kê queue qua RabbitMQ REST API")
>     parser.add_argument("--url", default="http://localhost:15672", help="URL gốc của Management API (mặc định: http://localhost:15672)")
>     parser.add_argument("-u", "--username", default="monitor", help="Username xác thực")
>     parser.add_argument("-p", "--password", default="monitorMe", help="Password xác thực")
>     parser.add_argument("-V", "--vhost", default="/", help="Tên Virtual Host (mặc định: /)")
>     parser.add_argument("queue", help="Tên queue cần kiểm tra")
>     args = parser.parse_args()
>
>     vhost_enc = urllib.parse.quote(args.vhost, safe="")
>     queue_enc = urllib.parse.quote(args.queue, safe="")
>     endpoint = f"{args.url.rstrip('/')}/api/queues/{vhost_enc}/{queue_enc}"
>
>     try:
>         resp = requests.get(
>             endpoint,
>             auth=(args.username, args.password),
>             timeout=5.0,
>         )
>         resp.raise_for_status()
>     except requests.HTTPError as err:
>         print(f"Lỗi API ({err.response.status_code}): {err.response.text}", file=sys.stderr)
>         sys.exit(2)
>     except requests.RequestException as err:
>         print(f"Lỗi kết nối tới {endpoint}: {err}", file=sys.stderr)
>         sys.exit(3)
>
>     payload = resp.json()
>     q_type = payload.get("type", "classic")
>     print(f"'{args.queue}' Queue Stats (type={q_type}, node={payload.get('node')})")
>     print("-" * 50)
>     print(f"\tMemory Used (bytes): {payload.get('memory', 0):,}")
>     print(f"\tConsumer Count:      {payload.get('consumers', 0)}")
>     print("\tMessages:")
>     print(f"\t\tUnack'd: {payload.get('messages_unacknowledged', 0)}")
>     print(f"\t\tReady:   {payload.get('messages_ready', 0)}")
>     print(f"\t\tTotal:   {payload.get('messages', 0)}")
>
> if __name__ == "__main__":
>     main()
> ```
>
> Chạy thử trên RabbitMQ 3.x/4.0:
> ```bash
> $ python3 queue_stats_modern.py --url http://localhost:15672 -u monitor -p monitorMe -V / test
> ```
> Nếu dự án của bạn dùng **`httpx`** (thư viện HTTP thế hệ mới cho Python 3 có API tương thích gần như 100% với `requests`), bạn chỉ cần đổi `import requests` thành `import httpx` và `requests.get(...)` thành `httpx.get(...)`!

---

## 9.4 Tự động hóa việc cấp phát vhost và user (Automating vhost and user provisioning)

Trước khi có management API, một trong những rắc rối lớn nhất khi triển khai RabbitMQ là tự động hóa việc tạo các virtual host và user mà ứng dụng cần. Thường thì khi triển khai ứng dụng bằng các công cụ triển khai tự động như Chef hay Puppet, "công thức" (*recipe*) triển khai ứng dụng được chạy trên một server **khác** với server RabbitMQ mà ứng dụng cần giao tiếp.

Khi tất cả những gì bạn có chỉ là `rabbitmqctl` để tạo user và virtual host, đó là một vấn đề: nó có nghĩa là Erlang và `rabbitmqctl` phải được cài trên **mọi** server ứng dụng, chỉ với mục đích duy nhất là tạo user và vhost trên server RabbitMQ trung tâm. Chẳng phải sẽ tuyệt sao nếu có cách tạo user và vhost từ chính các server chạy ứng dụng mà không phải cài Erlang lên từng máy (và đồng bộ tất cả về cùng một Erlang cookie)? Vì chúng ta đang sống trong thời đại của Rabbit management API, đừng lo lắng nữa, hỡi lập trình viên Rabbit gan dạ! Bạn có thể viết một script dòng lệnh cho phép tạo, xóa, xem và liệt kê user bằng management API (mở rộng script để cấp phát vhost cũng sẽ rất đơn giản). Không cần Erlang hay cookie gì cả!

> 📝 **Giải thích thêm (không có trong sách):** "Cấp phát" (*provisioning*) ở đây nghĩa là chuẩn bị sẵn tài nguyên mà ứng dụng cần trước khi nó chạy: tạo vhost riêng cho ứng dụng, tạo user/mật khẩu cho ứng dụng, gán quyền cho user trên vhost đó…

Vì chúng ta đã đi qua cách kết nối, xác thực và gửi một API request cơ bản, ở đây ta sẽ tập trung vào điểm khác biệt của một request **tạo** hoặc **xóa** user (hoặc vhost). Hãy bắt đầu bằng việc xem toàn bộ script `user_manager.py`, như trong listing sau.

**Listing 9.5 – user_manager.py — trình quản lý user RabbitMQ**

```python
import sys, json, httplib, base64

# (B) Gán các tham số
if len(sys.argv) < 5:
    print "USAGE: user_manager.py server_name:port " + \
          "auth_user auth_pass",
    print "ACTION [PARAMS...]"
    sys.exit(1)

server, port = sys.argv[1].split(":")
username = sys.argv[2]
password = sys.argv[3]
action = sys.argv[4]

if len(sys.argv) > 5:
    res_params = sys.argv[5:]
else:
    res_params = []

# (C) Dựng đường dẫn API
base_path = "/api/users"

if action == "list":
    path = base_path
    method = "GET"
if action == "create":
    path = base_path + "/" + res_params[0]
    method = "PUT"
if action == "delete":
    path = base_path + "/" + res_params[0]
    method = "DELETE"
if action == "show":
    path = base_path
    method = "GET"

# (D) Dựng các tham số JSON
json_args = ""
if action == "create":
    json_args = {"password" : res_params[1],
                 "administrator" : json.loads(res_params[2])}
    json_args = json.dumps(json_args)

# (E) Gửi API request
conn = httplib.HTTPConnection(server, port)
credentials = base64.b64encode("%s:%s" % (username, password))
conn.request(method, path, json_args,
             {"Content-Type" : "application/json",
              "Authorization" : "Basic " + credentials})
response = conn.getresponse()
if response.status > 299:
    print "Error executing API call (%d): %s" % (response.status,
                                                 response.read())
    sys.exit(2)

# (F) Phân tích và hiển thị response
resp_payload = response.read()
if action in ["list", "show"]:
    resp_payload = json.loads(resp_payload)

    # (G) Xử lý kết quả của "list"
    if action == "list":
        print "Count: %d" % len(resp_payload)
        for user in resp_payload:
            print "User: %(name)s" % user
            print "\tPassword: %(password_hash)s" % user
            print "\tAdministrator: %(administrator)s\n" % user

    # (H) Xử lý kết quả của "show"
    if action == "show":
        print "User: %(name)s" % resp_payload
        print "\tPassword: %(password_hash)s" % resp_payload
        print "\tAdministrator: %(administrator)s\n" % resp_payload

# (I) Request create và delete không có kết quả trả về
else:
    print "Completed request!"

sys.exit(0)
```

Chú thích trong listing:
- **(B) Assign arguments** – Gán các tham số.
- **(C) Build API path** – Dựng đường dẫn API.
- **(D) Build JSON arguments** – Dựng các tham số JSON.
- **(E) Issue API request** – Gửi API request.
- **(F) Parse and display response** – Phân tích và hiển thị response.
- **(G) Process list results** – Xử lý kết quả của hành động `list`.
- **(H) Process show results** – Xử lý kết quả của hành động `show`.
- **(I) Create and delete requests have no result** – Request create và delete không có kết quả trả về.

> 📝 **Giải thích thêm (không có trong sách):** Hãy để ý: trong Listing 9.5 (chép nguyên văn từ sách), nhánh `action == "show"` đặt `path = base_path`, tức là gửi `GET /api/users` — đó là đường dẫn **liệt kê tất cả user**, sẽ trả về một *mảng* và làm phần in kết quả của `show` bị lỗi. Ngay sau đây, khi giải thích lại đoạn code này, sách viết đúng là `path = base_path + "/" + res_params[0]`. Hãy dùng phiên bản đúng đó khi chạy script. Ngoài ra, dấu phẩy ở cuối dòng `print "... auth_pass",` là cú pháp Python 2 để **không xuống dòng**, nhờ đó hai lệnh `print` tạo thành một dòng hướng dẫn sử dụng duy nhất.

Cũng như script thống kê queue, bạn kiểm tra **(B)** các tham số dòng lệnh và gán chúng vào những biến dễ nhớ hơn. Điểm khác biệt duy nhất cho đến đây là số lượng tham số đã tăng lên. Thay vì các tham số cuối chỉ ra vhost và tên queue, giờ chúng đại diện cho **hành động** cần thực hiện (`create`, `delete`, `list`, `show`) và **các tham số cho hành động đó**. Ví dụ:

```bash
$ python user_manager.py localhost:55672 guest guest \
    create myuser password true
```

Lệnh này sẽ kết nối tới API server trên localhost bằng username và password `guest`, và sẽ tạo một user mới tên `myuser` có mật khẩu là `password` và là quản trị viên (`true`). Nếu bạn đổi `true` thành `false`, user sẽ không được tạo với quyền quản trị viên.

Tiếp theo, bạn bắt đầu dựng **(C)** đường dẫn của request. Đường dẫn gốc cho mọi lời gọi API về user là `/api/users`. Vào đường dẫn này, bạn nối thêm username (nằm ở chỉ số 0 của mảng `res_params`) mà bạn muốn thực hiện hành động lên:

```python
base_path = "/api/users"
if action == "list":
    path = base_path
    method = "GET"
if action == "create":
    path = base_path + "/" + res_params[0]
    method = "PUT"
if action == "delete":
    path = base_path + "/" + res_params[0]
    method = "DELETE"
if action == "show":
    path = base_path + "/" + res_params[0]
    method = "GET"
```

Đây là chỗ bạn thấy rõ cách API diễn giải các động từ HTTP khác nhau trên **cùng một** đường dẫn request:

- Nếu bạn đặt phương thức là `PUT`, API server sẽ **tạo** user được chỉ định bởi `/api/users/<username>`.
- Nhưng đặt phương thức là `DELETE` cho cùng đường dẫn đó sẽ khiến user bị **xóa**.
- Tương tự, `GET` sẽ được API server hiểu là lệnh **trả về chi tiết** về user dưới dạng một hash table JSON.

Kẻ "lập dị" duy nhất là cách bạn **liệt kê tất cả user**. Thay vì gửi `GET` tới `/api/users/<username>` (vốn chỉ định một user cụ thể), bạn gửi request `GET` tới `/api/users`. Vì đường dẫn request đó không chỉ định user cụ thể nào, API server sẽ trả về danh sách tất cả user trong server RabbitMQ (server trả về một **danh sách các hash table**, trong đó mỗi hash table chứa đầy đủ chi tiết của một user).

> 💡 **Ví dụ bổ sung (không có trong sách):** Tổng hợp ánh xạ "hành động → động từ + đường dẫn" của `user_manager.py` (ví dụ với user `alice`):
>
> | Hành động | Động từ HTTP | Đường dẫn | Body request | Response |
> |---|---|---|---|---|
> | `list` | `GET` | `/api/users` | rỗng | Mảng JSON các hash table user |
> | `show alice` | `GET` | `/api/users/alice` | rỗng | Một hash table JSON |
> | `create alice pw true` | `PUT` | `/api/users/alice` | `{"password": "pw", "administrator": true}` | Rỗng (`204`) |
> | `delete alice` | `DELETE` | `/api/users/alice` | rỗng | Rỗng (`204`) |
>
> Đây là mẫu chung của REST: **đường dẫn tập hợp** (`/api/users`) dùng để liệt kê; **đường dẫn phần tử** (`/api/users/alice`) dùng để xem/tạo/xóa một phần tử.

Sau khi dựng xong đường dẫn, bạn cần xác định **(D)** xem request đang được tạo có phải để **tạo user** hay không. Nếu đúng, bạn cần dựng một hash table được mã hóa JSON chứa mật khẩu và trạng thái quản trị viên của user mới:

```python
json_args = ""
if action == "create":
    json_args = {"password" : res_params[1],
                 "administrator" : json.loads(res_params[2])}
    json_args = json.dumps(json_args)
```

Hash table để tạo user cần hai trường: `password` và `administrator`. `password` chứa mật khẩu mong muốn ở dạng văn bản thô (*plaintext*), còn `administrator` là một giá trị Boolean, đặt là `true` hoặc `false`. Khi request đã được dựng xong, bạn kết nối và gửi **(E)** request y như đã làm với script thống kê queue. Điểm khác biệt duy nhất là giờ bạn chỉ định **body** cho request khi hành động là tạo user.

> 📝 **Giải thích thêm (không có trong sách):** Tại sao lại dùng `json.loads(res_params[2])`? Vì tham số dòng lệnh luôn là **chuỗi**: `"true"` hoặc `"false"`. Gọi `json.loads("true")` sẽ ra giá trị Boolean Python `True` (và `json.loads("false")` ra `False`). Sau đó `json.dumps(...)` chuyển cả dictionary thành chuỗi JSON, trong đó `True` được viết thành `true` đúng chuẩn JSON. Nếu bạn gõ `True` (viết hoa) trên dòng lệnh, `json.loads` sẽ báo lỗi vì đó không phải JSON hợp lệ.
>
> ```python
> >>> import json
> >>> json.dumps({"password": "pw", "administrator": json.loads("true")})
> '{"password": "pw", "administrator": true}'
> ```
>
> (Ở RabbitMQ 3.x, trường `administrator` được thay bằng trường `tags`, ví dụ `{"password": "pw", "tags": "administrator"}`.)

Khi request đã được gửi và bạn đã kiểm tra rằng server không trả về lỗi, bạn đọc **(F)** response và bắt tay vào xử lý. Nếu hành động là tạo hoặc xóa user **(I)**, sẽ không có kết quả nào, nên bạn chỉ in ra một xác nhận rằng request đã thành công cho người dùng. Nếu hành động được yêu cầu là liệt kê hoặc xem user, bạn giải mã JSON cho response. Sau khi response được giải mã, bạn có một cấu trúc dữ liệu Python gốc (*native*) để làm việc. Nếu người dùng yêu cầu danh sách user, response là một **mảng các hash table**. Vì vậy bạn cần lặp **(G)** qua danh sách user và in ra `name`, `password_hash` và trạng thái `administrator` của từng user:⁴

```python
if action == "list":
    print "Count: %d" % len(resp_payload)
    for user in resp_payload:
        print "User: %(name)s" % user
        print "\tPassword: %(password_hash)s" % user
        print "\tAdministrator: %(administrator)s\n" % user
```

(⁴ Chú thích chân trang: Có thể bạn để ý chúng tôi đang dùng khả năng định dạng chuỗi (*string formatting*) của Python. Ví dụ, `"User: %(name)s" % user` bảo Python tạo một chuỗi mới, thay giá trị của `user["name"]` vào vị trí của `%(name)s` trong chuỗi. Bạn cũng có thể dùng cú pháp `"User: " + user["name"]` và có cùng kết quả.)

Nếu hành động là xem một user cụ thể, response sẽ **không** phải là mảng các hash table. Thay vào đó, bản thân response sẽ là **một** hash table duy nhất, giống như những gì bạn đã thấy với script thống kê queue, chỉ khác là lần này hash table **(H)** chứa `name`, `password_hash` và trạng thái `administrator` của user:

```python
if action == "show":
    print "User: %(name)s" % resp_payload
    print "\tPassword: %(password_hash)s" % resp_payload
    print "\tAdministrator: %(administrator)s\n" % resp_payload
```

> 💡 **Ví dụ bổ sung (không có trong sách):** Hình dạng dữ liệu mà hai nhánh trên nhận được (RabbitMQ 2.x):
>
> ```javascript
> // GET /api/users  → mảng (nhánh "list" lặp qua từng phần tử)
> [
>   {"name": "guest",   "password_hash": "...", "administrator": true},
>   {"name": "monitor", "password_hash": "...", "administrator": true}
> ]
>
> // GET /api/users/monitor  → một đối tượng (nhánh "show" dùng trực tiếp)
> {"name": "monitor", "password_hash": "...", "administrator": true}
> ```
>
> Với dữ liệu trên, nhánh `list` sẽ in `Count: 2` rồi hai khối thông tin user. Đây cũng là lý do lỗi `path = base_path` trong nhánh `show` của Listing 9.5 gây hỏng: `resp_payload` khi đó là một *mảng*, và `"%(name)s" % resp_payload` sẽ ném lỗi `TypeError`.

Giờ bạn đã có một tiện ích đầy đủ tính năng, có thể tạo, xóa, xem và liệt kê user trên bất kỳ server RabbitMQ nào đã cài Management plugin. Hãy xem việc tạo rồi xem một user bằng tiện ích này trông như thế nào:

```bash
$ python user_manager.py localhost:55672 guest guest \
    create newuser newpass true
Completed request!
$ python user_manager.py localhost:55672 guest guest \
    show newuser
User: newuser
    Password: o/ZEH9Z86FNUtzu2MzNlmDSTDFE=
    Administrator: True
```

> 📝 **Giải thích thêm (không có trong sách):** Dòng `Password:` hiển thị **giá trị băm** (*hash*) của mật khẩu (đã mã hóa Base64), không phải mật khẩu `newpass` gốc. RabbitMQ không bao giờ lưu hay trả về mật khẩu dạng thô — đó là lý do trường có tên `password_hash`.
>
> Lưu ý bảo mật trong RabbitMQ hiện đại (3.6+ đến 4.0): Ở RabbitMQ 2.x, `password_hash` dùng thuật toán băm **MD5** có kèm 4 byte salt. Từ RabbitMQ 3.6 trở đi, thuật toán mặc định đã đổi sang **SHA-256** (`rabbit_password_hashing_sha256`, hoặc bạn có thể chọn `rabbit_password_hashing_sha512`). Khi tạo user qua REST API (`PUT /api/users/<name>`), bạn có thể truyền hoặc `"password": "plaintext_password"` (broker sẽ tự băm bằng SHA-256), hoặc truyền trực tiếp `"password_hash": "<base64_hash>"` kèm `"hashing_algorithm": "rabbit_password_hashing_sha256"` để **không bao giờ phải lưu mật khẩu dạng rõ trong các script CI/CD hay GitOps**!

Dù có lẽ bạn sẽ muốn thêm phần kiểm tra tham số chặt chẽ hơn cho script khi dùng trong production, sự thật là bạn có thể dùng nó **nguyên trạng** để quản lý việc tạo user từ bất kỳ hệ thống nào. Thực tế, các tác giả cũng dùng một script tương tự để tự động hóa việc tạo user RabbitMQ khi triển khai ứng dụng.

Qua việc xây dựng script này, bạn đã học được không chỉ cách xem một đối tượng đơn lẻ qua management API, mà còn cách xem danh sách đối tượng, cũng như cách tạo và xóa các đối tượng đó. Bạn có thể áp dụng những khái niệm này để làm việc với bất kỳ loại đối tượng/tài nguyên (*item/resource*) nào có trong API (user, queue, exchange, connection, permission, v.v.). Trong mã nguồn đi kèm chương này, chúng tôi có kèm một script tên `user_vhost_manager.py`. Nó mở rộng script quản lý user này để quản lý việc tạo vhost, đồng thời cho phép thao tác quyền user/vhost.

> 💡 **Ví dụ bổ sung (không có trong sách):** *Bài toán:* Tự viết phần mở rộng tương tự `user_vhost_manager.py` — thêm hai hành động `create_vhost <vhost>` và `grant <vhost> <user>` (cấp toàn quyền). Chỉ cần thêm vào phần dựng đường dẫn và phần dựng JSON của Listing 9.5:
>
> ```python
> if action == "create_vhost":
>     path = "/api/vhosts/" + urllib.quote(res_params[0], safe='')
>     method = "PUT"
> if action == "grant":
>     path = "/api/permissions/%s/%s" % (urllib.quote(res_params[0], safe=''),
>                                        urllib.quote(res_params[1], safe=''))
>     method = "PUT"
>
> # ... trong phần dựng json_args:
> if action == "grant":
>     json_args = json.dumps({"configure": ".*", "write": ".*", "read": ".*"})
> ```
>
> (Nhớ thêm `urllib` vào dòng `import`.) Khi đó, một bước trong "công thức" Chef/Puppet triển khai ứng dụng `shop` chỉ cần ba lệnh, chạy từ bất kỳ máy nào không có Erlang:
>
> ```bash
> python user_manager.py rabbit01:55672 admin secret create_vhost shop
> python user_manager.py rabbit01:55672 admin secret create shop_app s3cret false
> python user_manager.py rabbit01:55672 admin secret grant shop shop_app
> ```

> 💡 **Ví dụ bổ sung (không có trong sách) — `user_vhost_manager_modern.py` trên Python 3 (`requests` + `argparse`) cho RabbitMQ 3.8–4.0:**
>
> Dưới đây là script tự động hóa cấp phát vhost, user (dùng `tags` thay cho `administrator`) và phân quyền trên RabbitMQ 3.8–4.0 (cổng `15672`) viết bằng Python 3 hiện đại:
>
> ```python
> #!/usr/bin/env python3
> """user_vhost_manager_modern.py — Quản lý user & vhost trên RabbitMQ 3.8–4.0."""
> import argparse
> import urllib.parse
> import requests
>
> def main() -> None:
>     parser = argparse.ArgumentParser(description="Quản lý User & Vhost qua RabbitMQ REST API")
>     parser.add_argument("--url", default="http://localhost:15672", help="Base URL (mặc định: http://localhost:15672)")
>     parser.add_argument("-u", "--auth-user", default="guest", help="Admin username")
>     parser.add_argument("-p", "--auth-pass", default="guest", help="Admin password")
>
>     sub = parser.add_subparsers(dest="action", required=True)
>     sub.add_parser("list-users")
>
>     p_show = sub.add_parser("show-user")
>     p_show.add_argument("username")
>
>     p_create = sub.add_parser("create-user")
>     p_create.add_argument("username")
>     p_create.add_argument("password")
>     p_create.add_argument("--tags", default="", help="Danh sách tag cách nhau bởi dấu phẩy (vd: monitoring,management)")
>
>     p_del = sub.add_parser("delete-user")
>     p_del.add_argument("username")
>
>     p_vhost = sub.add_parser("provision-vhost", help="Tạo vhost (mặc định Quorum Queue) và gán quyền cho user")
>     p_vhost.add_argument("vhost")
>     p_vhost.add_argument("username")
>
>     args = parser.parse_args()
>     base = args.url.rstrip("/")
>
>     with requests.Session() as session:
>         session.auth = (args.auth_user, args.auth_pass)
>         session.headers.update({"Content-Type": "application/json"})
>
>         if args.action == "list-users":
>             resp = session.get(f"{base}/api/users", timeout=5.0)
>             resp.raise_for_status()
>             users = resp.json()
>             print(f"Count: {len(users)}")
>             for u in users:
>                 print(f"User: {u['name']} | Tags: {u.get('tags')} | Hash: {u.get('password_hash')}")
>
>         elif args.action == "show-user":
>             u_enc = urllib.parse.quote(args.username, safe="")
>             resp = session.get(f"{base}/api/users/{u_enc}", timeout=5.0)
>             resp.raise_for_status()
>             u = resp.json()
>             print(f"User: {u['name']}\n\tTags: {u.get('tags')}\n\tPassword Hash: {u.get('password_hash')}")
>
>         elif args.action == "create-user":
>             u_enc = urllib.parse.quote(args.username, safe="")
>             resp = session.put(
>                 f"{base}/api/users/{u_enc}",
>                 json={"password": args.password, "tags": args.tags},
>                 timeout=5.0,
>             )
>             resp.raise_for_status()
>             print(f"Đã tạo/cập nhật user '{args.username}' (tags='{args.tags}').")
>
>         elif args.action == "delete-user":
>             u_enc = urllib.parse.quote(args.username, safe="")
>             resp = session.delete(f"{base}/api/users/{u_enc}", timeout=5.0)
>             resp.raise_for_status()
>             print(f"Đã xóa user '{args.username}'.")
>
>         elif args.action == "provision-vhost":
>             v_enc = urllib.parse.quote(args.vhost, safe="")
>             u_enc = urllib.parse.quote(args.username, safe="")
>             # Ở RabbitMQ 3.11+ / 4.0, có thể đặt default_queue_type cho vhost ngay khi tạo!
>             r1 = session.put(
>                 f"{base}/api/vhosts/{v_enc}",
>                 json={"description": f"Vhost for {args.vhost}", "default_queue_type": "quorum"},
>                 timeout=5.0,
>             )
>             r1.raise_for_status()
>             r2 = session.put(
>                 f"{base}/api/permissions/{v_enc}/{u_enc}",
>                 json={"configure": ".*", "write": ".*", "read": ".*"},
>                 timeout=5.0,
>             )
>             r2.raise_for_status()
>             print(f"Đã tạo vhost '{args.vhost}' (default_queue_type=quorum) và cấp quyền cho '{args.username}'.")
>
> if __name__ == "__main__":
>     main()
> ```

> 📝 **Giải thích thêm (không có trong sách) — Công cụ `rabbitmqadmin`: Từ Script Python đến Binary viết bằng Rust ở RabbitMQ 4.0:**
>
> Trong thực tế vận hành, bạn hiếm khi phải tự viết từ đầu một script như `user_manager.py` chỉ để gọi REST API từ dòng lệnh hay từ Ansible/Shell script. Ngay khi bật plugin `rabbitmq_management`, RabbitMQ đã cung cấp sẵn công cụ CLI chính chủ mang tên **`rabbitmqadmin`** (bản chất là một HTTP client bọc ngoài REST API):
>
> - **Trên RabbitMQ 3.8–3.13**: `rabbitmqadmin` là một script Python 3 nằm sẵn tại địa chỉ `http://localhost:15672/cli/rabbitmqadmin`. Bạn chỉ cần tải về (`curl -sO http://localhost:15672/cli/rabbitmqadmin && chmod +x rabbitmqadmin`) là có thể chạy từ bất kỳ máy nào có Python 3 mà không cần Erlang.
> - **Bước ngoặt trên RabbitMQ 4.0 (`rabbitmqadmin` v2 viết bằng Rust)**: Đội ngũ RabbitMQ đã viết lại hoàn toàn **`rabbitmqadmin` v2 bằng ngôn ngữ Rust** (cung cấp dưới dạng file thực thi nhị phân độc lập — *standalone static binary* cho Linux, macOS, Windows). Phiên bản v2 này khởi động tức thì, tốc độ thực thi cực nhanh, cấu trúc lệnh trực quan hơn, và **hoàn toàn không cần cài đặt Python hay bất kỳ runtime nào trên server**!
>
> | Tác vụ | Tự viết script Python (trong sách) | Dùng `rabbitmqadmin` v1 (RabbitMQ 3.x) | Dùng `rabbitmqadmin` v2 (Rust — RabbitMQ 4.0) |
> |---|---|---|---|
> | **Phụ thuộc cần cài đặt** | Python + thư viện HTTP | Python 3 | **Không cần gì cả** (1 file binary Rust duy nhất) |
> | **Liệt kê user** | `python user_manager.py host:55672 u p list` | `rabbitmqadmin list users` | `rabbitmqadmin users list` |
> | **Tạo vhost mới** | Tự viết hàm gọi `PUT /api/vhosts/<name>` | `rabbitmqadmin declare vhost name=shop` | `rabbitmqadmin vhosts declare --name shop --default-queue-type quorum` |
> | **Tạo user mới** | `python user_manager.py ... create u p true` | `rabbitmqadmin declare user name=u password=p tags=monitoring` | `rabbitmqadmin users declare --name u --password p --tags monitoring` |
> | **Xuất/Nhập toàn bộ cấu hình** | Tự gọi `/api/definitions` | `rabbitmqadmin export backup.json` | `rabbitmqadmin definitions export --file backup.json` |
> | **Rebalance Queue Leaders** | Tự gọi `POST /api/rebalance/queues` | Phải dùng `curl` | `rabbitmqadmin rebalance queues` |

---

## 9.5 Tóm tắt (Summary)

Trước khi bắt đầu chương này, bạn bị giới hạn ở việc quản lý các server Rabbit **bằng tay**, hoặc qua `rabbitmqctl`, hoặc qua giao diện web quản lý. Bạn hoàn toàn không thể viết các script hay tiện ích tự động tích hợp với Rabbit để giúp quản lý cấu hình hoặc giám sát trạng thái bên trong của nó.

Nhưng nhờ học được cách Rabbit management API hoạt động, giờ bạn có thể xây dựng những công cụ cho phép giám sát trạng thái queue và quản lý user theo nhu cầu riêng của mình. Bạn không còn phải "phó mặc" cho việc quản lý server RabbitMQ thủ công nữa — giờ bạn có thể viết các tiện ích dùng management API để tự động hóa việc cấp phát Rabbit, sao cho phù hợp với quy trình riêng của tổ chức bạn. Với management API trong tay, giới hạn duy nhất cho những công cụ quản lý Rabbit mà bạn có thể xây dựng chính là trí tưởng tượng của bạn.

Một trong những lĩnh vực mà API mở ra là **giám sát tự động sức khỏe của RabbitMQ**. Giờ bạn đã hiểu cách xây dựng công cụ bằng API, bạn đã sẵn sàng dùng những kỹ năng đó sâu hơn để nâng cấp việc giám sát hạ tầng Rabbit của mình, và đảm bảo chúng luôn chạy trong tình trạng tốt nhất.

> 📝 **Giải thích thêm (không có trong sách):** Tóm tắt nhanh các ý chính của chương:
> 1. Management plugin đi kèm một REST API qua HTTP — không cần Erlang/cookie ở phía client.
> 2. Tên đối tượng nằm trong URL; động từ HTTP quyết định hành động: `GET` (xem/liệt kê), `PUT`/`POST` (tạo), `DELETE` (xóa).
> 3. Tham số cho `PUT`/`POST` gửi dưới dạng hash table JSON trong body, kèm header `Content-Type: application/json`.
> 4. Xác thực bằng HTTP Basic với user RabbitMQ thường; ở bản 2.x user phải có cờ admin (`set_admin`).
> 5. Nhớ escape tên vhost/queue (`/` → `%2F`).
> 6. Response: body rỗng cho tạo/xóa; hash table (hoặc mảng hash table) JSON cho xem/liệt kê; mã > 299 là lỗi.

---

## Bảng thuật ngữ của chương

| Thuật ngữ (English) | Nghĩa tiếng Việt | Giải thích ngắn |
|---|---|---|
| REST (Representational State Transfer) | Chuyển giao trạng thái biểu diễn | Quy ước thiết kế API HTTP: tên đối tượng nằm trong URL, hành động thể hiện bằng động từ HTTP. |
| RESTful API | API theo phong cách REST | API tuân thủ quy ước REST. |
| Management plugin | Plugin quản lý | Plugin của RabbitMQ cung cấp giao diện web và REST API quản lý. |
| Management API | API quản lý | REST API do Management plugin cung cấp, dưới đường dẫn `/api`. |
| HTTP verb / method | Động từ / phương thức HTTP | `GET`, `POST`, `PUT`, `DELETE` — cho biết hành động trên đối tượng. |
| Endpoint / request path | Điểm cuối / đường dẫn request | Phần URL chỉ tới tài nguyên, ví dụ `/api/queues/<vhost>/<queue>`. |
| JSON (JavaScript Object Notation) | Ký pháp đối tượng JavaScript | Định dạng dữ liệu dạng văn bản dùng hash table, mảng, chuỗi, số. |
| Hash table / dictionary | Bảng băm / từ điển | Cấu trúc khóa–giá trị; JSON object được giải mã thành dictionary trong Python. |
| Header | Phần đầu (của HTTP) | Các dòng siêu dữ liệu đi kèm request/response, như `Content-Type`, `Authorization`. |
| Body | Phần thân | Dữ liệu chính của request/response. |
| Response code / status code | Mã trạng thái HTTP | Số cho biết kết quả: 2xx thành công, 4xx lỗi phía client, 5xx lỗi phía server. |
| HTTP Basic authentication | Xác thực cơ bản HTTP | Gửi `username:password` mã hóa Base64 trong header `Authorization: Basic ...`. |
| Base64 | Mã hóa Base64 | Cách biểu diễn dữ liệu bất kỳ chỉ bằng ký tự ASCII; không phải mã hóa bảo mật. |
| Credentials | Thông tin xác thực | Username và password dùng để đăng nhập. |
| Permissions (configure / write / read) | Quyền (cấu hình / ghi / đọc) | Ba loại quyền của user trên một vhost, mỗi loại là một regex. |
| Admin flag (`set_admin`) | Cờ quản trị | Ở RabbitMQ 2.x, user phải có cờ này mới được dùng API. |
| Escape / URL-encode (quote) | Mã hóa ký tự URL | Thay ký tự đặc biệt bằng `%XX`, ví dụ `/` → `%2F`. |
| Scrape | Cào dữ liệu | Phân tích output dạng văn bản để trích thông tin — cách làm dễ vỡ. |
| Brittle | Mong manh, dễ vỡ | Giải pháp dễ hỏng khi có thay đổi nhỏ. |
| Provisioning | Cấp phát (tài nguyên) | Tạo sẵn vhost, user, quyền… cho ứng dụng trước khi nó chạy. |
| Deployment | Triển khai | Đưa ứng dụng lên server để chạy. |
| Chef / Puppet / CFEngine | (tên công cụ) | Công cụ quản lý cấu hình và triển khai tự động. |
| Health check | Kiểm tra sức khỏe | Script/quy trình kiểm tra định kỳ hệ thống có hoạt động bình thường. |
| Introspect | Soi vào bên trong | Xem trạng thái nội bộ của hệ thống đang chạy. |
| Erlang cookie | Cookie Erlang | Chuỗi bí mật mà các node Erlang phải dùng chung để giao tiếp với nhau. |
| Unacknowledged (Unack'd) messages | Message chưa được xác nhận | Message đã giao cho consumer nhưng consumer chưa gửi ack. |
| Ready messages | Message sẵn sàng | Message đang chờ được giao cho consumer. |
| Ingress / egress rate | Tốc độ vào / ra | Tốc độ message đi vào và đi ra khỏi queue. |
| Password hash | Giá trị băm của mật khẩu | Dạng mật khẩu đã băm mà RabbitMQ lưu, thay vì mật khẩu thô. |
| Plaintext | Văn bản thô | Dữ liệu chưa mã hóa/băm. |
| cURL | (tên công cụ) | Chương trình dòng lệnh để gửi/nhận HTTP request. |
