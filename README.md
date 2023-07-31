### Vấn đề

- Tìm solution về infrastructure cho bài toán lượng truy cập cao vào 1 link bài báo và user trên toàn cầu để hệ thống chạy ổn định, chịu được lượng tải cao trên platform AWS.

### Phân tích yêu cầu

- Với data: 1 bài báo với các nội dung tĩnh, không có sự thay đổi theo thời gian => có thể store ở S3 hoặc RDS hoặc DB ngay trên server.
- Về request: Lượng request cao trong 1 khoảng thời gian, với nhiều region => phải giảm được độ trễ khi call api ở các region khác nhau vậy nên cần CDN như AWS Cloufront.
- Về server: Do data chỉ là tĩnh và nhẹ nên có thể store data trên DB của server và đảm bảo response time phải thấp nhất có thể khi get từ DB ra và trả cho client => sử dụng các kỹ thuật caching như redis...
- Về domain: Không yêu cầu cụ thể nên có thể dùng các platform domain như Cloudflare...

### Solution

- Về server và DB: các nội dung tĩnh về bài báo như html, text có thể store ở DB, còn image thì store trên s3 và ở DB store path của image, khi call api get 1 bài báo, server sẽ trả text hoặc html, kèm theo là image key trên s3 cho client hoặc link tới image bằng domain của Cloudfront
- Về FE: khi có được text hoặc html kèm theo key của image, sử dụng domain Cloudfront để get image từ s3 và hiển thị trên FE, hoặc hiển thị luôn image bằng link qua domain Cloudfront nhận đc từ BE.
- Về CDN: sử dụng Cloudfront để get image từ s3 cho user toàn cầu, giảm độ trễ về khác region và sử dụng cloudfront cho server để cache response cho client ở nhiều nơi trên thế giới, giúp giảm độ trễ.

### Update

- Không cần sử dụng Cloudfront để cache cho api, để xử lý chịu tải cao, ta sử dụng LB và cache ở redis databse và dùng cơ chế CDC để instance không tác động trực tiếp vào DB mà phải qua CDC để cache và lấy data

### System design

[![System design](/abc.jpg "System design")](https://drive.google.com/file/d/1vNR-I4MNIf5wHrv44bl-4YKe4lSXXBZ6/view?usp=sharing)

### Các thành phần trong hệ thống

- Cloudfront(CDN): Sau khi đã có các nội dung tĩnh trên S3 như image của 1 bài báo, ta cấu hình CDN Cloudfront để có thể dễ dàng lấy casci data tĩnh như image từ nhiều client từ trên nhiều region, mà với tốc độ tối ưu, giảm độ trễ, chịu được tải cao.
- Load Balancer: Để hệ thống chịu được lượng tải cao, ta cần 1 Load Balancer làm nhiệm vụ load balancing, tối ưu việc routing request của client về instance có response time thấp nhất, ít bị delay nhất theo 1 số thuật toán.
- Kafka service: Xử lý event từ instance và từ worker để instance có thể biết lấy data từ Redis hoặc chờ để lấy lại data từ cache
- Binlog - Debezium: Xử lý bắn event cho kafka khi đã lấy data và cache, từ đó instance có thể lấy data từ cache.

### System design basic:

[![System design](/sys-basic.png "System design")](https://drive.google.com/file/d/1vNR-I4MNIf5wHrv44bl-4YKe4lSXXBZ6/view?usp=sharing)

### Các bước triển khai hệ thống basic:

- Bước 1: Thiết lập Cloudfront(CDN) trên AWS, sau đó trỏ CDN vào S3 để có thể phân tán nội dung trong s3 ở trên nhiều region, tối ưu, có độ trễ thấp nhờ nguyên lý cache của Cloudfront. Tạo S3 với các permission cho Cloudfront.
- Bước 2: Thiết lập instance với ngôn ngữ PHP, bao gồm DB là mysql, cache DB là Redis.
- Bước 3: Thiết lập logic get 1 bài báo và post 1 bài báo trên instance, kết hợp cache trên Redis và lấy data từ DB.
- Bước 4: Thiết lập domain trên Cloudflare, trỏ vào IP của instance.
