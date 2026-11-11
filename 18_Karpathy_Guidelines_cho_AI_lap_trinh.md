# DCAD-STD-18 — Quy tắc AI lập trình theo 4 nguyên tắc Karpathy

**Mã quy ước:** 18  
**Phiên bản:** 1.1  
**Ngày cập nhật:** 11/11/2026  
**Mức áp dụng:** BẮT BUỘC với mọi yêu cầu AI phân tích, viết, sửa, gỡ lỗi hoặc rà soát mã nguồn của DCAD.  
**Nguồn tham khảo:** https://github.com/multica-ai/andrej-karpathy-skills (MIT), dự án diễn giải các nhận xét của Andrej Karpathy về lỗi thường gặp của AI khi viết code. Nội dung dưới đây được điều chỉnh cho hệ thống DCAD, không sao chép toàn bộ tài liệu gốc.

## 1. Phạm vi và thứ tự ưu tiên

1. Mục tiêu là **đúng yêu cầu, ít thay đổi nhất, có chứng cứ kiểm chứng**.
2. Quy ước 18 **không tự cấp quyền viết/sửa code**; tuân theo STD-01: khảo sát trước, chỉ thực hiện thay đổi khi người dùng giao hoặc cho phép.
3. Các yêu cầu chuyên ngành về Geometry, XData/XRecord, DCL/UI, Settings, command, Build, CAD target, QA, phát hành và GitHub tại STD-01–17 vẫn có hiệu lực; STD-18 không thay thế hay giảm yêu cầu kiểm chứng của chúng.
4. Tuân thủ yêu cầu cụ thể đã được xác định. Nếu thiếu dữ liệu thiết yếu làm thay đổi kết quả, làm rõ hoặc báo rõ điều chưa chắc chắn; không hỏi lại thông tin đã có hoặc baseline/version mặc định theo STD-17.
5. Phân biệt **dữ kiện đã đọc, giả định, đề xuất, kết quả đã thử và kết quả chưa thử**. Không biến dự đoán thành PASS.

## 2. THINK BEFORE CODING — Suy nghĩ trước khi viết mã

**PHẢI:**
- Trước **mọi lần viết, sửa, refactor hoặc gỡ lỗi mã nguồn DCAD**, bắt buộc đọc **STD-02 phiên bản hiện hành**, đọc yêu cầu, baseline/source thực tế, quy ước áp dụng và luồng đang hoạt động. Không chỉ đọc STD-02 khi port CAD.
- **Bắt buộc lập bảng phân loại cho các file/hàm/luồng dự định thay đổi trước khi code**, theo STD-02: **A** = nghiệp vụ/dữ liệu/UI dùng chung, không phụ thuộc CAD SDK; **B** = quy tắc chung nhưng thao tác CAD cần Adapter; **C** = SDK/host/phiên bản riêng. Với thay đổi không tác động host ghi rõ A hoặc N/A và lý do, không tự phát sinh Adapter.
- Nêu mục tiêu, file/module cần tác động, những phần phải giữ nguyên, CAD target, dữ liệu/schema liên quan và rủi ro hồi quy.
- Khi có nhiều cách hiểu dẫn đến khác biệt quan trọng, nêu điểm khác biệt; không tự suy diễn nhu cầu, business logic hoặc lựa chọn cấu trúc dữ liệu.
- Nếu phương án đơn giản hơn đạt cùng mục tiêu, nêu lựa chọn đó cùng đánh đổi cần thiết.
- Khi thiếu nguồn, dữ liệu mâu thuẫn hoặc không thể xác minh baseline, báo rõ giới hạn và không tạo mã dựa trên giả định nguy hiểm.

**Đầu ra tối thiểu trước khi thực hiện:** mục tiêu, phạm vi, giới hạn, baseline, tiêu chí hoàn thành **và bảng phân loại A/B/C cho phần mã chịu ảnh hưởng**. Bảng tối thiểu gồm: `File/hàm hoặc luồng | Nhóm A/B/C | Lý do | SDK/host bị ảnh hưởng | Phần giữ nguyên | Kiểm thử dự kiến`. Có thể gộp các hàm đồng loại khi đủ rõ, nhưng không được bỏ qua phân loại. Chỉ đặt mã vào phần dùng chung nếu thực sự độc lập SDK; với logic trộn A/B/C phải phân ranh giới trước khi sửa.

## 3. SIMPLICITY FIRST — Đơn giản trước tiên

**PHẢI:**
- Viết lượng mã, API, xử lý và cấu hình **vừa đủ** cho chức năng được giao.
- Thiết kế phần dùng chung để có thể mở rộng nhiều CAD về sau, **nhưng chỉ tạo file, Adapter, cây thư mục, project và build cho target đã được người dùng yêu cầu**; không dựng sẵn host hoặc abstraction không có nhu cầu thực.
- Không tự thêm chức năng, hệ thống cấu hình, khả năng mở rộng, wrapper/framework hoặc abstraction cho tình huống chỉ dùng một lần khi không cần thiết.
- Giữ cách tổ chức, tên lệnh, hợp đồng dữ liệu, coding style và patterns đã có nếu vẫn phù hợp.
- Chọn cách rõ ràng, dễ kiểm tra và bảo trì; loại bỏ sự phức tạp do chính thay đổi tạo ra.

**KHÔNG ĐƯỢC:** đơn giản hóa bằng cách bỏ kiểm tra đầu vào cần thiết, transaction/Undo, xử lý lỗi có ý nghĩa, ràng buộc bảo toàn dữ liệu hoặc QA bắt buộc theo các quy ước khác.

**Kiểm chứng:** mọi thành phần mới phải liên hệ trực tiếp với một yêu cầu được giao.

## 4. SURGICAL CHANGES — Sửa đúng vị trí và ít nhất có thể

**PHẢI:**
- Chỉ sửa dòng, hàm, file và phụ thuộc trực tiếp cần thiết để đạt yêu cầu.
- Giữ nguyên các tính năng ngoài phạm vi; không tự đổi comment, format hàng loạt, tên command, schema, cấu trúc thư mục, layer hay thuật toán không liên quan.
- Tuân theo phong cách đang sử dụng thay vì tự áp đặt style mới.
- Chỉ xóa import, biến hoặc hàm trở thành không dùng **do chính thay đổi này gây ra**; code cũ đã tồn tại chưa liên quan thì ghi nhận, không tự xóa.
- Rà `diff` trước bàn giao: **từng thay đổi phải truy vết đến yêu cầu, phụ thuộc trực tiếp hoặc kiểm chứng bắt buộc**.
- Mọi lời gọi CAD SDK mới phải nằm đúng ranh giới C hoặc lớp thực thi B; không đưa API host vào Core/Data/UI dùng chung. Nếu buộc thay đổi hợp đồng chung, nêu rõ bằng chứng và rủi ro hồi quy.

**Với CAD:** ưu tiên bảo toàn Geometry, XData/XRecord, Settings/DUser, DCL/UI, đa ngôn ngữ, Support Path, quy trình build và tương thích AutoCAD 2022 / AutoCAD 2025 / GstarCAD 2027 theo phạm vi module được duyệt.

Nếu phát hiện lỗi ngoài phạm vi, báo hiện tượng và rủi ro; không âm thầm sửa lan sang tính năng khác.

## 5. GOAL-DRIVEN EXECUTION — Lập tiêu chí thành công, kiểm tra kết quả

**PHẢI:**
- Trước khi sửa, chuyển yêu cầu thành các tiêu chí PASS có thể đối chiếu: đầu vào, kết quả đúng, ca lỗi/biên, chức năng phải giữ nguyên, target liên quan.
- Với lỗi phần mềm, cố gắng tạo ca tái hiện trước khi sửa; với tính năng mới xác định ca thử; với refactor được duyệt đối chiếu hành vi trước–sau.
- Nếu công việc nhiều bước, lập kế hoạch ngắn dạng **bước → phép thử → điều kiện đạt**.
- Sau khi sửa, kiểm tra `diff` **đối chiếu bảng phân loại A/B/C**, kiểm tra tĩnh và test thực tế có thể thực hiện, rồi build đúng từng CAD target yêu cầu. Khi thay đổi kiến trúc/tách SDK, áp dụng thêm HOST-01–HOST-08 của STD-16; không coi kiểm tra tĩnh là Runtime PASS.
- Nếu kiểm thử FAIL, truy nguyên trước khi sửa tiếp; không vá mù, không dùng workaround chưa được xác minh để che lỗi.
- Ghi trạng thái **PASS / FAIL / NotVerified** kèm bằng chứng và host. Build PASS không đồng nghĩa NETLOAD hoặc Runtime PASS.

**KHÔNG ĐƯỢC:** nói đã build, đã kiểm tra CAD, đã đồng bộ GitHub hoặc đã nghiệm thu khi chưa thực sự thực hiện/xác nhận.

## 6. Quy trình 6 bước áp dụng cho AI

| Bước | Công việc | Cổng kiểm chứng |
|---|---|---|
| 1. Hiểu | Đọc đúng baseline, **STD-02**, yêu cầu, quy ước và giới hạn; phân loại file/hàm A/B/C | Có bảng phân loại và không còn giả định trọng yếu chưa giải quyết |
| 2. Định mục tiêu | Viết tiêu chí đạt và bộ ca kiểm tra tối thiểu | Có expected/actual có thể so sánh |
| 3. Chọn phương án | Đề xuất cách đơn giản, ít đụng chạm nhất | Không thêm chức năng ngoài phạm vi |
| 4. Thực hiện | Chỉ sửa phần được giao sau khi được cho phép | Từng dòng diff có lý do |
| 5. Kiểm tra | Đối chiếu bảng A/B/C và diff, test hồi quy, build/runtime theo đúng host và phạm vi | PASS/FAIL/NotVerified có bằng chứng |
| 6. Bàn giao | Mô tả phần đã đổi, đã giữ nguyên và chưa xác minh | Tuân thủ STD-01/16/17 |

**HARD STOP:** không xác định được baseline thiết yếu; phát hiện nguy cơ ghi đè bản cũ; khác schema/host; bug lặp chưa truy được nguyên nhân; hoặc cổng kiểm thử bắt buộc FAIL. Xử lý theo STD-01/15/16/17.

## 7. Ma trận nghiệm thu riêng STD-18

| Mã | Tiêu chí bắt buộc khi áp dụng | Bằng chứng |
|---|---|---|
| AI-01 | Đúng yêu cầu, baseline và CAD target | Version nguồn, phạm vi |
| AI-02 | Nêu rõ mục tiêu và những phần giữ nguyên | Scope và impact |
| AI-03 | Điểm không rõ quan trọng không bị AI tự đoán | Danh sách giả định/đối chiếu |
| AI-04 | Không tự thêm chức năng/API/config chưa được giao | Đối chiếu yêu cầu và diff |
| AI-05 | Không tự refactor, đổi format/comment ngoài phạm vi | Diff trước–sau |
| AI-06 | Từng thay đổi có lý do truy vết trực tiếp | Diff theo file/hàm |
| AI-07 | Tiêu chí đạt đo được và test case có expected/actual | Ca thử, kết quả |
| AI-08 | Với bug có tái hiện/hồi quy khi khả thi | Test hoặc lý do NotVerified |
| AI-09 | Build, NETLOAD, Runtime và từng CAD target độc lập | Log/bằng chứng riêng |
| AI-10 | Không sửa code/phát hành vượt quyền được giao | Lệnh người dùng, thay đổi thực |
| AI-11 | Bảo toàn schema, dữ liệu, UI và hợp đồng liên quan | Ma trận ảnh hưởng STD-01/16 |
| AI-12 | Minh bạch lỗi, giới hạn, tình trạng chưa kiểm chứng | PASS/FAIL/NotVerified và ghi chú |
| AI-13 | Trước khi code đã đọc STD-02 hiện hành và phân loại file/hàm/luồng dự định sửa thành A/B/C hoặc N/A có lý do | Bảng phân loại và ranh giới SDK |
| AI-14 | Mã chung không phụ thuộc SDK và không tạo Adapter/target chưa được yêu cầu; các thay đổi host có kiểm tra STD-16 HOST-01–HOST-08 | Static scan, diff, ZIP tree, QA theo host |

Kết quả sử dụng `PASS`, `FAIL`, `NotVerified` hoặc `N/A` theo điều kiện áp dụng, có lý do/bằng chứng. Việc đọc hoặc bổ sung quy ước này **không có nghĩa phần mềm đã đạt chuẩn**.

## 8. Quan hệ với bộ tiêu chuẩn

- **STD-01:** quy trình, quyền sửa mã, baseline và phiên bản.
- **STD-02:** **bắt buộc đọc trước mọi lần code** để phân loại A/B/C và xác định ranh giới SDK/host; không chỉ áp dụng khi thêm CAD mới.
- **STD-03–15:** cấu trúc triển khai, build, dữ liệu, nghiệp vụ CAD và an toàn theo tính năng.
- **STD-16:** kiểm tra AI-01–AI-14 và HOST-01–HOST-08 theo phạm vi tác động.
- **STD-17:** tự chọn baseline, tăng version phần mềm, quản lý ZIP/GitHub và ghi nhận nghiệm thu OK.

**Hiệu lực:** STD-18 áp dụng từ bộ quy ước V1.13; bổ sung v1.1 ngày 11/11/2026 có hiệu lực với các yêu cầu lập trình tiếp theo. Việc sửa quy ước không tự thay đổi code hoặc tăng version của add-in CAD.
