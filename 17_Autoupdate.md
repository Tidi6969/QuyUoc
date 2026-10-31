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

## 8. Đồng bộ bắt buộc với repository DCAD tổng hợp

**Kho tổng hợp chính:** `https://github.com/Tidi6969/DCAD` (nhánh `main`, trừ khi người dùng chỉ định khác). DCAD là **danh mục trạng thái của toàn bộ add-in**, không phải chỗ lưu bản sao ZIP/DLL/source của từng add-in.

- Khi người dùng yêu cầu phát hành hoặc lưu trữ một phiên bản module lên GitHub, yêu cầu ấy **đồng thời cho phép** cập nhật **thông tin metadata tương ứng** trong `Tidi6969/DCAD`, không cần xin xác nhận thứ hai. Không được nhân đó thay đổi quy chuẩn, source hay cấu hình runtime DCAD ngoài danh mục/thông tin phát hành.
- Trình tự: (1) xác minh bản ZIP/changelog trong repository module, (2) xác minh trạng thái build/QA/running thực tế, (3) cập nhật mục module trong `DCAD/README.md` hoặc manifest danh mục nếu có, (4) đọc lại cả hai repository và xác nhận khớp. **Chỉ đánh dấu "Hoàn tất đồng bộ" sau bước (4).**
- Mục module tối thiểu ghi: **Tên add-in, phiên bản mới nhất đã lưu, loại bản (snapshot nguồn / đã build / đã kiểm thử CAD), trạng thái Build/Runtime đúng bằng chứng, CAD target, ngày đồng bộ và đường dẫn ZIP dạng `/blob/<branch>/<path>`**, cùng link repository/changelog khi phù hợp.
- Trạng thái `Đã lưu trên GitHub` **khác** `Build PASS` và `CAD runtime PASS`; không suy diễn kết quả kiểm thử từ việc có ZIP hoặc DLL. Một phiên bản chưa được nghiệm thu có thể được ghi vào danh mục với nhãn `Chờ build/kiểm thử`, nhưng **không** được quảng bá là bản phát hành hoàn tất.
- Nếu upload module thành công nhưng cập nhật DCAD thất bại: báo rõ **module đã có trên GitHub, DCAD chưa đồng bộ**; không nói hoàn thành toàn bộ, không xóa bản đã upload và không ghi nhận sai trạng thái.
- Nếu phát hành module thất bại: không tăng phiên bản "mới nhất đã hoàn tất" trong DCAD; được ghi trạng thái lỗi đang xử lý khi cần và có bằng chứng, nhưng không được thay bản đã chạy ổn định.
- Nếu người dùng đang sửa và thử nghiệm một bản chưa nghiệm thu: có thể ghi phiên bản `đang thử nghiệm` bên cạnh `phiên bản đã nghiệm thu` khi thực sự có bản đã được lưu; **không** tự khẳng định người dùng đã test chạy CAD.
- Sau khi đồng bộ thành công, liên kết trả về theo mẫu yêu cầu vẫn là **một link ZIP tại repository module**; khi lỗi thì nêu rõ bước thất bại theo mục 6.

### Vòng lặp làm việc với người dùng

**ChatGPT nhận yêu cầu → sửa/kiểm tra/build (nếu có môi trường) → lưu phiên bản module lên GitHub → đồng bộ trạng thái DCAD → trả link ZIP → người dùng tải về chạy thử CAD → người dùng phản hồi lỗi/kết quả tại ChatGPT → cập nhật trạng thái và sửa phiên bản tiếp theo khi có yêu cầu.**

Không chờ một thao tác GitHub thủ công của người dùng ở các bước mà công cụ thực sự thực hiện được; không tự tuyên bố việc tải về chạy thử của người dùng đã xảy ra. Mọi lần cập nhật diễn ra trong phiên được yêu cầu, không tự chạy ngầm giữa các cuộc trò chuyện.

| Mã | Kiểm chứng bổ sung | Kết quả |
|---|---|---|
| AUTO-08 | Danh mục `Tidi6969/DCAD` phản ánh đúng phiên bản, URL, build/runtime status và CAD target | PASS/FAIL/NotVerified |
| AUTO-09 | Đọc lại repository module và DCAD sau commit, xác nhận không lệch trạng thái | PASS/FAIL/NotVerified |
