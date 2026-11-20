# DCAD-UI-CONTRACT-001 — Hợp đồng bố cục giao diện dùng chung C# / C++

**Phiên bản:** 1.0 — 20/11/2026.
**Áp dụng:** Chỉ cho module chủ động tích hợp DCAD UI Studio; không áp dụng hồi tố hàng loạt.
**Liên quan:** STD-03, 05, 06, 07, 13, 16, 18.

## 1. Mục tiêu
Studio độc lập đọc mô tả UI, cho kéo/thả, co giãn và lưu bố cục; add-in C# WinForms hoặc C++ Win32/MFC đọc dữ liệu đó khi chạy. Không thay đổi thuật toán CAD, nghiệp vụ, XData/XRecord, cài đặt thông số hoặc event của nút. Thống nhất hợp đồng giao diện, không bắt buộc dùng cùng ngôn ngữ lập trình.

## 2. ID ổn định
- Mỗi hộp thoại có khóa `form.key` phân biệt hoa/thường. C++ resource: `<RC file không phần mở rộng>.<Dialog ID>`, ví dụ `DJoin.IDD_SETTINGS`. WinForms: `<Namespace>.<PartialClass>`, ví dụ `Demo.ExampleForm`.
- Mỗi control có `id` duy nhất, không thay đổi theo thứ tự, vị trí hoặc phiên bản. C++ dùng ID trong `resource.h`; C# dùng `Control.Name`.
- `parent` là `form` hoặc ID cha. Bộ kết nối C++ thông dụng của V4 hỗ trợ control con trực tiếp của dialog; WinForms có thể lồng nhóm nếu parent khớp.
- Đổi tên/xóa ID phải có bước migration đã thử; không gán ID tương tự để đoán.

## 3. Dữ liệu UTF-8, schema 2
```json
{
  "schema": 2,
  "form": {"key": "DJoin.IDD_SETTINGS", "title": "DJOIN - Settings", "width": 784, "height": 720, "dpi": 96},
  "controls": [
    {"id": "IDC_FUZZ", "parent": "form", "type": "textbox", "text": "",
     "base": {"x":455,"y":20,"width":301,"height":32},
     "x":467,"y":20,"width":301,"height":32}
  ]
}
```
- `base` là kích thước/vị trí lúc nhập lần đầu; `x/y/width/height` là bản đã chỉnh. Đây là tọa độ bản xem trước tại DPI tham chiếu, không phải tọa độ CAD thế giới.
- Bộ nhập RC chuyển DLU sang hệ tọa độ preview xấp xỉ; điều này không có nghĩa khung preview khớp pixel từng font.
- Khi chạy, adapter tính hiệu `đã chỉnh - base`, co giãn theo DPI và áp trên vị trí native vừa được tạo, không thay thế tuyệt đối hàm bố cục gốc.
- `text` dùng để hiển thị trong Studio, không được ghi đè nhãn tại runtime (để bảo toàn đa ngôn ngữ và data binding).
- Hỗ trợ các loại control được adapter công bố; không tự sinh control ảo. Schema 1 cũ chỉ dùng để đọc bố cục legacy.

## 4. Nơi lưu và lỗi
- Studio gợi ý tên `<form.key>.ui.json`. Người sử dụng chọn nơi lưu; Studio chạy độc lập, không suy đoán DUser.
- Với DJoin, đọc một file từ DUser DIRECT đã phân giải: `DJoin.IDD_SETTINGS.ui.json`. Không tự tạo DUser hoặc fallback sang TEMP/AppData.
- Studio ghi UTF-8 vào file `.tmp` cùng thư mục, flush, tạo bản `.bak` khi ghi đè, thay thế an toàn. Chỉ báo lưu thành công sau khi hoàn tất.
- Không có tệp bố cục: giữ giao diện gốc. Sai schema/key/ID, trùng ID, sai kích thước hoặc dữ liệu hỏng: từ chối toàn bộ cấu hình tùy chỉnh, cảnh báo và không đổi nghiệp vụ.

## 5. Tích hợp tối thiểu
- C++ Win32/MFC: map ID -> GetDlgItem; gọi `Layout::load` và `Layout::apply` sau WM_INITDIALOG. Với form có tính toán lại bố cục: áp delta sau mỗi layout native và không cộng dồn.
- C# WinForms: dùng bộ kết nối `DcadLayout.Attach(form, path, expectedFormKey)` sau InitializeComponent; chỉ áp khi form đã bố trí xong.
- Không nạp và thực thi DLL/ARX bất kỳ trong EXE Studio để trích xuất giao diện. WPF/Qt, giao diện C++ tạo động và điều khiển tự vẽ cần adapter riêng; không tuyên bố tự động tương thích.

## 6. Kiểm thử và phạm vi
- Đối chiếu key, 100% ID trong form, parent, giới hạn kích thước, giới hạn dịch chuyển, DPI và file sai.
- Thử mở/chỉnh/lưu/mở lại; thiếu file, file sai, phiên bản không khớp; kiểm thao tác nút, ESC, Undo, data-binding và ứng dụng thật trên host tương ứng.
- Ghi PASS / FAIL / NotVerified riêng cho kiểm tra tĩnh, build, APPLOAD/NETLOAD, runtime UI theo STD-16/18.
- Không đưa JSON bố cục vào SQLite nghiệp vụ, XData, hoặc thay đổi CAD command.

**Hiệu lực:** Bổ sung cho STD-13 trong phạm vi module tích hợp UI Studio; không thay thế STD-03/05/06/07/16/18.