# DCAD-STD-05 — Support File Search Path và phân giải dữ liệu

**Phiên bản:** 1.11  
**Cập nhật:** 21/10/2026 — năm DIRECT bắt buộc; thứ tự/vị trí tự do; nguồn không phải entry.  
**Mức áp dụng:** Bắt buộc theo tài nguyên mà module sử dụng; kiểm tra cấu hình đầy đủ tại Setup/QA hệ thống  
**Hợp đồng:** `DCAD-DATA-DIR-001` — DIRECT-only và phân giải theo phạm vi DataManifest

## 1. Phạm vi áp dụng và nguyên tắc

DCAD có **năm thư mục logic bắt buộc đối với cấu hình đầy đủ cấp hệ thống**: `DData`, `DUser`, `DProgram`, `DHelp`, `Dicon`. Tên thư mục nguồn (ví dụ `DCAD`) chỉ dùng để chọn, quét và đối chiếu, **không phải entry bắt buộc hoặc tự động thêm vào Support File Search Path**. Việc kiểm tra được tách thành **hai cấp độc lập**:

1. **Module CAD:** chỉ xác định và kiểm tra những thư mục, tệp hoặc chương trình ngoài DWG mà module **thực sự sử dụng**, căn cứ `DataManifest` của module và chức năng đang thực hiện. Tài nguyên **bắt buộc** của module phải đạt kiểm tra trước khi sử dụng; thiếu tài nguyên không liên quan **không được gây `FATAL` hoặc ngăn lệnh chạy**.
2. **Setup/QA cấp hệ thống:** kiểm tra đúng **năm thư mục logic DIRECT**, tồn tại và truy cập được, không trùng tên logic khác đích; khi chọn nguồn, kiểm tra năm thư mục cùng thuộc nguồn đó. **Không bắt buộc thứ tự, tính liền kề hay vị trí cuối danh sách.** Thư mục nguồn không được tự động đăng ký. Đây là điều kiện nghiệm thu cấu hình toàn hệ thống, không phải điều kiện khởi động mặc định của từng module.

`DIRECT-only` áp dụng như nhau ở cả hai cấp: chỉ đường dẫn **đăng ký trực tiếp** tới chính thư mục logic trong Support File Search Path của CAD/profile hiện hành mới hợp lệ. Không suy ra thư mục con từ root/entry cha (`PARENT`), không suy ra thư mục anh em, không tìm đệ quy/quét ổ đĩa, không tự tạo thư mục logic và không chuyển sang đích dự phòng.

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
| Thư mục nguồn (ví dụ `DCAD`) | Chỉ định nơi quét/lựa chọn năm thư mục | **Không phải entry bắt buộc; không tự thêm vào Support Path và không dùng làm PARENT/fallback** |

**Cấu hình chuẩn do Setup/QA kiểm tra:** năm thư mục logic phải đăng ký **DIRECT** trên Profile CAD đang dùng. **Người dùng được tự do sắp xếp vị trí và thứ tự** của các entry trong `Support File Search Path` thông qua thao tác có xác nhận; không bắt buộc ở cuối, không bắt buộc liền kề và không tự chèn tên thư mục nguồn. Ví dụ một cấu hình hợp lệ (chỉ minh họa, không phải thứ tự bắt buộc):

```text
... (các đường dẫn mặc định của CAD)
D:\DCAD\DUser
... (đường dẫn khác do người dùng đăng ký)
D:\DCAD\DData
D:\DCAD\Dicon
D:\DCAD\DProgram
D:\DCAD\DHelp
```

`D:\DCAD` trong ví dụ chỉ là **thư mục nguồn**, không được thêm chỉ vì nó là cha của năm thư mục. Một mục nguồn tồn tại sẵn từ cấu hình cũ được hiển thị như entry hiện có, để người dùng xem và loại bỏ có xác nhận; không xóa âm thầm các entry không liên quan. Các đường dẫn mặc định CAD khác được bảo toàn nếu người dùng không chủ động thay đổi.

**Ưu tiên tìm kiếm:** Support Path có thứ tự và CAD có thể tìm tài nguyên theo thứ tự ấy. Vì vậy Dsetup cho phép di chuyển lên/xuống và xóa khỏi **phương án xem trước**. Trước khi ghi, cảnh báo khi thứ tự thay đổi hoặc khi xóa mục cũ; sao lưu, cập nhật riêng Support Path và đọc lại xác nhận. Không tự động sắp xếp lại chỉ để đạt một thứ tự ví dụ.

**Phạm vi module:** Module CAD chỉ kiểm tra các tài nguyên thật sự cần dùng theo DataManifest. Nó không được yêu cầu đủ năm thư mục hoặc một thứ tự cụ thể khi chức năng chỉ cần một vài tài nguyên; với tài nguyên cần dùng, DIRECT-only và xử lý trùng khác đích vẫn bắt buộc.

**Hiệu lực khi có tài liệu cũ khác:** Nếu một bản STD-03/STD-07 cũ còn mô tả nhóm sáu entry đặt cuối, thì đối với yêu cầu **Support File Search Path**, áp dụng quy tắc mới của STD-05 V1.11. Các tài liệu liên quan cần được cập nhật đồng bộ ở lần ban hành kế tiếp; không tự hiểu rằng chuẩn cũ vẫn bắt buộc root/thứ tự.

## 3. Hợp đồng DataManifest của module

1. Mỗi module phải khai báo mọi **tài nguyên ngoài DWG thực sự sử dụng** trong `DataManifest`: tối thiểu `FileName/ResourceName`, `LogicalFolder`, `OwnerModule`, `Required`, `AccessMode`, `DefaultBehavior`; những trường nguồn/đích, version, schema/encoding áp dụng cho từng dạng tệp. `Required=true` là yêu cầu bắt buộc cho **chức năng dùng tài nguyên đó**, không suy rộng sang mọi module DCAD.
2. Trước khi thực hiện chức năng, tạo tập phụ thuộc từ các mục `Required=true` và các phụ thuộc runtime bắt buộc được khai báo cho chức năng. **Chỉ phân giải thư mục chứa tài nguyên bắt buộc hoặc tài nguyên tùy chọn mà chức năng thực sự đang truy cập**. Tài nguyên tùy chọn (`Required=false`) chỉ được kiểm tra khi tính năng thực sự truy cập; thiếu tài nguyên tùy chọn không gây `FATAL` nếu `DefaultBehavior` có phương án được công bố.
3. Không suy ra phụ thuộc vào năm thư mục từ việc module mang tên DCAD hoặc do Setup/QA yêu cầu cấu trúc đầy đủ. Một module chỉ đọc `DUser\MIS_SET.SET` không phải xác định `DData`, `DProgram`, `DHelp`, `Dicon` hay thư mục nguồn.
4. Thiếu/không chính xác DataManifest là **lỗi tuân thủ/QA** cần hoàn thiện, không phải lý do để tự động bắt toàn bộ module kiểm tra năm đường dẫn. Không chế tạo mục file hoặc đường dẫn thay thế để bỏ qua manifest.
5. `MIS_SET.SET` bắt buộc với module **sử dụng cơ chế ngôn ngữ DCAD**. Các module không sử dụng tài nguyên này không phải đọc hoặc kiểm tra nó. Khi đã áp dụng, phải đọc trong `DUser` DIRECT; lỗi bắt buộc dẫn đến cảnh báo thiết lập lại và dừng **chức năng phụ thuộc**.
6. Lỗi ghi log runtime phải được thông báo trong CAD, không viết sang TEMP/AppData. Nếu log chỉ là tài nguyên phụ trợ không bắt buộc cho nghiệp vụ thì lỗi log không được tạo `FATAL` giả; nếu DataManifest xác định log là điều kiện bắt buộc của chức năng thì thực hiện xử lý lỗi đúng mức đó.

## 4. Nguồn và quy trình phân giải DIRECT-only

1. Lấy danh sách **Support File Search Path của CAD/profile hiện hành**. AutoCAD 2022/2025 có thể dùng `ACADPREFIX` lấy trong CAD; GstarCAD 2027 dùng adapter host đã kiểm chứng. Không lấy biến môi trường Windows `ACAD`, thư mục DLL, current directory hay `%APPDATA%` làm đường dẫn dự phòng.
2. Chuẩn hóa entry: đường dẫn tuyệt đối, dấu nháy/khoảng trắng bọc, dấu phân cách và gạch cuối; so sánh Windows không phân biệt hoa/thường. Giữ `SupportPathIndex` từ 1 theo thứ tự gốc; gộp entry trùng/alias hoặc junction đã chứng minh cùng thư mục vật lý.
3. **Chỉ xét tên thư mục logic nằm trong tập phụ thuộc của chức năng**. Entry được chấp nhận khi thành phần cuối khớp chính xác tên thư mục logic (DIRECT). Không tìm `DUser`, `DData`... bên trong entry root/cha hoặc thư mục logic khác.
4. Với từng thư mục **thuộc tập phụ thuộc**: đúng một DIRECT khác biệt, tồn tại và truy cập được → chọn; nhiều DIRECT khác đích → cảnh báo xung đột; không có DIRECT → cảnh báo chưa thiết lập; DIRECT đã đăng ký nhưng bị xóa/không truy cập được → cảnh báo không khả dụng. Không bỏ qua DIRECT lỗi để chuyển sang đích khác.
5. Kiểm tra sự tồn tại, schema, quyền và giá trị **các tài nguyên bắt buộc đã khai báo**. Nếu lỗi, cảnh báo yêu cầu thiết lập lại và dừng chức năng phụ thuộc; không thay bằng dữ liệu giả và không âm thầm ghi đè file chuẩn.
6. Phân giải một lần khi bắt đầu lượt lệnh/chức năng liên quan, dùng nhất quán trong lượt thao tác. Không giám sát lại đường dẫn khi chuyển tab/nhấn Lưu; lượt lệnh mới lấy lại Support Path hiện hành.

**Không được** yêu cầu đủ năm entry, áp đặt thứ tự hoặc vị trí trong danh sách như điều kiện `FATAL` khi khởi động **một module CAD**. Setup/QA cấp hệ thống kiểm đủ năm DIRECT hợp lệ, trùng/xung đột và cùng nguồn nếu đã chọn nguồn; **không kiểm vị trí, thứ tự, tính liền kề**.

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
- Khi Setup/QA kiểm tra **toàn hệ thống**, cảnh báo khi thiếu một trong **năm** DIRECT, thư mục không tồn tại/không truy cập được, trùng tên logic khác đích, hoặc năm thư mục không cùng nguồn được chọn. **Không cảnh báo sai vị trí, sai thứ tự hay không liền kề**. Nếu thư mục nguồn có sẵn trong Profile, cho người dùng xem và chủ động loại bỏ theo phương án; nguồn không thuộc năm entry bắt buộc.
- `USER_FOLDER_AMBIGUOUS` chỉ áp dụng khi có từ hai `DUser` DIRECT khác biệt **trong lượt kiểm tra DUser**, không đếm PARENT. Chẩn đoán nội bộ phân biệt `NOT_CONFIGURED`, `UNAVAILABLE`, `CONFLICT`, `INVALID_DATA` nhưng giao diện cung cấp hướng dẫn thiết lập lại dễ hiểu.
- Nếu DUser không ghi log được, phát cảnh báo trực tiếp tại CAD; không ghi ở đích khác và không yêu cầu file log mới có thể thông báo lỗi.
- Khi đã chọn DUser, settings và log runtime thuộc cùng module dùng thống nhất resolver; ghi file an toàn, validate, flush, sao lưu khi cần. File settings có thể khởi tạo lần đầu trong DUser hợp lệ nếu DataManifest cho phép; không tạo thư mục logic.

## 6. Kiểm thử bắt buộc theo phạm vi

**Phân lớp:** `SYS` = chạy trong Setup/QA **cấp hệ thống**; `MOD` = chạy khi module/chức năng có phụ thuộc tương ứng (không biến tài nguyên không liên quan thành điều kiện FAIL). Giữ mã ID DIR01–DIR28 để đối chiếu biên bản QA cũ, **điều chỉnh kỳ vọng các ca SYS** theo quy tắc mới. Bổ sung DIR29–DIR33 cho năm thư mục, thứ tự tự do, loại trừ nguồn và quyền thao tác người dùng.

| ID | Phạm vi | Điều kiện kiểm thử | Kết quả bắt buộc |
|---|---|---|---|
| DIR01 | SYS | Năm thư mục con DIRECT hợp lệ, cùng nguồn đã chọn, nằm ở bất kỳ vị trí nào | Setup/QA cấu hình đạt |
| DIR02 | SYS | Có nguồn nhưng thiếu một trong năm DIRECT | Setup/QA báo thiếu; không suy ra từ nguồn |
| DIR03 | SYS/MOD | Có thư mục thật dưới nguồn nhưng chỉ đăng ký nguồn; module cần thư mục đó | Không chấp nhận PARENT; module lỗi nếu tài nguyên bắt buộc |
| DIR04 | SYS/MOD | Có hai DUser DIRECT khác đích, lượt kiểm tra cần DUser | Báo xung đột; chỉ báo `USER_FOLDER_AMBIGUOUS` trong trường hợp này |
| DIR05 | SYS/MOD | Hai entry DUser là cùng một đích sau chuẩn hóa | Gộp, không báo xung đột giả |
| DIR06 | SYS/MOD | DUser đã đăng ký nhưng thư mục bị xóa, có phụ thuộc DUser | Cảnh báo thiết lập lại; không tự tạo |
| DIR07 | SYS/MOD | DUser không truy cập được, có phụ thuộc DUser | Cảnh báo thiết lập lại; không chuyển đích |
| DIR08 | SYS/MOD | DUser lỗi ghi log | Hiển thị trong CAD, không ghi TEMP/AppData; chỉ FATAL nếu log bắt buộc |
| DIR09 | SYS/MOD | `MIS_SET.SET` thiếu/sai, chức năng sử dụng ngôn ngữ DCAD | Cảnh báo thiết lập lại; dừng chức năng cần ngôn ngữ |
| DIR10 | SYS | Một hay nhiều entry khác xen giữa năm DIRECT hợp lệ | Cấu hình hợp lệ; không bắt buộc liền kề |
| DIR11 | SYS | Năm DIRECT nằm đầu/giữa/cuối theo vị trí người dùng chọn | Cấu hình hợp lệ; không bắt buộc đặt cuối |
| DIR12 | SYS | DData/DUser thuộc hai nguồn khác nhau khi đã chọn một nguồn chung | Setup/QA báo không thống nhất nguồn; module chỉ xét DIRECT cần dùng |
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
| DIR27 | SYS/MOD | Setup/QA báo thiếu một trong năm DIRECT; module có đủ phụ thuộc riêng | Setup/QA không đạt, module không FATAL vì thiếu tài nguyên không dùng |
| DIR28 | SYS/MOD | Tài nguyên bắt buộc của module thiếu hoặc sai schema | Module cảnh báo thiết lập lại và dừng đúng chức năng; Setup/QA ghi lỗi phù hợp |

| DIR29 | SYS | Năm DIRECT đầy đủ nhưng nằm rải rác và thứ tự bất kỳ | Setup/QA đạt; không cảnh báo vị trí hay thứ tự |
| DIR30 | SYS | Năm DIRECT đầy đủ, không có entry tên thư mục nguồn `DCAD` | Setup/QA đạt; không thêm nguồn tự động |
| DIR31 | SYS | Dsetup chọn lại nguồn mới và phát hiện entry nguồn cũ | Cho xem/xóa có xác nhận; không tự xóa path CAD khác |
| DIR32 | SYS | Người dùng Lên/Xuống/Xóa trong phương án | Cho phép chỉnh, cảnh báo ảnh hưởng ưu tiên và sao lưu trước khi ghi |
| DIR33 | SYS/MOD | Module chỉ sử dụng DUser, hệ thống thiếu bốn thư mục còn lại | Module chạy khi DUser/tệp hợp lệ; Setup/QA báo thiếu bốn |

**Bất biến:** `DIRECT-only` · `DataManifest xác định phạm vi từng module` · `không FATAL vì tài nguyên không sử dụng` · `Setup/QA kiểm đủ năm DIRECT, không bắt buộc vị trí/thứ tự` · `thư mục nguồn không bắt buộc đăng ký` · `không fallback đường dẫn` · `không giám sát liên tục trong hộp thoại`.
