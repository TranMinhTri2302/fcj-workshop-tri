---
title: "Worklog Tuần 2"
date: 2026-06-29
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

# Tuần 2: Phân tích bài toán – Thiết kế kiến trúc hệ thống – Lựa chọn stack kỹ thuật

## 1. Mục tiêu tuần

* Cập nhật yêu cầu đồ án, thống nhất đề tài nhóm và phân tích bài toán ở mức hệ thống.
* Xác định actor, use-case và chuẩn hóa các workflow nghiệp vụ cốt lõi để làm đầu vào cho quá trình phát triển.
* Thiết kế sơ đồ kiến trúc tổng thể trên AWS và cập nhật lại theo phản hồi của mentor.
* So sánh các hướng triển khai backend, lựa chọn stack kỹ thuật phù hợp và khởi tạo cấu trúc monorepo.

## 2. Nhật ký công việc chi tiết

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Cập nhật yêu cầu đồ án và hướng dẫn thực tập từ phía chương trình <br> - Làm quen với nhóm mới, trao đổi về năng lực và sở trường kỹ thuật của từng thành viên <br> - Thảo luận giữa các hướng đề tài: Smart Campus, Ticket System; đánh giá theo phạm vi nghiệp vụ, khả năng thể hiện nhiều dịch vụ AWS và mức độ mở rộng của hệ thống | 29/06/2026 | 29/06/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Thống nhất đề tài **Smart Campus Platform** – hệ thống quản lý công việc, điểm danh và nhân sự trên nền tảng AWS <br> - Phân tích bài toán: xác định 5 nhóm actor (nhân viên, quản lý, kỹ thuật viên, giám đốc, admin), liệt kê use-case theo từng vai trò <br> - Xác định mối liên hệ giữa các phân hệ: người dùng, điểm danh, công việc, thông báo, phân tích, trợ lý AI, giám sát | 30/06/2026 | 30/06/2026 | <https://docs.aws.amazon.com/> |
| 4 | - Nghiên cứu chuyên sâu các dịch vụ AWS dự kiến sử dụng: Cognito, DynamoDB, S3, Rekognition, Lambda, API Gateway, SNS, SES, EventBridge, Kinesis Firehose, Glue, Athena, Bedrock, X-Ray, CloudWatch <br> - Đọc tài liệu chính thức và ghi chú use-case phù hợp của từng dịch vụ với bài toán Smart Campus | 01/07/2026 | 01/07/2026 | <https://docs.aws.amazon.com/> |
| 5 | - Thiết kế sơ đồ kiến trúc hệ thống phiên bản đầu tiên, mô tả bài toán và các luồng xử lý chính <br> - Định nghĩa **8 workflow nghiệp vụ cốt lõi**: <br>&emsp; + WF1: Authentication (Cognito JWT) <br>&emsp; + WF2: Face Registration <br>&emsp; + WF3: Attendance + Rule Engine <br>&emsp; + WF4: Notification <br>&emsp; + WF5: Analytics <br>&emsp; + WF6: AI Assistant (Bedrock NL2SQL) <br>&emsp; + WF7: Security Monitoring <br>&emsp; + WF8: Task & Employee Management | 02/07/2026 | 02/07/2026 | <https://docs.aws.amazon.com/> <br> AWS Architecture Icons |
| 6 | - Gửi sơ đồ kiến trúc lên cộng đồng WhatsApp để nhận góp ý từ mentor: bổ sung Region, VPC, Subnet; chuẩn hóa AWS Architecture Icons; đánh số luồng xử lý; điều chỉnh cách tách FE/BE và phương án deploy <br> - Cập nhật lại sơ đồ kiến trúc theo phản hồi đã nhận <br> - So sánh các hướng triển khai backend theo gợi ý mentor: Lambda vs EC2/ECS/Elastic Beanstalk; nhận diện vấn đề cold start với Java/Spring Boot trên Lambda và chuyển hướng sang Python/FastAPI <br> - Tìm hiểu thiết kế private/public architecture, quản lý secrets bằng AWS Secrets Manager, tránh hard-code credential <br> - Khởi tạo cấu trúc thư mục monorepo, cài đặt môi trường Python 3.11 | 03/07/2026 | 04/07/2026 | <https://docs.aws.amazon.com/secretsmanager/> <br> AWS Well-Architected Framework |

## 3. Đóng góp cá nhân

* Phụ trách phần phân tích bài toán ở mức đầu vào kỹ thuật: xác định actor, gom use-case và hệ thống hóa lại thành 8 workflow để nhóm có chung cách hiểu trước khi code.
* Chuẩn hóa cách mô tả workflow và tài liệu hóa các luồng chính để thuận tiện cho việc chia module.
* Cập nhật sơ đồ kiến trúc theo phản hồi của mentor, nhất là phần VPC, Subnet, Region, icon AWS và đánh số luồng xử lý.
* Đề xuất hướng backend dùng **Python/FastAPI** để phù hợp hơn với mục tiêu PoC nhanh, serverless-friendly và giảm rủi ro cold start.
* Hỗ trợ khởi tạo monorepo và chuẩn bị phần architecture diagram/workflow description cho giai đoạn triển khai.

## 4. Kết quả đạt được

* **Phân tích nghiệp vụ:**
  * Thống nhất đề tài Smart Campus Platform với 5 nhóm actor và tập use-case tương ứng.
  * Chuẩn hóa 8 workflow cốt lõi làm nền tảng phân công và triển khai cho các tuần sau.

* **Thiết kế kiến trúc:**
  * Hoàn thiện sơ đồ kiến trúc hệ thống ở mức đủ để mentor phản biện và nhóm bắt đầu phát triển.
  * Làm rõ hơn các thành phần mạng và ranh giới giữa frontend, backend, dữ liệu và giám sát.

* **Lựa chọn công nghệ:**
  * Chốt hướng backend ưu tiên là Python/FastAPI theo định hướng serverless.
  * Khởi tạo cấu trúc monorepo và môi trường phát triển ban đầu để sẵn sàng bước sang giai đoạn code.