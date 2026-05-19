# Phân tích yêu cầu — vai Consumer

- Cặp đàm phán: Pair-03 Core Business ↔ Access Gate
- Product: Smart Campus Operations Platform
- Consumer service: Core Business Service
- Provider service: Access Gate Service
- Người viết: Hoài Nam
- Ngày: 2026-05-19

---

## 1. Resource Consumer cần nhận/gửi

| Resource | Consumer dùng để làm gì? | Field bắt buộc với Consumer | Field có thể tùy chọn |
|---|---|---|---|
| AccessLog | Core Business dùng để kiểm tra quẹt thẻ ngoài giờ, quẹt nhiều lần, truy cập khu vực cấm | logId, cardId, gateId, timestamp, direction, result | userId, zoneId, reason |
| GateStatus | Core Business dùng để kiểm tra trạng thái hiện tại của cổng | gateId, status, lastUpdated | zoneId |

---

## 2. API Consumer cần gọi

| Method | Path | Lúc nào gọi? | Kỳ vọng response |
|---|---|---|---|
| GET | `/access-logs` | Khi Core cần lấy danh sách log quẹt thẻ để kiểm tra rule nghiệp vụ | Trả về danh sách AccessLog |
| GET | `/access-logs/{logId}` | Khi Core cần xem chi tiết một log cụ thể | Trả về một AccessLog |
| GET | `/gates/{gateId}/status` | Khi Core cần kiểm tra trạng thái hiện tại của cổng | Trả về GateStatus |

---

## 3. Error case Consumer cần xử lý

| Status | Consumer hiểu là gì? | Consumer sẽ xử lý thế nào? |
|---:|---|---|
| 400 | Query parameter sai định dạng, ví dụ from/to không đúng date-time | Log lỗi và không xử lý rule với request đó |
| 401 | Thiếu token hoặc token không hợp lệ | Refresh/cấu hình lại token |
| 403 | Core không có quyền gọi API Access Gate | Báo lỗi quyền truy cập cho hệ thống |
| 404 | Không tìm thấy access log hoặc gate | Ghi nhận resource không tồn tại, không tạo alert từ dữ liệu này |
| 409 | Dữ liệu log bị trùng hoặc xung đột trạng thái | Retry hoặc bỏ qua bản ghi trùng |
| 422 | Dữ liệu đúng JSON nhưng thiếu ý nghĩa nghiệp vụ | Hiển thị/lưu lý do cụ thể để kiểm tra lại dữ liệu |

---

## 4. Giả định bổ sung

- Access Gate lưu được lịch sử quẹt thẻ và cho phép Core lọc theo thời gian.
- Timestamp sử dụng định dạng ISO 8601 UTC.
- Mỗi access log có `logId` duy nhất để tránh xử lý trùng.

---

## 5. Câu hỏi cho Provider

1. Access Gate có hỗ trợ lọc log theo `from`, `to`, `gateId`, `cardId` không?
2. `result` của access log gồm những trạng thái chính xác nào?
3. Gate status có cần trả thêm thông tin lỗi chi tiết khi status là `error` không?

---

## 6. Rủi ro tích hợp

| Rủi ro | Tác động | Đề xuất xử lý |
|---|---|---|
| Provider đổi tên field như cardId/gateId | Consumer parse lỗi | Chốt naming trong `openapi.yaml` |
| Timestamp không thống nhất timezone | Core đánh giá sai rule ngoài giờ | Dùng ISO 8601 UTC |
| Log bị trùng | Core tạo alert trùng | Dùng `logId` làm idempotency key |
| Provider thiếu mã lỗi | Consumer khó xử lý lỗi | Chuẩn hóa ErrorResponse |