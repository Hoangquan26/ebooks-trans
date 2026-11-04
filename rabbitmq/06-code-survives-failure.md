# Chương 6: Viết code sống sót qua sự cố (Writing code that survives failure)

## Mục lục chương

- [Chương này đề cập đến những gì](#chương-này-đề-cập-đến-những-gì)
- [Mở đầu](#mở-đầu)
- [6.1 Cân bằng tải cho các "chú thỏ" RabbitMQ (Load balancing your Rabbits)](#61-cân-bằng-tải-cho-các-chú-thỏ-rabbitmq-load-balancing-your-rabbits)
  - [6.1.1 Cài đặt HAProxy (Installing HAProxy)](#611-cài-đặt-haproxy-installing-haproxy)
  - [6.1.2 Cấu hình HAProxy (Configuring HAProxy)](#612-cấu-hình-haproxy-configuring-haproxy)
- [6.2 Mất kết nối và client gặp sự cố khi chuyển giữa các server (Lost connections and failing clients between servers)](#62-mất-kết-nối-và-client-gặp-sự-cố-khi-chuyển-giữa-các-server-lost-connections-and-failing-clients-between-servers)
- [6.3 Tóm tắt (Summary)](#63-tóm-tắt-summary)
- [Bảng thuật ngữ của chương](#bảng-thuật-ngữ-của-chương)

---

## Chương này đề cập đến những gì

*(This chapter covers)*

- Hiểu về cân bằng tải (*load balancing*)
- Cài đặt và cấu hình HAProxy để cân bằng tải cho Rabbit (RabbitMQ)
- Viết code biết tự kết nối lại (*reconnect*) và sống sót qua sự cố một cách thông minh

## Mở đầu

Xây dựng một RabbitMQ cluster (*cụm* — nhiều node RabbitMQ chạy cùng nhau như một hệ thống) để bảo đảm tính sẵn sàng (*availability*) và hiệu năng (*performance*) mới chỉ là một nửa cuộc chiến trong việc xây dựng một hạ tầng nhắn tin (*messaging infrastructure*) bền bỉ, chịu lỗi tốt (*resilient*). Nửa còn lại là viết các ứng dụng **luôn lường trước** việc node có thể hỏng, và biết cách kết nối lại vào cluster khi chuyện đó xảy ra.

Có khá nhiều chiến lược để xử lý việc kết nối lại vào cluster, nhưng chiến lược mà chúng ta sẽ tập trung là **dùng một bộ cân bằng tải (*load balancer*) để đảm nhận việc chọn node**. Khi dùng load balancer, bạn không chỉ giảm bớt độ phức tạp của phần code xử lý sự cố trong ứng dụng, mà còn bảo đảm các kết nối được phân bổ đồng đều trên toàn cluster.

Nhưng ngay cả khi đã có load balancer, viết một ứng dụng chịu được việc node hỏng vẫn đòi hỏi nhiều hơn là chỉ mở một kết nối mới tới cluster. Ứng dụng của bạn còn phải sẵn sàng **tạo lại các exchange và queue** mà có thể đã không "sống sót" sau khi node ban đầu bị hỏng. Điều này đặc biệt đúng khi bạn dùng hai node Rabbit độc lập (*standalone*) theo cấu hình chủ động/dự phòng (*active/standby*) — thứ mà chúng ta sẽ bàn ở chương 7. Trước khi bắt tay viết code xử lý sự cố trong ứng dụng, chúng ta hãy xem cần những gì để dùng một load balancer với RabbitMQ.

> 📝 **Giải thích thêm (không có trong sách):** Nhắc lại vài khái niệm của RabbitMQ dùng trong chương này:
> - **exchange**: "bưu cục" nhận message từ producer rồi định tuyến (route) message tới các queue theo quy tắc.
> - **queue**: hàng đợi chứa message chờ consumer lấy ra xử lý.
> - **binding**: quy tắc gắn một queue vào một exchange (kèm routing key).
> - **producer**: ứng dụng gửi (publish) message; **consumer**: ứng dụng nhận (consume) message.
> - **node**: một tiến trình RabbitMQ server; nhiều node ghép lại thành **cluster**.
>
> Trong cluster RabbitMQ "kiểu cũ" (như mô tả trong sách), định nghĩa exchange được sao chép lên mọi node, còn **nội dung và tiến trình của queue chỉ nằm trên đúng một node** — đó là lý do vì sao khi node hỏng, queue có thể "biến mất".

---

## 6.1 Cân bằng tải cho các "chú thỏ" RabbitMQ (Load balancing your Rabbits)

Tùy vào nền tảng kiến thức của bạn, có thể bạn đang thắc mắc: "load balancer là cái quái gì vậy?". Một **load balancer** (*bộ cân bằng tải*) đưa ra **một địa chỉ IP duy nhất**, và đằng sau địa chỉ đó là **nhiều server**.

Giả sử bạn có ba web server phục vụ website của mình, tên là `web1.acme.com`, `web2.acme.com` và `web3.acme.com`. Nếu không có load balancer, khách hàng của bạn sẽ phải tự vào `web1`, và nếu nó đang chết thì thử `web2` hoặc `web3`. Đòi hỏi như vậy là quá nhiều đối với khách hàng, mà nó cũng chẳng cho bạn chút quyền kiểm soát nào về việc mỗi server đang phải gánh bao nhiêu tải tại một thời điểm.

Nếu có load balancer, bạn có thể tạo một địa chỉ IP trên load balancer và đặt tên là `www.acme.com`. Khi đó, mỗi khi khách hàng kết nối tới `www`, load balancer sẽ **chuyển tiếp một cách trong suốt** (*transparently proxies*) kết nối đó tới `web1`, `web2` hoặc `web3` — tùy server nào đang có ít kết nối nhất. Nếu `web1` hỏng, load balancer cũng đủ thông minh để phát hiện ra và ngừng gửi kết nối tới `web1`, thay vào đó để `web2` và `web3` gánh phần tải. Với khách hàng, trông như thể bạn có một server khổng lồ duy nhất tên là `www`, và đó là tất cả những gì họ cần quan tâm. Toàn bộ việc cân bằng tải và phát hiện server hỏng đều do load balancer xử lý một cách trong suốt (xem hình 6.1).

**Hình 6.1 – Cân bằng tải cho nhiều web server để chúng trông như một server duy nhất**

![Hình 6.1 (trang render)](images/pages/page-133.png)

Các mảnh ảnh trích từ hình 6.1 (bóng đổ của khối "Customer" và biểu tượng server):

![Mảnh ảnh hình 6.1 – bóng đổ khối Customer](images/06-code-survives-failure/p133-1.png)
![Mảnh ảnh hình 6.1 – biểu tượng server](images/06-code-survives-failure/p133-3.png)

Vẽ lại hình 6.1:

```mermaid
flowchart TD
    C["Khách hàng (Customer)"] <--> LB["Load balancer<br/>Khách hàng chỉ thấy: www"]
    LB <--> W1["web1"]
    LB <--> W2["web2"]
    LB <--> W3["web3"]
```

*Mô tả hình:* Ở trên cùng là khối **Customer** (khách hàng). Bên cạnh có dòng chữ **"Customer sees: www"** (khách hàng chỉ nhìn thấy: `www`). Khách hàng nối với một thiết bị load balancer ở giữa. Load balancer có mũi tên hai chiều nối xuống ba web server **web1**, **web2**, **web3**. Ý nghĩa: khách hàng không cần biết có ba server — họ chỉ thấy một điểm vào duy nhất.

Khi dùng load balancer với RabbitMQ, **các node trong cluster chính là các server đằng sau load balancer**, còn **producer và consumer của bạn chính là các "khách hàng"**. Ứng dụng của bạn chỉ cần biết địa chỉ IP phía trước (*frontend IP*) của load balancer; load balancer sẽ âm thầm kết nối chúng tới node trong cluster đang có ít kết nối nhất. Nếu muốn mở rộng cluster để tăng hiệu năng, bạn không cần sửa bất kỳ ứng dụng nào — bạn chỉ cần cho node mới tham gia (*join*) vào cluster, rồi thêm node đó vào cấu hình của load balancer. Ứng dụng không phải thay đổi gì cả.

Nếu không có load balancer, ứng dụng của bạn sẽ phải được cấu hình thủ công để biết IP của từng node trong cluster. Chúng cũng sẽ phải tự lo việc chọn node trong cluster và tự phát hiện server hỏng. Vì mỗi ứng dụng tự chọn node theo cách riêng, bạn sẽ không có cách nào để dàn đều tải trên cluster — việc phân bổ tải về cơ bản là ngẫu nhiên. Vì vậy, bằng cách đặt một load balancer phía trước cluster Rabbit, bạn để nó gánh hết những việc phức tạp: chọn node, phát hiện server hỏng và phân phối tải (xem hình 6.2).

**Hình 6.2 – Cân bằng tải cho một RabbitMQ cluster**

![Hình 6.2 (trang render)](images/pages/page-134.png)

Các mảnh ảnh trích từ hình 6.2 (nền cluster, ba node RabbitMQ và bóng đổ của khối ứng dụng):

![Mảnh ảnh hình 6.2 – nền vùng localhost](images/06-code-survives-failure/p134-5.png)
![Mảnh ảnh hình 6.2 – node rabbit@localhost](images/06-code-survives-failure/p134-6.png)
![Mảnh ảnh hình 6.2 – node rabbit_1@localhost](images/06-code-survives-failure/p134-8.png)
![Mảnh ảnh hình 6.2 – node rabbit_2@localhost](images/06-code-survives-failure/p134-10.png)
![Mảnh ảnh hình 6.2 – bóng đổ khối ứng dụng](images/06-code-survives-failure/p134-13.png)

Vẽ lại hình 6.2:

```mermaid
flowchart TD
    CA["Consumer app"] <--> LB
    PA["Producer app"] <--> LB
    LB["Load balancer<br/>Client chỉ thấy: localhost:5670"]
    subgraph H["localhost"]
        N0["RabbitMQ rabbit@localhost<br/>localhost:5672<br/>Exchange A, Queue 1"]
        N1["RabbitMQ rabbit_1@localhost<br/>localhost:5673<br/>Exchange A, Queue 2"]
        N2["RabbitMQ rabbit_2@localhost<br/>localhost:5674<br/>Exchange A, Queue 3"]
        N0 <-- cluster --> N1
        N1 <-- cluster --> N2
    end
    LB <--> N0
    LB <--> N1
    LB <--> N2
```

*Mô tả hình:* Phía trên là hai khối **Consumer app** (ứng dụng consumer) và **Producer app** (ứng dụng producer), cả hai nối tới load balancer. Dòng chữ **"Client sees: localhost:5670"** nghĩa là client chỉ thấy một địa chỉ duy nhất `localhost:5670`. Phía dưới là vùng **localhost** chứa ba node RabbitMQ: `rabbit@localhost` (cổng 5672, chứa Queue 1), `rabbit_1@localhost` (cổng 5673, chứa Queue 2), `rabbit_2@localhost` (cổng 5674, chứa Queue 3). Cả ba node đều có **Exchange A** (vẽ bằng khung nét đứt, vì exchange được sao chép lên mọi node), còn mỗi queue chỉ nằm trên một node. Các node nối với nhau bằng nhãn **cluster**. Load balancer có mũi tên hai chiều tới cả ba node.

Có rất nhiều load balancer khác nhau, nhưng chúng thuộc một trong hai nhóm: **thiết bị phần cứng** (*hardware appliances*) hoặc **phần mềm** (*software*).

- **Thiết bị phần cứng** là những hệ thống mạng chuyên dụng "lực lưỡng", có thể xử lý hàng triệu kết nối mỗi giây mà "không chớp mắt". Chúng thường còn có khả năng clustering nâng cao, để hai load balancer có thể hoạt động như một khối duy nhất, nhờ đó loại bỏ việc chính load balancer trở thành **điểm hỏng duy nhất** (*single point of failure*). Nếu bạn đã có sẵn một load balancer phần cứng, chẳng có lý do gì để không dùng nó cân bằng tải cho cluster RabbitMQ. Chỉ cần cho thiết bị chạy ở chế độ **cân bằng tải tầng 4** (*layer 4 load balancing mode*).
- Nhưng trong hầu hết các tình huống, **một load balancer phần mềm là quá đủ**. Bạn có nhiều khả năng chạm tới giới hạn số node mà một cluster hỗ trợ được, hơn là vượt quá khả năng "tiếp tế" cho cluster của một load balancer phần mềm.

Trong số rất nhiều load balancer phần mềm, chúng ta sẽ dùng **HAProxy**. Nó miễn phí, cực kỳ ổn định, và đang gánh những lượng tải khổng lồ trên Internet cho các trang như StackOverflow. Ngoài ra, nó chạy trên gần như mọi nền tảng họ UNIX và cực kỳ dễ cấu hình. Thay vì ngồi kể HAProxy tuyệt vời ra sao, hãy để chúng tôi cho bạn thấy bằng cách cài đặt nó và cấu hình nó để cân bằng tải cho một RabbitMQ cluster.

> 📝 **Giải thích thêm (không có trong sách):** "Tầng 4" (layer 4) là tầng giao vận (transport — TCP/UDP) trong mô hình OSI. Load balancer tầng 4 chỉ chuyển tiếp các kết nối TCP mà không cần hiểu nội dung bên trong. Điều này rất hợp với RabbitMQ, vì giao thức AMQP chạy trên một kết nối TCP dài hạn và **không phải HTTP** — load balancer tầng 7 (hiểu HTTP) sẽ không phân tích được AMQP. Vì thế trong cấu hình HAProxy ở dưới, bạn sẽ thấy `mode tcp`.

> 💡 **Ví dụ bổ sung (không có trong sách):** So sánh hai cách cấu hình ứng dụng.
>
> *Không có load balancer* — mỗi ứng dụng phải giữ danh sách node và tự thử lần lượt:
> ```python
> NODES = [("10.0.0.1", 5672), ("10.0.0.2", 5672), ("10.0.0.3", 5672)]
> # Ứng dụng phải tự viết vòng lặp thử từng node, tự đoán node nào còn sống,
> # và khi thêm node thứ 4 thì phải sửa + triển khai lại MỌI ứng dụng.
> ```
> *Có load balancer* — ứng dụng chỉ biết một địa chỉ:
> ```python
> AMQP_HOST, AMQP_PORT = "rabbit-lb.acme.com", 5670
> # Thêm node thứ 4? Chỉ cần thêm một dòng "server ..." vào cấu hình HAProxy.
> ```
> Giả sử có 30 ứng dụng cùng kết nối: không có load balancer, nếu cả 30 ứng dụng đều ưu tiên thử node đầu danh sách thì node 1 sẽ gánh gần hết 30 kết nối. Với load balancer chia round-robin cho 3 node, mỗi node nhận khoảng 10 kết nối.

### 6.1.1 Cài đặt HAProxy (Installing HAProxy)

Cài đặt HAProxy rất đơn giản. Hầu hết các bản phân phối Linux hiện đại đều có sẵn HAProxy trong hệ thống quản lý gói, nhưng ở đây chúng ta sẽ build nó từ mã nguồn. Bước đầu tiên là tải HAProxy về máy phát triển và giải nén:

```bash
$ wget http://haproxy.1wt.eu/download/1.4/src/haproxy-1.4.14.tar.gz
Resolving haproxy.1wt.eu... 88.191.124.161
Connecting to haproxy.1wt.eu|88.191.124.161|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 812238 (793K) [application/x-gzip]
Saving to: `haproxy-1.4.14.tar.gz'

100%[=============================>] 812,238      192K/s   in 4.1s

2011-03-29 19:16:56 (192 KB/s) - `haproxy-1.4.14.tar.gz' saved
$ sudo tar xfz haproxy-1.4.14.tar.gz
```

Sau khi giải nén mã nguồn, bạn cần chạy `make` để build ra file thực thi HAProxy. Trước khi chạy `make`, bạn cần chọn nền tảng đích (*target platform*). Nếu bạn build trên hệ thống Linux 2.6, hãy đặt `TARGET=linux26` để bật hỗ trợ **epoll**. Với tất cả các hệ thống họ UNIX khác, `TARGET=generic` thường là lựa chọn đúng. Không dài dòng nữa, hãy bắt đầu build.

> **NOTE (Ghi chú):** **epoll** là một tính năng của nhân (*kernel*) Linux 2.6, cho phép các phần mềm mạng hướng sự kiện (*event-based*) như HAProxy **được thông báo** khi có gói tin mới đang chờ, thay vì phần mềm phải liên tục **thăm dò** (*poll*) theo chu kỳ đều đặn. Điều này có thể giảm mức sử dụng CPU và cải thiện hiệu năng trên các hệ thống chịu tải nặng. FreeBSD có một tính năng tương tự tên là **kqueues**, và HAProxy có thể tận dụng nó bằng cách đặt `TARGET=freebsd`.

```bash
$ cd haproxy-1.4.14
$ sudo make TARGET=generic
...
gcc -g -o haproxy src/haproxy.o src/sessionhash.o src/b...
```

Giờ bạn sẽ có một file thực thi tên `haproxy` trong thư mục build. Nếu mọi thứ được build đúng, bạn có thể chạy `haproxy --help` để xem các tùy chọn cấu hình của nó. Cuối cùng, hãy copy file thực thi `haproxy` vào `/usr/local/sbin` để nó có mặt trong UNIX path (đường dẫn tìm lệnh) của bạn. Bây giờ, tất cả những gì bạn cần là một file cấu hình cho HAProxy để nó có thể bắt đầu cân bằng tải cho RabbitMQ cluster trên máy phát triển của bạn.

> 📝 **Giải thích thêm (không có trong sách):** Phiên bản HAProxy 1.4.14 và URL `haproxy.1wt.eu` là của thời điểm sách được viết (2011). Ngày nay bạn thường cài bằng trình quản lý gói, ví dụ:
> ```bash
> # Debian/Ubuntu
> $ sudo apt-get install haproxy
> # macOS (Homebrew)
> $ brew install haproxy
> # Hoặc build từ nguồn trên Linux hiện đại (target linux26 đã được thay bằng linux-glibc)
> $ make TARGET=linux-glibc
> $ sudo cp haproxy /usr/local/sbin/
> ```
> Cú pháp `listen <tên> <ip>:<port>` dùng trong sách vẫn còn được hỗ trợ ở các bản cũ; ở các bản HAProxy mới, người ta viết `listen <tên>` rồi xuống dòng `bind <ip>:<port>`.

> 💡 **Ví dụ bổ sung (không có trong sách):** Hình dung sự khác nhau giữa *poll* và *epoll*. Giả sử HAProxy đang giữ 10.000 kết nối, nhưng mỗi lúc chỉ có khoảng 20 kết nối có dữ liệu mới. Cách "thăm dò" kiểu cũ giống như người bưu tá gõ cửa cả 10.000 nhà mỗi vòng để hỏi "có thư không?". Với epoll, chỉ 20 nhà có thư mới "giơ cờ", và kernel đưa cho HAProxy đúng danh sách 20 nhà đó — tốn ít CPU hơn rất nhiều.

### 6.1.2 Cấu hình HAProxy (Configuring HAProxy)

HAProxy dùng **một file cấu hình duy nhất** để định nghĩa mọi thứ, từ các IP phía trước (*frontend IP*) được công bố ra ngoài cho đến các server đứng phía sau chúng. Listing dưới đây là cấu hình bạn sẽ dùng để cân bằng tải cho cluster Rabbit trên máy cục bộ.

**Listing 6.1 – Cấu hình HAProxy cho RabbitMQ cluster cục bộ**

```
global                                  # Tùy chọn ghi log (Logging options)
    log 127.0.0.1    local0 info
    maxconn 4096
    stats socket /tmp/haproxy.socket uid haproxy mode 770 level admin
    daemon

defaults                                # Mặc định cho cân bằng tải (Load balancing defaults)
    log    global
    mode    tcp
    option    tcplog
    option    dontlognull
    retries    3
    option redispatch
    maxconn    2000
    timeout connect    5s
    timeout client 120s
    timeout server 120s

listen rabbitmq_local_cluster 127.0.0.1:5670        # (1) IP phía trước cho consumer và producer
    mode tcp
    balance roundrobin                              # (2) Tùy chọn cân bằng tải
    server rabbit 127.0.0.1:5672 check inter 5000 rise 2 fall 3      # (3) Các node trong cluster
    server rabbit_1 127.0.0.1:5673 check inter 5000 rise 2 fall 3    #     cần được cân bằng tải
    server rabbit_2 127.0.0.1:5674 check inter 5000 rise 2 fall 3

listen private_monitoring :8100                     # (4) Trang thống kê (Statistics page)
    mode http
    option httplog
    stats enable
    stats uri   /stats
    stats refresh 5s
```

Chú thích các phần trong listing (theo các mũi tên/số trong sách):

- **`global` – Logging options (Tùy chọn ghi log):** cấu hình toàn cục, gồm nơi gửi log (`log 127.0.0.1 local0 info`), số kết nối tối đa (`maxconn 4096`), socket thống kê/quản trị, và chạy nền (`daemon`).
- **`defaults` – Load balancing defaults (Các giá trị mặc định cho cân bằng tải):** áp dụng cho mọi phần `listen` phía sau.
- **(1) Front-end IP for consumers and producers – IP phía trước cho consumer và producer:** dòng `listen rabbitmq_local_cluster 127.0.0.1:5670`.
- **(2) Load balancing options – Tùy chọn cân bằng tải:** `mode tcp` và `balance roundrobin`.
- **(3) Cluster nodes to load balance – Các node trong cluster cần cân bằng tải:** ba dòng `server ...`.
- **(4) Statistics page – Trang thống kê:** phần `listen private_monitoring :8100`.

Ở (1), bạn định nghĩa IP và cổng mà client sẽ kết nối tới. Bạn dùng cổng `5670` để nó không đụng độ với chính các node trong RabbitMQ cluster. Sau đó bạn bảo HAProxy dùng **thuật toán xoay vòng** (*round-robin*) (2) để phân phối tải giữa các backend (xem hình 6.3).

Phần thú vị nhất là (3), nơi chúng ta định nghĩa các backend (các server thật đứng phía sau):

```
server rabbit 127.0.0.1:5672 check inter 5000 rise 2 fall 3
server rabbit_1 127.0.0.1:5673 check inter 5000 rise 2 fall 3
server rabbit_2 127.0.0.1:5674 check inter 5000 rise 2 fall 3
```

Mỗi chỉ thị (*directive*) cấu hình backend có năm phần:

- `server <name>` là định danh nội bộ đặt cho định nghĩa backend đó.
- `<IP>:<port>` là IP và số cổng của server backend cần kết nối tới.
- `check inter <value>` định nghĩa tần suất (tính bằng mili giây) kiểm tra xem backend có còn khả dụng hay không.
- `rise <value>` cho biết một backend, sau khi đã bị đánh dấu hỏng, phải vượt qua bao nhiêu lần kiểm tra sức khỏe (*health check*) thành công thì mới được coi là dùng được trở lại.
- `fall <value>` chỉ định một backend phải trượt bao nhiêu lần health check thì HAProxy mới ngừng dùng nó.

> 📝 **Giải thích thêm (không có trong sách):** Ý nghĩa các dòng khác trong listing 6.1 (theo tài liệu HAProxy):
> - `stats socket /tmp/haproxy.socket uid haproxy mode 770 level admin`: mở một UNIX socket để xem thống kê và ra lệnh quản trị (ví dụ tạm tắt một backend) khi HAProxy đang chạy.
> - `mode tcp`: chuyển tiếp ở mức TCP (tầng 4), cần thiết cho AMQP. `option tcplog`: ghi log dạng phù hợp với TCP. `option dontlognull`: không ghi log những kết nối không truyền dữ liệu nào (ví dụ các lần dò cổng).
> - `retries 3`: thử kết nối tới một server tối đa 3 lần nếu thất bại. `option redispatch`: nếu server đã chọn không kết nối được, cho phép chuyển yêu cầu sang server khác.
> - `maxconn 2000` (trong defaults): giới hạn số kết nối đồng thời cho mỗi frontend.
> - `timeout connect 5s`: chờ tối đa 5 giây để mở kết nối tới backend. `timeout client 120s` / `timeout server 120s`: nếu phía client/server im lặng quá 120 giây thì đóng kết nối. Lưu ý với AMQP: kết nối consumer có thể nhàn rỗi rất lâu, nên nên đặt heartbeat AMQP nhỏ hơn giá trị timeout này để HAProxy không cắt kết nối "đang ngủ".
> - `mode http`, `option httplog`, `stats enable`, `stats uri /stats`, `stats refresh 5s`: phần thống kê là một trang web HTTP, đặt tại đường dẫn `/stats`, tự làm mới mỗi 5 giây.

> 💡 **Ví dụ bổ sung (không có trong sách):** Bài toán: với `check inter 5000 rise 2 fall 3`, nếu node `rabbit_1` chết đột ngột thì sau bao lâu HAProxy ngừng gửi kết nối mới tới nó? Và khi node sống lại thì sau bao lâu nó được dùng lại?
>
> *Lời giải:* HAProxy kiểm tra mỗi 5000 ms = 5 giây. Cần trượt 3 lần liên tiếp (`fall 3`) → khoảng 3 × 5 = **tối đa ~15 giây** (cộng thêm thời gian timeout của mỗi lần kiểm tra) thì `rabbit_1` bị đánh dấu DOWN. Khi node sống lại, cần 2 lần kiểm tra thành công liên tiếp (`rise 2`) → khoảng 2 × 5 = **~10 giây** thì nó được đánh dấu UP trở lại. Trong khoảng ~15 giây đầu, một client mới vẫn có thể bị chuyển tới node chết — nhờ `retries 3` và `option redispatch`, HAProxy sẽ thử lại và chuyển sang node khác.
>
> Muốn phát hiện nhanh hơn? Giảm `inter` xuống (ví dụ `check inter 2000 rise 2 fall 2` → phát hiện trong ~4 giây), đổi lại là nhiều lưu lượng kiểm tra hơn.

**Hình 6.3 – Cân bằng tải xoay vòng (round-robin)**

![Hình 6.3 (trang render)](images/pages/page-137.png)

Các mảnh ảnh trích từ hình 6.3 (nền cluster, ba node RabbitMQ và bóng đổ của khối consumer):

![Mảnh ảnh hình 6.3 – nền vùng localhost](images/06-code-survives-failure/p137-6.png)
![Mảnh ảnh hình 6.3 – node rabbit@localhost](images/06-code-survives-failure/p137-7.png)
![Mảnh ảnh hình 6.3 – node rabbit_1@localhost](images/06-code-survives-failure/p137-9.png)
![Mảnh ảnh hình 6.3 – node rabbit_2@localhost](images/06-code-survives-failure/p137-11.png)
![Mảnh ảnh hình 6.3 – bóng đổ khối consumer](images/06-code-survives-failure/p137-14.png)

Vẽ lại hình 6.3:

```mermaid
flowchart TD
    C1["Consumer #1"] --> LB["Load balancer (HAProxy)"]
    C2["Consumer #2"] -.-> LB
    C3["Consumer #3"] -.-> LB
    subgraph H["localhost"]
        N0["RabbitMQ rabbit@localhost<br/>localhost:5672<br/>Exchange A, Queue 1"]
        N1["RabbitMQ rabbit_1@localhost<br/>localhost:5673<br/>Exchange A, Queue 2"]
        N2["RabbitMQ rabbit_2@localhost<br/>localhost:5674<br/>Exchange A, Queue 3"]
        N0 <-- cluster --> N1
        N1 <-- cluster --> N2
    end
    LB -- "Consumer #1" --> N0
    LB -. "Consumer #2" .-> N1
    LB -. "Consumer #3" .-> N2
```

*Mô tả hình:* Ba consumer (**Consumer #1**, **#2**, **#3**) lần lượt kết nối tới load balancer. Theo thuật toán round-robin, kết nối của **Consumer #1** được chuyển tới node `rabbit@localhost` (5672), **Consumer #2** tới `rabbit_1@localhost` (5673), **Consumer #3** tới `rabbit_2@localhost` (5674). Mỗi kiểu nét mũi tên (liền, chấm, gạch) biểu diễn đường đi của một consumer. Các node vẫn nối với nhau bằng nhãn **cluster**, mỗi node có Exchange A và một queue riêng.

> 💡 **Ví dụ bổ sung (không có trong sách):** Round-robin nghĩa là "chia bài lần lượt". Với 3 backend và 7 kết nối đến theo thứ tự, HAProxy chia như sau:
>
> | Kết nối thứ | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
> |---|---|---|---|---|---|---|---|
> | Backend | rabbit | rabbit_1 | rabbit_2 | rabbit | rabbit_1 | rabbit_2 | rabbit |
>
> Kết quả: rabbit có 3 kết nối, rabbit_1 và rabbit_2 mỗi node 2. Nếu rabbit_1 bị đánh dấu DOWN, các kết nối mới chỉ xoay vòng giữa rabbit và rabbit_2. (Ngoài `roundrobin`, HAProxy còn có `balance leastconn` — chọn backend đang có ít kết nối nhất — thường hợp với các kết nối dài hạn như AMQP.)

Phần cấu hình cuối cùng là dành cho trang thống kê (4). Phần này là tùy chọn, nhưng khi bật nó trên cổng `8100`, bạn có thể vào http://localhost:8100 để xem trạng thái hiện tại của HAProxy, bao gồm việc mỗi node trong RabbitMQ cluster đang phục vụ bao nhiêu kết nối. Điều này hữu ích khi bạn muốn xem tải trên cluster, hoặc xem hiện có bao nhiêu node đang sống (*up*) hay đang chết (*down*). HAProxy còn rất nhiều tùy chọn cấu hình khác, cho phép làm mọi thứ, từ các quy tắc cân bằng tải phức tạp cho đến việc đánh dấu một số node backend là **server dự phòng** (*backup servers*) — chỉ được dùng khi tất cả các backend chính đều chết. Hãy xem tài liệu hướng dẫn của HAProxy để tìm hiểu thêm tại http://haproxy.1wt.eu/download/1.4/doc/configuration.txt.

Giờ hãy khởi động HAProxy với cấu hình mới và kiểm tra xem nó có chạy không. Chạy `/usr/local/sbin/haproxy -f config_file`, trong đó `config_file` là file cấu hình bạn vừa tạo. Nếu mọi thứ suôn sẻ, bạn có thể mở trang http://localhost:8100/stats và thấy một trang web giống như hình 6.4.

```bash
$ /usr/local/sbin/haproxy -f config_file
```

**Hình 6.4 – Trang thống kê của HAProxy**

![Hình 6.4](images/06-code-survives-failure/p138-1.png)

*Mô tả hình:* Ảnh chụp trình duyệt mở địa chỉ `localhost:8100/stats`, tiêu đề tab "Statistics Report for HAProxy". Đầu trang ghi **"HAProxy version 1.4.14, released 2011/03/29"** và **"Statistics Report for pid 30430"** (báo cáo thống kê cho tiến trình có pid 30430). Mục **"General process information"** (thông tin chung về tiến trình) cho biết: pid = 30430, uptime (thời gian chạy) 0d 0h00m53s, giới hạn hệ thống (memmax = unlimited, ulimit-n = 8208), maxsock = 8208, maxconn = 4096, số kết nối hiện tại = 3, số tác vụ đang chạy 1/6. Có bảng chú giải màu: *active UP* (đang hoạt động), *active UP, going down* (đang hoạt động, sắp xuống), *active DOWN, going up* (đang chết, sắp lên), *active or backup DOWN* (chính hoặc dự phòng đang chết), *active or backup DOWN for maintenance (MAINT)* (đang chết do bảo trì), *backup UP*, *backup UP, going down*, *backup DOWN, going up*, *not checked* (không kiểm tra); ghi chú "UP with load-balancing disabled is reported as NOLB" (UP nhưng bị tắt cân bằng tải sẽ hiện là NOLB). Bên phải có các tùy chọn hiển thị (ẩn server DOWN, tắt tự làm mới, làm mới ngay, xuất CSV) và liên kết ngoài (trang chủ, cập nhật v1.4, tài liệu trực tuyến).

Bảng **rabbitmq_local_cluster** có các cột nhóm: Queue (hàng đợi), Session rate (tốc độ phiên), **Sessions** (phiên: Cur = hiện tại, Max, Limit, Total, LbTot), Bytes (In/Out), Denied (bị từ chối), Errors (lỗi), Warnings (cảnh báo), Server (Status, LastChk, Wght, Act, Bck, Chk, Dwn, Dwntme, Thrtle). Các dòng: *Frontend* (trạng thái OPEN, Cur = 2), *rabbit* (Cur = 1, trạng thái "42s UP", LastChk "L4OK in 0ms"), *rabbit_1* (Cur = 1, "53s UP", L4OK), *rabbit_2* (Cur = 0, "53s UP", L4OK), *Backend* (Cur = 2, "53s UP"). Bảng thứ hai **private_monitoring** là chính trang thống kê (Frontend OPEN, Backend 53s UP).

> 📝 **Giải thích thêm (không có trong sách):** Cách đọc nhanh trang thống kê: cột **Sessions → Cur** cho biết mỗi node đang giữ bao nhiêu kết nối ngay lúc này (trong hình: rabbit = 1, rabbit_1 = 1, rabbit_2 = 0). Cột **Status** cho biết node UP hay DOWN và đã ở trạng thái đó bao lâu. **LastChk = "L4OK"** nghĩa là lần health check gần nhất ở tầng 4 (mở được kết nối TCP) đã thành công. Bạn sẽ dùng cột Cur này ở mục 6.2 để biết consumer đang nối vào node nào.

> 📝 **Giải thích thêm (không có trong sách) — Health Check hiện đại cho HAProxy / Load Balancer với RabbitMQ 3.8 – 4.0 (HTTP Health Check API thay vì chỉ TCP L4):**
>
> Trong Listing 6.1 và Hình 6.4, HAProxy chỉ thực hiện **TCP Health Check ở tầng 4 (`L4OK`)** lên cổng `5672` (tức là hễ mở được socket TCP tới cổng `5672` thì coi như node đó khỏe mạnh). Trong vận hành thực tế với RabbitMQ hiện đại, **chỉ kiểm tra TCP cổng 5672 là chưa đủ** vì 3 tình huống sau:
> 1. **Node đang bị báo động tràn RAM hoặc đầy ổ cứng (*Memory / Free Disk Space Alarm*)**: Khi chạm ngưỡng báo động, RabbitMQ vẫn mở cổng TCP `5672` (nên `L4OK` vẫn xanh!), nhưng sẽ lập tức chặn (`connection.blocked`) mọi producer kết nối vào.
> 2. **Node đang trong chế độ bảo trì nâng cấp cuốn chiếu (`rabbitmq-upgrade drain`)**: Node đang chuẩn bị tắt để nâng cấp, không nên nhận thêm kết nối client mới.
> 3. **Virtual host chưa khởi tạo xong hoặc bị treo (`down`)**: Cổng `5672` đã mở nhưng các vhost bên trong chưa sẵn sàng phục vụ.
>
> **Giải pháp chuẩn trong RabbitMQ 3.8 – 4.0:**
> RabbitMQ cung cấp bộ **HTTP Health Check Endpoints** chuyên dụng (không cần xác thực hoặc dùng Basic Auth nhẹ) trên cổng HTTP API (`15672`), trả về HTTP `200 OK` khi khỏe mạnh và `503 Service Unavailable` khi có sự cố:
>
> | HTTP Endpoint trên cổng `15672` | Ý nghĩa kiểm tra | Khi nào trả về `503`? |
> |---|---|---|
> | `GET /api/health/checks/local-alarms` | Kiểm tra báo động tài nguyên trên chính node đó *(Khuyên dùng cho Load Balancer)* | Khi node bị tràn RAM, đầy đĩa, hoặc đang ở chế độ `drain` bảo trì |
> | `GET /api/health/checks/alarms` | Kiểm tra báo động trên **toàn bộ cluster** | Khi bất kỳ node nào trong cluster bị báo động RAM/đĩa |
> | `GET /api/health/checks/virtual-hosts` | Kiểm tra tất cả các `vhost` trên node có đang chạy không | Khi có vhost bị crash/chưa khởi động xong |
> | `GET /api/health/checks/port-listener/5672` | Kiểm tra listener AMQP cổng `5672` có đang hoạt động không | Khi cổng `5672` chưa mở hoặc bị tắt |
>
> 💡 **Ví dụ bổ sung (không có trong sách) — File cấu hình `haproxy.cfg` chuẩn hiện đại (HAProxy 2.x / 3.x) cho cụm RabbitMQ 3.8 – 4.0:**
>
> Lưu ý ở HAProxy 2.x/3.x, cú pháp viết gộp `listen <name> <ip>:<port>` trong sách đã bị loại bỏ (phải tách dòng `bind <ip>:<port>`), đồng thời ta kết hợp **chuyển tiếp TCP trên cổng 5672** với **kiểm tra sức khỏe HTTP (`httpchk`) trên cổng 15672** và thuật toán **`leastconn`**:
> ```haproxy
> global
>     log stdout format raw local0 info
>     maxconn 4096
>
> defaults
>     log     global
>     mode    tcp
>     option  tcplog
>     option  dontlognull
>     retries 3
>     option  redispatch
>     timeout connect 5s
>     # Đặt timeout lớn hơn AMQP heartbeat (thường là 60s) và bật TCP keepalive
>     timeout client  180s
>     timeout server  180s
>     option  clitcpka
>     option  srvtcpka
>
> listen rabbitmq_cluster
>     bind 0.0.0.0:5670
>     mode tcp
>     # leastconn tốt hơn roundrobin cho kết nối AMQP dài hạn vì phân bổ theo số kết nối đang mở thực tế
>     balance leastconn
>     # Dùng HTTP Health Check hỏi endpoint /api/health/checks/local-alarms trên cổng 15672!
>     option httpchk
>     http-check send meth GET uri /api/health/checks/local-alarms ver HTTP/1.1 hdr Host localhost
>     http-check expect status 200
>     # Giao lưu lượng AMQP vào cổng 5672/5673/5674, nhưng gửi gói khám sức khỏe vào cổng HTTP Management
>     server rabbit   127.0.0.1:5672 check port 15672 inter 3000 rise 2 fall 3
>     server rabbit_1 127.0.0.1:5673 check port 15673 inter 3000 rise 2 fall 3
>     server rabbit_2 127.0.0.1:5674 check port 15674 inter 3000 rise 2 fall 3
>
> listen stats
>     bind 0.0.0.0:8100
>     mode http
>     stats enable
>     stats uri /stats
>     stats refresh 5s
> ```

Giờ bạn đã có một load balancer chạy được trên máy phát triển, chúng ta sẽ đi sâu vào việc dùng nó để xây dựng khả năng chuyển đổi dự phòng (*failover*) và khả năng chịu lỗi (*resilience*) cho các ứng dụng nhắn tin của bạn.

---

## 6.2 Mất kết nối và client gặp sự cố khi chuyển giữa các server (Lost connections and failing clients between servers)

Khi một node trong cluster hỏng, ứng dụng của bạn đột nhiên phải đưa ra một quyết định: **"tiếp theo mình kết nối vào đâu?"**. Để trả lời câu hỏi đó một cách hiệu quả, bạn phải lường trước nó trong code từ rất lâu trước khi sự cố xảy ra. Xử lý êm đẹp (*gracefully*) việc node hỏng đòi hỏi một sự thay đổi trong tư duy. Clustering **không có nghĩa** là ứng dụng của bạn sẽ không bao giờ gặp sự cố với Rabbit; nó có nghĩa là **khi có sự cố, các node vẫn còn chỗ khác để đi tới và tiếp tục chạy**. Vì vậy, bước đầu tiên là lùi lại một bước và cân nhắc xem bạn có thể đưa ra những giả định nào trước khi viết code:

1. ***Nếu tôi kết nối lại tới một server mới, chuyện gì xảy ra với các channel của tôi và tất cả các vòng lặp tiêu thụ message (consumption loop) gắn với chúng?*** Chúng giờ đã vô hiệu và chẳng trỏ tới đâu cả. Bạn cần dựng lại cả hai.
2. ***Khi kết nối lại, tôi có thể giả định rằng mọi exchange, queue và binding của tôi vẫn còn trong cluster không? Tôi có thể chỉ việc kết nối lại và ngay lập tức tiếp tục consume từ các queue của mình không?*** Câu trả lời là **không**. Bạn không thể giả định rằng queue và binding đã sống sót sau khi node hỏng. Bạn phải giả định rằng tất cả các queue bạn đang consume đều được đặt (*hosted*) trên chính node mà bạn đang gắn vào — và giờ chúng không còn tồn tại nữa. Điều tương tự áp dụng cho binding của các queue đó. Tuy nhiên, exchange lại là chuyện khác: nếu bạn đang dùng tính năng clustering tích hợp sẵn của Rabbit, bạn có thể giả định exchange sẽ sống sót khi node hỏng, vì chúng được sao chép (*replicated*) lên mọi node. Nhưng nếu bạn dùng mô hình active/standby như chúng tôi mô tả ở phần tiếp theo, thì bạn thậm chí **không thể giả định** exchange sẽ sống sót sau khi chuyển đổi dự phòng (*failover*).

> 📝 **Giải thích thêm (không có trong sách):** **channel** là một "kết nối ảo" nhẹ, chạy bên trong một kết nối TCP (connection) tới RabbitMQ; mọi thao tác như khai báo queue, publish, consume đều đi qua channel. Vì channel "sống nhờ" connection, nên khi connection chết thì mọi channel trên nó — và mọi subscription (đăng ký consume) gắn với các channel đó — đều chết theo.

Điều bạn có thể rút ra từ những câu hỏi trên là: **bạn không thể giả định bất cứ điều gì về trạng thái của cluster khi bạn failover sang một node mới**. Dù cluster Rabbit đã cho ứng dụng của bạn một nơi mới để kết nối, bạn không thể giả định thứ gì còn và thứ gì không còn. Ở một khía cạnh nào đó, bạn nên luôn coi việc failover **như thể bạn đang kết nối tới một RabbitMQ server hoàn toàn không liên quan**, chứ không phải một node trong cluster có chung một phần trạng thái. Do đó, mỗi khi có node hỏng, việc đầu tiên cần làm sau khi phát hiện sự cố và kết nối lại là **dựng lại "kết cấu" (*fabric*) gồm các exchange, queue và binding** mà ứng dụng của bạn cần để hoạt động.

> 💡 **Ví dụ bổ sung (không có trong sách):** Một tình huống cụ thể với cluster 3 node ở hình 6.2:
> - Consumer kết nối qua HAProxy, được đưa tới node `rabbit` (5672), rồi khai báo queue `cluster_test` → queue này **nằm trên node `rabbit`**. Exchange `cluster_test` được sao chép lên cả 3 node.
> - Node `rabbit` chết. HAProxy chuyển kết nối mới của consumer tới `rabbit_1`.
> - Trên `rabbit_1`: exchange `cluster_test` **vẫn còn** (vì đã được sao chép), nhưng queue `cluster_test` và binding của nó **không dùng được** (queue sống trên node đã chết).
> - Nếu consumer chỉ gọi `basic_consume("cluster_test")` mà không khai báo lại queue, nó sẽ nhận lỗi "queue không tồn tại/không truy cập được". Cách an toàn: luôn chạy lại `exchange_declare` → `queue_declare` → `queue_bind` rồi mới `basic_consume`. Các lệnh `declare` trong AMQP là **idempotent** (gọi lại nhiều lần không sao): nếu đã có thì chẳng thay đổi gì, nếu chưa có thì tạo mới.

Trước khi đi vào code, hãy nói về những gì bạn cần để chạy nó. Giống như các ví dụ ở chương 3, bạn cần một bản cài đặt Python 2.6 hoạt động được và đã cài Pika 0.9.6 trở lên. Ngoài ra, bạn còn cần:

- Một RabbitMQ cluster được dựng trên máy phát triển cục bộ của bạn.
- HAProxy đã được cấu hình và đang chạy trên cùng máy đó, lắng nghe trên cổng `5670` cho các kết nối AMQP.

Bạn sẽ xây dựng một producer và một consumer mẫu có thể sống sót khi một node trong cluster hỏng. Xây dựng một consumer "biết về cluster" (*cluster-aware*) là việc khó hơn, vì chính consumer là bên dựng nên kết cấu nhắn tin (exchange, queue và binding). Do đó, việc dựng lại kết cấu đó sau khi node hỏng là trách nhiệm của consumer. Với một consumer tiêu chuẩn, phần thân chính của ứng dụng sẽ trông giống như listing dưới đây.

**Listing 6.2 – Phần thân chính của một consumer tiêu chuẩn**

```python
conn_broker = pika.BlockingConnection(conn_params)            # (B) Kết nối tới RabbitMQ

channel = conn_broker.channel()                                # (C) Hành vi kết nối tùy biến

channel.exchange_declare( exchange="cluster_test",             # (D) Khai báo exchange,
                          type="direct",                       #     queue và binding
                          auto_delete=False)
channel.queue_declare( queue="cluster_test",
                       auto_delete=False)
channel.queue_bind( queue="cluster_test",
                    exchange="cluster_test",
                    routing_key="cluster_test")

print "Ready for testing!"
channel.basic_consume( msg_rcvd,                               # (E) Bắt đầu consume message
                       queue="cluster_test",
                       no_ack=False,
                       consumer_tag="cluster_test")
channel.start_consuming()
```

Chú thích:
- **(B) Connect to RabbitMQ** – Kết nối tới RabbitMQ.
- **(C) Custom connection behavior** – Hành vi kết nối tùy biến (mở channel).
- **(D) Declare exchanges, queues, & bindings** – Khai báo exchange, queue và binding.
- **(E) Start consuming messages** – Bắt đầu tiêu thụ message.

Với các tham số kết nối đã được thiết lập từ trước, bạn tạo kết nối tới server (B). Sau đó bạn mở channel (C) và bắt đầu khai báo exchange, queue và binding (D) — tức là kết cấu nhắn tin của bạn. Sau khi kết cấu đã được dựng, bạn tạo một đăng ký tiêu thụ (*consumption subscription*) (E) (được xử lý bởi hàm `msg_rcvd`) và bắt đầu consume message.

Tại thời điểm này, nếu node gặp sự cố, chương trình của bạn sẽ **crash với một exception không được xử lý** (*unhandled exception*). Đó là vì đoạn code này không biết phải làm gì khi xảy ra lỗi kết nối. Điều bạn cần làm là bọc đoạn code này trong một bộ xử lý exception (*exception handler*) và khởi động lại việc kết nối khi sự cố xảy ra. Nhưng bắt đầu bọc từ đâu? Phần nào trong thân chính cần được chạy lại sau một sự cố? **Tất cả**. Nếu bạn không giả định bất kỳ phần nào của kết cấu nhắn tin sống sót sau khi node hỏng, thì toàn bộ phần thân chính cần được thực thi lại mỗi khi có lỗi. Nếu viết lại phần thân chính với suy nghĩ này, nó sẽ trông giống listing dưới đây.

**Listing 6.3 – Phần thân chính của consumer biết về cluster (cluster-aware)**

```python
while True:                                                        # (B) Khi có lỗi, kết nối lại RabbitMQ
    try:
        conn_broker = pika.BlockingConnection(conn_params)         # (C) Thiết lập kết nối tới RabbitMQ
        channel = conn_broker.channel()

        channel.exchange_declare( exchange="cluster_test",         # (D) Khai báo exchange,
                                  type="direct",                   #     queue, binding
                                  auto_delete=False)
        channel.queue_declare( queue="cluster_test",
                               auto_delete=False)
        channel.queue_bind( queue="cluster_test",
                            exchange="cluster_test",
                            routing_key="cluster_test")

        print "Ready for testing!"
        channel.basic_consume( msg_rcvd,                           # (E) Bắt đầu consume message
                               queue="cluster_test",
                               no_ack=False,
                               consumer_tag="cluster_test")
        channel.start_consuming()
    except Exception, e:                                           # (F) Bắt lỗi kết nối và in ra
        traceback.print_exc()
```

Chú thích:
- **(B) On fault, reconnect to RabbitMQ** – Khi có lỗi, kết nối lại tới RabbitMQ.
- **(C) Establish connection to RabbitMQ** – Thiết lập kết nối tới RabbitMQ.
- **(D) Declare exchange, queues, bindings** – Khai báo exchange, queue, binding.
- **(E) Start consuming messages** – Bắt đầu tiêu thụ message.
- **(F) Trap connection errors and print them** – Bắt các lỗi kết nối và in chúng ra.

Bằng cách bọc phần thân chính trong một khối `try...except`, giờ bạn có thể phát hiện các lỗi kết nối (F) và ngăn chúng làm consumer crash. Trong trường hợp này, bạn bắt mọi lỗi và in chúng ra màn hình. Nhưng đó mới chỉ là một nửa giải pháp. Bạn không còn bị crash khi node hỏng nữa, nhưng bạn vẫn cần kết nối lại và dựng lại kết cấu. Để làm điều đó, bạn bọc toàn bộ phần thân chính (bao gồm cả khối `try...except` mới) bằng một **vòng lặp vô hạn** (B).

Khi ứng dụng mới khởi động, nó đi vào vòng lặp rồi dựng kết nối (C) và kết cấu (D). Sau đó nó "tạm dừng" vòng lặp để bắt đầu consume (E). Chừng nào không có lỗi, vòng lặp ngoài sẽ chỉ tiến đến được đó mà thôi. Nhưng ngay khi một node hỏng, một lỗi kết nối xảy ra. Điều này khiến quyền điều khiển chuyển từ đoạn code consume (E) sang bộ xử lý exception bên ngoài (F). Bộ xử lý exception khi đó ngăn chương trình crash bằng cách bắt lỗi, in nó ra màn hình, rồi trả quyền điều khiển lại cho vòng lặp ngoài. Lúc này vòng lặp bắt đầu lại toàn bộ quá trình kết nối từ đầu (B) bằng cách dựng một kết nối mới (C) và kết cấu mới (D) y như trước.

Đó là một thay đổi đơn giản, nhưng nó giúp chương trình của bạn xử lý được việc node hỏng trong một RabbitMQ cluster. Nó cũng sẽ xử lý được **một biến thể của việc node hỏng** mà chúng ta chưa bàn tới, nhưng bạn cần biết.

```mermaid
flowchart TD
    B["(B) while True"] --> C["(C) Tạo connection + channel"]
    C --> D["(D) Khai báo exchange, queue, binding"]
    D --> E["(E) basic_consume + start_consuming<br/>(chạy mãi khi không có lỗi)"]
    E -- "Lỗi kết nối / consumer bị hủy" --> F["(F) except: in traceback"]
    F --> B
```

*(Sơ đồ trên do người dịch vẽ để minh họa luồng điều khiển của listing 6.3.)*

Cho đến giờ, khi nói về việc một node trong cluster hỏng từ góc nhìn của ứng dụng, chúng ta luôn nói đó là **node mà ứng dụng đang kết nối tới**. Có thể bạn đã nghĩ rằng chừng nào node đang nối với ứng dụng không hỏng thì ứng dụng chẳng có gì phải lo. Điều đó không hoàn toàn đúng. Nếu bạn còn nhớ cách queue hoạt động trong cluster, bạn sẽ để ý rằng **queue chỉ tồn tại trên một node**. Vì ứng dụng không biết queue nằm trên node nào khi nó bắt đầu consume, nên hoàn toàn có thể ứng dụng đang kết nối tới node A trong cluster nhưng lại đang consume từ một queue nằm trên node B. Vậy chuyện gì xảy ra khi node B hỏng? Ứng dụng **không gặp lỗi kết nối nào cả**, nhưng queue mà nó đang được cho là consume thì **không còn tồn tại nữa**.

Nếu bạn đang chạy một phiên bản RabbitMQ cũ hơn 2.4.0 thì bạn khá là xui. Consumer của bạn sẽ ngồi đó "ngây thơ và vui vẻ", chẳng làm gì cả mãi mãi (ít nhất là cho tới khi bạn khởi động lại nó). Giới hạn này thuộc về đặc tính của AMQP nhiều hơn là của bản thân RabbitMQ. Nhưng cùng với RabbitMQ 2.4.0 là một phần mở rộng (*extension*) mới cho AMQP gọi là **thông báo hủy** (*cancellation notification*). Với cancellation notification, consumer của bạn sẽ nhận được thông báo khi đăng ký (*subscription*) của nó bị chấm dứt vì **bất kỳ lý do nào khác ngoài việc chính consumer tự hủy nó**. Trong Pika, điều này thể hiện bằng một exception được ném ra trong đoạn code consume. Exception đó sẽ bị bộ xử lý exception của bạn bắt lại, và đoạn code kết nối lại/dựng lại của bạn sẽ thiết lập lại kết nối và dựng lại kết cấu.

> 📝 **Giải thích thêm (không có trong sách):** Ở mức giao thức, khi queue mà consumer đang đăng ký bị xóa hoặc node chứa queue chết, broker RabbitMQ (≥ 2.4.0) gửi về client một method `basic.cancel`. Client phải báo cho broker biết là mình hiểu extension này (thông qua "client capability" `consumer_cancel_notify` khi mở kết nối); các thư viện client hiện đại đều tự làm việc đó. Mỗi thư viện thể hiện sự kiện theo cách riêng: Pika bản cũ (như trong sách) ném exception ra khỏi `start_consuming()`; Pika bản mới cho phép đăng ký callback bằng `channel.add_on_cancel_callback(...)`; thư viện Java có `Consumer.handleCancel(...)`.

> 💡 **Ví dụ bổ sung (không có trong sách):** Bài toán "node A – node B":
> - Consumer nối vào **node A** (qua HAProxy), nhưng queue `orders` đã được tạo từ trước trên **node B**.
> - Node B chết. Kết nối TCP giữa consumer và node A **vẫn hoàn toàn bình thường** → không có `AMQPConnectionError`.
> - *RabbitMQ < 2.4.0*: consumer vẫn "chờ message" trên một queue đã biến mất → treo vô thời hạn, không báo lỗi gì.
> - *RabbitMQ ≥ 2.4.0*: node A gửi `basic.cancel` cho consumer → Pika ném exception trong `start_consuming()` → khối `except` ở (F) bắt được → vòng lặp `while True` chạy lại → `queue_declare("orders")` tạo lại queue (lúc này trên một node còn sống) → consume tiếp.
>
> Như vậy, **cùng một vòng lặp** `while True / try / except` xử lý được cả hai loại sự cố: mất kết nối và mất queue.

Khi đã có đủ các mảnh ghép, listing dưới đây cho thấy toàn bộ consumer trông như thế nào.

**Listing 6.4 – Consumer biết về cluster (cluster-aware consumer)**

```python
import sys, json, pika, time, traceback

def msg_rcvd(channel, method, header, body):                       # In ra và xác nhận (ack) message
    message = json.loads(body)
    print "Received: %(content)s/%(time)d" % message
    channel.basic_ack(delivery_tag=method.delivery_tag)

if __name__ == "__main__":
    AMQP_SERVER = sys.argv[1]                                      # Thiết lập thông số broker
    AMQP_PORT = int(sys.argv[2])

    creds_broker = pika.PlainCredentials("guest", "guest")         # Thiết lập kết nối tới broker
    conn_params = pika.ConnectionParameters( AMQP_SERVER,
                                             port=AMQP_PORT,
                                             virtual_host="/",
                                             credentials=creds_broker)

    while True:                                                    # Khi có lỗi, kết nối lại RabbitMQ
        try:
            conn_broker = pika.BlockingConnection(conn_params)     # Thiết lập kết nối tới RabbitMQ
            channel = conn_broker.channel()                        # Hành vi kết nối tùy biến

            channel.exchange_declare( exchange="cluster_test",     # Khai báo exchange,
                                      type="direct",               # queue, binding
                                      auto_delete=False)
            channel.queue_declare( queue="cluster_test",
                                   auto_delete=False)
            channel.queue_bind( queue="cluster_test",
                                exchange="cluster_test",
                                routing_key="cluster_test")

            print "Ready for testing!"
            channel.basic_consume( msg_rcvd,                       # Bắt đầu consume message
                                   queue="cluster_test",
                                   no_ack=False,
                                   consumer_tag="cluster_test")
            channel.start_consuming()
        except Exception, e:                                       # Bắt lỗi kết nối và in ra
            traceback.print_exc()
```

Chú thích (theo các mũi tên trong sách):
- **Print and acknowledge message** – In ra và xác nhận (ack) message (hàm `msg_rcvd`).
- **Broker settings** – Thông số broker (đọc địa chỉ và cổng từ tham số dòng lệnh).
- **Establish broker connection** – Thiết lập thông tin kết nối broker (credentials và connection parameters).
- **On fault, reconnect to RabbitMQ** – Khi có lỗi, kết nối lại tới RabbitMQ (`while True`).
- **Establish connection to RabbitMQ** – Thiết lập kết nối tới RabbitMQ.
- **Custom connection behavior** – Hành vi kết nối tùy biến (mở channel).
- **Declare exchange, queues, bindings** – Khai báo exchange, queue, binding.
- **Start consuming messages** – Bắt đầu tiêu thụ message.
- **Trap connection errors and print them** – Bắt các lỗi kết nối và in chúng ra.

Như bạn thấy, chuyển bất kỳ ứng dụng consumer nào thành dạng biết về cluster không hề khó. Nó chỉ đòi hỏi bạn hiểu chuyện gì xảy ra bên trong RabbitMQ khi một node hỏng, và điều chỉnh code để thích ứng với những hành vi đó. Giờ hãy chạy consumer và xem chuyện gì xảy ra:

```bash
$ python cluster_test_consumer.py localhost 5670
Ready for testing!
```

Hmmm… bạn đã kết nối được vào cluster thông qua load balancer và đã dựng xong kết cấu (hãy xem http://localhost:8100 để biết bạn đang được nối tới node nào trong cluster). Nhưng consumer của bạn chưa làm được gì thú vị cả. Thứ bạn cần là một **producer cho cluster** để đưa cho consumer chút nội dung mà hiển thị! Producer có vòng đời ngắn (*short-lived*) và không cần bất kỳ đoạn code xử lý sự cố cầu kỳ nào. Đó là vì mỗi lần chạy producer, nó lại thiết lập một kết nối mới từ đầu, nhờ đó load balancer có thể chọn một node mới đang hoạt động tốt. Listing dưới đây cho thấy producer của bạn trông như thế nào.

**Listing 6.5 – Producer biết về cluster (cluster-aware producer)**

```python
import sys, time, json, pika

AMQP_HOST = sys.argv[1]                                        # (B) Thiết lập kết nối tới broker
AMQP_PORT = int(sys.argv[2])
creds_broker = pika.PlainCredentials("guest", "guest")
conn_params = pika.ConnectionParameters(AMQP_HOST,
                                        port=AMQP_PORT,
                                        virtual_host = "/",
                                        credentials = creds_broker)

conn_broker = pika.BlockingConnection(conn_params)             # (C) Kết nối tới RabbitMQ và gửi message
channel = conn_broker.channel()

msg = json.dumps({"content": "Cluster Test!",
                  "time" : time.time()})
msg_props = pika.BasicProperties(content_type="application/json")

channel.basic_publish(body=msg,
                      exchange="cluster_test",
                      properties=msg_props,
                      routing_key="cluster_test")

print "Sent cluster test message."
```

Chú thích:
- **(B) Establish connection to broker** – Thiết lập thông tin kết nối tới broker.
- **(C) Connect to RabbitMQ and send message** – Kết nối tới RabbitMQ và gửi message.

Phần đầu tiên của producer (B) là thiết lập kết nối như bạn đã làm trước đây. Nó xác định địa chỉ IP và cổng của "server" RabbitMQ từ tham số dòng lệnh thứ nhất và thứ hai truyền vào producer. Bạn cũng dùng tài khoản `guest` có sẵn trong mọi bản cài Rabbit để xác thực. Sau đó bạn tạo một message JSON (C) để gửi cho consumer, chứa cụm từ `Cluster Test!` và dấu thời gian (*timestamp*) hiện tại. Việc cuối cùng bạn làm trước khi publish message là đặt header `content_type` của message, để consumer biết message của bạn được mã hóa bằng JSON. Cuối cùng bạn publish message và để nó "phóng vèo" qua Rabbit tới consumer. Trên dòng lệnh, nó trông thế nào?

> 📝 **Giải thích thêm (không có trong sách) — Điểm yếu của Producer trong Listing 6.5 & Cách viết Producer chịu lỗi chuẩn hiện đại (`pika 1.x` trên Python 3):**
>
> Trong Listing 6.5, tác giả giả định Producer chỉ là một script chạy một lần (*short-lived*) rồi thoát, và không bật **Publisher Confirms** (`channel.confirm_delivery()`). Trong hệ thống thực tế hiện nay:
> 1. Các dịch vụ backend (FastAPI, Django, Celery, microservices...) giữ **kết nối dài hạn (*long-lived connection*)** để publish hàng nghìn message mỗi giây thay vì mở kết nối TCP mới cho từng message.
> 2. Nếu không bật `channel.confirm_delivery()`, lệnh `basic_publish` chỉ đẩy byte vào bộ đệm socket hệ điều hành rồi trả về ngay — nếu đúng lúc đó node RabbitMQ đằng sau HAProxy bị sập, message sẽ bốc hơi mà Producer vẫn in ra `"Sent cluster test message."`!
>
> 💡 **Ví dụ bổ sung (không có trong sách) — Producer chuẩn Production bằng Python 3 + `pika 1.x` (Có Publisher Confirms, tự động kết nối lại đa node và Retry):**
> ```python
> import json
> import random
> import time
> import pika
> from pika.exceptions import AMQPError
>
> # Có thể truyền danh sách tất cả các node trong cluster (hoặc địa chỉ HAProxy)
> CLUSTER_NODES = [
>     pika.ConnectionParameters(host="127.0.0.1", port=5672, heartbeat=30),
>     pika.ConnectionParameters(host="127.0.0.1", port=5673, heartbeat=30),
>     pika.ConnectionParameters(host="127.0.0.1", port=5674, heartbeat=30),
> ]
>
> def publish_with_retry(exchange: str, routing_key: str, payload: dict, max_retries: int = 5):
>     body = json.dumps(payload).encode("utf-8")
>     props = pika.BasicProperties(
>         content_type="application/json",
>         delivery_mode=pika.DeliveryMode.Persistent,  # Persistent (2)
>     )
>     for attempt in range(1, max_retries + 1):
>         try:
>             nodes = CLUSTER_NODES.copy()
>             random.shuffle(nodes)  # Phân tán tải ngẫu nhiên giữa các node nếu không qua HAProxy
>             # Pika 1.x tự động thử lần lượt từng node trong danh sách nếu có node đang chết!
>             with pika.BlockingConnection(nodes) as conn:
>                 ch = conn.channel()
>                 ch.confirm_delivery()  # Bật Publisher Confirms để đảm bảo Quorum đã ghi xuống đĩa
>                 ch.basic_publish(
>                     exchange=exchange,
>                     routing_key=routing_key,
>                     body=body,
>                     properties=props,
>                     mandatory=True,
>                 )
>                 print(f"Đã gửi và được cluster xác nhận (Confirm): {payload}")
>                 return
>         except AMQPError as exc:
>             wait = min(2 ** (attempt - 1), 15)
>             print(f"[Lần {attempt}/{max_retries}] Lỗi khi publish ({exc}), thử lại sau {wait}s...")
>             time.sleep(wait)
>     raise RuntimeError("Không thể gửi message sau nhiều lần thử lại!")
>
> if __name__ == "__main__":
>     publish_with_retry("cluster_test", "cluster_test", {"content": "Cluster Test!", "time": int(time.time())})
> ```

```bash
$ python cluster_test_producer.py localhost 5670
Sent cluster test message.
```

Và nếu bạn quay lại xem consumer đang chạy…

```bash
$ python cluster_test_consumer.py localhost 5670
Ready for testing!
Received: Cluster Test!/1301531152
```

Trúng phóc! Message đã được producer đưa vào một node của cluster và được consumer nhận ở một node khác! Giờ hãy xem bằng chứng thực sự (*the real proof in the pudding* — "muốn biết bánh pudding ngon không thì phải ăn thử") và khởi động lại node đang kết nối với consumer.

Trước tiên, dùng http://localhost:8100 để tìm xem consumer của bạn đang kết nối tới node nào trong cluster. Đó sẽ là node có số **1** trong cột **Cur** thuộc nhóm **Sessions** trên trang thống kê HAProxy. Sau đó, từ thư mục cài đặt RabbitMQ, chạy `./sbin/rabbitmqctl -n node_name stop_app`, trong đó `node_name` là tên node bạn vừa xác định được trên trang thống kê HAProxy.

```bash
$ ./sbin/rabbitmqctl -n node_name stop_app
```

Giờ nếu quay lại xem consumer, bạn sẽ thấy vài lỗi kết nối, tiếp theo là một lần kết nối lại thành công vào cluster:

```bash
$ python cluster_test_consumer.py localhost 5670
Ready for testing!
Traceback (most recent call last):
  File "cluster_test_consumer.py", line 57, in <module>
    channel.start_consuming()
  File "/Library/Python/2.6/site-packages/pika-0.9.6-py2.6.egg/pika/adapters/blocking_connection.py", line 293, in start_consuming
    self.transport.connection.process_data_events()
  File "/Library/Python/2.6/site-packages/pika-0.9.6-py2.6.egg/pika/adapters/blocking_connection.py", line 87, in process_data_events
    raise AMQPConnectionError
AMQPConnectionError
Ready for testing!
```

Dòng `Ready for testing!` thứ hai cho thấy consumer của bạn đã hồi phục thành công sau khi node hỏng và đã kết nối lại vào cluster. Giờ nếu bạn publish một message mới vào cluster, bạn sẽ thấy nó được in ra bởi consumer vừa kết nối lại:

```bash
$ python cluster_test_consumer.py localhost 5670
Ready for testing!
Traceback (most recent call last):
  File "cluster_test_consumer.py", line 57, in <module>
    channel.start_consuming()
  File "/Library/Python/2.6/site-packages/pika-0.9.6-py2.6.egg/pika/adapters/blocking_connection.py", line 293, in start_consuming
    self.transport.connection.process_data_events()
  File "/Library/Python/2.6/site-packages/pika-0.9.6-py2.6.egg/pika/adapters/blocking_connection.py", line 87, in process_data_events
    raise AMQPConnectionError
AMQPConnectionError
Ready for testing!
Received: Cluster Test!/1301531677
```

Mọi thứ đều chạy! Giờ bạn đã có một hệ thống RabbitMQ được cluster hóa đầy đủ, kèm một load balancer lo việc chọn node, cùng consumer và producer biết về cluster, có thể **tiếp tục "chạy băng băng"** (*keep trucking*) ngay cả khi một node trong cluster chết.

Đến đây bạn có thể nghỉ ngơi, hài lòng vì đã nắm vững tính năng clustering tích hợp của RabbitMQ. Nhưng clustering tích hợp không bao phủ mọi tình huống sử dụng. Ví dụ, sẽ ra sao nếu bạn **tuyệt đối không thể chấp nhận rủi ro mất bất kỳ message nào** trong một durable queue (*queue bền vững* — queue được lưu xuống đĩa và tồn tại qua lần khởi động lại broker) khi một node hỏng? Các phiên bản RabbitMQ hiện tại sẽ khôi phục durable queue cùng nội dung của nó khi node của queue đó tham gia lại cluster, nhưng các phiên bản trước 1.8.0 thì không. Điều này đòi hỏi một cách tiếp cận khác: dùng các RabbitMQ server độc lập theo kiểu **chủ động và dự phòng** (*active and standby*), cho phép bạn failover mà không mất nội dung của queue cũ.

> 💡 **Ví dụ bổ sung (không có trong sách):** Một phiên bản "lịch sự" hơn của listing 6.4 cho môi trường thật — viết bằng Python 3 với Pika 1.x (API đã đổi so với sách): bắt đúng loại lỗi, chờ một chút trước khi kết nối lại để không "dội bom" cluster khi tất cả node đều chết, và bật heartbeat để HAProxy không cắt kết nối nhàn rỗi.
> ```python
> import json, time, traceback
> import pika
> from pika.exceptions import AMQPConnectionError, ChannelClosedByBroker, ConsumerCancelled
>
> params = pika.ConnectionParameters("localhost", port=5670, virtual_host="/",
>                                    credentials=pika.PlainCredentials("guest", "guest"),
>                                    heartbeat=30)   # nhỏ hơn timeout client/server 120s của HAProxy
>
> def msg_rcvd(ch, method, props, body):
>     message = json.loads(body)
>     print("Received: %(content)s/%(time)d" % message)
>     ch.basic_ack(delivery_tag=method.delivery_tag)
>
> delay = 1
> while True:
>     try:
>         conn = pika.BlockingConnection(params)
>         ch = conn.channel()
>         ch.exchange_declare(exchange="cluster_test", exchange_type="direct", auto_delete=False)
>         ch.queue_declare(queue="cluster_test", auto_delete=False)
>         ch.queue_bind(queue="cluster_test", exchange="cluster_test", routing_key="cluster_test")
>         print("Ready for testing!")
>         delay = 1                                   # kết nối thành công -> reset thời gian chờ
>         ch.basic_consume(queue="cluster_test", on_message_callback=msg_rcvd,
>                          auto_ack=False, consumer_tag="cluster_test")
>         ch.start_consuming()
>     except (AMQPConnectionError, ChannelClosedByBroker, ConsumerCancelled):
>         traceback.print_exc()
>         time.sleep(delay)
>         delay = min(delay * 2, 30)                  # exponential backoff: 1, 2, 4, ... tối đa 30s
>     except KeyboardInterrupt:
>         break
> ```
> Ý tưởng cốt lõi vẫn y hệt sách: **mỗi lần có lỗi thì làm lại tất cả** — kết nối, channel, exchange, queue, binding, consume.

> 📝 **Giải thích thêm (không có trong sách) — Kết nối đa node không cần Load Balancer, Automatic Topology Recovery & Tối ưu hóa Locality trong RabbitMQ hiện đại:**
>
> So với thời điểm năm 2012 (khi Pika 0.9.6 chỉ nhận 1 địa chỉ host duy nhất và hầu hết các thư viện client đều bắt lập trình viên tự viết vòng lặp `while True` để dựng lại exchange/queue/binding), hệ sinh thái RabbitMQ hiện nay đã tiến một bước rất dài:
>
> 1. **Truyền danh sách nhiều node trực tiếp vào `pika.BlockingConnection` (Không bắt buộc phải có HAProxy):**
>    - Ở mục 6.1, sách nói rằng nếu không có Load Balancer thì ứng dụng phải tự viết vòng lặp dò từng IP. Trong **`pika 1.x`**, bạn có thể truyền trực tiếp một `list` các `pika.ConnectionParameters` vào `pika.BlockingConnection([params_node1, params_node2, params_node3])`. Nếu node đầu không phản hồi, Pika tự động chuyển sang node tiếp theo trong danh sách!
> 2. **Tự động khôi phục Kết nối & Topology (*Automatic Connection & Topology Recovery*):**
>    - Hầu hết các thư viện client chính thức hiện đại đã tích hợp sẵn bộ máy **Automatic Recovery** ngay trong thư viện: khi đứt kết nối hoặc node sập, thư viện tự động kết nối sang node khác trong danh sách, tự mở lại channel, **tự khai báo lại toàn bộ exchange, queue, binding đã tạo trước đó** và tự đăng ký lại `basic.consume`!
>
> | Thư viện / Ngôn ngữ | Truyền danh sách nhiều Node | Tự động Reconnect | Tự động dựng lại Topology (Exchange/Queue/Binding/Consumer) |
> |---|---|---|---|
> | **Pika 0.9.6 (Python 2 - trong sách)** | Không (chỉ nhận 1 `ConnectionParameters`) | Không (phải tự viết `while True`) | Không (phải tự gọi lại `declare` / `bind`) |
> | **Pika 1.x (Python 3 - đồng bộ)** | **Có** (`BlockingConnection([p1, p2, p3])`) | Không (vẫn dùng vòng lặp `while True` ngoài) | Không (tự gọi lại trong vòng lặp `while True`) |
> | **aio-pika (Python 3 `asyncio`)** | **Có** (hoặc qua URL / Load Balancer) | **Có sẵn** (`connect_robust()`) | **Có sẵn** (`RobustChannel` tự động dựng lại toàn bộ topology & consumer!) |
> | **Java (`amqp-client` 4.0+ / 5.x)** | **Có** (`factory.newConnection(List<Address>)`) | **Bật mặc định** (`automaticRecoveryEnabled = true`) | **Bật mặc định** (`topologyRecoveryEnabled = true`) |
> | **.NET (`RabbitMQ.Client` 6.x / 7.x)** | **Có** (`CreateConnection(IList<AmqpTcpEndpoint>)`) | **Bật mặc định** (`AutomaticRecoveryEnabled = true`) | **Bật mặc định** (`TopologyRecoveryEnabled = true`) |
>
> 3. **Lưu ý về Kết nối tới Quorum Queue Leader (*Queue Leader Locality*):**
>    - Như câu nói ở đoạn cuối mục 6.2 trong sách (*"sẽ ra sao nếu bạn tuyệt đối không thể chấp nhận rủi ro mất bất kỳ message nào trong một durable queue khi một node hỏng?"* — thời sách viết tác giả phải hẹn sang Chương 7 dùng Pacemaker/DRBD phức tạp), ngày nay với **Quorum Queue (`x-queue-type: quorum`)**, queue bền vững vẫn hoạt động liên tục ngay khi 1 node sập (vì Follower trên node còn sống lập tức bầu làm Leader mới chỉ sau vài trăm mili-giây).
>    - Tuy nhiên, nếu Consumer/Producer kết nối qua HAProxy vào **Node 3**, mà Raft Leader của Quorum Queue lại nằm trên **Node 1**, thì mọi message phải đi thêm 1 chặng mạng nội bộ (*intra-cluster hop*) giữa Node 3 và Node 1. Để tối ưu hiệu năng:
>      - Hãy dùng lệnh `rabbitmq-queues rebalance "all"` hoặc đặt thuộc tính `x-queue-leader-locator: balanced` để các Raft Leader phân bổ đều khắp các node.
>      - Với **RabbitMQ Streams** (sử dụng giao thức nhị phân cổng `5552` và thư viện `rstream` / `stream-java-client`), client tự động hỏi cluster để kết nối **trực tiếp** tới đúng node chứa Stream Leader (cho Producer) và các node Replica (cho Consumer) mà không cần đi vòng qua HAProxy!
>
> 💡 **Ví dụ bổ sung (không có trong sách) — Viết Consumer bất đồng bộ tự động khôi phục 100% Connection & Topology bằng `aio-pika` (Python 3) kết hợp Quorum Queue:**
> ```python
> import asyncio
> import json
> import aio_pika
>
> async def main():
>     # connect_robust tự động kết nối lại VÀ tự động tạo lại Exchange, Queue, Binding, Consumer khi node sập!
>     connection = await aio_pika.connect_robust(
>         "amqp://guest:guest@localhost:5670/",
>         reconnect_interval=2.0,
>     )
>     async with connection:
>         channel = await connection.channel()
>         await channel.set_qos(prefetch_count=10)
>
>         # Khai báo Exchange và Quorum Queue bền vững (tự động được ghi nhớ để khôi phục khi reconnect)
>         exchange = await channel.declare_exchange(
>             "cluster_test", aio_pika.ExchangeType.DIRECT, durable=True
>         )
>         queue = await channel.declare_queue(
>             "cluster_test_quorum",
>             durable=True,
>             arguments={"x-queue-type": "quorum"},
>         )
>         await queue.bind(exchange, routing_key="cluster_test")
>
>         print("Sẵn sàng nhận message (Tự động khôi phục Connection & Topology với aio-pika)...")
>         async with queue.iterator() as queue_iter:
>             async for message in queue_iter:
>                 async with message.process():
>                     data = json.loads(message.body.decode())
>                     print(f"Đã nhận: {data['content']} / {data['time']}")
>
> if __name__ == "__main__":
>     asyncio.run(main())
> ```
>
> ---
>
> 📝 **Giải thích thêm (không có trong sách) — Single Active Consumer (`x-single-active-consumer: true` từ RabbitMQ 3.8+): Giải pháp chuẩn cho Consumer Dự phòng (Failover) bảo toàn thứ tự message!**
>
> Trong chương này, tác giả hướng dẫn cách giúp **một** tiến trình consumer tự kết nối lại khi node RabbitMQ bị sập. Nhưng nếu chính **máy chủ chạy tiến trình consumer bị sập** thì sao? Bạn bắt buộc phải chạy ít nhất 2–3 tiến trình consumer trên các máy khác nhau để dự phòng!
>
> Tuy nhiên, nếu bạn chạy 2 tiến trình consumer cùng lắng nghe một queue theo cách thông thường, RabbitMQ sẽ chia bài *round-robin* cho cả 2 tiến trình chạy song song. Với những bài toán đòi hỏi **bảo toàn nghiêm ngặt thứ tự sự kiện (FIFO)** (ví dụ: sự kiện `Tạo tài khoản` → `Nạp tiền` → `Thanh toán` của cùng một đơn hàng), việc 2 consumer xử lý song song sẽ làm đảo lộn thứ tự!
>
> Từ **RabbitMQ 3.8 trở đi**, tính năng **Single Active Consumer (SAC)** giải quyết triệt để bài toán này chỉ bằng một tham số khi khai báo queue: `arguments={"x-single-active-consumer": True}`:
>
> ```mermaid
> flowchart LR
>     Q["Quorum Queue<br/>(x-single-active-consumer: true)"]
>     C1["Consumer #1 (ACTIVE)<br/>Nhận 100% message theo đúng thứ tự FIFO"]
>     C2["Consumer #2 (STANDBY - Dự phòng nóng)<br/>Đã kết nối sẵn nhưng đứng chờ"]
>     C3["Consumer #3 (STANDBY - Dự phòng nóng)<br/>Đã kết nối sẵn nhưng đứng chờ"]
>     Q ==>|"Giao message"| C1
>     Q -.->|"Chờ tự động thay thế<br/>ngay khi #1 đứt kết nối"| C2
>     Q -.->|"Chờ dự phòng"| C3
> ```
>
> - Bạn có thể khởi chạy 3 tiến trình Consumer (#1, #2, #3) kết nối vào 3 node khác nhau của cluster.
> - RabbitMQ chỉ chọn **đúng 1 Consumer duy nhất (#1)** làm *Single Active Consumer* để nhận toàn bộ message theo thứ tự FIFO nghiêm ngặt. Consumer #2 và #3 vẫn giữ kết nối AMQP nóng (*hot standby*) nhưng không nhận message nào.
> - Ngay khoảnh khắc Consumer #1 bị crash hoặc node mà Consumer #1 đang nối vào bị sập, RabbitMQ **tự động chuyển quyền Active sang Consumer #2 tức thì** mà không cần đợi khởi động lại tiến trình!
>
> 💡 **Ví dụ bổ sung (không có trong sách) — Khai báo Quorum Queue kèm Single Active Consumer trong Python 3 (`pika 1.x`):**
> ```python
> import pika
>
> conn = pika.BlockingConnection(pika.ConnectionParameters("localhost", port=5670, heartbeat=30))
> ch = conn.channel()
>
> # Kết hợp Quorum Queue (chống sập node broker) + Single Active Consumer (chống sập consumer & giữ thứ tự FIFO)
> ch.queue_declare(
>     queue="ordered_financial_events",
>     durable=True,
>     arguments={
>         "x-queue-type": "quorum",
>         "x-single-active-consumer": True,  # Chỉ 1 consumer active tại mọi thời điểm, các consumer khác chờ dự phòng!
>     },
> )
> ```

---

## 6.3 Tóm tắt (Summary)

Cluster hóa RabbitMQ mới chỉ là một nửa những gì bạn cần để có một hạ tầng nhắn tin bền bỉ. Nửa còn lại phụ thuộc vào ứng dụng của bạn. Giờ bạn đã biết cách viết ứng dụng chịu được việc node trong cluster hỏng, bằng cách kết nối lại tới node mới và dựng lại kết cấu (*fabric*) mà ứng dụng cần để tiếp tục hoạt động. Quan trọng không kém, giờ bạn có thể dựng và sử dụng một load balancer làm "chất keo" — thứ xác định node nào trong cluster đã hỏng và định tuyến ứng dụng tới các node mới một cách thông minh khi chúng kết nối lại.

Những kỹ thuật này, khi kết hợp với một RabbitMQ cluster, cho bạn một hạ tầng nhắn tin vững chắc, có thể hứng chịu sự cố mà ứng dụng vẫn không "lỡ nhịp". Nhưng vẫn còn vài câu hỏi chưa có lời đáp về việc làm cho Rabbit có tính sẵn sàng cao (*highly available*). Ví dụ, làm sao thiết kế một hạ tầng Rabbit mà khi một node sập thì các durable queue trên node đó không trở nên không khả dụng với ứng dụng? Và làm sao thiết kế một kiến trúc Rabbit có thể sống sót khi mất cả một trung tâm dữ liệu (*data center*) cùng các cluster trong đó? Để có câu trả lời cho những câu hỏi đó, chúng ta cần "vác xẻng" ra và đào sâu vào vài cái "hang thỏ" (*warrens* — chơi chữ: "warren" vừa là hang thỏ, vừa ám chỉ các cấu hình RabbitMQ sẽ được trình bày ở chương sau).

> 💡 **Ví dụ bổ sung (không có trong sách):** Bài tập tự kiểm tra.
> 1. Vì sao producer trong listing 6.5 không cần vòng lặp `while True`? → *Vì mỗi lần chạy nó mở kết nối mới; HAProxy sẽ chỉ chuyển tới node đang UP.*
> 2. Consumer kết nối vào node A, queue nằm trên node B, node B chết. Với RabbitMQ 2.4.0+, điều gì kích hoạt việc kết nối lại? → *Cancellation notification (`basic.cancel`), được Pika thể hiện bằng exception.*
> 3. Với `check inter 5000 rise 2 fall 3`, node chết bị loại sau khoảng bao lâu? → *~15 giây (3 lần × 5 giây).*
> 4. Sau khi failover, vì sao phải khai báo lại cả exchange dù dùng cluster tích hợp (nơi exchange được sao chép)? → *Vì nguyên tắc "không giả định gì"; và trong mô hình active/standby (chương 7) exchange có thể không còn. Khai báo lại là idempotent nên không có hại.*

---

## Bảng thuật ngữ của chương

| Thuật ngữ (Anh) | Nghĩa tiếng Việt | Giải thích ngắn |
|---|---|---|
| Load balancer | Bộ cân bằng tải | Thiết bị/phần mềm đưa ra một địa chỉ IP duy nhất và phân phối kết nối tới nhiều server phía sau. |
| Load balancing | Cân bằng tải | Việc dàn đều kết nối/tải giữa nhiều server. |
| Frontend IP | IP phía trước | Địa chỉ mà client kết nối tới trên load balancer (ở đây `127.0.0.1:5670`). |
| Backend | Server phía sau | Server thật nhận kết nối do load balancer chuyển tới (ở đây là các node RabbitMQ). |
| Transparently proxy | Chuyển tiếp trong suốt | Chuyển kết nối mà client không hề nhận ra. |
| Hardware appliance | Thiết bị phần cứng | Load balancer dạng thiết bị mạng chuyên dụng. |
| Single point of failure | Điểm hỏng duy nhất | Thành phần mà nếu hỏng thì cả hệ thống ngừng hoạt động. |
| Layer 4 load balancing | Cân bằng tải tầng 4 | Cân bằng ở mức TCP, không phân tích nội dung giao thức ứng dụng. |
| HAProxy | HAProxy | Load balancer phần mềm miễn phí, ổn định, dùng trong chương. |
| epoll / kqueues | epoll / kqueues | Cơ chế thông báo sự kiện mạng của Linux 2.6 / FreeBSD, thay cho việc thăm dò định kỳ. |
| Poll | Thăm dò | Liên tục kiểm tra theo chu kỳ xem có dữ liệu mới không. |
| Round-robin | Xoay vòng | Thuật toán chia lần lượt từng kết nối cho từng backend. |
| Health check | Kiểm tra sức khỏe | Phép thử định kỳ xem backend còn hoạt động không (`check inter`). |
| rise / fall | rise / fall | Số lần health check thành công để coi là sống lại / thất bại để coi là đã chết. |
| Backup server | Server dự phòng | Backend chỉ được dùng khi mọi backend chính đều chết. |
| Statistics page | Trang thống kê | Trang web của HAProxy cho thấy trạng thái và số kết nối của từng backend. |
| Cluster | Cụm | Nhiều node RabbitMQ cùng hoạt động như một hệ thống. |
| Node | Nút | Một tiến trình RabbitMQ server trong cluster. |
| Failover | Chuyển đổi dự phòng | Chuyển sang node/server khác khi node hiện tại hỏng. |
| Resilience / resilient | Khả năng chịu lỗi / bền bỉ | Khả năng tiếp tục hoạt động khi có sự cố. |
| Active/standby | Chủ động/dự phòng | Mô hình một server chạy chính, một server chờ sẵn để thay thế (chương 7). |
| Messaging fabric | Kết cấu nhắn tin | Tập hợp exchange, queue và binding mà ứng dụng cần. |
| Cluster-aware | Biết về cluster | Ứng dụng được viết để xử lý sự cố node và kết nối lại. |
| Channel | Kênh | Kết nối ảo bên trong một connection AMQP. |
| Consumption loop / subscription | Vòng lặp tiêu thụ / đăng ký tiêu thụ | Cơ chế consumer đăng ký nhận message từ queue (`basic_consume`). |
| Unhandled exception | Exception không được xử lý | Lỗi không được bắt, khiến chương trình crash. |
| Exception handler | Bộ xử lý exception | Khối `try...except` bắt và xử lý lỗi. |
| Cancellation notification | Thông báo hủy | Phần mở rộng AMQP của RabbitMQ 2.4.0: báo cho consumer khi subscription bị chấm dứt không phải do nó tự hủy. |
| Durable queue | Queue bền vững | Queue được lưu trên đĩa, tồn tại qua lần khởi động lại broker. |
| Short-lived producer | Producer vòng đời ngắn | Producer chạy, gửi message rồi thoát; mỗi lần chạy mở kết nối mới. |
| Highly available | Sẵn sàng cao | Hệ thống hầu như luôn phục vụ được, kể cả khi có thành phần hỏng. |
| Data center | Trung tâm dữ liệu | Cơ sở chứa hạ tầng máy chủ; mất cả data center là kịch bản sự cố lớn nhất. |
| Warren | Hang thỏ | Ẩn dụ của tác giả cho các kiến trúc RabbitMQ sẽ bàn ở chương sau. |
