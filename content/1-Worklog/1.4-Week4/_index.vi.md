---
title: "Worklog Tuần 4"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---



### Mục tiêu tuần 4:

* Triển khai dự án Smart Campus bằng Python/FastAPI, xây dựng cấu trúc module hóa rõ ràng với các thành phần routers, services, models và schemas.
* Xây dựng hệ thống xác thực người dùng thông qua Amazon Cognito, bao gồm các flow đăng ký, đăng nhập, xác nhận tài khoản và middleware giải mã JWT.
* Thiết kế cơ sở dữ liệu DynamoDB cho các thực thể Users, Events và Checkins, đồng thời tích hợp S3 thông qua presigned URL để tối ưu hóa quá trình upload ảnh.
* Thiết lập giám sát bằng CloudWatch và xây dựng quy trình CI/CD với GitHub Actions nhằm nâng cao chất lượng và độ tin cậy của hệ thống.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Triển khai dự án Smart Campus bằng Python/FastAPI, tổ chức code theo từng module rõ ràng (routers, services, models, schemas) | 13/07/2026 | 13/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Xây dựng chức năng xác thực qua Amazon Cognito, hoàn thiện flow đăng ký – đăng nhập – xác nhận và viết middleware giải mã JWT tự động | 14/07/2026 | 14/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Kết nối DynamoDB bằng boto3 để tạo các bảng Users, Events, Checkins và thiết lập GSI cho truy vấn lịch sử điểm danh | 15/07/2026 | 15/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Phát triển các API quản lý sự kiện và điểm danh, đồng thời triển khai cơ chế chống trùng lặp bằng ConditionExpression | 16/07/2026 | 16/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Tích hợp S3 thông qua presigned URL cho phép client upload ảnh trực tiếp <br> - Cấu hình CloudWatch để thu log và tạo alarm khi lỗi vượt ngưỡng <br> - Xây dựng CI/CD trên GitHub Actions với pytest, lint và moto | 17/07/2026 | 17/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 7 | - Tổng kết tuần: nắm vững hơn về JWT/OIDC trong Cognito, thiết kế dữ liệu NoSQL và quy trình CI/CD thực tế | 18/07/2026 | 18/07/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 4:

* **Phát triển backend bằng Python/FastAPI:**
  * Triển khai dự án Smart Campus với cấu trúc module hóa để hệ thống dễ bảo trì và mở rộng hơn.

* **Xây dựng hệ thống xác thực với Cognito:**
  * Hoàn thiện toàn bộ flow đăng ký – đăng nhập – xác nhận và triển khai middleware giải mã JWT để tự động bảo vệ các route cần xác thực.

* **Kết nối database và lưu trữ:**
  * Kết nối DynamoDB cho các bảng Users, Events và Checkins, thiết lập Global Secondary Index giúp truy vấn lịch sử điểm danh theo user hiệu quả hơn.
  * Tích hợp S3 bằng presigned URL để client có thể upload ảnh trực tiếp, giảm tải cho backend.

* **Giám sát và CI/CD:**
  * Cấu hình CloudWatch để thu thập log và phát cảnh báo khi tỉ lệ lỗi vượt ngưỡng.
  * Xây dựng quy trình CI/CD bằng GitHub Actions để tự động chạy pytest và lint, đồng thời sử dụng moto để mock DynamoDB và Cognito trong kiểm thử.
  * Nắm vững hơn về JWT/OIDC trong Cognito, cách thiết kế dữ liệu NoSQL hiệu quả và quy trình CI/CD trong môi trường phát triển thực tế.



