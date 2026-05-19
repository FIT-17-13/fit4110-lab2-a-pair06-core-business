# Event Contract sơ bộ — Pair 04 (Core Business → Notification)

## 1. Thông tin dependency
- **Producer:** Core Business (Nhóm A6)
- **Consumer:** Notification (Nhóm A7)
- **Cơ chế:** Queue async (RabbitMQ)

## 2. Mục đích nghiệp vụ
Khi Core Business phát hiện ra hành vi vi phạm chính sách hoặc nhận cảnh báo nguy hiểm từ AI Vision, nó sẽ sinh ra một sự kiện alert để dịch vụ Notification gửi tin nhắn cảnh báo đa kênh (Telegram, Email) tới ban quản lý.

## 3. Kênh & Tên Sự kiện
- **Event Name:** `alert.created`
- **Topic/Queue dự kiến:** `smartcampus.exchange.alerts` / `queue.notification.alerts`

## 4. Payload tối thiểu (JSON)
```json
{
  "eventId": "f7823f6c-829d-4c8d-8a8b-18a0980c67ba",
  "eventType": "alert.created",
  "occurredAt": "2026-05-19T13:45:00Z",
  "correlationId": "trace-998877",
  "source": "core-business",
  "data": {
    "alertId": "ALT-102",
    "severity": "high",
    "message": "Cảnh báo: Thẻ RFID bị khóa cố gắng quẹt tại Cổng Chính lúc 23h đêm.",
    "target": "security_team"
  }
}