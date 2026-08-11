---
title: "Worklog Tuần 8"
date: 2026-08-10
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

# Tuần 8: Face Liveness – Workshop Documentation – Rà soát hệ thống và tổng kết đồ án

## 1. Mục tiêu tuần

* Hoàn thiện phần giao diện và kiểm thử cho các luồng cuối kỳ, đặc biệt là Face Liveness và các tính năng liên quan đến điểm danh.
* Rà soát lại kết quả giám sát hệ thống, cảnh báo và các thành phần phục vụ nghiệm thu để bảo đảm phần giao diện phản ánh đúng luồng thực tế.
* Hoàn thiện workshop documentation, báo cáo thu hoạch, tài liệu kỹ thuật và phần tổng kết đồ án cuối kỳ.
* Chuẩn bị tài liệu bàn giao và slide phục vụ trình diễn/nghiệm thu.

## 2. Nhật ký công việc chi tiết

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Rà soát lại các luồng hệ thống sau khi nhóm bổ sung phần observability, đối chiếu Service Map, trace và các điểm cảnh báo với hành vi thực tế trên giao diện <br> - Ghi chú lại các thành phần cần đưa vào workshop documentation và phần tổng hợp kỹ thuật cuối kỳ, đặc biệt ở nhóm Monitoring/Tracing và luồng xử lý điểm danh <br> - Kiểm tra lại các response lỗi và trạng thái bất thường có thể ảnh hưởng đến trải nghiệm người dùng | 10/08/2026 | 10/08/2026 | <https://docs.aws.amazon.com/xray/> |
| 3 | - Kiểm tra lại luồng cảnh báo và các trường hợp lỗi có ảnh hưởng đến trải nghiệm người dùng, đặc biệt trong các tình huống hệ thống phản hồi lỗi hoặc timeout <br> - Rà soát phần giao diện/ghi chú liên quan đến pipeline sự kiện để bảo đảm tài liệu mô tả đúng với hệ thống đang chạy <br> - Hỗ trợ đối chiếu thêm các điểm thay đổi trong flow xử lý để phần tổng kết kỹ thuật không bị lệch so với implementation | 11/08/2026 | 11/08/2026 | <https://docs.aws.amazon.com/AmazonCloudWatch/> <br> <https://docs.aws.amazon.com/sns/> |
| 4 | - Tích hợp giao diện Face Liveness bằng SDK `@aws-amplify/ui-react-liveness`, thay thế dần luồng chụp frame tĩnh trong `Attendance.jsx` <br> - Kiểm tra lại các trạng thái hiển thị, hướng dẫn người dùng căn chỉnh khuôn mặt và luồng chuyển tiếp sau khi nhận kết quả từ Face Liveness <br> - Đối chiếu lại ngưỡng `confidence` và dữ liệu phản hồi để phần giao diện xử lý đúng trường hợp đạt/không đạt | 12/08/2026 | 12/08/2026 | <https://docs.aws.amazon.com/rekognition/latest/dg/face-liveness.html> <br> <https://ui.docs.amplify.aws/react/connected-components/liveness> |
| 5 | - Thực hiện System End-to-End Testing cho toàn bộ hệ thống, rà soát lại các luồng chính trên giao diện trước khi bàn giao <br> - Hoàn thiện **Workshop Documentation** theo FCJ Workshop Template, bổ sung ảnh minh họa và mô tả kỹ thuật theo từng phần <br> - Viết/chỉnh nội dung cho các mục: Workshop Overview, Prerequisite, Auth & Security, Database & Storage, Data Analytics, Monitoring & Tracing, Testing & Validation | 13/08/2026 | 13/08/2026 | FCJ Workshop Template |
| 6 | - Hoàn tất phần workshop documentation, tổng hợp lại tài liệu kỹ thuật như architecture, API, database schema và các luồng chính của hệ thống <br> - Cập nhật báo cáo thu hoạch cho 3 sự kiện đã tham dự, đối chiếu lại nội dung với phần workshop/report tổng hợp <br> - Rà soát toàn bộ kiến trúc hệ thống lần cuối, viết phần tổng kết đồ án, định hướng phát triển tương lai, chuẩn bị tài liệu bàn giao và slide thuyết trình | 14/08/2026 | 15/08/2026 | — |

## 3. Đóng góp cá nhân

* Phụ trách phần giao diện liên quan đến Face Liveness, kiểm thử end-to-end và rà soát lại trải nghiệm người dùng trước khi bàn giao.
* Cập nhật và hoàn thiện đáng kể phần workshop documentation, đặc biệt ở các mục sau của phần 5 - Workshop:
  * Workshop Overview
  * Prerequisite
  * Auth & Security
  * Database & Storage
  * Data Analytics
  * Monitoring & Tracing
  * Testing & Validation
* Theo dõi phần observability ở mức tích hợp và đối chiếu kết quả trace/cảnh báo với luồng hệ thống thực tế để bảo đảm phần mô tả kỹ thuật khớp với sản phẩm.
* Bổ sung phần tổng hợp kỹ thuật, báo cáo thu hoạch sự kiện và tài liệu bàn giao cuối kỳ.

## 4. Kết quả đạt được

* Hoàn thiện phần giao diện Face Liveness và rà soát lại toàn bộ luồng người dùng quan trọng trước khi nghiệm thu.
* System End-to-End Testing giúp phát hiện và chỉnh lại các lỗi còn sót ở giai đoạn cuối.
* Hoàn tất workshop documentation, tài liệu kỹ thuật, báo cáo thu hoạch sự kiện, phần tổng kết đồ án và tài liệu bàn giao.
* Phần Monitoring/Tracing và các nội dung kỹ thuật liên quan được cập nhật sát hơn với hệ thống thực tế.
* Hệ thống Smart Campus Platform đạt mức hoàn chỉnh để trình diễn và nghiệm thu cuối kỳ.