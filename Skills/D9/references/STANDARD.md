# DCAD-STD-09 — Hình học, layer, text và DIM

**Phiên bản:** 1.6  
**Mức áp dụng:** Theo tính năng


## Hình học và sai số

- Đơn vị mặc định của Geometry, Dia, Gap, Depth, Offset, dung sai là **mm**, trừ khi nghiệp vụ khai báo khác rõ ràng.
- Các giá trị trong file dữ liệu/XData viết bằng **InvariantCulture**, dùng dấu chấm thập phân.
- Không so sánh số thực `double` bằng `==`. Ngưỡng tham chiếu: `ZeroTolerance=1e-9 mm`, `GeometryTolerance=1e-6 mm`. Sai số nhóm/so sánh sản xuất phải khai báo riêng, không mặc nhiên bằng GeometryTolerance.
- Validate kích thước >0, biên dạng kín/hợp lệ, hướng, khoảng cách và ràng buộc riêng của object **trước khi vẽ**.
- Tách `RawGap`, `BaseDia`, `FinalDia`, `DisplayDia`, `EffectiveDia`, `ToleranceDia`; không tái sử dụng một biến cho các giá trị khác nghĩa.

## LayerCatalog

| Option | Layer nguồn/đích |
|---|---|
| `c` | Current layer |
| `src` | Layer của đối tượng nguồn |
| `p` | Layer Plate/Base |
| `d` | `DIM` |
| `o` | `<TênGốc>_O` |
| `w` | `<TênGốc>_W` |
| `m` | `<TênGốc>_M` |
| `_t` | `<TênGốc>_TXT` |
| `t` | `TEXT` |
| `*` | Layer `0` |
| `*_` | Layer `1` |

### PlateCode và layer khi dựng Hole [THEO TÍNH NĂNG]

- Nếu module Hole đọc PlateCatalog kiểu `(UP)` từ DData theo STD-05, phải phân biệt `RawPlateCode`, `PlateCode` và `LayerName`. Từ chuỗi nguồn: trim khoảng trắng ngoài; bỏ đúng **một hậu tố layer đã biết** `_O`, `_W`, `_M`, `_TXT` ở cuối nếu có; sau đó chỉ bỏ **một cặp ngoặc cân đối bao toàn bộ** `(...)` nếu có. Không xóa ngoặc ở giữa mã, không cắt theo dấu gạch dưới cuối và không thay DAT gốc. Các ví dụ: `(UP)` → `UP`, `(PH_P)` → `PH_P`, `(DIE2)_W` → `DIE2`, `PH_P_W` → `PH_P`.
- Combo chọn Plate hiển thị mã `PlateCode` không ngoặc: `UP`, `UB`, `PH`… Khi chọn option `o` cho lỗ thường (Normal), tạo `<PlateCode>_O`; với mode Gap `W` hoặc `T`, chọn option `w`, tạo `<PlateCode>_W`. Ví dụ `UP_O` / `UP_W`, `PH_P_O` / `PH_P_W`. Không ghép lặp hậu tố thành `UP_O_O` hay `UP_W_W`; A và B của lỗ hai bước xác định layer độc lập theo mode từng bước, theo STD-11.
- Option `p` vẫn có nghĩa layer Plate/Base chính là `PlateCode` (ví dụ `UP`), **không** tự thêm `_O`; các option khác trong LayerCatalog không bị ép đổi. Quy tắc `_O` mặc định ở đây chỉ áp dụng cho nghiệp vụ vẽ Hole theo Plate đã được duyệt, không tự sửa các module không có hợp đồng này.
- Dùng lại layer đích đã tồn tại mà không thay màu, linetype hay lineweight của nó; nếu chưa có, tạo trong transaction với thuộc tính theo cài đặt/tiêu chuẩn được duyệt, không tự ấn định màu ACI/RGB. Không thay current layer hoặc đối tượng có sẵn. Preview, hình vẽ, XData phải lấy cùng kết quả calculation.
- QA bắt buộc theo tính năng: `(UP)` → combo `UP`, Normal → `UP_O`, W/T → `UP_W`; `(PH_P)` → combo `PH_P`, Normal → `PH_P_O`, W/T → `PH_P_W`; `(DIE2)_W` không thành `(DIE2)_W_O`; lặp thao tác không lặp hậu tố; DAT/layer gốc giữ nguyên; phân biệt layer A/B.

- Chỉ loại hậu tố đã biết: `_O`, `_W`, `_M`, `_TXT`. Không cắt theo dấu `_` cuối nếu tên gốc có dấu `_`.
- Tạo layer mới trong transaction, không tự thay current layer hoặc layer của entity có sẵn; màu, nét, lineweight do cấu hình/tiêu chuẩn riêng quyết định.
- Không tạo layer debug trong Release.

## Text và DIM

- Default tham chiếu: `TextStyleName=STANDARD`, `TextWidthFactor=0.8` cho Mark/Note/DIM; chỉ áp dụng khi project chọn bộ mặc định này.
- Không tự sửa style/font/kích thước của đối tượng có sẵn. Chiều cao, góc nghiêng, width factor phải validate.
- Tolerance, precision và zero suppression phải theo settings/dimension contract đã được dự án chốt; không ghi DIM bằng số khác với hình học thực.
- Preview, drawing và XData phải sử dụng **cùng một kết quả calculation**; không để mỗi phần tính Dia khác nhau.
