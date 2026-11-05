# DCAD-STD-13 — Giao diện, icon và Preview

**Phiên bản:** 1.6
**Cập nhật:** 05/11/2026  
**Mức áp dụng:** Theo tính năng


## Giao diện

- Bố cục nhóm các trường theo nghiệp vụ; không nhồi nhiều chế độ không dùng. Label, tooltip, lỗi và thông báo phải theo quy chuẩn đa ngôn ngữ.
- Tách UI handler khỏi business logic, validation, đọc/ghi dữ liệu và vẽ CAD.
- Kiểm tra tỷ lệ DPI, chiều cao hàng/cột, vùng scroll, focus, tab order và khả năng sao chép text.
- ESC đóng/cancel form đúng cách, không làm mất dữ liệu đã lưu hoặc khiến lệnh CAD bên dưới hủy ngoài ý muốn.

## Icon, logo và tài nguyên

- Icon/logo nguồn nằm trong `icon` của project; nếu deploy ngoài DLL thì dùng thư mục logic `Dicon`, có khai báo trong DataManifest.
- Không dùng đường dẫn tuyệt đối tùy máy, không phụ thuộc file ảnh trong source tree khi chạy production.
- Màu sắc, kích thước icon và bộ nhận diện áp theo theme dự án đã phê duyệt; không ép đổi giao diện của module đang hoạt động ổn định chỉ để đồng bộ hình thức.

## Logo DCAD trên hộp thoại của module

- **BẮT BUỘC đối với hộp thoại WinForms do module DCAD tự tạo:** khi có logo DCAD đã phê duyệt, sử dụng icon đó tại thanh tiêu đề (Form.Icon/ShowIcon), thay icon mặc định thừa kế từ host CAD. Không thay icon, tài nguyên hay cửa sổ chính của AutoCAD/GstarCAD.
- Chỉ sử dụng bộ nhận diện chính thức được phê duyệt, không tự vẽ hoặc suy diễn logo. Nếu thiếu/sai tài nguyên đã duyệt, ghi NotVerified cho mục nhận diện, không nhận là đã thay thành công.
- Tệp nguồn icon đặt ở thư mục icon của project. Ưu tiên nhúng vào assembly với tên resource ổn định và nạp khi tạo form; không phụ thuộc đường dẫn source tree lúc chạy. Nếu đã duyệt icon ngoài DLL, dùng Dicon DIRECT cùng DataManifest theo STD-05/16. Không bắt module không dùng icon ngoài DLL kiểm Dicon.
- Chỉ hiển thị icon taskbar DCAD đối với cửa sổ riêng có taskbar button. Form con hoặc modal đang ShowInTaskbar=false không bị ép hiển thị trên taskbar.
- Chỉ sửa phần icon thuộc phạm vi yêu cầu; bảo toàn ESC/Cancel, DPI, ngôn ngữ, Preview, focus, hành vi của hộp thoại và phần mềm CAD chính.
- **QA:** kiểm nguồn/phê duyệt logo, resource nhúng trong DLL, hiển thị thanh tiêu đề thật trên từng host, taskbar nếu áp dụng và không tác động icon ứng dụng CAD. Báo riêng PASS/FAIL/NotVerified; khai báo resource không đồng nghĩa runtime PASS.

## Preview và DrawJig

- Preview dùng **cùng Model/Calculation** với đối tượng vẽ thật. Thông số, vị trí, layer, số bước lỗ và kích thước phải nhất quán.
- Khi có preview di chuyển theo chuột, dùng DrawJig: clone object tạm, biến đổi bằng `Matrix`, hiển thị qua `WorldDraw`, không sửa object thật.
- Chỉ commit geometry khi người dùng xác nhận; ESC/cancel phải hủy preview, dispose tài nguyên tạm, không để entity rác.
- Không dùng `SendStringToExecute MOVE` thay cho cơ chế preview realtime khi workflow cần bắt chuột mượt.
