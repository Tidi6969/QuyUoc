# DCAD-STD-10 — Danh mục HoleType và ý nghĩa hình học

**Phiên bản:** 1.5  
**Mức áp dụng:** Theo tính năng

## 17 mã HoleType canonical

| # | Mã kỹ thuật | Hình học / nguồn hình học | Số Hole thật | PointAngle / ghi chú |
|---|---|---|---|---|
| 01 | `head.sink` | Body Circle + Head Circle | 2 bước | 0°, Head/H/T |
| 02 | `head_profile.sink` | Body Profile + Head Circle | 2 bước | 0°, Head/H/T |
| 03 | `head.sink_profile` | Body Circle + Head Profile | 2 bước | 0°, Head/H/T |
| 04 | `user` | User/PGeometry | 1 | 0° |
| 05 | `user_circle` | Circle ngoại tiếp User/PGeometry | 1 | 0°; tính ngoại tiếp tại Gap=0 rồi áp Gap |
| 06 | `body` | Body Circle | 1 | 0° |
| 07 | `body_profile` | Body Profile | 1 | 0° |
| 08 | `head` | Head Circle | 1 | 0°; cần Head |
| 09 | `body_clear.user` | User/PGeometry + Body Circle clearance | 2 | 120°; Main xuyên, Secondary Depth B |
| 10 | `body_profile_clear.user` | User/PGeometry + Body Profile clearance | 2 | 120°; Main xuyên, Secondary Depth B |
| 11 | `pin` | Pin Circle từ Dpin | 1 | Xuyên 0°, mù 120°; PIN_D/PIN_L |
| 12 | `user.slug` | User/PGeometry ở cả hai bước | 2 | 120°; **cả hai Hole thật**, không dùng Body D cho bước 2 |
| 13 | `user.slug_angle` | User/PGeometry + outline relief mặt đối diện | **1** | PointAngle 0°; CutEdge/ReliefAngle riêng; outline mô tả không XData/STT |
| 14 | `user.circle_slug` | User/PGeometry + Circle ngoại tiếp | 2 | 120°; Circle ngoại tiếp tính tại Gap=0 |
| 15 | `thread` | Lỗ ren: danh nghĩa M + lỗ lõi | 1 nghiệp vụ ren | Xuyên 0°, mù 120°; dùng catalog MSW/Ti.dat |
| 16 | `mscrew` | Set screw + metadata vít trí | 1 nghiệp vụ vít | Xuyên 0°, mù 120°; có MSW metadata |
| 17 | `input_out` | Người dùng nhập trực tiếp 1–2 Circle | Tối đa 2 bước | 120° theo family; kích thước theo contract A/B |

## Quy tắc kỹ thuật

1. **Cùng technical key = cùng ý nghĩa hình học**, không thêm tiền tố tên command chỉ vì nguồn hình học ở command khác.
2. GeometryProvider của mỗi family chịu trách nhiệm cung cấp Body/Head/User/Profile/Thread/Pin geometry; HoleType quy định cách kết hợp các nguồn này. Không đồng nhất geometry source với HoleType.
3. Khi có hai bước: Main/Secondary, Gap A/B và Depth A/B phải được khai báo rõ theo từng loại. `user.slug_angle` là **1 Hole thật**, không phải 2.
4. `ReliefAngle` của slug angle là góc thoát, **khác** drill `PointAngle`; `Lcone = Thickness − CutEdge`, `Grow = Lcone × tan(ReliefAngleDeg)`.
5. **CP allow-list** khi dùng quy tắc DPunch đã công bố: `user`, `user.slug`, `user.slug_angle`; đây là lọc giao diện, không xóa các khóa khác khỏi catalog.
6. Writer luôn ghi **canonical key**, không ghi tên đã dịch. Reader được phép chuẩn hóa alias legacy nhưng không viết lại key cũ.
7. Alias reader: `sink→head.sink`, `select→user`, `bod_slu`/`bod_slug`/`body_clear`/`body.clear_user→body_clear.user`, `sel_slu`/`sel_slug`/`user_slug→user.slug`, `user_slug_angle→user.slug_angle`, `threaded→thread`, `msw→mscrew`, `i_out→input_out`.
8. **Ngoại lệ chỉ cho settings legacy:** khi đọc một số settings schema cũ, `NormalizeLegacy("user")→input_out`; canonical hiện hành `user` vẫn là `user`.
9. Command dùng chung key phải kiểm thử Catalog/UI, geometry source, Gap/Depth, calculation, preview, drawing, XData writer, reader, thống kê và đổi ngôn ngữ; không khẳng định hỗ trợ chỉ vì key có mặt trong code.

## Tách dữ liệu thật và mô tả

- `user.slug`: **2 entity Hole thật** cùng UserGeometry, hai Hole XData đúng contract.
- `user.slug_angle`: **1 entity Hole thật** và outline mô tả ở mặt đối diện; outline không Hole XData, không STT và không được thống kê thành lỗ thứ hai.
- Hàng snapshot trong Punch MASTER không phải entity lỗ thực; không dùng để tăng quantity.
