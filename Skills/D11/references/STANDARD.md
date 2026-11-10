# DCAD-STD-11 — Gap, Dia, Depth và Offset

**Phiên bản:** 1.6  
**Mức áp dụng:** Theo tính năng

## Gap, Dia, Depth, Offset, W/T/R và Create↔Read [THEO TÍNH NĂNG]

**Hai ngữ cảnh**: module **Create** tính và vẽ dữ liệu mới; module **Read** đọc/nhóm/thống kê từ entity có sẵn. Có thể khác cài đặt nội bộ, nhưng phải giữ **cùng hợp đồng dữ liệu** và round-trip nhất quán.

- Parser chung trong phạm vi project cần giữ `RawGap`, `MainGap`, `SecondaryGap`, `MainMode`, `SecondaryMode`, `HasSecondary`, `IsValid`, `XDataGap`, `UsesWireLayer`, `SuppressOffset`. Có thể có `A` hoặc `A/B`. Không vứt dấu âm, hậu tố W/T/R hoặc chuỗi nguồn.
- **Normal** (khi type quy định): `FinalDia = BaseDia + 2 × Gap`.
- **W**: dùng layer `*_W`, **không** cộng số trước W vào đường kính (`FinalDia = BaseDia`); giữ mode và giá trị gốc theo contract.
- **T**: Wire đã offset sẵn, dùng layer Wire nhưng **có cộng** `TValue`: `FinalDia = BaseDia + 2 × TValue`; XDataGap của **từng token T** chuẩn hóa thành `0.0W` sau khi tính hình học. Ví dụ `0.1T/0 → 0.0W/0`.
- **R**: không có công thức global; mỗi HoleType/provider Create phải định nghĩa rõ, reader phải giữ mode R và áp đúng contract; không tự coi như Normal.
- **Gap âm**: không cấm trên toàn hệ thống, phải theo `AllowNegativeGap`/`GapSignMode` của HoleType. Không `Math.Abs` toàn bộ Gap hoặc Depth làm mất dấu/hướng.
- Hai bước A/B parse và chọn BaseDia, Depth, layer, writer cho **từng phần độc lập**. Số trong `T` là offset theo bán kính mm; `W` và `T` cùng Wire layer **nhưng khác cách tính Dia**.
- Trong các module vẽ Hole theo PlateCatalog (STD-05/09), layer phải lấy từ `PlateCode` đã chuẩn hóa: Normal theo option `o` là `<PlateCode>_O`, còn W/T theo option `w` là `<PlateCode>_W`. Ví dụ DAT `(UP)` tạo `UP_O` hoặc `UP_W`, không tạo `(UP)_O` hay `(UP)_W`. Việc chọn layer này không thay đổi phép tính Dia và không làm mất Gap gốc; bước A/B chọn layer độc lập.
- Với `user.slug_angle`: `Lcone = Thickness − CutEdge`, `Grow = Lcone × tan(ReliefAngleDeg)` theo hợp đồng HoleType; descriptive geometry không tự trở thành entity Hole thứ hai. Angle relief không ghi vào PointAngle.
- **Round-trip tối thiểu**: `1`, `0`, `0.1W`, `0.1T`, `0.2R`, `1/0`, `0.01W/0`, `0.01T/0`, `0.01T/0.02T`, `-0.02`, `-0.02R`, `-0.02/0` (chỉ các trường hợp hợp lệ theo từng HoleType). Create → DWG → Read phải giữ FinalDia/geometry/mark/note/layer/XData nhất quán.
- Khi schema không hỗ trợ một mode/âm/R: trả trạng thái không hỗ trợ với code + RawGap, không tạo kết quả sai để “cho chạy được”.
