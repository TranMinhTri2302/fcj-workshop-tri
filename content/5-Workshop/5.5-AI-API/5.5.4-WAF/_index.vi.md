---
title : "Cấu hình AWS WAF"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.5.4. </b> "
---

#### 5.5.4. Cấu hình bảo vệ API bằng AWS WAF và CloudFront




Để đảm bảo rằng API điểm danh `/api/attendance/recognize` chỉ có thể được truy cập từ mạng nội bộ của trường học (Campus Network), chúng ta sẽ sử dụng AWS WAF. Do API Gateway loại HTTP API không hỗ trợ đính kèm WAF trực tiếp, chúng ta sẽ tạo một Web ACL (Global), sau đó triển khai một CloudFront Distribution đứng trước API Gateway để gắn WAF bảo vệ.

**Bước 1: Tạo IP Set chứa địa chỉ IP của Campus**
1. Tìm kiếm và chọn dịch vụ **WAF & Shield** trên thanh công cụ.
> ![Search WAF](/aws-image/setupWAF/waf1.png)
2. Ở menu bên trái, chọn **IP sets**. Tại mục *Region scope*, chọn **CloudFront (Global)** và bấm **Create IP address set**.
> ![Manage IP Sets](/aws-image/setupWAF/waf2.png)
3. Điền thông tin IP Set:
   - **IP set name**: `SmartCampusIPSet`
   - **Scope**: `CloudFront`
   - **IP version**: `IPv4`
   - **IP addresses**: Nhập địa chỉ IP Public của Campus (ví dụ: `113.22.28.228/32`), sau đó bấm **Save**.
> ![Create IP Set](/aws-image/setupWAF/waf3.png)
> ![IP Set Success](/aws-image/setupWAF/waf4.png)

**Bước 2: Tạo Web ACL với Custom Rule chặn truy cập bên ngoài**
4. Chọn **Protection packs (web ACLs)** ở menu trái và bấm **Create protection pack (web ACL)**.
> ![Protection Packs](/aws-image/setupWAF/waf5.png)
5. Tại trang cấu hình:
   - **App categories**: Chọn `Other`
   - **App focus**: Chọn `API`
   - **Select resources to protect**: Bấm `Skip for now` (chúng ta sẽ gắn vào CloudFront sau).
> ![Web ACL Details](/aws-image/setupWAF/waf6.png)
6. Tại phần *Choose initial protections*, chọn **Build your own pack...** và bấm **Next** -> **Custom rule** -> **Next**.
> ![Choose Custom Rule](/aws-image/setupWAF/waf7.png)
> ![Add Custom Rule](/aws-image/setupWAF/waf8.png)
7. Cấu hình Custom rule chặn truy cập điểm danh từ bên ngoài:
   - **Action**: `Block`
   - **Rule name**: `BlockAttendanceOutsideCompany`
   - **If a request**: `matches all the statement (AND)`
   - **Statement 1**: Chọn Inspect `URI path`, Match type `Starts with string`, String to match `/api/attendance/recognize`.
> ![Rule Statement 1](/aws-image/setupWAF/waf9.png)
   - Bấm **Add another statement**.
   - **Statement 2**: Chọn Inspect `Originates from an IP address in`, IP address list `SmartCampusIPSet`. Đánh dấu tích vào **Negate statement results** và chọn **Source IP address**. Sau đó bấm **Add rule**.
> ![Rule Statement 2](/aws-image/setupWAF/waf10.png)
8. Đặt tên Web ACL là `SmartCampusAPIWebACL`. Mở rộng phần *Customize protection pack* để bật log:
   - **Logging destination type**: `Amazon CloudWatch Logs`
   - Bấm **Create new** để tạo log group mới.
> ![Web ACL Name](/aws-image/setupWAF/waf11.png)
9. Đặt tên Log group là `aws-waf-logs-smartcampus`, chọn Retention `Never expire`, Log class `Standard` và bấm **Create**.
> ![Create Log Group](/aws-image/setupWAF/waf12.png)
10. Quay lại trang tạo WAF, chọn Log group vừa tạo và bấm **Create protection pack (web ACL)**.
> ![Select Log Group](/aws-image/setupWAF/waf13.png)

**Bước 3: Tạo CloudFront Distribution bảo vệ HTTP API**
11. Tìm kiếm và truy cập dịch vụ **CloudFront**.
> ![Search CloudFront](/aws-image/setupWAF/waf14.png)
12. Chọn **Distributions** và bấm **Create distribution**.
> ![Create Distribution](/aws-image/setupWAF/waf15.png)
13. Khai báo thông tin:
   - **Distribution name**: `smart-campus-api-cf`
   - **Distribution type**: Chọn `Single website or app`
   - Bấm **Next**.
> ![Distribution Options](/aws-image/setupWAF/waf16.png)
14. Cấu hình Origin và Settings:
   - **Origin type**: Chọn `API Gateway`.
   - **Origin**: Chọn Invoke URL của API Gateway.
   - **Origin settings**: Chọn `Use recommended origin settings`.
> ![Origin Config](/aws-image/setupWAF/waf17.png)
   - **Cache settings**: Chọn `Use recommended cache settings tailored to serving API Gateway content`.
> ![Cache Config](/aws-image/setupWAF/waf18.png)
   - **Web Application Firewall (WAF)**: Chọn `Enable security protections`, chọn `Use existing WAF configuration` và chọn Web ACL `SmartCampusAPIWebACL` vừa tạo.
> ![WAF Config](/aws-image/setupWAF/waf19.png)
   - *(Tùy chọn)* **Connectivity**: Tại phần IPv6, chọn `Off`.
> ![Connectivity Config](/aws-image/setupWAF/waf22.png)
15. Kiểm tra lại thông tin ở trang **Review and create** và bấm **Create distribution**.
> ![Review and Create](/aws-image/setupWAF/waf20.png)
16. Chờ quá trình *Deploying* hoàn tất. Bạn có thể sử dụng **Distribution domain name** làm Endpoint mới để gọi API thay cho Invoke URL trực tiếp!
> ![Distribution Success](/aws-image/setupWAF/waf21.png)

> **[GIẢI THÍCH KIẾN TRÚC]**
> Do HTTP API của API Gateway không hỗ trợ tích hợp trực tiếp với AWS WAF, việc đưa CloudFront làm lớp đệm trung gian vừa giúp tăng tốc độ qua Edge Network, vừa là điểm gắn WAF để chặn đứng truy cập trái phép.
