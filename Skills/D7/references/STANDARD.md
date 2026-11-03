# DCAD-STD-07 — Đa ngôn ngữ và Unicode

**Phiên bản:** 1.11  
**Cập nhật:** 03/11/2026  
**Mức áp dụng:** Bắt buộc khi có giao diện hoặc text

## Nguồn và cách chọn ngôn ngữ

- `Buildtime/lang.dat` là dữ liệu đa ngôn ngữ **nhúng vào DLL**; runtime không tìm `lang.dat` cạnh DLL hoặc ngoài đĩa.
- Mặc định lấy mã ngôn ngữ từ **dòng 28** của `DUser/MIS_SET.SET`; vị trí file bắt buộc lấy từ **bộ phân giải DUser DIRECT theo DCAD-DATA-DIR-001** (quy chuẩn 05), không tự dò lại thư mục trong `Lang.cs`. Chỉ chấp nhận `DUser` được đăng ký trực tiếp trong Support File Search Path, không tìm thư mục con từ entry cha dù file có tồn tại. `MIS_SET.SET` là tệp bắt buộc cho module sử dụng cơ chế ngôn ngữ DCAD; tên thư mục cuối của entry phải là DUser (không phải `DUser\abc`); Setup/QA và module không ép đủ năm thư mục hay vị trí cuối danh sách, chỉ kiểm phụ thuộc thực tế theo STD-05. Plugin không quản lý ngôn ngữ chỉ được **đọc**, không tự ghi vào file chung.
- Khi bắt đầu command, dùng `Lang.BeginCommand()` và `Lang.Reload()` trong các project triển khai hạ tầng `Lang` này.
- UI gọi `Lang.T("KeyName")`; không viết các nhánh `if/switch` ngôn ngữ rải rác trong business logic, form và drawing.

## Bộ ngôn ngữ

| Block | Ngôn ngữ |
|---|---|
| `Lang_0(English_en-US)` | Tiếng Anh (dự phòng khi thiếu key bản dịch) |
| `Lang_1(Vietnamese_vi-VN)` | Tiếng Việt |
| `Lang_2(SimplifiedChinese_zh-CN)` | Tiếng Trung giản thể |
| `Lang_3(Japanese_ja-JP)` | Tiếng Nhật |
| `Lang_4(Korean_ko-KR)` | Tiếng Hàn |
| `Lang_5(Thai_th-TH)` | Tiếng Thái |

- Cho phép thêm `Lang_6`, `Lang_7`... Parser nhận động mọi `Lang_X`; các block có cùng danh sách key, không để value rỗng.
- Header hợp lệ theo mẫu `^Lang_(\d+)\(([^_()]+)_([^_()]+)\)$`; mỗi block kết thúc theo format `lang.dat` đã chốt.
- Fallback **ngôn ngữ hiện hành → Lang_0 → key** chỉ áp dụng khi `MIS_SET.SET` hợp lệ nhưng **một key bản dịch** bị thiếu. Khi `MIS_SET.SET` thiếu/sai hoặc DUser DIRECT không hợp lệ, hiển thị **cảnh báo yêu cầu thiết lập lại**, dừng chức năng cần ngôn ngữ DCAD; không âm thầm chọn English làm ngôn ngữ vận hành. Thông báo cấu hình có thể dùng chuỗi thông báo tối thiểu đã nhúng để vẫn hiển thị được khi hệ thống chọn ngôn ngữ gặp lỗi. Không tự sửa file hoặc tìm ở nơi khác.
- Technical key XData, HoleType, mã layer, code loại đối tượng **không dịch** theo ngôn ngữ UI.

## Unicode và font

- Đảm bảo Unicode hiển thị đúng trong WinForms, CAD DBText/MText, DIM và preview; kiểm thử cả TTF/SHX trên từng host mục tiêu.
- Không ép một font cố định nếu dự án chưa chốt, không thay font của các đối tượng có sẵn ngoài phạm vi.
- File dữ liệu cũ theo ANSI/UTF-8 phải theo encoding riêng của format; không tự chuyển mã khi chưa có kế hoạch migration.
