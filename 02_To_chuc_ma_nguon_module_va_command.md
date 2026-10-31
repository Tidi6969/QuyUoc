# DCAD-STD-02 — Tổ chức mã nguồn, module và command

**Phiên bản:** 1.5  
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
