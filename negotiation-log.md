# Biên bản đàm phán hợp đồng API

- Cặp đàm phán: Pair-03 Core Business ↔ Access Gate
- Product: Smart Campus Operations Platform
- Provider: Access Gate Service
- Consumer: Core Business Service
- Phiên: v1.0
- Ngày: 2026-05-19

---

## Issue #1

- Raised by: Consumer
- Endpoint: `GET /access-logs`
- Concern:
  Core Business cần lọc access log theo khoảng thời gian để phát hiện quẹt thẻ ngoài giờ hoặc nhiều lần trong thời gian ngắn.
- Proposal:
  Provider hỗ trợ query parameter:
  - `from`
  - `to`
- Resolution: Accepted
- Rationale:
  Giảm payload không cần thiết và cho phép Core chỉ lấy dữ liệu liên quan đến khoảng thời gian cần kiểm tra.
- Impact:
  API `/access-logs` phải hỗ trợ filter theo thời gian.

---

## Issue #2

- Raised by: Consumer
- Endpoint: `GET /access-logs`
- Concern:
  Core Business cần xác định bất thường theo cổng hoặc người dùng.
- Proposal:
  Provider hỗ trợ thêm query:
  - `gateId`
  - `cardId`
- Resolution: Accepted
- Rationale:
  Cho phép Core Business phân tích rule theo cổng hoặc một thẻ cụ thể.
- Impact:
  Provider phải hỗ trợ filter động.

---

## Issue #3

- Raised by: Provider
- Endpoint: `GET /access-logs`
- Concern:
  Timestamp không thống nhất có thể gây lỗi rule kiểm tra ngoài giờ.
- Proposal:
  Dùng chuẩn `ISO 8601 UTC`.

  Ví dụ:

  `2026-05-19T10:30:00Z`
- Resolution: Accepted
- Rationale:
  Tránh lỗi timezone giữa các service trong hệ thống distributed.
- Impact:
  Toàn bộ field `timestamp` trong contract dùng format `date-time`.

---

## Issue #4

- Raised by: Consumer
- Endpoint: `GET /access-logs`
- Concern:
  Core Business cần biết trạng thái quẹt thẻ để phân biệt truy cập hợp lệ và bất thường.
- Proposal:
  Chuẩn hóa field `result`.

  Giá trị hợp lệ:
  - `allowed`
  - `denied`
  - `error`
- Resolution: Accepted
- Rationale:
  Giúp Core Business dễ dàng áp dụng rule và tạo alert.
- Impact:
  `result` phải được định nghĩa bằng enum trong `openapi.yaml`.

---

## Issue #5

- Raised by: Consumer
- Endpoint: `GET /gates/{gateId}/status`
- Concern:
  Core Business cần kiểm tra trạng thái realtime của cổng để xác định hành vi bất thường hoặc lỗi thiết bị.
- Proposal:
  Provider cung cấp endpoint:

  `GET /gates/{gateId}/status`

  Response gồm:
  - `gateId`
  - `status`
  - `lastUpdated`
  - `zoneId` (optional)
- Resolution: Accepted
- Rationale:
  Core Business cần biết cổng đang `open`, `closed`, `locked` hay `error`.
- Impact:
  Provider cần expose thêm endpoint gate status.

---

## Issue #6

- Raised by: Provider
- Endpoint: All endpoints
- Concern:
  Consumer cần chuẩn hóa xử lý lỗi khi resource không tồn tại hoặc request không hợp lệ.
- Proposal:
  Thống nhất response lỗi chuẩn:

```json
{
  "code": "ACCESS_LOG_NOT_FOUND",
  "message": "Access log not found"
}
```

Status:
- `400` Bad Request
- `401` Unauthorized
- `403` Forbidden
- `404` Not Found
- `409` Conflict
- `422` Unprocessable Entity

- Resolution: Accepted
- Rationale:
  Giúp Consumer parse lỗi đồng nhất.
- Impact:
  Provider phải implement `ErrorResponse schema`.

---

# Chốt hợp đồng v1.0

Provider sign-off: Access Gate Team  

Consumer sign-off: Core Business Team  

Witness (GV/TA): ____________________    

Date: 2026-05-19

---

## Ghi chú warning nếu Spectral còn cảnh báo

| Warning | Lý do chấp nhận tạm thời | Kế hoạch sửa |
|---|---|---|
| operation-description | Chưa mô tả đầy đủ description cho từng endpoint | Bổ sung ở version v1.1 |
| examples-missing | Một số response chưa có example đầy đủ | Hoàn thiện sau khi mock xong |