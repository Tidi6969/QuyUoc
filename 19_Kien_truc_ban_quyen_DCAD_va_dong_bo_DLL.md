# DCAD-STD-19 — Kiến trúc bản quyền DCAD và đồng bộ DCADManage ↔ DLL

**Mã quy ước:** 19  
**Phiên bản:** 1.2  
**Ngày cập nhật:** 15/11/2026  
**Mức áp dụng:** BẮT BUỘC khi phát triển, quản trị, cấp phép hoặc tích hợp bản quyền DCAD. Chỉ chấm yêu cầu trong phạm vi chức năng đã triển khai; hạng mục chưa có bằng chứng là `NotVerified`.  
**Hợp đồng:** `DCAD-LICENSE-ARCH-001`; bổ sung `DCAD-LICENSE-CHAIN-007` tại §13–14.  
**Tham chiếu:** DCAD-STD-01, 03, 04, 05, 07, 13, 15, 16, 17 và 18. STD-19 không thay các quy chuẩn này.

## 1. Mục tiêu và phân chia 8 tầng

Một hệ thống Dmanage cấp quyền tập trung cho nhiều DLL CAD. Công cụ và tính năng có thể tăng theo phiên bản, không hardcode danh mục trong Dmanage. Một File ID có thể cấp nhiều công cụ và từng tính năng với thời hạn khác nhau.

| Hệ thống | Tầng | Trách nhiệm |
|---|---|---|
| Dmanage | 1. Tool/Feature Registry | Đăng ký và quản lý danh tính công cụ/tính năng |
| Dmanage | 2. Package/Entitlement | Quản lý gói, quyền, thời hạn riêng |
| Dmanage | 3. Integration Export | Duyệt manifest và xuất mã/chứng nhận tích hợp |
| Dmanage | 4. Key/Issuance/Audit | Bảo vệ khóa, ký file, quản trị tài khoản, ghi lịch sử |
| DLL Development | 5. Manifest | Công cụ tự khai báo các tính năng có thật |
| DLL Development | 6. License Gate | Gắn điểm kiểm tra trước nghiệp vụ cần cấp phép |
| License Runtime | 7. License Client | Giải mã (nếu áp dụng), xác minh và đánh giá quyền |
| License Runtime | 8. Hardening/QA | Chống can thiệp, kiểm chứng, bảo trì và khôi phục |

**Bất biến:** DLL khai báo *có gì*; Dmanage quyết định *đăng ký và cấp quyền nào*; License Client quyết định *có được chạy hay không*. Dmanage không thể tự suy ra các hàm nghiệp vụ của DLL hay tự bảo vệ hàm chưa có gate.

## 2. Tool ID, Feature ID và UID

*Xem quy tắc sinh, vòng đời, tính duy nhất và tương thích từng định danh tại mục 12.*

1. Mỗi công cụ khai báo `ToolId` ổn định, ví dụ `DPunch`. Mỗi tính năng khai báo `FeatureId` ổn định **trong phạm vi Tool**, ví dụ `HOLE_THREAD`. Mã dùng ASCII không dấu, không khoảng trắng; so sánh chính xác, phân biệt hoa thường; không tự đổi nghĩa.
2. Dmanage cấp `ToolUid` và `FeatureUid` dạng UUID v4 duy nhất khi đăng ký lần đầu. UID là **danh tính công khai, không phải bí mật**, không thay chữ ký số. Một FeatureUid thuộc đúng một ToolUid.
3. Cùng công cụ qua C22/C25/G27 dùng cùng ToolUid; cùng nghiệp vụ dùng cùng FeatureUid. Thông tin CAD Host, framework, assembly và phiên bản build là các thuộc tính khác.
4. Nhập lại manifest giữ UID khi ID + quan hệ chủ sở hữu không đổi; chỉ cấp UID mới cho tính năng mới đã duyệt. Đổi tên hiển thị/ngôn ngữ không đổi UID. Mã đã xóa/ngừng hoạt động phải giữ tombstone, không tái sử dụng.
5. Registry lưu `Draft/Approved/Retired`, version manifest, checksum, người duyệt và audit. Chỉ mục `Approved` được cấp License và xuất chứng nhận.
6. Đổi chủ sở hữu, đổi nghĩa mã, xóa tính năng hoặc trùng ID phải được báo thành thay đổi có rủi ro; không âm thầm di trú quyền đã cấp.

## 3. Tool Manifest là danh mục động

Mỗi tool nộp một manifest **UTF-8 JSON** trong nguồn `Buildtime/DCAD.ToolManifest.json`, ví dụ:

```json
{
  "SchemaVersion": 1,
  "Publisher": "DCAD",
  "ToolId": "DPunch",
  "ToolVersion": "V391",
  "TargetHosts": ["C22"],
  "Features": [
    {"FeatureId":"HOLE_CIRCLE","Name":"Tạo lỗ tròn","Group":"Hole","LicenseMode":"FeatureRequired"},
    {"FeatureId":"HOLE_THREAD","Name":"Tạo lỗ ren","Group":"Hole","LicenseMode":"FeatureRequired"}
  ]
}
```

- `ToolId`, `ToolVersion`, `TargetHosts`, `FeatureId`, `LicenseMode` là đầu vào hợp đồng; `Name/Group` chỉ dùng hiển thị và phải theo STD-07 khi có UI.
- `LicenseMode=FeatureRequired`: phải có cả ToolGrant và FeatureGrant. `LicenseMode=ToolOnly`: thao tác thuộc phạm vi ToolGrant, không chế FeatureGrant giả; không được biến tính năng đã quyết định cấp riêng thành ToolOnly để lách kiểm tra.
- Manifest không có thời hạn khách hàng, mật khẩu, mã máy hay private key. Thời hạn do quản trị cấp tại **từng giao dịch License**.
- Dmanage phải validate schema/version, mã trùng, owner, bản thay đổi, phiên bản và phê duyệt. Manifest **không tự chứng minh nguồn chính thức** chỉ vì ghi `Publisher=DCAD`.
- Build/QA phải đối chiếu FeatureId khai báo với mã được dùng và các vị trí License Gate. Chỉ có file JSON không chứng minh tính năng đã được bảo vệ.

## 4. Quy trình đăng ký và xuất bộ tích hợp

*Danh sách `.g.cs` dưới đây là ví dụ adapter managed. Với ARX/GRX native, bộ mã `.h/.cpp` và hai ZIP theo chuỗi 7 bước được quy định tại §13. Không được coi một file mẫu hoặc bộ mã sinh là bằng chứng đã tích hợp.*

**Luồng phát triển → quản trị → phát triển:**

1. Người phát triển đặt mã Feature ID ổn định và kiểm tra quyền tại điểm nghiệp vụ.
2. Build sinh/kiểm tra manifest từ danh mục nguồn duy nhất; người phát triển gửi manifest/bản build để Dmanage thẩm định.
3. Dmanage so sánh Registry, giữ hoặc cấp UID, hiển thị diff và yêu cầu người có quyền phê duyệt.
4. Dmanage xuất **bộ tích hợp đã được duyệt**, có version và hash:

```text
<ToolId>-LicenseIntegration/
├── DCADLicenseIds.g.cs
├── DCADLicenseGate.g.cs
├── DCADLicenseContract.json
├── DCADToolCertificate.bin
├── IntegrationReport.txt
└── README.txt
```

5. Nhà phát triển đưa mã sinh vào source/buildtime của nhánh CAD, nhúng chứng nhận vào DLL khi build, không yêu cầu đọc các file tích hợp rời từ đường dẫn tùy tiện ở runtime.
6. Build và kiểm thử độc lập tính khớp ToolUid/FeatureUid, host, chứng nhận, các gate và tình huống thiếu quyền.

- Chứng nhận Tool/Feature phải được ký bởi **khóa phát hành được DCAD ủy quyền** và xác minh dựa trên trust anchor riêng, không tin khóa công khai tự đưa trong file.
- Bộ xuất **tuyệt đối không mang** khóa gốc/private signing key, khóa bí mật Dmanage hay password.
- UID/chứng nhận có thể bị sao chép; chúng **không tự chứng minh DLL chưa bị sửa**. Khi cần bảo vệ mã phải bổ sung ký mã phát hành, build hash được duyệt và hardening đã kiểm thử.
- Dmanage không được tự chèn lệnh check vào vị trí nghiệp vụ bằng cách dò chuỗi mã nguồn rồi khẳng định đã bảo vệ.

## 5. Gói, ToolGrant và FeatureGrant

Mô hình chuẩn: **License → ToolGrant → FeatureGrant**. Gói/bundle chỉ là cơ chế chọn sản phẩm; khi phát hành phải **snapshot quyền thật** để thay đổi gói sau đó không làm License cũ biến đổi.

- ToolGrant: `ToolUid`, `Enabled`, `ExpiresUtc` hoặc `NEVER`.
- FeatureGrant: `ToolUid` cha, `FeatureUid`, `Enabled`, `ExpiresUtc` hoặc `NEVER`.
- Một File ID có thể chứa nhiều Tool và Feature, mỗi mục có ngày hết hạn riêng.
- Một FeatureRequired chỉ được chạy nếu License/chữ ký/máy/host hợp lệ, Tool đang bật còn hạn, Feature thuộc đúng Tool, đang bật và còn hạn.
- Tool hết hạn → **tất cả tính năng con từ chối**, kể cả con có `NEVER`; một Feature hết hạn không được ảnh hưởng các Feature khác đang hợp lệ.
- **Deny by default:** không có grant, không có ID, sai UID/owner, feature bị gỡ, sai thời hạn hoặc lỗi đọc → từ chối. License cũ **không tự có quyền cho Feature mới**; không có chế độ `AllowAllFeatures` ngầm.
- Với các thao tác `ToolOnly` được khai báo và duyệt, chỉ cần ToolGrant. Không ép mọi công cụ phải chia nhiều tính năng không cần thiết.
- Thời gian UTC, dấu mốc hết hạn và điều kiện biên (ví dụ `now >= ExpiresUtc`) phải thống nhất trong schema và test ở đúng giây hết hạn.

## 6. File ID, nguồn phát hành và mật mã

1. File ID chứa `SchemaVersion`, `LicenseId`, `IssuedUtc`, `IssuerId`, `KeyId`, `MachineId`, CAD hosts, danh sách Tool/Feature grants và thời hạn. Mọi trường có ý nghĩa phải nằm trong **payload ký số canonical**.
2. Chỉ tin chữ ký của issuer nằm trong chuỗi chứng nhận từ **DCAD Root Trust** đã ghim sẵn ở phía xác minh; không chấp nhận khóa công khai tự khai trong File ID là căn cứ tin cậy. `CreatedBy=DCADManage` đơn lẻ không chứng minh file do app gốc tạo.
3. Khóa ký riêng thuộc Dmanage/thiết bị ký được ủy quyền; **không bao giờ** đưa khóa ký vào DLL CAD, License Client hay bộ mã tích hợp.
4. File ID phát hành mới nên là **nhị phân được mã hóa và xác thực** bằng thuật toán chuẩn, khóa ngẫu nhiên và nonce/IV đúng chuẩn; chữ ký số bảo vệ xuất xứ và quyền, mã hóa bảo vệ bí mật dữ liệu. Không chế thuật toán bằng ký tự đặc biệt.
5. Phải **tách khóa giải mã máy khách** (ví dụ khóa riêng CNG/TPM của máy nhận .dreq) khỏi khóa ký của nhà phát hành. Phải quy định rõ cài lại Windows, đổi máy, cấp lại và sao lưu. Dùng TPM không được mặc định máy nào cũng có hoặc đảm bảo chống trích xuất tuyệt đối.
6. Việc đổi tên, sao chép, nén RAR hoặc chuyển Internet không làm mất hiệu lực chữ ký nếu file giữ nguyên byte. Notepad Save giữ nguyên byte vẫn hợp lệ. Sửa byte được xác thực phải bị từ chối. Windows Properties/`Date Modified`/EXIF không phải chứng minh nguồn gốc; hash file gốc trong hồ sơ Dmanage chỉ là đối chiếu bổ sung.
7. License V9 cũ chỉ có Tool grants; không tự chuyển thành toàn bộ quyền Feature. Reader legacy phải riêng và có migration hoặc fail-closed rõ ràng. `Format=4` nếu dùng là **schema dự kiến**, không được mặc nhiên coi đã code.
8. Không cam kết “chỉ app DCAD đọc được tuyệt đối” khi phải giải mã/chạy offline trên máy khách: người kiểm soát máy vẫn có thể phân tích runtime. Bảo vệ khóa, ký số và QA là biện pháp chính.

## 7. Quản trị, khóa, audit, loại License

- Chuỗi tin cậy: **DCAD Root Key → Authorized Issuer Key → Tool Certificate / License**. Không phân phối khóa gốc/issuer riêng tới khách hàng. Phải có `KeyId`, luân chuyển, khôi phục, audit và chính sách thu hồi/đổi khóa; ưu tiên HSM/TPM cho khóa ký nếu có.
- Dmanage phân quyền cấp phép, duyệt Tool/Feature, xuất bộ tích hợp, tạo/cấp lại License, backup và audit. Vai trò có thể gồm `SUPER_ADMIN`, `LICENSE_MANAGER`, `SALES`, `AUDITOR`; tài khoản không được tự nâng quyền hoặc cấp License khi không có quyền.
- Nhật ký ghi ai, lúc nào, Tool/Feature UID, thay đổi và hash; **không ghi** mật khẩu, private key hoặc dữ liệu nhạy cảm không cần thiết.
- Tài khoản `admin/admin` được chấp nhận **chỉ ở môi trường phát triển theo yêu cầu hiện tại**. **Không sử dụng mặc định này với khóa ký sản xuất**; phát hành thương mại phải có cổng đổi credential/ủy quyền an toàn và kiểm thử.
- Hai chế độ đích: `FILE_ID` offline (ưu tiên thực hiện trước) và `USB_DONGLE` (giai đoạn sau, challenge-response bằng phần cứng bảo mật, không coi USB serial đủ chống giả mạo). Cùng dùng Tool/Feature Registry. Khi đồng thời có hai License phải có chính sách chọn nguồn rõ và test riêng.
- Thu hồi tức thì hoặc chống lùi đồng hồ **không thể đảm bảo** cho License offline trên máy khách không nhận cập nhật; công bố chính sách và lựa chọn kiểm tra online/dongle khi cần.

## 8. License Client và chuẩn đường dẫn runtime

- Client cung cấp API tương đương `CheckTool(ToolUid)` và `CheckFeature(ToolUid, FeatureUid)`; trả `ALLOW / DENY / ERROR` với mã chẩn đoán. Đây là **giao diện mục tiêu**, không khẳng định code V9 đã có các API Feature.
- Gate tại **điểm thực thi nghiệp vụ**, không chỉ khi mở dialog hoặc khi khởi chạy CAD. Không cho thực thi nếu thiếu một điều kiện bắt buộc hoặc lỗi đọc/verify.
- Tool UID dùng chung các host; adapter theo AutoCAD 2022 (.NET Framework 4.8), AutoCAD 2025 (.NET 8 Windows), GstarCAD 2027 (.NET 8 Windows) **theo phạm vi từng dự án đã duyệt**. Build/NETLOAD/smoke test từng host độc lập theo STD-03/04.
- Manifest trong `Buildtime`, mã sinh ở `src`, chứng nhận **nhúng DLL**. DLL portable ở `DProgram`. File ID runtime CAD ở `DUser` DIRECT theo STD-05 và DataManifest (`LogicalFolder/RelativePath/Required/AccessMode/schema`). Không dò từ root/cha hoặc fallback `AppData/TEMP` khi DUser bắt buộc lỗi.
- `Dmanage` là khu vực nguồn và ứng dụng quản trị **độc lập**, không phải tên thư mục logic CAD thứ sáu và không đăng ký trong Support File Search Path. Vị trí khóa/db quản trị là hợp đồng riêng của ứng dụng quản trị và phải có backup/restore, không tự áp dụng resolver CAD.
- Theo STD-03/07/13/16/18: không đổi UI, XData, geometry, logo hoặc nghiệp vụ CAD ngoài phạm vi để sửa lỗi license.

## 9. Trình tự triển khai và tương thích

*Trình tự kỹ thuật bên dưới được giữ để bảo toàn kiến trúc 8 tầng; luồng bàn giao/nghiệm thu chính thức gồm **7 bước liên thông** tại §13. Không bỏ qua các cổng QA hoặc tin cậy mật mã khi quy đổi giữa hai cách nhìn.*

1. **Chốt hợp đồng** Manifest, Registry, Tool Certificate, payload ký, trust anchor, API, Machine ID và schema/migration.
2. **Dmanage Registry:** import/diff/approve danh mục Tool/Feature, cấp/giữ UID, audit.
3. **Integration Export:** sinh mã `.g.cs`, contract JSON, chứng nhận ký, báo cáo; test không chứa private key.
4. **File ID:** cấp ToolGrant/FeatureGrant với thời hạn riêng, mã hóa/ký, reader legacy riêng.
5. **Client:** kiểm tra signatures, MachineId, host, UID, quyền, hết hạn, dữ liệu lỗi; negative tests.
6. **Tích hợp một DLL AutoCAD 2022 thật** và kiểm thử NETLOAD/thao tác; sau đó port C25/G27 theo phạm vi duyệt.
7. **Gia cố** môi trường sản xuất: tài khoản, ký mã, sao lưu, đánh giá chống can thiệp và lộ trình dongle.

Các dự án chưa tích hợp license không tự bị coi là sai STD-19; sau khi một feature được quảng bá là có license thì mọi đường thực thi feature đó phải kiểm tra đúng. Không chấp nhận “có chứng nhận là đã bảo vệ” hoặc “build PASS là license PASS”.

## 10. Ma trận QA DCAD-STD-19

*LIC01–LIC26 giữ nguyên. Các ca kiểm chứng theo bảy mắt xích nằm tại LIC27–LIC36 ở §13.10.*

| ID | Ca kiểm tra | Kết quả bắt buộc |
|---|---|---|
| LIC01 | Tool/Feature đăng ký lần đầu | UID duy nhất, đúng owner, có audit |
| LIC02 | Nhập lại manifest, đổi Name/Group | Giữ UID cũ |
| LIC03 | Bổ sung feature mới | UID mới, license cũ không tự được quyền |
| LIC04 | ID trùng/đổi owner/đổi nghĩa | Từ chối và hiện diff |
| LIC05 | Manifest chưa được duyệt | Không cấp chứng nhận/License |
| LIC06 | Xuất bộ tích hợp | Đúng ID/UID/version, không chứa private key |
| LIC07 | Certificate sai Tool/host/bản build | QA và gate từ chối theo hợp đồng |
| LIC08 | Tool + Feature hợp lệ còn hạn | ALLOW |
| LIC09 | Có Tool nhưng thiếu FeatureRequired | DENY |
| LIC10 | Tool hết hạn, Feature còn hạn/NEVER | DENY |
| LIC11 | Một Feature hết hạn, Feature khác còn hạn | Chỉ Feature hết hạn bị DENY |
| LIC12 | Feature sai ToolUid/UID giả | DENY |
| LIC13 | Sai máy hoặc host CAD | DENY |
| LIC14 | Chữ ký sai/issuer không trusted | DENY |
| LIC15 | Sửa header/IV/ciphertext/tag | DENY, không dùng dữ liệu |
| LIC16 | Đổi tên/copy/RAR byte không đổi | Vẫn xác minh được |
| LIC17 | License legacy chỉ có ToolGrant | Không tự có FeatureRequired mới |
| LIC18 | Private key bị đưa vào client/ZIP | Packaging QA FAIL |
| LIC19 | DUser thiếu, trùng DIRECT, không truy cập được | Cảnh báo và dừng chức năng phụ thuộc, không fallback |
| LIC20 | Không đọc được file/exception | DENY, không crash CAD, log an toàn |
| LIC21 | Feature không có gate thực | QA FAIL, không công bố bảo vệ |
| LIC22 | Cài lại Windows, chuyển máy, backup/mất key | Xử lý đúng chính sách cấp lại |
| LIC23 | Khóa issuer bị thay/thu hồi | Xác minh theo trust policy, audit |
| LIC24 | License offline bị thu hồi từ Dmanage | Không nói máy khách biết ngay nếu chưa nhận cập nhật |
| LIC25 | Build/NETLOAD/runtime/security/standards | Ghi trạng thái độc lập từng host |
| LIC26 | `admin/admin` với khóa ký sản xuất | Không được cấp License thương mại |

**Cổng nghiệm thu STD-16:** Mỗi ca thuộc phạm vi triển khai ghi `Applies | Expected | Actual | Evidence | Host | PASS/FAIL/NotVerified/N/A`. `N/A` chỉ dùng khi ngoài phạm vi thực sự; `NotVerified` không được coi PASS. Các ca static/test mô phỏng không thay việc build và chạy thật trên Windows/CAD.

## 11. Xung đột, phạm vi và hiệu lực

- **STD-03/05** có thẩm quyền về 5 thư mục runtime và DIRECT-only; **STD-04** về CAD target/build; **STD-07/13** về UI/logo; **STD-16** về bằng chứng và QA; **STD-17** về phiên bản/đồng bộ GitHub; **STD-18** về cách triển khai code; **STD-19** về bản quyền/UID/cấp phép/xác minh. Nếu có xung đột **báo người dùng**, không âm thầm sửa quy ước cũ.
- Mẫu JSON, `ToolCertificate`, API và các file `.g.cs` là **hợp đồng mục tiêu cần triển khai**, không phải xác nhận V9 đã có. V9 có nền tảng cấp Tool nhưng chưa có Tool/Feature Registry, cấp quyền theo Feature hay bộ mã tích hợp hoàn chỉnh; kiểm tra thực tế từng bản trước khi ghi PASS.
- STD-19 v1.0 hiệu lực từ **bộ quy ước V1.14**, không tự nâng phiên bản hoặc sửa code DCADManage/DLL chỉ vì cập nhật tài liệu.

## 12. Quy chuẩn tạo mã và quản lý định danh DCAD (bổ sung 13/11/2026)

*Các cột mô tả V15 trong mục này là dữ kiện lịch sử, không phải chứng nhận V25 đã đạt. Các quy tắc phiên bản/migration và cổng kiểm tra hiện tại đọc thêm tại §13–14.*

**Mục đích:** Mọi mã phải có quy tắc tạo, đầu vào, định dạng, phạm vi duy nhất, cách lưu và điều kiện thay đổi/khôi phục; không chấp nhận mô tả chung “Dmanage tự sinh”. Đây là **hợp đồng định danh** cho Dmanage, manifest, công cụ CAD, License Client và hồ sơ hỗ trợ. Phân biệt rõ **đã có trong Test-V15** với **quy tắc mục tiêu cần phát triển**.

### 12.1. Danh mục mã và công thức tổng quát

| Mã | Ai/cái gì tạo, công thức | Định dạng, phạm vi | Trạng thái đối chiếu V15 |
|---|---|---|---|
| Application ID | Người dùng khai báo; Dmanage kiểm tra trùng, lưu cố định | Ví dụ `Dtesr`; tên logic, **không phải** tên file `DtesrC22.arx` | **Mới đề xuất**; V15 chưa có Application Registry độc lập |
| CAD Host ID | Danh mục cố định, không sinh ngẫu nhiên | `C22` / `C25` / `G27` tương ứng AutoCAD 2022 / AutoCAD 2025 / GstarCAD 2027 | V15 phát hành C22; giấy phép đang sử dụng host token `AutoCAD2022` |
| Tool ID | Người phát triển/chủ ứng dụng đặt, Dmanage kiểm tra trùng | Ví dụ `Dcad1050`, giữ nguyên cho một công cụ | Có |
| Tool UID | UUID v4, sinh lần đầu, lưu Registry | 36 ký tự, hex thường và 4 dấu `-` | Có |
| Tool Version | Người phát triển tăng số theo STD-01/17 | `V14`, `V15`...; **không phải** CAD Host | Có |
| Feature/Process ID | Người phát triển khai báo; mã tuần tự theo tầng | `P01`, `P02`, `P01_S01` | Feature ID có; cây Process chưa có |
| Feature UID | UUID v4 riêng cho từng Feature, gắn Tool UID cha | 36 ký tự, duy nhất trong Registry | Có; chưa có quan hệ Process nhiều tầng |
| Tool Key ID | SHA-256 của khóa công khai Tool Signing Key | 12 byte đầu, 24 HEX **hoa** | Có |
| Issuer Key ID | Cùng cách tính Key ID, nhưng từ Issuer Public Key | 24 HEX **hoa**, khác Tool Key ID | Có |
| Machine ID | SHA-256 từ vật liệu máy chuẩn hóa, có nhánh dự phòng CNG | `DCAD1-` + 32 HEX **hoa** | Có |
| License ID | UUID v4 mới mỗi lần phát hành | 32 HEX **thường**, không gạch ngang | Có |
| Customer ID | UUID v4 khi tạo khách hàng | 32 HEX thường, không gạch ngang | Có |
| User/Issuer ID | UUID v4 khi tạo tài khoản; trong V15 IssuerId là ID người phát hành | 32 HEX thường, không gạch ngang | Có |

**Phân biệt bắt buộc:** Application ID ≠ Tool ID ≠ Tool UID ≠ Feature ID/UID ≠ Tool/Issuer Key ID ≠ Machine ID ≠ License ID. Tên file `dcad_license.id` không phải License ID bên trong. Mã nhận diện **không phải** khóa bí mật, chữ ký số, mật khẩu hay quyền sử dụng.

### 12.2. UUID v4: tạo, chống trùng và lưu giữ

1. Theo RFC 9562, UUID v4 gồm 128 bit: 4 bit version đặt `0100`, 2 bit variant đặt `10`, 122 bit còn lại lấy từ nguồn ngẫu nhiên mật mã. Trong mã V15 dùng `Guid.NewGuid()` trên Windows/.NET; không suy UID bằng cách băm Tool ID, ngày tháng hay đếm số thứ tự.
2. **Tool UID/Feature UID:** `Guid.NewGuid().ToString("D")`, mẫu `xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx`, với `y` là 8/9/a/b, chữ HEX thường; dài 36 ký tự.
3. **License ID/Customer ID/User ID:** `Guid.NewGuid().ToString("N")`, 32 HEX thường, không dấu gạch ngang.
4. Dmanage phải kiểm tra đúng format/UUID version, tính duy nhất tại Registry và các mã đã ngừng (tombstone); nếu va chạm, tạo lại và ghi nhận. Khả năng trùng xác suất rất thấp nhưng không bỏ kiểm tra ràng buộc duy nhất.
5. Đăng ký lại cùng Tool ID và chủ sở hữu phải **giữ Tool UID cũ**; nhập lại Feature ID cùng Tool UID phải **giữ Feature UID cũ**. Đổi host C22/C25/G27, tên hiển thị hay Tool Version không được tự sinh UID mới.
6. UID đã retired không tái sử dụng cho nghiệp vụ khác. Khôi phục Registry phải bảo toàn UID; không âm thầm sinh UID mới khi mất cơ sở dữ liệu rồi coi File ID cũ tương thích.

### 12.3. Application ID, Tool ID, CAD Host và phiên bản

1. **Application ID — quy tắc mục tiêu chưa triển khai:** chủ ứng dụng đặt mã ASCII không dấu/không khoảng trắng; đề xuất biểu thức `^[A-Za-z][A-Za-z0-9_]{1,63}$`. Ví dụ `Dtesr`, còn `DtesrC22.arx` là tên binary theo host. Kiểm tra trùng **không phân biệt hoa/thường** trong phạm vi Publisher, lưu nguyên mã gốc, không tái sử dụng mã đã ngừng.
2. **Tool ID:** ổn định và không tự đổi khi cập nhật. V15 có hai ràng buộc **chưa thống nhất**: manifest `^[A-Za-z][A-Za-z0-9_]{1,63}$`; License Format 4 `^D[A-Za-z0-9]{1,79}$`. Công cụ đi qua File ID Format 4 hiện phải thỏa **cả hai**. Không giả định một regex mới đã được triển khai hoặc tự đổi ID để qua kiểm tra.
3. **CAD Host:** quản lý mã ngắn `C22`, `C25`, `G27` nhưng **File ID dùng token canonical có chữ ký**; V15 thực tế dùng `AutoCAD2022`. Phải có bảng ánh xạ có phiên bản, QA từng host; đổi token đã ký chỉ bằng schema/migration có chứng cứ. Không cấp quyền chỉ vì tên ARX có hậu tố C22/C25/G27.
4. **Tool Version:** `V` + số nguyên dương; trong V15 manifest chấp nhận `^V[1-9][0-9]{0,8}$`. Một Tool UID có thể có nhiều build/host/version; mỗi binary cần đường dẫn, Host và SHA-256 riêng. Build mới không tạo Tool UID hoặc Tool Key ID mới nếu không có sự kiện đổi danh tính/luân chuyển khóa.

### 12.4. Feature/Process ID nhiều tầng (quy tắc mục tiêu; chưa code trong V15)

1. Feature ID hiện có phải thỏa `^[A-Z][A-Z0-9_]{1,63}$`, duy nhất **trong phạm vi Tool**. Để đặt mã quá trình: `P` + số nguyên dương với ít nhất hai chữ số: `P01`, `P02`, `P10`, `P100`; cấm hai cách viết cùng một số, ví dụ `P1` và `P01`.
2. Quá trình con: `P01_S01`, `P01_S02`; nếu cần sâu hơn: `P01_S01_S01`. Mỗi đoạn `S` đánh số tương tự. Đường dẫn hiển thị có thể là `Dcad1050/P01/P01_S01`, nhưng **Feature ID không nối Tool ID vào thành chuỗi khác**.
3. Cấp một Feature UID v4 cho mỗi Process riêng; lưu chủ sở hữu Tool UID và quan hệ `ParentFeatureUid` (hoặc cấu trúc cha tương đương) tường minh. Kiểm tra không trùng, không chu trình, không nhận cha thuộc Tool khác; không suy quyền chỉ bằng cắt chuỗi mã.
4. **Phân quyền theo cấp:** tại điểm nghiệp vụ phải có giấy phép hợp lệ, ToolGrant hợp lệ, **mọi FeatureGrant bắt buộc của các cấp cha**, và FeatureGrant hiện tại hợp lệ. Cha bị tắt/hết hạn ⇒ con DENY. License cũ không tự có quyền Feature mới; mọi đường thực thi nghiệp vụ có bảo vệ phải có gate thực.
5. Đây là thay đổi **schema và quyền mục tiêu**, không phải hành vi đã có trong File ID Format 4 V15. Khi lập trình phải cập nhật payload ký, phiên bản hợp đồng, kiểm thử migration/compatibility và các ca DENY.

### 12.5. Tool Key ID và Issuer Key ID: công thức byte chính xác của V15

1. V15 tạo cặp khóa ký Tool **RSA 3072 bit**; chuỗi khóa công khai lấy bằng `rsa.ToXmlString(false)`, tuyệt đối không đưa private key vào manifest, ZIP tích hợp, ARX hoặc máy khách.
2. Công thức trong `Dmanage/Core/LicenseCore.cs`:
   **`KeyId = UpperHex(SHA256(UTF8(PublicKeyXml))[0..11])`**.
   SHA-256 trả 32 byte, lấy **12 byte đầu**, biểu diễn thành **24 ký tự HEX HOA** không dấu phân cách.
3. Issuer Key ID dùng **cùng thuật toán** nhưng trên **khóa công khai Issuer**, không dùng Tool public key. Tool/Issuer Key ID chỉ là mã chọn khóa để xác minh chữ ký theo trust chain DCAD; Key ID không phải chữ ký hoặc bằng chứng hợp lệ tự thân.
4. V15 băm trực tiếp byte UTF-8 của chuỗi RSA XML, **không phải RFC 7638/JWK thumbprint** và chưa có canonical XML chung. Phải cố định cách xuất XML và lưu chính xác bản công khai đã đăng ký; thay whitespace/định dạng XML có thể làm Key ID đổi dù cùng khóa toán học.
5. Tăng Tool Version/build host mới **không tự đổi Key ID**. Chỉ đổi khi có luân chuyển cặp khóa; phải lưu lịch sử Key ID, chính sách giấy phép cũ, audit và thu hồi/khôi phục phù hợp.
6. **Recipient CNG private key của máy khách khác hoàn toàn khóa ký Tool/Issuer.** Khóa này dùng giải mã File ID, không bao giờ đưa vào `.dreq` hoặc thông tin hỗ trợ.

### 12.6. Machine ID: công thức đầu vào chính xác của V15

1. Đọc WMI tuần tự: `Win32_ComputerSystemProduct.UUID`, `Win32_BaseBoard.SerialNumber`, `Win32_BIOS.SerialNumber`. Với từng giá trị: `Trim()`, `ToUpperInvariant()`; loại trống, giá trị dài trên 160 ký tự và placeholder sau: `TO BE FILLED BY O.E.M.`, `DEFAULT STRING`, `NONE`, `UNKNOWN`, `NOT SPECIFIED`, `SYSTEM SERIAL NUMBER`, `INVALID`, `N/A`, `NO ASSET INFORMATION`. Chỉ giữ ký tự `char.IsLetterOrDigit`; loại chuỗi sau lọc ngắn hơn 6 ký tự hoặc tất cả ký tự giống nhau.
2. Nếu **có ít nhất hai giá trị hợp lệ**, nối chuỗi *đúng thứ tự và ký tự LF*:
   `"DCAD-MACHINE-ID-V1\n" + [dòng "UUID=<value>\n" nếu hợp lệ] + [dòng "BOARD=<value>\n" nếu hợp lệ] + [dòng "BIOS=<value>\n" nếu hợp lệ]`.
   Không thêm trường bị thiếu; có LF kết thúc; **không thay LF bằng CRLF**.
3. Nếu **dưới hai nguồn hợp lệ**, dùng khóa công khai CNG máy khách:
   `"DCAD-MACHINE-KEY-FALLBACK-V1\n" + RecipientPublicXml + "\n"`.
   V15 dùng khóa RSA 3072 bit tên `DCAD-License-Recipient-V1` từ `MicrosoftSoftwareKeyStorageProvider`; khóa riêng cấu hình `ExportPolicy=None`, gắn profile Windows.
4. Với cả hai nhánh, công thức:
   **`MachineId = "DCAD1-" + UpperHex(SHA256(UTF8(material))[0..15])`**.
   SHA-256 lấy **16 byte đầu** ⇒ 32 ký tự HEX HOA. Tiền tố `DCAD1-` định danh thuật toán V1, không tự thay khi đang có File ID đã cấp.
5. File yêu cầu `.dreq` V15 chứa **Machine ID và khóa Recipient công khai**, không chứa private key. Machine ID khớp một mình **không chứng minh** máy có private key; khi kích hoạt còn phải giải mã, xác minh chữ ký, host, hạn và quyền.
6. Khi thay mainboard/SMBIOS, mất khóa CNG, cài lại Windows hoặc chuyển profile có thể cần cấp lại File ID. Việc thay thuật toán Machine ID phải dùng schema/phân nhánh phiên bản mới, kiểm tra hồi quy Managed/Native và không vô hiệu hóa giấy phép cũ một cách âm thầm.
7. Không hiển thị serial phần cứng thô hoặc private key trong phiếu hỗ trợ. Machine ID là mã nhận diện/phục vụ đối chiếu, **không phải bí mật**.

### 12.7. License ID, Customer ID, Issuer ID và thông báo lỗi

1. Mỗi giao dịch phát hành File ID sinh **License ID mới** theo UUID v4 dạng `N`; không tính từ Machine ID, Tool ID hay ngày tháng. Gia hạn/cấp lại bằng lần phát hành mới => **License ID mới**, hồ sơ phải lưu quan hệ với bản cũ; không ghi đè định danh hoặc dùng lại License ID cũ.
2. **Customer ID:** tạo một lần khi tạo hồ sơ khách hàng. **User/Issuer ID:** tạo một lần khi tạo tài khoản, trong V15 `IssuerId` của File ID là **ID tài khoản phát hành**, không phải Issuer Key ID. Giữ các ID khi sửa hồ sơ; xóa/khôi phục theo audit.
3. Phân biệt ba trạng thái: **đã tìm thấy file ID**, **ID giải mã/xác minh hợp lệ**, **được quyền thực thi Tool/Process**. Chỉ trạng thái sau cùng được `ALLOW` tại License Gate.
4. Giao diện thông báo lỗi mục tiêu cần có: Application/Tool ID, Tool Version, CAD Host, trạng thái ID, trạng thái xác minh, **mã lỗi ổn định**, cách khắc phục và khả năng sao chép phiếu hỗ trợ. License ID chỉ được hiển thị với ghi chú phù hợp nếu đã xác minh an toàn, không tin ID tự khai trong file hỏng.
5. **Mã lỗi mục tiêu (chưa khẳng định V15 đã hỗ trợ):** `LIC-NOT-FOUND`, `LIC-SIGNATURE`, `LIC-MACHINE-MISMATCH`, `LIC-HOST-DENIED`, `LIC-EXPIRED`, `LIC-FEATURE-DENIED`, `LIC-CLIENT-KEY-MISSING`, `LIC-FORMAT`. Phải phân biệt thiếu file, không mở được khóa CNG, sai máy và lỗi xác minh; mặc định từ chối khi không chắc chắn.
6. Phiếu hỗ trợ chỉ gồm mã lỗi, Tool, Host, phiên bản, giờ UTC, Machine ID và License ID **nếu đã xác minh**. Không gửi password, private key, raw hardware serial hoặc vật liệu ký số.

### 12.8. Vòng đời và ma trận kiểm chứng STD-18

- Với **mọi loại mã**, hồ sơ quy chuẩn phải có: `Algorithm`, `Inputs`, `Encoding/Normalization`, `Format`, `UniquenessScope`, `Persistence`, `Rotation/Migration`, `SourceOfTruth`. Mã băm bắt buộc chỉ rõ tiền tố domain, thứ tự trường, số byte cắt, chữ hoa/thường.
- Không được thay UID, Machine ID, khóa hoặc mã định danh đang có trong V15 chỉ vì đổi tài liệu. Quy tắc mới phải được thực hiện bằng thay đổi code/schema riêng khi người dùng cho phép, kèm migration/test và giữ chức năng cấp phép đã nghiệm thu.
- **QA-ID01:** UUID v4 đúng format/version, cấp duy nhất, nhập lại manifest giữ UID, mã retired không được tái sử dụng.
- **QA-ID02:** Application/Tool/Process trùng, đổi nghĩa, sai owner, sai cha hoặc có chu trình phải bị từ chối và ghi audit.
- **QA-ID03:** tính Key ID trùng khớp byte SHA-256 của public XML, đổi khóa dẫn đến ID mới, chữ ký/khóa không trusted phải DENY, không đóng gói private key.
- **QA-ID04:** cùng SMBIOS chuẩn hóa cho cùng Machine ID; kiểm tra placeholder, thứ tự dữ liệu, LF, CNG fallback, Managed/Native parity.
- **QA-ID05:** sai máy, thiếu/mất Recipient Key, host không được cấp, thiếu quyền hoặc hết hạn phải DENY với lỗi đúng, không crash CAD.
- **QA-ID06:** mỗi lần phát hành mới có License ID mới, vẫn tra cứu được khách hàng, Issuer, Tool, liên kết cấp lại và dấu vết audit.
- **QA-ID07:** kiểm thử mapping CAD Host và binary/hash theo C22/C25/G27 riêng; không suy PASS cho host chưa thử.
- **QA-ID08:** từng Process có Gate thật, Tool/cha/con hết hạn, License cũ thiếu Process mới => DENY, dữ liệu ký không bị lách.
- Từng ca trong phạm vi phải ghi `Expected / Actual / Evidence / Host` và `PASS / FAIL / NotVerified / N/A` theo STD-16/18. **Cập nhật quy ước không đồng nghĩa đã code hoặc test PASS**.

**Nguồn đối chiếu:** `Test-V15.zip`, `Dmanage/Admin/ToolRegistryService.cs`, `Dmanage/Admin/AdminService.cs`, `Dmanage/Core/LicenseCore.cs`, `Dmanage/Core/MachineId.cs`, `Dmanage/Core/ClientRecipientKey.cs`, `Dmanage/Contracts/ToolManifestContract.cs`, `Dmanage/Core/LicenseFormat4.cs`; chuẩn UUID v4: RFC 9562.
## 13. Hợp đồng bắt buộc của chuỗi 7 bước (bổ sung 15/11/2026)

**Hợp đồng:** DCAD-LICENSE-CHAIN-007. **Phạm vi:** ứng dụng quản trị DCADLicense/Dmanage, bộ xuất SourceStructure/IntegrationRequest, mã License Gate/Client và từng add-in CAD được bảo vệ. Đây là **quy trình nghiệm thu**, không phải tuyên bố các chương trình hiện có đã PASS.

**Chuỗi chính thức, không được bỏ mắt xích:** **Căn cứ thiết kế → Hai ZIP xuất ra → Mã tích hợp → Build → QA → Cấp key → License Gate thực thi.**

### 13.1. Bất biến dùng chung cho bảy bước

1. **Một nguồn danh tính:** Registry đã duyệt là nguồn gốc duy nhất của Application ID/UID, Tool ID/UID, Feature ID/UID, chủ sở hữu, các host được duyệt và chính sách Edition. Không sinh UID mới khi tạo ZIP, build, cấp key hoặc đổi CAD host.
2. **Một hợp đồng bất biến theo bản phát hành:** mỗi hồ sơ phải tham chiếu đúng **byte SHA-256 của DCAD.ToolContract.json** đã duyệt. Phải định nghĩa cách serialize canonical (UTF-8 không BOM, thứ tự trường, số, dấu xuống dòng) theo SchemaVersion; không băm tùy tiện JSON sau khi parse rồi xuất lại.
3. **Tách danh tính và quyền:** Tool ID/Feature ID là mã nghiệp vụ; Tool UID/Feature UID là UUID v4 được đối chiếu khi cấp quyền/chạy; ToolVersion là phiên bản nghiệp vụ, không phải CAD host; Edition ST/PRO/UL không tự đồng nghĩa mọi Feature đều được cấp.
4. **Tách checkpoint và Feature:** P01/P02/P01_S01 là mã **điểm hoặc quá trình tích hợp** nếu chưa khai báo là tính năng có UID riêng. Không suy FeatureGrant bằng cách cắt chuỗi P01. Nếu cấp phép theo cây tính năng, phải có schema ParentFeatureUid, quyền các cấp cha và kiểm thử migration trước khi sử dụng.
5. **Tách loại bằng chứng:** ZIP mẫu, JSON, mã nguồn có lời gọi Gate, build PASS, QA tĩnh, QA runtime và license đã ký là các trạng thái khác nhau. Không suy bước sau PASS chỉ vì bước trước PASS.
6. **Tin cậy theo chữ ký:** SHA-256 dùng để đối chiếu tính toàn vẹn với bản đã duyệt, **không thay chữ ký hoặc chứng nhận**. Khóa Issuer được tin phải đến từ root/chuỗi ủy quyền ghim trước theo §6–7; khóa public tự chứa trong ZIP hoặc trong File ID không tự tạo trust.
7. **Không phá tương thích ngầm:** các token host đã ký, Machine ID, canonical payload, Tool/Feature UID, thuật toán ký, đường dẫn DUser DIRECT và format license cũ phải giữ nguyên cho đến khi có schema mới + migration + negative test.
8. **Không key sản xuất khi thiếu bằng chứng:** thiếu Root/Issuer trust chain, xác minh .dreq, QA bản build hoặc Gate runtime có liên quan ⇒ không công bố hồ sơ là sẵn sàng phát hành thương mại. Không thay bằng khóa mẫu, kiểm tra giả hoặc cho phép vượt Gate.

### 13.2. Bước 1 — Căn cứ thiết kế

**Đầu vào:** hồ sơ công cụ, định danh được khai báo bởi nhà phát triển, manifest tính năng thực có, danh sách host và chính sách ST/PRO/UL. **Xử lý:** kiểm tra schema, ID trùng, owner, UID, trạng thái Draft/Approved/Retired, quan hệ Feature–Tool, thay đổi chính sách; ghi diff và người duyệt. **Đầu ra:** Registry Approved có audit, ToolContract theo schema hiện hành và SHA-256 của đúng byte hợp đồng.

**Nhà phát triển khai báo** Tool/Feature ID và nghiệp vụ thực tế; **ứng dụng quản trị** cấp/giữ Tool/Feature UID. Không đưa Device ID, .dreq, hạn khách hàng hoặc private key vào ToolContract. Một Tool UID dùng chung C22/C25/G27 theo cùng công cụ; ToolVersion và CAD target phải được xác định ở từng bản build.

**Cổng 1:** không có trạng thái Approved, có UID trùng/sai owner, Feature thiếu UID hoặc ToolContract không tái tạo được cùng SHA-256 ⇒ **STOP**, không xuất hồ sơ tích hợp chính thức.

### 13.3. Bước 2 — Hai ZIP xuất ra, vai trò và cách đọc

| Gói | Người tạo | Nội dung tối thiểu | Người đọc / mục đích |
|---|---|---|---|
| **SourceStructure ZIP** | Bộ phát hành | DCAD.ToolContract.json, manifest cấu trúc nguồn, hash hợp đồng, cây host thuộc phạm vi duyệt, adapter/mã Gate tương ứng hoặc vị trí chèn đã quy định, README trạng thái | Nhà phát triển lấy cấu trúc làm nền; **không** được coi ZIP mẫu là nguồn ARX/GRX nghiệp vụ đã hoàn thiện |
| **IntegrationRequest ZIP** | Yêu cầu build | **Cùng byte** DCAD.ToolContract.json, mã phiếu, Tool/host/version, Edition, danh sách Feature ID/UID, CheckPositions P01/P02, SourceRef/SourceSHA256, yêu cầu QA, bộ tích hợp liên quan | Nhà phát triển thực hiện yêu cầu cụ thể; bộ QA đối chiếu hồ sơ với nguồn và binary |

- Cả hai phải có **hợp đồng cùng SHA-256**, cùng Tool UID, Feature UID, host và phiên bản tương ứng; thay đổi Registry sau khi xuất làm hồ sơ cũ không còn tự động đạt.
- **SourceStructure là mẫu**, không khẳng định chứa toàn bộ source CAD thật; **IntegrationRequest là yêu cầu thiết kế**, không phải giấy chứng nhận build/QA hay key. ZIP nào chứa mã Gate cũng chưa chứng minh Gate được gọi.
- Mỗi gói phải nêu SchemaVersion, nguồn phát hành, mã/phiên bản công cụ, các hash nguồn và trạng thái được kiểm chứng; không chứa khóa ký riêng, password, secret hoặc private key CNG của khách hàng.
- Tên trường hash hiện còn khác nhau giữa một số tệp V25 như **toolContractSHA256**, **ToolContractSHA256**, **toolContractSha256**. Không dùng cách viết khác nhau làm nguồn sự thật độc lập: parser phải giải theo **schema và loại tệp**, đối chiếu cùng byte hash; khi chọn một tên canonical mới phải tăng schema/migration, không đổi ngầm chữ ký/file đã xuất.
- Khi Source ZIP được nhà phát triển nộp lại, phải đối chiếu **SourceSHA256** trên file ZIP thực, manifest được công cụ khai báo và hợp đồng Approved. Không tin trường Publisher hoặc đường dẫn tự khai là bằng chứng nguồn chính thức.

**Cổng 2:** thiếu một ZIP, SHA không khớp, mất Feature UID/Edition/target, sai source hash hoặc lấy dữ liệu từ Registry chưa duyệt ⇒ **STOP** trước build/QA.

### 13.4. Bước 3 — Mã tích hợp: tạo, chèn và xác minh

**Đối với ARX/GRX native:** bộ tích hợp có thể gồm ApprovedScope.h, IssuerPublicKey.h, DcadLicenseCheck.h/.cpp, DcadGate.h/.cpp, CngDetail.h và Integration.json; **đối với DLL managed:** bộ mã .g.cs tương đương theo host/framework đã duyệt. Tên tệp cụ thể phụ thuộc schema/adapter, nhưng **không thay đổi danh tính hay quyền**.

**Quy trình bắt buộc:**
1. Adapter lấy Tool UID, ToolVersion, host token, Feature ID → Feature UID, Edition tối thiểu từ ToolContract Approved; không nhập thủ công các UUID khác vào từng nhánh.
2. Mã công cụ thực sự gọi **CheckTool** trước thao tác ToolOnly; với FeatureRequired gọi **CheckTool + CheckFeature** tại tất cả đường thực thi tính năng. Kiểm tra trước khi sửa hình học/XData/bản vẽ hoặc thực hiện nghiệp vụ được bảo vệ.
3. Đường dẫn **dcad_license.id** được resolve từ **DUser DIRECT** qua hợp đồng DataManifest/STD-05; không đọc tùy tiện từ thư mục source, registry key bí mật, PARENT, TEMP hay AppData fallback khi DUser là bắt buộc.
4. Bộ tích hợp nhúng public trust anchor/chuỗi ủy quyền đã duyệt, UID và chứng nhận cần thiết trong binary; không mở rộng tin cậy chỉ vì file tích hợp đi kèm public key.
5. Tool Manifest do nhà phát triển khai báo phải khớp Feature ID thực tế, LicenseMode, các vị trí CheckPositions; không tự tạo manifest giả từ danh sách Registry rồi coi là bằng chứng code có feature.

**Cổng 3:** thiếu lời gọi Gate ở đường thực thi, ID trong code khác hợp đồng, thiếu trust anchor hợp lệ, chọn sai host, hoặc module chỉ chép thư viện nhưng không gọi Gate ⇒ **QA FAIL / không quảng bá đã bảo vệ**.

### 13.5. Bước 4 — Build có chứng cứ mới

- Build từng host **chỉ khi thuộc phạm vi duyệt**: C22 = AutoCAD 2022, C25 = AutoCAD 2025, G27 = GstarCAD 2027; SDK/framework/reference đúng host theo STD-03/04, không trộn header/lib giữa CAD.
- Hệ thống build nhận **ZIP nguồn thật**, phiếu tích hợp, ToolContract hash, SDK khai báo và mã Gate; không tự đổi Tool UID/Feature UID, Edition policy, version hay license schema.
- Ghi tên binary đúng target, SHA-256 của source ZIP và **binary mới** vừa build, toolchain, return code, log và thời điểm. Kết quả compiler PASS không thay chứng cứ tích hợp hoặc QA runtime.
- Build_All.cmd là điểm chạy duy nhất ở gói build; script con dùng /nopause, script tổng log/hiển thị màu/pause cuối khi chạy trực tiếp, bảo toàn ERRORLEVEL, không publish artifact cũ hoặc chỉ PASS một phần các target được giao.
- **DCADLicense.exe của ứng dụng quản trị** là sản phẩm riêng với **DJoinC22.arx / DJoinC25.arx / DJoinG27.grx** của add-in. Build ứng dụng quản trị không được coi là đã build add-in.

**Cổng 4:** thiếu log/return code, binary mới, SHA256, SDK, sai tên/host hoặc bất kỳ nhánh bắt buộc chưa PASS ⇒ **BUILD FAIL/NotVerified**, không đánh dấu release approved.

### 13.6. Bước 5 — QA độc lập với Build

- **QA tĩnh:** so ToolContract hash giữa Registry, hai ZIP, source và artifact manifest; Tool/Feature UID, host/version/Edition, public trust anchor/chứng nhận và mã gọi Gate; source ZIP hash, binary hash và danh mục vị trí P01/P02. Kiểm biểu hiện thiếu Gate nhưng không công nhận runtime chỉ qua tìm chuỗi nguồn.
- **QA thực thi:** dùng CAD/host thật với license đúng, thiếu file, sai chữ ký, sai máy, sai Tool UID, sai host/version, Edition dưới mức yêu cầu, thiếu FeatureGrant, hết hạn Tool/Feature, sửa byte license; thử mọi entry point của nghiệp vụ; bản vẽ không thay đổi khi DENY/ERROR.
- **QA riêng từng host:** C22, C25, G27 không kế thừa PASS từ host khác; với feature có parent, kiểm cả chuỗi cha/con đã ký; legacy license không mở Feature mới.
- **Chứng cứ:** lưu hồ sơ build, thời gian, binary hash, QA checklist, log và kết quả **PASS/FAIL/NotVerified** riêng cho build, gate static, CAD runtime và an toàn phát hành.

**Cổng 5:** chỉ khi các thử nghiệm bắt buộc của scope PASS mới đánh dấu QA Approved để được cấp key. QA static PASS nhưng CAD runtime NotVerified **không được công bố License Gate đã chạy thành công**.

### 13.7. Bước 6 — Người dùng yêu cầu .dreq và người quản trị cấp key

**Người dùng:** chạy lệnh **Dcad_id** tạo **.dreq** từ Device/Machine ID, khóa công khai recipient CNG, Request ID và chữ ký yêu cầu theo schema đã chốt. **Không** tự điền Tool UID, Edition hay ngày hết hạn vào .dreq.

**Dmanage/DCADLicense:** đọc byte .dreq, xác minh format, chữ ký và ràng buộc máy/recipient, lưu hash + bản gốc; liên kết khách hàng và nhiều quyền độc lập. Mỗi quyền chọn **một bản build QA approved** với Tool UID, host, ToolVersion, Edition, Feature UID, số suất và hạn riêng. Quyền con phải được phép theo chính sách Edition/Feature đã duyệt; không cấp Feature không nằm trong hợp đồng.

**Issuer:** tái xác minh .dreq, hạn, quyền, QA, source/binary hash và ToolContract rồi ký payload canonical. Mô hình là **License → ToolGrant → FeatureGrant**, không tự cấp mọi Feature chỉ vì có PRO/UL. Ký toàn bộ trường ảnh hưởng quyết định chạy; có License ID mới, Issuer/Key ID và biên nhận/audit, không ghi đè giấy phép cũ. Cơ chế key production phải đáp ứng chuỗi Root → Authorized Issuer của §6–7; private key không rời miền phát hành.

**Tương thích V25:** payload DCAD-LICENSE-1 trong mã nguồn V25 hiện diễn đạt dòng quyền **ToolUid|CadTarget|ToolVersion|-|Edition|ExpiresUnix** và **ToolUid|CadTarget|ToolVersion|FeatureUid|Edition|ExpiresUnix**; Tool ID và ToolContract SHA nằm trong Registry/biên nhận nhưng **không phải trường Gate V4 trực tiếp so trong payload đã ký**. Không tuyên bố Gate đang kiểm tra Tool ID/hash nếu chưa có schema mới và mã reader tương thích. Không chuyển máy khách sang schema mới hoặc mã hóa mới chỉ bằng sửa quy ước.

**Cổng 6:** .dreq lỗi, thiếu bằng chứng QA, quyền không thuộc Tool/Feature, khác host/version/Edition, thiếu issuer trust hoặc khóa sản xuất không đủ bảo vệ ⇒ **KHÔNG KÝ**. Chỉ phát hành khi hành động ký là thực, có biên nhận và tệp license đã xác thực.

### 13.8. Bước 7 — License Gate thực thi trong add-in

**Đầu vào:** license byte từ DUser DIRECT, trust anchor nhúng đã ủy quyền, device key ở máy khách, Scope có Tool UID, host, ToolVersion, Edition tối thiểu và Feature UID được yêu cầu.

**Thứ tự kiểm tra logic (phụ thuộc format):** (1) file/schema/giới hạn độ dài; (2) chữ ký Issuer và trust chain; (3) giải mã/xác thực nếu schema áp dụng; (4) thiết bị/recipient; (5) Tool UID; (6) CAD host token; (7) ToolVersion/Edition; (8) ToolGrant đang bật, còn hạn; (9) FeatureGrant thuộc đúng Tool, còn hạn và tất cả cha bắt buộc nếu schema hỗ trợ. Dữ liệu chưa xác thực không dùng để cho phép thực thi.

**Đầu ra:** ALLOW hoặc DENY/ERROR kèm mã chẩn đoán ổn định. Khi khác máy/host/tool/version, thiếu Feature hay mất DUser, code **không thực hiện nghiệp vụ được bảo vệ**, không gây crash CAD, không sửa bản vẽ. Thiếu file hoặc lỗi parse **không** chuyển thành license mặc định.

**Không đánh đồng:** add-in tìm thấy file ≠ file hợp lệ ≠ có quyền Tool ≠ có quyền Feature ≠ gate được gọi tại mọi điểm nghiệp vụ. Thu hồi offline chỉ có hiệu lực trên khách hàng khi cơ chế cập nhật/kiểm tra phù hợp đến được máy khách.

### 13.9. Sổ đối chiếu nguồn → kết quả → điều kiện dừng

| Mắt xích | Nguồn đọc chính | Kết quả phải giữ | Kiểm tra bắt buộc |
|---|---|---|---|
| Căn cứ thiết kế | Registry + Manifest nhà phát triển | ToolContract Approved, UID, chính sách, SHA | Duyệt, uniqueness, owner, schema |
| ZIP 1 & ZIP 2 | **Cùng** ToolContract + yêu cầu build | SourceStructure & IntegrationRequest | Contract SHA giống nhau, Source SHA đúng, danh mục Feature đủ |
| Mã tích hợp | Hai ZIP + source CAD thật | Header/adapter, manifest, gate tại nghiệp vụ | Mã ID/UID/host/Feature đúng, vị trí gọi thực |
| Build | Source ZIP + SDK/target + integration | Binary mới + hash + log | Compiler return code, đúng target/version |
| QA | Registry + source + binary + license thử | Build QA record và runtime evidence | LIC01–LIC26 + ca bổ sung, deny trước nghiệp vụ |
| Cấp key | .dreq thật + khách hàng + grants + QA | License được ký + biên nhận | Device, Tool/Feature, scope, Edition, hạn, issuer |
| Gate thực thi | License + trust anchor + Scope | ALLOW/DENY/ERROR | Ký, thiết bị, host/version, Tool/Feature grant |

### 13.10. Ma trận bổ sung cho chuỗi 7 bước (không thay LIC01–LIC26)

| ID | Trường hợp | Kết quả bắt buộc |
|---|---|---|
| LIC27 | Hai ZIP cùng ToolContract theo đúng byte SHA-256 | Khớp; thay đổi 1 byte hoặc sai Registry → STOP |
| LIC28 | SourceStructure chỉ là mẫu / IntegrationRequest là hồ sơ | Không công bố build/runtime PASS từ ZIP mẫu hoặc JSON |
| LIC29 | Features trong Registry khác hồ sơ/mã sinh | QA FAIL; không bỏ rơi Feature ID/UID khi xuất |
| LIC30 | C++ native và managed adapter trên host riêng | Không đổi UID/host/schema; mã Gate được gọi thật |
| LIC31 | Source SHA và binary SHA/host/version bị thay | QA FAIL; không cấp key cho binary không được duyệt |
| LIC32 | .dreq thật của khách hàng, gắn nhiều Tool/Feature | Verify máy/request, quyền từng mục, hạn riêng |
| LIC33 | ToolGrant hợp lệ nhưng FeatureRequired thiếu/sai UID | DENY trước xử lý hình học |
| LIC34 | Gate có tên trong ZIP nhưng không được gọi tại một entry point | QA FAIL, không công bố bảo vệ |
| LIC35 | Mã Host/ToolVersion/Edition thay đổi giữa Issuer và Gate | DENY hoặc migration được định nghĩa; không đổi token đã ký âm thầm |
| LIC36 | Root/Issuer certificate hoặc khóa signing production chưa hợp lệ | KHÔNG PHÁT HÀNH thương mại; ghi lý do cụ thể |

**Mỗi LIC27–LIC36:** ghi Applies, Expected, Actual, Evidence, Host, kết quả PASS/FAIL/NotVerified/N/A theo STD-16/18. Test tĩnh và hình thức ZIP không được thay thế test thật trong Windows/CAD.

## 14. Tương thích lịch sử và ranh giới nghiệm thu (bổ sung 15/11/2026)

1. **Giữ nguyên dữ liệu đã cấp:** Tool UID, Feature UID, Key ID, Machine ID, License ID, các token CAD được ký và License V4 hiện hữu không được thay đổi hoặc cấp lại chỉ vì cập nhật quy ước. Reader legacy phải chọn theo schema và fail-closed nếu không xác định được.
2. **V25 là mốc đối chiếu mã nguồn, không phải chứng nhận phát hành:** SourceStructure, IntegrationRequest, CheckTool/CheckFeature wrapper và cấp quyền Feature đã có mã tại V25, nhưng Windows MSBuild, .dreq→CNG→Gate thực tế, C22/C25/G27 và Root/Issuer trust chain đầy đủ **chưa có bằng chứng nghiệm thu**. Không mô tả chúng là PASS.
3. **Feature theo cây** (P01/P01_S01 là license Feature thực sự) chỉ được áp dụng khi có quan hệ cha, schema được ký và reader kiểm tra cha/con. Ở hợp đồng V25, mã P01/P02 là checkpoint tích hợp, không tự phát sinh FeatureGrant.
4. **Mã hóa license nhị phân** là mục tiêu bảo mật nêu tại §6, không phải xác nhận DCAD-LICENSE-1 V4 hiện đã mã hóa. Chuyển từ text-signed V4 sang binary encrypted format phải có schema mới, encoder/decoder, tương thích và kiểm thử sửa byte/tamper; không đổi ngầm hoặc giả tạo bảo mật.
5. **Quản trị bảo mật:** account phát triển admin/admin, chuỗi Root/Issuer chưa có, hoặc key phát hành chưa được ủy quyền là các điều kiện khiến RELEASE_APPROVED=NO dù source/build có thể PASS.
6. **Ưu tiên triển khai sau cập nhật quy ước:** (a) chốt schema/host token/trust anchor; (b) thống nhất hai ZIP và Feature UID; (c) tích hợp Gate vào một add-in C22 thật; (d) build + CAD runtime + negative tests; (e) cấp key production sau khi Root/Issuer, quản trị khóa và chứng cứ QA đều PASS. Port C25/G27 sau khi baseline đã được nghiệm thu theo phạm vi.
7. **Thẩm quyền:** §13–14 diễn giải chi tiết chuỗi triển khai, **không bãi bỏ** §1–12 hoặc STD-01–18. Khi dữ liệu V25 chưa tuân thủ yêu cầu mới, ghi FAIL/NotVerified và sửa phần mềm có version +1 ở một yêu cầu triển khai riêng; **không sửa ngầm schema thông qua tài liệu**.
