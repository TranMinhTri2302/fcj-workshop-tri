---
title: "Worklog Tuần 7"
date: 2026-08-03
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

# Tuần 7: Leave Management – RBAC Chuẩn hóa – Workshop & Blog – Deploy S3/CloudFront

## 1. Mục tiêu tuần

* Phát triển module Leave Management ở cả mức giao diện và phần tích hợp với các luồng dữ liệu liên quan.
* Chuẩn hóa lại RBAC ở frontend và đồng bộ với permission matrix của hệ thống.
* Hoàn thành các hạng mục cá nhân bắt buộc như blog kỹ thuật, cập nhật GitHub và phần báo cáo liên quan đến workshop.
* Cấu hình deploy frontend lên Amazon S3 kết hợp CloudFront.

## 2. Nhật ký công việc chi tiết

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Rà soát lại cấu trúc thư mục báo cáo workshop, nội dung minh họa và tiến độ hoàn thành trong nhóm <br> - Bắt đầu xây dựng giao diện cho module Leave Management, xác định các trạng thái hiển thị chính trên lịch và biểu mẫu đăng ký <br> - Đồng bộ lại các loại đơn nghỉ, trạng thái xét duyệt và cách hiển thị tương ứng trên giao diện | 03/08/2026 | 03/08/2026 | <https://docs.aws.amazon.com/amazondynamodb/> |
| 3 | - Xây dựng trang `Leaves.jsx` với Interactive Calendar tô màu theo trạng thái (Ngày lễ, Nghỉ phép, WFH, Cuối tuần) <br> - Hoàn thiện form đăng ký nghỉ phép, tự động điền ngày khi chọn trực tiếp trên lịch <br> - Kiểm tra lại cấu trúc dữ liệu trả về cho các loại đơn và trạng thái phê duyệt để bảo đảm hiển thị chính xác | 04/08/2026 | 04/08/2026 | — |
| 4 | - Hoàn thiện `Leaves.jsx`: nút Check-in WFH cho nhân sự đã được phê duyệt, đồng bộ PRESENT sang Attendance; nút Hủy và badge “Đã hủy” phân biệt với “Từ chối” <br> - Hoàn thiện style theo giao diện chung, tối ưu lại trải nghiệm cho lịch và biểu mẫu <br> - Rà soát permission mapping để bảo đảm từng role nhìn thấy đúng chức năng liên quan đến đơn từ <br> - Hỗ trợ kiểm tra thêm các thay đổi về role, department và trạng thái task liên quan đến permission matrix mới | 05/08/2026 | 05/08/2026 | — |
| 5 | - Xây dựng `PermissionGuard` để ẩn/disable UI theo role trong JWT <br> - Đồng bộ lại giao diện với permission matrix mới sau khi hệ thống rút gọn còn 5 role <br> - Kiểm tra lại các trường dữ liệu chịu ảnh hưởng từ migration schema, nhất là ở phần role, department, trạng thái đơn và luồng điểm danh liên quan WFH <br> - Rà soát thêm các trường hợp lỗi phát sinh do dữ liệu cũ chưa đồng bộ | 06/08/2026 | 06/08/2026 | — |
| 6 | - Cấu hình deploy frontend lên Amazon S3 với Static Website Hosting và thiết lập CloudFront Distribution, HTTPS, kiểm tra lại UI trên nhiều thiết bị <br> - Soạn, viết và đăng **3 bài blog kỹ thuật** lên group AWS Study Group; chỉnh sửa và deploy lại GitHub cá nhân phục vụ phần minh chứng <br> - Cập nhật báo cáo thu hoạch cho các sự kiện đã tham dự và đồng bộ nội dung liên quan vào phần workshop/report tổng hợp | 07/08/2026 | 08/08/2026 | <https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html> <br> <https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/> |

## 3. Đóng góp cá nhân

* Phụ trách phần giao diện của tuần này:
  * trang `Leaves.jsx` với Interactive Calendar;
  * form đăng ký nghỉ phép;
  * nút Check-in WFH;
  * badge phân biệt trạng thái;
  * `PermissionGuard` cho RBAC ở giao diện;
  * deploy frontend lên S3 + CloudFront.
* Theo sát module Leave Management từ phần hiển thị đến logic đồng bộ với Attendance, đặc biệt ở trường hợp WFH đã được duyệt.
* Rà soát lại permission matrix giữa giao diện và hệ thống để tránh việc quyền bị hard-code rải rác.
* Kiểm tra lại các trường dữ liệu bị ảnh hưởng khi schema thay đổi để giảm lỗi phát sinh ở giai đoạn tích hợp.
* Tự chuẩn bị và hoàn thành 3 bài blog kỹ thuật cho nhóm, đồng thời cập nhật GitHub cá nhân và phần báo cáo liên quan đến workshop/sự kiện.

## 4. Kết quả đạt được

* Hoàn thiện module Leave Management với lịch tương tác, form đăng ký và xử lý WFH.
* RBAC ở giao diện rõ ràng hơn nhờ `PermissionGuard`.
* Frontend deploy thành công trên S3 + CloudFront với HTTPS.
* Các thay đổi về role, permission và schema dữ liệu được phản ánh lại rõ hơn ở phía giao diện.
* Hoàn thành 3 bài blog kỹ thuật và cập nhật phần báo cáo liên quan đến workshop/sự kiện.