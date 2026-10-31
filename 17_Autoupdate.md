# DCAD-STD-17 — Autoupdate: Tự động chuẩn hóa, build, đóng gói và cập nhật GitHub

**Mã quy ước:** 17  
**Phiên bản:** 1.0  
**Mức áp dụng:** Bắt buộc khi người dùng yêu cầu tạo hoặc cập nhật phiên bản phần mềm trong kho GitHub.

## 1. Mục đích và thẩm quyền

Sau khi người dùng gửi mã nguồn mẫu (ZIP hoặc repository) và chỉ rõ phần mềm, bản chuẩn (baseline), phiên bản đích cùng các CAD mục tiêu, trợ lý thực hiện quy trình trọn gói trong **cùng phiên làm việc** nếu công cụ và môi trường cho phép; không buộc người dùng tải về, giải nén, sắp xếp thư mục và upload thủ công.

- Người dùng giao yêu cầu qua ChatGPT; GitHub là nơi lưu bản phiên bản và nhật ký thay đổi.
- Lệnh tạo phiên bản và tự commit/push là **ủy quyền thực hiện thao tác tương ứng** trong repository được chỉ định; không tự ý sửa mã nguồn hoặc phát hành phiên bản khác.
- Giữ nguyên baseline đã duyệt, không thay đổi các chức năng ngoài phạm vi và không ghi đè/xóa các phiên bản đã lưu.
- Nếu thiếu thông tin thiết yếu không thể suy ra an toàn (ví dụ repository đích, baseline chưa được xác định), phải báo thiếu thay vì tự chọn ngẫu nhiên.
- Không tuyên bố đang tiếp tục chạy nền hay hẹn cập nhật sau; chỉ báo kết quả đã thực hiện và xác minh trong phiên.

## 2. Câu lệnh yêu cầu chuẩn

> **Lấy bản [X] làm chuẩn, làm phiên bản [V…], build cho [các phiên bản CAD]. Đóng gói mã nguồn và các DLL thành một ZIP, tự commit và push lên GitHub. Xong chỉ gửi anh link GitHub dạng `/blob/<nhánh>/<đường dẫn ZIP>`. Nếu build hoặc upload bị lỗi thì báo rõ, không gửi bản chưa hoàn tất.**

Câu lệnh trên là mẫu điều khiển. Thay các trường trong dấu ngoặc bằng thông tin thực tế. Có thể bổ sung yêu cầu sửa chức năng trước khi đóng gói. **Không được coi mẫu câu lệnh là bằng chứng build/upload đã xong.**

## 3. Quy trình tự động khi nhận code mẫu

1. **Nhận:** đọc ZIP/repository nguồn; kiểm tra định dạng, các file có thể trích xuất, phiên bản, cấu trúc dự án và DLL/SDK thực tế. Không chạy script không đáng tin cậy chỉ vì có trong ZIP.
2. **Đối chiếu:** xác định baseline [X] đã duyệt, danh sách thay đổi, phiên bản đích [V…], repository, nhánh và CAD mục tiêu thuộc phạm vi đã phê duyệt; áp dụng các quy chuẩn 01–16 liên quan.
3. **Chuẩn hóa:** tái cấu trúc source, project, reference, tài nguyên, script build và log theo STD-03/04/16 mà không xóa dữ liệu nguồn cần thiết, không trộn SDK giữa các CAD và không đưa target chưa duyệt vào gói.
4. **Sửa theo yêu cầu:** chỉ thay đổi code trong phạm vi được người dùng cho phép; giữ nguyên logic còn lại.
5. **Build:** thực hiện Release build riêng cho từng CAD mục tiêu bằng môi trường/SDK phù hợp; kiểm tra mã thoát, DLL mới thực sự được tạo và log. Kết quả **PASS, FAIL, NotVerified** phải ghi chính xác cho từng host; không lấy build host này suy ra host khác.
6. **Kiểm thử:** rà cấu trúc, tên DLL, manifest, thay đổi và hồi quy. NETLOAD/smoke test trên CAD thực là một trạng thái riêng; không tự gắn PASS nếu chưa thử.
7. **Đóng gói:** tạo một file `<ProjectName>-<Version>.zip` chứa toàn bộ source cần thiết, project, script và Reference/SDK được phép phân phối, tài nguyên và các DLL **mới build thành công**. Kiểm lại ZIP sau khi xuất, không để DLL cũ/giả hoặc thiếu file thiết yếu.
8. **Changelog:** lập `Change log/<Version>.txt` với nội dung thay đổi, target, build/test status, vấn đề còn tồn tại và các hạn chế (nếu có).
9. **Upload:** khi cổng phát hành đạt yêu cầu, tự commit/push bằng GitHub connector hoặc công cụ được cấp quyền vào đúng repository và nhánh. Kiểm tra blob thực tế sau commit, kích thước/đường dẫn và tính toàn vẹn tệp khi có thể.
10. **Báo kết quả:** chỉ khi bước build, QA, đóng gói và upload bắt buộc đều thành công mới trả **duy nhất liên kết GitHub trực tiếp đến ZIP** theo dạng `https://github.com/<owner>/<repo>/blob/<branch>/<version-folder>/<project>-<version>.zip`.

## 4. Cấu trúc lưu trữ phiên bản trong repository

```text
<ProjectName>/
├── V<X>/
│   └── <ProjectName>-V<X>.zip
├── V<Version>/
│   └── <ProjectName>-<Version>.zip
└── Change log/
    ├── V<X>.txt
    └── <Version>.txt
```

Mỗi phiên bản là một snapshot độc lập; ví dụ `DBox/V5/DBox-V5.zip` hoặc `DPunch/V391/DPunch-V391.zip` trong repository tương ứng. Giữ tên phiên bản đúng theo người dùng (V4, V390, ...), không tự tăng nhảy số. **Bên trong ZIP** phải tuân theo STD-03/04; cấu trúc bên ngoài chỉ để lưu trữ lịch sử phiên bản.

## 5. Cổng phát hành bắt buộc — HARD STOP

- **Không upload bản được gọi là hoàn tất** nếu bất kỳ CAD target bắt buộc nào build FAIL hoặc NotVerified; không dùng DLL cũ để thay thế DLL thiếu.
- **Không upload** nếu ZIP thiếu source/Reference hợp lệ, sai cấu trúc, chưa kiểm chứng nội dung hoặc còn lỗi kiểm tra bắt buộc.
- **Không upload** nếu quá trình cần nghiệm thu runtime mà NETLOAD/smoke test vẫn chưa được xác nhận, trừ khi người dùng yêu cầu rõ một bản `source/chờ kiểm thử` tách biệt và có trạng thái minh bạch; bản đó không được gọi là bản hoàn chỉnh.
- Khi môi trường không có Windows, CAD SDK hoặc công cụ build tương thích, ghi `BuildStatus=NotVerified` và báo trở ngại; **không giả nhận đã build**.
- Nếu thiếu quyền GitHub, connector không thể upload binary, giới hạn dung lượng hoặc lỗi commit/push, báo lỗi thực tế; không nói đã upload khi chưa kiểm chứng.
- Không gửi đường dẫn `/blob/...` giả hoặc dẫn đến file chưa tồn tại; nếu thất bại, phản hồi ngắn gồm bước lỗi, nguyên nhân đã xác minh và phần chưa hoàn tất.
- Tránh ghi đè phiên bản cũ. Nếu đích đã tồn tại, đối chiếu nội dung và dừng nếu việc thay thế chưa được duyệt.
- Quy ước này **không thay thế** các điều kiện phát hành/QA của STD-01, 03, 04, 15, 16; khi xung đột, áp dụng yêu cầu kiểm chứng nghiêm ngặt hơn.

## 6. Quy ước phản hồi

- **Thành công hoàn toàn:** chỉ gửi một URL dạng `https://github.com/<owner>/<repo>/blob/<nhánh>/<đường dẫn ZIP>` và không kèm diễn giải.
- **Thất bại hoặc chưa thể kiểm chứng:** nói rõ `FAIL` hoặc `NotVerified`, bước bị chặn, lý do và hành động cần thiết; **không** gửi đường dẫn ZIP hay tự công bố phát hành hoàn tất.
- **Yêu cầu chỉ lưu trữ bản nguồn gốc, không yêu cầu build:** được tổ chức và upload bản snapshot nguồn nếu người dùng nói rõ đây là bản lưu trữ (không gọi là bản đã build). Báo theo phạm vi công việc thực hiện, không gán nhãn phát hành đã build.

## 7. Tiêu chí nghiệm thu STD-17

| Mã | Kiểm chứng | Kết quả |
|---|---|---|
| AUTO-01 | Đúng repository, baseline, version, target được duyệt | PASS/FAIL/NotVerified |
| AUTO-02 | Snapshot có source, project, Reference hợp lệ, tài nguyên và cấu trúc STD-03 | PASS/FAIL/NotVerified |
| AUTO-03 | Build Release riêng cho từng CAD và DLL mới, log/mã thoát | PASS/FAIL/NotVerified |
| AUTO-04 | QA, NETLOAD/runtime theo phạm vi yêu cầu và STD-16 | PASS/FAIL/NotVerified |
| AUTO-05 | ZIP mở lại được, đúng nội dung và checksum nếu có | PASS/FAIL/NotVerified |
| AUTO-06 | GitHub commit/push thành công; đọc lại đường dẫn và blob | PASS/FAIL/NotVerified |
| AUTO-07 | Khi PASS, phản hồi chỉ link `/blob/...`; khi lỗi, báo chặn rõ ràng | PASS/FAIL/NotVerified |
