# DDLC – Quy ước tạm thời V0.1

- Ngày lập: 2026-11-24
- Trạng thái: DRAFT – phương án đã thống nhất để nghiên cứu, triển khai từng bước; chưa phải tiêu chuẩn chính thức.
- Phạm vi: hệ thống phát triển phần mềm DCAD và dự án xây dựng DDLC.
- Không thay thế, sửa đổi hoặc vô hiệu hóa quy ước DCAD hiện hành (đặc biệt quy ước 18). Nếu có mâu thuẫn, ghi nhận và trình người quản lý quyết định trước khi áp dụng.
- Không bắt đầu lập trình DDLCManager trước khi đặc tả nền tảng được duyệt. Chính DDLC là dự án đầu tiên áp dụng phương pháp DDLC.

## 1. Mục tiêu và phương pháp

DDLC (DCAD Development Life Cycle) là tên nội bộ cho hệ thống quản lý, điều phối và kiểm soát phát triển phần mềm bằng AI. Con người quyết định mục tiêu, quy tắc, phạm vi và nghiệm thu; AI thực thi trong quyền hạn; công cụ kiểm chứng kết quả. Không chỉ là bộ prompt hay lịch sử chat.

- SDLC: khung vòng đời; ADLC (Agentic Development Life Cycle): AI thực hiện quy trình, không coi đây là tiêu chuẩn quốc tế thống nhất.
- SDD: phát triển theo đặc tả; DDD: thiết kế theo nghiệp vụ khi phù hợp; TDD: kiểm thử trước triển khai theo chu trình red-green-refactor; BDD/ATDD: hành vi và tiêu chí nghiệm thu.
- SCM, RCA, Impact Analysis, Design by Contract, SOLID, DRY, KISS, YAGNI, Clean Code, Karpathy Guidelines; ADR cho quyết định kiến trúc, RTM cho truy xuất yêu cầu.
- CI/CD, DevSecOps, SAST/DAST/SCA khi phù hợp; UAT, HITL, Policy as Code.
- Vibe Coding chỉ dùng thử nghiệm ý tưởng trong môi trường tách biệt; không tự động đưa prototype vào release.
- Chọn phương pháp theo tính chất/rủi ro, không áp dụng hình thức mọi kỹ thuật cho mọi nhiệm vụ.

## 2. Kiến trúc dự kiến (chưa chốt triển khai)

| Thành phần | Trách nhiệm |
|---|---|
| DDLC Manager | Giao diện dự án, yêu cầu, phiên bản, quy ước, nhiệm vụ, trạng thái, phê duyệt, kiểm thử và phát hành |
| DDLC Rules | Quy ước có phiên bản, phạm vi, thứ tự ưu tiên và lịch sử |
| DDLC Database | Metadata, liên kết, trạng thái, bằng chứng và audit |
| DDLC Engine | State Machine, điều phối nhiệm vụ, Quality Gate, xử lý thất bại |
| DDLC Agents | Kết nối AI, cấp quyền và giao việc có cấu trúc |
| DDLC Validator | Kiểm tra quy ước, phạm vi, mã nguồn, build, test |
| DDLC Release | Quản lý thành phẩm, phát hành, truy xuất và rollback |

Định hướng: DDLCManager.exe trên Windows (C#/WPF), SQLite cục bộ, Git/GitHub quản lý mã nguồn và quy ước, Visual Studio/SDK và CAD thực tế để build/test, NAS lưu backup nhất quán và release. Không mở SQLite trực tiếp qua NAS. Cô lập môi trường build/agent, không cấp quyền tùy ý tới bí mật, khóa ký, NAS hoặc hệ thống. Không đưa bí mật vào repo hay log. Nghiên cứu GitHub Spec Kit làm tham khảo SDD; Codex CLI và adapter AI khác; GitHub Actions/self-hosted runner, PowerShell, CTest/GoogleTest, Cppcheck/Clang-Tidy khi phù hợp. Các lựa chọn công nghệ phải được đánh giá và phê duyệt trước khi triển khai.

Cấu trúc lưu trữ dự kiến: Manager/, Database/DDLC.db, Rules/History/, Engine/Workflows/, Engine/Gates/, Engine/Scripts/, Agents/, Templates/, Projects/, Workspace/, Build/, Releases/, Logs/, Backup/. Đây là cấu trúc nội bộ DDLC, không thay đổi quy ước đóng gói DCAD.

Dữ liệu dự kiến: Projects, Versions, Requirements, Specifications, Designs, Rules, RuleVersions, Tasks, Agents, AgentRuns, Changes, TestCases, TestResults, Builds, Approvals, Releases, AuditLogs. Git là nguồn mã nguồn; DB lưu metadata, ID, phiên bản và liên kết bằng chứng, không sao chép toàn bộ source. Thiết kế schema và migration ở giai đoạn tương ứng.

## 3. Vòng đời 12 giai đoạn

Mỗi giai đoạn có đầu vào đã xác định, danh sách công việc, đầu ra và điều kiện chuyển bước. Chi tiết thủ tục sẽ được chuẩn hóa trong DDLC-03/04.

### 01. Governance – Quản lý quy ước
- Đọc toàn bộ quy ước; xác định phiên bản, quy tắc bắt buộc/có điều kiện, phạm vi áp dụng; kiểm tra xung đột, thứ tự ưu tiên; xác định quyền hạn, hành động cấm; khóa phiên bản quy ước cho nhiệm vụ.
- Đầu ra: bộ quy tắc áp dụng có phiên bản.
- Gate: không còn xung đột chưa giải quyết.

### 02. Baseline – Nền tảng mã nguồn
- Xác định phiên bản gốc đã được xác nhận; kiểm tra lịch sử, chức năng hiện có, lỗi đã biết, cấu trúc, dependency, môi trường build; tạo workspace độc lập; ghi commit và hash/tính toàn vẹn.
- Đầu ra: baseline và thông tin môi trường.
- Gate: nguồn gốc rõ ràng, không sử dụng nhầm bản hoặc tự lấy bản lỗi làm nền.

### 03. Requirement – Yêu cầu
- Xác định mục đích, loại công việc (mới/cải tiến/sửa lỗi), yêu cầu chức năng và phi chức năng, phạm vi và phần không được đổi, mâu thuẫn, ngoại lệ, ưu tiên, tiêu chí nghiệm thu.
- Đầu ra: Requirement Specification.
- Gate: đầy đủ, có thể kiểm chứng, được phê duyệt.

### 04. Analysis – Phân tích kỹ thuật
- Đọc source, luồng thực thi, dữ liệu, quan hệ module; tái hiện lỗi khi khả thi, tìm nguyên nhân gốc, phân tích chức năng liên quan và phạm vi ảnh hưởng, rủi ro, giải pháp thay thế, tương thích.
- Đầu ra: Analysis Report/Impact Analysis.
- Gate: đủ căn cứ chọn giải pháp; ghi rõ giả định và điểm chưa xác minh.

### 05. Specification – Đặc tả kỹ thuật
- Xác định đầu vào/đầu ra, kiểu dữ liệu, công thức, logic, giới hạn, mặc định, điều kiện hợp lệ/không hợp lệ, thao tác, xử lý lỗi, trạng thái, ví dụ kết quả mong muốn.
- Đầu ra: Technical Specification.
- Gate: logic rõ, kiểm thử được, đã thống nhất.

### 06. Design – Thiết kế
- Kiến trúc, module và trách nhiệm, class/interface, thuật toán, cấu trúc dữ liệu, DB, UI và luồng thao tác, ngoại lệ, tương thích đa phiên bản/đa ngôn ngữ, bảo mật, hiệu năng; ADR khi có quyết định quan trọng.
- Đầu ra: Software Design Document.
- Gate: đáp ứng đặc tả, không phức tạp hóa không cần thiết, phê duyệt theo hồ sơ rủi ro.

### 07. Test Design – Thiết kế kiểm thử
- Xây dựng test case và expected result trước code; trường hợp thường, biên, invalid, ngoại lệ, chức năng cũ, tương thích; chuẩn bị fixture, DWG/DXF mẫu, phương pháp đo và dung sai hình học khi cần; phân loại tự động/thủ công; liên kết với yêu cầu.
- Đầu ra: Test Plan, Test Cases, fixtures và RTM.
- Gate: mọi yêu cầu quan trọng có tiêu chí kiểm thử tương ứng.

### 08. Implementation – Lập trình
- Phân chia task, thứ tự phụ thuộc, file được/không được sửa; thực hiện đúng thiết kế và quy ước 18; tái sử dụng phù hợp, tránh chắp vá và thay đổi thừa; kiểm tra tên, quản lý bộ nhớ, lỗi; TDD khi khả thi, kiểm thử thành phần, ghi nhận thay đổi, cập nhật tài liệu liên quan.
- Đầu ra: source mới, commit/diff, kết quả kiểm thử thành phần.
- Gate: nhiệm vụ đủ điều kiện review, không tự mở rộng phạm vi.

### 09. Review – Rà soát
- Reviewer độc lập với phiên thực hiện; so sánh baseline và candidate; kiểm tra từng thay đổi, scope, spec, quy ước, thuật toán, trùng lặp/dead code trong phạm vi, lỗi tiềm ẩn, bộ nhớ, bảo mật, khả năng bảo trì; static analysis khi phù hợp.
- Đầu ra: Code Review Report và diff.
- Gate: không còn lỗi nghiêm trọng hoặc vi phạm bắt buộc chưa xử lý.

### 10. Build & Testing – Biên dịch và kiểm thử
- Kiểm tra môi trường, SDK, dependency; build sạch từ source xác định; kiểm tra lỗi/cảnh báo và artifact; unit, integration, functional, regression, performance, compatibility, security test theo phạm vi/rủi ro; lưu log, lệnh, môi trường và kết quả thực tế.
- Đầu ra: Build Report, Test Report, artifacts gắn với commit/hash.
- Gate: build thành công, kiểm thử bắt buộc PASS. Không có môi trường hoặc chưa chạy thì NOT_RUN/BLOCKED, không đánh PASS.

### 11. Acceptance – Nghiệm thu
- Cài đặt trên môi trường thực tế; kiểm tra thao tác, kết quả hình học, giao diện, tốc độ, ổn định, tương thích dữ liệu cũ và toàn bộ acceptance criteria; ghi vấn đề; người có quyền duyệt hoặc từ chối đúng artifact.
- Đầu ra: Acceptance Report/Approval.
- Gate: nghiệm thu đạt và được người có quyền xác nhận.

### 12. Release & Maintenance – Phát hành và bảo trì
- Số phiên bản, tên file, cấu trúc gói, dependency, bản quyền, ký số nếu cần; changelog, package, source tương ứng, hash; GitHub, backup NAS, lịch sử phiên bản; theo dõi lỗi sau phát hành, phương án rollback và tạo yêu cầu sửa tiếp theo.
- Đầu ra: release chính thức có thể truy xuất.
- Gate: chỉ phát hành artifact đã nghiệm thu, có thể kiểm chứng và khôi phục.

## 4. Phân cấp và cấu trúc nhiệm vụ

Cấp 0 Constitution (nguyên tắc tối cao); cấp 1 Lifecycle (12 giai đoạn); cấp 2 Process; cấp 3 Procedure; cấp 4 Task; cấp 5 Evidence.

Mỗi task có: ID/tên; giai đoạn/quy trình; vai trò thực hiện; bắt buộc hay điều kiện áp dụng; dependency; đầu vào và phiên bản; quyền truy cập/sửa; công cụ; bước thực hiện; đầu ra; bằng chứng; điều kiện PASS/FAIL; cách xử lý lỗi và quay lại; quyền phê duyệt/miễn trừ (nếu được phép). Không tự ý bỏ qua quy tắc bắt buộc.

Ví dụ 09.03.01 / REV-SCOPE-001: Reviewer chỉ đọc baseline, candidate và đặc tả; chạy Git Diff, đối chiếu file và nội dung thay đổi với scope được phê duyệt; xuất ScopeReview.json và diff; PASS khi đúng scope, FAIL khi có thay đổi ngoài scope chưa duyệt; quay lại Implementation hoặc xin duyệt thay đổi scope; không tự sửa code hoặc miễn kiểm tra.

Traceability: REQ → SPEC → DESIGN → TASK → COMMIT → TEST CASE → TEST RESULT → APPROVAL → RELEASE. Liên kết có thể nhiều-nhiều; không ép một commit chỉ thuộc một yêu cầu. Khi thay đổi đầu vào, đánh dấu phê duyệt/kết quả phụ thuộc cần kiểm tra lại.

## 5. Điều phối và quyền hạn

- Rule Agent: đọc và kiểm tra quy ước.
- Analyst Agent: phân tích yêu cầu/mã nguồn, không sửa source.
- Architect Agent: thiết kế/đặc tả, không tự sửa source.
- Developer Agent: chỉ sửa phạm vi được cấp.
- Reviewer Agent: phiên độc lập, chỉ đọc, báo cáo.
- Test Agent: viết/chạy kiểm thử theo quyền, không sửa production source hoặc tự đổi expected result để đạt PASS.
- Security Agent: đánh giá bảo mật, báo cáo.
- Build Agent: chạy build đã cho phép, không sửa source.
- Release Agent: đóng gói/phát hành chỉ theo lệnh và phê duyệt hợp lệ.
- Admin/Product Owner: phê duyệt yêu cầu, phương án, phạm vi, nghiệm thu, release và thay đổi quy tắc.

Có thể dùng cùng một mô hình AI qua các phiên độc lập. Gói nhiệm vụ gồm baseline, quy ước, yêu cầu, đặc tả, scope, quyền, test criteria và định dạng đầu ra. AI không tự cấp quyền hoặc tự phê duyệt. Ghi lại agent/run, phiên bản cấu hình, công cụ, kết quả và bằng chứng.

## 6. Rules, State Machine và Quality Gates

Quy ước có một nguồn chính thức; ba lớp đồng bộ: Markdown cho người/AI; JSON cho máy; script/EXE cho kiểm tra thực thi. AGENTS.md hướng dẫn AI, không phải biện pháp cưỡng chế tuyệt đối. Không định nghĩa trùng lặp mâu thuẫn. Khóa phiên bản theo nhiệm vụ; thay đổi quy tắc cần duyệt và đánh giá tác động.

DDLC Engine quản lý trạng thái và điều kiện chuyển bước, không để AI tự đánh dấu hoàn thành. Trạng thái/kết quả cần phân biệt PASS, FAIL, NOT_RUN, BLOCKED, và N/A có lý do nếu không áp dụng. Gate kiểm chứng bằng bằng chứng và/hoặc phê duyệt có thẩm quyền. Kiểm tra tên, folder, version, diff, artifact, log bằng máy khi khả thi; chất lượng thuật toán, nguyên nhân gốc và chống chắp vá cần review kỹ thuật, không thể chỉ kiểm bằng script.

Khi thất bại quay về nguyên nhân thực sự: sai code → Implementation; sai thiết kế → Design; sai đặc tả → Specification; sai mục tiêu/scope → Requirement; lỗi môi trường → Baseline/Build. Ghi lý do, vô hiệu hóa kết quả bị ảnh hưởng, chạy lại các kiểm tra liên quan. Không mặc định sửa code cho mọi lỗi. Giới hạn retry và yêu cầu người quản lý xử lý nếu bế tắc. Không phát hành khi build/test bắt buộc thất bại hoặc chưa có nghiệm thu.

## 7. Hồ sơ quy trình theo rủi ro

| Hồ sơ | Ví dụ | Cách áp dụng |
|---|---|---|
| Rút gọn | Đổi nội dung hiển thị, icon, tên file ít rủi ro | Gộp bước/tài liệu khi phù hợp, vẫn xác định baseline/scope, review, kiểm tra bắt buộc, nghiệm thu |
| Tiêu chuẩn | Cải tiến UI, thêm chức năng, sửa thuật toán | Thực hiện đầy đủ các bước liên quan, test hồi quy |
| Nghiêm ngặt | App mới, thay kiến trúc, database, bản quyền, bảo mật | Đặc tả/thiết kế chi tiết, review độc lập, kiểm thử và phê duyệt tăng cường |

Chọn theo tác động thực tế, không chỉ theo tên công việc; có thể nâng mức. Không hồ sơ nào bỏ quy ước bắt buộc hoặc tạo PASS giả.

## 8. Nguyên tắc không được phá vỡ

1. Không code khi yêu cầu chưa thống nhất; không dùng bản lỗi làm nền để chắp vá.
2. Không tự đổi chức năng ngoài scope. Đổi scope phải duyệt lại.
3. Tiêu chí nghiệm thu có trước khi code; review diff và kiểm thử hồi quy phù hợp.
4. Build thành công không đồng nghĩa chức năng đúng. Không tuyên bố đã test nếu chưa thực hiện.
5. Chỉ phát hành khi đủ điều kiện và nghiệm thu.
6. Mọi thay đổi, quyết định, kết quả và artifact phải truy xuất được.
7. Bảo vệ khóa, dữ liệu, quyền truy cập; không tự thay quy ước.
8. Đơn giản hóa có căn cứ; không triển khai thừa; không phụ thuộc một AI cụ thể.

## 9. Bộ tiêu chuẩn triển khai lần lượt

| Mã | Nội dung | Trạng thái |
|---|---|---|
| DDLC-01 | Nguyên tắc và kiến trúc tổng thể | Chờ thiết kế |
| DDLC-02 | Quản lý quy ước | Chờ thiết kế |
| DDLC-03 | Quản lý vòng đời phát triển | Chờ thiết kế |
| DDLC-04 | Quy trình và thủ tục chi tiết | Chờ thiết kế |
| DDLC-05 | Quản lý AI Agent và quyền hạn | Chờ thiết kế |
| DDLC-06 | Quản lý dữ liệu và đặc tả | Chờ thiết kế |
| DDLC-07 | Kiểm tra và kiểm soát chất lượng | Chờ thiết kế |
| DDLC-08 | Quản lý mã nguồn và môi trường build | Chờ thiết kế |
| DDLC-09 | Quản lý nghiệm thu và phát hành | Chờ thiết kế |
| DDLC-10 | Quản lý lịch sử, bảo trì và cải tiến | Chờ thiết kế |

Thứ tự: nghiên cứu, phân tích, đặc tả, thiết kế, review và phê duyệt từng bộ; lưu phiên bản và trạng thái; không làm đồng loạt hoặc tự coi dự thảo là đã phê duyệt. Nếu có phụ thuộc ngược, ghi nhận và cập nhật qua quy trình quản lý thay đổi.

Tài liệu nền dự kiến: Constitution.md, AGENTS.md, Rules.json, Workflow.json, QualityGates.json, Roles.json, Database.sql, ProjectTemplate, TestTemplate, ReleaseTemplate. Định dạng và nguồn dữ liệu chính thức sẽ được chốt trong từng bộ tiêu chuẩn.

Lộ trình hệ thống: (1) chuẩn hóa tiêu chuẩn DDLC; (2) database và Manager; (3) Engine/Quality Gates; (4) tích hợp AI, Git, Build, Test; (5) nghiệm thu, release, backup và bảo trì. Ưu tiên thử nghiệm end-to-end với một dự án nhỏ trước khi mở rộng.

## 10. Trạng thái hiện tại

- V0.1: lưu phương án tổng thể từ thảo luận, chưa triển khai phần mềm, chưa xác minh tính khả thi của từng cơ chế.
- Bước tiếp theo: DDLC-01 – nguyên tắc và kiến trúc tổng thể. Chỉ chuyển bước sau khi người quản lý thống nhất kết quả.
