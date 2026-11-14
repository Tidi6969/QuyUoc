# DCAD-STD-18 — Quy tắc AI lập trình theo 4 nguyên tắc Karpathy

**Mã quy ước:** 18  
**Phiên bản:** 1.1  
**Ngày cập nhật:** 14/11/2026  
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
- Đọc yêu cầu, baseline/source thực tế, quy ước áp dụng và luồng đang hoạt động trước khi sửa.
- Nêu mục tiêu, file/module cần tác động, những phần phải giữ nguyên, CAD target, dữ liệu/schema liên quan và rủi ro hồi quy.
- Khi có nhiều cách hiểu dẫn đến khác biệt quan trọng, nêu điểm khác biệt; không tự suy diễn nhu cầu, business logic hoặc lựa chọn cấu trúc dữ liệu.
- Nếu phương án đơn giản hơn đạt cùng mục tiêu, nêu lựa chọn đó cùng đánh đổi cần thiết.
- Khi thiếu nguồn, dữ liệu mâu thuẫn hoặc không thể xác minh baseline, báo rõ giới hạn và không tạo mã dựa trên giả định nguy hiểm.

**Đầu ra tối thiểu trước khi thực hiện:** mục tiêu, phạm vi, giới hạn, baseline và tiêu chí hoàn thành.

## 3. SIMPLICITY FIRST — Đơn giản trước tiên

**PHẢI:**
- Viết lượng mã, API, xử lý và cấu hình **vừa đủ** cho chức năng được giao.
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

**Với CAD:** ưu tiên bảo toàn Geometry, XData/XRecord, Settings/DUser, DCL/UI, đa ngôn ngữ, Support Path, quy trình build và tương thích AutoCAD 2022 / AutoCAD 2025 / GstarCAD 2027 theo phạm vi module được duyệt.

Nếu phát hiện lỗi ngoài phạm vi, báo hiện tượng và rủi ro; không âm thầm sửa lan sang tính năng khác.

## 5. GOAL-DRIVEN EXECUTION — Lập tiêu chí thành công, kiểm tra kết quả

**PHẢI:**
- Trước khi sửa, chuyển yêu cầu thành các tiêu chí PASS có thể đối chiếu: đầu vào, kết quả đúng, ca lỗi/biên, chức năng phải giữ nguyên, target liên quan.
- Với lỗi phần mềm, cố gắng tạo ca tái hiện trước khi sửa; với tính năng mới xác định ca thử; với refactor được duyệt đối chiếu hành vi trước–sau.
- Nếu công việc nhiều bước, lập kế hoạch ngắn dạng **bước → phép thử → điều kiện đạt**.
- Sau khi sửa, kiểm tra `diff`, chạy kiểm tra tĩnh và test thực tế có thể thực hiện, rồi build đúng từng CAD target yêu cầu.
- Nếu kiểm thử FAIL, truy nguyên trước khi sửa tiếp; không vá mù, không dùng workaround chưa được xác minh để che lỗi.
- Ghi trạng thái **PASS / FAIL / NotVerified** kèm bằng chứng và host. Build PASS không đồng nghĩa NETLOAD hoặc Runtime PASS.

**KHÔNG ĐƯỢC:** nói đã build, đã kiểm tra CAD, đã đồng bộ GitHub hoặc đã nghiệm thu khi chưa thực sự thực hiện/xác nhận.

## 5.1. CHUYỂN NGÔN NGỮ TRAO ĐỔI THÀNH NGÔN NGỮ GIAO DIỆN (BẮT BUỘC)

**PHẢI:**
- Phân biệt **ký hiệu/quy ước dùng để trao đổi thiết kế hoặc viết mã** với **nhãn, thông báo, hướng dẫn dành cho người sử dụng**. Không bê nguyên các từ viết tắt tạm thời (ví dụ C1/C2), tên biến, câu trao đổi, hay hướng dẫn dành cho AI vào giao diện, trừ khi đó là mã kỹ thuật chính thức người dùng cần xem.
- Viết thông báo theo ngôn ngữ tự nhiên, đúng nghiệp vụ, ngắn gọn và có hành động tiếp theo. Ví dụ: **“Vui lòng xác nhận thư mục nguồn trong Bộ phát hành.”**, không viết **“Trước tiên chọn thư mục nguồn C1/C2 trong Bộ phát hành.”**
- Phân biệt các tình trạng thực tế: **chưa chọn thư mục**, **đã chọn nhưng chưa có hệ thống/phiên bản**, **đã nhận diện nhưng thiếu dữ liệu/thư mục con**, **không đọc được thư mục**. Không dùng một thông báo cho nhiều nguyên nhân khác nhau và không yêu cầu chọn lại thứ đã chọn.
- Trước khi bàn giao, rà soát nhãn, tiêu đề, placeholder, tooltip, trạng thái, lỗi, thông báo xác nhận và văn bản xuất cho người dùng; bảo đảm thuật ngữ nhất quán, dễ hiểu, đúng quy chuẩn đa ngôn ngữ tại STD-07/13.
- Chỉ thay đổi những chuỗi và logic hiển thị trực tiếp liên quan đến yêu cầu; không tự đổi mã định danh, schema, dữ liệu hay thuật ngữ kỹ thuật trong hợp đồng máy-máy để “làm đẹp” giao diện.

**KIỂM CHỨNG:** thử tối thiểu luồng chưa có dữ liệu, đã thiết lập, dữ liệu thiếu và dữ liệu hợp lệ; kiểm tra thông báo đúng trạng thái. Với thay đổi giao diện, đối chiếu ảnh chụp hoặc kết quả UI thực tế khi có môi trường.

## 6. Quy trình 6 bước áp dụng cho AI

| Bước | Công việc | Cổng kiểm chứng |
|---|---|---|
| 1. Hiểu | Đọc đúng baseline, yêu cầu, quy ước và giới hạn | Không còn giả định trọng yếu chưa giải quyết |
| 2. Định mục tiêu | Viết tiêu chí đạt và bộ ca kiểm tra tối thiểu | Có expected/actual có thể so sánh |
| 3. Chọn phương án | Đề xuất cách đơn giản, ít đụng chạm nhất | Không thêm chức năng ngoài phạm vi |
| 4. Thực hiện | Chỉ sửa phần được giao sau khi được cho phép | Từng dòng diff có lý do |
| 5. Kiểm tra | Test hồi quy, build/runtime theo đúng host và phạm vi | PASS/FAIL/NotVerified có bằng chứng |
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
| AI-13 | Chuỗi giao diện dùng ngôn ngữ nghiệp vụ, không sao chép ký hiệu trao đổi; thông báo đúng trạng thái | Rà chuỗi và kiểm thử UI các trạng thái |

Kết quả sử dụng `PASS`, `FAIL`, `NotVerified` hoặc `N/A` theo điều kiện áp dụng, có lý do/bằng chứng. Việc đọc hoặc bổ sung quy ước này **không có nghĩa phần mềm đã đạt chuẩn**.

## 8. Quan hệ với bộ tiêu chuẩn

- **STD-01:** quy trình, quyền sửa mã, baseline và phiên bản.
- **STD-02–15:** hợp đồng kiến trúc, nghiệp vụ CAD và an toàn theo tính năng.
- **STD-16:** cập nhật ma trận QA áp dụng STD-18/AI-01–AI-12.
- **STD-17:** tự chọn baseline, tăng version phần mềm, quản lý ZIP/GitHub và ghi nhận nghiệm thu OK.

**Hiệu lực:** từ bản bộ quy ước **V1.13**. Chỉ bổ sung quy tắc làm việc với AI; không tự thay đổi code hoặc tăng version của add-in CAD.
