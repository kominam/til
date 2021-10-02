Queue là một cấu trúc dữ liệu quen thuộc trong thiết kế hệ thống (software architecture). Nó cho phép thực hiện giao tiếp bất đồng bộ giữa các services, system khác nhau và đặc biệt hữu hiệu khi throughput của hệ thống là không giống nhau. 

AWS SQS là một web service cho phép bạn truy xuất và message queue để lưu trữ message trong khi đợi chúng được xử lý. SQS là một distributed queue system (hệ thống queue phân tán) cho phép app sử dụng message queue nhanh chóng và tin cậy khi một component trong app tạo ra và được xử lý bởi một thành phần khác. Nó trao đổi message thông qua cơ chế **polling** model và được quản lý, vân hành bởi AWS nên bạn chỉ cần config một chút. Reliable, highly-scalable

### Common use case
* **Work Queues**: tách thành phần của distributed app để không phải xử lý một khối lượng công việc đồng thời
* **Buffer and Batch Operations** tăng khả năng scale hệ thống và độ tin cậy (reliability) và giúp vận hành trơn tru mà không làm mất message hay tăng độ trễ (latency)
* **Request Offloading** với những công việc cần respond nhanh, không tương tác nhiều bằng việc enqueue request đó
* **Fan-out** Kết hợp SQS và SNS để gửi mốt số message đặc thù tới nhiều queue để cùng xử lý song song

### Outstanding Functionality
* **Unlimited queues and messages**: không giới hạn AWS SQS queue với số message trên mỗi region
* **Payload size**: Message payloads có thể lên tới 256kb text với bất kỳ định dạng nào.
* **Batches**: gửi, nhận và xóa message trong batches lên tới 10 messages hoặc 256kb. Cost tính như 1 message (single message) 
* **Long Polling**:  Giảm thiểu những polling ko liên quan (extraneous polling) để giảm cost trong khi nhận message mới càng nhanh càng tốt. Khi queue troóng thì long polling request sẽ đợi message tiếp theo trong vòng 20s. Cost được tính như những request bình thường.
* **Retain messages in queue for up to 14 days**: default là 4 ngày
* **Send and read messages simultaneously.**
* **Server-side encryption (SSE)**: using AWS SSE (Server side encryption), message sẽ được bảo mật hơn trong suốt quá trình trong queue.
* **Dead letter queue** tách những message chưa được xử lý ngay thời điểm đó
### SQS Standard Queue
Đây được AWS sử dụng làm queue mặc định và có các đặc điểm sau:
* High throughput: nealy-unlimited number of transactions per second (gần như không giới hạn số lượng transactions mỗi giây).
* At-Least-Once delivery: đảm bảo rằng mỗi message được deliver ít nhất 1 lần. (tuy nhiên trong một vài trường hợp một vài bản copy của message sẽ được deliver không theo thứ tự)
* Best effort ordering: đảm bảo rằng message sẽ thường được deliver theo đúng thứ tự

![](https://d1.awsstatic.com/AmazonSQS/sqs-what-is-sqs-standard-queue-diagram.29963b2823bc048492c7af2757535d500aa2c159.png)

### SQS FIFO Queue
FIFO (First In First Out) deliver và chính xác chỉ 1 message đc process:
* Preserved order message: thứ tự của message được đảm bảo chính xác và được devliver 1 lần. Sau đó sẽ ở trạng thái available cho đến khi nó được xử lý và xóa khỏi queue. (dựa trên message group id)
* Support message groups: cho phép nhiều message groups đã được sắp xếp trong 1 single queue. 
* Giới hạn 300 transactions mỗi giây và 3000 messages mỗi giây với batching

![](https://d1.awsstatic.com/AmazonSQS/sqs-what-is-sqs-fifo-queue-diagram.8f1c8d366f58845ce03bb2983c16349102cf1524.png)

### Queue and Message Identifiers
#### Queue URL 
* được xác định bởi unique queue name trên cùng 1 AWS account
* gán mỗi queue với 1 Queue URL. eg: http://sqs.us-east-1.amazonaws.com/123456789012/queue2
* cần được thực hiện một operation nào đó
#### Message ID
* dùng để định danh message
* mỗi message sẽ được hệ thống gán cho 1 id và được trả về trong `SendMessage`
* để xóa message thì cần `ReceiptHandle` chứ ko phải id của message
* maximum 100 ký tự
#### Receipt Handle
* khi queue nhận message, 1 receipt handle được trả về với message, nó được gán với hành động nhận message
* cần thiết nếu muốn xóa hay thay đổi visibility của message
* nêú message được nhận nhiều lần thì mỗi lần sẽ sinh ra 1 receipt handle và cái mới nhất sẽ được sử dụng
### Visibility timeout
![](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/images/sqs-visibility-timeout-diagram.png)

* **SQS does not delete the message once it is received by a consumer** tức là SQS sẽ không xóa message sau khi nó được nhận bởi consumer. bởi vì là hệ thống phân tán (distributed system) nên không có gì đảm bảo là consumer sẽ thực sự nhận được message (eg: mất kết nối mạng, hoặc component lỗi trước khi nhận message)
* Sau khi nhận và xử lý thành công message, consumer sẽ phải xóa message.
* Bởi vì message vẫn còn khả dụng trong queue và consumer khác có thể sẽ nhận và xử lý message đó. Điều này nên được ngăn chặn

3 behaviors trên sẽ được giải quyết bởi **Visibility timeout**
* SQS sẽ block visibility của message trong một khoảng thời gian, điều này ngăn chặn việc consumer khác nhận và xử lý message đó
* Consumer nên xóa message trong khoảng thời gian visibility timeout. Nếu xóa fail và visibility timeout expired thì consumer khác có thể nhận và xử lý message đó
* Vì 1 visibile message có thể được tiêu thụ và xử lý bởi consumer khác nên có thể dẫn đến duplicate mesage

#### Visibility considerations
* thời gian bắt đầu được tính khi SQS trả về messages
* đủ lớn để có thể xử lý xong message
* mặc định là 30s và có thể thay đổi bởi queue level
* khi nhận messages thì có thể set 1 visibility timeout đặc biệt mà không ảnh hưởng đến tổng thời gian timeout của queue thông qua receipt handle
* có thể extend bởi consumer thông qua `ChangeMessageVisibility`.
* Visibility timeout extention chỉ được áp dụng với receipt cụ thể, không ảnh hưởng tới timeout của queue hoặc receipt sau đó

SQS giới hạn 120,000 inflight messages mỗi queue. eg message recieved nhưng chưa xóa và bất kể những message nào sau sau khi quá giới hạn
### Message Lifecycle
![](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/images/sqs-message-lifecycle-diagram.png)

- Component 1 gửi message vào queue
- Component 2  lấy message từ queue và message A được trả về. Khi message A được xử lý thì nó vẫn nằm trong queue nhưng ko được trả về trong những request tiếp theo trong thời gian visibility timeout.
- Component 2 xóa message A khỏi queue trước khi visibility timeout của message A expire

### Notes
- Có 2 cách để nhận message từ queue đó là long polling và short polling (default). Long polling được khuyến khích nên dùng
- SQS là pull-based, not push-based
- payload message maximum là 256kb. Nếu lớn hơn có thể dùng S3 hoặc dynamo để lưu content của message và trong message của queue thì trỏ tới object S3 hoặc Dynamo
- Inflight message maximum của standard queue là 120,000 và 20,000 cho FIFO queue.
- default và maximum batch size cho `ReceiveMessage` call là 10.
- Visibilility timeout maximum là 12h
### References
* https://aws.amazon.com/sqs/features/
* https://jayendrapatil.com/aws-sqs-simple-queue-service/