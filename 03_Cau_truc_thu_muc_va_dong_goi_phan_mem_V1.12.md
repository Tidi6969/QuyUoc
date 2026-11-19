# DCAD-STD-03 — Cấu trúc thư mục và đóng gói phần mềm

**Phiên bản:** 1.16  
**Cập nhật:** 19/11/2026  
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
- `bin/<ProjectName>.dll` (managed), `bin/<ProjectName>.arx` (native AutoCAD) hoặc `bin/<ProjectName>.grx` (native GstarCAD) là thành phẩm mới trong nhánh host riêng; **không thêm hậu tố** `C22`, `C25`, `G27` hoặc mã host tương tự vào tên gốc. Không giữ output cũ để báo build PASS; không tạo thư mục con `Debug`, `Release`, `x64`, `net48`, `net8` trong `bin`.
- Với **AutoCAD**, managed dùng `accoremgd.dll`, `acdbmgd.dll`, `acmgd.dll` đúng đời CAD, `Private=False`; nguồn tra cứu mặc định là Reference chung **đúng host** tại máy build (xem cuối tài liệu), có thể chứa nhiều tầng thư mục. Không đóng gói các DLL CAD tham chiếu vào `bin`/runtime.
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
└── Dprogram/          # Thành phẩm portable phân nhánh theo host; chỉ publish khi tất cả target được duyệt PASS
```

- Một ZIP duy nhất cho mỗi lần bàn giao. Các nhánh độc lập cùng version, không trộn framework/reference/source giữa các host. Baseline mặc định là bản gần nhất và version đích mặc định tăng đúng 1 theo STD-01/17; đồng bộ version ZIP/thư mục/project/BuildInfo/changelog, không ghi đè bản cũ.
- Không đóng thư mục Tests/Test, file tạm và dữ liệu QA rác vào gói bàn giao; vẫn giữ đủ source, tham chiếu hợp lệ và dữ liệu runtime cần thiết. Với SDK native dùng chung, tham chiếu được cung cấp tại máy build theo STD-04 thay vì chép vào ZIP.
- `Dprogram` trong ZIP **phân thư mục theo host**: `Dprogram/AutoCAD2022/`, `Dprogram/AutoCAD2025/`, `Dprogram/GstarCAD2027/` (chỉ host được duyệt), tên file thành phẩm **giữ nguyên basename module** bên trong từng thư mục. Không gộp các binary khác host vào root để tránh ghi đè. Khi cài đặt, chỉ chép file thuộc host đang dùng vào `DProgram` runtime dạng phẳng; thư mục host trong ZIP không phải Support File Search Path của CAD.
- Quy chuẩn `.md` nằm trong **ZIP tiêu chuẩn DCAD**, không chèn vào ZIP phần mềm.


## Quy ước tên file thành phẩm theo mẫu CAD (BẮT BUỘC)

Mỗi module giữ **một tên gốc ổn định** cho Module, ZIP và project. Output có dạng `<ProjectName>.dll` (managed), `<ProjectName>.arx` (native AutoCAD) hoặc `<ProjectName>.grx` (native GstarCAD); nếu có phần mở rộng khác do module đã duyệt thì giữ extension đó. **Không thêm** `C22`, `C25`, `G27`, mã CAD khác hoặc version vào basename. `AssemblyName` trong `.csproj` hoặc `TargetName` trong project native phải khớp tên gốc và đồng bộ build script, BuildInfo, DataManifest, lệnh nạp add-in.

| Target được duyệt | Thư mục project | Managed output trong `bin` | Thành phẩm portable |
|---|---|---|---|
| AutoCAD 2022 | `DProject-AutoCAD2022-Vn` | `DProject.dll` | `Dprogram/AutoCAD2022/DProject.dll` |
| AutoCAD 2025 | `DProject-AutoCAD2025-Vn` | `DProject.dll` | `Dprogram/AutoCAD2025/DProject.dll` |
| GstarCAD 2027 | `DProject-GstarCAD2027-Vn` | `DProject.dll` | `Dprogram/GstarCAD2027/DProject.dll` |

- Với native, file tương ứng là `DProject.arx` (AutoCAD 2022/2025) và `DProject.grx` (GstarCAD 2027), trong đúng thư mục host như bảng. Không tự đổi module managed thành native hoặc ngược lại.
- **Ba file cùng tên có thể là ba binary khác nhau**. Không được ghi đè, thay thế chéo, hoặc nạp sai host; tên command CAD (ví dụ `DPROJECT`) độc lập với tên file.
- Một nhánh bắt buộc đủ `bin`, `log`, `Reference`, `Runtime`, `icon`, `Buildtime`, `Userdata`, `src`, `Build_Release.cmd`, `.sln`, `.csproj` (hoặc cấu hình project native tương ứng). Thư mục rỗng có thể được giữ bằng entry ZIP directory, không đặt binary giả/cũ.
- `Reference` chỉ chứa đúng thư viện host nếu được phép đóng gói; khi dùng thư viện SDK chung máy build, thư mục này có thể rỗng trong ZIP. Không thay SDK thiếu bằng DLL giả; Reference/Build/NETLOAD phải phản ánh `NotVerified/FAIL` đúng thực tế.
- Khi chỉ duyệt AutoCAD 2022, chỉ tạo nhánh 2022 và `Dprogram/AutoCAD2022/<ProjectName>.<ext>`; không tạo nhánh, thư viện hoặc binary của host khác.

## Cổng QA cấu trúc và tên thành phẩm trước khi phát hành

1. Đọc `.csproj` hoặc project native để lấy `AssemblyName`/`TargetName`, phần mở rộng và `OutputPath` thực; không suy diễn chỉ từ ZIP hay command.
2. Đối chiếu chính xác basename **không hậu tố CAD**, phần mở rộng và thư mục host với `Build_Release.cmd`, `Build_All.cmd`, `BuildInfo`, lệnh NETLOAD/APPLOAD, staging và `Dprogram`.
3. **Dừng đóng gói** nếu còn output/đường dẫn dùng tên có hậu tố host, nếu hai bản CAD bị ghi đè trong cùng thư mục portable, hoặc nếu thư mục host sai. Cùng basename ở **hai thư mục host khác nhau** là đúng quy ước.
4. Mở lại ZIP vừa xuất, kiểm cây thư mục và nội dung các file CMD, không chỉ kiểm cây source trước khi nén.
5. Kiểm chứng trên Windows cho Build và NETLOAD; nếu chưa chạy thật, ghi **NotVerified**, tuyệt đối không quảng bá PASS.

## Điều kiện nghiệm thu cấu trúc / phạm vi CAD

- Kiểm tra cây file trong ZIP thực tế: chỉ có nhánh host đã duyệt, tên nhánh và version đúng; không lẫn Reference hoặc DLL của host khác.
- Kiểm tra đồng thời cây thư mục, tên project, `.sln/.csproj`, target build, SDK/Reference, DLL, log và manifest. Không chấp nhận tình trạng xóa cây host ngoài phạm vi nhưng còn giữ cấu hình hoặc đầu ra phục vụ host đó.
- Kiểm tra output trong `bin` của từng host, cấu trúc `Dprogram/<Host>/` của ZIP, tên thành phẩm và tài liệu triển khai runtime; kết quả lưu trong biên bản DCAD-STD-16.

## Bổ sung 12/11/2026 — SDK native dùng chung ở giai đoạn build

- **Áp dụng riêng cho ARX/GRX native khi người dùng đã cung cấp thư mục SDK dùng chung.** Giữ nguyên bộ SDK/cây `inc`, `lib-x64`, `arx/inc`... của nhà phát hành trong thư mục máy build; ZIP source không cần chứa bản sao SDK lớn. Thư mục logic `Reference` của nhánh được phép để rỗng trong ZIP nhằm bảo toàn cấu trúc nguồn.
- Root Reference mặc định trên máy build cho **managed và native**: `D:\000\Reference\Autocad 2022`, `D:\000\Reference\Autocad 2025`, `D:\000\Reference\GstarCAD 2027`. Chỉ override bằng cấu hình tường minh có log. Tìm sâu trong phạm vi host để xác định DLL managed hoặc gốc SDK native; không fallback sang thư viện của CAD khác.
- Build phải kiểm tra **một bộ SDK hoàn chỉnh và đúng host** trước khi compiler chạy; không ghép `.h`/`.lib` từ nhiều bộ, không fallback sang SDK host khác. ZIP không nhúng SDK là **source/chờ build** nếu chưa có bằng chứng build; không gọi thành phẩm đã biên dịch.
- `Reference` của **C# managed** giữ hợp đồng thư viện host chính xác và không sao chép DLL CAD tham chiếu vào `bin` hay triển khai runtime. SDK native máy build khác năm thư mục dữ liệu/Support Path CAD tại STD-05.

## Cập nhật 19/11/2026 — Bỏ hậu tố CAD khi build

- Phân biệt host bằng thư mục project và portable, không bằng basename DLL/ARX/GRX; giữ output đúng module và không ghi đè giữa các host.
- `Dprogram/<Host>/` là cấu trúc **đóng gói**; `DProgram` runtime vẫn dạng phẳng khi cài đúng file theo CAD. Chỉ sửa tài liệu, không tự nhận đã đổi tên/built binary hiện hữu.
