# DCAD-STD-08 — Command, selection và ESC

**Phiên bản:** 1.5  
**Mức áp dụng:** Bắt buộc theo thao tác

## Command, selection, typed input và hành vi ESC [CORE/điều kiện]

- Khi lệnh yêu cầu **chọn entity**, dùng selection/prompt entity thật; không giả lập chọn đối tượng bằng GetPoint. Không tự quét toàn database nếu nghiệp vụ chỉ cần miền chọn của người dùng.
- Nếu kết hợp chọn đối tượng và nhập thông số tại một lời nhắc: phân biệt nhánh chọn và nhánh typed input; không phá selection flow có sẵn; validate typed input trước tính toán; kết quả preview/điểm chèn phải thống nhất.
- Khi có WinForms, ESC phải đóng/cancel phù hợp ngay cả khi focus ở TextBox, ComboBox, Button, control động. ESC không được lọt xuống CAD làm hủy thao tác không chủ đích. Không dùng `Application.Exit()` để đóng một form. ESC tại dialog xác nhận không lưu dữ liệu chưa commit.
- Mọi thông báo lỗi người dùng cần qua Lang; phân biệt UserCancel với lỗi validation/runtime/AutoCAD. Không in stack trace vào UI; log kỹ thuật có context.
- Quy chuẩn chung yêu cầu **một command/project**; đề xuất command tiện ích kiểm tra `DDATAPATH` trong quy chuẩn dữ liệu **chưa tự động thành ngoại lệ đã duyệt**.
