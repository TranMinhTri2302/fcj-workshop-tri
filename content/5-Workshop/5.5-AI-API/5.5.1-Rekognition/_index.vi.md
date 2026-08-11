---
title : "Cấu hình Amazon Rekognition"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.5.1. </b> "
---

#### 5.5.1. Tạo Collection trên Amazon Rekognition
Amazon Rekognition không giữ ảnh gốc để nhận diện, mà nó trích xuất và lưu trữ siêu dữ liệu (metadata/vector) của các đặc điểm khuôn mặt vào một kho chứa được gọi là **Collection**.

Hiện tại, AWS Management Console **không hỗ trợ** giao diện (UI) để tạo Collection. Tuy nhiên, chúng ta không cần cài đặt phức tạp ở máy tính cá nhân. Thay vào đó, chúng ta sẽ sử dụng **AWS CloudShell** - một môi trường dòng lệnh có sẵn ngay trên trình duyệt của AWS.

**Bước 1: Mở AWS CloudShell**
1. Trên thanh tìm kiếm của AWS Console, gõ **Rekognition** và truy cập vào dịch vụ.
2. Nhìn lên góc trên cùng bên phải màn hình (cạnh icon chuông thông báo), bấm vào biểu tượng **CloudShell** `>_` để mở giao diện dòng lệnh.
> ![Mở CloudShell](/aws-image/setupRekognition/regco-1.png)
*(Quá trình khởi tạo CloudShell có thể mất vài chục giây)*.

**Bước 2: Tạo Rekognition Collection**
3. Khi dấu nhắc lệnh `$` xuất hiện, hãy copy và dán dòng lệnh sau vào (chú ý đổi region nếu bạn dùng region khác):
   ```bash
   aws rekognition create-collection --collection-id smart-campus-faces --region ap-southeast-1
   ```
4. Nếu thành công, bạn sẽ nhận được kết quả JSON trả về với `StatusCode: 200`:
> ![Tạo Collection Thành công](/aws-image/setupRekognition/regco-2.png)

**Bước 3: Kiểm tra lại danh sách Collection**
5. Bạn có thể chạy lệnh sau để liệt kê các Collection đang có trong tài khoản nhằm đảm bảo nó đã được tạo đúng:
   ```bash
   aws rekognition list-collections --region ap-southeast-1
   ```
6. Kết quả sẽ hiển thị `smart-campus-faces` trong mảng `CollectionIds`:
> ![Kiểm tra Collection](/aws-image/setupRekognition/regco-3.png)

Vậy là xong! Kho chứa dữ liệu khuôn mặt ảo đã sẵn sàng. Ở phần sau, khi nhân viên đăng ký hệ thống qua API, Lambda sẽ nhận ảnh từ S3 và gọi hàm `IndexFaces` để đẩy vector khuôn mặt của họ vào Collection này.
