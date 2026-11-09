# DCAD-STD-19 — Kiến trúc bản quyền DCAD và đồng bộ DCADManage ↔ DLL

**Mã quy ước:** 19  
**Phiên bản:** 1.0  
**Ngày cập nhật:** 09/11/2026  
**Mức áp dụng:** BẮT BUỘC khi phát triển, quản trị, cấp phép hoặc tích hợp bản quyền DCAD. Chỉ chấm yêu cầu trong phạm vi chức năng đã triển khai; hạng mục chưa có bằng chứng là `NotVerified`.  
**Hợp đồng:** `DCAD-LICENSE-ARCH-001`.  
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

1. **Chốt hợp đồng** Manifest, Registry, Tool Certificate, payload ký, trust anchor, API, Machine ID và schema/migration.
2. **Dmanage Registry:** import/diff/approve danh mục Tool/Feature, cấp/giữ UID, audit.
3. **Integration Export:** sinh mã `.g.cs`, contract JSON, chứng nhận ký, báo cáo; test không chứa private key.
4. **File ID:** cấp ToolGrant/FeatureGrant với thời hạn riêng, mã hóa/ký, reader legacy riêng.
5. **Client:** kiểm tra signatures, MachineId, host, UID, quyền, hết hạn, dữ liệu lỗi; negative tests.
6. **Tích hợp một DLL AutoCAD 2022 thật** và kiểm thử NETLOAD/thao tác; sau đó port C25/G27 theo phạm vi duyệt.
7. **Gia cố** môi trường sản xuất: tài khoản, ký mã, sao lưu, đánh giá chống can thiệp và lộ trình dongle.

Các dự án chưa tích hợp license không tự bị coi là sai STD-19; sau khi một feature được quảng bá là có license thì mọi đường thực thi feature đó phải kiểm tra đúng. Không chấp nhận “có chứng nhận là đã bảo vệ” hoặc “build PASS là license PASS”.

## 10. Ma trận QA DCAD-STD-19

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
