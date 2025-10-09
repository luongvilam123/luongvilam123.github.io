---
layout: post
title: Saga Pattern
---
### Hiện Trường Vụ Án :
- Kiến trúc hệ thống của bạn dùng để xây dựng sản phẩm/dự án đang là Microservices
- Các Services bên trong đang giao tiếp với nhau bằng việc tiếp nhận và xử lý các sự kiện xảy ra trong hệ thống (Event-Driven Architecture)
- Các Services bên trong hệ thống của các bạn đều sử dụng một Database riêng của chính nó
- Bạn có các Distributed Transactions - một chuỗi các hành động xảy ra phân tán, trải dài ra toàn bộ các services bên trong Microservices của bạn và bạn cần một cơ chế để có thể triển khai Distributed Transactions này trên Microservices của mình
### Ví Dụ Nè
  
  ![Ví dụ về Saga Pattern](/images/example-1.jpg)

- Phần ví dụ trên đã được mình đơn giản hóa một cách dễ hiễu nhất nhưng trong thực tế thì lượng services trong Microservices nhiều hơn và giao tiếp với nhau phức tạp hơn rất nhiều nha
- Ở đây khi khách hàng tiến hành vay tiền và được giải ngân số tiền thì sẽ là một Transaction hoàn tất
- Trong Transaction này có nhiều hành động xảy ra được đảm nhận bởi các services cụ thể (1, 2, 3, 4), mỗi hành động ở một service được gọi là một Local Transaction (cập nhật kết quả vào database của chính nó và tiến hành tạo ra một sự kiện hoặc phản hồi để kích hoạt Local Transaction tiếp theo)

### Đặt Vấn Đề :

- Làm sao bạn có thể triển khai Distributed Transactions này ?

### Giải Pháp :

- 2PC (**Distributed transaction**) -> không được lựa chọn vì tiêu đề của bài viết này không phải là Two-Phase Commit nha hehe
- Saga Pattern -> chọn ngay thôi

### Saga Pattern Là Gì ? :

- Saga Pattern là một Design Pattern (là những giải pháp điển hình cho các vấn đề thường gặp trong thiết kế phần mềm. Chúng giống như những bản thiết kế được tạo sẵn mà bạn có thể tùy chỉnh để giải quyết vấn đề thường gặp trong lúc triển khai phần mềm của mình).
- Saga Pattern là giải pháp giúp bạn quản lý các Distributed Transactions trong Microservices bằng cách điều phối các local transactions (là các transactions được đánh số 1, 2, 3, 4 trong ví dụ mà ) trải dài trên các services.
- Có hai hướng để điều phối Saga :
   - Saga Choreography (Điều Phối Phân Tán): mỗi local transaction (giao dịch cục bộ) sẽ phát ra các sự kiện (domain events) để kích hoạt các local transaction (giao dịch cục bộ) ở các services khác.
   - Saga Orchestration: một trình điều phối (orchestrator) sẽ chỉ định cho các thành phần tham gia biết cần thực hiện local transaction (giao dịch cục bộ) nào.

### Ví Dụ Cho Saga Choreography (Điều Phối Phân Tán)

![Ví dụ về Saga Pattern Choreography](/images/example-2.png)

Tính năng cho vay ở trên đã được mình triển khai Saga Choreography để quản lý Distributed Transaction như sau: 

1. Khi khách hàng tiến hành yêu cầu một khoản vay, một `POST/eligibilityCheck` request được gửi đến `Eligibility Service`.
2. `Eligibility Service` sau đó sẽ tiến hành thực hiện các logic về kiểm tra lý lịch của khách hàng, khi đã xử lý thành công các thông tin pháp lý của khách hàng được lưu lại ở local database và phát đi một sự kiện để kích hoạt bước tiếp theo.
3. `Affordability Service` nhận được sự kiện về việc khách hàng đã đủ điều kiện pháp lý để vay tiền (pass eligibility check), tiến hành thực hiện logic kiểm tra thu nhập hàng tháng của khách hàng, khi đã xác nhận thu nhập hàng tháng của khách hàng đủ điều kiện để trả tiền hàng tháng cho khoản vay, thông tin được lưu lại ở local database và phát đi một để kích hoạt bước tiếp theo.
5. `Decision Service` sau khi nhận được sự kiện từ việc khách hàng đã đủ điều kiện để trả tiền cho khoản vay hàng tháng, tiến hành tạo ra một khoản vay cho khách hàng, lưu thông tin này lại ở local database và phát đi một sự kiện để kích hoạt bước cuối cùng là giải ngân số tiền vay đến tài khoản của khách hàng.
7. `Disbursement Service` sau khi nhận được sự kiện về quyết định giải ngân số tiền 2 tỉ cho khách hàng, tiến hành giải ngân số tiền này và đồng thời lưu lại thông tin tùy theo accounting flow của doanh nghiệp để ghi nhận và phục vụ cho việc đối soát.

Vậy là Distributed Transaction của chúng ta đã hoàn thành !!!

### Ví Dụ Cho Saga Orchestration (Điều Phối Tập Trung)

![Ví dụ về Saga Pattern Orchestration](/images/example-3.png)

Tính năng cho vay ở trên đã được mình triển khai Saga Orchestration để quản lý Distributed Transaction như sau: 

1. Khi khách hàng tiến hành yêu cầu một khoản vay, một `POST/eligibilityCheck` request được gửi đến `Orchestrator Service`, đây sẽ là vị nhạc trưởng để điều phối các local transactions của chúng ta.
2. Nhạc trưởng sẽ gửi một sự kiện đến `Eligibility Service` yêu cầu tiến hành thực hiện các logic về kiểm tra lý lịch của khách hàng, khi đã xử lý thành công các thông tin pháp lý của khách hàng được lưu lại ở local database và phát đi một sự kiện thông báo rằng đã thực hiện thành công.
3. Nhạc trưởng nhận được sự kiện thông báo rằng `Eligibility Service` đã thực hiện thành công, nhạc trưởng tiến hành phát đi một sự kiện đến `Affordability Service` yêu cầu tiến hành thực hiện logic kiểm tra thu nhập hàng tháng của khách hàng, khi đã xác nhận thu nhập hàng tháng của khách hàng đủ điều kiện để trả tiền hàng tháng cho khoản vay, thông tin được lưu lại ở local database và phát đi một sự kiện thông báo rằng đã thực hiện thành công.
4. Nhạc trưởng nhận được sự kiện thông báo rằng `Affordability Service` đã thực hiện thành công, nhạc trưởng tiến hành phát đi một sự kiện đến `Decision Service`, yêu cầu tiến hành tạo ra một khoản vay cho khách hàng, lưu thông tin này lại ở local database và phát đi một sự kiện thông báo rằng đã thực hiện thành công.
5.  Nhạc trưởng nhận được sự kiện thông báo rằng `Decision Service` đã thực hiện thành công, nhạc trưởng tiến hành phát đi một sự kiện đến `Disbursement Service`, yêu cầu tiến hành giải ngân số tiền 2 tỉ cho khách hàng, tiến hành giải ngân số tiền này và đồng thời lưu lại thông tin tùy theo accounting flow của doanh nghiệp để ghi nhận và phục vụ cho việc đối soát.

Vậy là Distributed Transaction của chúng ta đã hoàn thành !!!

### Ưu Điểm Và Nhược Điểm Của Saga Pattern
