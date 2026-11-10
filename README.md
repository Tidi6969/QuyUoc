# QuyUoc — Bộ tiêu chuẩn DCAD V1.15

Cập nhật 10/11/2026. Xem [mục lục 00–19](00_MUC_LUC.md).

- Support Path: tên thư mục cuối phải khớp DData/DUser/DProgram/Dicon/Dhelp; không bắt đủ năm, cùng root hoặc vị trí cuối danh sách. Module/Setup chỉ kiểm phụ thuộc thực tế; đường dẫn con ứng dụng khai báo riêng (STD-05/06).
- Phiên bản phần mềm: không chỉ định baseline thì lấy bản gần nhất đã lưu; không chỉ định version đích thì max đã lưu + 1. Giữ các ZIP/changelog cũ, thêm bản mới, đồng bộ README module và DCAD; tách mới nhất/đã nghiệm thu (STD-01/17).
- [ZIP bộ quy ước V1.15](Releases/V1.15/QuyUoc-V1.15.zip). Tên file Markdown hiện có được giữ để bảo toàn URL; phiên bản hiệu lực nằm trong nội dung.
- Lần cập nhật này chỉ sửa tài liệu, không sửa hoặc build lại phần mềm CAD. Bản tài liệu trước có thể xem trong lịch sử Git; các ZIP tài liệu đã lưu được giữ nguyên.
- **STD-18:** AI suy nghĩ trước khi code, ưu tiên đơn giản, sửa tối thiểu và kiểm chứng theo mục tiêu. [Chi tiết quy ước 18](18_Karpathy_Guidelines_cho_AI_lap_trinh.md).
- **STD-19:** kiến trúc License DCAD, ToolUid/FeatureUid, Manifest, Dmanage xuất bộ mã, File ID, License Client và QA LIC01–LIC26. [Chi tiết quy ước 19](19_Kien_truc_ban_quyen_DCAD_va_dong_bo_DLL.md).
- Bản V1.14 chỉ bổ sung/sửa tài liệu; giữ nguyên ZIP V1.13 và lịch sử trước đó. Không sửa hoặc build lại phần mềm CAD.
- **STD-05/09/11:** DAT `(UP)` hiển thị `UP`, Hole Normal vẽ `UP_O`, Wire W/T vẽ `UP_W`; giữ nội dung DAT và mã có `_` như `PH_P`. Không tự đổi màu layer cũ.
- V1.15 chỉ cập nhật quy ước và tài liệu, không build hay thay đổi ứng dụng CAD; các bản V1.14 trở về trước vẫn được giữ lại.
