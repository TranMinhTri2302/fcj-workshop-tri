---

title: "Đề xuất dự án"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
--------------------

# OpsTicket CI/CD — Hệ thống quản lý sự cố nội bộ

## Xây dựng và triển khai hệ thống CI/CD trên AWS cho ứng dụng Web quản lý sự cố nội bộ

### 1. Tóm tắt dự án

OpsTicket CI/CD là một hệ thống quản lý sự cố nội bộ được thiết kế cho văn phòng, trường học, phòng lab hoặc các nhóm làm việc nhỏ. Hệ thống cho phép người dùng báo cáo các sự cố nội bộ như Wi-Fi yếu, máy chiếu hỏng, máy in không hoạt động, máy lạnh gặp vấn đề, ổ điện hư hoặc tài khoản nội bộ bị khóa.

Trong nhiều môi trường làm việc thực tế, các sự cố này thường được báo qua tin nhắn, nhóm chat hoặc trao đổi trực tiếp. Cách làm này nhanh, nhưng không dễ theo dõi. Tin nhắn có thể bị trôi, người phụ trách không rõ ràng, và không có cách đáng tin cậy để biết sự cố đang ở trạng thái mới tạo, đang xử lý hay đã hoàn tất.

OpsTicket giải quyết vấn đề này bằng cách chuyển các báo cáo rời rạc thành các ticket có cấu trúc. Mỗi ticket bao gồm tiêu đề, mô tả, loại sự cố, địa điểm, mức độ ưu tiên, trạng thái, nhóm phụ trách và thời gian tạo. Nhân sự hỗ trợ có thể cập nhật trạng thái ticket, gán ticket cho nhóm xử lý phù hợp và theo dõi tổng quan quá trình xử lý.

Về mặt AWS và DevOps, dự án tập trung vào việc xây dựng, đóng gói, triển khai và giám sát ứng dụng thông qua quy trình CI/CD. Backend được xây dựng bằng Java Spring Boot, đóng gói thành Docker image, lưu trữ trên Amazon ECR và triển khai lên Amazon ECS Fargate. Dữ liệu ticket được lưu trong Amazon DynamoDB. Log, metric và alarm được theo dõi thông qua Amazon CloudWatch. GitHub Actions được sử dụng để tự động hóa các bước kiểm thử, build image, push image và triển khai ứng dụng.

Mục đích của dự án không chỉ là xây dựng một ứng dụng web, mà còn là thể hiện một quy trình triển khai ứng dụng hoàn chỉnh trên AWS.

---

### 2. Phát biểu bài toán

### Vấn đề hiện tại là gì?

Trong các tổ chức nhỏ, văn phòng hoặc môi trường học tập, việc báo cáo sự cố nội bộ thường được xử lý thủ công hoặc không có cấu trúc rõ ràng. Ví dụ, một sinh viên có thể nhắn cho admin rằng máy chiếu ở phòng B203 không hoạt động, hoặc một nhân viên có thể báo miệng với bộ phận cơ sở vật chất rằng máy lạnh bị hỏng. Những cách báo cáo này dễ thực hiện, nhưng khó quản lý lâu dài.

Các vấn đề chính bao gồm:

* Báo cáo sự cố có thể bị trôi trong nhóm chat.
* Không có trạng thái xử lý rõ ràng.
* Khó xác định ai hoặc nhóm nào đang phụ trách sự cố.
* Sự cố khẩn cấp không được phân biệt rõ với sự cố thông thường.
* Không có dashboard đơn giản để xem số lượng ticket đang mở, đã xử lý hoặc quá hạn.
* Việc triển khai ứng dụng thủ công có thể gây sai lệch phiên bản.
* Nếu không có monitoring, lỗi backend có thể chỉ được phát hiện khi người dùng phản ánh.

### Giải pháp đề xuất

OpsTicket cung cấp một quy trình quản lý ticket rõ ràng cho các sự cố nội bộ. Người dùng có thể tạo ticket, còn nhân sự hỗ trợ có thể xử lý ticket theo các trạng thái cụ thể:

```txt id="6xjda7"
NEW → ASSIGNED → IN_PROGRESS → RESOLVED → CLOSED
```

Ticket được phân loại theo mức độ ưu tiên:

```txt id="75k68r"
P1 - Sự cố nghiêm trọng
P2 - Sự cố ưu tiên cao
P3 - Sự cố thông thường
```

Backend cũng hỗ trợ quy tắc tự động gán nhóm xử lý. Ví dụ:

```txt id="vdi13q"
WIFI, PROJECTOR, PRINTER, ACCOUNT → IT_SUPPORT
AIR_CONDITIONER, LIGHT, POWER_SOCKET → FACILITY
GENERAL → GENERAL_ADMIN
```

Hệ thống được triển khai trên AWS thông qua quy trình CI/CD:

```txt id="l6xobm"
GitHub → GitHub Actions → Docker Build → Amazon ECR → Amazon ECS Fargate → CloudWatch
```

Điều này giúp OpsTicket trở thành một use-case thực tế trên AWS. AWS không chỉ được dùng như nơi để host ứng dụng, mà còn là một phần trong quy trình triển khai, vận hành, lưu trữ dữ liệu, giám sát, bảo mật và tối ưu chi phí.

### Lợi ích của dự án

OpsTicket mang lại hai lợi ích chính.

Thứ nhất, hệ thống giúp cải thiện quy trình xử lý sự cố nội bộ. Mỗi sự cố được chuyển thành một ticket có trạng thái, mức độ ưu tiên, nhóm phụ trách và thời gian ghi nhận. Điều này giúp nhóm hỗ trợ tránh bỏ sót báo cáo và làm cho quá trình xử lý minh bạch hơn.

Thứ hai, dự án là một bài thực hành DevOps và AWS thực tế. Dự án bao gồm Docker, CI/CD, Amazon ECR, Amazon ECS, DynamoDB, CloudWatch, IAM và clean-up tài nguyên. Đây là các kỹ năng quan trọng khi triển khai và vận hành ứng dụng trên AWS.

---

### 3. Kiến trúc giải pháp

OpsTicket sử dụng kiến trúc AWS dựa trên container kết hợp với quy trình CI/CD tự động.

Ở mức tổng quan, luồng hoạt động như sau:

```txt id="5mxm7e"
Developer push code lên GitHub
→ GitHub Actions chạy kiểm thử và build Docker image
→ Docker image được push lên Amazon ECR
→ Amazon ECS Fargate chạy backend container
→ Application Load Balancer public API backend
→ Backend lưu dữ liệu ticket vào DynamoDB
→ CloudWatch thu thập log và metric
→ CloudWatch Alarm và SNS có thể gửi cảnh báo khi có sự cố
```

![OpsTicket CI/CD Architecture](/images/2-Proposal/opsticket-cicd-architecture.png)

### Các dịch vụ AWS được sử dụng

* **Amazon ECR**: Lưu trữ Docker image của backend Spring Boot.
* **Amazon ECS Fargate**: Chạy backend container mà không cần tự quản lý EC2 server.
* **Application Load Balancer**: Public API backend và thực hiện health check.
* **Amazon DynamoDB**: Lưu dữ liệu ticket như tiêu đề, loại sự cố, mức ưu tiên, trạng thái và nhóm phụ trách.
* **Amazon S3**: Có thể dùng để host frontend hoặc lưu ảnh đính kèm của ticket.
* **Amazon CloudFront**: Có thể dùng để phân phối frontend nếu frontend được triển khai dưới dạng static web application.
* **Amazon CloudWatch**: Thu thập log, metric và alarm của hệ thống.
* **Amazon SNS**: Gửi email cảnh báo khi hệ thống có lỗi hoặc khi có ticket quan trọng.
* **AWS IAM**: Quản lý quyền truy cập bằng IAM Role và chính sách least privilege.
* **GitHub Actions**: Tự động hóa quá trình test, build Docker image, push image lên ECR và deploy lên ECS.

### Thiết kế thành phần

* **Backend API**: Được xây dựng bằng Java Spring Boot. Backend cung cấp API cho health check, tạo ticket, xem danh sách ticket, xem chi tiết ticket, cập nhật trạng thái, gán nhóm xử lý, thêm bình luận và xem dashboard tổng quan.
* **Container Layer**: Backend được đóng gói thành Docker image để có thể chạy nhất quán trên môi trường local, CI và AWS.
* **CI/CD Pipeline**: GitHub Actions chạy kiểm thử, build Docker image, push image lên ECR và deploy phiên bản mới lên ECS.
* **Runtime Layer**: ECS Fargate chạy backend container. Application Load Balancer chuyển request từ người dùng đến ECS task đang chạy.
* **Data Layer**: DynamoDB lưu trữ dữ liệu ticket bằng dịch vụ NoSQL managed của AWS.
* **Monitoring Layer**: CloudWatch cung cấp log, metric và alarm để hỗ trợ vận hành và xử lý lỗi.

---

### 4. Triển khai kỹ thuật

### Các giai đoạn triển khai

#### Giai đoạn 1: Xây dựng backend application

Dự án bắt đầu với backend Spring Boot. API đầu tiên là `/health`, được dùng để kiểm thử local và dùng làm health check cho Load Balancer sau khi triển khai.

Sau đó, các API liên quan đến ticket được xây dựng:

* Tạo ticket
* Xem danh sách ticket
* Xem chi tiết ticket
* Cập nhật trạng thái ticket
* Gán ticket cho nhóm xử lý
* Thêm bình luận
* Xem dashboard tổng quan

Ở giai đoạn đầu, backend có thể dùng in-memory storage để kiểm thử nhanh hành vi API. Trong phiên bản triển khai cuối trên AWS, dữ liệu ticket phải được lưu vào DynamoDB.

#### Giai đoạn 2: Container hóa backend

Backend được đóng gói thành Docker image. Việc này giúp ứng dụng chạy nhất quán ở local, trên GitHub Actions và trên AWS.

Kết quả mong đợi là Docker image expose port `8080` và trả về response thành công từ:

```txt id="2ry58m"
GET /health
```

#### Giai đoạn 3: Push image lên Amazon ECR

Một Amazon ECR repository được tạo để lưu Docker image của backend. Image được tag và push lên ECR. Bước này chuẩn bị image ứng dụng để ECS có thể pull và chạy container.

#### Giai đoạn 4: Deploy lên Amazon ECS Fargate

Ứng dụng được triển khai lên ECS Fargate thông qua ECS cluster, task definition, ECS service, target group, Application Load Balancer và security group. Load Balancer sử dụng endpoint `/health` để kiểm tra backend có hoạt động đúng hay không.

Kết quả mong đợi là backend có thể được truy cập thông qua DNS name của Load Balancer.

#### Giai đoạn 5: Thiết lập CI Pipeline

GitHub Actions được cấu hình để chạy khi có pull request. CI pipeline cần thực hiện:

* Checkout source code
* Cấu hình Java
* Chạy Maven test
* Build application
* Build Docker image

Bước này giúp hạn chế việc merge code lỗi vào branch chính.

#### Giai đoạn 6: Thiết lập CD Pipeline

GitHub Actions được cấu hình để chạy khi code được push vào branch `main`. CD pipeline cần thực hiện:

* Build Docker image mới
* Tag image bằng Git commit SHA
* Push image lên Amazon ECR
* Render ECS task definition mới
* Deploy task definition mới lên ECS
* Chờ ECS service ổn định

#### Giai đoạn 7: Tích hợp DynamoDB

Backend được cập nhật để lưu dữ liệu ticket vào DynamoDB. Điều này giúp dữ liệu không bị mất khi ECS task restart hoặc khi ứng dụng được deploy phiên bản mới.

#### Giai đoạn 8: Thêm monitoring và alert

CloudWatch Logs được dùng để xem log backend. CloudWatch Metrics được dùng để theo dõi ECS và Load Balancer. Ít nhất một CloudWatch Alarm nên được tạo, ví dụ alarm cho lỗi backend 5xx hoặc CPU usage cao. SNS có thể được dùng để gửi email khi alarm được kích hoạt.

### Yêu cầu kỹ thuật

* **Ngôn ngữ lập trình**: Java
* **Backend Framework**: Spring Boot
* **Build Tool**: Maven
* **Containerization**: Docker
* **CI/CD Tool**: GitHub Actions
* **Container Registry**: Amazon ECR
* **Runtime Platform**: Amazon ECS Fargate
* **Database**: Amazon DynamoDB
* **Monitoring**: Amazon CloudWatch
* **Notification**: Amazon SNS
* **Security**: AWS IAM Role và least-privilege policy
* **Region khuyến nghị**: `ap-southeast-1`

### Các API backend chính

* `GET /health`
* `POST /tickets`
* `GET /tickets`
* `GET /tickets/{ticketId}`
* `PATCH /tickets/{ticketId}/status`
* `PATCH /tickets/{ticketId}/assign`
* `POST /tickets/{ticketId}/comments`
* `GET /dashboard/summary`
* `POST /admin/sla/check`

### Các quy tắc nghiệp vụ quan trọng

OpsTicket không nên chỉ là một ứng dụng CRUD đơn giản. Backend cần có một số quy tắc nghiệp vụ cơ bản để hệ thống thực tế hơn.

#### Quy trình trạng thái ticket

```txt id="ejg76l"
NEW → ASSIGNED → IN_PROGRESS → RESOLVED → CLOSED
```

#### Priority và SLA

```txt id="5z0hxn"
P1: Sự cố nghiêm trọng, thời gian phản hồi mong muốn trong 4 giờ
P2: Sự cố ưu tiên cao, thời gian phản hồi mong muốn trong 24 giờ
P3: Sự cố thông thường, thời gian phản hồi mong muốn trong 72 giờ
```

#### Tự động gán nhóm xử lý

```txt id="2oy5o4"
WIFI, PROJECTOR, PRINTER, ACCOUNT, SOFTWARE, HARDWARE → IT_SUPPORT
AIR_CONDITIONER, LIGHT, POWER_SOCKET, TABLE_CHAIR → FACILITY
GENERAL → GENERAL_ADMIN
```

---

### 5. Timeline & Milestones

### Lộ trình dự án

Dự án được triển khai theo lộ trình thực tập 12 tuần.

* **Week 1: Tìm hiểu DevOps, CI/CD, Git, Docker và các lựa chọn deploy trên AWS**
  Nắm quy trình phát triển và triển khai phần mềm, đồng thời so sánh EC2, ECS và Elastic Beanstalk.

* **Week 2: Xây dựng backend OpsTicket ban đầu**
  Tạo project Spring Boot, API health check và các API ticket cơ bản.

* **Week 3: Container hóa ứng dụng**
  Tạo Dockerfile, build image local và test container.

* **Week 4: Push Docker image lên Amazon ECR**
  Tạo ECR repository, tag image và push image lên AWS.

* **Week 5: Thiết lập CI Pipeline**
  Cấu hình GitHub Actions để chạy Maven test và build Docker image.

* **Week 6: Deploy backend lên Amazon ECS Fargate**
  Tạo ECS cluster, task definition, ECS service, target group, Load Balancer và security group.

* **Week 7: Thiết lập CD Pipeline**
  Cấu hình GitHub Actions để tự động deploy version mới lên ECS khi code được push vào `main`.

* **Week 8: Cấu hình CloudWatch Logs và Metrics**
  Kiểm tra log backend và metric của ECS/Load Balancer trên CloudWatch.

* **Week 9: Tích hợp DynamoDB và SNS Alert nếu có**
  Lưu ticket vào DynamoDB và cấu hình cảnh báo nếu cần.

* **Week 10: Cải thiện bảo mật và tối ưu chi phí**
  Kiểm tra IAM permissions, tránh hard-code credential, giảm chi phí không cần thiết và chuẩn bị kế hoạch cleanup.

* **Week 11: Hoàn thiện blog và tài liệu workshop**
  Viết blog post và hoàn thiện website workshop song ngữ.

* **Week 12: Kiểm thử end-to-end, chụp screenshot, cleanup và hoàn thiện báo cáo**
  Kiểm thử toàn bộ workflow, thu thập minh chứng, dọn dẹp tài nguyên AWS và hoàn thiện báo cáo cuối kỳ.

---

### 6. Ước tính ngân sách

Dự án được thiết kế cho mục đích workshop và demo với lượng truy cập thấp. Chi phí sẽ được kiểm soát nếu tài nguyên chỉ chạy trong thời gian kiểm thử và được dọn dẹp sau khi hoàn tất.

Các thành phần chi phí chính:

* **Amazon ECS Fargate**: Phụ thuộc vào CPU, memory và thời gian chạy task.
* **Application Load Balancer**: Phụ thuộc vào thời gian chạy và lưu lượng request.
* **Amazon ECR**: Phụ thuộc vào dung lượng Docker image được lưu trữ.
* **Amazon DynamoDB**: Chế độ on-demand phù hợp cho workload nhỏ và không ổn định.
* **Amazon CloudWatch Logs**: Phụ thuộc vào dung lượng log và thời gian lưu log.
* **Amazon S3 và CloudFront**: Chỉ sử dụng nếu triển khai frontend hoặc attachment.
* **Amazon SNS**: Chi phí thấp nếu chỉ gửi số lượng nhỏ email notification.

Chiến lược tối ưu chi phí:

* Dùng ECS Fargate task size nhỏ.
* Dừng hoặc xóa ECS resource sau khi test.
* Dùng DynamoDB on-demand mode.
* Cấu hình CloudWatch log retention trong 7 hoặc 14 ngày.
* Xóa các ECR image cũ.
* Dọn dẹp ALB, ECS, ECR, DynamoDB, S3, CloudWatch alarm và log group sau workshop.

Chi phí cuối cùng nên được ước tính bằng AWS Pricing Calculator dựa trên region, ECS task size, thời gian chạy và lưu lượng dự kiến.

---

### 7. Đánh giá rủi ro

### Risk Matrix

| Rủi ro                        | Mức ảnh hưởng | Khả năng xảy ra | Mô tả                                                                           |
| ----------------------------- | ------------- | --------------- | ------------------------------------------------------------------------------- |
| Lỗi IAM permission            | Cao           | Trung bình      | GitHub Actions, ECS hoặc backend có thể lỗi nếu thiếu quyền cần thiết.          |
| Docker build thất bại         | Trung bình    | Trung bình      | Dockerfile hoặc cấu hình Maven có thể gây lỗi build.                            |
| ECS task không chạy           | Cao           | Trung bình      | Sai image URI, port mapping hoặc health check có thể làm deploy thất bại.       |
| ALB health check thất bại     | Cao           | Trung bình      | Nếu `/health` lỗi, ECS có thể liên tục thay task mới.                           |
| Lỗi tích hợp DynamoDB         | Trung bình    | Trung bình      | Sai table name, region hoặc IAM role có thể làm backend không lưu được dữ liệu. |
| Cấu hình CloudWatch alarm sai | Trung bình    | Thấp            | Alarm có thể không trigger nếu chọn sai metric hoặc threshold.                  |
| Phát sinh chi phí             | Trung bình    | Thấp            | ECS, ALB và log có thể phát sinh chi phí nếu không cleanup.                     |
| Rủi ro bảo mật                | Cao           | Thấp            | Hard-code credential hoặc public resource sai cách có thể gây rủi ro.           |

### Chiến lược giảm thiểu

* Test backend và Docker image ở local trước khi deploy lên AWS.
* Bắt đầu ECS deployment với endpoint `/health` đơn giản.
* Sử dụng IAM Role và least-privilege permissions.
* Không commit AWS access key hoặc secret lên GitHub.
* Dùng CloudWatch Logs để debug lỗi ECS task.
* Giữ hạ tầng ở mức nhỏ, phù hợp với workshop.
* Viết rõ các bước cleanup trong tài liệu.

### Kế hoạch dự phòng

* Nếu ECS deployment lỗi, rollback về task definition revision hoạt động trước đó.
* Nếu CD pipeline lỗi, deploy thủ công một lần và ghi nhận lỗi trong phần reflection.
* Nếu DynamoDB chưa hoàn thiện, có thể dùng tạm in-memory storage để test API, nhưng bản cuối phải lưu dữ liệu vào DynamoDB.
* Nếu SNS chưa hoàn thiện, CloudWatch Alarm vẫn có thể được dùng làm minh chứng tối thiểu cho phần alerting.

---

### 8. Kết quả mong đợi

### Kết quả kỹ thuật

Sau khi hoàn thành dự án, các kết quả kỹ thuật mong đợi gồm:

* Backend Spring Boot hoạt động cho OpsTicket.
* REST API cho quản lý ticket và dashboard summary.
* Docker image cho backend.
* Amazon ECR repository lưu backend image.
* Amazon ECS Fargate service chạy backend.
* Application Load Balancer public API.
* DynamoDB table lưu dữ liệu ticket.
* GitHub Actions CI workflow.
* GitHub Actions CD workflow.
* CloudWatch Logs, Metrics và ít nhất một CloudWatch Alarm.
* Hướng dẫn cleanup toàn bộ tài nguyên AWS.

### Kết quả nghiệp vụ

OpsTicket cung cấp một cách đơn giản nhưng có cấu trúc để quản lý sự cố nội bộ. Thay vì báo lỗi qua các tin nhắn rời rạc, mỗi sự cố trở thành một ticket có trạng thái, mức ưu tiên, nhóm phụ trách và thời gian ghi nhận.

Ví dụ, ticket “Máy chiếu ở phòng B203 không hoạt động” có thể được tạo với priority `P2`, tự động gán cho `IT_SUPPORT`, đi qua quy trình xử lý và được đánh dấu resolved sau khi sửa xong.

### Kết quả học tập

Dự án giúp thể hiện kiến thức thực tế về:

* Phát triển backend API bằng Java Spring Boot.
* Thiết kế workflow nghiệp vụ cho hệ thống ticket.
* Đóng gói ứng dụng bằng Docker.
* Tự động hóa CI/CD bằng GitHub Actions.
* Triển khai container trên AWS bằng ECR và ECS Fargate.
* Lưu trữ dữ liệu bằng DynamoDB.
* Giám sát hệ thống bằng CloudWatch.
* Áp dụng IAM security cơ bản và tối ưu chi phí.

### Giá trị dài hạn

OpsTicket có thể được mở rộng thành một hệ thống vận hành nội bộ hoàn chỉnh hơn. Các hướng phát triển tương lai có thể bao gồm xác thực người dùng bằng Amazon Cognito, upload ảnh ticket lên S3, kiểm tra SLA bằng EventBridge, gửi thông báo bằng SNS, xây dựng React dashboard và quản lý hạ tầng bằng Terraform, AWS CDK hoặc CloudFormation.

Dự án được thiết kế như một use-case AWS thực tế. Nó cho thấy AWS không chỉ hỗ trợ host ứng dụng, mà còn hỗ trợ tự động hóa triển khai, lưu trữ dữ liệu, giám sát, bảo mật và kiểm soát chi phí vận hành.
