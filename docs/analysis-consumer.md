# Phân tích yêu cầu — vai Consumer (Core Business Service)
**Các cặp phụ thuộc REST sync đóng vai Consumer:** Pair 02 và Pair 03

---

## 1. Resource Consumer cần nhận/gửi từ Provider

| Resource | Consumer dùng để làm gì? | Field bắt buộc với Consumer | Field có thể tùy chọn |
|---|---|---|---|
| `FaceMatchResult` (Từ AI Vision - Pair 02) | Lấy kết quả đối sánh khuôn mặt để kiểm tra xem có phải người lạ xâm nhập hay không. | `detectionId`, `confidence`, `modelVersion`, `timestamp`, `traceId` | `riskLevel`, `faceEmbeddingRef` |
| `AccessLog` (Từ Access Gate - Pair 03) | Truy xuất lịch sử quẹt thẻ tại các cổng để phục vụ việc kiểm tra và audit hệ thống. | `logId`, `cardId`, `gateId`, `direction`, `timestamp`, `status` | `operatorNote` |

---

## 2. API Consumer cần gọi

| Method | Path | Lúc nào gọi? | Kỳ vọng response |
|---|---|---|---|
| POST | `/vision/face-match` | Khi Core Business cần yêu cầu AI Vision đối sánh một ảnh khuôn mặt cụ thể. | `200 OK` kèm thông tin khớp lệnh và độ tin cậy (`confidence`). |
| GET | `/access/logs/recent` | Định kỳ hoặc khi có sự cố, Core cần lấy danh sách các lượt quẹt thẻ gần đây để đối chiếu. | `200 OK` kèm mảng danh sách log quẹt thẻ. |

---

## 3. Error case Consumer cần xử lý

| Status | Consumer hiểu là gì? | Consumer sẽ xử lý thế nào? |
|---:|---|---|
| 400 | Request gửi sang Provider bị sai cấu trúc JSON hoặc thiếu trường. | Log lỗi hệ thống, không thực hiện hành động tiếp theo để tránh sai nghiệp vụ. |
| 401 | Thẻ xác thực (Bearer token) của Core Business gửi sang bị hết hạn hoặc sai. | Hệ thống tự động kích hoạt luồng làm mới token (Refresh token) hoặc báo lỗi cấu hình môi trường. |
| 404 | Không tìm thấy mã `detectionId` hoặc `logId` bên phía Provider. | Hiển thị thông báo tài nguyên không tồn tại trên hệ thống giám sát. |
| 422 | AI Vision chạy mô hình lỗi hoặc độ tin cậy quá thấp không thể đưa ra kết luận. | Đánh dấu sự kiện là "Cần kiểm tra thủ công" và đẩy cảnh báo nghi vấn sang hệ thống Notification. |
| 500 | Server của AI Vision hoặc Access Gate bị sập / lỗi database. | Áp dụng cơ chế Retry (thử lại sau 3 lần), nếu vẫn lỗi thì kích hoạt chế độ cảnh báo khẩn cấp. |

---

## 4. Giả định bổ sung
- **Giả định 1:** Dữ liệu hình ảnh đối sánh khuôn mặt gửi sang AI Vision đã được chuẩn hóa kích thước, không vượt quá 5MB để tránh gây nghẽn timeout.
- **Giả định 2:** Phía Access Gate lưu trữ log quẹt thẻ cục bộ tối thiểu 30 ngày để đảm bảo Core Business luôn có thể truy xuất dữ liệu audit khi cần.

---

## 5. Câu hỏi dành cho Provider (Đưa ra bàn đàm phán)
1. *Hỏi AI Vision (Pair 02):* Khi mô hình AI hoạt động nhưng độ tin cậy thấp (`confidence < 0.5`), phía bạn sẽ trả về HTTP `200 OK` kèm trạng thái thấp hay trả về lỗi `422`?
2. *Hỏi Access Gate (Pair 03):* Endpoint lấy log gần đây `/access/logs/recent` có hỗ trợ lọc theo khoảng thời gian (`startTime`, `endTime`) hay không?

---

## 6. Rủi ro tích hợp

| Rủi ro | Tác động | Đề xuất xử lý |
|---|---|---|
| AI Vision phản hồi quá chậm (Timeout) | Core Business bị nghẽn luồng xử lý nghiệp vụ, gây chậm trễ hệ thống. | Đặt timeout nghiêm ngặt ở phía client là 1500ms khi gọi AI Vision. |
| Format mã lỗi của các bên không đồng nhất | Core Business khó đọc và phân tích nguyên nhân lỗi để xử lý tự động. | Ép buộc các nhóm Provider phải tuân thủ đúng chuẩn `application/problem+json` (RFC 7807). |