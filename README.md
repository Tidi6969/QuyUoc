# QuyUoc — Bộ tiêu chuẩn DCAD V1.17

Tài liệu trực tuyến cập nhật 19/11/2026 (Reference và tên output không suffix CAD); ZIP phát hành gần nhất vẫn là bản lưu V1.17. Xem [mục lục 00–18](00_MUC_LUC.md).

- Support Path: tên thư mục cuối phải khớp DData/DUser/DProgram/Dicon/Dhelp; không bắt đủ năm, cùng root hoặc vị trí cuối danh sách. Module/Setup chỉ kiểm phụ thuộc thực tế; đường dẫn con ứng dụng khai báo riêng (STD-05/06).
- Phiên bản phần mềm: không chỉ định baseline thì lấy bản gần nhất đã lưu; không chỉ định version đích thì max đã lưu + 1. Giữ các ZIP/changelog cũ, thêm bản mới, đồng bộ README module và DCAD; tách mới nhất/đã nghiệm thu (STD-01/17).
- [ZIP bộ quy ước V1.17](Releases/V1.17/QuyUoc-V1.17.zip). Tên file Markdown hiện có được giữ để bảo toàn URL; phiên bản hiệu lực nằm trong nội dung.
- Lần cập nhật này chỉ sửa tài liệu, không sửa hoặc build lại phần mềm CAD. Bản tài liệu trước có thể xem trong lịch sử Git; các ZIP tài liệu đã lưu được giữ nguyên.
- **STD-12 v1.6:** chuẩn MASTER XData dựa trên Dtscr1 V84 cho Drestore; giữ DPunch V381 trong nhánh reader legacy. [Chi tiết quy ước 12](12_Ghi_doc_XData_va_XRecord.md).
- **STD-19:** đã rút và xóa tài liệu hiện hành ngày 17/11/2026 theo yêu cầu chủ dự án. Nội dung cũ chỉ còn trong lịch sử Git, không còn là quy ước đang áp dụng.
- **STD-18:** AI suy nghĩ trước khi code, ưu tiên đơn giản, sửa tối thiểu và kiểm chứng theo mục tiêu. [Chi tiết quy ước 18](18_Karpathy_Guidelines_cho_AI_lap_trinh.md).
- Bản V1.14 chỉ bổ sung/sửa tài liệu; giữ nguyên ZIP V1.13 và lịch sử trước đó. Không sửa hoặc build lại phần mềm CAD.
- **STD-05/09/11:** DAT `(UP)` hiển thị `UP`, Hole Normal vẽ `UP_O`, Wire W/T vẽ `UP_W`; giữ nội dung DAT và mã có `_` như `PH_P`. Không tự đổi màu layer cũ.
- V1.15 chỉ cập nhật quy ước và tài liệu, không build hay thay đổi ứng dụng CAD; các bản V1.14 trở về trước vẫn được giữ lại.

- **V1.16 — Reference native dùng chung:** Build ARX/GRX tìm sâu theo cấu trúc SDK tại `D:\000\Reference\<CAD>`, kiểm tra file và xung đột SHA trước compiler; ZIP source không cần chép SDK nặng. Chi tiết STD-03/04/17; STD-05 runtime giữ nguyên.

- **V1.17 — Chọn ngôn ngữ DCAD:** theo STD-07 chỉ đọc dòng 1 của `DUser/Shared_data.Set` và lấy số trong `[]`. Đồng bộ STD-05/06/16, không dùng file cấu hình cũ hoặc giả định English. Chỉ cập nhật tài liệu, không build CAD.

- **Cập nhật 19/11/2026 — tên thành phẩm:** STD-03 v1.16 / STD-04 v1.16 / STD-16 v1.17 yêu cầu output `<ProjectName>.dll`, `.arx` hoặc `.grx` **không thêm `C22`/`C25`/`G27`**. Thư mục `Dprogram/AutoCAD2022`, `Dprogram/AutoCAD2025`, `Dprogram/GstarCAD2027` phân biệt binary trong ZIP; khi cài chọn đúng host đưa vào `DProgram` phẳng. Root Reference dùng ba thư mục `D:\000\Reference\Autocad 2022`, `D:\000\Reference\Autocad 2025`, `D:\000\Reference\GstarCAD 2027`. Chỉ cập nhật tài liệu online, không sửa code module; ZIP V1.17 không đổi.
