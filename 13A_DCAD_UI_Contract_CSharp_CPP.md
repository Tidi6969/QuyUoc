# DCAD-UI-CONTRACT-001 — Hợp đồng bố cục giao diện dùng chung C# / C++

**Phiên bản:** 1.1 — 20/11/2026.
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
## 7. Manifest nguồn giao diện — schema 1 (bổ sung V1.1)

**Khác với layout schema 2**: manifest schema 1 là file duy nhất người thiết kế chọn để Studio biết phải đọc nguồn nào, nạp preview nào và lưu bố cục về đâu. Manifest không chứa giá trị nghiệp vụ hay kết quả chỉnh vị trí. Ví dụ DJoin:

```json
{
  "schema": 1,
  "project": "DJoin",
  "kind": "native-win32",
  "formKey": "DJoin.IDD_SETTINGS",
  "source": {
    "resource": "DJoin-V29/DJoinARX-AutoCAD2022-V19/src/DJoin.rc",
    "resourceIds": "DJoin-V29/DJoinARX-AutoCAD2022-V19/src/resource.h",
    "controller": "DJoin-V29/DJoinARX-AutoCAD2022-V19/src/SettingsNative.cpp",
    "language": "DJoin-V29/DJoinARX-AutoCAD2022-V19/Buildtime/lang.dat"
  },
  "preview": {
    "module": "NativePreview/Dprogram/DJoinUiPreview.dll"
  },
  "runtime": {
    "folder": "DUser",
    "settingsFile": "Djoin_settings.ini",
    "languageSelection": "Shared_data.Set",
    "layoutFile": "DJoin.IDD_SETTINGS.ui.json"
  }
}
```

- `source` và `preview.module`: đường dẫn tương đối từ thư mục chứa manifest, không cho đường dẫn tuyệt đối hay `..`. Kiểm tra tệp tồn tại và tính tương thích ID trước khi mở.
- `runtime.folder`: thư mục logic DUser DIRECT, **không** suy ra từ project, thư mục EXE, root CAD hay AppData. Trong Studio độc lập người dùng chọn DUser; add-in CAD tiếp tục phân giải qua Support Path đúng STD-05. `runtime.settingsFile` và `languageSelection` phải khớp thật với adapter.
- `kind`: `native-win32` dùng module preview được build riêng từ **cùng** file RC và mã điền UI với add-in; `winforms` dùng assembly UI-only và `preview.formClass` là tên đầy đủ của lớp Form. Chưa hỗ trợ WPF/Qt tự động.
- Đối với DJoin, trình build preview biên dịch đúng `DJoin.rc`, `resource.h`, `Buildtime/lang.dat` và cùng `SettingsUiView.cpp` mà ARX/GRX biên dịch. Khi thay các nguồn này **phải build lại module preview**, không chấp nhận preview cũ như giao diện thực mới.
- Studio không được nạp `.arx/.grx` hoặc chạy ObjectARX để mở preview. Chỉ nạp module/assembly giao diện từ dự án người dùng tin cậy; phải có thông báo trước khi thực thi mã dự án.

## 8. Luồng UI Studio chuẩn

1. Mở một `*.ui.manifest.json` đã được xác thực.
2. Kiểm tra `formKey`, các tệp nguồn, module preview, ID và tương thích công nghệ.
3. Mở **control thật** từ resource/assembly đúng manifest, không tự vẽ các nút giả. Nạp dữ liệu mặc định nếu chưa chọn DUser; phải thông báo rõ chưa có dữ liệu thực.
4. Người dùng chọn DUser DIRECT hợp lệ để nạp `Shared_data.Set` dòng 1, `Djoin_settings.ini` (nếu có); chỉ đọc, không ghi settings.
5. Cho chuyển giữa chế độ **chỉnh bố cục** và **xem control hoạt động**. Không chạy nghiệp vụ CAD, không lưu settings khi bấm nút preview.
6. Chỉ ghi `runtime.layoutFile` dưới DUser (atomic + backup theo STD-06). JSON bố cục giữ schema 2, `form.key`, `id`, `parent`, `base`, `x/y/width/height` và DPI chuẩn; không ghi đè giao diện nguồn, SQLite, INI hay bản dịch.
7. Mở lại trong CAD: adapter áp **delta** từ bản gốc một lần mỗi chu kỳ layout. Trường hợp JSON sai/không tương thích phải giữ giao diện gốc và báo rõ.

## 9. QA riêng cho manifest & preview

- **MAN01:** file manifest hợp lệ và đúng schema; **MAN02:** thiếu/sai nguồn; **MAN03:** đường dẫn tuyệt đối/vượt thư mục; **MAN04:** module preview thiếu/không có export; **MAN05:** đủ, không trùng ID; **MAN06:** dùng đúng resource & presenter gốc; **MAN07:** DUser DIRECT đúng và tách default/real; **MAN08:** giữ dữ liệu INI/SQLite/XData; **MAN09:** save/reopen/backup schema 2; **MAN10:** kiểm build Windows, DPI, UI thực và APPLOAD trên từng CAD host.
- Kết quả `PASS/FAIL/NotVerified` phải phân biệt phân tích tĩnh với compiler, Windows runtime và APPLOAD/NETLOAD. Build preview không thay cho kiểm thử ARX. Không công bố hỗ trợ toàn bộ C++/WinForms chỉ dựa trên ví dụ DJoin.

**Lưu ý triển khai:** mẫu V5 chọn một dialog theo `formKey`; đối với dự án nhiều dialog, khai báo manifest riêng cho từng dialog cho đến khi hỗ trợ tập hợp form có kiểm chứng.
