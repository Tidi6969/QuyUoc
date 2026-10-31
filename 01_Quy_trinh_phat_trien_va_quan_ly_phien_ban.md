# DCAD-STD-01 — Quy trình phát triển và quản lý phiên bản

**Phiên bản:** 1.9  
**Mức áp dụng:** Bắt buộc

## Mức độ áp dụng

- **BẮT BUỘC:** thực hiện với mọi project phù hợp phạm vi.
- **THEO TÍNH NĂNG:** chỉ áp dụng khi có nghiệp vụ tương ứng; không tạo chức năng giả để đáp ứng checklist.
- **KHUYẾN NGHỊ:** mặc định áp dụng, ngoại lệ phải được chấp thuận.
- **THAM CHIẾU:** ví dụ, schema chuyên ngành; không tự áp sang module khác.

## Quy trình thực hiện

1. Xác định phạm vi, mục tiêu CAD, nghiệp vụ và baseline đã được nghiệm thu.
2. Khảo sát code, dữ liệu, log và lỗi thực tế; không suy đoán nguyên nhân từ hiện tượng.
3. Xác định file/module cần sửa; đánh giá Geometry, XData, Settings, UI, ngôn ngữ, build và rủi ro hồi quy.
4. Đề xuất phương án và phản biện nguy cơ phá logic; **chỉ code sau khi được cho phép**.
5. Chỉ sửa phần có liên quan; không refactor, đổi command hoặc đổi schema ngoài phạm vi.
6. Build/test đúng từng CAD; ghi trung thực PASS, FAIL hoặc NotVerified.
7. Tăng version và bàn giao log thay đổi, KnownIssues, trạng thái nghiệm thu.

## Quản lý baseline và phiên bản

- Chỉ dùng baseline đã được người dùng duyệt; không lấy bản chưa kiểm chứng làm nguồn chính.
- Khi chuyển host, giữ business logic, sửa riêng framework/API đã có bằng chứng không tương thích.
- Các nhánh cùng lần giao có **cùng số version package**, nhưng trạng thái build/test là độc lập.
- Nếu lỗi lặp sau nhiều bản sửa: **HARD STOP**, truy nguyên nhân và đối chiếu baseline trước khi tiếp tục.

## Cổng kiểm tra tuân thủ trước khi bàn giao

1. Xác định **ma trận áp dụng** theo chức năng và CAD đích: yêu cầu nào là BẮT BUỘC, THEO TÍNH NĂNG, KHUYẾN NGHỊ hoặc không áp dụng; không đặt trạng thái `N/A` cho yêu cầu mặc định bắt buộc chỉ vì chưa triển khai.
2. Với mỗi quy chuẩn áp dụng, đối chiếu yêu cầu với **bằng chứng thực tế**: cấu hình/source, kết quả build, log, phép thử, ảnh chụp UI/console hoặc bản vẽ kiểm thử tùy tính chất. Không coi việc có tài liệu là bằng chứng code thực thi đúng.
3. Ghi `PASS`, `FAIL`, `NotVerified`, `N/A` theo từng hạng mục. `NotVerified` nghĩa là chưa có bằng chứng đủ; `N/A` chỉ khi tính năng hoặc target thực sự ngoài phạm vi và có lý do.
4. Phân biệt rõ: **BuildStatus** (compiler/DLL), **StandardsStatus** (mức tuân thủ) và **RuntimeStatus** (NETLOAD/thao tác CAD). Không cộng gộp các trạng thái để đưa ra thông báo PASS thiếu căn cứ.
5. Không công bố bản **phát hành đã nghiệm thu** nếu một yêu cầu bắt buộc còn `FAIL` hoặc `NotVerified`. Có thể giao bản **source/chờ kiểm thử** khi công khai rõ hạn chế và không nhận là hoàn thiện.
6. Mỗi yêu cầu bắt buộc phải có mã kiểm tra và tiêu chí đạt trong DCAD-STD-16; thiếu phương pháp kiểm chứng được xem là **thiếu quy chuẩn nghiệm thu**, cần hoàn thiện trước khi chốt phát hành.
