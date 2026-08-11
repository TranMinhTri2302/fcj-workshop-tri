---
title: "Worklog Tuần 3"
date: 2026-07-06
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

# Tuần 3: Khởi tạo Frontend & Backend – Tích hợp Cognito, DynamoDB, S3 – Thiết lập CI/CD

## 1. Mục tiêu tuần

* Khởi tạo dự án React bằng Vite, xây dựng hệ thống CSS dùng chung, bộ component cơ bản và layout chính của ứng dụng.
* Dựng nền tảng backend Python/FastAPI theo hướng module hóa với chuẩn Repository – Service – Router.
* Tích hợp các dịch vụ AWS cốt lõi: Amazon Cognito, DynamoDB và S3 để hình thành khung hệ thống ban đầu.
* Thiết lập pipeline CI/CD cơ bản và kiểm thử bước đầu để kiểm soát chất lượng mã nguồn.

## 2. Nhật ký công việc chi tiết

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - **Frontend:** Khởi tạo dự án React bằng Vite (React + JavaScript), dọn dẹp file mặc định, thiết lập cấu trúc thư mục chuẩn: components, pages, hooks, services, utils, assets <br> - **Backend:** Thống nhất cấu trúc tổ chức code theo module: routers, services, models, schemas; chuẩn hóa naming convention và coding convention dùng chung <br> - Phân chia các module backend: users, faces, attendance, notifications, reports, ai_assistant, tasks | 06/07/2026 | 06/07/2026 | <https://vitejs.dev/guide/> <br> <https://fastapi.tiangolo.com/> |
| 3 | - **Frontend:** Xây dựng hệ thống style cơ sở theo hướng Glassmorphism: cấu hình biến CSS toàn cục, màu sắc, khoảng cách, border-radius, shadow, backdrop-filter và các utility class dùng chung <br> - **Backend:** Dựng khung backend ban đầu theo chuẩn Repository – Service – Router; tạo file cấu hình chung: settings, dependency injection, error handler <br> - Bắt đầu xây dựng luồng xác thực qua Amazon Cognito: đăng ký, đăng nhập, xác nhận tài khoản | 07/07/2026 | 07/07/2026 | <https://docs.aws.amazon.com/cognito/> |
| 4 | - **Frontend:** Xây dựng các Core Component tái sử dụng: Button, Input, Select/Dropdown, Loading, Alert/Toast, Empty State <br> - **Backend:** Hoàn thiện middleware giải mã JWT cho các route cần bảo vệ; kết nối DynamoDB: tạo bảng Users, Events, Checkins; thiết lập GSI cho Checkins để truy vấn lịch sử điểm danh theo user | 08/07/2026 | 08/07/2026 | <https://docs.aws.amazon.com/amazondynamodb/> <br> <https://boto3.amazonaws.com/v1/documentation/api/latest/> |
| 5 | - **Frontend:** Thiết kế layout chính của ứng dụng: Sidebar Navigation, Header, vùng nội dung chính, breadcrumb và khung hiển thị trang con <br> - **Backend:** Tích hợp S3: sinh presigned URL cho phép client upload ảnh trực tiếp, giảm tải cho backend; xây dựng service layer thao tác DynamoDB qua boto3 theo hướng tách biệt khỏi router | 09/07/2026 | 09/07/2026 | <https://docs.aws.amazon.com/AmazonS3/latest/userguide/> |
| 6 | - **Frontend:** Xây dựng component DataTable tái sử dụng với phân trang, sắp xếp, tìm kiếm và khả năng responsive trên mobile <br> - **Backend:** Kiểm tra toàn bộ endpoint qua Swagger UI (/docs), đối chiếu với workflow/use-case đã phân tích ở tuần 2; phát hiện và bổ sung các API còn thiếu <br> - Thiết lập CI/CD: GitHub Actions chạy pytest và lint; xây dựng test case sử dụng moto để mô phỏng DynamoDB và Cognito <br> - Rà soát tổng thể code và review đầu ra kỹ thuật trước khi tích hợp | 10/07/2026 | 11/07/2026 | <https://docs.github.com/en/actions> <br> <https://github.com/getmoto/moto> |

## 3. Đóng góp cá nhân

* Phụ trách chính phần frontend của tuần này: khởi tạo dự án Vite, thiết lập hệ thống CSS, xây dựng bộ component cơ bản, layout chính và DataTable dùng chung.
* Rà soát lại đặc tả các API cốt lõi để frontend và backend bám sát đúng workflow đã chốt ở tuần 2.
* Bổ sung phần xác thực Cognito/JWT ở mức middleware và logic bảo vệ route để làm nền cho RBAC về sau.
* Hỗ trợ tổ chức service layer cho DynamoDB, giữ cho phần xử lý dữ liệu tách khỏi router và dễ test hơn.
* Kết nối presigned URL với luồng upload file từ frontend để chuẩn bị cho tính năng xử lý ảnh khuôn mặt ở tuần tiếp theo.
* Kiểm tra lại endpoint qua Swagger UI và ghi nhận các điểm chưa thống nhất về tên trường, payload và response.

## 4. Kết quả đạt được

* **Frontend:**
  * Hoàn thành khung ứng dụng React với hệ thống CSS dùng chung, bộ Core Component và Layout chính.
  * DataTable dùng chung đã sẵn sàng cho các trang quản lý dữ liệu.

* **Backend:**
  * Dựng được nền tảng backend theo hướng module hóa với 7 nhóm chức năng chính.
  * Swagger UI hoạt động ổn định, thuận tiện cho kiểm thử thủ công và đối chiếu đặc tả.

* **Tích hợp AWS và CI/CD:**
  * Hoàn thành luồng xác thực nền tảng với Cognito/JWT.
  * DynamoDB, S3 presigned URL và pipeline GitHub Actions đã được thiết lập, tạo nền cho các tính năng nghiệp vụ ở tuần sau.