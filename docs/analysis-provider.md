# Phân tích của Provider (Core Business Service)
**Pair:** 10 (Access Gate → Core Business)

## 1. Năng lực hiện tại
- Core Business có khả năng đánh giá quyền truy cập (allow/deny) dựa trên policy đã được thiết lập.
- Có thể phản hồi nhanh (độ trễ kỳ vọng < 200ms) để tránh gây kẹt cổng Access Gate.

## 2. Dữ liệu yêu cầu từ Consumer
Để đánh giá chính xác, Core Business cần Access Gate gửi lên:
- `cardId` hoặc `personId` (Bắt buộc).
- `gateId` và `direction` (IN/OUT) (Bắt buộc).
- `timestamp` tại thời điểm quẹt thẻ.
- `correlationId` để dễ dàng truy vết log (Traceability).

## 3. Cấu trúc Response đề xuất
- **Trạng thái hợp lệ (200 OK):** Trả về kết quả `allow` (boolean), `reasonCode` (ví dụ: VALID_CARD, EXPIRED), `policyId` được áp dụng, và `operatorNote` (có thể null).
- **Trạng thái lỗi (4xx, 5xx):** Áp dụng chuẩn [RFC 7807] Problem Details for HTTP APIs để trả lỗi rõ ràng.

## 4. Những điều không thể đáp ứng (hoặc rủi ro)
- Nếu Access Gate gọi liên tục cho cùng một thẻ trong 1 giây (do lỗi quẹt nhiều lần), Core Business cần có cơ chế idempotency (nhận biết request trùng lặp) để tránh xử lý dư thừa. Đề xuất Access Gate gửi kèm `idempotencyKey`.