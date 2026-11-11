# DCAD-STD-02 — Tổ chức mã nguồn, module và command

**Phiên bản:** 1.6  
**Mức áp dụng:** Bắt buộc

## Cấu trúc trách nhiệm

- Một project có **một command chính**; không thêm command Setup, Settings, Info hoặc Utility riêng nếu chưa được duyệt.
- Ranh giới trách nhiệm: `Command` → `Parser` → `Model` → `Validation/Calculation` → `Drawing` → `UI/Settings` → `Data/XData`.
- Command và Form không chứa thuật toán hình học phức tạp; Drawing chỉ nhận dữ liệu đã validate; code dữ liệu không phụ thuộc caption giao diện.
- Source chia theo module nghiệp vụ. Chỉ chia sẻ helper khi đầu vào, đầu ra, tolerance, fallback và vòng đời thay đổi thực sự giống nhau.
- Không hợp nhất logic Plate, Part, Punch, Hole chỉ vì class hoặc tên trường tương tự.

## Command và thay đổi tương thích

- Tên command ngắn, không trùng lệnh CAD phổ biến; đổi command hiện hành phải có yêu cầu và kế hoạch tương thích rõ.
- Nếu dùng hạ tầng `Lang`, đầu lệnh gọi `Lang.BeginCommand()` và `Lang.Reload()` để nạp lại ngôn ngữ hiện hành.
- Nếu code kế thừa Create hoặc Read từ module khác, chốt riêng baseline mẫu cho mỗi vai trò; không mặc định mọi module dùng chung một engine nghiệp vụ.
- Mọi API theo host CAD phải được kiểm tra bằng thư viện đúng phiên bản.

## Ranh giới mã dùng chung và tích hợp đa CAD — 11/11/2026

Mục tiêu: khi SDK/host/phiên bản thay đổi, chỉ sửa phần tích hợp thật sự khác; không sửa dây chuyền thuật toán, dữ liệu và giao diện đã nghiệm thu.

- A — Dùng chung: tính toán, Geometry plan không gắn Entity, validation, DAT parser, Settings, ngôn ngữ, schema XData/XRecord, trạng thái UI và Preview. Không include hoặc gọi SDK CAD.
- B — Chung quy tắc, riêng thao tác: Layer, Entity, tọa độ CAD, chọn đối tượng, ghi XData/XRecord, Undo/Transaction; phần chung quyết định dữ liệu, Adapter host thực thi.
- C — Riêng CAD và phiên bản: SDK/Reference, đăng ký lệnh, lifecycle, Palette docking, đọc Support Paths, quản lý tài liệu/database, build/link.
- Phân loại đến từng hàm/luồng. Không chuyển nguyên một hàm đang trộn nghiệp vụ và CAD sang Adapter. Không gộp Plate/Part/Punch/Hole chỉ vì tên giống nhau.
- UI Win32/MFC có thể dùng chung trên các CAD Windows; phần tạo/hiện/ghim Palette phải nằm sau ranh giới host.
- Chỉ tạo source, Adapter, thư mục và project cho host đang được giao. Các cây GstarCAD/ZWCAD/BricsCAD/OtherCAD trong sơ đồ chỉ là định hướng, không được tạo sẵn.
- Ưu tiên hàm giao tiếp nhỏ và rõ nghĩa thay vì framework chung phức tạp. Khi port phải kiểm tra đặc tính thực tế của host và dữ liệu, không mặc định API trùng tên sẽ tương đương.

Cổng kiểm: Core/Data/UI chung không có lời gọi SDK CAD; cùng dữ liệu trước/sau tách cho cùng kết quả Geometry, Layer, XData, Preview; adapter chứa khác biệt host; mọi thay đổi có diff và test theo STD-18. Không có host chạy thật = NotVerified.
