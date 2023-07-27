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

### Tìm hiểu về các thành phần của hệ thống

- Load Balancer:

  - Có nhiệm vụ cần bằng tải cho hệ thống gồm nhiều server/cụm server để đáp ứng được lượng request lên tới hàng triệu trong 1 khoảng thời gian ngắn 1 cách hiệu quả.
  - Routing các request đến các server có khả năng xử lý các request đó sao cho tối ưu nhất về tốc độ, hiệu suất và đảm bảo không có server nào họat động quá tải.
  - Yêu cầu trong design system:

- CDC(Binlog - Debezium)

- Cloudfront(CDN)

- Kafka
