# DCAD-STD-13 — Giao diện, icon và Preview

**Phiên bản:** 1.7
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
- **BẮT BUỘC nhúng logo DCAD vào DLL:** tệp logo chính thức đã phê duyệt (ví dụ `DcadLogo.ico`) đặt trong thư mục nguồn `icon/` của **từng project/host**, khai báo `EmbeddedResource` với tên resource nhất quán và nạp từ assembly khi tạo form. Khi chạy, **không** đọc logo từ `Dicon`, `DUser`, thư mục chương trình, thư mục source hoặc đường dẫn tuyệt đối; file ICO bên ngoài không phải điều kiện để mở hộp thoại. Việc sử dụng `Dicon` cho **tài nguyên khác logo DCAD** chỉ được phép theo DataManifest/STD-05/16 khi có yêu cầu đã duyệt.
- **BẮT BUỘC áp dụng logo cho cửa sổ DIALOG thực tế:** gán `Form.Icon` và `ShowIcon=true` đối với form có thanh tiêu đề. Nếu host/Windows có thể gán lại biểu tượng khi tạo handle hoặc hiện form, đăng ký xử lý ở `HandleCreated` và `Shown` để áp dụng lại; khi cần, gửi `WM_SETICON` cho **chính handle của form DCAD** với cả `ICON_SMALL` và `ICON_BIG`. Quản lý vòng đời của `Icon`/handle đúng cách, không rò rỉ GDI và không ghi đè biểu tượng cửa sổ chính của AutoCAD/GstarCAD.
- **Đổi logo trong các lần phát hành sau:** lấy tệp ICO mới **được người dùng phê duyệt**, thay đúng tệp nguồn ở `icon/` của các nhánh CAD liên quan, giữ tên resource/hợp đồng nạp trừ khi có chỉ định khác, build lại DLL của từng host, rồi kiểm tra rằng tài nguyên mới được nhúng và được hiển thị thật. Không được tự thiết kế logo thay thế hoặc đổi logo mà không được giao.
- Chỉ hiển thị icon taskbar DCAD đối với cửa sổ riêng có taskbar button. Form con hoặc modal đang `ShowInTaskbar=false` không bị ép hiển thị trên taskbar.
- Chỉ sửa phần icon thuộc phạm vi yêu cầu; bảo toàn ESC/Cancel, DPI, ngôn ngữ, Preview, focus, hành vi của hộp thoại và phần mềm CAD chính.
- **QA (STD-13/16/18):** kiểm hash/tệp logo đã duyệt; khai báo `EmbeddedResource` và resource thật bên trong DLL; mở hộp thoại SS/F/R hay form thuộc module trên từng CAD host và kiểm tra thanh tiêu đề, tái tạo handle, hiển thị lại và taskbar (nếu có); thử khi không có file ICO ngoài DLL; xác nhận icon chính của host không bị đổi. Báo riêng `PASS`/`FAIL`/`NotVerified` cho kiểm tra tĩnh, build và runtime. Việc có resource hoặc code `WM_SETICON` không đồng nghĩa runtime PASS.

## Preview và DrawJig

- Preview dùng **cùng Model/Calculation** với đối tượng vẽ thật. Thông số, vị trí, layer, số bước lỗ và kích thước phải nhất quán.
- Khi có preview di chuyển theo chuột, dùng DrawJig: clone object tạm, biến đổi bằng `Matrix`, hiển thị qua `WorldDraw`, không sửa object thật.
- Chỉ commit geometry khi người dùng xác nhận; ESC/cancel phải hủy preview, dispose tài nguyên tạm, không để entity rác.
- Không dùng `SendStringToExecute MOVE` thay cho cơ chế preview realtime khi workflow cần bắt chuột mượt.
