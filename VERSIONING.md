# Chính sách quản lý phiên bản API (API Versioning Policy)
**Service:** Core Business Service (Nhóm A6)

Hệ thống áp dụng nguyên tắc **Semantic Versioning (SemVer)** phiên bản dạng `MAJOR.MINOR.PATCH` để quản lý sự thay đổi của hợp đồng API.

## 1. Định nghĩa các thay đổi hình thức
- **MAJOR (Phá vỡ tương thích - Breaking Change):** Tăng khi có các thay đổi làm hỏng code của Consumer đang chạy.
  - *Ví dụ:* Xóa một endpoint, đổi tên trường bắt buộc (`cardId` đổi thành `uid`), đổi kiểu dữ liệu trường trả về.
- **MINOR (Tương thích ngược - Backward-compatible):** Tăng khi bổ sung tính năng hoặc trường mới nhưng không làm ảnh hưởng consumer cũ.
  - *Ví dụ:* Thêm một endpoint mới, thêm trường tùy chọn (`idempotencyKey`) vào request body.
- **PATCH (Vá lỗi tài liệu - Patch/Bugfix):** Tăng khi sửa lỗi chính tả tài liệu hoặc mô tả schema mà không đổi cấu trúc.

## 2. Chiến lược xử lý deprecation (Ngừng hỗ trợ)
Khi một API chuẩn bị lỗi thời và sẽ bị xóa ở phiên bản MAJOR tiếp theo, nhóm sẽ thực hiện:
1. Đánh dấu trường `deprecated: true` tại API đó trong file `openapi.yaml`.
2. Gửi kèm 2 HTTP Header trong mọi response của API đó:
   - `Deprecation: true`
   - `Sunset: Tue, 01 Sep 2026 00:00:00 GMT` (Thời gian chính thức khai tử API).

## 3. Quản lý Endpoint Versioning
- Phiên bản hiện tại được định tuyến qua URL: `http://core-business:8000/v1/...`
- Khi lên `v2`, các endpoint cũ vẫn giữ nguyên để đảm bảo các hệ thống của nhóm khác không bị sập đột ngột.