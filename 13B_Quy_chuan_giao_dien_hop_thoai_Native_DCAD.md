# DCAD-STD-13B — Quy chuẩn kiến trúc và trình bày hộp thoại Native

**Mã chuẩn:** DCAD-NATIVE-UI-001  
**Phiên bản tài liệu:** 1.1  
**Ngày ban hành:** 24/11/2026  
**Cập nhật:** 24/11/2026 — bổ sung resize/redraw chống lưu ảnh.  
**Cơ sở tham chiếu:** Dtscr V118, `Dtscr-AutoCAD2022-V118/src/Dtscr.rc`, `src/PaletteWorkspace.h`, `src/ThemeStoreV118.h`.  
**Phạm vi:** Các hộp thoại, form cài đặt và workspace do DCAD tự xây dựng. Bắt buộc đối với **hộp thoại native mới hoặc phần giao diện native được giao thiết kế lại**. Không bắt buộc hồi tố những module đã nghiệm thu, không tự cấp quyền sửa mã ngoài yêu cầu.  
**Mức áp dụng:** (A) Bắt buộc; (B) Bắt buộc khi có tính năng; (C) Linh động theo nghiệp vụ.

> **Nguyên tắc:** Chuẩn hóa **cách tổ chức, chất lượng, thành phần và hành vi** của giao diện; **không** chuẩn hóa cứng một hình dạng hộp thoại, một kích thước cửa sổ hoặc một công nghệ vẽ duy nhất. Dtscr V118 là mẫu kiến trúc tham chiếu, không phải bộ tọa độ dùng cho mọi dự án.

## 1. Mục tiêu và giới hạn

1. Hộp thoại DCAD phải đồng nhất về phong cách, dễ đọc, rõ chức năng, hoạt động ổn định khi resize/DPI và không làm ảnh hưởng dữ liệu hoặc lệnh CAD.
2. Tái sử dụng kiến trúc native đã có của Dtscr: **dialog host → client/workspace → trạng thái → bố cục → vẽ/control → dữ liệu/nghiệp vụ**. Được rút gọn với hộp thoại đơn giản; không ép mọi ứng dụng có đủ lớp/file.
3. Đối với add-in C++ native, hộp thoại thao tác chính phải là giao diện Windows/CAD thật (Win32/MFC, control native hoặc tự vẽ native); **không** biến HTML/WebView thành runtime thay thế cho một hộp thoại đã được giao làm native.
4. HTML, ảnh mẫu và bản prototype có thể dùng để trao đổi thiết kế. Chúng **không chứng minh** giao diện CAD chạy đúng, không thay kiểm thử trên host và không tự động là mã triển khai.
5. Không thay hợp đồng command, geometry, XData/XRecord, dữ liệu, SQLite, hệ ngôn ngữ hoặc quy trình cấp phép chỉ vì thay đổi giao diện.

## 2. Phân cấp quy định

| Mức | Ý nghĩa | Ví dụ |
|---|---|---|
| **A — Bắt buộc** | Mọi hộp thoại thuộc phạm vi phải tuân thủ, trừ ngoại lệ được chấp thuận và ghi nhận | Logo, Unicode, tách nghiệp vụ, kiểm tra resize, xử lý ESC, QA |
| **B — Theo tính năng** | Chỉ bắt buộc khi có thành phần/tính năng tương ứng | Grid, preview hình học, palette, kéo thả, SQLite, thay theme |
| **C — Linh động** | Quyết định theo dữ liệu, người dùng và luồng thao tác | Kích thước, số cột, tỷ lệ vùng, modal/modeless, vị trí preview |

Nếu quy định ở tài liệu này khác một chuẩn chuyên biệt hiện hành, áp dụng chuẩn chuyên biệt cho phần nghiệp vụ tương ứng (đặc biệt STD-05/06/07/08/13/13A/16/18). Không được dùng mức C để bỏ các nghĩa vụ A/B.

## 3. Kiến trúc hộp thoại (A)

Hộp thoại native phải phân định rõ **trách nhiệm**, dù lớp/hàm có thể gộp hợp lý:

| Thành phần | Trách nhiệm | Không được làm |
|---|---|---|
| **Dialog Host** | Tạo/đóng cửa sổ, owner CAD, thanh tiêu đề, icon, kích thước, modal/modeless, focus, ESC | Chứa thuật toán CAD hoặc truy vấn dữ liệu trong thao tác paint |
| **UI State / View Model** | Giá trị hiển thị, lựa chọn, dirty, lỗi nhập liệu, trạng thái nhóm, scroll | Dùng tọa độ màn hình làm định danh dữ liệu |
| **Layout** | Tính một lần theo client size, DPI và quy tắc vùng; xuất hình học cho control/renderer | Rải tọa độ ngẫu nhiên ở nhiều event handler; cộng dồn sai lệch resize |
| **Renderer / Controls** | Vẽ panel, nhãn, viền, bảng và preview; điều khiển nhập/chọn thực | Tự ghi dữ liệu hoặc tự thay đổi geometry trong WM_PAINT |
| **Binding + Validation** | Liên kết control ID ↔ trường dữ liệu; kiểm tra kiểu, miền giá trị, hợp lệ | Suy đoán schema từ vị trí hoặc số thứ tự hàng |
| **Service / Business** | Đọc/ghi dữ liệu, tính toán nghiệp vụ, thao tác CAD; báo kết quả/lỗi rõ | Phụ thuộc trực tiếp vào pixel hoặc HWND của giao diện |

**Được phép rút gọn:** Một form 2–3 nút không cần tạo sáu class, một UI framework riêng hay DLL chung. Nhưng dữ liệu, bố cục và trách nhiệm vẫn phải phân định; tuân thủ STD-18 — đơn giản, sửa đúng phạm vi.

## 4. Khung cửa sổ và vòng đời (A)

1. Dùng owner/parent đúng của cửa sổ module trong CAD; không đổi biểu tượng hoặc thuộc tính của cửa sổ AutoCAD/GstarCAD chính.
2. Tiêu đề, nút đóng, icon DCAD chính thức **nhúng trong DLL/ARX theo STD-13**; không nạp logo từ đường dẫn ngoài khi chạy.
3. Mỗi lần mở phải có kích thước khởi đầu, vị trí hiển thị và giới hạn tối thiểu phù hợp màn hình đang sử dụng; không ép fullscreen hoặc cửa sổ lớn khi nội dung ít.
4. `WM_SIZE`/sự kiện layout phải cập nhật chính xác kích thước vùng con; resize và maximize/restore không chồng lấn, mất nút hoặc tạo khoảng trống bất hợp lý.
5. ESC, Cancel, X, Save, Apply và đóng form phải có ngữ nghĩa rõ; **không được mặc định lưu dữ liệu chưa xác nhận khi hủy**, trừ khi hợp đồng nghiệp vụ nêu rõ autosave. Không để editor đang mở ghi nhầm trường mới.
6. Modal/modeless/palette phải phù hợp workflow CAD. Nếu thao tác chọn đối tượng/vẽ cần command context, triển khai theo lifecycle CAD đúng chuẩn; không thao tác geometry tùy tiện khi dialog modal còn chiếm lệnh.
7. Giải phóng `HWND`, font, brush, bitmap, icon và subclass đúng vòng đời; không truyền ngoại lệ C++ vượt qua callback Win32 không có tầng chặn an toàn.
8. Không tạo nhiều phiên bản cửa sổ gây trùng cập nhật dữ liệu nếu module không hỗ trợ nhiều instance; thông báo rõ khi form đang mở.

## 5. Layout, DPI và khả năng co giãn (A)

1. **Một nguồn tính layout:** kích thước lấy từ **client area**, DPI thực và thông số thiết kế. Không khai báo lại tọa độ mâu thuẫn ở hàm tạo control, paint, hit-test và editor.
2. Tọa độ thiết kế được định nghĩa ở đơn vị logic **DIP/px tại 96 DPI**; chuyển sang pixel thực có quy tắc làm tròn thống nhất. Mọi phép đo, hit-test và vùng tương tác dùng cùng hệ tọa độ.
3. Chia vùng bằng quy tắc: kích thước cố định khi cần, kích thước tối thiểu, khoảng cách và vùng flex; không ép mọi cửa sổ chia trái/phải theo tỷ lệ Dtscr.
4. Nội dung phải đọc và thao tác được ở DPI **100%, 125%, 150%**; kiểm thêm màn hình nhỏ. Khi hệ thống hỗ trợ đổi DPI động, xử lý hoặc tạo lại font/layout theo DPI mới; trường hợp host không hỗ trợ phải ghi rõ giới hạn và kiểm thử.
5. Không được dùng co giãn để làm cột/nút nhỏ đến mức mất nhãn, mất dữ liệu hoặc không thể click; ưu tiên min-size, cuộn đúng cách hoặc thu gọn bố cục có quy tắc.
6. Bảng, preview và vùng phụ phải lấp khoảng trống có chủ đích; không để diện tích trống khổng lồ chỉ vì kích thước hộp thoại bị phóng lớn.
7. Cửa sổ kích thước khác hoặc bản dịch dài hơn không được khiến nhãn chồng lên giá trị. Đo chữ bằng font thực; có tooltip hoặc cuộn khi nội dung không thể hiển thị hết.

### 5.1. Resize và chống lưu ảnh (A — bắt buộc với cửa sổ co giãn)

1. **Tính → bố trí → vẽ:** tính toàn bộ vùng và vị trí control theo client area/DPI trước khi áp dụng; chỉ cập nhật vị trí/kích thước thực sự đổi. Không để paint xen giữa các bước bố trí từng control. Không nạp lại dữ liệu, thay lựa chọn, reset editor hoặc ghi DB do resize.
2. Với Win32 nhiều HWND con cùng parent, ưu tiên `BeginDeferWindowPos / DeferWindowPos / EndDeferWindowPos` để áp dụng theo lô; dùng cờ không kích hoạt/đổi Z-order và hạn chế redraw trung gian khi phù hợp. Kiểm tra kết quả API; nếu không áp dụng được theo lô, bố trí lại đầy đủ từ cùng layout rồi redraw, không để UI ở trạng thái nửa cũ nửa mới. Công nghệ native khác được dùng cơ chế tương đương.
3. Bảo đảm clipping giữa parent và child/sibling ở nơi phù hợp (`WS_CLIPCHILDREN`, `WS_CLIPSIBLINGS` với Win32); không vẽ nền lên control con. Không giữ pixel cũ khi kích thước/vị trí thay đổi; làm mới cả vùng vừa lộ và vùng control mới. Không dùng `MoveWindow(..., TRUE)` tuần tự cho toàn bộ control trong mỗi lượt resize nếu làm phát sinh redraw riêng lẻ.
4. Vùng tự vẽ phải phủ kín nền trong vùng invalid, dùng double buffering/dirty region khi cần; tôn trọng clipping, vòng đời DC/bitmap/brush, không giữ ảnh chụp cũ như nguồn dữ liệu. Control native dùng double buffering do control hỗ trợ (ví dụ ListView); không bật chế độ compositing toàn host CAD một cách mặc định.
5. Sau khi bố trí xong, yêu cầu redraw đồng bộ parent và child, gồm phần non-client nếu bị thay đổi (ví dụ `RedrawWindow` với `RDW_INVALIDATE | RDW_ERASE | RDW_ALLCHILDREN | RDW_FRAME` theo nhu cầu). Trong lúc kéo, ưu tiên gộp invalidation và để message loop xử lý; khi kết thúc kéo (`WM_EXITSIZEMOVE`), maximize/restore hoặc đổi DPI, bảo đảm có lượt vẽ hoàn chỉnh, không gọi redraw đệ quy.
6. Giữ focus, caret, giá trị chưa lưu, lựa chọn hàng, scroll và định danh dữ liệu khi thay đổi hình học; không ép cập nhật lại nội dung hay chiều rộng cột không cần thiết. Không dùng tắt/bật redraw, ẩn/hiện cả form hoặc timer để che lỗi; nếu cần tạm ngăn redraw trong phạm vi hẹp phải khôi phục chắc chắn kể cả khi lỗi.
7. **Điều kiện đạt:** kéo rộng/hẹp liên tục, đổi chiều kéo, maximize/restore nhiều lần, đổi DPI (nếu host hỗ trợ), kéo cột và thao tác khi editor đang có dữ liệu chưa lưu: không lưu ảnh cũ, nhân bản nút/chữ, nhòe viền, chồng control, mất focus hoặc nhấp nháy bất thường. Kiểm thử trực tiếp trong CAD, có ảnh/video hoặc log thao tác; kiểm thử layout mô phỏng không chứng minh chất lượng repaint.

## 6. Bộ thiết kế giao diện dùng chung (A)

Mỗi ứng dụng sử dụng các **semantic tokens** chuẩn thay vì tự phân tán RGB, kích thước và kiểu chữ:

- **Màu:** `WindowBg`, `PanelBg`, `PanelBorder`, `BodyText`, `MutedText`, `Accent`, `ButtonBg`, `ButtonText`, `InputBg`, `Focus`, `SelectedBg`, `Error` và các vai trò chuyên biệt khi cần.
- **Chữ:** font Unicode thống nhất (mặc định khuyến nghị **Segoe UI** trên Windows); mức chữ theo vai trò `Title`, `Section`, `Body`, `Hint`, `Table`.
- **Khoảng cách:** dùng thang spacing thay vì tự chọn vô hạn các số; khuyến nghị **4 / 8 / 12 / 16 px tại DPI 96**.
- **Điều khiển:** định nghĩa các mức chiều cao nút/ô nhập/hàng bảng và vùng click thống nhất. Mức khuyến nghị: **32/36 px** cho nhập liệu, **28/32/36 px** cho hàng bảng; được điều chỉnh có lý do theo mật độ dữ liệu.
- **Đường viền/trạng thái:** thống nhất normal, hover, focus, selected, disabled, error; tương phản đủ đọc; không chỉ dùng màu để báo lỗi.

**Không bắt buộc** mọi module phải có chính xác 54 role như Dtscr, 24 theme, cùng dữ liệu Theme.db hoặc cùng màu mặc định. Số role/theme và nơi lưu theo hợp đồng dữ liệu của từng module. Nếu dùng theme từ dữ liệu, một vai trò màu có ý nghĩa duy nhất trên toàn form; đổi theme không làm hỏng trạng thái nhập liệu.

## 7. Thành phần dùng chung (A nếu có loại thành phần tương ứng)

1. Button, Edit, ComboBox, Checkbox, Tab/Group, Scrollbar phải có cách hiển thị và tương tác nhất quán. Ưu tiên control native của Windows cho nhập liệu, Unicode, clipboard và bàn phím.
2. Cho phép GDI/GDI+, Direct2D/DirectWrite hoặc cơ chế native tương đương để vẽ phần cần custom; không ép tất cả module dùng một renderer duy nhất.
3. Đối với phần tự vẽ: sự kiện paint **chỉ vẽ từ state**, không truy vấn đĩa, ghi DB, dựng lại geometry thật; dùng double-buffer/dirty-region khi cần để không nhấp nháy.
4. Control ID, property key, record key phải ổn định; không dựa vào vị trí hoặc thứ tự control.
5. Thứ tự Tab, focus, Enter, ESC, rê chuột, cuộn và phím mũi tên phải đúng chức năng; không tạo lớp custom làm mất hành vi nhập liệu thông thường.
6. Nhãn, tooltip, placeholder, thông báo và lỗi sử dụng ngôn ngữ nghiệp vụ đúng STD-07/18; khóa máy, cột dữ liệu, tên file và định danh kỹ thuật không được dịch sai.

## 8. Các thành phần có điều kiện (B)

### 8.1. Bảng chỉnh sửa

Nếu có bảng chỉnh sửa được: áp dụng **DCAD-GRID-001 tại STD-13** và hợp đồng lưu tương ứng STD-05. Bắt buộc khóa hàng/cột ổn định, editor đúng ô, cột có minimum width, cuộn/kéo cột không đổi dữ liệu, Enter/ESC rõ nghĩa, nhập sai không ghi và kiểm thử đầy đủ.

### 8.2. Preview hình học

Nếu có preview CAD: dùng cùng model/calculation với kết quả tạo thật; hiển thị đúng tỷ lệ/hướng/layer có liên quan. Preview trên form không tự tạo entity CAD; khi có preview theo con trỏ, tuân thủ DrawJig/Undo theo STD-13/14.

### 8.3. Palette dockable

Nếu sử dụng `CAdUiPaletteSet`/`CAdUiPalette` hoặc palette tương đương: áp dụng quy tắc docking, focus, khởi tạo/hủy, state khi ẩn/hiện. **Không coi launcher palette là hộp thoại chính** nếu chúng là hai phần khác nhau.

### 8.4. Settings / SQLite / theme / lưu layout

Nếu có nhập và lưu settings: áp dụng STD-05/06, transaction/backup khi bắt buộc, xác nhận thay đổi và xử lý lỗi. Không tự thêm file cấu hình hoặc DB chỉ vì chuẩn giao diện. Nếu tích hợp DCAD UI Studio, **áp dụng thêm 13A / DCAD-UI-CONTRACT-001** đúng phạm vi và schema đã quy định; chuẩn 13B không thay thế contract đó.

## 9. Những thuộc tính được phép linh động (C)

- Chiều rộng, chiều cao, min-size và trạng thái maximize ban đầu.
- Số vùng: một, hai, ba hoặc nhiều vùng; tỷ lệ và hướng chia; splitter cố định hoặc kéo được.
- Vị trí tab, menu, bảng, vùng thông tin, preview, thanh nút, trạng thái.
- Số bảng, cột và hàng; chọn compact/dense/comfortable theo dữ liệu.
- Modal, modeless, palette; vị trí lưu cửa sổ; có nhớ kích thước/vị trí hay không.
- Backend native cụ thể, cơ chế layout, cách phân file/class — **miễn đạt các yêu cầu A/B**.

**Không được biến linh động thành tùy tiện:** mỗi lựa chọn C phải phục vụ nghiệp vụ, có hình mẫu/tiêu chí duyệt rõ và không gây hồi quy.

## 10. Mẫu tham chiếu Dtscr V118 — dùng đúng phạm vi

Các chi tiết khảo sát được từ nguồn `Dtscr-V118.zip`:

- `IDD_WORKSPACE` trong resource làm dialog host; `WorkspaceDialogProc` tạo cửa sổ client `DCAD_DTSCR_WORK` và gán icon DCAD cho dialog.
- `Ui` giữ state, control handles, font, theme và tình trạng editor; `Resize(Ui&)` tính vùng; `ClientProc` xử lý paint, chuột, bàn phím, scroll và sự kiện control.
- Dtscr phối hợp control Win32 thật với GDI tự vẽ panel/bảng/preview; font Segoe UI, scaling từ DPI.
- Các ngưỡng `1260×800`, `940×660`, chia vùng tham chiếu `69%`, số cột bảng `8`, và **54 vai trò màu** là **chi tiết riêng Dtscr**, không sao chép thành giá trị bắt buộc cho dự án khác.
- Khả năng đổi DPI khi kéo cửa sổ sang màn hình khác **chưa được chứng minh** từ rà soát handler V118; chuẩn mới yêu cầu kiểm thử phần này, không mặc nhiên ghi PASS.
- V118 là **tham chiếu cấu trúc mã nguồn**, không tự coi là bản UI đã đạt QA CAD runtime.

## 11. Quy trình thiết kế và triển khai

1. **Baseline:** đọc nguồn đã duyệt và các chuẩn liên quan; xác định form nào cần làm, phần nào phải giữ.
2. **UI contract:** mô tả nhóm chức năng, ID, state, hành vi Save/Cancel, các vùng và ràng buộc min-size/DPI.
3. **Duyệt mẫu:** có thể dùng bản phác thảo hoặc HTML để duyệt trực quan. Nếu duyệt HTML, phải ghi rõ đây là **mẫu thiết kế**, không phải runtime native và không bảo đảm pixel-perfect tự động.
4. **Native implementation:** dùng đúng kiến trúc của module hoặc thành phần chung đã kiểm chứng; không đổi nghiệp vụ/định dạng dữ liệu ngoài phạm vi.
5. **Layout verification:** đối chiếu nhóm, nhãn, kích thước, spacing, theme và hành vi; ảnh xem trên trình duyệt không thay ảnh kiểm thử CAD.
6. **QA và bàn giao:** tách PASS/FAIL/NotVerified cho kiểm tra tĩnh, build, APPLOAD/NETLOAD và runtime CAD trên từng target.

Khi chỉ thay đổi thông số giao diện đã được layout engine hỗ trợ thì không cần refactor nghiệp vụ. Khi đổi cấu trúc control cần thay mã và kiểm thử lại; **không hứa chỉ sửa HTML/JSON là tạo được mọi loại hộp thoại native không cần build**.

## 12. Ma trận QA bắt buộc

| Mã | Ca kiểm tra | Điều kiện đạt |
|---|---|---|
| **NUI-01** | Baseline, phạm vi và diff | Không sửa sai module hoặc logic ngoài yêu cầu |
| **NUI-02** | Mở/đóng form trên CAD host | Không crash, không thay cửa sổ CAD chính |
| **NUI-03** | Logo/title/resource | Icon DCAD đúng, nhúng, không phụ thuộc file ngoài |
| **NUI-04** | Kích thước khởi đầu / min / max | Nội dung vừa màn hình, không quá cỡ hoặc mất nút |
| **NUI-05** | Resize, maximize, restore | Không chồng/mất control; không vùng trắng vô lý; không lưu ảnh/nhân bản/nhòe viền |
| **NUI-06** | DPI 100/125/150%, đa màn hình | Chữ và control đọc/chọn được; nêu giới hạn DPI động nếu có |
| **NUI-07** | Theme và trạng thái tương tác | Normal/hover/focus/disabled/selected/error rõ |
| **NUI-08** | Unicode và đa ngôn ngữ | Chữ không lỗi font; label dài không bị cắt làm sai nghĩa |
| **NUI-09** | Tab/focus/Enter/ESC/Cancel/Save | Không lưu nhầm, không hủy lệnh CAD ngoài ý muốn |
| **NUI-10** | Grid, scroll, editor (nếu có) | Đúng DCAD-GRID-001; không đổi khóa dữ liệu |
| **NUI-11** | Preview hình học (nếu có) | Khớp model tính/vẽ thật; ESC không tạo entity rác |
| **NUI-12** | Tạo/hủy nhiều lần, lỗi Win32 | Không rò tài nguyên hoặc treo UI |
| **NUI-13** | Thiếu/sai file settings, DB (nếu có) | Có lỗi rõ ràng, không âm thầm ghi đè dữ liệu |
| **NUI-14** | Modal/modeless/palette (nếu có) | Đúng command context và vòng đời CAD |
| **NUI-15** | So sánh mẫu được duyệt | Các khác biệt có lý do, không tự đổi layout/style |
| **NUI-16** | Build/APPLOAD/runtime từng host | Báo trạng thái và bằng chứng riêng; không suy diễn PASS |
| **NUI-17** | Kéo giãn liên tục, đảo chiều, maximize/restore lặp lại, kéo cột, DPI khi có hỗ trợ; editor đang dirty | Không ghosting, nhấp nháy bất thường, mất focus/caret, đổi dữ liệu hoặc khóa hàng; có bằng chứng UI thực tế trong CAD. |

Với ca không áp dụng: ghi `N/A` và lý do. Với môi trường chưa thử: ghi **NotVerified**, không ghi PASS. Mọi thay đổi phải tuân thủ STD-18.

## 13. Áp dụng cho module đã có

- Module đã nghiệm thu **không phải sửa hàng loạt** chỉ vì chuẩn này ra đời.
- Chỉ áp dụng khi tạo form mới, được giao thiết kế lại form cũ, hoặc sửa chính phần UI thuộc phạm vi; giữ nguyên chức năng bên ngoài.
- Nếu cơ chế hiện tại ổn định và đạt A/B, không ép thay Win32 ListView sang GDI custom, hay ngược lại, chỉ để giống Dtscr.
- Nếu chưa thể đáp ứng một yêu cầu bắt buộc do giới hạn host/SDK, ghi **ngoại lệ có lý do, phạm vi, rủi ro, kiểm thử bù và phê duyệt**; không âm thầm bỏ qua.
- Dshared chỉ được dùng Dtscr làm **kiến trúc UI**; không sao chép dữ liệu `Theme.db`, kích thước workspace Dtscr, bảng plate hoặc các thuật toán riêng.

## 14. Quan hệ với quy ước hiện hành

- **STD-01/02/03/04:** quy trình, module, cấu trúc và build.
- **STD-05/06:** dữ liệu, SQLite và lưu settings.
- **STD-07/08:** Unicode, ngôn ngữ, command/ESC.
- **STD-13:** icon, preview, DrawJig, **DCAD-GRID-001**.
- **STD-13A:** **DCAD-UI-CONTRACT-001** khi tích hợp DCAD UI Studio; **không** bị thay thế.
- **STD-14/15/16:** Undo, an toàn lỗi, QA/PASS.
- **STD-17/18:** phiên bản, GitHub/đóng gói, quy tắc AI lập trình Karpathy.

**Hiệu lực:** Từ ngày được bổ sung vào bộ QuyUoc DCAD. Chuẩn này quy định giao diện, **không tự thay đổi mã nguồn hay phát hành lại bất kỳ add-in nào**.
