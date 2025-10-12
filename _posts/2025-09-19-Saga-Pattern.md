---
layout: post
title: Saga Pattern
---

Khi các hệ thống của chúng ta chuyển dịch sang kiến trúc Microservices, câu chuyện về việc phối hợp giữa vô số services trở nên phức tạp hơn bao giờ hết. Bài viết này sẽ dẫn bạn đi qua một "hiện trường" quen thuộc, cùng khám phá vì sao Saga Pattern được xem là lời giải thân thiện cho những distributed transaction dài hơi.

### Bối Cảnh Vấn Đề

Hãy tưởng tượng bạn đang sở hữu một hệ thống Microservices với những đặc điểm sau:

- Mỗi service có database riêng và trao đổi với nhau thông qua các sự kiện (Event-Driven Architecture).
- Dòng nghiệp vụ của bạn kéo theo nhiều bước xử lý nằm rải rác ở các services khác nhau.
- Bạn cần một cơ chế bảo đảm toàn bộ chuỗi thao tác đó hoàn thành trọn vẹn, hoặc nếu có lỗi thì hệ thống vẫn hồi phục được.

Đó chính là lúc distributed transactions bước lên sân khấu – và cũng là lúc Saga Pattern thể hiện giá trị.

### Câu Chuyện Minh Họa

![Ví dụ về Saga Pattern](/images/example-1.jpg)

Để dễ hình dung, hãy cùng nhìn vào hành trình giải ngân một khoản vay. Chuỗi các bước 1, 2, 3, 4 lần lượt được xử lý bởi những services chuyên trách khác nhau. Mỗi bước được gọi là một *local transaction*: service thực hiện logic của nó, lưu kết quả vào database riêng và phát đi tín hiệu (event/response) để kích hoạt bước kế tiếp. Khi toàn bộ chuỗi hoàn thành, chúng ta có một distributed transaction trọn vẹn.

### Bài Toán Đặt Ra

Vậy làm sao để điều phối chuỗi local transactions này? Two-Phase Commit (2PC) là một lựa chọn, nhưng trong phạm vi bài viết chúng ta sẽ tập trung vào phương án được ưa chuộng hơn trong thế giới microservices: **Saga Pattern**.

### Saga Pattern Là Gì?

Saga Pattern là một design pattern dùng để quản lý distributed transactions bằng cách chia chúng thành các local transactions được điều phối tuần tự. Có hai chiến lược chính:

1. **Saga Choreography (Điều Phối Phân Tán)** – mỗi local transaction phát ra domain event để kích hoạt service kế tiếp.
2. **Saga Orchestration (Điều Phối Tập Trung)** – một orchestrator đóng vai trò nhạc trưởng, điều khiển các services lần lượt thực hiện local transaction của mình.

### Khi Choreography Lên Ngôi

![Ví dụ về Saga Pattern Choreography](/images/example-2.png)

Trong phiên bản choreographed của bài toán cho vay, mọi thứ vận hành theo các sự kiện được phát tán:

1. Khách hàng gửi yêu cầu vay → `Eligibility Service` nhận request `POST /eligibilityCheck`, xử lý thông tin pháp lý và phát sự kiện thành công.
2. `Affordability Service` nhận sự kiện, kiểm tra thu nhập khách hàng, lưu kết quả và tiếp tục phát sự kiện mới.
3. `Decision Service` tạo khoản vay, ghi nhận dữ liệu và phát sự kiện quyết định giải ngân.
4. `Disbursement Service` nhận sự kiện cuối cùng, giải ngân khoản vay và ghi sổ kế toán.

Dòng sự kiện khép lại khi tiền được giải ngân – distributed transaction hoàn tất một cách nhẹ nhàng.

### Khi Orchestration Dẫn Dắt

![Ví dụ về Saga Pattern Orchestration](/images/example-3.png)

Với chiến lược orchestration, chúng ta thêm một nhân vật trung tâm – orchestrator:

1. Khách hàng gửi yêu cầu vay → orchestrator nhận request và chủ động gửi sự kiện đến `Eligibility Service`.
2. Khi `Eligibility Service` hoàn tất, orchestrator nhận thông báo và ra lệnh cho `Affordability Service` tiếp tục.
3. Từng bước lần lượt được orchestrator điều khiển tới `Decision Service` và `Disbursement Service`.
4. Khi bước cuối cùng hoàn thành, orchestrator xác nhận distributed transaction đã kết thúc.

Về bản chất, hai chiến lược khác nhau ở cách điều phối, nhưng đều sử dụng các local transactions để xây dựng nên một workflow dài hơi.

### Ưu Và Nhược Điểm

## Ưu Điểm
1. **Khả năng mở rộng & độc lập** – Services chỉ cần lo phần việc của mình, không chia sẻ database, dễ dàng scale độc lập và tránh sự phụ thuộc chặt chẽ.
2. **Giảm áp lực khóa dữ liệu** – Local transaction commit ngay khi xong việc, giảm thời gian giữ lock, phù hợp với quy trình dài hạn và hạn chế deadlock.
3. **Khả năng phục hồi** – Khi một bước thất bại, Saga có thể chạy *compensating transaction* để quay ngược các bước trước đó, đảm bảo eventual consistency.
4. **Ăn ý với Event-Driven Architecture** – Saga phát huy sức mạnh khi kết hợp với message queue như Kafka, RabbitMQ, EventBridge,...

## Nhược Điểm (Cons)

- **Consistency chỉ là “eventual”** – Không có đảm bảo atomic như 2PC. Trong một khoảng thời gian, dữ liệu có thể ở trạng thái trung gian (inconsistent).
- **Phức tạp khi định nghĩa compensating transaction** – Không phải lúc nào cũng dễ để “undo” một bước (ví dụ: đã gửi email/SMS cho khách thì khó mà rollback).
- **Quản lý workflow phức tạp** – Với choreography, hệ thống có nguy cơ tạo “event spaghetti” khó đọc và khó maintain; trong khi orchestration tạo dependency vào một orchestrator trung tâm dễ trở thành bottleneck.
- **Debug và giám sát khó khăn hơn** – Transaction được phân tán qua nhiều service và xử lý bất đồng bộ, khiến việc trace và monitor khó hơn so với giao dịch đơn lẻ.
- **Latency cao hơn** – Các bước thực hiện nối tiếp thông qua messaging làm độ trễ lớn hơn so với một ACID transaction trong cùng database.
### Lời Kết

Saga Pattern không phải là viên đạn bạc, nhưng là một công cụ hiệu quả để giữ cho distributed transactions trong kiến trúc microservices vận hành an toàn, linh hoạt. Tùy vào bài toán, bạn có thể chọn choreography để tận dụng kiến trúc phân tán, hoặc orchestration nếu muốn sự điều phối tập trung. Điều quan trọng nhất là hiểu rõ bối cảnh hệ thống của bạn để lựa chọn giải pháp phù hợp.
