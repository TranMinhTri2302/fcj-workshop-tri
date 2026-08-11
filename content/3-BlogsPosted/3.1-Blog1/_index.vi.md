---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# PAGINATION STRATEGY TRONG AMAZON DYNAMODB - CÁCH TIẾT KIỆM CHI PHÍ VÀ TỐI ƯU PERFORMANCE

---

Amazon DynamoDB cung cấp cơ chế pagination thông qua **LastEvaluatedKey** và **ExclusiveStartKey**, cho phép bạn chia nhỏ kết quả truy vấn thành từng phần (page) thay vì load toàn bộ dữ liệu một lần. Đây không phải là tính năng tùy chọn mà là chiến lược thiết yếu giúp giảm chi phí Read Capacity Units (RCU) lên đến hàng nghìn lần, cải thiện thời gian phản hồi từ vài phút xuống còn vài trăm millisecond, và đảm bảo ứng dụng có thể scale được khi dữ liệu tăng trưởng.

---

## Vấn đề thực tế mình gặp phải

Bài viết này dựa trên kinh nghiệm làm việc thực tế của mình với DynamoDB qua nhiều dự án. Mình nhận ra rằng rất nhiều bạn khi mới bắt đầu với DynamoDB thường mắc một sai lầm khá phổ biến, đó là cố gắng load toàn bộ dữ liệu cùng một lúc. Điều này nghe có vẻ vô hại khi dữ liệu còn ít, nhưng khi bảng DynamoDB của bạn phình to lên hàng trăm nghìn, hàng triệu bản ghi, thì đó là lúc vấn đề bắt đầu xuất hiện rõ ràng.

Cụ thể, khi bạn có một bảng DynamoDB chứa hàng triệu bản ghi và bạn thực hiện một query hoặc scan để lấy hết dữ liệu, không phải chỉ là nó sẽ chậm, mà còn tốn kém cả về mặt tiền bạc lẫn hiệu suất hệ thống. DynamoDB tính phí dựa trên **Read Capacity Units (RCU)** — mỗi lần bạn đọc một lượng dữ liệu nhất định, bạn sẽ bị tính phí tương ứng. Nếu bạn cứ load toàn bộ từ đầu mỗi request, chi phí sẽ tăng theo cấp số nhân, đặc biệt khi ứng dụng có nhiều người dùng đồng thời.

Ngoài vấn đề chi phí, việc load toàn bộ dữ liệu còn gây ra **throttling** khi vượt quá provisioned capacity, làm tăng latency nghiêm trọng và trong nhiều trường hợp dẫn đến **timeout** ở phía client. Người dùng sẽ phải ngồi đợi cả chục giây, thậm chí vài phút, chỉ để thấy một danh sách sản phẩm hoặc lịch sử đơn hàng.

---

## Giải pháp: Pagination với LastEvaluatedKey

Giải pháp thực tế là sử dụng **pagination** — thay vì lấy tất cả, bạn chỉ lấy một phần nhỏ trước (ví dụ 10 hoặc 20 bản ghi), sau đó khi người dùng muốn xem thêm, bạn mới tiếp tục lấy batch tiếp theo.

Cách hoạt động của pagination trong DynamoDB khá đơn giản nhưng cực kỳ hiệu quả. Khi bạn thực hiện một **Query** hoặc **Scan**, ngoài kết quả trả về (Items), DynamoDB cũng trả kèm một giá trị gọi là **LastEvaluatedKey**. Đây là khóa chỉ ra chính xác vị trí bạn dừng lại trong lần truy vấn trước. Lần query tiếp theo, bạn chỉ cần truyền khóa này vào tham số **ExclusiveStartKey**, DynamoDB sẽ biết phải bắt đầu từ vị trí nào và tiếp tục trả về batch tiếp theo mà không phải quét lại từ đầu.

---

## Các điểm chính cần nắm

- **LastEvaluatedKey** là khóa mà DynamoDB trả về sau mỗi lần query/scan, chỉ ra vị trí cuối cùng đã được đọc. Khi giá trị này là `null`, nghĩa là đã hết dữ liệu.

- **ExclusiveStartKey** là tham số bạn truyền vào request tiếp theo, chính là giá trị LastEvaluatedKey từ response trước đó, để DynamoDB biết bắt đầu từ đâu.

- **Tham số Limit** giới hạn số lượng items được **evaluate** (kiểm tra) chứ không phải số items được **trả về**. Nếu bạn có FilterExpression, số items thực tế trả về có thể ít hơn Limit → đây là "gotcha" phổ biến nhất mà nhiều người mới gặp phải.

- **Giới hạn 1MB mỗi response**: DynamoDB tự động phân trang khi kết quả vượt quá 1MB, kể cả khi bạn không set Limit. Điều này có nghĩa là dù bạn muốn lấy hết, DynamoDB cũng buộc bạn phải gọi nhiều lần.

- **Chi phí RCU giảm tỷ lệ thuận** với lượng dữ liệu đọc mỗi lần. Load 20 items × 1KB = 20 RCU, thay vì load 1 triệu items = 1 triệu RCU. Nếu người dùng chỉ xem 5 trang, tổng chi phí chỉ khoảng 100 RCU — hiệu quả gấp **10.000 lần**.

- **Thời gian phản hồi cải thiện đáng kể**: từ vài giây hoặc vài phút (load all) xuống còn vài trăm millisecond (load per page), cải thiện trải nghiệm người dùng rõ rệt.

- **Pagination hoạt động với cả Query và Scan**, nhưng nên ưu tiên dùng Query vì Scan sẽ quét toàn bộ bảng và tốn RCU hơn rất nhiều.

- **LastEvaluatedKey bao gồm đầy đủ primary key** (partition key + sort key nếu có), nên kích thước của nó phụ thuộc vào cấu trúc khóa của bảng, nhưng luôn nhỏ hơn rất nhiều so với toàn bộ dữ liệu.

---

## So sánh chi phí cụ thể

Để thấy rõ sự khác biệt, hãy xem ví dụ sau:

| Phương pháp | Số items đọc | RCU tiêu tốn | Thời gian phản hồi |
|---|---|---|---|
| Load toàn bộ 1 triệu items | 1.000.000 | ~1.000.000 RCU | Vài phút hoặc timeout |
| Pagination 20 items/page × 5 trang | 100 | ~100 RCU | ~200-500ms mỗi page |

Trong thực tế, đa số người dùng chỉ xem 2-3 trang đầu tiên. Điều đó có nghĩa là pagination không chỉ tiết kiệm chi phí về mặt lý thuyết mà còn phản ánh đúng hành vi sử dụng thực tế — bạn chỉ trả tiền cho những gì người dùng thực sự cần xem.

---

## Lưu ý khi triển khai

**Thứ nhất**, khi phía client nhận LastEvaluatedKey, bạn nên **encode nó** (ví dụ base64) trước khi trả về cho frontend, vì key này chứa thông tin về cấu trúc bảng mà bạn không muốn expose trực tiếp.

**Thứ hai**, DynamoDB pagination là kiểu **forward-only cursor-based pagination**, không hỗ trợ nhảy đến trang bất kỳ (ví dụ nhảy thẳng đến trang 50). Nếu ứng dụng của bạn yêu cầu kiểu pagination đó, bạn cần cân nhắc kết hợp với caching layer hoặc thiết kế lại data model.

**Thứ ba**, cần xử lý cẩn thận trường hợp dữ liệu bị thay đổi giữa các lần gọi (items được thêm mới hoặc bị xóa), vì điều này có thể gây ra trùng lặp hoặc bỏ sót dữ liệu ở ranh giới giữa các page.

**Thứ tư**, với FilterExpression, bạn có thể nhận về page rỗng (0 items) nhưng LastEvaluatedKey vẫn khác null. Lúc này bạn cần tiếp tục gọi query cho đến khi LastEvaluatedKey là null hoặc đã đủ số items cần thiết.

---

## Khi nào nên dùng pagination?

Tính năng này đặc biệt hữu ích khi bạn xây dựng các chức năng như danh sách sản phẩm, lịch sử giao dịch, feed hoạt động, hay bất kỳ giao diện nào hiển thị danh sách dữ liệu lớn. Thay vì ép người dùng chờ đợi và hệ thống gồng gánh toàn bộ dataset, pagination cho phép bạn phục vụ từng phần nhỏ một cách nhanh chóng, tiết kiệm và mượt mà.

Nếu bạn đang xây dựng bất kỳ ứng dụng nào với DynamoDB, hãy nghĩ đến pagination ngay từ giai đoạn thiết kế. Nó không phải là tính năng "nice-to-have" mà là một phần thiết yếu của kiến trúc ứng dụng — giúp bạn tiết kiệm chi phí vận hành, cải thiện hiệu suất hệ thống, và tạo ra trải nghiệm tốt hơn cho người dùng.

---
**Link bài đăng Blog:**
https://www.facebook.com/share/p/1BxRgPHRBn/

**Nguồn tham khảo chính:**
[Paginating table query results — Amazon DynamoDB Developer Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Query.Pagination.html)

**Hình của Blog:**  
![Blog 1](images/3-Blog/Blog-1.png)