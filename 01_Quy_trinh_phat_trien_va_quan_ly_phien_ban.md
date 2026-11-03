# DCAD-STD-01 — Quy trình phát triển và quản lý phiên bản

**Phiên bản:** 1.10  
**Cập nhật:** 03/11/2026  
**Mức áp dụng:** Bắt buộc

## Mức độ áp dụng

- **BẮT BUỘC:** thực hiện với mọi project phù hợp phạm vi.
- **THEO TÍNH NĂNG:** chỉ áp dụng khi có nghiệp vụ tương ứng; không tạo chức năng giả để đáp ứng checklist.
- **KHUYẾN NGHỊ:** mặc định áp dụng, ngoại lệ phải được chấp thuận.
- **THAM CHIẾU:** ví dụ, schema chuyên ngành; không tự áp sang module khác.

## Quy trình thực hiện

1. Xác định phạm vi, mục tiêu CAD, nghiệp vụ và baseline theo chỉ định hoặc phiên bản gần nhất theo STD-17; ghi riêng trạng thái nghiệm thu của baseline.
2. Khảo sát code, dữ liệu, log và lỗi thực tế; không suy đoán nguyên nhân từ hiện tượng.
3. Xác định file/module cần sửa; đánh giá Geometry, XData, Settings, UI, ngôn ngữ, build và rủi ro hồi quy.
4. Đề xuất phương án và phản biện nguy cơ phá logic; **chỉ code sau khi được cho phép**.
5. Chỉ sửa phần có liên quan; không refactor, đổi command hoặc đổi schema ngoài phạm vi.
6. Build/test đúng từng CAD; ghi trung thực PASS, FAIL hoặc NotVerified.
7. Tăng version và bàn giao log thay đổi, KnownIssues, trạng thái nghiệm thu.

## Quản lý baseline và phiên bản

- Ưu tiên baseline người dùng chỉ định hoặc nguồn cụ thể người dùng giao để sửa. Khi không chỉ định baseline, tự lấy **phiên bản gần nhất đã lưu của đúng module trên GitHub** theo số `V<n>` lớn nhất, đọc và xác minh nguồn; không tự lùi về bản cũ chỉ vì bản mới chưa nghiệm thu. Ghi đúng trạng thái chưa kiểm chứng, không biến việc chọn baseline thành xác nhận Build/Runtime PASS.
- Khi không chỉ định version đích cho lần sửa/tạo bản mới, lấy số phiên bản lớn nhất đã lưu của module rồi tăng **đúng 1**, ví dụ V49 → V50. Nếu cố ý sửa từ V47 nhưng kho đã có V49, bản mới mặc định vẫn là V50 để giữ lịch sử; chỉ định rõ của người dùng có ưu tiên nhưng không mặc nhiên cho phép ghi đè.
- Chỉ lưu trữ bản gốc, kiểm tra hoặc cập nhật metadata không tạo bản phần mềm mới. Khi upload chưa thành công, không coi số dự kiến là bản đã lưu; kiểm lại kho trước khi thử lại. Chi tiết chọn nguồn, xung đột version và lưu lịch sử ở STD-17.
- Khi chuyển host, giữ business logic của baseline đã chọn, sửa riêng framework/API đã có bằng chứng không tương thích.
- Các nhánh cùng lần giao có **cùng số version package**, nhưng trạng thái build/test là độc lập.
- Nếu lỗi lặp sau nhiều bản sửa: **HARD STOP**, truy nguyên nhân và đối chiếu baseline trước khi tiếp tục.

## Cổng kiểm tra tuân thủ trước khi bàn giao

1. Xác định **ma trận áp dụng** theo chức năng và CAD đích: yêu cầu nào là BẮT BUỘC, THEO TÍNH NĂNG, KHUYẾN NGHỊ hoặc không áp dụng; không đặt trạng thái `N/A` cho yêu cầu mặc định bắt buộc chỉ vì chưa triển khai.
2. Với mỗi quy chuẩn áp dụng, đối chiếu yêu cầu với **bằng chứng thực tế**: cấu hình/source, kết quả build, log, phép thử, ảnh chụp UI/console hoặc bản vẽ kiểm thử tùy tính chất. Không coi việc có tài liệu là bằng chứng code thực thi đúng.
3. Ghi `PASS`, `FAIL`, `NotVerified`, `N/A` theo từng hạng mục. `NotVerified` nghĩa là chưa có bằng chứng đủ; `N/A` chỉ khi tính năng hoặc target thực sự ngoài phạm vi và có lý do.
4. Phân biệt rõ: **BuildStatus** (compiler/DLL), **StandardsStatus** (mức tuân thủ) và **RuntimeStatus** (NETLOAD/thao tác CAD). Không cộng gộp các trạng thái để đưa ra thông báo PASS thiếu căn cứ.
5. Không công bố bản **phát hành đã nghiệm thu** nếu một yêu cầu bắt buộc còn `FAIL` hoặc `NotVerified`. Có thể giao bản **source/chờ kiểm thử** khi công khai rõ hạn chế và không nhận là hoàn thiện.
6. Mỗi yêu cầu bắt buộc phải có mã kiểm tra và tiêu chí đạt trong DCAD-STD-16; thiếu phương pháp kiểm chứng được xem là **thiếu quy chuẩn nghiệm thu**, cần hoàn thiện trước khi chốt phát hành.

- Trước khi bàn giao, ghi rõ baseline, phiên bản lớn nhất trước cập nhật, version đích, lý do chọn và trạng thái `mới nhất đã lưu` / `đã nghiệm thu` riêng biệt; giữ ZIP/changelog cũ, thêm phiên bản mới và đồng bộ repo module + DCAD theo STD-17.
