# DCAD-STD-16 — Quy trình QA mã nguồn, Build, kiểm thử và nghiệm thu

**Phiên bản:** 1.22  
**Cập nhật:** 26/11/2026  
**Mức áp dụng:** Bắt buộc

## 1. Bộ tiêu chí kỹ thuật QA mã nguồn [CORE]

**DCAD-CODE-QA-001.** Bắt buộc đánh giá mỗi lần tạo/sửa mã nguồn DCAD, gồm C++, C#, AutoLISP, script, project/build và dữ liệu/cấu hình liên quan. Đây là bộ tiêu chí và bằng chứng chất lượng, **không phải quy trình lập trình thứ hai**. STD-18 (AI-01–AI-18) là nguồn duy nhất trong hai quy ước về phương pháp làm việc của AI: khảo sát, baseline/phạm vi, nguyên nhân, lựa chọn giải pháp, diff, review, lập ca thử, retest và bàn giao. STD-16 sử dụng kết quả và bằng chứng từ STD-18, không lập lại checklist hoặc quy trình tương đương. Các hợp đồng kỹ thuật chuyên ngành vẫn thuộc quy ước tương ứng.

### 1.1. Tiêu chí kiểm toán kỹ thuật

Đánh giá các tiêu chí dưới đây theo tính năng và phạm vi tác động. Một tiêu chí không áp dụng phải có lý do; không tự coi một phép thử nguồn là bằng chứng runtime. Bằng chứng có thể dùng chung với ma trận STD-18 hoặc bộ test chuyên ngành, chỉ tham chiếu ID và vị trí, không tạo báo cáo/test trùng.

Giữ nguyên ID kỹ thuật từ V1.21 để truy vết. Các ID QA01–QA06, QA10–QA11, QA23–QA25 được rút khỏi checklist riêng vì đã thuộc STD-18; không cấp lại ID, không đánh N/A giả cho chúng. QA15 chỉ quy định chất lượng validation kỹ thuật, không lặp ma trận ca thử AI-16.

| Mã | Hạng mục | Tiêu chí kỹ thuật và bằng chứng |
|---|---|---|
| QA07 | Logic | Công thức, thuật toán, đơn vị, dấu, sai số, bất biến và kết quả nghiệp vụ đúng hợp đồng; số đo/đối chứng |
| QA08 | Control Flow | Tính đúng của điều kiện, vòng lặp, trạng thái, đường thoát, Cancel/ESC; kiểm nhánh bất khả đạt, vòng lặp vô hạn và trạng thái dở dang |
| QA09 | Data Flow | Kiểu dữ liệu, mapping, reader/writer, schema, default, chuyển đổi đơn vị, độ chính xác và encoding; dump/round-trip/đối chiếu dữ liệu |
| QA12 | Memory | Ownership, lifetime, null/dangling pointer, bounds, leak, double free; review hoặc analyzer theo ngôn ngữ |
| QA13 | Resources | RAII/dispose, file/handle/DB, event, lock, cleanup; không rò rỉ hoặc khóa tài nguyên sau khi thoát |
| QA14 | Exception | Hành vi khi thất bại, lỗi có ý nghĩa, rollback/cleanup, không nuốt lỗi hoặc để trạng thái hỏng |
| QA15 | Input Validation | Ràng buộc kiểu/miền giá trị, overflow, NaN/Inf, dữ liệu thiếu, encoding và giá trị biên theo hợp đồng; tham chiếu ca thử AI-16 |
| QA16 | Security | Validation tại ranh giới tin cậy, đường dẫn/quyền, bí mật, integrity và dependency; hợp đồng STD-20 nếu áp dụng; bằng chứng phân tích/test bảo mật |
| QA17 | Dependency | SDK/reference/runtime, kiến trúc, phiên bản và hợp đồng thư mục đúng host; đối chiếu manifest/import/dependency |
| QA18 | Compatibility | Host/OS/framework, dữ liệu cũ/mới, schema, command/settings theo hợp đồng; kết quả riêng từng target |
| QA19 | Performance | Chi phí vòng lặp, I/O, cấp phát, UI/CAD responsiveness; số đo trước/sau khi có thay đổi hiệu năng hoặc rủi ro đáng kể |
| QA20 | Data Integrity | Không mất dữ liệu, ghi nhầm record/layer/XData; tính nhất quán, transaction/Undo/rollback/backup; đối chứng trước/sau |
| QA21 | Static Analysis | Kết quả analyzer hoặc phương pháp phân tích tĩnh phù hợp, phạm vi, cảnh báo, phiên bản công cụ và giới hạn; review thủ công không được ghi là analyzer PASS |
| QA22 | Build | Bằng chứng theo STD-04 và checklist bên dưới: log, exit code, artifact mới, reference đúng từng target |
| QA26 | Packaging | Đối chiếu ZIP, version, BuildInfo, DataManifest, log, file thừa/thiếu, checksum khi cần và output từng host |

### 1.2. Phương pháp kiểm chứng và phân loại lỗi

- Công cụ lựa chọn theo ngôn ngữ/rủi ro/môi trường: MSVC Code Analysis, clang-tidy, Cppcheck, AddressSanitizer, Roslyn Analyzers, CodeQL, unit test C++/.NET hoặc script kiểm thử độc lập; không bắt buộc cài tất cả. Ghi phương pháp thực tế, phạm vi, kết quả và giới hạn. Không dùng kết quả của một phương pháp để chứng nhận phương pháp khác chưa chạy.
- P0 Critical: mất/hỏng dữ liệu, lỗ hổng nghiêm trọng; P1 High: crash, sai kết quả, lỗi chức năng chính; P2 Medium: lỗi phụ, hiệu năng hoặc tương thích; P3 Low: lỗi nhỏ/bảo trì; P4 Info: ghi nhận/đề xuất.
- P0/P1 ảnh hưởng bản bàn giao chưa xử lý chặn phát hành. Mọi vi phạm yêu cầu bắt buộc áp dụng đều chặn phát hành bất kể mức độ. Ngoại lệ chỉ dành cho yêu cầu khuyến nghị khi chủ dự án phê duyệt; không đổi FAIL thành PASS. Phát hiện ngoài phạm vi được ghi nhận, quyền sửa theo STD-18/01.

### 1.3. Hồ sơ và tổng hợp kết quả QA

Dùng một hồ sơ chung cho STD-18 và STD-16, tham chiếu bằng chứng hiện có thay vì lập lại. Mỗi hàng kỹ thuật/ca thử có `RequirementID | Applies | Expected | Actual | Result | Evidence | Host/Tool | Notes`; dùng PASS / FAIL / NotVerified / N/A theo phần điều kiện nghiệm thu bên dưới. Evidence phải truy được đến source/file:hàm, analyzer, log, test, ảnh hoặc hash phù hợp với phép kiểm. Danh sách phát hiện ghi ID, P0–P4, vị trí, ảnh hưởng, trạng thái xử lý và bằng chứng retest; tham chiếu AI-15–AI-18 cho thông tin review, diff, baseline, scope và bàn giao, không yêu cầu bản sao.

- `SOURCE_QA_RESULT`: tổng hợp QA07–QA21 còn hiệu lực (các ID kỹ thuật tại bảng trên) và yêu cầu chất lượng nguồn chuyên ngành áp dụng. FAIL nếu có tiêu chí bắt buộc FAIL; nếu không FAIL nhưng thiếu kiểm chứng bắt buộc thì NotVerified; PASS khi tất cả tiêu chí nguồn bắt buộc áp dụng đã PASS, N/A có lý do.
- `BUILD_RESULT`, `STANDARD_RESULT`, `RUNTIME_RESULT`, `RELEASE_APPROVED`: theo phần trạng thái và điều kiện bàn giao bên dưới. STANDARD_RESULT bao gồm kết quả STD-18/AI-01–AI-18 và các quy ước khác áp dụng. RELEASE_APPROVED chỉ YES nếu SOURCE_QA_RESULT và mọi kết quả bắt buộc khác áp dụng đều PASS, không còn lỗi chặn. Không suy từ QA tĩnh hoặc build sang runtime.
- Thiếu môi trường/SDK/host hoặc dữ liệu: ghi NotVerified, blocker và phép thử cần thực hiện. Điều kiện gửi source chờ kiểm thử thực hiện theo STD-18 AI-18 và cổng nghiệm thu bên dưới; không tự xác nhận production.

## DataManifest, BuildInfo, checklist và nghiệm thu [CORE]

*Cập nhật v1.17: bổ sung nghiệm thu root Reference đúng CAD, output không suffix host và portable phân nhánh; giữ nguyên các kiểm thử XData và điều kiện khác.*

**Mỗi package** tối thiểu cần có hoặc khai báo rõ:

- `BuildInfo`: project/version, assembly, package, host target, framework, platform, output, build status và thời điểm/checksum nếu có.
- `DataManifest`: mỗi tài nguyên thực sự sử dụng: `FileName/ResourceName`, `Type`, `Required`, `LogicalFolder` (nếu dùng Support Path), `AccessMode`, `SourcePath`, `DeployPath`, `Embedded`, `DefaultBehavior`, `OwnerModule`; bổ sung `RelativePath` cho tài nguyên trong thư mục con và hợp đồng vị trí settings/log/job riêng; bổ sung schema/version/encoding khi liên quan. Trường `Required` xác định tính bắt buộc **theo module/chức năng**, không suy ra phụ thuộc vào toàn bộ thư mục DCAD. Các type: `BuildTime`, `RuntimeSystem`, `RuntimeUser`, `RuntimeProgram`, `RuntimeIcon`, `Sample`, `Documentation`.
- `Log`: Version/Date/Summary/FilesChanged/CommandsChanged/DataImpact/LanguageImpact/LayerImpact/XDataImpact/Compatibility/BuildResult/KnownIssues.
- **Checklist trước bàn giao**:

  - [ ] Với AI, tham chiếu kết quả và bằng chứng STD-18 (AI-01–AI-18); không lập lại checklist phương pháp lập trình.
  - [ ] Target/framework/reference đúng cho từng CAD, không lẫn host.
  - [ ] Xác nhận các target được triển khai đều thuộc AutoCAD 2022, AutoCAD 2025 hoặc GstarCAD 2027 và đúng phạm vi dự án. Không còn nhánh source, SDK/Reference, script, project target, DLL, log, output portable hoặc mục manifest của CAD ngoài phạm vi.
  - [ ] Release build từng target được kiểm chứng riêng; `Build_All.cmd` ở root điều phối `Build_Release.cmd /nopause` đúng thư mục `%~3`, không dùng nhãn `%~2` thay đường dẫn.
  - [ ] Kiểm tra ngay `ERRORLEVEL` của script con + sự tồn tại DLL mới; dù một nhánh FAIL các nhánh còn lại vẫn chạy để tổng hợp.
  - [ ] Staging/publish portable `Dprogram/<Host>/` chỉ sau khi toàn bộ target PASS; không trộn hoặc ghi đè file cùng tên giữa hai CAD.
  - [ ] Output dùng đúng basename module, không có `C22`/`C25`/`G27`; extension `.dll`/`.arx`/`.grx` theo công nghệ. Đối chiếu project, script, BuildInfo, manifest, NETLOAD/APPLOAD.
  - [ ] Reference/SDK đúng ba root mặc định `D:\000\Reference\Autocad 2022`, `D:\000\Reference\Autocad 2025`, `D:\000\Reference\GstarCAD 2027` hoặc override tường minh có log; không trộn CAD.
  - [ ] Khi chạy trực tiếp `Build_All.cmd` hoặc `Build_Release.cmd`, **dù PASS hoặc FAIL**, sau tổng kết/log đều in `Nhấn phím bất kỳ để thoát...` và `pause >nul`; cửa sổ chỉ đóng sau khi có phím nhấn, kể cả lỗi sớm (thiếu SDK/script, build/publish lỗi).
  - [ ] Script con chạy với `/nopause` không chờ giữa các nhánh; script tổng chạy thủ công chỉ chờ **một lần ở cuối**; mọi đường kết thúc đi qua `:FINISH`.
  - [ ] Sau khi chờ phím, mã thoát vẫn là `0` khi PASS, khác `0` khi FAIL; không dùng `ERRORLEVEL` của lệnh `pause` làm kết quả build. Màu PASS xanh/FAIL đỏ khi terminal hỗ trợ và fallback chữ thuần khi `NO_COLOR`.
  - [ ] Có **bằng chứng màu hiển thị trực tiếp** cho `[BUILD]` xanh dương, `[PASS]` xanh lá, `[FAIL]` đỏ, `[WARNING]` vàng, tiêu đề cyan; kiểm tra ANSI/PowerShell khi có hỗ trợ, nhánh `NO_COLOR`/`COLOR_UNAVAILABLE` có lý do. Chỉ có dòng mô tả màu hoặc ANSI string trong source **không đạt kiểm chứng**.
  - [ ] Root `log/` có Build_All và log từng target, văn bản thuần; không nhầm Build PASS với NETLOAD PASS.
  - [ ] NETLOAD/smoke test thực tế đúng host, ghi PASS/FAIL/NotVerified; kiểm tra Undo/ESC khi có draw/Form.
  - [ ] `DataManifest` liệt kê đúng **các tài nguyên từng module/chức năng thực sự sử dụng** và mức `Required`. Khi chạy module, chỉ kiểm tra các thư mục/tệp bắt buộc trong phạm vi đó. **Không FATAL** vì thiếu `DData`, `DProgram`, `DHelp`, `Dicon`, `DUser` hoặc root `DCAD` nếu module/chức năng không sử dụng. DIRECT-only và phát hiện xung đột vẫn áp dụng cho thư mục cần dùng.
  - [ ] **Setup/QA cấp hệ thống và module** kiểm tài nguyên theo phạm vi chức năng triển khai; không ép đủ năm thư mục, cùng root, liền kề hoặc thứ tự/vị trí. Tên thư mục cuối của entry DIRECT phải khớp logic; `DUser\abc` không thay DUser, root không thay thư mục con. RelativePath ứng dụng phải khai báo rõ, không suy từ entry cha.
  - [ ] Chạy **DIR01–DIR38** theo phạm vi SYS/MOD của STD-05 hiện hành: không lỗi vì thư mục không dùng; reject PARENT và entry con sai tên cuối; chấp nhận tên cha tự do/khác hoa-thường; kiểm relative path ứng dụng cố định. DIR33 phải xác nhận Setup/QA và module không ép đủ năm khi chỉ cần DUser.
  - [ ] Module **có sử dụng DUser** phải dùng một resolver DUser thống nhất cho settings/log. Lỗi ghi log hiển thị trong CAD mà không đòi file log và không ghi sang nơi khác; không gây FATAL nếu log là tùy chọn. `Shared_data.Set` **bắt buộc đối với chức năng dùng ngôn ngữ DCAD**, không phải điều kiện cho module không dùng tệp này; fallback Lang_0 chỉ áp dụng cho key bản dịch thiếu trong bộ ngôn ngữ hợp lệ. Log BUILD tuân theo quy chuẩn 04.
  - [ ] 6 ngôn ngữ và font thể hiện không lỗi trên host mục tiêu nếu phần mềm có text/UI.
  - [ ] Khi dùng ngôn ngữ DCAD, kiểm `DUser/Shared_data.Set` dòng 1, lấy số nguyên không âm trong `[]` và đối chiếu block `Lang_X` nhúng; không fallback file cũ hay English khi cấu hình sai. Chạy LANG01–LANG08 tại STD-07.
  - [ ] Geometry, layer, DIM, preview đúng và nhất quán; không để entity/command debug.
  - [ ] Khi có HoleType: catalog/key, Gap A/B, W/T/R/âm, Count lỗ thật và Create↔Read round-trip.
  - [ ] Khi có XData/XRecord: công bố **target schema + nguồn/version**; không dùng schema V356 để viết bản ghi V381; carrier, Tok0/STT, reader legacy, round-trip, không đếm descriptive entity.
  - [ ] Khi tương thích DPunch-V381: Circle `Tok0=1/Tok2=Dia`, Profile `Tok0=5/Tok2=STT`; Hole NAME 16 token; Tok13 PointAngle khớp `HOLE_TYPE.Tok3`; Tok14/15 relief, Note `$`, placeholder và alias chỉ reader.
  - [ ] Khi dùng MASTER DPunch-V381: `HOLE_TABLE`/`PROJECTION`, yêu cầu projection để hợp lệ; CP1 đúng 7 token; không coi snapshot là Hole entity; STT CP giữ đúng nguồn `5 CP1`.
  - [ ] Khi có XRecord: kiểm toán writer/reader/schema/version theo module, **không tuyên bố DPunch-V381 có `DCAD_DPUNCH` XRecord** khi schema DPunch-V381 không quy định nó.
  - [ ] Settings DUser hoặc portable riêng theo manifest lưu atomic có backup, Last Used đúng variant, Factory chỉ restore nguồn đã chốt; DNesting JSON cạnh EXE không bị chuyển về DUser.
  - [ ] Package ZIP tổng một file, các nhánh cùng version, không đóng nhầm reference sang runtime/end-user. Không đưa file `DCAD-BUILD-CMD-001.md` vào ZIP phần mềm.

## Ma trận điều kiện nghiệm thu theo từng quy chuẩn

**Quy tắc chung:** Tách `SYS` (Setup/QA toàn hệ thống) khỏi `MOD` (module/chức năng). Chỉ đặt `FAIL` cho tài nguyên **bắt buộc thuộc phạm vi kiểm tra**, không gán `FAIL`/`FATAL` cho module vì thiếu tài nguyên không dùng. Mỗi hàng áp dụng phải được ghi `PASS`, `FAIL`, `NotVerified` hoặc `N/A` kèm **bằng chứng** và **host/phiên bản kiểm tra**. `N/A` chỉ dùng cho tính năng không có hoặc host ngoài phạm vi được duyệt. Kiểm tra tĩnh chỉ xác nhận code/cấu hình; không thay thế kiểm tra hành vi UI/CAD.

| Mã | Điều kiện kiểm tra trọng tâm | Bằng chứng tối thiểu khi áp dụng |
|---|---|---|
| STD-01 | Baseline chỉ định hoặc gần nhất; version đích mặc định max đã lưu + 1; phạm vi thay đổi và lịch sử đúng | Version nguồn, version lớn nhất đã lưu, lý do chọn, version đích và biên bản ảnh hưởng |
| STD-02 | Một command; tách module; không pha API host | Danh sách command từ source, đối chiếu dependency và reference |
| STD-03 | Cây riêng host, basename không suffix, portable chia host, không có artifact CAD ngoài phạm vi | ZIP, tên binary, cây `Dprogram/<Host>/`, Reference/manifest |
| STD-04 | Build đúng Reference từng host; artifact DLL/ARX/GRX mới không suffix, publish chia host; màu/pause/log | Log, `ERRORLEVEL`, so tên file, kiểm ghi đè và màu/`NO_COLOR` |
| STD-05 | SYS/MOD kiểm phụ thuộc thực tế; tên cuối DIRECT đúng; không ép đủ năm/cùng root/vị trí; không PARENT; relative path ứng dụng đúng hợp đồng | DataManifest/Required/RelativePath; DIR01–DIR38; thử entry DUser/abc và DProgram/Dnest, EXE cố định |
| STD-06 | Khi tính năng dùng DUser: Save/Last Used/Factory đúng DUser, atomic + backup | File trước/sau Save/Cancel/Fail/Restore, test lỗi quyền và file hỏng |
| STD-07 | Khi module dùng ngôn ngữ DCAD: Shared_data.Set dòng 1 lấy số trong `[]`; đủ 6 ngôn ngữ; Unicode | Đối chiếu key `lang.dat`, ảnh/text trong CAD theo từng ngôn ngữ |
| STD-08 | Đúng command/selection/typed input; ESC không để state rác | NETLOAD và thao tác thực tế với đối tượng/ESC từng host |
| STD-09 | Hình học, layer, DIM và sai số cùng logic | DWG đối chứng, số đo, layer và DIM sau lệnh |
| STD-10 | HoleType canonical đúng nguồn hình học/số bước | Ma trận HoleType được hỗ trợ; hình học và reader kiểm chứng |
| STD-11 | Gap, dấu, W/T/R, Depth/Offset theo hợp đồng | Bộ test Create→DWG→Read và so kích thước, layer, note |
| STD-12 | XData/XRecord đúng schema và token; backward reader | Dump XData/XRecord, test round-trip, dữ liệu cũ và mới |
| STD-13 | UI/DPI/ngôn ngữ/Preview đúng; icon hợp lệ | Ảnh kiểm tra thực tế UI/Preview, thao tác focus/ESC |
| STD-14 | Transaction/Undo/ESC và hiệu năng không phá bản vẽ | DWG trước/sau Cancel/Undo, log ngoại lệ, phép thử tập lớn khi liên quan |
| STD-15 | Lỗi hiển thị đúng, log/diagnostic đủ, HARD STOP khi lặp | Ca lỗi giả lập hoặc tái hiện; log lỗi và hành vi khôi phục |
| STD-16 | Manifest, BuildInfo, release gating, trạng thái đúng | Bảng nghiệm thu cuối và đối chiếu thực tế từng file/host |
| STD-18 | Đạt ma trận AI-01–AI-18 theo phạm vi | Tham chiếu ma trận và bằng chứng STD-18, không lập lại |
| STD-17 | Khi cập nhật GitHub hoặc xác nhận OK: baseline/version, lưu bản mới, metadata và kiểm tra sau commit | AUTO-01–AUTO-13 theo STD-17, commit, ZIP/metadata và trạng thái nghiệm thu riêng |
| STD-20 | Khi tích hợp bản quyền/bộ bàn giao Key: tuân thủ hợp đồng hiện hành, không làm suy yếu Gate | Ma trận và bằng chứng nghiệm thu theo STD-20 đúng target, build/runtime tách riêng |

### Trạng thái và điều kiện bàn giao

1. **`BUILD_RESULT`:** chỉ phản ánh build/package DLL (`PASS/FAIL/NotVerified`). Không sử dụng nó để kết luận màu sắc, ngôn ngữ hoặc nghiệp vụ đều PASS.
2. **`STANDARD_RESULT`:** tổng hợp các yêu cầu bắt buộc áp dụng từ ma trận trên. Một mục `FAIL` hoặc `NotVerified` → **chưa đạt nghiệm thu đầy đủ**; các mục `N/A` phải có lý do. Yêu cầu khuyến nghị chưa đạt chỉ được giữ ngoại lệ khi có phê duyệt.
3. **`RUNTIME_RESULT`:** kiểm thử NETLOAD và tương tác thực tế trên đúng phiên bản CAD; độc lập với compile và QA tĩnh.
4. **`RELEASE_APPROVED`:** chỉ được ghi `YES` khi `SOURCE_QA_RESULT` đạt PASS (nếu áp dụng), các yêu cầu bắt buộc **của phạm vi bàn giao** đã PASS, build của các target bắt buộc PASS và runtime cần nghiệm thu đã PASS. Setup/QA cấu hình hệ thống cũng chỉ yêu cầu tài nguyên của các chức năng triển khai; không lấy điều kiện đủ năm thư mục hoặc entry root làm cổng chặn. Hợp đồng portable riêng phải được QA theo đúng module, không tự đổi vị trí settings. Nếu chỉ giao source chờ chạy trên CAD thực, phải ghi `NOT_VERIFIED`, không tự tuyên bố đã nghiệm thu.
5. **Sai màu CMD:** nếu biên dịch/publish đã thành công thì `BUILD_RESULT` vẫn phản ánh đúng là PASS; riêng mục hiển thị màu `FAIL/NotVerified` làm **chưa đạt điều kiện phát hành**, không được bỏ qua chỉ vì DLL đã tạo thành công. Trường hợp cố ý `NO_COLOR` là ca kiểm tra tương thích và không phải lỗi màu.
6. Kèm theo bản giao biên bản ngắn: `RequirementID | Applies | Result | Evidence | Host | Notes`; file này thuộc **Userdata/log package phần mềm khi phù hợp**, không chèn bản sao bộ tiêu chuẩn `.md` vào ZIP phần mềm. Không đặt file rác Test/Tests vào ZIP bàn giao.

- Kiểm tra cập nhật GitHub theo STD-17 AUTO-01–AUTO-13: chọn baseline gần nhất khi không có chỉ định; version mới mặc định max đã lưu + 1; không ghi đè bản cũ; README module và DCAD có trạng thái mới nhất/đã nghiệm thu riêng; đọc lại ZIP/changelog/metadata sau commit. Chỉ lưu bản gốc hoặc metadata không tự tăng version phần mềm.

- Khi AI phân tích/viết/sửa/rà soát code, kiểm tra **STD-18, AI-01–AI-18** theo phạm vi; không gán PASS khi thiếu bằng chứng.

## Bổ sung 19/11/2026 — Kiểm thử EXE portable trong thư mục con DProgram

- Chỉ kích hoạt khi module dùng ứng dụng portable độc lập. `DataManifest` phải ghi chính xác tên EXE, `LogicalFolder=DProgram`, `RelativePath` nếu biết, và `PortableExeDiscovery=DirectChild` nếu cần tìm tự động một cấp. Chỉ xét `DProgram` DIRECT từ CAD, không nhận root/entry con như DIRECT thay thế, không quét sâu hoặc chọn tùy tiện `*.exe`.
- **PORT01:** EXE tại đường dẫn `RelativePath` hợp lệ → chọn đúng và không thay bằng ứng viên khác.
- **PORT02:** Bật discovery, EXE ở thư mục gốc hoặc đúng **một** thư mục con trực tiếp của `DProgram` → nhận diện và chạy đúng EXE; working directory theo thư mục EXE khi cần.
- **PORT03:** Có hai thư mục con chứa EXE cùng tên → báo không rõ nguồn và dừng chức năng, không chọn tùy tiện.
- **PORT04:** EXE chỉ có ở cấp con sâu hơn, root DCAD, `System32`, `PATH` hoặc `AppData` → không tự tìm; `RelativePath` cố định nhiều cấp chỉ hợp lệ khi đã khai báo và kiểm tra rõ.
- **PORT05:** Thiếu EXE/thiếu DProgram, quyền truy cập sai, EXE không đúng hoặc reparse point thoát khỏi DProgram → cảnh báo đúng chức năng, không làm module khác FATAL.
- **PORT06:** Không xê dịch DLL/dữ liệu đi kèm portable, không thay đổi quy định `DProgram\sqlite3.dll` cho SQLite Runtime, không chuyển DB/settings ra khỏi folder đã công bố.
- Kết quả `PASS/FAIL/NotVerified` dựa trên thử nghiệm trên CAD/Windows thật; chỉ đọc quy ước hoặc có source dò file không được tính là runtime PASS.

## Bổ sung 19/11/2026 — Cổng QA cho SQLite Runtime dùng chung

- **Chỉ áp dụng cho module dùng SQLite:** kiểm `DProgram\sqlite3.dll` x64, nguồn/phiên bản/SHA-256 đã phê duyệt, đúng một runtime dùng chung; không cần cài SQLite trên máy, không tìm/cài DLL ở `System32`, không cho module tự thay bản runtime của module khác. Theo STD-03 và hợp đồng `DCAD-SQLITE-RUNTIME-001` tại STD-05.
- **Kiểm build/native import:** đối chiếu project/linker và import table của ARX/GRX. Không để liên kết `WinSQLite3.lib` hoặc `sqlite3.lib` khiến DLL SQLite trở thành phụ thuộc bắt buộc ngay lúc CAD nạp ARX; dùng nạp động theo đường dẫn tuyệt đối từ `DProgram`, kiểm API và phiên bản.
- **Kiểm runtime độc lập:** thực hiện SQL01–SQL08 trong STD-05. Tách `ARX LOAD` khi thiếu DLL, khi có DLL, thao tác đọc/ghi DB, trường hợp không có DProgram DIRECT, DLL sai kiến trúc, DB hỏng/khóa, hai add-in dùng chung. Các trạng thái Build/Load/DB Runtime phải ghi riêng `PASS/FAIL/NotVerified`, không lấy compile PASS làm ARX LOAD PASS.
- **DataManifest:** khai báo SQLite DLL là runtime dùng chung ở DProgram theo tính năng; khai báo riêng từng DB với LogicalFolder/AccessMode/schema, không gộp DLL và dữ liệu. Module không dùng SQLite ghi `N/A`, không yêu cầu `DProgram` chỉ vì hệ thống có SQLite.


## Bổ sung 20/11/2026 — QA chung cho bảng dữ liệu và SQLite Grid

**Áp dụng mọi module có bảng sửa được, bất kể nguồn dữ liệu.** Kiểm theo hợp đồng `DCAD-GRID-001` (STD-13); module SQLite kiểm thêm `DCAD-SQLITE-GRID-001` (STD-05) và SQL01–SQL08 về thư viện chạy. Không tự đánh Build PASS thay cho UI/runtime PASS.

| Mã | Ca kiểm thử | Kết quả bắt buộc |
|---|---|---|
| GRID01 | Mở bảng có nhiều hàng và cột, giá trị Unicode, ký tự dài | Đúng tiêu đề, đúng ô, đúng trường; không ghép TAB/CSV thành một `EDIT` |
| GRID02 | Sửa lần lượt ô đầu/giữa/cuối, Enter, click ra ngoài, ESC | Đúng ô được chốt/hủy; không làm đổi ô liền kề |
| GRID03 | Thêm/xóa/move/sort/filter rồi sửa ô | Khóa bản ghi ổn định, không ghi vào hàng khác do đổi thứ tự |
| GRID04 | Dán TAB/xuống dòng hoặc giá trị vượt giới hạn vào ô | Bị từ chối có ý nghĩa hoặc xử lý qua tính năng paste-bảng đã khai báo; không lệch cột |
| GRID05 | Kéo cột tới giới hạn, kéo hàng, resize cửa sổ, đổi DPI | Cột không biến mất; dữ liệu không sang cột khác; có giới hạn nhỏ nhất |
| GRID06 | Bảng đủ rộng và bảng hẹp | Mép cuối khép kín khi đủ rộng; cuộn ngang đúng khi thiếu rộng; không có khoảng trống ngoài cột cuối |
| GRID07 | Rê chuột liên tục, cuộn bánh xe, Home/End/Page Up/Down | Không vẽ chồng/nhấp nháy bất thường; không thay giá trị |
| GRID08 | Đang sửa rồi cuộn/kéo/resize/đóng; mở lại | Editor không “trôi” sai hàng/cột; Save/Cancel rõ; trạng thái hợp lệ |
| GRID09 | Trường khóa, quan hệ và tên nhóm | Không đổi ngầm khóa; lỗi nhập xác định đúng hàng/cột |
| GRID10 | Nhiều cửa sổ cùng sửa dữ liệu | Cảnh báo xung đột, không ghi đè âm thầm |

**Chỉ module SQLite:** có thêm ca kiểm chứng dữ liệu, không thay thế SQL01–SQL08.

| Mã | Ca kiểm thử | Kết quả bắt buộc |
|---|---|---|
| SQL09 | Ánh xạ SELECT cột tường minh → grid → UPDATE/INSERT bind theo khóa | Một ô = một cột; không dùng `SELECT *` hoặc phụ thuộc chỉ số hàng hiển thị |
| SQL10 | Thay một trường rồi lưu, so DB trước/sau | Chỉ bản ghi liên quan thay đổi; không DELETE toàn bảng và reinsert |
| SQL11 | Xóa/thêm, mã trùng, nhóm sai, khóa ngoại, số âm/NaN, Unicode/TAB | Validation theo cột; transaction rollback và backup nhất quán khi thất bại |
| SQL12 | Lưu sau khi DB bị phiên khác sửa; DB khóa/bận, DB read-only | Không mất dữ liệu, báo xung đột/lỗi đúng; không tự fallback sang nguồn khác |
| SQL13 | DData nguồn và DUser cấu hình | Form cấu hình không ghi vào nguồn; chỉ form quản lý danh mục được quyền ghi nguồn |
| SQL14 | Không có shared `DProgram\\sqlite3.dll`; chỉ có `winsqlite3.dll` hoặc DLL khác PATH | Không fallback DLL; ARX nạp độc lập, phần DB báo lỗi rõ và dừng an toàn |
| SQL15 | DB schema cũ/khác version và cài lại source | Không tự ghi đè DB, không tự sửa schema khi thiếu chính sách migration |

Ma trận nghiệm thu mỗi ca: `RequirementID | Applies | Expected | Actual | Evidence | Host | PASS/FAIL/NotVerified`. Kiểm thử C++ độc lập có thể PASS cho logic, nhưng các ca đòi Win32/CAD runtime vẫn **NotVerified** nếu chưa chạy thực tế.
