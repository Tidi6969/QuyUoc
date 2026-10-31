# DCAD-STD-09 — Hình học, layer, text và DIM

**Phiên bản:** 1.5  
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

- Chỉ loại hậu tố đã biết: `_O`, `_W`, `_M`, `_TXT`. Không cắt theo dấu `_` cuối nếu tên gốc có dấu `_`.
- Tạo layer mới trong transaction, không tự thay current layer hoặc layer của entity có sẵn; màu, nét, lineweight do cấu hình/tiêu chuẩn riêng quyết định.
- Không tạo layer debug trong Release.

## Text và DIM

- Default tham chiếu: `TextStyleName=STANDARD`, `TextWidthFactor=0.8` cho Mark/Note/DIM; chỉ áp dụng khi project chọn bộ mặc định này.
- Không tự sửa style/font/kích thước của đối tượng có sẵn. Chiều cao, góc nghiêng, width factor phải validate.
- Tolerance, precision và zero suppression phải theo settings/dimension contract đã được dự án chốt; không ghi DIM bằng số khác với hình học thực.
- Preview, drawing và XData phải sử dụng **cùng một kết quả calculation**; không để mỗi phần tính Dia khác nhau.
