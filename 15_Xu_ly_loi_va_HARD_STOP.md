# DCAD-STD-15 — Xử lý lỗi và HARD STOP

**Phiên bản:** 1.10  
**Mức áp dụng:** Bắt buộc

## Log, xử lý lỗi và HARD STOP [CORE]

**Quy trình**: hiện tượng → bằng chứng (source/log/command line/exception/screenshot/runtime/version) → nguyên nhân gốc (nếu chưa đủ: **Chưa xác định**) → ảnh hưởng (geometry/XData/settings/UI/build/host) → phương án + phản biện → người dùng duyệt → code → QA độc lập.

- Không báo giả thành công nếu build chưa chạy, NETLOAD chưa thử. Phân biệt `PASS`, `FAIL`, `NotVerified`; lỗi lặp nhiều phiên bản thì **HARD STOP việc vá biểu hiện**, đối chiếu baseline và kiến trúc.
- Lỗi file/path phải nêu: Module, host, profile nếu có, thư mục logic, tên file, Support Path nguồn, vị trí resolve, lỗi cụ thể, hành động sửa gợi ý. Log kỹ thuật có context đủ, UI không lộ stack trace.
- Khi **tài nguyên bắt buộc của chức năng/module** không đạt (thiếu DIRECT, không truy cập được thư mục/tệp, trùng DIRECT khác đích, dữ liệu sai), phát **cảnh báo yêu cầu thiết lập lại**, nêu thành phần/nguyên nhân/hướng dẫn `OPTIONS > Files > Support File Search Path` và dừng chức năng phụ thuộc. Không `FATAL` chỉ vì thiếu tài nguyên không sử dụng. Kiểm tra bố trí đủ nhóm DCAD ở cuối Support Path là trách nhiệm Setup/QA hệ thống.
- Setup/QA toàn hệ thống yêu cầu năm thư mục `DData`, `DUser`, `DProgram`, `DHelp`, `Dicon` DIRECT và root `DCAD` liền kề ở cuối Support Path. **Module chỉ kiểm tra tài nguyên theo DataManifest**, không yêu cầu toàn bộ nhóm này để khởi động. Root không được dùng để suy ra thư mục con. Chỉ báo `USER_FOLDER_AMBIGUOUS` khi có từ hai `DUser` DIRECT khác biệt trong lượt kiểm tra DUser; không đếm PARENT.
- Nếu DUser bị lỗi, không bắt buộc có file log để thông báo: hiện cảnh báo trực tiếp trên Command Line/hộp thoại CAD; không ghi log sang TEMP, AppData hoặc thư mục dự phòng. Kiểm tra Support Path ở **đầu lượt lệnh**, không kiểm tra lại liên tục trong quá trình tương tác giao diện.
- Nghiệp vụ đọc nhiều object: công bố rõ `fail-all` hay `skip-invalid`, số lượng thành công/bỏ qua/lỗi; không im lặng hoặc vẽ sai vì parse thiếu.
- Tăng version phải kèm log thay đổi có danh mục source tác động, thay đổi data/schema, migration, ngôn ngữ, layer, XData, tương thích, build/QA, KnownIssues. Không dùng mô tả chung chung “fixed bugs”.
