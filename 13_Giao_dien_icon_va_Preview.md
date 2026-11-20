# DCAD-STD-13 — Giao diện, icon và Preview

**Phiên bản:** 1.9
**Cập nhật:** 20/11/2026  
**Mức áp dụng:** Theo tính năng


## Giao diện

**Bổ sung cho module tích hợp DCAD UI Studio:** áp dụng [DCAD-UI-CONTRACT-001](13A_DCAD_UI_Contract_CSharp_CPP.md) về định danh Form/Control, schema 2, runtime adapter và kiểm thử bố cục. Không áp đặt cho module chưa tích hợp, không thay quy tắc UI/Preview hiện hành.

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


## Bảng hàng/cột có cấu trúc — tiêu chuẩn chung DCAD

**Hợp đồng UI:** `DCAD-GRID-001`. Áp dụng cho **mọi bảng dữ liệu chỉnh sửa được trong tất cả module DCAD**, bất kể nguồn là SQLite, DAT, CSV hay cấu hình trong bộ nhớ. Không dùng tên bảng/cột riêng của một project để đặt làm quy tắc cho project khác.

1. **Đơn vị hiển thị:** hàng = một bản ghi nghiệp vụ; cột = một thuộc tính có tên/kiểu/đơn vị xác định; ô = giao điểm của một hàng và một cột. Một ô sửa được chỉ sửa một thuộc tính. Không tạo “bảng giả” bằng một `EDIT` nhiều dòng chứa các dòng phân cách TAB, CSV hoặc dấu cách.
2. **Định danh:** mỗi hàng có khóa nghiệp vụ ổn định, độc lập với thứ tự hiển thị, chỉ số hàng, lọc/sắp xếp và scroll; mỗi cột có khóa thuộc tính ổn định, độc lập với vị trí kéo hoặc độ rộng. Trước ghi phải đối chiếu cả hai khóa. STT chỉ là thứ tự nghiệp vụ khi schema có quy định, không làm khóa ngầm.
3. **Hiển thị dễ kiểm soát:** tiêu đề cột rõ, đường kẻ hàng/cột hoặc phân cách đủ nhận biết, giá trị hiển thị đúng cột; loại dữ liệu và đơn vị không lẫn nhau. Màu, icon và phần trình bày phụ phải nằm trong ô được quy định, không được tự tạo cột logic mới nếu dữ liệu không có cột đó.
4. **Sửa từng ô:** click/nhấp đúp/F2/Enter theo thiết kế đã công bố; Enter hoặc mất focus phải xác nhận đúng ô đang sửa; ESC hủy ô chưa lưu. Chuyển ô, cuộn, kéo cột/hàng, đóng form hoặc thay đổi kích thước phải xử lý editor đang mở một cách xác định; sự kiện focus/Enter của editor cũ không được tác động editor mới.
5. **Kiểm tra đầu vào:** kiểm theo kiểu dữ liệu ngay tại ô; ngăn dán nhiều cột vào một ô khi tính năng dán bảng chưa được thiết kế. Trường khóa/định danh và quan hệ được khóa hoặc đổi qua quy trình có xác nhận. Thêm, xóa, đổi thứ tự hàng phải là thao tác riêng và có kiểm tra quan hệ.
6. **Kích thước/căn chỉnh:** có giới hạn chiều rộng tối thiểu phù hợp label và khả năng đọc; không được kéo cột về 0 làm biến mất dữ liệu. Thay đổi kích thước cột không thay đổi cột được liên kết trong model. Khi tổng cột vừa vùng bảng, cột cuối phải khép kín tới mép phải và không để vùng trắng “không thuộc cột nào”. Khi tổng độ rộng tối thiểu lớn hơn vùng hiển thị, **được cuộn ngang đúng cách**, không ép cột nhỏ đến mất nội dung.
7. **Hàng, cuộn, DPI:** chiều cao hàng đủ đọc; nếu cho kéo giãn phải giới hạn hợp lý, không làm mất nội dung; con lăn, thanh cuộn, bàn phím hoạt động tự nhiên và không làm đổi dữ liệu. Tính toán ở DPI thực, tránh vẽ chồng/lặp làm nhấp nháy khi rê chuột.
8. **Trạng thái:** mở/đóng/phóng to/thu nhỏ/phục hồi phải nhất quán; khi đóng phải phân biệt Save/Cancel và cảnh báo thay đổi chưa lưu. Tùy chọn nhớ kích thước/vị trí chỉ lưu khi dự án yêu cầu, không tự tạo cấu hình ngoài phạm vi.
9. **Giao dịch:** UI không tự serialize, ghi DB hoặc tự suy schema; tầng lưu nhận bản ghi có cấu trúc và khóa của bản ghi. Ghi dữ liệu SQLite theo `DCAD-SQLITE-GRID-001` tại STD-05; loại nguồn khác phải có hợp đồng ghi/rollback tương ứng.
10. **Kiểm chứng bắt buộc:** thử chỉnh ô đầu/giữa/cuối, cột đầu/cuối, thêm/xóa/move, dán TAB, giá trị sai, cuộn/kéo cột/hàng lúc editor mở, đổi thứ tự rồi sửa, nhiều cửa sổ đồng thời, DPI nhỏ/lớn, đóng/ESC và khôi phục trạng thái. Cần bằng chứng UI thực tế trên host, không chỉ kiểm thử hàm tính kích thước.

**Không áp đặt công nghệ giao diện:** Win32 ListView, DataGridView, WPF, Palette hoặc control tự vẽ đều có thể đạt nếu đáp ứng hành vi trên. Không tự thay control chỉ để đổi kiểu triển khai.
