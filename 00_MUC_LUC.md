# BỘ TIÊU CHUẨN DCAD — V1.21

## Cập nhật trực tuyến 26/11/2026 — [STD-20 V1.2](20_Quy_uoc_ban_giao_key_Addin.md)

- Chữ ký RSA-PSS của bộ bàn giao, xác thực một lần khi tiếp nhận; thay thế cơ chế phê duyệt máy build V1.1. Giữ sáu file, Gate và Licensecc. Ngày 26/11/2026 chủ dự án xác nhận TidiDcad V55 chạy OK ngay lần đầu (`UserReportedWorking=YES`); log build và kiểm thử ký/xác minh Windows, nghiệm thu tích hợp Add-in/CAD riêng vẫn `NotVerified` theo chứng cứ độc lập. [Tiến độ DCAD](https://github.com/Tidi6969/DCAD/blob/main/README.md). ZIP tổng V1.17 giữ nguyên.

## Cập nhật trực tuyến 26/11/2026 — STD-20 V1.1

- [STD-20 v1.1](20_Quy_uoc_ban_giao_key_Addin.md): phê duyệt KeyId một lần qua file công khai độc lập, preflight thống nhất và kiểm soát log/artifact từng build. Giữ ZIP Integration sáu file, hợp đồng Gate hiện tại và schema ToolInfo 1.0. Không tự coi mã nguồn V54 hay các Add-in là đã nghiệm thu Windows/CAD.
- ZIP tổng V1.17 giữ nguyên; phiên bản trực tuyến V1.21.

## Cập nhật trực tuyến 25/11/2026 — Bàn giao Key Add-in

- Bổ sung **DCAD-STD-20 v1.0**: cấu trúc ZIP đúng sáu file, hợp đồng tích hợp Gate, hướng dẫn project/Build_All/Build_Release/README và ma trận nghiệm thu. Phạm vi đã khảo sát: native C++ x64, AutoCAD 2022; các target khác cần hợp đồng và nghiệm thu tương ứng.
- STD-19 vẫn đã rút. Chỉ cập nhật tài liệu trực tuyến, ZIP tổng V1.17 giữ nguyên; không sửa hoặc nghiệm thu phần mềm.

## Cập nhật trực tuyến 24/11/2026 — Bổ sung quy chuẩn hộp thoại native

- Thêm **DCAD-STD-13B / DCAD-NATIVE-UI-001 v1.0**, dựa trên Dtscr V118: kiến trúc bắt buộc, tính năng có điều kiện và bố cục linh động; không thay STD-13/13A và không áp dụng hồi tố hàng loạt.
- Phiên bản mục lục trực tuyến V1.18; trạng thái phát hành ZIP tổng được quản lý riêng theo STD-17.

## Cập nhật trực tuyến 20/11/2026 — Quy tắc bảng dữ liệu dùng chung

- Bổ sung `DCAD-UI-CONTRACT-001` (mục 13A): Form ID, Control ID, schema 2, dữ liệu delta và adapter C#/C++; chỉ áp dụng khi module tích hợp UI Studio. Không sửa các thuật toán CAD.

- STD-05 v1.17: hợp đồng `DCAD-SQLITE-GRID-001`, bắt buộc SQLite map bản ghi/cột theo khóa, kiểm tra từng ô, không dùng chuỗi TAB/CSV làm bảng, ghi có transaction và không xóa-chèn hàng loạt để sửa một ô; không thay hợp đồng nạp `DProgram\\sqlite3.dll` dùng chung.
- STD-13 v1.8: hợp đồng `DCAD-GRID-001` áp dụng cho **mọi module và mọi bảng có thể chỉnh sửa**, không phụ thuộc SQLite hay công nghệ control cụ thể.
- STD-16 v1.20: bộ test GRID01–GRID10 và SQL09–SQL15, tách QA logic, UI/CAD runtime và Build.
- Đây là cập nhật tài liệu trực tuyến; ZIP tổng đã lưu V1.17 chưa chứa nội dung mới. Không tự thay đổi hay nghiệm thu app CAD.


**Phạm vi CAD hỗ trợ:** AutoCAD 2022, AutoCAD 2025 và GstarCAD 2027. Mỗi dự án triển khai các target đã được duyệt trong danh sách này.  
**Bàn giao:** Một file ZIP chứa các file Markdown độc lập, không chứa bản sao tài liệu nguồn hoặc file phụ trợ.  
**Cách sử dụng:** Áp dụng quy chuẩn chung cho mọi project; tài nguyên của từng module được kiểm tra theo `DataManifest`. Setup/QA hệ thống kiểm tra cấu hình DCAD đầy đủ, độc lập với kiểm tra khởi động module.

## Danh mục

| Mã | Quy chuẩn | Áp dụng |
|---|---|---|
| 01 | [Quy trình phát triển và quản lý phiên bản](01_Quy_trinh_phat_trien_va_quan_ly_phien_ban.md) | Bắt buộc |
| 02 | [Tổ chức mã nguồn, module và command](02_To_chuc_ma_nguon_module_va_command.md) | Bắt buộc |
| 03 | [Cấu trúc thư mục và đóng gói phần mềm](03_Cau_truc_thu_muc_va_dong_goi_phan_mem_V1.12.md) | Bắt buộc |
| 04 | [Build CMD đa phiên bản CAD](04_Build_CMD_da_phien_ban_CAD_V1.12.md) | Bắt buộc theo target |
| 05 | [Support File Search Path và đọc dữ liệu](05_Support_File_Search_Path_va_doc_du_lieu_V1.11.md) | Bắt buộc khi đọc dữ liệu |
| 06 | [Lưu settings và Reset Factory](06_Luu_settings_va_Reset_Factory.md) | Theo tính năng |
| 07 | [Đa ngôn ngữ và Unicode](07_Da_ngon_ngu_va_Unicode.md) | Bắt buộc khi có giao diện hoặc text |
| 08 | [Command, selection và ESC](08_Command_selection_va_ESC.md) | Bắt buộc theo thao tác |
| 09 | [Hình học, layer, text và DIM](09_Hinh_hoc_layer_text_va_DIM.md) | Theo tính năng |
| 10 | [Danh mục HoleType và ý nghĩa hình học](10_Danh_muc_HoleType_va_y_nghia_hinh_hoc.md) | Theo tính năng |
| 11 | [Gap, Dia, Depth và Offset](11_Gap_Dia_Depth_va_Offset.md) | Theo tính năng |
| 12 | [XData/XRecord và khôi phục hình học dùng chung](12_Ghi_doc_XData_va_XRecord.md) | Bắt buộc khi module ghi/đọc XData hoặc tham gia Drestore |
| 13 | [Giao diện, icon và Preview](13_Giao_dien_icon_va_Preview.md) | Theo tính năng |
| 13A | [DCAD UI Contract — Đồng bộ bố cục C# / C++](13A_DCAD_UI_Contract_CSharp_CPP.md) | Bắt buộc khi dùng DCAD UI Studio |
| 13B | [DCAD Native Dialog — Kiến trúc và trình bày hộp thoại](13B_Quy_chuan_giao_dien_hop_thoai_Native_DCAD.md) | Bắt buộc khi tạo mới/thiết kế lại hộp thoại native; thành phần theo tính năng |
| 14 | [Transaction, Undo và hiệu năng](14_Transaction_Undo_va_hieu_nang.md) | Bắt buộc theo thao tác |
| 15 | [Xử lý lỗi và HARD STOP](15_Xu_ly_loi_va_HARD_STOP.md) | Bắt buộc |
| 16 | [DataManifest, BuildInfo và QA](16_DataManifest_BuildInfo_va_QA.md) | Bắt buộc |
| 17 | [Autoupdate — Tự động build, đồng bộ GitHub và ghi nhận xác nhận OK](17_Autoupdate.md) | Bắt buộc khi cập nhật GitHub hoặc người dùng xác nhận phiên bản OK |
| 18 | [Nguyên tắc AI lập trình theo Karpathy](18_Karpathy_Guidelines_cho_AI_lap_trinh.md) | Bắt buộc khi AI phân tích/viết/sửa/rà soát code |
| 20 | [Bàn giao Key và tích hợp bản quyền Add-in DCAD](20_Quy_uoc_ban_giao_key_Addin.md) | Bắt buộc khi bàn giao bộ tích hợp và tích hợp Add-in theo hợp đồng đã xác nhận |

## Nguyên tắc sử dụng

1. Không sửa code hoặc dữ liệu phần mềm khi chưa có yêu cầu cụ thể.
2. Không suy diễn quy tắc của một project sang project khác nếu schema/nghiệp vụ chưa được xác nhận.
3. Khi khác biệt giữa chuẩn hiện hành và dữ liệu cũ: writer sử dụng canonical schema đã công bố; reader chỉ đọc legacy qua nhánh tương thích.
4. Các trạng thái Build, NETLOAD và runtime phải ghi đúng kết quả đã kiểm chứng; chưa chạy thì để `NotVerified`.
5. Mỗi lần cập nhật phát hành **một ZIP tổng mới**, chỉ gồm các file quy chuẩn `.md` độc lập tại root; số phiên bản gói và phiên bản riêng từng tài liệu được ghi rõ, không buộc tài liệu không sửa phải tăng phiên bản.
6. Các tên `DData`, `DUser`, `DProgram`, `Dicon`, `Dhelp` phải là **thành phần thư mục cuối cùng của đường dẫn đăng ký trực tiếp** trong Support File Search Path; không phải năm entry ở cuối danh sách. Không bắt buộc đủ cả năm, liền kề, cùng thư mục cha hay thứ tự/vị trí cụ thể. Setup/QA và module chỉ yêu cầu các tài nguyên của phạm vi đang kiểm tra theo `DataManifest`; không `FATAL` vì thư mục không dùng. Không tìm xuống từ root/cha hoặc chấp nhận entry kết thúc bằng thư mục con khác. Chỉ phân giải đầu lượt lệnh. Đường dẫn tương đối bên trong thư mục logic, ví dụ `DProgram\Dnest\Dnest.exe`, phải được quy định riêng trong DataManifest; chi tiết STD-05/06/16.

7. Khi sửa/tạo bản phần mềm mới qua ChatGPT: nếu không chỉ định baseline, tự lấy phiên bản gần nhất đã lưu của đúng module trên GitHub; nếu không chỉ định phiên bản đích, tự tăng số phiên bản lớn nhất đã lưu đúng 1 (`V49 → V50`). Không hỏi lại chỉ để chọn số mặc định. Lưu bản mới độc lập, giữ bản cũ và tách bản mới nhất đã lưu khỏi bản đã nghiệm thu; chi tiết STD-01/17. Chỉ lưu bản gốc hoặc sửa tài liệu không tự làm tăng version phần mềm.

## Hiệu lực và kiểm chứng tuân thủ

- Các từ **PHẢI / BẮT BUỘC / KHÔNG ĐƯỢC** là yêu cầu nghiệm thu; **NÊN / KHUYẾN NGHỊ** là định hướng, ngoại lệ phải được chấp thuận; **THEO TÍNH NĂNG** chỉ kích hoạt khi có nghiệp vụ tương ứng. Ví dụ và mẫu code không thay thế yêu cầu nghiệm thu.
- Không được kết luận phần mềm tuân thủ chỉ vì đã đọc tài liệu, có dòng mô tả trong source hoặc build tạo được DLL. Mỗi yêu cầu bắt buộc phải có **điều kiện áp dụng, phương pháp kiểm chứng, bằng chứng và kết quả** theo DCAD-STD-16.
- **Build PASS**, **tuân thủ tiêu chuẩn PASS** và **NETLOAD/runtime PASS** là các kết quả độc lập. Lỗi trình bày CMD không được đổi một kết quả biên dịch thành FAIL, nhưng bản bàn giao chính thức không được tuyên bố đạt chuẩn khi yêu cầu trình bày bắt buộc chưa đạt.
- Có thể bàn giao bản nguồn đang kiểm thử với các mục `NotVerified` được công bố rõ, nhưng không gọi bản đó là đã nghiệm thu đầy đủ. Không được suy diễn tình trạng CAD đích từ kết quả của host khác.
- Mọi nhánh CAD, cấu hình build, SDK/Reference, DLL, log và manifest thuộc phạm vi bàn giao phải khớp danh sách target đã được duyệt theo quy chuẩn 03/04; không đưa target ngoài phạm vi vào gói.
- Phiên bản ZIP tổng là **V1.17**; số phiên bản ở đầu từng file cho biết lần sửa nội dung của chính quy chuẩn đó, nên một số file không sửa vẫn giữ phiên bản cũ.

## Cập nhật V1.12 — 03/11/2026

- Đồng bộ quy tắc tên thư mục cuối, kiểm tra theo phụ thuộc và thư mục con ứng dụng được khai báo.
- Tự chọn baseline gần nhất, tăng phiên bản phần mềm đúng 1 khi không có chỉ định; giữ lịch sử và tách trạng thái nghiệm thu.
- Giữ tên file hiện có để bảo toàn URL; hậu tố phiên bản trong tên file cũ là định danh lịch sử, phiên bản hiệu lực nằm ở đầu nội dung.

## Bổ sung ngày 03/11/2026 — Xác nhận OK và tự đồng bộ GitHub

- Khi người dùng xác nhận phiên bản chạy OK/đạt/duyệt và xác định được module/version, trợ lý phải cập nhật ngay thông tin xác nhận trong repository module và danh mục DCAD, không cần người dùng nhắc lại.
- Chỉ cập nhật metadata nghiệm thu; không tự tăng phiên bản, ghi đè ZIP hoặc gán PASS cho CAD host chưa có bằng chứng. Kiểm tra commit và thông báo trung thực nếu đồng bộ chưa hoàn tất. Chi tiết tại STD-17 mục 9.

## Cập nhật V1.13 — 03/11/2026

- Bổ sung STD-18: Think Before Coding, Simplicity First, Surgical Changes, Goal-Driven Execution.
- Đồng bộ STD-16/QA, mục lục, README, changelog và ZIP tổng mới. Không sửa phần mềm CAD hay tăng version module.

## Cập nhật V1.14 — 09/11/2026

- Bổ sung DCAD-STD-19 v1.0: 8 tầng bản quyền, Tool/Feature Registry động, ToolUid/FeatureUid, manifest, xuất bộ tích hợp, chứng nhận, khóa tin cậy, File ID và License Client.
- Đồng bộ ma trận QA STD-16 (LIC01–LIC26), mục lục, README, changelog và ZIP tổng V1.14; giữ lại ZIP V1.13 và mọi bản cũ.
- Chỉ cập nhật tài liệu; chưa thay đổi hoặc nghiệm thu mã nguồn DCADManage/DLL.

## Cập nhật V1.15 — 10/11/2026

- STD-05 v1.13: PlateCatalog đọc từ `DData` DIRECT, bỏ ngoặc bao ngoài trong UI nhưng giữ DAT gốc.
- STD-09 v1.6: PlateCode → layer Hole Normal `_O`, W/T `_W`, không cắt sai các mã như `PH_P` và không ghi đè màu layer hiện có.
- STD-11 v1.6: chọn layer theo PlateCode đã chuẩn hóa, giữ nguyên công thức Gap/Depth và xử lý A/B.
- Đồng bộ bản lưu Skill D5/D9/D11, README, changelog, ZIP V1.15; không chỉnh mã nguồn CAD. Các bản V1.14 trở về trước được giữ lại.

## Cập nhật V1.16 — 12/11/2026

- STD-03 v1.14: cho phép SDK native dùng chung tại máy build, ZIP nguồn không cần nhúng SDK nặng.
- STD-04 v1.14: tìm SDK đệ quy đúng host, kiểm đủ file trước compiler, phát hiện xung đột SHA, build đủ ARX/GRX.
- STD-17 v1.2: phân biệt gói source/chờ build với release đã biên dịch khi Reference ở ngoài ZIP.
- STD-05 runtime và STD-18 được giữ nguyên; không thay đổi logic phần mềm CAD.

## Cập nhật V1.17 — 15/11/2026

- Chuyển nguồn chọn ngôn ngữ DCAD sang `DUser/Shared_data.Set`: đọc đúng dòng 1, lấy số nguyên không âm trong `[]`, không dùng file cũ hoặc fallback English khi cấu hình sai.
- Đồng bộ STD-05 v1.14, STD-06 v1.12, STD-07 v1.12 và STD-16 v1.15; bổ sung kiểm thử LANG01–LANG08. Giữ nguyên những quy định ngoài phạm vi.
- Chỉ cập nhật tài liệu; không tự sửa mã, build hoặc xác nhận nghiệm thu ứng dụng CAD.

## Cập nhật tài liệu trực tuyến — 19/11/2026

- STD-03 v1.15 và STD-04 v1.15: đổi ba thư mục tra cứu Reference/SDK mặc định trên máy build sang `D:\000\Reference\Autocad 2022`, `D:\000\Reference\Autocad 2025`, `D:\000\Reference\GstarCAD 2027`. Giữ quy tắc tìm SDK đệ quy trong phạm vi host, kiểm đủ thư viện, đối chiếu SHA khi nhiều bản và không trộn thư viện giữa CAD.
- Các đường dẫn này là nguồn tham chiếu **lúc build**, không phải DData/DUser/DProgram/Dicon/Dhelp của Support File Search Path lúc chạy. Chỉ thay tài liệu trực tuyến, không sửa source module hay gán Build PASS.

## Cập nhật tài liệu trực tuyến — 17/11/2026

- STD-12 v1.6: bổ sung cấu trúc MASTER XData thống nhất theo Dtscr1 V84, Drestore và nhánh đọc DPunch V381/legacy. Chưa sửa dữ liệu bản vẽ hoặc mã ứng dụng.
- Rút STD-19 khỏi danh mục và xóa tài liệu đang áp dụng theo yêu cầu chủ dự án; nội dung lịch sử vẫn truy được từ Git. Các mục V1.14 phía trên là ghi chép lịch sử, **không còn là quy ước hiện hành**.
- ZIP V1.17 là snapshot lịch sử, không được coi là ZIP đã chứa lần cập nhật trực tuyến này.

## Cập nhật tài liệu trực tuyến — 19/11/2026 (chuẩn tên thành phẩm)

- STD-03 v1.16, STD-04 v1.16: bỏ hậu tố CAD `C22`, `C25`, `G27` khỏi file sau build. Tên file theo đúng tên gốc module, ví dụ `DProject.dll`, `DProject.arx`, `DProject.grx`; mã host chỉ nằm ở nhánh project và `Dprogram/<Host>/` để không ghi đè hai bản build cùng tên. Khi triển khai chỉ chép binary đúng host vào `DProgram` runtime dạng phẳng.
- Ba root tham chiếu mặc định: `D:\000\Reference\Autocad 2022`, `D:\000\Reference\Autocad 2025`, `D:\000\Reference\GstarCAD 2027`; dùng cho managed và native, kiểm đúng host trước build.
- STD-16 v1.17: bổ sung QA đối chiếu filename, Reference và host-specific staging. Đây là cập nhật tài liệu; không tự động đổi code module, build hoặc nghiệm thu runtime.
- ZIP V1.17 vẫn là snapshot trước các cập nhật tài liệu trực tuyến, không được xem là đã chứa quy định mới.

## Cập nhật tài liệu trực tuyến — 19/11/2026: SQLite dùng chung trong DProgram

- STD-03 v1.17: một `DProgram\sqlite3.dll` Windows x64 dùng chung, không cài đặt SQLite/SQL Server hay ghi DLL vào System32, không trùng bản sao theo mỗi add-in; cập nhật runtime có kiểm tra tương thích.
- STD-05 v1.15: hợp đồng `DCAD-SQLITE-RUNTIME-001`, nạp động DLL bằng đường dẫn tuyệt đối từ `DProgram` DIRECT, không hard-link `WinSQLite3.lib` làm lỗi ARX LOAD; thiếu runtime chỉ dừng đúng chức năng dùng DB; bổ sung SQL01–SQL08.
- STD-16 v1.18: DataManifest, QA và điều kiện kiểm chứng bản DLL dùng chung, import table ARX, ARX LOAD, đọc/ghi dữ liệu.
- Chỉ sửa quy ước trực tuyến; **ZIP phát hành gần nhất vẫn V1.17**, chưa tạo bản ZIP tổng mới, không sửa/build các add-in.

## Cập nhật tài liệu trực tuyến — 19/11/2026: Ứng dụng portable trong DProgram

- STD-03 v1.18: ứng dụng portable có EXE được giữ cả thư mục con trong `DProgram`, không cần tách EXE ra thư mục phẳng. File add-in CAD vẫn triển khai phẳng đúng host.
- STD-05 v1.16: ưu tiên đường dẫn `RelativePath` đã khai báo; khi có `PortableExeDiscovery=DirectChild` mới tìm đúng tên EXE ở `DProgram` và các thư mục con trực tiếp (một cấp). Nhiều kết quả báo xung đột, không tự chọn; không quét ổ đĩa/thư mục khác. Không thay cách nạp `DProgram\sqlite3.dll`.
- STD-16 v1.19: bổ sung kiểm thử PORT01–PORT06 cho EXE portable.
- Chỉ cập nhật tài liệu trực tuyến; ZIP quy ước đã phát hành V1.17 và mã các add-in không thay đổi.
