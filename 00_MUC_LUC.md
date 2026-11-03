# BỘ TIÊU CHUẨN DCAD — V1.12

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
| 12 | [Ghi, đọc XData và XRecord](12_Ghi_doc_XData_va_XRecord.md) | Theo tính năng |
| 13 | [Giao diện, icon và Preview](13_Giao_dien_icon_va_Preview.md) | Theo tính năng |
| 14 | [Transaction, Undo và hiệu năng](14_Transaction_Undo_va_hieu_nang.md) | Bắt buộc theo thao tác |
| 15 | [Xử lý lỗi và HARD STOP](15_Xu_ly_loi_va_HARD_STOP.md) | Bắt buộc |
| 16 | [DataManifest, BuildInfo và QA](16_DataManifest_BuildInfo_va_QA.md) | Bắt buộc |
| 17 | [Autoupdate — Tự động chuẩn hóa, build, đóng gói và upload GitHub](17_Autoupdate.md) | Bắt buộc khi cập nhật GitHub |

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
- Phiên bản ZIP tổng là **V1.12**; số phiên bản ở đầu từng file cho biết lần sửa nội dung của chính quy chuẩn đó, nên một số file không sửa vẫn giữ phiên bản cũ.

## Cập nhật V1.12 — 03/11/2026

- Đồng bộ quy tắc tên thư mục cuối, kiểm tra theo phụ thuộc và thư mục con ứng dụng được khai báo.
- Tự chọn baseline gần nhất, tăng phiên bản phần mềm đúng 1 khi không có chỉ định; giữ lịch sử và tách trạng thái nghiệm thu.
- Giữ tên file hiện có để bảo toàn URL; hậu tố phiên bản trong tên file cũ là định danh lịch sử, phiên bản hiệu lực nằm ở đầu nội dung.
