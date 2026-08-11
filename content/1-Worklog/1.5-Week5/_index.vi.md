---
title: "Worklog Tuần 5"
date: 2026-07-20
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

# Tuần 5: Quản lý Công việc UI – Notification & Analytics Pipeline – Tổng hợp PoC nhóm

## 1. Mục tiêu tuần

* Xây dựng giao diện module Quản lý công việc với các chế độ hiển thị phù hợp và tích hợp upload file trực tiếp lên S3 bằng presigned URL.
* Hoàn thiện hạ tầng dùng chung, tổng hợp PoC của nhóm vào monorepo thống nhất và chuẩn hóa môi trường phát triển.
* Hoàn thiện WF4 – Notification đa kênh theo hướng gần với production hơn.
* Xây dựng WF5 – Analytics Pipeline hai pha và bắt đầu triển khai giao diện Analytics.

## 2. Nhật ký công việc chi tiết

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - **Hạ tầng nhóm:** Hoàn thiện cấu hình AWS Organization, Organizational Unit và IAM Role để nhóm dùng chung tài nguyên có kiểm soát <br> - Tổng hợp mã nguồn PoC từ các thành viên, chuẩn hóa môi trường phát triển bằng `uv.lock`, `.gitignore` và khai báo dependency <br> - Review giao diện và backend, ghi nhận các rủi ro liên quan đến injection và dữ liệu đầu vào <br> - **Frontend:** Thiết kế giao diện module Quản lý công việc với List View và cấu trúc component ban đầu | 20/07/2026 | 20/07/2026 | <https://docs.aws.amazon.com/organizations/> |
| 3 | - Phân chia lại công việc trong nhóm theo vai trò cụ thể hơn <br> - Hoàn thành khoảng 80% sơ đồ kiến trúc AWS để gửi mentor <br> - **Frontend:** Mở rộng giao diện công việc theo dạng Kanban với các cột trạng thái chính <br> - **Backend:** Bắt đầu xây dựng WF4 – Notifications: định nghĩa 5 message template (AttendanceRecorded, AttendanceRejected, UnknownFaceDetected, SecurityIncidentCreated, Custom) | 21/07/2026 | 21/07/2026 | <https://docs.aws.amazon.com/sns/> <br> <https://docs.aws.amazon.com/ses/> |
| 4 | - **Frontend:** Tích hợp component Upload File với logic xin presigned URL rồi PUT trực tiếp file lên S3; bổ sung Progress Bar hiển thị tiến trình upload <br> - **Backend:** Hoàn thiện WF4: publish qua Amazon SNS ARN thực, ghi Audit Trail vào DynamoDB (SENT/FAILED), phát sự kiện `NotificationSent` lên EventBridge <br> - Nâng cấp luồng thông báo từ broadcast sang email cá nhân 1-1 bằng SES, fallback sang SNS khi SES lỗi hoặc thiếu email | 22/07/2026 | 22/07/2026 | <https://docs.aws.amazon.com/ses/> <br> <https://docs.aws.amazon.com/eventbridge/> |
| 5 | - **Backend:** Xây dựng WF5 – Analytics Pipeline Phase 1: truy vấn trực tiếp từ DynamoDB cho dashboard <br> - Triển khai Phase 2 – Data Lake: Lambda Worker lắng nghe AttendanceRecorded, stream qua Kinesis Firehose xuống S3 phân vùng `year/month/day`, Glue Crawler detect schema cập nhật Data Catalog, Athena truy vấn SQL <br> - **Frontend:** Bắt đầu xây dựng trang Analytics để hiển thị dữ liệu thống kê | 23/07/2026 | 23/07/2026 | <https://docs.aws.amazon.com/athena/> <br> <https://docs.aws.amazon.com/glue/> <br> <https://docs.aws.amazon.com/firehose/> |
| 6 | - **Backend:** Nâng cấp module Analytics/Reports: bổ sung endpoint và schema cho dashboard, hỗ trợ filter theo phòng ban <br> - Phát triển WF8 – Quản lý Công việc & Sự cố: bảng `smart-campus-tasks` (13 attributes, 3 GSI), chuẩn hóa attribute name sang `snake_case`, thêm unit test kiểm tra serialize/deserialize <br> - **Frontend:** Tối ưu state management cho module Quản lý công việc, kết nối API mới và hoàn thiện trang Analytics ở mức sử dụng được <br> - Viết test ban đầu cho pipeline dữ liệu chính và rà soát tổng thể các module phát triển trong tuần | 24/07/2026 | 25/07/2026 | <https://docs.aws.amazon.com/amazondynamodb/> |

## 3. Đóng góp cá nhân

* Phụ trách phần frontend của tuần này:
  * giao diện Quản lý công việc theo List View;
  * mở rộng hiển thị theo hướng Kanban;
  * upload file trực tiếp lên S3 bằng presigned URL;
  * progress bar khi upload;
  * trang Analytics ở giai đoạn đầu.
* Hỗ trợ tổng hợp PoC của nhóm vào monorepo chung, rà soát lại dependency, cấu trúc thư mục và naming để tránh vỡ môi trường khi tích hợp.
* Chuẩn hóa lại một số đầu vào kỹ thuật giữa frontend và backend, nhất là ở phần Notification, Analytics schema và payload upload file.
* Ghi nhận các rủi ro liên quan đến input/injection và phản hồi lại cho nhóm trước khi mở rộng thêm API.
* Cập nhật sơ đồ kiến trúc và mô tả lại luồng dữ liệu hai pha để bám sát hơn với phần implementation.

## 4. Kết quả đạt được

* **Frontend:**
  * Module Quản lý công việc đã có giao diện sử dụng được, kèm cơ chế upload file trực tiếp lên S3.
  * Trang Analytics được khởi tạo và bắt đầu kết nối với dữ liệu thống kê từ backend.

* **Hạ tầng và tích hợp nhóm:**
  * Môi trường phát triển dùng chung ổn định hơn sau khi chuẩn hóa dependency và cấu trúc monorepo.
  * Sơ đồ kiến trúc đạt khoảng 80%, đủ để gửi mentor và tiếp tục hoàn thiện.

* **Backend và dữ liệu:**
  * WF4 hoàn thiện với 5 template, đa kênh, audit trail và cơ chế fallback SES → SNS.
  * WF5 hình thành rõ cả hai pha: truy vấn nhanh từ DynamoDB và phân tích dài hạn qua S3/Glue/Athena.
  * WF8 được mở rộng thêm về dữ liệu và kiểm thử, tạo nền cho phần giao diện và tích hợp tuần sau.