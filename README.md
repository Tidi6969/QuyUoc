# QuyUoc — Bộ tiêu chuẩn DCAD V1.22 (trực tuyến)

Bổ sung Plan phát triển TidiDcad trực tiếp trong mục 1 của [STD-20 V1.3](20_Quy_uoc_ban_giao_key_Addin.md): baseline V55, bảy hạng mục bảo mật, yêu cầu/tiến độ từng bước và quy tắc cập nhật. Không tạo file Plan riêng, không thay hợp đồng kỹ thuật V1.2, không sửa mã nguồn; ZIP tổng V1.17 giữ nguyên.

Cập nhật trực tuyến 26/11/2026 — [STD-20 V1.2](20_Quy_uoc_ban_giao_key_Addin.md)

- Chữ ký RSA-PSS của bộ bàn giao, xác thực một lần khi tiếp nhận; thay thế cơ chế phê duyệt máy build V1.1. Giữ sáu file, Gate và Licensecc. Ngày 26/11/2026 chủ dự án xác nhận TidiDcad V55 chạy OK ngay lần đầu (`UserReportedWorking=YES`); log build và kiểm thử ký/xác minh Windows, nghiệm thu tích hợp Add-in/CAD riêng vẫn `NotVerified` theo chứng cứ độc lập. [Tiến độ DCAD](https://github.com/Tidi6969/DCAD/blob/main/README.md). ZIP tổng V1.17 giữ nguyên.

- **Cập nhật trực tuyến 26/11/2026 — [STD-20 v1.1](20_Quy_uoc_ban_giao_key_Addin.md):** phê duyệt KeyId độc lập một lần, preflight dùng chung, cô lập log/artifact; không đổi Gate/License và cấu trúc sáu file. ZIP tổng V1.17 giữ nguyên; Windows/CAD chưa được nghiệm thu chỉ vì cập nhật tài liệu.

- **Cập nhật trực tuyến 25/11/2026 — STD-20 v1.0:** [Quy ước bàn giao Key và tích hợp bản quyền Add-in DCAD](20_Quy_uoc_ban_giao_key_Addin.md). Bộ ZIP tích hợp đúng sáu file; hướng dẫn cấu hình project, Build_All, Build_Release, README và nghiệm thu theo STD-18. Phạm vi đã khảo sát: native C++ x64, AutoCAD 2022. STD-19 vẫn đã rút. Xem [mục lục V1.21](00_MUC_LUC.md). ZIP tổng đã lưu V1.17 không đổi; không sửa mã hoặc xác nhận nghiệm thu Add-in.

- **Cập nhật trực tuyến 20/11/2026 — Bảng dữ liệu có cấu trúc:** STD-05 v1.17 bổ sung `DCAD-SQLITE-GRID-001` (đọc/ghi SQLite theo đúng bản ghi/cột, khóa ổn định, prepared statement, transaction); STD-13 v1.8 bổ sung `DCAD-GRID-001` dùng chung cho mọi module và mọi nguồn bảng, không tạo ô EDIT dạng TAB giả bảng; STD-16 v1.20 bổ sung GRID01–GRID10 và SQL09–SQL15. Vẫn áp dụng `DProgram\\sqlite3.dll` dùng chung theo STD-03/05. Chỉ sửa tài liệu trực tuyến, ZIP V1.17 chưa được cập nhật.


Tài liệu trực tuyến cập nhật 19/11/2026 (Reference và tên output không suffix CAD); ZIP phát hành gần nhất vẫn là bản lưu V1.17. Xem [mục lục và phụ lục 13A](00_MUC_LUC.md).

- Support Path: tên thư mục cuối phải khớp DData/DUser/DProgram/Dicon/Dhelp; không bắt đủ năm, cùng root hoặc vị trí cuối danh sách. Module/Setup chỉ kiểm phụ thuộc thực tế; đường dẫn con ứng dụng khai báo riêng (STD-05/06).
- Phiên bản phần mềm: không chỉ định baseline thì lấy bản gần nhất đã lưu; không chỉ định version đích thì max đã lưu + 1. Giữ các ZIP/changelog cũ, thêm bản mới, đồng bộ README module và DCAD; tách mới nhất/đã nghiệm thu (STD-01/17).
- [ZIP bộ quy ước V1.17](Releases/V1.17/QuyUoc-V1.17.zip). Tên file Markdown hiện có được giữ để bảo toàn URL; phiên bản hiệu lực nằm trong nội dung.
- Lần cập nhật này chỉ sửa tài liệu, không sửa hoặc build lại phần mềm CAD. Bản tài liệu trước có thể xem trong lịch sử Git; các ZIP tài liệu đã lưu được giữ nguyên.
- **STD-12 v1.6:** chuẩn MASTER XData dựa trên Dtscr1 V84 cho Drestore; giữ DPunch V381 trong nhánh reader legacy. [Chi tiết quy ước 12](12_Ghi_doc_XData_va_XRecord.md).
- **STD-19:** đã rút và xóa tài liệu hiện hành ngày 17/11/2026 theo yêu cầu chủ dự án. Nội dung cũ chỉ còn trong lịch sử Git, không còn là quy ước đang áp dụng.
- **DCAD-UI-CONTRACT-001 (13A):** Chuẩn đồng bộ Form/Control ID, JSON schema 2 và adapter C#/C++ khi tích hợp DCAD UI Studio. [Quy ước 13A](13A_DCAD_UI_Contract_CSharp_CPP.md).
- **STD-18:** AI suy nghĩ trước khi code, ưu tiên đơn giản, sửa tối thiểu và kiểm chứng theo mục tiêu. [Chi tiết quy ước 18](18_Karpathy_Guidelines_cho_AI_lap_trinh.md).
- Bản V1.14 chỉ bổ sung/sửa tài liệu; giữ nguyên ZIP V1.13 và lịch sử trước đó. Không sửa hoặc build lại phần mềm CAD.
- **STD-05/09/11:** DAT `(UP)` hiển thị `UP`, Hole Normal vẽ `UP_O`, Wire W/T vẽ `UP_W`; giữ nội dung DAT và mã có `_` như `PH_P`. Không tự đổi màu layer cũ.
- V1.15 chỉ cập nhật quy ước và tài liệu, không build hay thay đổi ứng dụng CAD; các bản V1.14 trở về trước vẫn được giữ lại.

- **V1.16 — Reference native dùng chung:** Build ARX/GRX tìm sâu theo cấu trúc SDK tại `D:\000\Reference\<CAD>`, kiểm tra file và xung đột SHA trước compiler; ZIP source không cần chép SDK nặng. Chi tiết STD-03/04/17; STD-05 runtime giữ nguyên.

- **V1.17 — Chọn ngôn ngữ DCAD:** theo STD-07 chỉ đọc dòng 1 của `DUser/Shared_data.Set` và lấy số trong `[]`. Đồng bộ STD-05/06/16, không dùng file cấu hình cũ hoặc giả định English. Chỉ cập nhật tài liệu, không build CAD.

- **Cập nhật 19/11/2026 — tên thành phẩm:** STD-03 v1.16 / STD-04 v1.16 / STD-16 v1.17 yêu cầu output `<ProjectName>.dll`, `.arx` hoặc `.grx` **không thêm `C22`/`C25`/`G27`**. Thư mục `Dprogram/AutoCAD2022`, `Dprogram/AutoCAD2025`, `Dprogram/GstarCAD2027` phân biệt binary trong ZIP; khi cài chọn đúng host đưa vào `DProgram` phẳng. Root Reference dùng ba thư mục `D:\000\Reference\Autocad 2022`, `D:\000\Reference\Autocad 2025`, `D:\000\Reference\GstarCAD 2027`. Chỉ cập nhật tài liệu online, không sửa code module; ZIP V1.17 không đổi.

- **Cập nhật trực tuyến 19/11/2026 — SQLite Runtime dùng chung:** STD-03 v1.17, STD-05 v1.15, STD-16 v1.18 quy định duy nhất `DProgram\sqlite3.dll` Windows x64, cài một lần cho các add-in có dùng SQLite; không cần cài SQLite lên ổ C/System32, không bắt `WinSQLite3.dll` lúc ARX LOAD. Các module nạp động bằng đường dẫn `DProgram` DIRECT, thiếu DLL chỉ dừng chức năng DB, không làm hỏng ARX; kiểm thử SQL01–SQL08. **Chỉ cập nhật tài liệu online; ZIP V1.17 chưa đổi.**

- **Cập nhật trực tuyến 19/11/2026 — Portable EXE trong DProgram:** STD-03 v1.18 / STD-05 v1.16 / STD-16 v1.19 cho phép giữ nguyên thư mục ứng dụng portable chính thức dưới `DProgram`, nạp EXE bằng `RelativePath` hoặc tìm tên EXE chính xác trong **thư mục con trực tiếp** khi được khai báo; không tìm sâu, không quét ổ và báo lỗi nếu có nhiều kết quả. SQLite runtime `DProgram\sqlite3.dll` không đổi. Bổ sung PORT01–PORT06; chỉ cập nhật tài liệu online, ZIP V1.17 giữ nguyên.
