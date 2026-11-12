# DCAD-STD-03 — Cấu trúc thư mục và đóng gói phần mềm

**Phiên bản:** 1.14  
**Cập nhật:** 12/11/2026  
**Mức áp dụng:** Bắt buộc

## Một cây project cho một CAD mục tiêu

```text
<ProjectName>-AutoCAD2022-Vxxx/
├── bin/                 # DLL thành phẩm
├── log/                 # Nhật ký nhánh
├── Reference/           # Tham chiếu đúng host
├── Runtime/             # Dữ liệu triển khai nguồn
├── icon/                # Tài nguyên ảnh/icon nguồn
├── Buildtime/           # lang.dat và tài nguyên nhúng
├── Userdata/            # BuildInfo, DataManifest, hướng dẫn
├── src/                 # Các module source (khi cần)
├── Build_Release.cmd
├── <ProjectName>.sln
└── <ProjectName>.csproj
```

- Bảy thư mục `bin`, `log`, `Reference`, `Runtime`, `icon`, `Buildtime`, `Userdata` tuân theo cấu trúc phẳng đã chốt. Không đặt source trong các thư mục này.
- `bin/<AssemblyNameHost>.dll` là đầu ra Release mới theo host; `<AssemblyNameHost>` phải kết thúc bằng hậu tố CAD tương ứng (`C22`, `C25`, `G27`), không dùng tên DLL chung không có hậu tố; không giữ DLL cũ để báo build thành công. Không để thư mục con `Debug`, `Release`, `x64`, `net48`, `net8` trong `bin`.
- Với **AutoCAD**, `Reference` dùng ba DLL `accoremgd.dll`, `acdbmgd.dll`, `acmgd.dll` đúng đời CAD; đặt `Private=False`; không đưa ba DLL này vào `bin`/gói runtime.
- Với **GstarCAD 2027**, sử dụng SDK và Reference của đúng GstarCAD 2027; không copy bộ tham chiếu Autodesk sang.
- **Phạm vi CAD được hỗ trợ:** AutoCAD 2022, AutoCAD 2025 và GstarCAD 2027. Chỉ tạo và đóng gói các cây project thuộc target đã được duyệt trong danh sách này. Không tạo, phục hồi hoặc giữ lại cây source, SDK/Reference, `.sln/.csproj`, DLL, cấu hình build, script, log hay manifest dành riêng cho target ngoài phạm vi giao của dự án.
- `Runtime` chỉ là nguồn đóng gói dữ liệu; dữ liệu chạy thực tế ở `DData`, `DUser`, `DProgram`, `DHelp`, `Dicon` theo DataManifest. `lang.dat` được nhúng DLL từ `Buildtime`. Các thư mục vận hành DCAD phải được đăng ký trực tiếp theo DCAD-STD-05; không sử dụng đường dẫn root để suy ra thư mục con.

## Cấu hình môi trường dữ liệu CAD

**Setup/QA và module CAD** kiểm các tài nguyên thật sự sử dụng trong phạm vi cấu hình/chức năng theo `DataManifest`; không yêu cầu đủ năm thư mục nếu không dùng. Entry DIRECT phải có tên thư mục cuối là `DData`, `DUser`, `DProgram`, `Dicon` hoặc `Dhelp` (không phân biệt hoa/thường trên Windows); không yêu cầu thứ tự/vị trí, liền kề, cùng root hay entry root `DCAD`. `DUser\abc` không phải entry DUser; chỉ đăng ký root cũng không hợp lệ thay DUser. Đường dẫn con ứng dụng phải khai báo riêng, ví dụ từ DProgram DIRECT đến `Dnest\Dnest.exe`; không tìm đệ quy hoặc chấp nhận entry con thay entry logic. Áp dụng STD-05 hiện hành. Đây là đường dẫn runtime, khác cây `Runtime`, `icon`, `Buildtime` của source.

## ZIP phần mềm đa phiên bản

```text
<ProjectName>-Vxxx.zip
├── Build_All.cmd
├── <ProjectName>-AutoCAD2022-Vxxx/
├── <ProjectName>-AutoCAD2025-Vxxx/
├── <ProjectName>-GstarCAD2027-Vxxx/
├── log/               # Tạo khi chạy build
└── Dprogram/          # DLL portable đặt trực tiếp trong thư mục này; chỉ publish khi các target đã duyệt PASS
```

- Một ZIP duy nhất cho mỗi lần bàn giao. Các nhánh độc lập cùng version, không trộn framework/reference/source giữa các host. Baseline mặc định là bản gần nhất và version đích mặc định tăng đúng 1 theo STD-01/17; đồng bộ version ZIP/thư mục/project/BuildInfo/changelog, không ghi đè bản cũ.
- Không đóng thư mục Tests/Test, file tạm và dữ liệu QA rác vào gói bàn giao; vẫn giữ đủ source, tham chiếu hợp lệ và dữ liệu runtime cần thiết. Với SDK native dùng chung, tham chiếu được cung cấp tại máy build theo STD-04 thay vì chép vào ZIP.
- `Dprogram` ở root ZIP là **đích xuất DLL portable dạng phẳng** (`Dprogram/<AssemblyNameHost>.dll`), **không** tự thêm thư mục `AutoCAD2022/`, `AutoCAD2025/` hoặc `GstarCAD2027/` bên trong nếu không có chuẩn module riêng đã duyệt. Không tự coi đây là `DProgram` runtime của Support File Search Path.
- Quy chuẩn `.md` nằm trong **ZIP tiêu chuẩn DCAD**, không chèn vào ZIP phần mềm.


## Quy ước tên DLL và kết cấu theo mẫu CAD (BẮT BUỘC)

Mỗi module đặt **một tên gốc ổn định** cho `Module`, ZIP và thư mục project. Tên đầu ra DLL phải có **hậu tố target**, khớp tuyệt đối trong project, build script, log, DataManifest/BuildInfo và tài liệu NETLOAD. Mẫu cho module `DProject`:

| Target đã duyệt | Tên nhánh | AssemblyName / output trong `bin` | DLL portable sau build PASS |
|---|---|---|---|
| AutoCAD 2022 | `DProject-AutoCAD2022-V2` | `DProjectC22.dll` | `Dprogram/DProjectC22.dll` |
| AutoCAD 2025 (chỉ sau duyệt) | `DProject-AutoCAD2025-Vn` | `DProjectC25.dll` | `Dprogram/DProjectC25.dll` |
| GstarCAD 2027 (chỉ sau duyệt) | `DProject-GstarCAD2027-Vn` | `DProjectG27.dll` | `Dprogram/DProjectG27.dll` |

- **Không nhầm tên lệnh CAD** (`DPROJECT`) với tên DLL; tên lệnh không thay đổi khi DLL có hậu tố host.
- Cấu trúc một nhánh bắt buộc đủ `bin`, `log`, `Reference`, `Runtime`, `icon`, `Buildtime`, `Userdata`, `src`, `Build_Release.cmd`, `.sln`, `.csproj`. Thư mục rỗng có thể được giữ bằng entry ZIP directory; không đặt DLL giả hoặc DLL build cũ để giữ thư mục.
- `Reference` chỉ chứa thư viện đúng bản CAD nếu có quyền đóng gói; khi chưa có SDK thật, không thay bằng DLL dựng giả. Phần nguồn có thể kèm hướng dẫn lấy Reference, nhưng **Reference QA/Build/NETLOAD phải là NotVerified/FAIL đúng thực tế**, không gọi là PASS.
- Khi chỉ được duyệt AutoCAD 2022: **chỉ tạo một nhánh 2022**. Root `Build_All.cmd` và `Dprogram` chỉ xử lý `DProjectC22.dll`; không tạo nhánh hoặc DLL 2025/G27 trước khi được duyệt.

## Cổng QA cấu trúc và tên DLL trước khi phát hành

1. Đọc `.csproj` để lấy `AssemblyName` thật và đường dẫn `OutputPath`; không suy diễn chỉ từ tên ZIP/tên lệnh.
2. Đối chiếu chính xác (kể cả tên target/suffix) với `Build_Release.cmd`, `Build_All.cmd`, `BuildInfo`, lệnh NETLOAD, staging và `Dprogram`.
3. **Dừng đóng gói** nếu còn chuỗi chỉ DLL cũ, nếu `Dprogram` thêm thư mục target trái phép, hoặc một tên DLL xuất hiện tại nhiều vị trí khác nghĩa.
4. Mở lại ZIP vừa xuất, kiểm cây thư mục và nội dung các file CMD, không chỉ kiểm cây source trước khi nén.
5. Kiểm chứng trên Windows cho Build và NETLOAD; nếu chưa chạy thật, ghi **NotVerified**, tuyệt đối không quảng bá PASS.

## Điều kiện nghiệm thu cấu trúc / phạm vi CAD

- Kiểm tra cây file trong ZIP thực tế: chỉ có nhánh host đã duyệt, tên nhánh và version đúng; không lẫn Reference hoặc DLL của host khác.
- Kiểm tra đồng thời cây thư mục, tên project, `.sln/.csproj`, target build, SDK/Reference, DLL, log và manifest. Không chấp nhận tình trạng xóa cây host ngoài phạm vi nhưng còn giữ cấu hình hoặc đầu ra phục vụ host đó.
- Kiểm tra đầu ra DLL, cấu trúc thư mục phẳng và tài liệu runtime bằng kiểm tra nội dung ZIP; kết quả lưu trong biên bản DCAD-STD-16.

## Bổ sung 12/11/2026 — SDK native dùng chung ở giai đoạn build

- **Áp dụng riêng cho ARX/GRX native khi người dùng đã cung cấp thư mục SDK dùng chung.** Giữ nguyên bộ SDK/cây `inc`, `lib-x64`, `arx/inc`... của nhà phát hành trong thư mục máy build; ZIP source không cần chứa bản sao SDK lớn. Thư mục logic `Reference` của nhánh được phép để rỗng trong ZIP nhằm bảo toàn cấu trúc nguồn.
- Mặc định tra cứu dưới `I:\ABC\Reference\Autocad 2022`, `I:\ABC\Reference\Autocad 2025`, `I:\ABC\Reference\GstarCAD 2027`; root có thể cấu hình rõ trên máy build. Tên thư mục CAD là phạm vi tìm kiếm, gốc SDK có thể nằm nhiều tầng bên dưới.
- Build phải kiểm tra **một bộ SDK hoàn chỉnh và đúng host** trước khi compiler chạy; không ghép `.h`/`.lib` từ nhiều bộ, không fallback sang SDK host khác. ZIP không nhúng SDK là **source/chờ build** nếu chưa có bằng chứng build; không gọi thành phẩm đã biên dịch.
- `Reference` của **C# managed** giữ hợp đồng thư viện host chính xác và không sao chép DLL CAD tham chiếu vào `bin` hay triển khai runtime. SDK native máy build khác năm thư mục dữ liệu/Support Path CAD tại STD-05.
