---
title: "Worklog Tuần 7"
date: 2026-08-03
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

# Tuần 7: Leave Management – RBAC Chuẩn hóa – Workshop & Blog – Deploy S3/CloudFront

## 1. Mục tiêu tuần

* Phát triển module Leave Management ở cả mức dữ liệu, API và giao diện người dùng.
* Chuẩn hóa lại hệ thống RBAC trên toàn codebase và xử lý migration dữ liệu DynamoDB cũ.
* Hoàn thành các hạng mục cá nhân bắt buộc của chương trình như blog kỹ thuật, cập nhật GitHub và phần báo cáo liên quan đến workshop.
* Cấu hình deploy frontend lên Amazon S3 kết hợp CloudFront.

## 2. Nhật ký công việc chi tiết

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Thúc đẩy tiến độ hoàn thành workshop trong nhóm để kịp hạn nộp, đồng thời rà soát lại cấu trúc thư mục báo cáo và nội dung cần minh họa <br> - Bắt đầu phát triển module **Leave Management**: thiết kế bảng `smart-campus-leaves` với GSI `user_id-index`, `status-index`; định nghĩa các attribute chính như `leave_id`, `user_id`, `leave_type`, `date_from`, `date_to`, `reason`, `status`, `approved_by`, `approved_at`, `cancel_reason` <br> - Bổ sung 4 loại đơn: WFH, ANNUAL_LEAVE, SICK_LEAVE, BUSINESS_TRIP | 03/08/2026 | 03/08/2026 | <https://docs.aws.amazon.com/amazondynamodb/> |
| 3 | - **Backend:** Hoàn thiện API Leave Management: luồng nhân viên nộp đơn → quản lý duyệt/từ chối; API quản lý ngày lễ cho Admin với bảng `smart-campus-holidays` <br> - **Frontend:** Xây dựng trang Leaves.jsx với Interactive Calendar tô màu theo trạng thái (Ngày lễ, Nghỉ phép, WFH, Cuối tuần); form đăng ký tự động điền ngày khi chọn trên lịch | 04/08/2026 | 04/08/2026 | — |
| 4 | - **Frontend:** Hoàn thiện Leaves.jsx: nút Check-in WFH cho nhân sự đã được phê duyệt, đồng bộ PRESENT sang Attendance mà không cần Rekognition; nút Hủy và badge “Đã hủy” phân biệt với “Từ chối”; hoàn thiện style theo giao diện chung <br> - **Backend:** Chuẩn hóa phân quyền: rút gọn còn 5 role (ADMIN, DIRECTOR, MANAGER, STAFF, TECHNICIAN), đổi department `MAINTENANCE` → `TECHNICAL`, mở quyền DIRECTOR quản lý Users và WAF, giới hạn STAFF chỉ tạo task loại INCIDENT | 05/08/2026 | 05/08/2026 | — |
| 5 | - **Backend:** Tập trung permission matrix trong `permissions.py` <br> - **Frontend:** Xây dựng `PermissionGuard` để ẩn/disable UI theo role trong JWT <br> - Viết script migration cập nhật dữ liệu DynamoDB cũ để xử lý lỗi 500 do schema không tương thích <br> - Hoàn thiện luồng Check-out điểm danh, sửa lỗi hard-code thời gian Check-in WFH | 06/08/2026 | 06/08/2026 | — |
| 6 | - **Deploy Frontend:** Cấu hình Amazon S3 Bucket với Static Website Hosting và Bucket Policy; thiết lập Amazon CloudFront Distribution làm CDN, cấu hình HTTPS và kiểm tra lại UI trên nhiều thiết bị <br> - Soạn, viết và đăng **3 bài blog kỹ thuật** lên group AWS Study Group; chỉnh sửa và deploy lại GitHub cá nhân phục vụ phần minh chứng <br> - Cập nhật báo cáo thu hoạch cho các sự kiện đã tham dự và đồng bộ nội dung liên quan vào phần workshop/report tổng hợp | 07/08/2026 | 08/08/2026 | <https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html> <br> <https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/> |

## 3. Đóng góp cá nhân

* Phụ trách phần frontend của tuần này:
  * trang Leaves.jsx với Interactive Calendar;
  * form đăng ký nghỉ phép;
  * nút Check-in WFH;
  * badge phân biệt trạng thái;
  * PermissionGuard cho RBAC ở giao diện;
  * deploy frontend lên S3 + CloudFront.
* Theo sát module Leave Management từ phần giao diện đến logic đồng bộ với Attendance, đặc biệt ở trường hợp WFH đã được duyệt.
* Rà soát lại permission matrix giữa backend và frontend để tránh việc quyền bị hard-code rải rác.
* Hỗ trợ cập nhật migration cho dữ liệu DynamoDB cũ, giúp giảm lỗi phát sinh khi schema thay đổi.
* Tự chuẩn bị và hoàn thành 3 bài blog kỹ thuật, đồng thời cập nhật GitHub cá nhân và phần báo cáo liên quan đến workshop/sự kiện.

## 4. Kết quả đạt được

* **Frontend:**
  * Hoàn thiện module Leave Management với lịch tương tác, form đăng ký và xử lý WFH.
  * RBAC ở giao diện rõ ràng hơn nhờ PermissionGuard.
  * Frontend deploy thành công trên S3 + CloudFront với HTTPS.

* **Backend và dữ liệu:**
  * Leave Management API hoạt động với luồng duyệt rõ ràng.
  * RBAC được chuẩn hóa về 5 role và permission matrix tập trung hơn.
  * Migration dữ liệu cũ giúp xử lý dứt điểm một số lỗi 500 do schema không đồng bộ.

* **Minh chứng cá nhân và báo cáo:**
  * Hoàn thành 3 bài blog kỹ thuật.
  * Cập nhật GitHub cá nhân và phần báo cáo thu hoạch/workshop liên quan đến các sự kiện đã tham dự.