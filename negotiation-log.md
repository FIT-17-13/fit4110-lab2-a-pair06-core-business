# Biên bản đàm phán Hợp đồng API (Negotiation Log)

- **Pair:** 10 (Access Gate → Core Business)
- **Ngày đàm phán:** 2026-05-19
- **Provider:** Core Business (Nhóm A6)
- **Consumer:** Access Gate (Nhóm A3)

## Issue 1: Giới hạn thời gian phản hồi (Timeout)
- **Raised by:** Access Gate (Consumer)
- **Endpoint:** `POST /access/check`
- **Concern:** Access Gate yêu cầu phản hồi dưới 100ms để không gây ùn tắc tại cổng.
- **Proposal (Consumer):** Core Business phải cam kết SLA < 100ms.
- **Resolution:** Thống nhất SLA mục tiêu là 200ms. Nếu Core Business phản hồi chậm hơn 500ms, Access Gate sẽ tự động kích hoạt chế độ "fail-closed" (không mở cổng) và ghi log lỗi.
- **Rationale:** Việc kiểm tra nhiều lớp policy phức tạp ở Core Business không thể đảm bảo 100% dưới 100ms. 200ms là mức chấp nhận được.
- **Impact:** Access Gate cần cài đặt timeout là 500ms ở phía client.

## Issue 2: Xử lý quẹt thẻ liên tục (Idempotency)
- **Raised by:** Core Business (Provider)
- **Endpoint:** `POST /access/check`
- **Concern:** Sinh viên có thể quẹt 1 thẻ nhiều lần trong 1 giây, dẫn đến việc Core Business phải chạy rule engine liên tục và sinh ra nhiều cảnh báo rác.
- **Proposal (Provider):** Access Gate phải tạo và gửi kèm một `idempotencyKey` duy nhất cho mỗi lượt khách quẹt.
- **Resolution:** Access Gate đồng ý thêm trường `idempotencyKey` vào payload của AccessRequest.
- **Rationale:** Giúp Provider nhận diện và bỏ qua các request trùng lặp trong thời gian rất ngắn.
- **Impact:** Consumer phải sinh `idempotencyKey` (ví dụ: dùng hash của cardId + timestamp làm tròn đến giây).

## Issue 3: Kiểu dữ liệu của `operatorNote`
- **Raised by:** Core Business (Provider)
- **Endpoint:** `POST /access/check` (Response)
- **Concern:** Đôi khi bảo vệ (operator) có ghi chú lý do ghi đè quyền qua cổng, nhưng không phải lúc nào cũng có.
- **Proposal (Provider):** Sử dụng union type `[string, "null"]` cho trường `operatorNote` trong response AccessDecision.
- **Resolution:** Consumer đồng ý parse dữ liệu có thể là string hoặc null.
- **Rationale:** OpenAPI 3.1.0 hỗ trợ union type, giúp mô tả chính xác bản chất dữ liệu thay vì dùng chuỗi rỗng "".
- **Impact:** Consumer cần cập nhật model nhận dữ liệu để xử lý giá trị null mà không bị lỗi NullPointerException.

## Issue 4: Định dạng mã lỗi (Error Response)
- **Raised by:** Core Business (Provider)
- **Endpoint:** Toàn bộ API
- **Concern:** Cần chuẩn hóa cấu trúc lỗi trả về để Consumer dễ dàng xử lý.
- **Proposal (Provider):** Sử dụng chuẩn RFC 7807 (Problem Details).
- **Resolution:** Thống nhất dùng schema Problem Details cho tất cả mã lỗi 4xx và 5xx.
- **Rationale:** Dễ dàng mở rộng thuộc tính lỗi trong tương lai và tuân thủ chặt chẽ tiêu chí chấm điểm của Lab.
- **Impact:** Cả Consumer và Provider phải tuân thủ đúng schema này.

## Issue 5: Trạng thái thẻ bị khóa
- **Raised by:** Access Gate (Consumer)
- **Endpoint:** `POST /access/check`
- **Concern:** Khi thẻ bị khóa, Core Business sẽ trả mã lỗi 403 hay trả 200 với trạng thái allow=false?
- **Proposal (Consumer):** Đề xuất trả 200 OK với allow=false và reasonCode="CARD_BLOCKED" để tách biệt với lỗi hệ thống.
- **Resolution:** Thống nhất trả HTTP Status 200, trường allow=false, reasonCode="CARD_BLOCKED".
- **Rationale:** Quẹt thẻ bị khóa là một kịch bản nghiệp vụ bình thường (Business Logic), không phải là lỗi HTTP (Client/Server Error).
- **Impact:** Consumer xử lý mượt mà cảnh báo trên màn hình thay vì quăng Exception.

## Issue 6: Phân biệt các loại Policy (Polymorphism)
- **Raised by:** Core Business (Provider)
- **Endpoint:** `GET /policies/access/{policyId}`
- **Concern:** Hệ thống có nhiều loại policy (theo giờ, theo chức vụ...).
- **Proposal (Provider):** Sử dụng `oneOf` và `discriminator` để trả về loại policy phù hợp.
- **Resolution:** Thống nhất sử dụng trường `policyType` làm discriminator, map với `timeBased` và `roleBased`.
- **Rationale:** Tuân thủ tiêu chí đánh giá đa hình của OpenAPI 3.1, giúp Consumer biết chính xác mình đang nhận loại chính sách nào.
- **Impact:** Consumer phải viết logic parse JSON linh hoạt dựa trên trường `policyType`.

---
**Chữ ký xác nhận:**
- Provider (Core Business): Nguyễn Phương Nam (Đã ký)
- Consumer (Access Gate): Trưởng nhóm A3 (Đã ký)
- Witness (Giảng viên/TA): _________________
