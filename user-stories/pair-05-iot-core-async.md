# Event Contract sơ bộ — Pair 05 (IoT Ingestion → Core Business)

## 1. Thông tin dependency
- **Producer:** IoT Ingestion (Nhóm A1)
- **Consumer:** Core Business (Nhóm A6)
- **Cơ chế:** Queue async (MQTT / RabbitMQ)

## 2. Mục đích nghiệp vụ
IoT Ingestion tiếp nhận tín hiệu liên tục từ các cảm biến phần cứng (nhiệt độ, phòng máy, cảm biến khói) và đẩy sự kiện lên Queue để Core Business tiêu thụ và chạy Rule Engine nhằm phát hiện cháy nổ hoặc quá nhiệt.

## 3. Kênh & Tên Sự kiện
- **Event Name:** `sensor.reading.created`
- **Topic/Queue dự kiến:** `smartcampus.exchange.telemetry` / `queue.core.sensor_readings`

## 4. Payload tối thiểu (JSON)
```json
{
  "eventId": "a1122334-b556-c778-d990-112233445566",
  "eventType": "sensor.reading.created",
  "occurredAt": "2026-05-19T13:46:00Z",
  "correlationId": "iot-trace-abc123",
  "source": "iot-ingestion",
  "data": {
    "deviceId": "esp32-room302",
    "sensorType": "temperature",
    "value": 41.5,
    "unit": "CELSIUS",
    "locationId": "ZONE-SERVER-ROOM"
  }
}