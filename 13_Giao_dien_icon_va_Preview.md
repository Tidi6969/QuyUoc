# DCAD-STD-13 — Giao diện, icon và Preview

**Phiên bản:** 1.5  
**Mức áp dụng:** Theo tính năng


## Giao diện

- Bố cục nhóm các trường theo nghiệp vụ; không nhồi nhiều chế độ không dùng. Label, tooltip, lỗi và thông báo phải theo quy chuẩn đa ngôn ngữ.
- Tách UI handler khỏi business logic, validation, đọc/ghi dữ liệu và vẽ CAD.
- Kiểm tra tỷ lệ DPI, chiều cao hàng/cột, vùng scroll, focus, tab order và khả năng sao chép text.
- ESC đóng/cancel form đúng cách, không làm mất dữ liệu đã lưu hoặc khiến lệnh CAD bên dưới hủy ngoài ý muốn.

## Icon, logo và tài nguyên

- Icon/logo nguồn nằm trong `icon` của project; nếu deploy ngoài DLL thì dùng thư mục logic `Dicon`, có khai báo trong DataManifest.
- Không dùng đường dẫn tuyệt đối tùy máy, không phụ thuộc file ảnh trong source tree khi chạy production.
- Màu sắc, kích thước icon và bộ nhận diện áp theo theme dự án đã phê duyệt; không ép đổi giao diện của module đang hoạt động ổn định chỉ để đồng bộ hình thức.

## Preview và DrawJig

- Preview dùng **cùng Model/Calculation** với đối tượng vẽ thật. Thông số, vị trí, layer, số bước lỗ và kích thước phải nhất quán.
- Khi có preview di chuyển theo chuột, dùng DrawJig: clone object tạm, biến đổi bằng `Matrix`, hiển thị qua `WorldDraw`, không sửa object thật.
- Chỉ commit geometry khi người dùng xác nhận; ESC/cancel phải hủy preview, dispose tài nguyên tạm, không để entity rác.
- Không dùng `SendStringToExecute MOVE` thay cho cơ chế preview realtime khi workflow cần bắt chuột mượt.
