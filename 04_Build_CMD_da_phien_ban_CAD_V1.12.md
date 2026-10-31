# DCAD-STD-04 — Build CMD đa phiên bản CAD

**Phiên bản:** 1.12  
**Cập nhật:** 21/10/2026 — thêm Build CMD byte-level QA và bắt buộc đối chiếu tên DLL theo host.  
**Mức áp dụng:** Bắt buộc theo target

## Target CAD, framework, reference, Build CMD và version [CORE theo target]

**Hợp đồng:** `DCAD-BUILD-CMD-001` — chỉ ghi Build/NETLOAD PASS khi đã chạy kiểm chứng.

### 1. Target, framework và Reference

| Host/target | Framework theo host | Reference/SDK | Xác minh thực tế |
|---|---|---|---|
| AutoCAD 2022 | .NET Framework 4.8; x64 | 3 Autodesk Managed DLL đúng bản 2022 | Build Release, NETLOAD, smoke test |
| AutoCAD 2025 | .NET 8 Windows; x64 | 3 Autodesk Managed DLL đúng bản 2025 | Build Release, NETLOAD, smoke test |
| GstarCAD 2027 | .NET 8 Windows; x64  | Bộ thư viện GstarCAD 2027 riêng, không dùng DLL Autodesk | Build Release, NETLOAD, smoke test; SDK cụ thể cần đối chiếu với bộ thực tế |

- Một target = một cây source/project độc lập, một `.sln/.csproj`, framework và Reference đúng host, `Debug|x64` và `Release|x64`. Không trộn nhiều host/reference chung một project.
- **Phạm vi target bắt buộc:** Chỉ hỗ trợ AutoCAD 2022, AutoCAD 2025 và GstarCAD 2027. Danh sách nhánh trong `Build_All.cmd` phải khớp các target thuộc phạm vi dự án đã được duyệt; không bổ sung nhánh `Build_Release.cmd`, mục `:TARGET`, SDK/Reference, DLL, log hoặc output portable của host ngoài phạm vi.
- Khi port: chốt baseline đã duyệt → tạo cây target mới → thay framework, reference, project plumbing tối thiểu → **BUILD FIRST** → phân loại lỗi thực → sửa đúng nguyên nhân → build lại → regression. Không biến lỗi build thành lý do thay đổi Geometry/XData/Settings/UI.
- Các nhánh cùng **số version bàn giao** trong một ZIP tổng, nhưng `Build`, `NETLOAD` và `Smoke Test` báo độc lập `PASS/FAIL/NotVerified`.

### 2. Cấu trúc script và điểm chạy duy nhất

```text
<ProjectName>-Vxxx/
├─ Build_All.cmd                         # điểm chạy duy nhất ở gốc ZIP
├─ <ProjectName>-AutoCAD2022-Vxxx/
│  └─ Build_Release.cmd
├─ <ProjectName>-AutoCAD2025-Vxxx/
│  └─ Build_Release.cmd
├─ <ProjectName>-GstarCAD2027-Vxxx/
│  └─ Build_Release.cmd
├─ log/                                  # tạo khi build
├─ Dprogram/                             # DLL portable dạng phẳng, chỉ publish nếu toàn bộ target đã duyệt PASS
└─ FactoryDefaults_<ProjectName>/        # nếu module có factory đã duyệt; không sửa để chữa build
```

- `Build_All.cmd` là **một điểm chạy** khi người dùng nhấn đúp; mỗi nhánh có `Build_Release.cmd` riêng được gọi nội bộ với `/nopause`. Không tự tạo thêm tập script build tay ở root.
- Chỉ đưa những nhánh target thực sự thuộc phạm vi giao của project vào danh sách build. **Khi khi giao đủ 3 target** AutoCAD 2022, AutoCAD 2025, GstarCAD 2027, cả ba target là điều kiện để `BUILD ALL: PASS`.
- Nếu có `FactoryDefaults_<ProjectName>` thì đây là dữ liệu nguồn độc lập, không tự sửa hoặc sinh lại chỉ để sửa lỗi build. Không quy định số file Factory chung cho mọi module.

### 3. Hợp đồng tham số `:TARGET` không được tráo

Cú pháp tham số cố định:

```bat
call :TARGET S22 "AutoCAD 2022" "<ProjectName>-AutoCAD2022-Vxxx" "AutoCAD2022.log"
```

- `%~1`: biến ghi kết quả (`S22`).
- `%~2`: **nhãn hiển thị** (`AutoCAD 2022`), không phải tên thư mục.
- `%~3`: **tên thư mục project thật** (`<ProjectName>-AutoCAD2022-Vxxx`), dùng resolve `Build_Release.cmd` và `bin\<AssemblyNameHost>.dll`.
- `%~4`: tên log của nhánh (`AutoCAD2022.log`).

**HARD RULE:** Không được đảo `%~2` với `%~3`; nếu đảo sẽ gây lỗi không tìm thấy script. Không lấy chuỗi nhãn hiển thị làm đường dẫn file.

### 4. Quy trình kiểm tra và publish an toàn

1. Trước khi gọi script con, kiểm tra `Build_Release.cmd` tồn tại đúng đường dẫn; nếu thiếu, ghi `[FAIL]` và đường dẫn thiếu, **tiếp tục kiểm tra các target còn lại**.
2. Gọi script con với `/nopause`; bắt **mã trả về ngay sau `CALL`** (`ERRORLEVEL`), không đánh giá theo câu `BUILD OK`.
3. Xác nhận có `bin\<AssemblyNameHost>.dll` **mới từ lượt build này**; DLL cũ không chứng minh PASS. Dọn output portable cũ trước build, không công bố kết quả chưa đủ các nhánh.
4. Gom DLL của các nhánh vào **staging**; chỉ publish sang `Dprogram/` khi **tất cả target thuộc phạm vi build** đều PASS và đủ DLL. Nếu copy/publish lỗi, xóa phần công bố dở, không để gói portable trạng thái nửa thành công.
5. `BUILD ALL: PASS` + `ERRORLEVEL=0` **chỉ khi toàn bộ nhánh đã PASS và publish hoàn tất**. Nếu một nhánh FAIL hoặc publish lỗi: `BUILD ALL: FAIL`, `ERRORLEVEL` khác 0.
6. Không thay đổi business logic, tài nguyên người dùng hoặc dữ liệu Factory để sửa build.

### 5. Màu, console, pause và logging

| Trạng thái | Nhãn | Màu khi hỗ trợ |
|---|---|---|
| Bắt đầu nhánh | `[BUILD]` | Xanh dương, ANSI 94 |
| Thành công | `[PASS]`, `BUILD ALL: PASS` | Xanh lá, ANSI 92 |
| Thất bại | `[FAIL]`, `BUILD ALL: FAIL` | Đỏ, ANSI 91 |
| Cảnh báo | `[WARNING]` | Vàng, ANSI 93 |
| Tiêu đề, đường phân cách | Tên module, version, tổng kết | Cyan, ANSI 96 |
| Nội dung thường | Đường dẫn log, hướng dẫn | Màu mặc định |

- **Màu là yêu cầu hiển thị bắt buộc khi môi trường console hỗ trợ**; không phải trang trí tùy chọn. `Build_All.cmd` và `Build_Release.cmd` phải có hàm/đường gọi in trạng thái thống nhất áp dụng cho **tất cả** `[BUILD]`, `[PASS]`, `[FAIL]`, `[WARNING]` và tiêu đề. Không đủ nếu chỉ liệt kê màu trong tài liệu hoặc sử dụng `echo` chữ thuần cho mọi trường hợp.
- Cơ chế lựa chọn phương thức hiển thị: `ANSI` khi terminal đã được xác nhận hỗ trợ escape màu; nếu ANSI không khả dụng, dùng `PowerShell Write-Host -ForegroundColor` khi môi trường thực thi hỗ trợ; chỉ dùng `PLAIN` nếu người dùng đặt `NO_COLOR` hoặc cả hai cách thể hiện màu thực sự không khả dụng. **Không được tự chọn PLAIN để đơn giản hóa code** và không xem `NO_COLOR` là mặc định.
- Mọi lần chọn PLAIN phải hiển thị/ghi lý do (`NO_COLOR` hoặc `COLOR_UNAVAILABLE`); không ghi các escape ANSI vào file log. Nếu console xuất ra tệp/pipe không có khả năng thể hiện màu, ghi log văn bản thuần, còn khả năng màu phải được đánh giá riêng khi chạy trực tiếp.
- Trước khi phát hành phải xác nhận **màu nhìn thấy thực tế** trong một môi trường Windows có hỗ trợ (ANSI hoặc PowerShell); kiểm tra `[BUILD]` xanh dương, `[PASS]` xanh lá, `[FAIL]` đỏ, `[WARNING]` vàng, tiêu đề cyan, reset đúng và chữ vẫn đọc được. Kiểm thử đường lỗi thật hoặc chế độ mô phỏng hiển thị trạng thái **không thay kết quả build**. Không kết luận đạt chỉ vì chuỗi ANSI nằm trong source.
- Phải kiểm thử thêm `NO_COLOR`: chữ thuần đầy đủ, không xuất escape, không thay đổi mã thoát/build logic. Khi cả hai cơ chế tô màu không khả dụng, ghi `COLOR_MODE=PLAIN; REASON=COLOR_UNAVAILABLE` và đánh dấu kết quả kiểm tra màu theo điều kiện môi trường, không giả vờ đã nhìn thấy màu.
- **Bắt buộc với mọi file CMD build** (`Build_All.cmd` và `Build_Release.cmd`) **khi được chạy trực tiếp bằng nhấn đúp hoặc gọi thủ công**: sau khi hiển thị kết quả **PASS hoặc FAIL**, kể cả lỗi thiếu SDK, thiếu script, biên dịch lỗi hoặc publish lỗi, phải in rõ `Nhấn phím bất kỳ để thoát...` (có thể kèm thông điệp tiếng Anh) và thực hiện `pause >nul`. **Tuyệt đối không tự đóng cửa sổ trước khi người dùng nhấn một phím.**
- Chỉ cho phép bỏ phần chờ phím khi CMD **được gọi nội bộ/tự động với đối số `/nopause`**. Trong `Build_All.cmd`, gọi các `Build_Release.cmd /nopause` để không phải nhấn phím sau mỗi nhánh; lệnh tổng **vẫn chờ phím ở cuối** nếu chạy trực tiếp.
- Tập trung mọi đường kết thúc (thành công, lỗi sớm, lỗi ngoại lệ được xử lý) về **một nhãn `:FINISH`**. Không sử dụng `exit`, `exit /b`, `goto :EOF` hoặc nhánh trả về khác **trước bước in kết quả và chờ phím**. Chờ phím phải đặt sau phần tổng kết và đường dẫn log, không được che mất lỗi.
- **Giữ nguyên mã lỗi build:** lưu mã PASS/FAIL vào một biến riêng trước bước `pause`. Kết thúc bằng `exit /b` với mã đã lưu, không dùng `ERRORLEVEL` do `pause`, `echo` hoặc lệnh hiển thị màu tạo ra để quyết định kết quả.

**Mẫu kết thúc bắt buộc (chỉ minh họa pause/mã thoát; không phải implementation màu):**

```bat
@echo off
setlocal EnableExtensions
set "BUILD_EXIT_CODE=0"
set "NO_PAUSE=0"
if /I "%~1"=="/nopause" set "NO_PAUSE=1"

rem Thuc hien cac buoc build; khi loi:
rem set "BUILD_EXIT_CODE=1"
rem goto :FINISH
rem Khong dung exit/goto :EOF som bo qua FINISH.

:FINISH
if "%BUILD_EXIT_CODE%"=="0" (
    echo [PASS] BUILD
) else (
    echo [FAIL] BUILD
)
rem Hien thi trang thai qua ham in mau chung da duoc kiem chung.
rem In tong ket va vi tri log truoc khi doi phim.
if "%NO_PAUSE%"=="0" (
    echo.
    echo Nhan phim bat ky de thoat...
    pause >nul
)
endlocal & exit /b %BUILD_EXIT_CODE%
```

*Lưu ý:* Nội dung mẫu dùng ASCII để CMD Windows hiển thị an toàn; giao diện tiếng Việt có dấu được phép nếu code page/encoding đã được kiểm thử. Mẫu chỉ mô tả **đoạn kết thúc** và cố ý chưa triển khai hàm màu; **không được sao chép nguyên mẫu này rồi tuyên bố đạt quy chuẩn về màu**. Phải bổ sung hàm hiển thị màu theo hợp đồng phía trên và kiểm chứng trực tiếp trước khi phát hành.
- Root `log/` lưu `Build_All.log`, `AutoCAD2022.log`, `AutoCAD2025.log`, `GstarCAD2027.log` tương ứng các target có tham gia; script con được phép thêm log. **Log là văn bản thuần, không chứa ANSI escape**.
- Bảng tổng kết cuối màn hình phải hiển thị **riêng trạng thái từng target**, sau đó `BUILD ALL: PASS/FAIL` và đường dẫn log. Không công bố `Build PASS`, `NETLOAD PASS` hay kiểm thử Windows PASS nếu thực tế chỉ mới rà tĩnh.

### 6. Build Gate và QA bắt buộc

- **Hai kết quả riêng:** `BUILD_RESULT` phản ánh compiler/DLL/publish; `BUILD_UI_COMPLIANCE` phản ánh màu, thông điệp, pause, tổng kết và log. Không làm giả `BUILD_RESULT=FAIL` khi chỉ lỗi màu, nhưng **không công bố `RELEASE_APPROVED=YES` nếu UI bắt buộc FAIL/NotVerified**.
- Đối chiếu mã nguồn **và** kiểm thử console thực tế: có hàm màu dùng được trên `Build_All.cmd` và các script nhánh; khi có hỗ trợ màu không được tự rơi về chữ thuần; kiểm thử cơ chế dự phòng PowerShell, `NO_COLOR` và không gây mất mã lỗi build.
- Ghi bằng chứng kiểm thử màu: môi trường Windows/console, phương thức màu, `[BUILD]/[PASS]/[FAIL]/[WARNING]`, khả năng quan sát các màu, trạng thái `PASS/FAIL/NotVerified`. Log build văn bản thuần **không đủ làm bằng chứng màu**.
- Kiểm tra tham số `%~3` chính xác với tên thư mục trên đĩa; `%~4` phải đúng tên log.
- Chạy `Build_All.cmd` trên Windows có SDK phù hợp; thử thành công, thiếu `Build_Release.cmd`, từng target FAIL riêng, DLL cũ còn sót, thiếu/copy lỗi DLL và chế độ `NO_COLOR`. **Với mỗi trường hợp PASS/FAIL, kiểm tra lời nhắc nhấn phím bất kỳ xuất hiện, cửa sổ không tự đóng và chỉ thoát sau khi nhấn phím.**
- Chạy trực tiếp từng `Build_Release.cmd` ở cả trạng thái PASS và FAIL: có lời nhắc chờ phím; khi `Build_All.cmd` gọi nhánh với `/nopause`: **không chờ giữa các nhánh**, chỉ `Build_All.cmd` chờ ở cuối. Kiểm tra cả trường hợp **lỗi phát sinh trước bước build chính**.
- Sau khi nhấn phím, **mã thoát phải giữ đúng**: PASS trả `0`, FAIL trả khác `0`; hành vi chờ phím không được biến lỗi thành PASS.
- Rà danh sách `:TARGET`, script, output DLL, log và manifest: chỉ chứa target đã được duyệt thuộc AutoCAD 2022, AutoCAD 2025, GstarCAD 2027; không còn cấu hình hoặc artifact của target ngoài phạm vi.
- Mỗi nhánh độc lập ghi `Source QA`, `Reference`, `Release Build`, `DLL`, `NETLOAD`, `Smoke Test`: `PASS/FAIL/NotVerified`; không lấy kết quả một host suy ra cho host khác.
- Kiểm tra đủ số lượng và version DLL portable; xác nhận log root đầy đủ, return code thất bại khác 0, không có DLL publish dở.
- **Không đặt file quy chuẩn `.md` vào ZIP phần mềm.** Bộ quy chuẩn có ZIP tài liệu độc lập.


### 7. Cổng kiểm soát file CMD, chống lỗi đọc lệnh và sai tên DLL (BẮT BUỘC)

**Nhóm CMD encoding / line ending:**

1. `Build_All.cmd` và tất cả `Build_Release.cmd` phải lưu đúng **CRLF (`0D 0A`)**, không có `LF` đơn lẻ, không có `CRCRLF`, và **UTF-8 không BOM** (ba byte mở đầu `EF BB BF` bị cấm). Dòng đầu bắt buộc là `@echo off` và kết thúc bằng CRLF.
2. Không dùng hàm ghi text mặc định theo hệ điều hành khi tạo CMD; trình đóng gói phải ghi byte với CRLF rõ ràng hoặc chuẩn hóa sau khi sinh. Kiểm cả **file bên trong ZIP cuối cùng**, không chỉ nguồn trên disk.
3. Kiểm chuỗi ký tự điều khiển: không NUL; tránh ghép lệnh do sai `&`, `(`, `)`, `!`, `%`; kiểm dấu nháy, `goto :FINISH`, `/nopause`, lưu `ERRORLEVEL` ngay sau `CALL`, và một nhánh kết thúc duy nhất cho luồng main. `exit /b` của các subroutine vẫn được phép.
4. **Bắt buộc chạy `cmd.exe` thực tế trên Windows** tối thiểu một ca thiếu Reference/SDK để xác nhận script lỗi gọn, không hiện chuỗi `'SE' is not recognized`, `RINT`, `INISH`, `Unable to initialize device PRN`; không thể dùng kiểm tĩnh thay thế kiểm thử CMD thật.
5. Chạy thêm một ca điều kiện tạo được output hoặc mô phỏng có kiểm soát **đường đi script** để xác minh `:TARGET`, staging, delete stale DLL và đường dẫn `Dprogram` không phát sinh lỗi; không bao giờ dùng mô phỏng để công bố BUILD PASS.
6. Các trường hợp kiểm thử trực tiếp và `/nopause` đều phải xác nhận dòng `Nhấn phím bất kỳ để thoát...`, cửa sổ chờ phím, mã trả về đúng. Kiểm màu và `NO_COLOR` theo phần 5.

**Nhóm tên artifact:**

7. Lấy `AssemblyName` từ `.csproj`. Mọi chỗ tên DLL trong build, output, staging, portable và hướng dẫn NETLOAD phải khớp **chính xác**. Các hậu tố target bắt buộc: `C22` (AutoCAD 2022), `C25` (AutoCAD 2025), `G27` (GstarCAD 2027). Ví dụ DProject chỉ AutoCAD 2022: `bin\DProjectC22.dll` → `Dprogram\DProjectC22.dll`.
8. `Dprogram` portable có cấu trúc **phẳng**, không thêm `AutoCAD2022/` hay thư mục target khác nếu không có quy ước module đặc biệt đã được duyệt; không giữ DLL cũ hoặc tên chung `<ProjectName>.dll` khi module dùng suffix host.
9. Trước khi nén, và sau khi nén mở ZIP để đối chiếu: thư mục nhánh, `Build_Release.cmd`, `.csproj`, `BuildInfo`, tên DLL, tham số `:TARGET`, `log` và `Dprogram`. Sai bất kỳ mắt xích nào → **PACKAGING QA FAIL, không bàn giao**.
10. Với module đang chỉ duyệt một target, chỉ test/publish target đó và ghi rõ **one approved target**; không xuất hiện tên project, output, Reference hay log của host chưa được duyệt.

**Trạng thái riêng:** `CMD_BYTES_QA`, `CMD_WINDOWS_QA`, `ARTIFACT_NAME_QA`, `PACKAGE_TREE_QA`, `BUILD_RESULT`, `NETLOAD` và `SMOKE_TEST` phải báo riêng PASS/FAIL/NotVerified. `CMD_BYTES_QA=PASS` không suy ra `CMD_WINDOWS_QA=PASS` hoặc `BUILD_RESULT=PASS`.

