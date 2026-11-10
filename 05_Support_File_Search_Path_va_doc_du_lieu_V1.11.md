# DCAD-STD-05 — Support File Search Path và phân giải dữ liệu

**Phiên bản:** 1.13  
**Cập nhật:** 10/11/2026  
**Mức áp dụng:** Bắt buộc theo tài nguyên mà module sử dụng; Setup/QA kiểm theo phạm vi chức năng được triển khai  
**Hợp đồng:** `DCAD-DATA-DIR-001` — DIRECT-only và phân giải theo phạm vi DataManifest

## 1. Phạm vi áp dụng và nguyên tắc

DCAD quy định năm **tên thư mục logic**: `DData`, `DUser`, `DProgram`, `Dicon`, `Dhelp`. Đây là các tên được chấp nhận ở **thành phần cuối cùng của đường dẫn đăng ký**, không phải yêu cầu có đủ năm thư mục hoặc đặt năm entry ở cuối danh sách. Thư mục cha (ví dụ `DCAD`) không có tên/vị trí bắt buộc và không được dùng thay entry DIRECT. Hai cấp kiểm tra đều theo phụ thuộc thực tế:

1. **Module CAD:** chỉ xác định và kiểm tra những thư mục, tệp hoặc chương trình ngoài DWG mà module **thực sự sử dụng**, căn cứ `DataManifest` của module và chức năng đang thực hiện. Tài nguyên **bắt buộc** của module phải đạt kiểm tra trước khi sử dụng; thiếu tài nguyên không liên quan **không được gây `FATAL` hoặc ngăn lệnh chạy**.
2. **Setup/QA cấp hệ thống:** xác định các chức năng/module được triển khai và tập tài nguyên cần dùng theo DataManifest; kiểm các DIRECT tương ứng, khả năng truy cập, xung đột và tệp bắt buộc. **Không bắt buộc đủ năm thư mục, cùng root, liền kề, đúng thứ tự hay ở cuối danh sách.** Không đánh lỗi vì thiếu thư mục không được dùng trong phạm vi cấu hình. Chọn thư mục cha chỉ phục vụ xem/lựa chọn, không tự đăng ký cha hoặc suy thư mục con khi runtime.

`DIRECT-only` áp dụng như nhau ở cả hai cấp: chỉ đường dẫn **đăng ký trực tiếp**, có **tên thư mục cuối khớp tên logic** trong Support File Search Path của CAD/profile hiện hành mới hợp lệ. Sau chuẩn hóa dấu gạch cuối và so sánh Windows không phân biệt hoa/thường: `D:\A\DUser` và `E:\B\abc\DUser` hợp lệ cho DUser; `D:\A\DUser\abc` không hợp lệ vì tên cuối là `abc`. `DHelp` và `Dhelp` được nhận như cùng tên logic. Không suy ra thư mục con từ root/entry cha (`PARENT`), không suy ra thư mục anh em, không tìm đệ quy/quét ổ đĩa, không tự tạo thư mục logic và không chuyển sang đích dự phòng.

Thao tác **chọn nguồn trong Setup** được phép liệt kê các thư mục con trực tiếp của thư mục do người dùng chọn, chỉ để hiển thị/lựa chọn; việc này **không** biến nguồn thành một entry DIRECT hay cho phép runtime phân giải PARENT.

Tệp nhúng assembly và XData/XRecord trong DWG không phải tài nguyên cần phân giải qua Support Path. `Find`, `Read`, `Save` và `Restore` không được tự động kiểm tra thêm thư mục ngoài phạm vi thao tác.

## 2. Danh mục thư mục và cấu hình chuẩn của hệ thống

| Thư mục | Mục đích | Khi module phải kiểm tra |
|---|---|---|
| `DData` | Bảng dữ liệu kỹ thuật | Khi dùng tệp kỹ thuật từ DData |
| `DUser` | Settings, Last Used, `MIS_SET.SET`, log runtime | Khi đọc/ghi tệp tương ứng ở DUser |
| `DProgram` | Ứng dụng phụ trợ và metadata runtime | Khi gọi/đọc tài nguyên DProgram |
| `DHelp` | Tài liệu trợ giúp | Khi truy cập trợ giúp ngoài DLL |
| `Dicon` | Icon/ảnh ngoài DLL | Khi nạp tài nguyên ngoài DLL từ Dicon |
| Thư mục nguồn (ví dụ `DCAD`) | Chỉ định nơi xem/lựa chọn các thư mục cần dùng | **Không phải entry bắt buộc; không tự thêm vào Support Path và không dùng làm PARENT/fallback** |

**Cấu hình do Setup/QA kiểm tra:** chỉ những thư mục logic có tài nguyên bắt buộc trong phạm vi chức năng được triển khai mới phải có DIRECT hợp lệ. Không ép đủ năm, không ép cùng thư mục cha. Người dùng tự sắp xếp thứ tự/vị trí của các entry; không bắt buộc ở cuối, liền kề hoặc thêm root. Ví dụ dưới đây chỉ triển khai chức năng cần DUser và DProgram, không cần ba thư mục còn lại:

```text
... (các đường dẫn mặc định của CAD)
D:\DCAD\DUser
... (đường dẫn khác do người dùng đăng ký)
E:\UngDung\DProgram
```

`D:\DCAD` trong ví dụ chỉ là **thư mục cha**, không được thêm để thay đường dẫn DIRECT của thư mục logic. Một mục nguồn tồn tại sẵn từ cấu hình cũ được hiển thị như entry hiện có, để người dùng xem và loại bỏ có xác nhận; không xóa âm thầm các entry không liên quan. Các đường dẫn mặc định CAD khác được bảo toàn nếu người dùng không chủ động thay đổi.

**Ưu tiên tìm kiếm:** Support Path có thứ tự và CAD có thể tìm tài nguyên theo thứ tự ấy. Vì vậy Dsetup cho phép di chuyển lên/xuống và xóa khỏi **phương án xem trước**. Trước khi ghi, cảnh báo khi thứ tự thay đổi hoặc khi xóa mục cũ; sao lưu, cập nhật riêng Support Path và đọc lại xác nhận. Không tự động sắp xếp lại chỉ để đạt một thứ tự ví dụ.

**Phạm vi module:** Module CAD chỉ kiểm tra các tài nguyên thật sự cần dùng theo DataManifest. Nó không được yêu cầu đủ năm thư mục hoặc một thứ tự cụ thể khi chức năng chỉ cần một vài tài nguyên; với tài nguyên cần dùng, DIRECT-only và xử lý trùng khác đích vẫn bắt buộc.

**Hiệu lực:** STD-05 V1.12 thay các mô tả cũ bắt đủ năm/sáu entry, cùng root hoặc ở cuối danh sách. Tên cuối đường dẫn, phụ thuộc theo chức năng và DIRECT-only là yêu cầu; vị trí entry trong danh sách không phải yêu cầu nghiệm thu.

## 3. Hợp đồng DataManifest của module

1. Mỗi module phải khai báo mọi **tài nguyên ngoài DWG thực sự sử dụng** trong `DataManifest`: tối thiểu `FileName/ResourceName`, `LogicalFolder`, `OwnerModule`, `Required`, `AccessMode`, `DefaultBehavior`; những trường nguồn/đích, version, schema/encoding áp dụng cho từng dạng tệp. `Required=true` là yêu cầu bắt buộc cho **chức năng dùng tài nguyên đó**, không suy rộng sang mọi module DCAD.
2. Trước khi thực hiện chức năng, tạo tập phụ thuộc từ các mục `Required=true` và các phụ thuộc runtime bắt buộc được khai báo cho chức năng. **Chỉ phân giải thư mục chứa tài nguyên bắt buộc hoặc tài nguyên tùy chọn mà chức năng thực sự đang truy cập**. Tài nguyên tùy chọn (`Required=false`) chỉ được kiểm tra khi tính năng thực sự truy cập; thiếu tài nguyên tùy chọn không gây `FATAL` nếu `DefaultBehavior` có phương án được công bố.
3. Không suy ra phụ thuộc vào năm thư mục từ việc module mang tên DCAD hoặc do Setup/QA đang kiểm nhiều module. Một module chỉ đọc `DUser\MIS_SET.SET` không phải xác định `DData`, `DProgram`, `DHelp`, `Dicon` hay thư mục nguồn.
4. Thiếu/không chính xác DataManifest là **lỗi tuân thủ/QA** cần hoàn thiện, không phải lý do để tự động bắt toàn bộ module kiểm tra năm đường dẫn. Không chế tạo mục file hoặc đường dẫn thay thế để bỏ qua manifest.
5. `MIS_SET.SET` bắt buộc với module **sử dụng cơ chế ngôn ngữ DCAD**. Các module không sử dụng tài nguyên này không phải đọc hoặc kiểm tra nó. Khi đã áp dụng, phải đọc trong `DUser` DIRECT; lỗi bắt buộc dẫn đến cảnh báo thiết lập lại và dừng **chức năng phụ thuộc**.
6. Lỗi ghi log thuộc DUser phải được thông báo trong CAD, không chuyển sang TEMP/AppData làm đích dự phòng. Log/job có hợp đồng vị trí riêng áp dụng mục 7. Nếu log chỉ là tài nguyên phụ trợ không bắt buộc cho nghiệp vụ thì lỗi log không được tạo `FATAL` giả; nếu DataManifest xác định log là điều kiện bắt buộc của chức năng thì thực hiện xử lý lỗi đúng mức đó.

### 3.1. Danh mục tấm (PlateCatalog) từ DData [THEO TÍNH NĂNG]

- Khi module khai báo danh mục tấm DAT (ví dụ Dtscr2 dùng `tscr_plates.dat`), chỉ đọc file từ `DData` DIRECT theo DataManifest. Không lấy danh mục layer hiện có thay DAT, không ghi ngược file chỉ vì chọn tấm, không thay thế/ghi đè `Ti.dat` dùng chung; giữ thứ tự, mã hóa và nội dung DAT, xử lý EOF DOS `0x1A` nếu có.
- Tách `RawPlateCode` (nguyên chuỗi nguồn), `PlateCode` (mã đã chuẩn hóa dùng trong UI và bản vẽ), `LayerName` (layer đích theo STD-09). Với PlateCatalog có cú pháp ngoặc ngoài: `(UP)` → `UP`, `(UB)` → `UB`, `(PH_P)` → `PH_P`. Chỉ bỏ một cặp ngoặc `(` `)` bao trọn tên sau khi trim; không cắt `_P` thuộc tên gốc, không áp quy tắc bỏ ngoặc cho XData, tên file hoặc tất cả loại layer khác.
- Hộp thoại chỉ hiển thị `UP`, `UB`, `PH`…; mã kỹ thuật không dịch. Bản ghi mã trống, ngoặc thiếu đối xứng hoặc hai bản ghi trùng nhau sau chuẩn hóa phải được báo lỗi dữ liệu, không tự bịa tên tấm khác.
- QA: `(UP)` hiển thị `UP`, `(PH_P)` hiển thị `PH_P`; số lượng/thứ tự không đổi, DAT giữ nguyên byte, thiếu tài nguyên bắt buộc báo theo STD-05, khi vẽ dùng `PlateCode` đã chuẩn hóa theo STD-09.

## 4. Nguồn và quy trình phân giải DIRECT-only

1. Lấy danh sách **Support File Search Path của CAD/profile hiện hành**. AutoCAD 2022/2025 có thể dùng `ACADPREFIX` lấy trong CAD; GstarCAD 2027 dùng adapter host đã kiểm chứng. Không lấy biến môi trường Windows `ACAD`, thư mục DLL, current directory hay `%APPDATA%` làm đường dẫn dự phòng.
2. Chuẩn hóa entry: đường dẫn tuyệt đối, dấu nháy/khoảng trắng bọc, dấu phân cách và gạch cuối; so sánh Windows không phân biệt hoa/thường. Giữ `SupportPathIndex` từ 1 theo thứ tự gốc; gộp entry trùng/alias hoặc junction đã chứng minh cùng thư mục vật lý.
3. **Chỉ xét tên thư mục logic nằm trong tập phụ thuộc của chức năng**. Entry được chấp nhận khi thành phần cuối khớp chính xác tên thư mục logic (DIRECT). Không tìm `DUser`, `DData`... bên trong entry root/cha hoặc thư mục logic khác.
4. Với từng thư mục **thuộc tập phụ thuộc**: đúng một DIRECT khác biệt, tồn tại và truy cập được → chọn; nhiều DIRECT khác đích → cảnh báo xung đột; không có DIRECT → cảnh báo chưa thiết lập; DIRECT đã đăng ký nhưng bị xóa/không truy cập được → cảnh báo không khả dụng. Không bỏ qua DIRECT lỗi để chuyển sang đích khác.
5. Kiểm tra sự tồn tại, schema, quyền và giá trị **các tài nguyên bắt buộc đã khai báo**. Nếu lỗi, cảnh báo yêu cầu thiết lập lại và dừng chức năng phụ thuộc; không thay bằng dữ liệu giả và không âm thầm ghi đè file chuẩn.
6. Phân giải một lần khi bắt đầu lượt lệnh/chức năng liên quan, dùng nhất quán trong lượt thao tác. Không giám sát lại đường dẫn khi chuyển tab/nhấn Lưu; lượt lệnh mới lấy lại Support Path hiện hành.

**Không được** yêu cầu đủ năm entry, áp đặt thứ tự hoặc vị trí trong danh sách như điều kiện `FATAL` khi khởi động **một module CAD**. Setup/QA cấp hệ thống cũng chỉ kiểm các DIRECT/tài nguyên trong phạm vi chức năng triển khai; không ép đủ năm hoặc cùng nguồn; **không kiểm vị trí, thứ tự, tính liền kề**.

## 5. Cảnh báo và xử lý lỗi

Khi **tài nguyên bắt buộc trong phạm vi chức năng** thiếu, sai, xung đột hoặc không truy cập được, thông báo:

```text
[DCAD] YÊU CẦU THIẾT LẬP LẠI
Module/Chức năng: <module/chức năng>
Thành phần bắt buộc: <thư mục hoặc tệp>
Nguyên nhân: <thiếu DIRECT | không tồn tại | không truy cập được |
             xung đột DIRECT | thiếu tệp | dữ liệu không hợp lệ>
Hướng dẫn: Kiểm tra tệp và thư mục tương ứng;
           mở OPTIONS > Files > Support File Search Path;
           đăng ký trực tiếp đúng thư mục cần sử dụng.
```

- Dừng **chức năng phụ thuộc**, không đánh dấu toàn bộ module hoặc hệ thống là không dùng được do một thư mục không liên quan.
- Khi Setup/QA kiểm tra **hệ thống**, chỉ cảnh báo thiếu/hỏng/xung đột tài nguyên bắt buộc của các chức năng đã chọn triển khai. Không cảnh báo vì thiếu thư mục không dùng, khác thư mục cha, khác vị trí/thứ tự hoặc không liền kề. Entry root cũ không thay được DIRECT; chỉ chỉnh/xóa entry theo phương án có xác nhận, không tự xóa path CAD khác.
- `USER_FOLDER_AMBIGUOUS` chỉ áp dụng khi có từ hai `DUser` DIRECT khác biệt **trong lượt kiểm tra DUser**, không đếm PARENT. Chẩn đoán nội bộ phân biệt `NOT_CONFIGURED`, `UNAVAILABLE`, `CONFLICT`, `INVALID_DATA` nhưng giao diện cung cấp hướng dẫn thiết lập lại dễ hiểu.
- Nếu DUser không ghi log được, phát cảnh báo trực tiếp tại CAD; không ghi ở đích khác và không yêu cầu file log mới có thể thông báo lỗi.
- Khi đã chọn DUser, settings và log runtime được khai báo lưu tại DUser của cùng module dùng thống nhất resolver; ghi file an toàn, validate, flush, sao lưu khi cần. File settings có thể khởi tạo lần đầu trong DUser hợp lệ nếu DataManifest cho phép; không tạo thư mục logic.

## 6. Kiểm thử bắt buộc theo phạm vi

**Phân lớp:** `SYS` = chạy trong Setup/QA **cấp hệ thống**; `MOD` = chạy khi module/chức năng có phụ thuộc tương ứng (không biến tài nguyên không liên quan thành điều kiện FAIL). Giữ mã ID DIR01–DIR28 để đối chiếu biên bản QA cũ, **điều chỉnh kỳ vọng các ca SYS** theo quy tắc mới. DIR29–DIR33 kiểm phạm vi triển khai, thứ tự tự do, loại trừ root và quyền thao tác; DIR34–DIR38 kiểm tên thư mục cuối và đường dẫn con ứng dụng.

| ID | Phạm vi | Điều kiện kiểm thử | Kết quả bắt buộc |
|---|---|---|---|
| DIR01 | SYS | Chỉ có các DIRECT cần cho chức năng triển khai, tên thư mục cuối hợp lệ | Setup/QA đạt nếu đủ phụ thuộc; không cần đủ năm |
| DIR02 | SYS | Thiếu một trong năm tên logic nhưng chức năng triển khai không dùng nó | Không báo thiếu; chỉ báo nếu tài nguyên đó thực sự bắt buộc |
| DIR03 | SYS/MOD | Có thư mục thật dưới nguồn nhưng chỉ đăng ký nguồn; module cần thư mục đó | Không chấp nhận PARENT; module lỗi nếu tài nguyên bắt buộc |
| DIR04 | SYS/MOD | Có hai DUser DIRECT khác đích, lượt kiểm tra cần DUser | Báo xung đột; chỉ báo `USER_FOLDER_AMBIGUOUS` trong trường hợp này |
| DIR05 | SYS/MOD | Hai entry DUser là cùng một đích sau chuẩn hóa | Gộp, không báo xung đột giả |
| DIR06 | SYS/MOD | DUser đã đăng ký nhưng thư mục bị xóa, có phụ thuộc DUser | Cảnh báo thiết lập lại; không tự tạo |
| DIR07 | SYS/MOD | DUser không truy cập được, có phụ thuộc DUser | Cảnh báo thiết lập lại; không chuyển đích |
| DIR08 | SYS/MOD | DUser lỗi ghi log | Hiển thị trong CAD, không ghi TEMP/AppData; chỉ FATAL nếu log bắt buộc |
| DIR09 | SYS/MOD | `MIS_SET.SET` thiếu/sai, chức năng sử dụng ngôn ngữ DCAD | Cảnh báo thiết lập lại; dừng chức năng cần ngôn ngữ |
| DIR10 | SYS | Một hay nhiều entry khác xen giữa năm DIRECT hợp lệ | Cấu hình hợp lệ; không bắt buộc liền kề |
| DIR11 | SYS | Năm DIRECT nằm đầu/giữa/cuối theo vị trí người dùng chọn | Cấu hình hợp lệ; không bắt buộc đặt cuối |
| DIR12 | SYS/MOD | DData và DUser DIRECT thuộc hai thư mục cha/ổ đĩa khác nhau | Hợp lệ nếu từng tài nguyên cần dùng đạt; không ép cùng nguồn |
| DIR13 | MOD | Settings module chưa có, DUser hợp lệ, manifest cho phép tạo | Chỉ tạo tệp settings trong DUser, không tạo thư mục |
| DIR14 | SYS/MOD | Host không cung cấp nguồn Support Path cho thư mục cần dùng | Cảnh báo thiết lập lại; không dùng fallback |
| DIR15 | MOD | Form thao tác nhiều bước | Giữ kết quả phân giải trong lượt, không kiểm tra mỗi thao tác UI |
| DIR16 | MOD | Support Path được thay đổi rồi chạy lệnh mới | Lượt mới nhận cấu hình mới |
| DIR17 | SYS/MOD | Thư mục nguồn có Ti.dat nhưng module cần DData mà DData không DIRECT | Không đọc Ti.dat thông qua nguồn; cảnh báo khi DData bắt buộc |
| DIR18 | MOD | Ghi/đọc DUser lỗi sau khi chọn đúng DUser | Không fallback; cảnh báo và xử lý chức năng phụ thuộc |
| DIR19 | SYS/MOD | Thiếu tệp bắt buộc được khai báo trong DataManifest của chức năng | Dừng chức năng phụ thuộc, không âm thầm tiếp tục |
| DIR20 | SYS | CAD có path mặc định trước/giữa/sau năm DIRECT | Setup/QA đạt; giữ nguyên path mặc định nếu người dùng không chỉnh |
| DIR21 | SYS/MOD | Hai DUser DIRECT khác đích, một thư mục không truy cập được, cần DUser | Báo xung đột/không khả dụng; không bỏ một DIRECT để chọn cái còn lại |
| DIR22 | MOD | Module chỉ cần DUser/MIS_SET.SET; DData/DHelp/Dicon/DProgram/nguồn thiếu | Module vẫn hoạt động nếu DUser/MIS_SET.SET hợp lệ |
| DIR23 | MOD | Module chỉ cần DUser/MIS_SET.SET; năm entry không liền kề hoặc không ở cuối | Module không FATAL do vị trí/thứ tự entry |
| DIR24 | MOD | Module không dùng DData; DData DIRECT thiếu hoặc không truy cập được | Không phân giải DData; không cảnh báo/FATAL liên quan DData |
| DIR25 | MOD | Module dùng DProgram cho EXE bắt buộc, không dùng DData/DHelp/Dicon | Chỉ kiểm tra DProgram và các phụ thuộc thực tế đã khai báo |
| DIR26 | MOD | Tệp `Required=false` không tồn tại | Không FATAL nếu DefaultBehavior đã công bố; không fallback đường dẫn trái phép |
| DIR27 | SYS/MOD | Thiếu thư mục không được chức năng/module triển khai sử dụng | Cả Setup/QA và module không FAIL/FATAL vì phần không dùng |
| DIR28 | SYS/MOD | Tài nguyên bắt buộc của module thiếu hoặc sai schema | Module cảnh báo thiết lập lại và dừng đúng chức năng; Setup/QA ghi lỗi phù hợp |
| DIR29 | SYS | Năm DIRECT đầy đủ nhưng nằm rải rác và thứ tự bất kỳ | Setup/QA đạt; không cảnh báo vị trí hay thứ tự |
| DIR30 | SYS | Năm DIRECT đầy đủ, không có entry tên thư mục nguồn `DCAD` | Setup/QA đạt; không thêm nguồn tự động |
| DIR31 | SYS | Dsetup chọn lại nguồn mới và phát hiện entry nguồn cũ | Cho xem/xóa có xác nhận; không tự xóa path CAD khác |
| DIR32 | SYS | Người dùng Lên/Xuống/Xóa trong phương án | Cho phép chỉnh, cảnh báo ảnh hưởng ưu tiên và sao lưu trước khi ghi |
| DIR33 | SYS/MOD | Chỉ triển khai chức năng dùng DUser, thiếu bốn thư mục còn lại | Setup/QA và module đạt phần đường dẫn nếu DUser/tệp cần dùng hợp lệ |
| DIR34 | SYS/MOD | Entry kết thúc bằng `DUser\abc`, file có trong đó | Không nhận là DUser; không dùng file để lách tên thư mục cuối |
| DIR35 | SYS/MOD | Entry `E:\abc\DUser`, không có root tên DCAD | Nhận DUser DIRECT; thư mục cha tự do |
| DIR36 | SYS/MOD | Chỉ đăng ký DProgram DIRECT, EXE nằm đúng `Dnest\Dnest.exe` theo manifest | Cho phép đường dẫn con cố định; không cần đăng ký Dnest |
| DIR37 | SYS/MOD | Chỉ đăng ký `DProgram\Dnest`, hoặc chỉ đăng ký root chứa DProgram | Không nhận thay DProgram DIRECT; không dò lên/xuống để chữa |
| DIR38 | SYS/MOD | Entry có gạch cuối hoặc khác hoa/thường, tên cuối sau chuẩn hóa là Dhelp | Nhận cùng thư mục logic; không dùng so khớp chuỗi suffix tùy ý |

**Bất biến:** `tên thư mục cuối khớp logic` · `DIRECT-only` · `không ép đủ năm hoặc cùng root` · `DataManifest xác định phạm vi SYS/MOD` · `không FATAL vì tài nguyên không dùng` · `đường dẫn con chỉ theo hợp đồng khai báo` · `không fallback hoặc tìm đệ quy` · `không giám sát liên tục trong hộp thoại`.

## 7. Đường dẫn con ứng dụng và dữ liệu portable riêng

- Phân biệt **entry đăng ký** và **đường dẫn tài nguyên bên trong**. Entry luôn kết thúc bằng tên logic; tài nguyên có thể nằm trong đường dẫn con cố định nếu DataManifest khai báo `LogicalFolder`, `RelativePath`, owner, Required, AccessMode và chính sách lưu. RelativePath không được tuyệt đối hoặc chứa `..` thoát khỏi thư mục logic.
- Ví dụ DNesting: đăng ký `D:\DCAD\DProgram`, rồi tìm đúng `Dnest\Dnest.exe`; không đăng ký `DProgram\Dnest` thay DProgram và không dò từ root. Đây là hợp đồng riêng DNesting, không cho phép mọi module quét các thư mục con tùy ý.
- DNesting V49 quy định `DNest_settings.json`, `DNest_cad-command.json`, `DNest_window.json` nằm cạnh đúng `Dnest.exe` đã chọn. DUser vẫn dùng riêng cho `MIS_SET.SET` khi chạy trong CAD. Không tự chuyển settings portable về DUser hay áp cách lưu này sang module khác.
- Trường hợp có nguồn settings/log/job riêng đã công bố phải khai báo rõ trong manifest/hợp đồng module; đó là vị trí được chỉ định, không phải đích dự phòng khi DIRECT lỗi. Không mở rộng ngoại lệ ngầm từ mã legacy. Mã cũ còn nhận PARENT phải ghi không tuân thủ DIRECT-only và chỉ sửa khi có yêu cầu sửa phần mềm.
