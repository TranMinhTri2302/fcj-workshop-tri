---
title: "Worklog Tuần 8"
date: 2026-08-10
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

# Tuần 8: Observability & Message Queue – Face Liveness – Workshop Documentation – Tổng kết đồ án

## 1. Mục tiêu tuần

* Bổ sung lớp giám sát – quan sát cho hệ thống bằng AWS X-Ray, CloudWatch Alarm và SNS Alerting.
* Tăng độ tin cậy của pipeline sự kiện thông qua Amazon SQS.
* Nghiên cứu và xây dựng PoC chống gian lận điểm danh bằng Amazon Rekognition Face Liveness.
* Hoàn thiện workshop documentation, báo cáo thu hoạch, tài liệu kỹ thuật và phần tổng kết đồ án cuối kỳ.

## 2. Nhật ký công việc chi tiết

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Triển khai lớp **Observability** với AWS X-Ray: thêm `aws-xray-sdk` vào `requirements.txt`, gọi `patch_all()` trong `main.py` để trace các lời gọi boto3 tới DynamoDB, Rekognition và S3 <br> - Xử lý xung đột Segment trên Lambda: kiểm tra biến `AWS_LAMBDA_FUNCTION_NAME` tại startup; nếu chạy trên Lambda thì không gắn `XRayMiddleware`, chỉ giữ `patch_all()` để trace sub-segment <br> - Kiểm tra Service Map và Trace Timeline trên X-Ray Console, ghi nhận `Rekognition:SearchFacesByImage` là tác vụ có độ trễ cao nhất | 10/08/2026 | 10/08/2026 | <https://docs.aws.amazon.com/xray/> |
| 3 | - Thiết lập **CloudWatch Alarm + SNS Alerting**: theo dõi metric `Errors` của Lambda `smart-campus-api`, gửi email cảnh báo cho Admin khi hệ thống gặp sự cố <br> - Tách riêng lỗi hệ thống (5xx) và lỗi người dùng (4xx) để tránh false alarm <br> - Kiểm chứng cơ chế cảnh báo bằng cách tạo lỗi có kiểm soát <br> - Bổ sung **Amazon SQS** giữa EventBridge và Lambda như một lớp đệm cho pipeline sự kiện | 11/08/2026 | 11/08/2026 | <https://docs.aws.amazon.com/AmazonCloudWatch/> <br> <https://docs.aws.amazon.com/sns/> <br> <https://docs.aws.amazon.com/sqs/> |
| 4 | - Nghiên cứu và triển khai PoC **Amazon Rekognition Face Liveness**: backend dùng `CreateFaceLivenessSession` → `GetFaceLivenessSessionResults`, chặn nếu `confidence < 90%`; nếu đạt ngưỡng thì mới tiếp tục `SearchFacesByImage` bằng `ReferenceImage` từ session <br> - **Frontend:** Tích hợp giao diện Face Liveness bằng SDK `@aws-amplify/ui-react-liveness`, thay thế dần luồng chụp frame tĩnh trong Attendance.jsx | 12/08/2026 | 12/08/2026 | <https://docs.aws.amazon.com/rekognition/latest/dg/face-liveness.html> <br> <https://ui.docs.amplify.aws/react/connected-components/liveness> |
| 5 | - Thực hiện System End-to-End Testing cho toàn bộ hệ thống, rà soát lại các luồng chính trên giao diện và backend trước khi bàn giao <br> - Hoàn thiện **Workshop Documentation** theo FCJ Workshop Template, bổ sung ảnh minh họa và mô tả kỹ thuật theo từng phần <br> - Viết/chỉnh nội dung cho các mục: Workshop Overview, Prerequisite, Auth & Security, Database & Storage, Data Analytics, Monitoring & Tracing, Testing & Validation | 13/08/2026 | 13/08/2026 | FCJ Workshop Template |
| 6 | - Hoàn tất phần workshop documentation, tổng hợp lại tài liệu kỹ thuật như architecture, API, database schema và các luồng chính của hệ thống <br> - Cập nhật báo cáo thu hoạch cho 3 sự kiện đã tham dự, đối chiếu lại nội dung với phần workshop/report tổng hợp <br> - Rà soát toàn bộ kiến trúc hệ thống lần cuối, viết phần tổng kết đồ án, định hướng phát triển tương lai, chuẩn bị tài liệu bàn giao và slide thuyết trình | 14/08/2026 | 15/08/2026 | — |

## 3. Đóng góp cá nhân

* Phụ trách phần frontend liên quan đến Face Liveness, kiểm thử end-to-end và rà soát lại trải nghiệm người dùng trước khi bàn giao.
* Cập nhật và hoàn thiện đáng kể phần workshop documentation, đặc biệt ở các mục:
  * Workshop Overview
  * Prerequisite
  * Auth & Security
  * Database & Storage
  * Data Analytics
  * Monitoring & Tracing
  * Testing & Validation
* Theo sát phần observability ở mức tích hợp và kiểm tra kết quả trace/cảnh báo để đối chiếu với luồng hệ thống thực tế.
* Bổ sung phần tổng hợp kỹ thuật, báo cáo thu hoạch sự kiện và tài liệu bàn giao cuối kỳ.

## 4. Kết quả đạt được

* **Giám sát – Quan sát:**
  * Hoàn thiện lớp observability cơ bản với X-Ray, CloudWatch Alarm và SNS Alerting.
  * Xác định được bottleneck chính tại `Rekognition:SearchFacesByImage`.
  * Xử lý được xung đột segment trên Lambda bằng cách điều chỉnh cách gắn trace phù hợp với môi trường runtime.

* **Độ tin cậy kiến trúc:**
  * Bổ sung SQS làm lớp đệm cho pipeline sự kiện, giúp giảm rủi ro mất dữ liệu khi lưu lượng tăng đột biến.

* **Chống gian lận điểm danh:**
  * Hoàn thiện PoC Face Liveness ở mức khả thi, có thể tiếp tục mở rộng nếu phát triển hệ thống về sau.

* **Tài liệu và nghiệm thu:**
  * Hoàn tất workshop documentation, tài liệu kỹ thuật, báo cáo thu hoạch sự kiện, phần tổng kết đồ án và tài liệu bàn giao.
  * Hệ thống Smart Campus Platform đạt mức hoàn chỉnh để trình diễn và nghiệm thu cuối kỳ.