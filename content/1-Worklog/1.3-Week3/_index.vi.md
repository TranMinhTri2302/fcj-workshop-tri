---
title: "Worklog Tuần 3"
date: 2026-07-06
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

# Tuần 3: Khởi tạo Frontend – Chuẩn hóa giao diện dùng chung – Đồng bộ API nền tảng

## 1. Mục tiêu tuần

* Khởi tạo dự án React bằng Vite và thiết lập cấu trúc thư mục chuẩn cho frontend.
* Xây dựng hệ thống CSS dùng chung, các component cơ bản và layout chính của ứng dụng.
* Chuẩn bị lớp gọi API phía frontend, đồng bộ lại đặc tả endpoint với workflow đã phân tích ở tuần 2.
* Hỗ trợ rà soát một số thành phần nền tảng liên quan đến xác thực, dữ liệu và upload file để chuẩn bị cho giai đoạn phát triển tính năng.

## 2. Nhật ký công việc chi tiết

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Khởi tạo dự án React bằng Vite (React + JavaScript), dọn dẹp file mặc định và thiết lập cấu trúc thư mục chuẩn: `components`, `pages`, `hooks`, `services`, `utils`, `assets` <br> - Thống nhất cách tổ chức mã nguồn để thuận tiện cho việc mở rộng theo module ở các tuần sau <br> - Rà soát lại workflow đã chốt ở tuần 2 để xác định các màn hình và nhóm chức năng cần ưu tiên triển khai trước | 06/07/2026 | 06/07/2026 | <https://vitejs.dev/guide/> |
| 3 | - Xây dựng hệ thống style cơ sở theo hướng Glassmorphism: cấu hình biến CSS toàn cục, màu sắc, khoảng cách, border-radius, shadow và các utility class dùng chung <br> - Chuẩn bị cấu trúc router, layout tổng thể và khung hiển thị chung của ứng dụng <br> - Đồng thời rà soát phần đặc tả API xác thực để bảo đảm phần giao diện bám sát đúng luồng đăng nhập/đăng ký dự kiến | 07/07/2026 | 07/07/2026 | <https://react.dev/> <br> <https://docs.aws.amazon.com/cognito/> |
| 4 | - Xây dựng các Core Component tái sử dụng: Button, Input, Select/Dropdown, Loading, Alert/Toast, Empty State <br> - Kiểm tra lại naming convention của các trường dữ liệu giữa giao diện và API nền tảng, đặc biệt ở nhóm user và xác thực <br> - Hỗ trợ rà soát cách bảo vệ route và xử lý JWT token ở mức đầu vào để chuẩn bị cho RBAC về sau | 08/07/2026 | 08/07/2026 | <https://docs.aws.amazon.com/amazondynamodb/> |
| 5 | - Thiết kế layout chính của ứng dụng: Sidebar Navigation, Header, vùng nội dung chính, breadcrumb và khung hiển thị trang con <br> - Chuẩn bị lớp `services` phía frontend để phục vụ gọi API, lưu token và xử lý request/response chung <br> - Rà soát luồng upload file bằng presigned URL để bảo đảm frontend có thể tích hợp thuận lợi ở giai đoạn sau | 09/07/2026 | 09/07/2026 | <https://docs.aws.amazon.com/AmazonS3/latest/userguide/> |
| 6 | - Xây dựng component DataTable tái sử dụng với phân trang, sắp xếp, tìm kiếm và khả năng responsive trên mobile <br> - Kiểm tra lại endpoint qua Swagger UI và đối chiếu với giao diện dự kiến để ghi nhận các điểm cần thống nhất về payload, response và status code <br> - Hỗ trợ rà soát cấu trúc dữ liệu trả về cho các module chính nhằm tránh lệch schema khi tích hợp ở các tuần sau | 10/07/2026 | 11/07/2026 | <https://fastapi.tiangolo.com/> <br> <https://docs.github.com/en/actions> |

## 3. Đóng góp cá nhân

* Phụ trách chính phần khởi tạo frontend: dựng dự án Vite, cấu trúc thư mục, hệ thống CSS dùng chung, bộ component cơ bản, layout chính và DataTable tái sử dụng.
* Rà soát lại đặc tả API cốt lõi để phần giao diện bám sát đúng workflow đã chốt ở tuần 2.
* Chuẩn bị lớp gọi API và cách tổ chức dữ liệu phía frontend để thuận tiện cho việc tích hợp với xác thực, upload file và các module nghiệp vụ ở giai đoạn sau.
* Hỗ trợ kiểm tra các điểm liên quan đến JWT, payload/response và naming convention để giảm sai lệch khi ghép với phần xử lý phía server.

## 4. Kết quả đạt được

* Hoàn thành khung ứng dụng React với cấu trúc thư mục rõ ràng, hệ thống CSS dùng chung, bộ Core Component và layout chính.
* DataTable dùng chung đã sẵn sàng cho các trang quản lý dữ liệu.
* Đồng bộ được các endpoint nền tảng với workflow đã phân tích, giúp giảm lệch giữa giao diện và API.
* Chuẩn bị xong lớp gọi API và tiền đề cho các luồng xác thực, upload ảnh và hiển thị dữ liệu ở các tuần sau.