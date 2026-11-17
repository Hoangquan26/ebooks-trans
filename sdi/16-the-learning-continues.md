# CHƯƠNG 16: HÀNH TRÌNH HỌC HỎI VẪN TIẾP TỤC (CHAPTER 16: THE LEARNING CONTINUES)

Việc thiết kế những hệ thống tốt đòi hỏi nhiều năm tích lũy kiến thức. Một con đường tắt là đi sâu vào các kiến trúc hệ thống trong thế giới thực. Dưới đây là tập hợp các tài liệu đọc hữu ích. Chúng tôi đặc biệt khuyên bạn nên chú ý đến cả những nguyên lý chung lẫn các công nghệ nền tảng bên dưới. Việc nghiên cứu từng công nghệ và hiểu rõ nó giải quyết vấn đề gì là một cách tuyệt vời để củng cố nền tảng kiến thức và trau dồi quy trình thiết kế của bạn.

## Các hệ thống thực tế (Real-world systems)

Những tài liệu sau đây có thể giúp bạn hiểu các ý tưởng thiết kế tổng quan về kiến trúc hệ thống thực tế đằng sau các công ty khác nhau.

- **Facebook Timeline: Brought To You By The Power Of Denormalization:** https://goo.gl/FCNrbm
- **Scale at Facebook:** https://goo.gl/NGTdCs
- **Building Timeline: Scaling up to hold your life story:** https://goo.gl/8p5wDV
- **Erlang at Facebook (Facebook chat):** https://goo.gl/zSLHrj
- **Facebook Chat:** https://goo.gl/qzSiWC
- **Finding a needle in Haystack: Facebook’s photo storage:** https://goo.gl/edj4FL
- **Serving Facebook Multifeed: Efficiency, performance gains through redesign:** https://goo.gl/adFVMQ
- **Scaling Memcache at Facebook:** https://goo.gl/rZiAhX
- **TAO: Facebook’s Distributed Data Store for the Social Graph:** https://goo.gl/Tk1DyH
- **Amazon Architecture:** https://goo.gl/k4feoW
- **Dynamo: Amazon’s Highly Available Key-value Store:** https://goo.gl/C7zxDL
- **A 360 Degree View Of The Entire Netflix Stack:** https://goo.gl/rYSDTz
- **It’s All A/Bout Testing: The Netflix Experimentation Platform:** https://goo.gl/agbA4K
- **Netflix Recommendations: Beyond the 5 stars (Part 1):** https://goo.gl/A4FkYi
- **Netflix Recommendations: Beyond the 5 stars (Part 2):** https://goo.gl/XNPMXm
- **Google Architecture:** https://goo.gl/dvkDiY
- **The Google File System (Google Docs):** https://goo.gl/xj5n9R
- **Differential Synchronization (Google Docs):** https://goo.gl/9zqG7x
- **YouTube Architecture:** https://goo.gl/mCPRUF
- **Seattle Conference on Scalability: YouTube Scalability:** https://goo.gl/dH3zYq
- **Bigtable: A Distributed Storage System for Structured Data:** https://goo.gl/6NaZca
- **Instagram Architecture: 14 Million Users, Terabytes Of Photos, 100s Of Instances, Dozens Of Technologies:** https://goo.gl/s1VcW5
- **The Architecture Twitter Uses To Deal With 150M Active Users:** https://goo.gl/EwvfRd
- **Scaling Twitter: Making Twitter 10000 Percent Faster:** https://goo.gl/nYGC1k
- **Announcing Snowflake (Snowflake là một dịch vụ mạng để tạo các số ID duy nhất ở quy mô lớn với một số đảm bảo đơn giản):** https://goo.gl/GzVWYm
- **Timelines at Scale:** https://goo.gl/8KbqTy
- **How Uber Scales Their Real-Time Market Platform:** https://goo.gl/kGZuVy
- **Scaling Pinterest:** https://goo.gl/KtmjW3
- **Pinterest Architecture Update:** https://goo.gl/w6rRsf
- **A Brief History of Scaling LinkedIn:** https://goo.gl/8A1Pi8
- **Flickr Architecture:** https://goo.gl/dWtgYa
- **How We've Scaled Dropbox:** https://goo.gl/NjBDtC
- **The WhatsApp Architecture Facebook Bought For $19 Billion:** https://bit.ly/2AHJnFn

> 📝 **Giải thích thêm (không có trong sách):** Khi đọc các bài viết về kiến trúc thực tế (case study) ở trên, bạn nên đặt 4 câu hỏi trọng tâm để rút ra bài học nhanh nhất:
> 1. **Nút thắt cổ chai (bottleneck) là gì?** (Ví dụ: I/O đĩa khi lưu hàng tỷ ảnh nhỏ trong Haystack, hay độ trễ đọc đồ thị xã hội trong TAO).
> 2. **Tại sao giải pháp tiêu chuẩn không còn đáp ứng được?** (Ví dụ: dùng RDBMS chuẩn với chuẩn hóa dữ liệu bị chậm khi join quá nhiều).
> 3. **Họ đã đánh đổi (trade-off) điều gì?** (Ví dụ: phi chuẩn hóa dữ liệu - denormalization - chấp nhận tốn dung lượng lưu trữ và phức tạp khi cập nhật để đổi lấy tốc độ đọc cực nhanh).
> 4. **Nếu thiết kế lại từ đầu ở quy mô nhỏ hơn, có cần dùng giải pháp đó ngay không?** (Tránh bẫy over-engineering khi hệ thống chưa đạt đến quy mô tương ứng).

## Blog kỹ thuật của các công ty (Company engineering blogs)

Nếu bạn sắp phỏng vấn với một công ty nào đó, việc đọc blog kỹ thuật của họ và làm quen với các công nghệ cũng như hệ thống được áp dụng và triển khai tại đó là một ý tưởng rất hay. Bên cạnh đó, các blog kỹ thuật mang lại những góc nhìn vô giá về những lĩnh vực nhất định. Đọc chúng thường xuyên có thể giúp chúng ta trở thành những kỹ sư giỏi hơn.

Dưới đây là danh sách các blog kỹ thuật của những công ty lớn và startup nổi tiếng:

- **Airbnb:** https://medium.com/airbnb-engineering
- **Amazon:** https://developer.amazon.com/blogs
- **Asana:** https://blog.asana.com/category/eng
- **Atlassian:** https://developer.atlassian.com/blog
- **Bittorrent:** http://engineering.bittorrent.com
- **Cloudera:** https://blog.cloudera.com
- **Docker:** https://blog.docker.com
- **Dropbox:** https://blogs.dropbox.com/tech
- **eBay:** http://www.ebaytechblog.com
- **Facebook:** https://code.facebook.com/posts
- **GitHub:** https://githubengineering.com
- **Google:** https://developers.googleblog.com
- **Groupon:** https://engineering.groupon.com
- **Highscalability:** http://highscalability.com
- **Instacart:** https://tech.instacart.com
- **Instagram:** https://engineering.instagram.com
- **Linkedin:** https://engineering.linkedin.com/blog
- **Mixpanel:** https://mixpanel.com/blog
- **Netflix:** https://medium.com/netflix-techblog
- **Nextdoor:** https://engblog.nextdoor.com
- **PayPal:** https://www.paypal-engineering.com
- **Pinterest:** https://engineering.pinterest.com
- **Quora:** https://engineering.quora.com
- **Reddit:** https://redditblog.com
- **Salesforce:** https://developer.salesforce.com/blogs/engineering
- **Shopify:** https://engineering.shopify.com
- **Slack:** https://slack.engineering
- **Soundcloud:** https://developers.soundcloud.com/blog
- **Spotify:** https://labs.spotify.com
- **Stripe:** https://stripe.com/blog/engineering
- **System design primer:** https://github.com/donnemartin/system-design-primer
- **Twitter:** https://blog.twitter.com/engineering/en_us.html
- **Thumbtack:** https://www.thumbtack.com/engineering
- **Uber:** http://eng.uber.com
- **Yahoo:** https://yahooeng.tumblr.com
- **Yelp:** https://engineeringblog.yelp.com
- **Zoom:** https://medium.com/zoom-developer-blog

> 💡 **Ví dụ bổ sung (không có trong sách):** Giả sử bạn sắp phỏng vấn vị trí Backend/System Design tại một công ty thương mại điện tử như Shopify hoặc eBay. Trước buổi phỏng vấn, bạn có thể tìm đọc 2–3 bài gần nhất trên blog kỹ thuật của họ về cách họ xử lý lưu lượng truy cập tăng đột biến trong dịp Black Friday / Flash Sale (ví dụ: cơ chế rate limiting, phân mảnh cơ sở dữ liệu theo tenant, hoặc hàng đợi xử lý đơn hàng). Điều này giúp bạn nắm được "ngôn ngữ kỹ thuật" và các bài toán thực tế mà đội ngũ của họ đang đối mặt.

# LỜI KẾT (AFTERWORD)

Chúc mừng bạn! Bạn đã đi đến cuối cuốn cẩm nang phỏng vấn này. Bạn đã tích lũy được những kỹ năng và kiến thức để thiết kế các hệ thống. Không phải ai cũng có đủ kỷ luật để học những gì bạn đã học. Hãy dành một chút thời gian để tự khen ngợi bản thân. Sự chăm chỉ của bạn sẽ được đền đáp xứng đáng.

Chinh phục công việc mơ ước là một hành trình dài và đòi hỏi nhiều thời gian cũng như công sức. Có công mài sắt, có ngày nên kim. Chúc bạn mọi điều may mắn nhất!

Cảm ơn bạn đã mua và đọc cuốn sách này. Nếu không có những độc giả như bạn, tác phẩm của chúng tôi sẽ không thể tồn tại. Chúng tôi hy vọng bạn đã tận hưởng cuốn sách!

Nếu bạn không phiền, xin hãy đánh giá cuốn sách này trên Amazon: https://tinyurl.com/y7d3ltbc. Điều đó sẽ giúp tôi thu hút thêm nhiều độc giả tuyệt vời như bạn.

Vui lòng đăng ký danh sách email của chúng tôi nếu bạn muốn nhận thông báo khi có các chương mới: https://bit.ly/3dtIcsE

Nếu bạn có nhận xét hoặc câu hỏi về cuốn sách này, hãy thoải mái gửi email cho chúng tôi theo địa chỉ systemdesigninsider@gmail.com. Bên cạnh đó, nếu bạn phát hiện bất kỳ lỗi nào, xin hãy cho chúng tôi biết để chúng tôi có thể chỉnh sửa trong ấn bản tiếp theo. Xin cảm ơn!

## Bảng thuật ngữ chương (Chapter Glossary)

| Thuật ngữ (English) | Tiếng Việt | Giải thích ngắn gọn |
| --- | --- | --- |
| Denormalization | Phi chuẩn hóa | Kỹ thuật thêm dữ liệu dư thừa vào cơ sở dữ liệu nhằm giảm phép join và tăng tốc độ đọc |
| Distributed Data Store | Kho lưu trữ dữ liệu phân tán | Hệ thống lưu trữ dữ liệu trên nhiều máy chủ vật lý hoặc nút mạng khác nhau để tăng khả năng mở rộng và chịu lỗi |
| Key-value Store | Kho lưu trữ khóa-giá trị | Cơ sở dữ liệu NoSQL lưu trữ dữ liệu dưới dạng các cặp khóa (key) và giá trị (value) để truy xuất nhanh |
| Social Graph | Đồ thị xã hội | Mô hình biểu diễn người dùng (đỉnh) và các mối quan hệ giữa họ (cạnh) trong mạng xã hội |
| Experimentation Platform | Nền tảng thử nghiệm | Hệ thống hỗ trợ chạy thử nghiệm (như A/B testing) để đo lường tác động của tính năng mới lên người dùng |
| Differential Synchronization | Đồng bộ hóa vi sai | Thuật toán đồng bộ tài liệu cộng tác theo thời gian thực bằng cách chỉ truyền và hợp nhất các phần thay đổi |
| Engineering Blog | Blog kỹ thuật | Kênh chia sẻ kiến thức, kiến trúc và bài toán thực tế từ đội ngũ kỹ sư của các công ty công nghệ |
