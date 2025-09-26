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

### Đặt Vấn Đề :

- Làm sao bạn có thể triển khai Distributed Transactions này ?

### Giải Pháp :

- 2PC (**Distributed transaction**) -> không được lựa chọn vì tiêu đề của bài viết này không phải là Two-Phase Commit nha hehe
- Saga Pattern -> chọn ngay thôi

### Saga Pattern Là Gì ? :

- Saga Pattern là một Design Pattern (là những giải pháp điển hình cho các vấn đề thường gặp trong thiết kế phần mềm. Chúng giống như những bản thiết kế được tạo sẵn mà bạn có thể tùy chỉnh để giải quyết vấn đề thường gặp trong lúc triển khai phần mềm của mình)
- Saga Pattern là giải pháp giúp bạn quản lý

![Saga coordination overview](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/89952217-be82-4c35-bc5f-a0e766992d51/Untitled.png)

There are two ways of coordination sagas:

- **Choreography** - each local transaction publishes domain events that trigger local transactions in other services
- **Orchestration** - an orchestrator (object) tells the participants what local transactions to execute

### **Example: Choreography-based saga**

![Choreography-based saga example](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/950a7357-bb55-4284-9d68-3a92d5879c6c/Untitled.png)

An e-commerce application that uses this approach would create an order using a choreography-based saga that consists of the following steps:

1. The `Order Service` receives the `POST /orders` request and creates an `Order` in a `PENDING` state
2. It then emits an `Order Created` event
3. The `Customer Service`’s event handler attempts to reserve credit
4. It then emits an event indicating the outcome
5. The `OrderService`’s event handler either approves or rejects the `Order`

### **Example: Orchestration-based saga**

![Orchestration-based saga example](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/301a53ac-8439-43c2-a858-31a3251becf4/Untitled.png)

An e-commerce application that uses this approach would create an order using an orchestration-based saga that consists of the following steps:

1. The `Order Service` receives the `POST /orders` request and creates the `Create Order` saga orchestrator
2. The saga orchestrator creates an `Order` in the `PENDING` state
3. It then sends a `Reserve Credit` command to the `Customer Service`
4. The `Customer Service` attempts to reserve credit
5. It then sends back a reply message indicating the outcome
6. The saga orchestrator either approves or rejects the `Order`
