---
layout: post
title: Saga Pattern
---
### Hiện Trường Vụ Án :
- Kiến trúc hệ thống của bạn dùng để xây dựng sản phẩm/dự án đang là Microservices
- Các Services bên trong đang giao tiếp với nhau bằng việc tiếp nhận và xử lý các sự kiện xảy ra trong hệ thống (Event-Driven Architecture)
- Các Services bên trong hệ thống của các bạn đều sử dụng một Database riêng của chính nó
- Bạn có các Business Transactions (một chuỗi các hành động xảy ra để đạt được một kết quả bạn mong đợi) nhưng không may là các chuỗi hành động này trải dài ra toàn bộ các services bên trong Microservices của bạn và bạn cần một cơ chế để có thể triển khai Business Transactions này trên Microservices của mình
### Ví Dụ Nè
  
  ![Ví dụ về Saga Pattern](/images/example-1.jpg)

### Problem :

- How to implement transactions that span services?

### Forces :

- 2PC (**Distributed transaction**) is not an option - must implement to handle **dual write problem**

### Solution :

- Implement each business transaction that spans multiple services is a saga. A saga is a sequence of local transactions. Each local transaction updates the database and publishes a message or event to trigger the next local transaction in the saga. If a local transaction fails because it violates a business rule then the saga executes a series of compensating transactions that undo the changes that were made by the preceding local transactions.

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
