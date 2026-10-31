# DCAD-STD-06 — Lưu settings và Reset Factory

**Phiên bản:** 1.10  
**Mức áp dụng:** Theo tính năng


## Lưu cấu hình DUser an toàn

1. **Chức năng đọc/ghi settings trong DUser** chỉ yêu cầu thư mục `DUser` DIRECT và các tệp bắt buộc được khai báo trong DataManifest của module; không cần kiểm đủ năm thư mục logic hoặc root `DCAD`. Chỉ sử dụng **DUser DIRECT duy nhất** đã phân giải ở đầu lượt lệnh. Cấu hình DUser lỗi/trùng/không khả dụng phải cảnh báo yêu cầu thiết lập lại và không ghi settings sang `DData`, `DProgram` hoặc đích khác. Kiểm tra đủ nhóm thư mục liền kề cuối Support Path chỉ thuộc Setup/QA hệ thống.
2. Đọc settings, ghi settings và log **runtime cùng module** dùng chung một kết quả phân giải DUser của lượt lệnh; không tự viết nhiều resolver. Nếu DUser không cho phép ghi log, hiển thị lỗi trên giao diện/Command Line, không chuyển log sang `%APPDATA%`, TEMP hay thư mục DLL. Không đặt bước kiểm tra lại Support Path tại mỗi lần nhấn Lưu trong form.
3. Validate schema, encoding, số, phạm vi và khóa trước khi ghi.
4. Ghi file tạm **cùng thư mục và cùng ổ đĩa**; flush, đóng file và kiểm tra lại khi dữ liệu quan trọng.
5. Thay thế bằng phương thức an toàn có **backup**; nếu lỗi phải giữ bản cũ. Không xóa đích trước rồi chép file mới.
6. Chỉ cập nhật trạng thái “đã lưu” của giao diện/bộ nhớ sau khi ghi thành công. Khi nhiều CAD cùng dùng file, cần khóa hoặc kiểm tra version/hash tránh ghi đè thay đổi.

## Last Used

- Lưu Last Used của **đúng module và đúng variant** sau thao tác thành công; không lưu khi Cancel/Fail.
- Khi nạp lại, validate dữ liệu trước khi dùng. Thứ tự ưu tiên phù hợp nghiệp vụ: dữ liệu entity/XData/geometry được chọn → Last Used cùng variant → default đã công bố → lỗi có giải thích.
- Không chia sẻ Last Used giữa các loại chi tiết có thông số cùng tên nhưng ý nghĩa khác nhau.

## Phân biệt thiếu, hỏng và khôi phục

- **Thiếu settings riêng của module:** chỉ tạo tại DUser DIRECT hợp lệ nếu DataManifest cho phép khởi tạo lần đầu; không sử dụng default để che lỗi cấu hình hệ thống.
- **Thiếu dữ liệu hệ thống bắt buộc:** cảnh báo yêu cầu thiết lập lại và dừng chức năng DCAD phụ thuộc môi trường; không trích xuất/khôi phục tự động để che lỗi.
- **File hỏng:** giữ backup, báo tên file/dòng/khóa/lý do; không âm thầm ghi đè bằng default.
- `MIS_SET.SET` là tệp chọn ngôn ngữ bắt buộc trong DUser của hệ thống DCAD; plugin không quản trị ngôn ngữ chỉ đọc dòng 28. Thiếu/sai: cảnh báo yêu cầu thiết lập lại, không tự tạo/sửa và không vận hành bằng lựa chọn ngôn ngữ giả định.

## Reset Factory

- Là **lệnh/hành động riêng có xác nhận**, không được thực hiện ngầm khi tìm hoặc đọc file.
- Sử dụng thư mục đích được phân giải theo DCAD-DATA-DIR-001 cho lượt thao tác Reset Factory; không tự tạo `DUser`/`DData` dưới một Support Path bất kỳ và không đổi nơi restore khi đích đã chọn bị lỗi.
- Chỉ dùng đúng bộ factory canonical của **dự án hiện tại** được khai báo trong DataManifest, có version/schema và kiểm tra tính toàn vẹn.
- Sao lưu file tùy chỉnh trước restore; ghi thay thế an toàn; thông báo những tệp sẽ bị thay đổi.
- Không tự tạo factory giả, không lấy bộ mặc định của dự án khác hoặc dùng Factory để chữa lỗi build.
