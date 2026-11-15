# DCAD-STD-07 — Đa ngôn ngữ và Unicode

**Phiên bản:** 1.12  
**Cập nhật:** 15/11/2026  
**Mức áp dụng:** Bắt buộc khi có giao diện hoặc text

## Nguồn và cách chọn ngôn ngữ

- `Buildtime/lang.dat` là dữ liệu đa ngôn ngữ **nhúng vào DLL**; runtime không tìm `lang.dat` cạnh DLL hoặc ngoài đĩa.
- Nguồn chọn ngôn ngữ duy nhất là **`DUser/Shared_data.Set`**. Đây là dữ liệu chung **bắt buộc** khi chức năng dùng cơ chế ngôn ngữ DCAD; khai báo `LogicalFolder=DUser`, `Required=true`, `AccessMode=ReadOnly` trong DataManifest. Không dùng file cũ hoặc nguồn dự phòng.
- Vị trí file lấy từ **bộ phân giải DUser DIRECT theo DCAD-DATA-DIR-001** (STD-05): chỉ chấp nhận mục đăng ký trực tiếp có tên thư mục cuối `DUser`. Không tìm từ thư mục cha, không tìm đệ quy, không lấy từ thư mục DLL. Không ép đủ năm thư mục hay thứ tự Support File Search Path; không xây dựng resolver riêng trong `Lang.cs`.
- Đọc đúng **dòng vật lý đầu tiên (dòng 1)** của `Shared_data.Set`. Sau BOM/khoảng trắng đầu dòng (nếu có), dòng phải bắt đầu bằng `[<mã>]`; chỉ lấy nội dung **trong cặp dấu `[` và `]`**, trim khoảng trắng trong ngoặc rồi đọc thành **số nguyên thập phân không âm** để chọn block `Lang_<mã>`. Bỏ qua phần mô tả sau `]`; không tìm mã ở dòng 2 hoặc ở phần khác của tệp.
- Ví dụ minh họa: `[1]Ngôn ngữ` chọn `Lang_1` (Tiếng Việt); `[0]Language` chọn `Lang_0` (Tiếng Anh). Giá trị `[6]` chỉ hợp lệ khi `lang.dat` nhúng có `Lang_6`. File trống, thiếu ngoặc, ngoặc rỗng, số âm, ký tự không phải số trong `[]`, tràn kiểu số hoặc thiếu block tương ứng đều là **dữ liệu cấu hình không hợp lệ**.
- Module không quản trị cấu hình ngôn ngữ chỉ **đọc** `Shared_data.Set`: không tự tạo, sửa hoặc đổi mã hóa file; giữ hợp đồng encoding của dữ liệu nguồn. Thay đổi mã ngôn ngữ được nạp khi bắt đầu lượt command tiếp theo, không giám sát theo thao tác từng tab/nút.
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
- Fallback **ngôn ngữ hiện hành → Lang_0 → key** chỉ áp dụng khi `Shared_data.Set` dòng 1 hợp lệ, mã trong `[]` khớp một block đã nhúng nhưng **thiếu một key bản dịch**. Khi file thiếu/sai, DUser DIRECT không hợp lệ hoặc mã không có block `Lang_X`, phải hiển thị **cảnh báo yêu cầu thiết lập lại** và dừng chức năng phụ thuộc; không âm thầm chọn English làm ngôn ngữ vận hành. Thông báo cấu hình có thể dùng chuỗi tối thiểu nhúng sẵn để báo lỗi ngay cả khi không xác định được ngôn ngữ. Không tự sửa file hoặc dò nguồn khác.
- Technical key XData, HoleType, mã layer, code loại đối tượng **không dịch** theo ngôn ngữ UI.

## Kiểm chứng nguồn chọn ngôn ngữ

| ID | Điều kiện thử | Kết quả bắt buộc |
|---|---|---|
| LANG01 | DUser DIRECT hợp lệ, dòng 1 `[1]Ngôn ngữ` | Chọn Lang_1; bỏ nội dung sau `]` |
| LANG02 | Dòng 1 `[0]Language` | Chọn Lang_0 theo yêu cầu rõ ràng, không phải fallback |
| LANG03 | Dòng 1 `[6]...`, có/không có Lang_6 nhúng | Có block → chọn Lang_6; thiếu block → cảnh báo, dừng |
| LANG04 | Dòng 1 rỗng, thiếu ngoặc, `[]`, `[-1]`, `[abc]`, `[999999999999]` | Cảnh báo thiết lập lại; không chuyển sang English |
| LANG05 | Dòng 1 sai, dòng 2 có `[1]` | Cảnh báo; không tự đọc dòng sau |
| LANG06 | Thiếu file, thiếu DUser DIRECT hoặc nhiều DUser DIRECT khác đích | Cảnh báo đúng nguyên nhân; không dùng nguồn thay thế |
| LANG07 | Dòng 1 hợp lệ nhưng thiếu key bản dịch trong Lang_X | Chỉ fallback key đó sang Lang_0 rồi key |
| LANG08 | Thay đổi dòng 1 trước lượt command mới | Lượt mới đọc giá trị mới; không ghi vào file |

Kiểm tra tĩnh parser/manifest **không thay thế** thử NETLOAD và giao diện trên từng CAD thuộc phạm vi module.

## Unicode và font

- Đảm bảo Unicode hiển thị đúng trong WinForms, CAD DBText/MText, DIM và preview; kiểm thử cả TTF/SHX trên từng host mục tiêu.
- Không ép một font cố định nếu dự án chưa chốt, không thay font của các đối tượng có sẵn ngoài phạm vi.
- File dữ liệu cũ theo ANSI/UTF-8 phải theo encoding riêng của format; không tự chuyển mã khi chưa có kế hoạch migration.
