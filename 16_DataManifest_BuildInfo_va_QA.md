# DCAD-STD-16 — DataManifest, BuildInfo và QA

**Phiên bản:** 1.14  
**Cập nhật:** 09/11/2026  
**Mức áp dụng:** Bắt buộc

## DataManifest, BuildInfo, checklist và nghiệm thu [CORE]

**Mỗi package** tối thiểu cần có hoặc khai báo rõ:

- `BuildInfo`: project/version, assembly, package, host target, framework, platform, output, build status và thời điểm/checksum nếu có.
- `DataManifest`: mỗi tài nguyên thực sự sử dụng: `FileName/ResourceName`, `Type`, `Required`, `LogicalFolder` (nếu dùng Support Path), `AccessMode`, `SourcePath`, `DeployPath`, `Embedded`, `DefaultBehavior`, `OwnerModule`; bổ sung `RelativePath` cho tài nguyên trong thư mục con và hợp đồng vị trí settings/log/job riêng; bổ sung schema/version/encoding khi liên quan. Trường `Required` xác định tính bắt buộc **theo module/chức năng**, không suy ra phụ thuộc vào toàn bộ thư mục DCAD. Các type: `BuildTime`, `RuntimeSystem`, `RuntimeUser`, `RuntimeProgram`, `RuntimeIcon`, `Sample`, `Documentation`.
- `Log`: Version/Date/Summary/FilesChanged/CommandsChanged/DataImpact/LanguageImpact/LayerImpact/XDataImpact/Compatibility/BuildResult/KnownIssues.
- **Checklist trước bàn giao**:

  - [ ] Tất cả source thay đổi đều có lý do và phạm vi impact; bảo toàn baseline những phần ngoài scope.
  - [ ] Với module dùng bản quyền DCAD, áp dụng STD-19: manifest, registry, UID, chứng nhận, Tool/Feature Gate, chữ ký tin cậy, không rò khóa, QA LIC01–LIC26 theo phạm vi.
  - [ ] Với AI, áp dụng STD-18: nêu mục tiêu, không suy diễn, không thêm chức năng ngoài phạm vi; diff tối thiểu và ca thử có expected/actual.
  - [ ] Target/framework/reference đúng cho từng CAD, không lẫn host.
  - [ ] Xác nhận các target được triển khai đều thuộc AutoCAD 2022, AutoCAD 2025 hoặc GstarCAD 2027 và đúng phạm vi dự án. Không còn nhánh source, SDK/Reference, script, project target, DLL, log, output portable hoặc mục manifest của CAD ngoài phạm vi.
  - [ ] Release build từng target được kiểm chứng riêng; `Build_All.cmd` ở root điều phối `Build_Release.cmd /nopause` đúng thư mục `%~3`, không dùng nhãn `%~2` thay đường dẫn.
  - [ ] Kiểm tra ngay `ERRORLEVEL` của script con + sự tồn tại DLL mới; dù một nhánh FAIL các nhánh còn lại vẫn chạy để tổng hợp.
  - [ ] Staging/publish portable `Dprogram/` chỉ sau khi toàn bộ target PASS; lỗi copy không để DLL công bố dở.
  - [ ] Khi chạy trực tiếp `Build_All.cmd` hoặc `Build_Release.cmd`, **dù PASS hoặc FAIL**, sau tổng kết/log đều in `Nhấn phím bất kỳ để thoát...` và `pause >nul`; cửa sổ chỉ đóng sau khi có phím nhấn, kể cả lỗi sớm (thiếu SDK/script, build/publish lỗi).
  - [ ] Script con chạy với `/nopause` không chờ giữa các nhánh; script tổng chạy thủ công chỉ chờ **một lần ở cuối**; mọi đường kết thúc đi qua `:FINISH`.
  - [ ] Sau khi chờ phím, mã thoát vẫn là `0` khi PASS, khác `0` khi FAIL; không dùng `ERRORLEVEL` của lệnh `pause` làm kết quả build. Màu PASS xanh/FAIL đỏ khi terminal hỗ trợ và fallback chữ thuần khi `NO_COLOR`.
  - [ ] Có **bằng chứng màu hiển thị trực tiếp** cho `[BUILD]` xanh dương, `[PASS]` xanh lá, `[FAIL]` đỏ, `[WARNING]` vàng, tiêu đề cyan; kiểm tra ANSI/PowerShell khi có hỗ trợ, nhánh `NO_COLOR`/`COLOR_UNAVAILABLE` có lý do. Chỉ có dòng mô tả màu hoặc ANSI string trong source **không đạt kiểm chứng**.
  - [ ] Root `log/` có Build_All và log từng target, văn bản thuần; không nhầm Build PASS với NETLOAD PASS.
  - [ ] NETLOAD/smoke test thực tế đúng host, ghi PASS/FAIL/NotVerified; kiểm tra Undo/ESC khi có draw/Form.
  - [ ] `DataManifest` liệt kê đúng **các tài nguyên từng module/chức năng thực sự sử dụng** và mức `Required`. Khi chạy module, chỉ kiểm tra các thư mục/tệp bắt buộc trong phạm vi đó. **Không FATAL** vì thiếu `DData`, `DProgram`, `DHelp`, `Dicon`, `DUser` hoặc root `DCAD` nếu module/chức năng không sử dụng. DIRECT-only và phát hiện xung đột vẫn áp dụng cho thư mục cần dùng.
  - [ ] **Setup/QA cấp hệ thống và module** kiểm tài nguyên theo phạm vi chức năng triển khai; không ép đủ năm thư mục, cùng root, liền kề hoặc thứ tự/vị trí. Tên thư mục cuối của entry DIRECT phải khớp logic; `DUser\abc` không thay DUser, root không thay thư mục con. RelativePath ứng dụng phải khai báo rõ, không suy từ entry cha.
  - [ ] Chạy **DIR01–DIR38** theo phạm vi SYS/MOD của STD-05 hiện hành: không lỗi vì thư mục không dùng; reject PARENT và entry con sai tên cuối; chấp nhận tên cha tự do/khác hoa-thường; kiểm relative path ứng dụng cố định. DIR33 phải xác nhận Setup/QA và module không ép đủ năm khi chỉ cần DUser.
  - [ ] Module **có sử dụng DUser** phải dùng một resolver DUser thống nhất cho settings/log. Lỗi ghi log hiển thị trong CAD mà không đòi file log và không ghi sang nơi khác; không gây FATAL nếu log là tùy chọn. `MIS_SET.SET` **bắt buộc đối với chức năng dùng ngôn ngữ DCAD**, không phải điều kiện cho module không dùng tệp này; fallback Lang_0 chỉ áp dụng cho key bản dịch thiếu trong bộ ngôn ngữ hợp lệ. Log BUILD tuân theo quy chuẩn 04.
  - [ ] 6 ngôn ngữ và font thể hiện không lỗi trên host mục tiêu nếu phần mềm có text/UI.
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
| STD-03 | Cây riêng từng host; đủ 7 thư mục; không có cấu hình hoặc artifact của CAD ngoài phạm vi | Kiểm tra ZIP/csproj/Reference/manifest và danh sách host |
| STD-04 | Build mỗi nhánh; DLL mới; publish; màu hiển thị; pause; kết quả/log | Log build, `ERRORLEVEL`, kiểm tra DLL, kiểm thử màu quan sát trực tiếp và `NO_COLOR` |
| STD-05 | SYS/MOD kiểm phụ thuộc thực tế; tên cuối DIRECT đúng; không ép đủ năm/cùng root/vị trí; không PARENT; relative path ứng dụng đúng hợp đồng | DataManifest/Required/RelativePath; DIR01–DIR38; thử entry DUser/abc và DProgram/Dnest, EXE cố định |
| STD-06 | Khi tính năng dùng DUser: Save/Last Used/Factory đúng DUser, atomic + backup | File trước/sau Save/Cancel/Fail/Restore, test lỗi quyền và file hỏng |
| STD-07 | Khi module dùng ngôn ngữ DCAD: MIS_SET.SET dòng 28; đủ 6 ngôn ngữ; Unicode | Đối chiếu key `lang.dat`, ảnh/text trong CAD theo từng ngôn ngữ |
| STD-08 | Đúng command/selection/typed input; ESC không để state rác | NETLOAD và thao tác thực tế với đối tượng/ESC từng host |
| STD-09 | Hình học, layer, DIM và sai số cùng logic | DWG đối chứng, số đo, layer và DIM sau lệnh |
| STD-10 | HoleType canonical đúng nguồn hình học/số bước | Ma trận HoleType được hỗ trợ; hình học và reader kiểm chứng |
| STD-11 | Gap, dấu, W/T/R, Depth/Offset theo hợp đồng | Bộ test Create→DWG→Read và so kích thước, layer, note |
| STD-12 | XData/XRecord đúng schema và token; backward reader | Dump XData/XRecord, test round-trip, dữ liệu cũ và mới |
| STD-13 | UI/DPI/ngôn ngữ/Preview đúng; icon hợp lệ | Ảnh kiểm tra thực tế UI/Preview, thao tác focus/ESC |
| STD-14 | Transaction/Undo/ESC và hiệu năng không phá bản vẽ | DWG trước/sau Cancel/Undo, log ngoại lệ, phép thử tập lớn khi liên quan |
| STD-15 | Lỗi hiển thị đúng, log/diagnostic đủ, HARD STOP khi lặp | Ca lỗi giả lập hoặc tái hiện; log lỗi và hành vi khôi phục |
| STD-16 | Manifest, BuildInfo, release gating, trạng thái đúng | Bảng nghiệm thu cuối và đối chiếu thực tế từng file/host |
| STD-18 | Hiểu yêu cầu, tối giản giải pháp, thay đổi đúng phạm vi, kiểm chứng theo mục tiêu | Scope, diff, ca kiểm thử, log và AI-01–AI-12 |
| STD-19 | License DCAD: Tool/Feature UID, manifest, bộ tích hợp, chứng nhận, phát hành và gate quyền | Registry, diff, contract/certificate, bộ test LIC01–LIC26 và test thực tế host áp dụng |

### Trạng thái và điều kiện bàn giao

1. **`BUILD_RESULT`:** chỉ phản ánh build/package DLL (`PASS/FAIL/NotVerified`). Không sử dụng nó để kết luận màu sắc, ngôn ngữ hoặc nghiệp vụ đều PASS.
2. **`STANDARD_RESULT`:** tổng hợp các yêu cầu bắt buộc áp dụng từ ma trận trên. Một mục `FAIL` hoặc `NotVerified` → **chưa đạt nghiệm thu đầy đủ**; các mục `N/A` phải có lý do. Yêu cầu khuyến nghị chưa đạt chỉ được giữ ngoại lệ khi có phê duyệt.
3. **`RUNTIME_RESULT`:** kiểm thử NETLOAD và tương tác thực tế trên đúng phiên bản CAD; độc lập với compile và QA tĩnh.
4. **`RELEASE_APPROVED`:** chỉ được ghi `YES` khi các yêu cầu bắt buộc **của phạm vi bàn giao** đã PASS, build của các target bắt buộc PASS và runtime cần nghiệm thu đã PASS. Setup/QA cấu hình hệ thống cũng chỉ yêu cầu tài nguyên của các chức năng triển khai; không lấy điều kiện đủ năm thư mục hoặc entry root làm cổng chặn. Hợp đồng portable riêng phải được QA theo đúng module, không tự đổi vị trí settings. Nếu chỉ giao source chờ chạy trên CAD thực, phải ghi `NOT_VERIFIED`, không tự tuyên bố đã nghiệm thu.
5. **Sai màu CMD:** nếu biên dịch/publish đã thành công thì `BUILD_RESULT` vẫn phản ánh đúng là PASS; riêng mục hiển thị màu `FAIL/NotVerified` làm **chưa đạt điều kiện phát hành**, không được bỏ qua chỉ vì DLL đã tạo thành công. Trường hợp cố ý `NO_COLOR` là ca kiểm tra tương thích và không phải lỗi màu.
6. Kèm theo bản giao biên bản ngắn: `RequirementID | Applies | Result | Evidence | Host | Notes`; file này thuộc **Userdata/log package phần mềm khi phù hợp**, không chèn bản sao bộ tiêu chuẩn `.md` vào ZIP phần mềm. Không đặt file rác Test/Tests vào ZIP bàn giao.

- Kiểm tra cập nhật GitHub theo STD-17 AUTO-01–AUTO-13: chọn baseline gần nhất khi không có chỉ định; version mới mặc định max đã lưu + 1; không ghi đè bản cũ; README module và DCAD có trạng thái mới nhất/đã nghiệm thu riêng; đọc lại ZIP/changelog/metadata sau commit. Chỉ lưu bản gốc hoặc metadata không tự tăng version phần mềm.

- Khi AI phân tích/viết/sửa/rà soát code, kiểm tra **STD-18, AI-01–AI-12** theo phạm vi; không gán PASS khi thiếu bằng chứng.

- Với module **có tích hợp hoặc quản lý License DCAD**, kiểm tra **STD-19 / LIC01–LIC26** theo phạm vi; không coi sự tồn tại file ID/manifest/code sinh là bằng chứng feature thực sự bị khóa.
