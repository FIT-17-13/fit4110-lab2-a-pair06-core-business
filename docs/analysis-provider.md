# Phân tích yêu cầu — vai Provider

- Cặp đàm phán: Pair-03 Core Business ↔ Access Gate
- Product: Smart Campus Operations Platform
- Provider service: Access Gate Service
- Consumer service: Core Business Service
- Người viết: Hoài Nam
- Ngày: 2026-05-19

---

## 1. Resource chính

| Resource | Mô tả | Thuộc tính bắt buộc | Thuộc tính tùy chọn |
|---|---|---|---|
| AccessLog | Bản ghi quẹt thẻ tại cổng ra/vào | logId, cardId, gateId, timestamp, direction, result | userId, zoneId, reason |
| GateStatus | Trạng thái hiện tại của một cổng | gateId, status, lastUpdated | zoneId |

---

## 2. Action/API dự kiến

| Method | Path | Mục đích | Consumer gọi khi nào? |
|---|---|---|---|
| GET | `/access-logs` | Cung cấp danh sách log quẹt thẻ | Khi Core cần kiểm tra rule theo khoảng thời gian |
| GET | `/access-logs/{logId}` | Cung cấp chi tiết một log | Khi Core cần tra cứu một log cụ thể |
| GET | `/gates/{gateId}/status` | Cung cấp trạng thái hiện tại của cổng | Khi Core cần kiểm tra cổng đang mở, đóng, khóa hay lỗi |

---

## 3. Error case

| Status | Tình huống | Response body dự kiến |
|---:|---|---|
| 400 | Query parameter sai định dạng | `ErrorResponse` |
| 401 | Thiếu Bearer token | `ErrorResponse` |
| 403 | Token hợp lệ nhưng không có quyền | `ErrorResponse` |
| 404 | Không tìm thấy log hoặc gate | `ErrorResponse` |
| 409 | Dữ liệu log bị trùng hoặc xung đột trạng thái | `ErrorResponse` |
| 422 | Dữ liệu đúng JSON nhưng vi phạm nghiệp vụ | `ErrorResponse` |

---

## 4. Giả định bổ sung

- Access Gate có khả năng lưu và truy xuất lịch sử access log.
- Mỗi access log có `logId` duy nhất.
- Gate status gồm các giá trị: `open`, `closed`, `locked`, `error`.

---

## 5. Câu hỏi cho Consumer

1. Core Business cần lấy log trong khoảng thời gian tối đa bao lâu?
2. Core có cần phân trang khi danh sách access logs lớn không?
3. Core có cần thêm field `zoneId` để kiểm tra khu vực cấm không?

---

## 6. Rủi ro tích hợp

| Rủi ro | Tác động | Đề xuất xử lý |
|---|---|---|
| Tên field không thống nhất | Consumer parse lỗi | Chốt naming trong `openapi.yaml` |
| Payload log quá lớn | Timeout/mock lỗi | Bổ sung pagination hoặc filter thời gian |
| Thiếu timezone trong timestamp | Core xử lý sai rule ngoài giờ | Dùng ISO 8601 UTC |
| Consumer cần field chưa có | Phải sửa contract nhiều lần | Đàm phán field bắt buộc/tùy chọn trước khi viết OpenAPI |