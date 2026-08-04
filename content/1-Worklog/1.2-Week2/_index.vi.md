---
title: "Worklog Tuần 2"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---



### Mục tiêu tuần 2:

* Thiết kế sơ đồ kiến trúc hệ thống cho đề tài và chuẩn hóa theo các góp ý từ mentor và cộng đồng.
* Phân tích, so sánh các giải pháp triển khai backend như Lambda, EC2, ECS và Elastic Beanstalk, đồng thời cân nhắc tác động của vấn đề cold start.
* Tiếp tục thực hành chuyên sâu các dịch vụ AWS nền tảng, nghiên cứu VPC networking nâng cao và bắt đầu quan tâm đến các bài toán FinOps và bảo mật trong thiết kế hệ thống.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Thiết kế sơ đồ kiến trúc hệ thống cho đề tài, mô tả bài toán và luồng xử lý chính | 29/06/2026 | 29/06/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - **Tiếp nhận phản hồi từ mentor và cộng đồng:** <br>&emsp; + Bổ sung đầy đủ các thành phần AWS như Region, VPC và Subnet <br>&emsp; + Chuẩn hóa sơ đồ bằng AWS Architecture Icons <br>&emsp; + Đánh số các luồng xử lý để làm rõ workflow <br>&emsp; + Điều chỉnh thiết kế frontend/backend và cách triển khai trên S3, CloudFront và API service | 30/06/2026 | 30/06/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - **Tìm hiểu và so sánh các hướng triển khai backend:** <br>&emsp; + Lambda so với EC2/ECS/Elastic Beanstalk <br>&emsp; + Nhận thức rõ về vấn đề cold start khi dùng Java (Spring Boot) trên Lambda và cân nhắc chuyển sang Python hoặc Node.js | 01/07/2026 | 01/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - **Thực hành chuyên sâu các dịch vụ AWS:** <br>&emsp; + S3 (Versioning, Lifecycle, Static Hosting, CORS) <br>&emsp; + RDS (MySQL, backup, security) <br>&emsp; + DynamoDB (NoSQL, GSI, TTL) | 02/07/2026 | 02/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - **Tiếp tục học về dịch vụ AWS và kiến trúc mạng:** <br>&emsp; + CloudFront (CDN, caching) <br>&emsp; + CloudWatch (monitoring, logs, alarms) <br>&emsp; + VPC networking nâng cao: NAT Gateway, VPC Flow Logs, Security Group, NACL | 03/07/2026 | 03/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 7 | - Bắt đầu nghiên cứu bài toán tối ưu chi phí (FinOps) và bảo mật trong thiết kế hệ thống cloud | 04/07/2026 | 04/07/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 2:

* **Chuẩn hóa kiến trúc hệ thống:**
  * Thiết kế sơ đồ kiến trúc cho đề tài, trình bày rõ bài toán và luồng xử lý chính.
  * Tiếp thu và hệ thống hóa các góp ý từ mentor và cộng đồng để hoàn thiện sơ đồ, bao gồm việc bổ sung Region, VPC, Subnet, chuẩn hóa biểu tượng theo AWS Architecture Icons và làm rõ từng luồng xử lý.

* **Phân tích lựa chọn backend và ngôn ngữ:**
  * Tìm hiểu và so sánh các hướng triển khai backend giữa Lambda và EC2/ECS/Elastic Beanstalk.
  * Nhận thức rõ hơn về vấn đề cold start khi dùng Java trên Lambda, từ đó cân nhắc hướng chuyển sang Python hoặc Node.js phù hợp hơn với mục tiêu hệ thống.

* **Thực hành chuyên sâu dịch vụ AWS:**
  * Làm quen và thực hành với S3, RDS, DynamoDB, CloudFront và CloudWatch để hiểu rõ hơn về vai trò của từng dịch vụ trong kiến trúc hệ thống.

* **Tư duy về mạng, chi phí và bảo mật:**
  * Tiếp tục nghiên cứu VPC networking nâng cao như NAT Gateway, VPC Flow Logs, Security Group và NACL.
  * Bắt đầu hình thành tư duy về tối ưu hóa chi phí (FinOps) và bảo mật trong thiết kế cloud từ giai đoạn đầu.



