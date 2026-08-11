---
title: "Worklog Tuần 1"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

# Tuần 1: Hội nhập môi trường – Nền tảng Cloud Computing – Thực hành dịch vụ AWS cốt lõi

## 1. Mục tiêu tuần

* Làm quen với môi trường làm việc tại văn phòng Bitexco, kết nối với các thành viên trong chương trình và bước đầu hình thành nhóm thực hiện dự án.
* Nắm các khái niệm nền tảng của Cloud Computing như Data Center, Region, Availability Zone, mô hình trách nhiệm chung và vai trò của hạ tầng cloud trong doanh nghiệp.
* Thiết lập tài khoản AWS cá nhân với các cấu hình bảo mật cơ bản, đồng thời thực hành các dịch vụ cốt lõi ban đầu gồm EC2, S3 và AWS CLI.
* Bắt đầu nghiên cứu kiến trúc mạng trên AWS (VPC) và đọc kỹ yêu cầu đồ án thực tập để chuẩn bị cho giai đoạn phân tích hệ thống.

## 2. Nhật ký công việc chi tiết

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Làm quen môi trường làm việc tại văn phòng Bitexco, kết nối với các thành viên trong chương trình <br> - Tìm hiểu nội quy, quy định và cách vận hành chung của đợt thực tập <br> - Bước đầu hình thành nhóm dự án, trao đổi về năng lực và định hướng kỹ thuật của từng thành viên | 22/06/2026 | 22/06/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Tìm hiểu tổng quan về chương trình AWS: mô hình học tập theo workshop/lab, chi phí sử dụng lab và credit, yêu cầu nền tảng lập trình và định hướng đánh giá <br> - Nghiên cứu các khái niệm cốt lõi của Cloud Computing: Data Center, Region, Availability Zone <br> - Tìm hiểu vai trò của hạ tầng cloud trong doanh nghiệp và lý do doanh nghiệp chọn cloud thay cho on-premise ở một số bài toán | 23/06/2026 | 23/06/2026 | <https://cloudjourney.awsstudygroup.com/> <br> <https://docs.aws.amazon.com/whitepapers/latest/aws-overview/> |
| 4 | - Tự tạo tài khoản AWS cá nhân (Free Tier), kích hoạt MFA, thiết lập nguyên tắc không sử dụng Root User cho thao tác hằng ngày <br> - Làm quen với AWS Management Console: điều hướng giữa các dịch vụ, kiểm tra region, theo dõi billing dashboard <br> - Cài đặt và cấu hình AWS CLI: thiết lập access key, kiểm tra kết nối, chạy các lệnh cơ bản | 24/06/2026 | 24/06/2026 | <https://docs.aws.amazon.com/cli/latest/userguide/> <br> <https://docs.aws.amazon.com/IAM/latest/UserGuide/> |
| 5 | - **Thực hành lab cơ bản:** <br>&emsp; + Tạo EC2 instance, cấu hình Security Group tối thiểu và kết nối SSH <br>&emsp; + Tạo S3 bucket, upload file, quản lý dữ liệu và kiểm tra quyền truy cập <br> - Ghi chú lại quy trình thao tác tài nguyên cloud trên Console và CLI để dùng cho các tuần sau | 25/06/2026 | 25/06/2026 | <https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/> <br> <https://docs.aws.amazon.com/AmazonS3/latest/userguide/> |
| 6 | - Nghiên cứu VPC và các thành phần mạng: Subnet, Route Table, Internet Gateway, Security Group <br> - Tìm hiểu mối liên hệ giữa public/private subnet trong thiết kế hệ thống <br> - Đọc kỹ yêu cầu đồ án AWS, ghi nhận các tiêu chí đánh giá và các hạng mục bắt buộc phải thể hiện trong báo cáo/workshop <br> - Thảo luận nội bộ nhóm về định hướng đề tài, so sánh sơ bộ giữa Smart Campus và Ticket System | 26/06/2026 | 26/06/2026 | <https://docs.aws.amazon.com/vpc/latest/userguide/> |

## 3. Đóng góp cá nhân

* Chủ động thiết lập tài khoản AWS cá nhân, cấu hình bảo mật và chuẩn bị môi trường thực hành riêng ngay từ đầu.
* Tự thực hiện các lab nền tảng với EC2, S3 và CLI để có trải nghiệm thực tế thay vì chỉ dừng ở mức đọc tài liệu.
* Tập hợp ghi chú kỹ thuật cơ bản về IAM, Region, Availability Zone, VPC và CLI để làm nền cho giai đoạn thiết kế sau đó.
* Trao đổi với các thành viên về năng lực kỹ thuật và định hướng công nghệ, hỗ trợ việc hình thành nhóm và chọn đề tài.
* Đọc sớm yêu cầu đồ án và ghi lại những đầu mục quan trọng liên quan đến báo cáo, workshop và phần triển khai kỹ thuật.

## 4. Kết quả đạt được

* **Nền tảng Cloud Computing và bảo mật:**
  * Nắm được các khái niệm cơ bản về Cloud Computing, kiến trúc hạ tầng AWS và mô hình trách nhiệm chung.
  * Hoàn tất thiết lập AWS Free Tier account, MFA và AWS CLI.

* **Thực hành dịch vụ AWS:**
  * Thực hiện thành công các lab nền tảng: tạo EC2 instance, kết nối SSH, tạo S3 bucket và quản lý dữ liệu.
  * Tạo được REST endpoint serverless đầu tiên, đồng thời làm quen với luồng thao tác tài nguyên cloud an toàn.

* **Chuẩn bị cho đồ án:**
  * Có nền tảng ban đầu về VPC và các thành phần mạng cốt lõi.
  * Hoàn tất bước chuẩn bị để chuyển sang giai đoạn phân tích bài toán và thiết kế hệ thống ở tuần tiếp theo.