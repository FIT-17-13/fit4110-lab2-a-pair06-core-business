# Event Contract sơ bộ — Pair 08 (Core Business → Analytics)

## 1. Thông tin dependency
- **Producer:** Core Business (Nhóm A6)
- **Consumer:** Analytics (Nhóm A5)
- **Cơ chế:** Queue async

## 2. Mục đích nghiệp vụ
Mỗi khi Core Business đưa ra quyết định chấp thuận/từ chối ra vào hoặc kích hoạt luật chính sách, nó sẽ bắn log sự kiện sang Analytics để lưu trữ, tính toán KPI vận hành và hiển thị lên biểu đồ dashboard của trường học.

## 3. Kênh & Tên Sự kiện
- **Event Name:** `policy.decision.created`
- **Topic/Queue dự kiến:** `smartcampus.exchange.analytics` / `queue.analytics.decisions`

## 4. Payload tối thiểu (JSON)
```json
{
  "eventId": "eeeeddf4-3921-49fa-9481-120dddaac992",
  "eventType": "policy.decision.created",
  "occurredAt": "2026-05-19T13:47:00Z",
  "correlationId": "trace-998877",
  "source": "core-business",
  "data": {
    "decisionId": "DEC-8899",
    "policyId": "POL-STUDENT-01",
    "subjectId": "SV001",
    "result": "DENIED",
    "reasonCode": "OUT_OF_HOURS",
    "reasonText": "Sinh viên cố gắng truy cập phòng Lab ngoài giờ quy định (07:00-18:00)."
  }
}