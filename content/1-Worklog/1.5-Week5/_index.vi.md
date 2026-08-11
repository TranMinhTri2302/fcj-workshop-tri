---
title: "Worklog Tuần 5"
date: 2026-07-20
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

# Tuần 5: Quản lý Công việc UI – Analytics ban đầu – Tổng hợp PoC và chuẩn hóa tích hợp

## 1. Mục tiêu tuần

* Xây dựng giao diện module Quản lý công việc với các chế độ hiển thị phù hợp và tích hợp upload file trực tiếp lên S3 bằng presigned URL.
* Hỗ trợ tổng hợp PoC của nhóm vào monorepo thống nhất và chuẩn hóa môi trường phát triển ở mức cần thiết cho frontend.
* Chuẩn bị phần giao diện và dữ liệu hiển thị cho Notification, Analytics và các module liên quan.
* Cập nhật lại sơ đồ kiến trúc và mô tả luồng dữ liệu theo phần implementation đã có.

## 2. Nhật ký công việc chi tiết

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Hỗ trợ rà soát mã nguồn PoC từ các thành viên để bảo đảm giao diện có thể tích hợp ổn định trong monorepo chung <br> - Kiểm tra lại dependency, cấu trúc thư mục và các cấu hình cần thiết cho môi trường phát triển dùng chung <br> - Thiết kế giao diện module Quản lý công việc với List View và cấu trúc component ban đầu | 20/07/2026 | 20/07/2026 | <https://docs.aws.amazon.com/organizations/> |
| 3 | - Mở rộng giao diện công việc theo hướng Kanban với các cột trạng thái chính <br> - Chuẩn hóa cách hiển thị các trạng thái task để đồng bộ với workflow đang được nhóm triển khai <br> - Cập nhật sơ đồ kiến trúc và rà soát lại các thành phần đã thay đổi sau giai đoạn baseline | 21/07/2026 | 21/07/2026 | <https://react.dev/> |
| 4 | - Tích hợp component Upload File với logic xin presigned URL rồi PUT trực tiếp file lên S3 <br> - Bổ sung Progress Bar hiển thị tiến trình upload và xử lý các trạng thái thành công/thất bại ở giao diện <br> - Đồng bộ lại payload upload file, tên trường và cách lưu liên kết file để tránh lệch giữa giao diện và API <br> - Kiểm tra lại thêm một số đầu ra của luồng Notification để chuẩn bị cho phần hiển thị thông báo về sau | 22/07/2026 | 22/07/2026 | <https://docs.aws.amazon.com/AmazonS3/latest/userguide/> |
| 5 | - Bắt đầu xây dựng trang Analytics để hiển thị dữ liệu thống kê từ hệ thống <br> - Rà soát lại schema dữ liệu trả về cho dashboard, đặc biệt ở các nhóm số liệu điểm danh, công việc và bộ lọc theo phòng ban <br> - Ghi chú lại luồng dữ liệu hai pha của Analytics để bảo đảm phần hiển thị phản ánh đúng nguồn dữ liệu | 23/07/2026 | 23/07/2026 | <https://docs.aws.amazon.com/athena/> <br> <https://docs.aws.amazon.com/glue/> |
| 6 | - Tối ưu state management cho module Quản lý công việc, kết nối các endpoint mới và hoàn thiện trang Analytics ở mức sử dụng được <br> - Kiểm tra lại các response liên quan đến Notification, Analytics và Tasks để đồng bộ naming convention giữa giao diện và hệ thống <br> - Hỗ trợ rà soát một số trường dữ liệu và cách serialize/deserialize để giảm lỗi tích hợp về sau <br> - Cập nhật lại phần mô tả kỹ thuật cho sát với implementation hiện có | 24/07/2026 | 25/07/2026 | <https://docs.aws.amazon.com/amazondynamodb/> |

## 3. Đóng góp cá nhân

* Phụ trách phần giao diện của tuần này:
  * module Quản lý công việc theo List View;
  * mở rộng hiển thị theo hướng Kanban;
  * upload file trực tiếp lên S3 bằng presigned URL;
  * progress bar khi upload;
  * trang Analytics ở giai đoạn đầu.
* Hỗ trợ tổng hợp PoC vào monorepo chung, rà soát dependency, cấu trúc thư mục và naming để tránh lỗi môi trường khi tích hợp.
* Chuẩn hóa lại một số đầu vào kỹ thuật giữa giao diện và API, nhất là ở phần upload file, Notification, Analytics schema và trạng thái task.
* Cập nhật sơ đồ kiến trúc và mô tả lại luồng dữ liệu hai pha để bám sát hơn với phần implementation hiện có.

## 4. Kết quả đạt được

* Module Quản lý công việc đã có giao diện sử dụng được, kèm cơ chế upload file trực tiếp lên S3.
* Trang Analytics được khởi tạo và bắt đầu kết nối với dữ liệu thống kê từ hệ thống.
* Môi trường phát triển dùng chung ổn định hơn sau khi chuẩn hóa dependency và cấu trúc monorepo.
* Notification, Analytics và Tasks đã có đầu vào/đầu ra rõ ràng hơn ở phía giao diện và tài liệu kỹ thuật.