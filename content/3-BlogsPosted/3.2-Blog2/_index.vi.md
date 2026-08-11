---
title: "Blog 2"
date: 2024-01-02
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# CHỐNG GIẢ MẠO NHẬN DIỆN KHUÔN MẶT VỚI AMAZON REKOGNITION FACE LIVENESS

---

Khi xây dựng các hệ thống nhận diện khuôn mặt, chúng ta thường gặp phải một lỗ hổng bảo mật kinh điển: làm sao để ngăn chặn người dùng lấy một bức ảnh chụp sẵn hoặc video quay sẵn trên điện thoại đưa ra trước camera để qua mặt hệ thống? Nếu chỉ dùng API **SearchFacesByImage** của Amazon Rekognition, AI sẽ chỉ tìm cách khớp khuôn mặt trong ảnh với database, chứ không biết đó là ảnh chụp người thật hay ảnh chụp lại màn hình điện thoại. **Amazon Rekognition Face Liveness** giải quyết triệt để bài toán này bằng cách xác minh người dùng đang đứng trước camera là người thật, trước khi tiến hành nhận diện danh tính.

---

## Vấn đề thực tế mình gặp phải

Bài viết này xuất phát từ kinh nghiệm thực tế khi mình xây dựng các hệ thống nhận diện khuôn mặt trên AWS. Trong quá trình triển khai, mình nhận ra rằng nếu chỉ sử dụng API **SearchFacesByImage** để so khớp khuôn mặt, hệ thống sẽ tồn tại một lỗ hổng bảo mật rất nghiêm trọng.

Cụ thể, API SearchFacesByImage chỉ làm một việc duy nhất: nhận ảnh đầu vào, tìm khuôn mặt trong ảnh và so khớp với các khuôn mặt đã được lưu trong **Rekognition Collection**. API này hoàn toàn không phân biệt được ảnh đó đến từ đâu — có thể là camera chụp trực tiếp người thật, hoặc cũng có thể là camera chụp lại một bức ảnh trên màn hình điện thoại, một tấm ảnh thẻ in ra giấy, hay thậm chí là một video quay sẵn đang phát trên iPad.

Điều này có nghĩa là bất kỳ ai có trong tay một bức ảnh rõ mặt của người khác đều có thể đưa ảnh đó ra trước camera và qua mặt hệ thống. Trong các bài toán như **eKYC** (xác minh danh tính điện tử), **chấm công bằng khuôn mặt**, hoặc **xác thực đăng nhập**, đây là rủi ro không thể chấp nhận được.

Một số hình thức gian lận phổ biến mà hệ thống chỉ dùng SearchFacesByImage không thể phát hiện:

- Dùng ảnh thẻ hoặc ảnh chân dung in ra giấy đưa trước camera.
- Mở ảnh khuôn mặt trên điện thoại hoặc máy tính bảng rồi đặt trước camera.
- Phát video quay sẵn của người thật trên một thiết bị khác.
- Dùng mặt nạ 3D hoặc ảnh 3D để đánh lừa camera.

Tất cả những hình thức này đều có thể vượt qua bước so khớp khuôn mặt thông thường nếu hệ thống không có lớp kiểm tra **liveness detection** — tức là kiểm tra xem người đang tương tác có phải là người thật đang có mặt trực tiếp trước camera hay không.

---

## Giải pháp: Tích hợp Amazon Rekognition Face Liveness

Rất may, AWS đã cung cấp tính năng **Face Liveness** trong Amazon Rekognition, giúp giải quyết triệt để bài toán chống giả mạo khuôn mặt. Thay vì chỉ so khớp ảnh, Face Liveness sử dụng camera stream kết hợp với các **challenge trực quan** (hiệu ứng ánh sáng ngẫu nhiên) để phân tích phản hồi thực tế từ khuôn mặt người dùng trong thời gian thực.

Dưới đây là cách mình đã tích hợp Face Liveness vào hệ thống.

---

## Luồng xử lý chi tiết

### Bước 1 — Backend tạo Face Liveness Session

Khi người dùng bắt đầu quét khuôn mặt, frontend gọi API của backend. Backend sau đó gọi API **CreateFaceLivenessSession** của Amazon Rekognition để tạo một phiên kiểm tra liveness mới. API này trả về một **SessionId** duy nhất, được gửi lại cho frontend để bắt đầu quá trình kiểm tra.

---

### Bước 2 — Frontend chạy Face Liveness Detector

Ở phía frontend, mình sử dụng SDK **@aws-amplify/ui-react-liveness**. SDK này cung cấp sẵn component UI để xử lý toàn bộ quá trình kiểm tra liveness.

Khi component được khởi chạy với SessionId, người dùng sẽ thấy một **giao diện hình bầu dục** hiển thị trên màn hình và được yêu cầu đưa khuôn mặt vào đúng khung hình. Sau đó, màn hình sẽ **chớp các dải màu ngẫu nhiên** — đây chính là **Challenge** mà AWS sử dụng để phân tích phản xạ ánh sáng trên khuôn mặt người dùng.

Trong suốt quá trình này, camera sẽ **stream video trực tiếp về AWS Rekognition** để hệ thống phân tích và đánh giá xem đây có phải là người thật hay không.

---

### Bước 3 — Backend lấy kết quả kiểm tra Liveness

Sau khi quá trình kiểm tra trên frontend hoàn tất, frontend thông báo cho backend. Backend tiếp tục gọi API **GetFaceLivenessSessionResults**, truyền vào **SessionId** đã tạo ở bước 1, để lấy kết quả đánh giá từ AWS.

AWS Rekognition sẽ trả về kết quả bao gồm:

- **Confidence**: điểm tin cậy cho biết khả năng đây là người thật (từ 0% đến 100%).
- **ReferenceImage**: ảnh khuôn mặt tốt nhất được AWS chọn ra từ quá trình kiểm tra.

---

## Đánh giá kết quả bằng Confidence Score

Sau khi nhận kết quả từ Face Liveness, mình sử dụng điểm **Confidence** để quyết định luồng xử lý tiếp theo:

- **Nếu Confidence < 90%**: Khả năng cao là giả mạo — người dùng có thể đang dùng mặt nạ, ảnh 3D, màn hình iPad hoặc video quay sẵn. Hệ thống **từ chối ngay lập tức** và không cho phép tiếp tục.

- **Nếu Confidence >= 90%**: Xác nhận là người thật. Lúc này, AWS trả kèm **Reference Image** — hình ảnh khung hình tốt nhất của khuôn mặt người dùng. Mình dùng ảnh này tiếp tục gọi **SearchFacesByImage** để xác định xem đó là ai trong Rekognition Collection.

Nói cách khác, luồng xử lý đúng sẽ là:

**Face Liveness Check → Nếu là người thật → SearchFacesByImage → Xác định danh tính**

Không nên bỏ qua bước liveness hoặc làm ngược thứ tự nếu hệ thống có yêu cầu bảo mật cao.

---

## Các điểm chính cần nắm

- **SearchFacesByImage chỉ dùng để nhận diện khuôn mặt** (so khớp danh tính), không có khả năng phát hiện giả mạo.

- **Face Liveness giúp chống spoofing** — tức là chống các hình thức giả mạo bằng ảnh, video, màn hình điện thoại, mặt nạ hoặc các kỹ thuật replay khác.

- **CreateFaceLivenessSession** được gọi từ backend để tạo một session kiểm tra liveness duy nhất cho mỗi lần xác thực.

- **SessionId** được frontend sử dụng để khởi chạy quá trình kiểm tra liveness thông qua Amplify UI component.

- **GetFaceLivenessSessionResults** được backend gọi sau khi quá trình kiểm tra kết thúc để lấy kết quả đánh giá.

- **Confidence Score** là chỉ số quan trọng nhất để quyết định người dùng có vượt qua bước kiểm tra liveness hay không. Ngưỡng 90% là mức phù hợp cho các hệ thống yêu cầu bảo mật cao.

- **Reference Image** là ảnh tốt nhất được AWS chọn ra sau khi xác minh người dùng là người thật, dùng tiếp cho bước nhận diện danh tính.

- **Face Liveness nên được đặt trước bước SearchFacesByImage** trong luồng xử lý — cần xác minh người thật trước, rồi mới xác định người đó là ai.

- **Video stream chỉ dùng để phân tích liveness** trong lúc thực thi và tự động bị hủy sau khi hoàn thành, đảm bảo tuân thủ quyền riêng tư dữ liệu (Data Privacy).

---

## So sánh kiến trúc trước và sau khi tích hợp Face Liveness

| Tiêu chí | Chỉ dùng SearchFacesByImage | Kết hợp Face Liveness + SearchFacesByImage |
|---|---|---|
| Kiểm tra danh tính | Có | Có |
| Phát hiện người thật | Không | Có |
| Chống ảnh chụp sẵn | Không | Có |
| Chống video quay sẵn | Không | Có |
| Chống mặt nạ 3D | Không | Có |
| Trải nghiệm người dùng | Đơn giản nhưng kém an toàn | Vẫn mượt mà, bảo mật cao hơn |
| Phù hợp eKYC / chấm công | Chưa đủ an toàn | Phù hợp |
| Rủi ro bị qua mặt | Cao | Thấp hơn đáng kể |

Việc thêm Face Liveness giúp hệ thống chuyển từ mô hình "ảnh này giống ai?" sang mô hình an toàn hơn: "đây có phải người thật không, và nếu đúng thì người này là ai?".

---

## Ưu điểm của kiến trúc này

**Bảo mật tuyệt đối**: Hoàn toàn loại bỏ được các trò gian lận dùng ảnh thẻ, ảnh trên điện thoại hay video quay sẵn. Hệ thống buộc người dùng phải là người thật đang có mặt trực tiếp trước camera.

**Trải nghiệm mượt mà**: Khác với các hệ thống cũ bắt người dùng phải thực hiện nhiều hành động thủ công như "chớp mắt 3 lần", "quay đầu sang trái/phải", AWS Face Liveness chỉ yêu cầu người dùng giữ yên khuôn mặt trong vùng bầu dục. Toàn bộ quá trình challenge và phân tích được xử lý cực nhanh ở background bởi AWS.

**Không lưu trữ video**: Video stream chỉ được sử dụng để phân tích liveness trong lúc thực thi và tự động bị hủy sau khi hoàn thành, đảm bảo tuân thủ quyền riêng tư dữ liệu (Data Privacy).

---

## Lưu ý khi triển khai

**Thứ nhất**, không nên gọi trực tiếp các API nhạy cảm của Rekognition từ frontend. Backend nên là nơi tạo session, xác thực user, kiểm tra quyền và quyết định kết quả cuối cùng.

**Thứ hai**, cần cấu hình IAM theo nguyên tắc **least privilege**. Backend chỉ nên được cấp các quyền cần thiết như `rekognition:CreateFaceLivenessSession`, `rekognition:GetFaceLivenessSessionResults`, `rekognition:SearchFacesByImage`. Không nên cấp quyền quá rộng nếu không cần thiết.

**Thứ ba**, cần chọn ngưỡng **Confidence** phù hợp với bài toán. Ngưỡng quá thấp có thể làm tăng rủi ro giả mạo, trong khi ngưỡng quá cao có thể khiến người dùng thật bị từ chối trong điều kiện ánh sáng hoặc camera không tốt. Mức 90% là ngưỡng AWS khuyến nghị cho các hệ thống yêu cầu bảo mật cao.

**Thứ tư**, cần xử lý các trường hợp lỗi ở frontend như người dùng không cấp quyền camera, camera bị che, ánh sáng yếu, kết nối mạng kém hoặc người dùng thoát giữa chừng.

**Thứ năm**, nên thiết kế UX để hướng dẫn người dùng rõ ràng: giữ khuôn mặt trong khung hình, không đeo khẩu trang, không đứng nơi quá tối, không để nhiều người xuất hiện trong camera cùng lúc.

**Thứ sáu**, cần cẩn trọng với dữ liệu sinh trắc học. Nếu lưu ảnh khuôn mặt (Reference Image), cần có chính sách bảo mật, mã hóa, phân quyền truy cập và tuân thủ quy định về quyền riêng tư dữ liệu tại khu vực triển khai.

---

## Khi nào nên dùng Amazon Rekognition Face Liveness?

Face Liveness đặc biệt phù hợp với các hệ thống cần xác thực khuôn mặt nhưng không muốn bị qua mặt bằng ảnh, video hoặc các hình thức giả mạo khác. Một số use case thực tế bao gồm:

- **eKYC** cho ngân hàng, ví điện tử, fintech — xác minh danh tính khách hàng từ xa.
- **Chấm công bằng khuôn mặt** — đảm bảo nhân viên phải có mặt trực tiếp, không thể nhờ người khác chấm hộ bằng ảnh.
- **Xác thực đăng nhập bằng khuôn mặt** — thay thế hoặc bổ sung cho mật khẩu.
- **Kiểm soát ra vào** văn phòng, nhà máy, khu vực bảo mật.
- **Xác minh danh tính** trước khi thực hiện giao dịch quan trọng.
- **Hệ thống thi online** cần xác thực thí sinh.

Nếu hệ thống của bạn đang chỉ dùng SearchFacesByImage, nó mới giải quyết được bài toán "người này giống ai?". Còn nếu muốn trả lời câu hỏi "người này có thật sự đang đứng trước camera không?", hãy tích hợp thêm Amazon Rekognition Face Liveness.

---

**Link bài đăng Blog:**
[\[Link bài viết\]](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2240573343374292/)

**Nguồn tham khảo chính:**

[Recommendations for Usage of Face Liveness — Amazon Rekognition Developer Guide](https://docs.aws.amazon.com/rekognition/latest/dg/recommendations-liveness.html)

[Amazon Rekognition Face Liveness — Amplify UI React Documentation](https://ui.docs.amplify.aws/react/connected-components/liveness)

**Hình của Blog:**  
![Blog 2-1](images/3-Blog/Blog-2.png)
![Blog 2-2](images/3-Blog/Blog-2-1.png)