---
title : "Giới thiệu"
date : 2024-01-01 
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### 5.1. Giới thiệu

### 1. Giới thiệu giải pháp (Use case)
Trong bối cảnh chuyển đổi số giáo dục và doanh nghiệp, việc điểm danh thủ công (quẹt thẻ từ, vân tay) vẫn tồn tại nhiều hạn chế lớn (pain points): ùn tắc vào giờ cao điểm, tình trạng quên thẻ, hoặc gian lận check-in hộ. Các hệ thống máy chủ vật lý nội bộ thường lãng phí tài nguyên khi không có ai sử dụng vào ban đêm, nhưng lại quá tải vào khung giờ 8h00 sáng.

**Smart Campus Platform** ra đời nhằm giải quyết triệt để vấn đề này bằng cách kết hợp trí tuệ nhân tạo nhận diện khuôn mặt (**Amazon Rekognition**) và kiến trúc **100% AWS Serverless**. Hệ thống không chỉ xử lý điểm danh siêu tốc mà còn đảm bảo bảo mật tuyệt đối, tự động hóa luồng thông báo và cung cấp giải pháp phân tích dữ liệu lớn (Big Data) với mức chi phí tối ưu nhất (Pay-as-you-go).

---

### 2. Sơ đồ kiến trúc & Quy trình hoạt động

> **Hình 1 - Sơ đồ Kiến trúc và luồng xử lý Smart Campus**
> ![Architecture Overview](/aws-image/AwsArchitecture.drawio.png)



Hệ thống Smart Campus được thiết kế theo kiến trúc Serverless 100% trên nền tảng AWS, áp dụng mô hình Event-Driven Architecture (Kiến trúc hướng sự kiện) nhằm đảm bảo hiệu năng cao, khả năng mở rộng tự động và tối ưu chi phí. Sơ đồ kiến trúc được chia thành các nhóm luồng nghiệp vụ chính như sau:

#### Nhóm 0: CI/CD Pipeline (Triển khai Tự động)
Hệ thống sử dụng bộ công cụ AWS Developer Tools để tự động hóa quá trình kiểm thử và triển khai mã nguồn mỗi khi có thay đổi.
- **C1. Push code:** Developer đẩy mã nguồn mới (Frontend/Backend) lên kho lưu trữ GitHub.
- **C2. Trigger pipeline:** AWS CodePipeline lắng nghe sự kiện từ GitHub và tự động kích hoạt luồng CI/CD.
- **C3. Build & Package:** AWS CodeBuild tải mã nguồn về, biên dịch (Build React) hoặc đóng gói thư viện (Zip Python/FastAPI) tạo thành các bản build hoàn chỉnh.
- **C4. Deploy:** 
  - *Frontend:* CodeBuild đẩy các file tĩnh (HTML/CSS/JS) lên S3 Frontend Bucket.
  - *Backend:* CodeBuild đẩy file zip lên AWS Lambda và cập nhật phiên bản mới.

#### Nhóm 1: Truy cập & Lấy Token (Access & Auth)
Bảo vệ hệ thống từ vòng ngoài và cung cấp cơ chế xác thực an toàn.
- **1a. Truy cập Web:** Người dùng gửi request truy cập từ trình duyệt.
- **1b. Secure Access (WAF):** AWS WAF kiểm tra IP và các quy tắc bảo mật trước khi cho phép request đi qua.
- **2. Tải giao diện (Serve SPA):** CloudFront lấy nội dung web tĩnh từ S3 Frontend và phân phối nhanh chóng tới người dùng qua mạng CDN toàn cầu.
- **3. Kích hoạt API:** Request nghiệp vụ từ Frontend được đẩy vào API Gateway.
- **4a & 4b. Authenticate:** API Gateway chuyển request đăng nhập cho Lambda. Lambda gọi **Amazon Cognito** để xác thực người dùng và lấy JWT Token trả về cho Frontend.
- **4c. Validate Token:** Các request sau này đều bị API Gateway chặn lại để nhờ Cognito kiểm tra tính hợp lệ của Token trước khi cho đi tiếp.

#### Nhóm 2: Quản lý Nhân sự & Đăng ký Khuôn mặt
Xử lý dữ liệu người dùng và tạo đặc trưng sinh trắc học.
- **5. Quản lý User:** Lambda đọc/ghi thông tin nhân sự cơ bản vào bảng `Users` trên DynamoDB.
- **6. Yêu cầu Đăng ký:** Luồng đăng ký khuôn mặt nhân viên mới.
- **7. Lưu ảnh gốc:** Lambda tải ảnh gốc (Raw image) lên S3 Images Bucket để làm tài liệu đối chiếu.
- **8. Trích xuất đặc trưng:** Lambda gọi **Amazon Rekognition** (IndexFaces) để trích xuất ma trận sinh trắc học.
- **9. Lưu Metadata:** ID khuôn mặt (FaceID) được lưu vào bảng `Faces` trên DynamoDB.

#### Nhóm 3: Cốt lõi Điểm danh (Face Attendance)
Đây là luồng xương sống của hệ thống, xử lý độ trễ thấp (< 1s).
- **10. Yêu cầu Điểm danh:** User checkin, hệ thông gửi ảnh checkin lên API Gateway.
- **11. Lấy thông tin:** Lambda truy vấn bảng `Users` để đối chiếu luật (Ca làm việc, giờ cho phép...).
- **12. Nhận diện:** Lambda gọi Amazon Rekognition (SearchFacesByImage) để so khớp khuôn mặt với độ chính xác cao.
- **13. Ghi nhận:** Bản ghi điểm danh được lưu ngay lập tức vào bảng `Attendance` trên DynamoDB.
- **14. Gửi Email cá nhân:** Lambda dùng **Amazon SES** gửi một biên lai điểm danh (HTML) trực tiếp vào email người đó.
- **15. Publish Event:** Để không làm chậm API, Lambda lập tức bắn sự kiện *"AttendanceRecorded"* (Điểm danh hoàn tất) lên **Amazon EventBridge** và trả về HTTP 200 cho Camera.

#### Nhóm 4: Bất đồng bộ (Event-Driven Async Flows)
Xử lý các tác vụ nặng ở background bằng kiến trúc Fan-out (1 sự kiện rẽ nhiều nhánh).
- **16a & 17a. Luồng Thông báo (Notification):** EventBridge đẩy sự kiện vào hàng đợi SQS, kích hoạt `Notification Worker Lambda`. 
- **18. Broadcast via SNS:** Worker này gọi Amazon SNS để "phát sóng" tin nhắn ra các kênh đa phương tiện (SMS, Mobile Push, Chatbot).
- **16b & 17b. Luồng Dữ liệu (Analytics):** Đồng thời, EventBridge cũng đẩy sự kiện vào SQS Analytics, kích hoạt `Analytics Worker Lambda`.
- **19. Lưu Data Lake:** Worker này đóng gói dữ liệu điểm danh thành các file JSON và đẩy vào S3 Data Lake để lưu trữ dài hạn (Cold storage) chi phí thấp.

#### Nhóm 5: Báo cáo Thống kê (Kiến trúc Hybrid / Lambda Architecture)
Kết hợp sức mạnh phân tích dữ liệu lớn và truy xuất dữ liệu thời gian thực.
- **20. Catalog Data:** Dịch vụ **AWS Glue** định kỳ quét S3 Data Lake để tự động học hỏi và tạo Lược đồ dữ liệu (Schema).
- **21. Yêu cầu Báo cáo:** User truy cập màn hình Dashboard, API gọi xuống Dashboard Lambda (Report Lambda).
- **22a. Lấy dữ liệu Real-time (Hot Data):** Dashboard Lambda query trực tiếp vào **DynamoDB** (Attendance Table / Task Table) để lấy dữ liệu điểm danh và công việc mới nhất trong ngày.
- **22b, 22c & 22d. Lấy dữ liệu Cũ (Cold Data):** Đồng thời, Dashboard Lambda yêu cầu **Amazon Athena** chạy các câu lệnh SQL siêu tốc, kết hợp với lược đồ từ Glue, để quét qua dữ liệu lịch sử trên S3 Data Lake.
- **Tổng hợp:** Lambda sẽ tự động "cộng gộp" (merge) số liệu từ cả 2 luồng này lại với nhau rồi trả về để hiển thị lên Dashboard một cách chính xác và tối ưu nhất.

#### Nhóm 6: Quản lý Công việc & Đơn từ
- **23. Yêu cầu Công việc:** Request giao việc hoặc xin nghỉ được đẩy vào Lambda.
- **24a & 24b. Đọc/Ghi DB:** Dữ liệu lưu vào bảng `Tasks` và `Leaves` độc lập.
- **24c. Lưu Thông báo:** Ghi lịch sử gửi thông báo vào bảng `Notifications`.
- **24d & 24e. Presigned URL Upload:** Thay vì tải file nặng xuyên qua Lambda, Lambda chỉ tạo ra 1 đường dẫn an toàn ngắn hạn (Presigned URL) và trả về. Trình duyệt của User sẽ dùng link này để upload PDF/Hình ảnh trực tiếp lên S3 Images, giúp tối ưu băng thông máy chủ.
- **25. Send Notification:** Gửi email báo có việc mới/đơn mới qua Amazon SES.

####  Nhóm 7: Cronjob (Quét trễ hạn)
- **26. Cron Trigger:** **EventBridge Scheduler** được hẹn giờ chạy mỗi X phút, tự động kích hoạt Lambda.
- **27. Quét trễ hạn:** Lambda quét bảng `Tasks` để tìm các công việc sát giờ hoặc đã quá hạn (Overdue).
- **28. Warning Email:** Gửi thư hối thúc nhân viên qua Amazon SES.

####  Nhóm 8: Quản trị, Bảo mật & Giám sát (Cross-cutting)
- **IAM (Identity and Access Management):** Toàn bộ các dịch vụ giao tiếp với nhau bằng nguyên tắc đặc quyền tối thiểu (Least Privilege). Lambda chỉ được ghi S3 bucket cụ thể, không được xóa bucket.
- **X-Ray & CloudWatch:** 
  - Lambda liên tục đẩy Logs/Metrics (số lượng request, thời gian xử lý) về CloudWatch.
  - AWS X-Ray vẽ bản đồ mạng nhện (Trace Map) để theo dõi request đi qua từng dịch vụ mất bao nhiêu mili-giây.
- **CloudWatch Alarms:** Khi phát hiện tỷ lệ lỗi (Faults) vượt mức cho phép, Alarm bị kích hoạt và gọi Amazon SNS bắn cảnh báo khẩn tới điện thoại của đội ngũ kỹ sư.

---

### 3. Các dịch vụ trong phạm vi  (In-Scope Services)

| STT | DỊCH VỤ AWS | VAI TRÒ & NHIỆM VỤ TRONG SMART CAMPUS | LÝ DO LỰA CHỌN & LỢI ÍCH KỸ THUẬT |
| :---: | :--- | :--- | :--- |
| 1 | **Amazon CloudFront** | Phân phối ứng dụng React Frontend từ S3 Bucket đến người dùng. Đóng vai trò mỏ neo cho AWS WAF. | Tăng tốc tải trang bằng caching ở các Edge Locations. Hỗ trợ HTTPS tự động, giảm tải băng thông. |
| 2 | **AWS WAF** | Tường lửa bảo vệ điểm danh, chặn các IP không thuộc văn phòng. | Ngăn chặn gian lận điểm danh từ xa, chống các đợt tấn công Web và Spam request. |
| 3 | **Amazon S3** | **Bucket 1:** Lưu trữ Frontend. <br> **Bucket 2:** Lưu trữ ảnh khuôn mặt & tài liệu bảo mật. <br> **Bucket 3:** S3 Data Lake lưu log. | Chi phí lưu trữ rẻ, độ tin cậy 99.999999999%. Hỗ trợ S3 Presigned URL ẩn file bảo mật. Tích hợp tốt với Athena. |
| 4 | **Amazon API Gateway** | Cổng giao tiếp RESTful/HTTP API tiếp nhận request từ Frontend và gọi AWS Lambda. | Hỗ trợ Rate Limiting, tích hợp sẵn xác thực JWT qua Cognito Authorizer mà không cần code. |
| 5 | **AWS Lambda** | **API Handler:** Xử lý logic API. <br> **Workers:** Xử lý Event chạy nền. | Mô hình Serverless Pay-As-You-Go (chỉ trả tiền khi code chạy). Tự động scale tức thì, không quản lý server. |
| 6 | **Amazon DynamoDB** | Lưu trữ toàn bộ dữ liệu nghiệp vụ (Users, Tasks, Leaves, Attendance). | Database NoSQL Serverless, tốc độ phản hồi tính bằng mili-giây, linh hoạt với Global Secondary Index. |
| 7 | **Amazon Cognito** | Quản lý User Pool, xác thực đăng nhập và cấp phát JWT Token. | Không phải tự build hệ thống Auth. Bảo mật cao, hỗ trợ bắt buộc đổi mật khẩu lần đầu đăng nhập. |
| 8 | **Amazon EventBridge** | Event Bus định tuyến các sự kiện (ví dụ: `AttendanceRecorded`) và chạy Cronjob. | Tách rời các module (Decoupling) theo chuẩn Event-Driven, dễ dàng thêm nghiệp vụ mới. |
| 9 | **Amazon SQS** | Hàng đợi tin nhắn (Queue) đứng trước các Worker. | Đảm bảo không mất dữ liệu khi có lỗi xảy ra. Tích hợp Dead Letter Queue (DLQ) để retry. |
| 10 | **Amazon Rekognition** | So khớp khuôn mặt nhân viên qua camera khi check-in. | AI có sẵn siêu mạnh, không tốn thời gian train model. Độ chính xác cao (Confidence > 95%). |
| 11 | **Glue & Athena** | Bộ đôi Glue & Athena pipeline gom log điểm danh trên S3, phân tích và truy vấn bằng SQL. | Batching file tự động để tiết kiệm phí S3/Athena. Tách bạch hệ thống OLTP và OLAP. |
| 12 | **AWS CodeBuild & CodePipeline** | Thiết lập CI/CD Pipeline tự động build Frontend và đóng gói Lambda Backend. | Triển khai liên tục (Continuous Deployment) một cách hoàn toàn tự động từ source code. Đảm bảo an toàn và nhất quán giữa các lần release. |

### 4. Kết quả mong đợi sau khi kết thúc Workshop
Kết thúc chuỗi bài thực hành này, bạn sẽ dựng hoàn chỉnh một nền tảng doanh nghiệp:
- **Frontend hoạt động tốt:** Có giao diện điểm danh và dashboard quản lý.
- **Xác thực bảo mật đa lớp:** Chống mạo danh bằng IAM Least Privilege, WAF và Cognito JWT.
- **Kiến trúc chịu tải cao:** Ứng dụng thành thạo Event-Driven (EventBridge + SQS) để triệt tiêu tình trạng thắt cổ chai giờ cao điểm.
- **Data Pipeline tự động:** Sở hữu hệ thống Data Lake tách biệt hoàn toàn OLTP và OLAP.
- **DevOps CI/CD:** Hệ thống CodePipeline tự động build và deploy code mà không cần thao tác tay.
- **Dọn dẹp (Cleanup):** Có khả năng dọn dẹp tài nguyên nhanh chóng để kiểm soát hoàn toàn chi phí AWS.

---

### 5. Định hướng phát triển tương lai

Mặc dù hệ thống Smart Campus đã hoàn thành các tính năng cốt lõi, nhóm vẫn xác định được nhiều hướng cải tiến tiềm năng để nâng cấp hệ thống lên tầm cao hơn trong tương lai:

#### 5.1. Nâng cấp hệ thống AI & Nhận diện
- **Liveness Detection (Anti-spoofing):** Tích hợp cơ chế chống giả mạo khuôn mặt bằng ảnh chụp hoặc video giả, đảm bảo độ chính xác tuyệt đối cho hệ thống điểm danh.
- **Chuyển sang Amazon Rekognition Video:** Hỗ trợ nhận diện khuôn mặt từ luồng camera trực tiếp (Live Stream) thay vì upload từng ảnh, tăng tốc độ xử lý lên nhiều lần.

#### 5.2. Phân tích dữ liệu nâng cao (Advanced Analytics)
- **Tích hợp Amazon QuickSight:** Thay vì tự vẽ biểu đồ trên Frontend, tích hợp **Amazon QuickSight** để tạo các bảng điều khiển BI (Business Intelligence) chuyên nghiệp, hỗ trợ drill-down và lọc dữ liệu đa chiều.
- **Machine Learning dự báo:** Sử dụng **Amazon SageMaker** để huấn luyện mô hình dự báo xu hướng đi muộn, dự đoán năng suất nhóm và đề xuất điều chỉnh ca làm việc tự động.
- **Real-time Streaming với Kinesis:** Thay thế SQS bằng **Amazon Kinesis Data Streams** cho luồng phân tích dữ liệu thời gian thực cực kỳ cao tải (hàng triệu event/giây).

#### 5.3. Tối ưu Hạ tầng & Chi phí
- **Infrastructure as Code (IaC):** Chuyển toàn bộ cấu hình tài nguyên AWS sang **AWS CDK** hoặc **Terraform** để quản lý hạ tầng theo phiên bản (version control) và tái sử dụng dễ dàng.
- **Multi-region Deployment:** Triển khai hệ thống trên nhiều AWS Region để đảm bảo tính sẵn sàng cao (High Availability) và giảm độ trễ cho người dùng toàn cầu.
- **AWS Savings Plans / Reserved Capacity:** Khi hệ thống đạt ngưỡng traffic ổn định, chuyển từ mô hình On-demand sang Reserved để tiết kiệm thêm 30-60% chi phí vận hành.

