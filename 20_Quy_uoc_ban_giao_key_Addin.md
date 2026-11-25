# DCAD-STD-20 — Quy ước bàn giao Key và tích hợp bản quyền Add-in DCAD

**Mã quy ước:** 20  
**Phiên bản:** 1.0  
**Ngày:** 25/11/2026  
**Trạng thái:** Quy ước được phê duyệt ngày 25/11/2026; bắt buộc khi bàn giao bộ tích hợp bản quyền và tích hợp Add-in trong phạm vi hợp đồng đã xác nhận.  
**Nguồn chuẩn:** TidiDcad-System V49 (Gate V6), DJoin V38 (tham chiếu tích hợp native), ToolCatalog.db.  
**Phạm vi đã khảo sát:** Add-in native C++ x64, AutoCAD 2022; không mặc định áp dụng cho .NET, AutoCAD 2025 hoặc GstarCAD 2027 khi chưa có hợp đồng và nghiệm thu tương ứng.  
**Quy ước liên quan:** STD-01, 03, 15, 16, 17, 18; STD-18 V1.2 áp dụng bắt buộc khi lập trình, rà soát và kiểm thử.

## 1. Mục tiêu và nguyên tắc

1. TidiDcad là nơi quản lý Tool, Feature, danh tính phát hành SEC06 và cấp License. Bộ tích hợp bàn giao chỉ phục vụ nhúng bảo mật vào mã nguồn Add-in; không phải bộ cài, License khách hàng hay gói khóa riêng.
2. Mỗi Tool có một ZIP riêng, đúng sáu file. Mã tích hợp Gate dùng chung; dữ liệu định danh Tool và Feature lấy từ đăng ký thực tế. Không tạo khóa RSA riêng cho mỗi Tool.
3. Người lập trình có thể build Add-in bằng nguồn Add-in, bộ Integration và SDK/compiler đúng target, không cần cài TidiDcad, không cần đăng nhập SEC06 hoặc có Gate DLL/License khi biên dịch.
4. Khi chạy, Add-in phải xác minh Gate được ký bởi danh tính đã duyệt, kiểm tra ABI, machine policy, License và Feature trước nghiệp vụ được bảo vệ. Chỉ ALLOW mới được thực hiện. Không dùng fallback không ký, bỏ qua Gate, quyền QA mặc định hoặc cấu hình tắt xác minh.
5. Không đổi thuật toán hình học, UI, settings, XData, tên lệnh, cấu trúc nguồn và target ngoài phạm vi cần thiết. Không thay đổi giao thức Gate, chữ ký, mã máy hoặc License trong công việc tích hợp.
6. Kiểm tra tĩnh không thay thế Build, APPLOAD và Runtime. Mỗi kết luận phải có chứng cứ trên nguồn cuối cùng theo STD-18.

## 2. Cấu trúc bàn giao bắt buộc

```text
<ToolName>-Integration.zip
├── ToolInfo.json
├── GateReleaseTrust.h
├── DcadLicenseGate.h
├── DcadGateClient.h
├── DcadGateClient.cpp
└── Integration.txt
```

Tên ZIP sử dụng tên Tool đã đăng ký, ví dụ `Djoin-Integration.zip`. Sáu file nằm ngay tầng gốc, không có thư mục bọc hoặc file thứ bảy. Không chấp nhận tên file trùng, đường dẫn thoát thư mục, mục rỗng hoặc placeholder trong bộ xuất chính thức. JSON và tài liệu văn bản dùng UTF-8; Header công khai và Header API được xuất nguyên bản, không đổi nội dung hoặc encoding tùy ý.

| File | Nguồn | Phạm vi |
|---|---|---|
| ToolInfo.json | ToolCatalog.db + hợp đồng Gate và thông tin target được duyệt | Riêng Tool |
| GateReleaseTrust.h | Public Header từ SEC06 đã phê duyệt | Chung cùng danh tính phát hành |
| DcadLicenseGate.h | Header API chuẩn từ Gate tương ứng | Chung |
| DcadGateClient.h | Chuẩn hóa từ LicenseGate.h và GateReleaseVerifier.h của DJoin V38 | Chung theo hợp đồng |
| DcadGateClient.cpp | Chuẩn hóa từ LicenseGate.cpp và GateReleaseVerifier.cpp của DJoin V38 | Chung theo hợp đồng |
| Integration.txt | Tài liệu kỹ thuật theo mục 8, điền thông tin Tool/target thật | Chung + riêng Tool |

**Cấm đóng gói:** `DcadLicenseGate.dll`, `DcadLicenseGate.release`, `Dcad.lic`, private key Licensecc, PFX, `.dbackup`, mật khẩu, PIN file nội bộ, database quản trị, thành phẩm ARX và file dữ liệu khách hàng. DLL và manifest được phát hành riêng bởi hệ thống trung tâm để dùng ở runtime. Không chép toàn bộ thư mục TidiDcad làm Integration.

## 3. ToolInfo.json — hợp đồng dữ liệu định danh

### 3.1. Nguồn dữ liệu và kiểm tra

Database hiện tại: `D:\000\TidiDcad\DData\ToolCatalog\ToolCatalog.db`.

| Trường JSON | Nguồn/kiểu | Ràng buộc |
|---|---|---|
| IntegrationVersion | chuỗi phiên bản đặc tả | `1.0` |
| Tool.ToolId | tools.tool_code, string | 001–999, giữ số 0 đầu |
| Tool.ToolName | tools.app_name, string | đúng tên đăng ký |
| Tool.Description | tools.note, string | giữ nội dung, có thể rỗng |
| Tool.Features[].FeatureId | features.feature_id, string | XXX-YY, XXX bằng ToolId, YY 01–99 |
| Tool.Features[].FeatureName | features.feature_name, string | đúng tên đã đăng ký |
| Security.GateVersion | hợp đồng V49 | 6 |
| Security.GateApiVersion | DcadLicenseGate.h | 2 |
| Security.MachinePolicyVersion | DcadLicenseGate.h | 1 |
| Security.ReleaseFormat | SEC06 | DCAD-GATE-RELEASE-V1 |
| Security.ReleaseKeyId | Header đã duyệt | 64 hex thường, không placeholder |
| Security.ReleaseAlgorithm | SEC06 | RSA-PSS |
| Security.HashAlgorithm | SEC06 | SHA-256 |
| Security.PssSaltBytes | SEC06 | 32 |
| Security.AllowedStatus | Gate API | 0 |
| Security.LicenseFormats | Gate API | DCAD-LIC-V2, DCAD-LIC-V3 |
| Build.Host | target xác nhận từ nguồn Add-in/hợp đồng | AutoCAD-2022-x64 trong phạm vi V1.0 |
| Build.Language | target | C++ |
| Build.Standard | target | C++17 theo mẫu |
| Build.Architecture | target | x64 |
| Build.Configuration | target | Release |
| Build.OutputFile | tên thành phẩm theo project được duyệt | ví dụ DJoin.arx |
| Runtime.GateFile | hợp đồng | DcadLicenseGate.dll |
| Runtime.ReleaseFile | hợp đồng | DcadLicenseGate.release |
| Runtime.LicenseFile | hợp đồng | Dcad.lic |
| Runtime.GateFolder | cấu trúc DCAD | DProgram |
| Runtime.LicenseFolder | cấu trúc DCAD | DUser |

Chỉ xuất Tool và Feature đang hoạt động (`deleted_utc IS NULL` ở cả hai bảng), có quan hệ đúng qua `features.tool_id = tools.id`; không có ID trùng, không tự cấp mã, không tái sử dụng mã đã retire. Feature phải có ít nhất một mục hợp lệ. Không sử dụng ID nội bộ số nguyên thay thế chuỗi mã quyền. Target và OutputFile không có sẵn trong schema database hiện tại; phải xác nhận từ project/thiết lập đã duyệt, không tự suy đoán từ tên Tool. Tên file thành phẩm có thể khác cách viết hoa thường của tên đăng ký.

### 3.2. Mẫu DJoin (minh họa schema, không phải gói xuất thực tế)

```json
{
  "IntegrationVersion": "1.0",
  "Tool": {
    "ToolId": "001",
    "ToolName": "Djoin",
    "Description": "Nối đường nâng cao,Tự chuyển đổi nét đặc biệt hoặc 3D thành nét cơ bản",
    "Features": [
      {"FeatureId": "001-01", "FeatureName": "Toàn bộ"}
    ]
  },
  "Security": {
    "GateVersion": 6,
    "GateApiVersion": 2,
    "MachinePolicyVersion": 1,
    "ReleaseFormat": "DCAD-GATE-RELEASE-V1",
    "ReleaseKeyId": "<KEYID_THUC_TE>",
    "ReleaseAlgorithm": "RSA-PSS",
    "HashAlgorithm": "SHA-256",
    "PssSaltBytes": 32,
    "AllowedStatus": 0,
    "LicenseFormats": ["DCAD-LIC-V2", "DCAD-LIC-V3"]
  },
  "Build": {
    "Host": "AutoCAD-2022-x64",
    "Language": "C++",
    "Standard": "C++17",
    "Architecture": "x64",
    "Configuration": "Release",
    "OutputFile": "DJoin.arx"
  },
  "Runtime": {
    "GateFile": "DcadLicenseGate.dll",
    "ReleaseFile": "DcadLicenseGate.release",
    "LicenseFile": "Dcad.lic",
    "GateFolder": "DProgram",
    "LicenseFolder": "DUser"
  }
}
```

Djoin 001 và Feature 001-01 (Toàn bộ) tồn tại trong database đã khảo sát, chưa bị xóa; thời điểm đăng ký UTC lần lượt 2026-11-22 13:20:08 và 13:20:34. Tuy nhiên DJoin V38 vẫn ghi `001-01` là QA placeholder. Phải xác nhận ánh xạ chính thức trước phát hành; việc trùng mã không chứng minh ARX cũ đã được nghiệm thu production. `<KEYID_THUC_TE>` chỉ được dùng trong ví dụ tài liệu, không được xuất ra Integration chính thức.

### 3.3. Sử dụng khi build

JSON là dữ liệu build-time. Ánh xạ từng Feature tới command/action đã xác nhận, nhúng chuỗi hằng trong binary; có thể sinh header nội bộ theo build nhưng không thêm file thứ bảy vào ZIP. Không đọc JSON bên ngoài để đổi Feature, target hoặc khóa tin cậy trong runtime. Với Tool nhiều Feature, không lấy quyền của Feature này thay quyền Feature khác. Không cấp quyền dựa trên ToolId đơn thuần.

## 4. GateReleaseTrust.h — khóa công khai SEC06

Nguồn: `D:\000\TidiDcad\DData\ReleaseTrust\GateReleaseTrust.h`. V49 tạo header với namespace `djoin`; giữ nguyên namespace và ký hiệu để tương thích hợp đồng hiện hành.

| Ký hiệu | Kiểu | Ràng buộc V49 |
|---|---|---|
| kGateReleaseModulus | inline constexpr unsigned char[] | 384–512 byte; khóa mới RSA 3072 bit (384 byte) |
| kGateReleaseExponent | inline constexpr unsigned char[] | 1–8 byte |
| kGateReleaseKeyId | inline constexpr char[] | 64 hex thường |

`KeyId = SHA256(ModulusBytes || ExponentBytes)`; nối byte theo đúng thứ tự, không hash văn bản Hex. RSA-PSS dùng SHA-256, salt 32 byte. Header V38 đính kèm chỉ có `#error` chưa enrollment, không dùng làm khóa thật.

**Tại bên xuất:** dùng cơ chế xác minh trust hiện hành của SEC06; Header hợp lệ, KeyId tính lại khớp Header và `IssuerPrivate\GateReleaseSigner.pin`; kiểm tra không có `identity.pending` hoặc `publish.pending`; không tự sinh, đổi hoặc thay pin. Chỉ xuất dữ liệu công khai. Việc xuất public material không yêu cầu truyền khóa riêng sang máy build.

**Tại bên nhận:** đối chiếu KeyId từ nguồn phát hành tin cậy đã duyệt (ví dụ xác nhận KeyId riêng với người quản trị). Kiểm tra JSON–Header–fingerprint tự khớp là cần thiết nhưng không đủ để xác thực nguồn gốc, vì cả gói có thể bị thay thế. Không suy ra public key tin cậy từ DLL hoặc manifest bất kỳ; không tự cập nhật khóa từ nguồn khác. KeyId thay đổi phải được duyệt và tích hợp/build lại các Add-in liên quan; thay mật khẩu nhưng giữ KeyId không cần đổi public header. Phân biệt SEC06 ký Gate với khóa Licensecc cấp License, không trộn hai loại khóa.

## 5. DcadLicenseGate.h — hợp đồng API

Xuất byte-for-byte từ `TidiDcad-System-V49/Source/DcadLicenseGate/Source/DcadLicenseGate.h`. Windows x64, C ABI, `__cdecl`, Unicode path; không yêu cầu import library khi nạp động. Không tự viết lại enum, tên export hoặc ABI.

| API | Ý nghĩa |
|---|---|
| DcadGate_GetApiVersion(void) | int, bắt buộc 2 |
| DcadGate_GetMachinePolicyVersion(void) | int, bắt buộc 1 |
| DcadGate_CheckLicenseW | kiểm tra quyền theo đường dẫn License và Feature |
| DcadGate_CheckLicenseDetailsW | cùng quyền, thêm thông tin khoảng UTC đã xác minh; chỉ dùng nếu nghiệp vụ cần |

Chữ ký tham khảo từ Header (không thay thế Header gốc):

```cpp
int __cdecl DcadGate_CheckLicenseW(
    const wchar_t* licensePath, const char* feature,
    wchar_t* diagnostic, uint32_t diagnosticChars) noexcept;
int __cdecl DcadGate_CheckLicenseDetailsW(
    const wchar_t* licensePath, const char* feature,
    wchar_t* diagnostic, uint32_t diagnosticChars,
    wchar_t* period, uint32_t periodChars) noexcept;
```

Đường dẫn License tuyệt đối; feature cố định; dung lượng buffer tính bằng ký tự (V38 dùng 512). Không dùng thông báo chuỗi làm căn cứ cấp quyền.

| Mã | Hằng số |
|---:|---|
| 0 | DCAD_GATE_ALLOW |
| 1 | DCAD_GATE_FILE_MISSING |
| 2 | DCAD_GATE_FORMAT_ERROR |
| 3 | DCAD_GATE_SIGNATURE_INVALID |
| 4 | DCAD_GATE_LICENSE_DENIED |
| 5 | DCAD_GATE_TIME_DENIED |
| 6 | DCAD_GATE_BAD_ARGUMENT |
| 7 | DCAD_GATE_SDK_MISMATCH |
| 8 | DCAD_GATE_INTERNAL_ERROR |

Chỉ 0 cho phép; mã không biết, thiếu export, exception hay lỗi I/O đều DENY. Gate xác minh License V2/V3, mã máy và quyền/thời gian theo hợp đồng; Client không viết lại logic Licensecc.

## 6. DcadGateClient.h — API dùng chung

Chuẩn hóa từ `src/LicenseGate.h`, `src/GateReleaseVerifier.h` của DJoin V38. API mục tiêu đã thống nhất (chưa triển khai trong V49):

```cpp
namespace dcad {
    bool CheckLicense(HMODULE cadModule,
                      const std::wstring& duser,
                      const char* featureId,
                      std::wstring& diagnostic) noexcept;
}
```

Header phải tự đủ khai báo kiểu cần thiết. `cadModule` là handle của chính Add-in, không phải executable CAD. `duser` là thư mục DUser đã xác định bằng cơ chế path của host; tuyệt đối và hợp lệ. `featureId` là hằng từ ánh xạ đã duyệt. `diagnostic` cho người dùng/log, không cấp quyền. `true` chỉ khi đầy đủ các điều kiện xác minh và Gate trả 0. `false` trong mọi lỗi. Không đưa API bỏ qua xác minh hoặc tham số cho phép tắt kiểm tra.

## 7. DcadGateClient.cpp — yêu cầu bảo mật

### 7.1. Nguồn và phạm vi chuẩn hóa

Tham chiếu `src/LicenseGate.cpp`, `src/GateReleaseVerifier.cpp` của DJoin V38. Thay hàm riêng DJoin bằng API tổng quát mục 6; bỏ phụ thuộc `DJoinFeature.h` trong client dùng chung, giữ ID cố định ở caller. Giữ thuật toán và hợp đồng. V38 có macro `DCAD_SEC06_C22`: khi chuẩn hóa, không để nhánh build nào có thể nạp Gate mà không qua xác minh SEC06. Nếu tiếp tục dùng macro trong giai đoạn chuyển đổi, thiếu macro phải làm build FAIL. Không coi bản hợp nhất mới đã được nghiệm thu chỉ vì nguồn mẫu có QA.

### 7.2. Luồng bắt buộc

1. Kiểm tra tham số, ID XXX-YY (001-01 đến 999-99), path, handle. Xác định Gate từ thư mục chứa Add-in được triển khai trong DProgram; không tìm DLL theo PATH hoặc thư mục làm việc.
2. Từ chối DLL/manifest thiếu, directory hoặc reparse point theo kiểm tra mẫu. Mở và giữ handle DLL chỉ đọc, `FILE_SHARE_READ` (không chia sẻ ghi/xóa) trong thời gian xác minh và nạp; duy trì vòng đời handle theo nguồn mẫu.
3. SHA-256 trên nội dung DLL từ handle đã khóa; kiểm tra giới hạn/đọc đầy đủ.
4. Đọc manifest và xác minh chính xác danh tính, Host, ABI, Policy, KeyId, hash và chữ ký RSA-PSS từ public key đã nhúng. Không nạp DLL nếu bất kỳ kiểm tra nào lỗi.
5. Nạp DLL bằng đường dẫn tuyệt đối và cơ chế tìm phụ thuộc hạn chế như nguồn mẫu. Kiểm tra module thực tế khớp file đã xác minh bằng Volume Serial và File Index; bảo toàn quản lý vòng đời DLL/handle.
6. Resolve export đúng ABI; xác minh API=2, Machine Policy=1; gọi `DcadGate_CheckLicenseW` với đường dẫn Dcad.lic và đúng Feature. Chỉ ALLOW. Xử lý lỗi/exception theo fail-closed.
7. Kiểm tra lại mỗi lần thực hiện chức năng được bảo vệ; không cache quyền qua những lần gọi lệnh khi License có thể thay đổi.

### 7.3. Hợp đồng manifest V1

```text
DCAD-GATE-RELEASE-V1
File=DcadLicenseGate.dll
Sha256=<64 hex thuong>
Host=AutoCAD-2022-x64
Api=2
Policy=1
ReleaseId=<16 hex dau cua Sha256>
KeyId=<64 hex thuong>
Signature=<Base64>
```

Trình tự và giá trị trường chính xác, LF và LF cuối, không CR/NUL, không thêm trường. Chữ ký trên toàn bộ byte từ dòng đầu đến hết LF của dòng KeyId (không gồm dòng Signature). Base64 canonical, giải mã có kích thước bằng modulus. Mọi sai khác DENY. Host hiện ghi cứng ở cả signer V49 và verifier V38; không đổi Host theo tham số tùy ý để tuyên bố hỗ trợ CAD khác.

| Giới hạn từ mẫu V38/V49 | Giá trị |
|---|---|
| Gate DLL | 1.024 byte đến 64 MiB |
| Manifest | 400 đến 8.192 byte |
| Block hash DLL | 65.536 byte |
| Buffer đường dẫn V38 | 32.768 wchar_t |
| Buffer thông báo V38 | 512 wchar_t |
| Digest SHA-256 | 32 byte |
| Base64 signature V38 | tối đa 1.024 ký tự, canonical |
| RSA public key | tối thiểu 3072 bit ở verifier; áp dụng giới hạn V49 mục 4 khi xuất |
| PSS salt | 32 byte |

Phụ thuộc client native: Win32/Kernel32, `Bcrypt.lib`, `Crypt32.lib`, C++17. Không link Licensecc SDK hoặc `DcadLicenseGate.lib`, không đọc SQLite, không mạng, không cần private key. Không tuyên bố cơ chế này chống mọi hình thức can thiệp binary; phạm vi nghiệm thu là hợp đồng và các ca bảo mật xác định.

## 8. Integration.txt — nội dung bắt buộc INT-01 đến INT-12

Mỗi ZIP phải có tài liệu độc lập đủ để người lập trình thực hiện toàn bộ tích hợp. Các mục dưới đây là yêu cầu nội dung, không phải một README chung sơ sài. Phải điền dữ liệu Tool/target/KeyId thật; phần áp dụng cho project cụ thể cần ghi nguồn baseline. Các ví dụ DJoin không được áp đặt cấu trúc hoặc toolset cho mọi Tool.

### INT-01. Đầu vào, phạm vi và hợp đồng

Ghi ToolId, tên Tool, mô tả, toàn bộ Feature đang hoạt động, target, output, phiên bản Integration, Gate ABI, Policy, KeyId; danh sách sáu file; nguồn phát hành và cách đối chiếu KeyId độc lập. Xác định baseline Add-in, mục tiêu chỉ tích hợp bảo mật và những phần phải giữ nguyên. Nêu rõ target nào chưa hỗ trợ. Đọc STD-18 trước sửa.

### INT-02. Bố trí file và chuyển đổi mã cũ

Giữ cấu trúc project hiện hành; ví dụ `src\Security\` chứa bốn file C++/header, `Buildtime\ToolInfo.json` chứa thông tin build. Không thêm file vào thư mục runtime để thay đổi quyền. Nếu project có `LicenseGate.cpp/.h`, `GateReleaseVerifier.cpp/.h` hoặc `DJoinFeature.h` cũ, khảo sát caller, project và QA; chuyển sang client chung, tránh compile hai bộ verifier song song; chỉ bỏ phần cũ sau khi xác minh không còn tham chiếu. Các helper build phát sinh nằm trong project Add-in, không tăng sáu file của ZIP.

### INT-03. Feature và điểm chặn nghiệp vụ

Lập bảng Feature ID → command/action được bảo vệ. Hằng chuỗi nhúng lúc build (ví dụ `inline constexpr char kFeatureId[] = "001-01";` sau phê duyệt). DJoin V38: `src/DJoin.cpp`, `command()` dòng 469–477, kiểm tra ngay trước `database()`, load settings và selection. API mới gọi `dcad::CheckLicense` với module handle, DUser, Feature, diagnostic; nếu false thì thông báo và return. Đặt kiểm tra trước selection, mở chức năng được bảo vệ, Transaction hoặc thay đổi dữ liệu/hình học; không chỉ kiểm tra ở APPLOAD. Rà các entry point khác có thể gọi trực tiếp nghiệp vụ; không để đường thực thi bỏ qua quyền. Xử lý lỗi tìm DUser cũng phải dừng an toàn.

### INT-04. Runtime và cấu trúc triển khai

```text
<DCAD-root>\DProgram\<Tool>.arx
<DCAD-root>\DProgram\DcadLicenseGate.dll
<DCAD-root>\DProgram\DcadLicenseGate.release
<DCAD-root>\DUser\Dcad.lic
```

DJoin V38 lấy đường dẫn module qua `GetModuleFileNameW`; Gate cùng thư mục module. `SettingsNative.cpp::userFolderFromSupportPath()` đọc Support File Search Path (ACADPREFIX), chọn thư mục có thành phần cuối DUser, bỏ dấu ngoặc kép và dấu phân cách cuối, kiểm tra tồn tại, canonical path để phát hiện nhiều DUser khác nhau; thiếu/xung đột/không tuyệt đối → DENY. Với project khác, giữ resolver đúng chuẩn hiện hành nếu tương đương; không tự tạo hoặc tìm License ở nơi khác. Gate + manifest dùng chung phải được triển khai đồng bộ, từ TidiDcad; không đưa vào mỗi ZIP tích hợp. License hợp lệ do hệ thống cấp cho máy và Feature tương ứng.

### INT-05. Cấu hình project (.vcxproj)

Tham chiếu `DJoin.vcxproj` V38: Release|x64, DynamicLibrary, C++17 (`stdcpp17`), Unicode, `/MD` (MultiThreadedDLL), `/EHsc` (Sync), `/utf-8`, Windows SDK 10.0, v142 ưu tiên, MFC Dynamic, WholeProgramOptimization=true, WarningLevel=Level3, Optimization=MaxSpeed, PCH=NotUsing, ConformanceMode=true, SubSystem=Windows, TargetMachine=MachineX64, EnableCOMDATFolding=true, GenerateDebugInformation=true, TargetExt=.arx. Đây là thông số mẫu, không tự đổi thiết lập không liên quan của Add-in khác.

Bổ sung item và include dir trong đúng configuration:

```xml
<ItemGroup>
  <ClCompile Include="src\Security\DcadGateClient.cpp" />
  <ClInclude Include="src\Security\DcadGateClient.h" />
  <ClInclude Include="src\Security\DcadLicenseGate.h" />
  <ClInclude Include="src\Security\GateReleaseTrust.h" />
</ItemGroup>
```

Append `$(ProjectDir)src\Security` vào AdditionalIncludeDirectories, giữ `%(AdditionalIncludeDirectories)` và danh sách cũ. Append `Bcrypt.lib;Crypt32.lib` vào AdditionalDependencies, giữ `%(AdditionalDependencies)` và danh sách cũ. Không thêm import lib Gate. Giữ `.def`, RC/icon, exports CAD, reference và flags nghiệp vụ. Bảo đảm SEC06 luôn bật ở mọi target phát hành; không dựa vào Debug/Release để bỏ xác minh. DJoin V38 có `DCAD_SEC06_C22`; client chuẩn không có đường build bỏ qua xác minh.

Thư viện V38 để tham khảo: rxapi.lib, acad.lib, acdb24.lib, accore.lib, acge24.lib, adui24.lib, acui24.lib, acgiapi.lib, Shell32.lib, Comctl32.lib, Bcrypt.lib, Crypt32.lib. Chỉ hai lib cuối là phần bổ sung bảo mật. Không sao chép bộ ObjectARX lib này sang target khác.

### INT-06. Preflight Tool và Public Key

Kiểm tra đúng sáu file/phiên bản/schema; ID, ánh xạ command/Feature; public header thực tế không chứa #error, modulus/exponent đúng, KeyId đủ 64 hex và tính lại khớp; đối chiếu ToolInfo và KeyId được duyệt độc lập như mục 4. Có thể chuyển logic `Buildtime/Check-ReleaseTrust.ps1` V38 vào project, bổ sung giới hạn và đối chiếu danh tính đã duyệt mà mẫu cũ chưa có. Không tự nhập key khác, không đọc key riêng, không yêu cầu Gate DLL/License để compile. Preflight sai → lỗi nonzero, dừng trước publish. Kiểm tra không cho build trực tiếp target bỏ qua preflight cần thiết (project/Build_Release có thể được gọi độc lập).

### INT-07. Build_All.cmd — luồng bắt buộc

1. Xác định root từ script, target, đường dẫn SDK và output; tạo log và staging đúng quy ước. Không thao tác vào dữ liệu vận hành.
2. Kiểm tra nguồn và project, sáu file tích hợp; kiểm tra ID/Feature, khóa tin cậy đã duyệt; kiểm tra toolchain/SDK.
3. Gọi Build_Release.cmd cho từng target đã duyệt, truyền thông số chính xác, nhận exit code. Không lấy output cũ để bù khi lỗi; xác minh output tạo mới, không rỗng.
4. Publish đúng tên thành phẩm theo quy ước, tạo thư mục nếu thiếu; kiểm tra nội dung file đích bằng SHA-256 hoặc so sánh byte. Không phá hủy bản đã phát hành chỉ vì build mới thất bại; dùng staging riêng, chỉ công bố kết quả đủ điều kiện.
5. Log chính ghi baseline, target, ToolId, Feature, KeyId, SDK root, toolset, bước build, output, kết quả và lỗi; log target tách riêng. Exit 0 chỉ khi build/publish đúng phạm vi; lỗi nonzero. Không coi exit 0 là APPLOAD/Runtime PASS.

**Khác mẫu V38:** bỏ phụ thuộc đồng bộ từ máy TidiDcad trong `Sync-ReleaseTrust.ps1` của Build_All khi bộ Integration đã chứa Public Key thật; thay bằng preflight khóa bàn giao. V38 còn xóa output staging trước khi build và có trạng thái SETUP REQUIRED (exit 2); quy ước mới yêu cầu không mất bản phát hành cũ, thiếu khóa bàn giao phải FAIL rõ nguyên nhân. Không tạo PFX/khóa, không đăng nhập, không yêu cầu Dcad.lic, không build/copy DLL hoặc manifest Gate. Không tự chạy ứng dụng hoặc mở Explorer. Có thể dùng `/nopause` cho build tự động.

### INT-08. Build_Release.cmd và SDK

Ví dụ AutoCAD 2022: reference `D:\000\Reference\Autocad 2022`; ObjectARX 2022 x64; PowerShell 5.1 cho preflight mẫu; Visual Studio/MSBuild có MSVC x64, v142 ưu tiên. V38 hỗ trợ v143/v145 thay thế hoặc `DCAD_MSVC_TOOLSET`, nhưng phải nghiệm thu CAD trên toolset thực tế. Có thể dùng `DCAD_REFERENCE_ROOT` khi được phê duyệt; không dùng file SDK trộn từ nhiều root.

`Buildtime/ResolveC22SDK.ps1` V38 kiểm tra arx.props/dbx.props, header và library cần thiết; script là nguồn danh sách chính xác cho baseline, không áp đặt cho target khác. Tìm một root SDK đầy đủ, file không rỗng; nếu nhiều bộ đầy đủ nhưng nội dung khác thì dừng; nếu giống nhau từng byte thì có thể chọn nhất quán. Không mở hai luồng ghi đồng thời vào cùng log (V38 SDK resolver tự ghi log).

Lệnh mẫu V38:

```cmd
"%MSBUILD%" "%BASE%DJoin.vcxproj" /t:Rebuild /p:Configuration=Release /p:Platform=x64 "/p:PlatformToolset=%TOOLSET%" "/p:ARX2022_SDK=%SDKROOT%" /m:1 /nologo "/flp:LogFile=%LOG%;Verbosity=normal;Append"
```

Output V38: `bin\DJoin.arx`; intermediate `Buildtime\obj\`; staging root `Dprogram\AutoCAD2022\DJoin.arx`; log chính `log\Build_All.log`, log target `log\Build_AutoCAD2022.log` trong thư mục target. Giữ bố trí theo quy ước hiện hành khi thực hiện. Không liên kết nhầm SDK/host. Build_Release cũng phải bảo đảm có public key hợp lệ và xác minh bắt buộc, không dựa vào Build_All một cách có thể bỏ qua.

### INT-09. Cập nhật README, BuildInfo và dữ liệu QA

README của Add-in phải có: tên/phiên bản/baseline; ToolId và Feature → command/action; CAD target; Security Contract (Gate version, ABI, Policy, License format); KeyId được nhúng và nguồn duyệt; các file tích hợp; điểm chặn; SDK/toolset/Windows SDK; cách chạy Build_All; output/log; cài đặt DProgram/DUser; hành vi ALLOW/DENY; cách xử lý thiếu file hoặc khóa không khớp; Known Issues và trạng thái Build/APPLOAD/Runtime riêng. Cập nhật BuildInfo, DataManifest, QA và lịch sử phiên bản theo STD-16/17 khi phù hợp. Không thêm README thứ hai vào ZIP Integration.

### INT-10. Ma trận kiểm thử bắt buộc

Xem mục 10. Mỗi ca có đầu vào/điều kiện, Expected, Actual, PASS/FAIL/NotVerified hoặc N/A có lý do, host và bằng chứng. DJoin V38 có `Buildtime/QA/VerifySEC06.cpp` và `MockGate.cpp` làm mẫu kiểm thử verifier; phải kiểm thử lại client chuẩn trên nguồn cuối cùng. Mock Gate chỉ trong môi trường QA cô lập, không phát hành hoặc tích hợp vào runtime thực tế.

### INT-11. Phát hành và quản lý thay đổi

Một Tool xuất một ZIP; một build chỉ dùng danh tính đã được phê duyệt. Khi Gate DLL được cập nhật với cùng KeyId và hợp đồng tương thích, bộ runtime mới vẫn cần nghiệm thu tương thích, không mặc định phải build lại mọi Add-in. Khi public key, ABI, Policy, manifest format hoặc target contract đổi, phải đánh giá ảnh hưởng, xuất bộ phù hợp, build và nghiệm thu lại các Tool liên quan; không tự đổi hợp đồng. Không tự đổi ID đã cấp. Không ghi đè dữ liệu người dùng. Ghi chứng cứ trên bản nguồn cuối cùng, không thay đổi ngoài phạm vi.

### INT-12. Chốt bàn giao và phân loại trạng thái

Build PASS ≠ APPLOAD PASS ≠ Runtime PASS. Chỉ phát hành bản nghiệm thu khi toàn bộ cổng áp dụng đã đạt theo STD-18; thiếu host phải ghi NotVerified, không giả lập thành PASS. Nếu người quản trị cho phép bàn giao bản nguồn để test, ghi rõ chưa nghiệm thu. Không ghi nhận production/OK vào hệ thống quản lý khi chưa được xác nhận. Integration.txt phải giải thích cách nghiệm thu cả Tool có một Feature và nhiều Feature.

## 9. Quy trình xuất ZIP tại TidiDcad (đặc tả chức năng, chưa triển khai)

1. Chọn Tool đã đăng ký → Xuất bộ tích hợp bản quyền. Đọc snapshot Tool/Feature đang hoạt động, xác nhận target và ánh xạ theo dữ liệu được duyệt. Hiển thị tên, mã, Feature, ABI, Policy, KeyId, sáu file.
2. Đọc public trust đã duyệt, kiểm tra trạng thái giao dịch, tính nhất quán; không cần xuất private key. Lấy API Header đúng phiên bản và client đã chuẩn hóa/kiểm thử, không dùng code do AI sinh tùy ý từng lần xuất.
3. Tạo ToolInfo và Integration.txt với thông tin thật, chép đúng các header/client; kiểm tra dữ liệu không thiếu, không placeholder, không có bí mật. Kiểm tra ZIP đúng sáu file, đọc lại nội dung, đối chiếu hash file nguồn/đích. Nếu đích đã tồn tại, yêu cầu xác nhận trước thay thế; không để ZIP dang dở khi lỗi.
4. Báo rõ xuất thành công hay lỗi cụ thể (Tool đã retire, Feature thiếu, KeyId chưa duyệt, phiên bản client không phù hợp, v.v.). Xuất ZIP PASS chỉ là nghiệm thu bộ dữ liệu tích hợp, không phải nghiệm thu Add-in đích.

Không tạo thêm schema database, phần mềm quản lý khóa hay cơ chế ký mới chỉ vì xuất ZIP. Nếu cần mở rộng dữ liệu CAD target, phải duyệt riêng phương án; V1.0 có thể lấy từ thông tin target đã xác nhận khi xuất.

## 10. Ma trận QA bắt buộc theo phạm vi

| Mã | Điều kiện | Expected |
|---|---|---|
| EXP01 | Tool có một/nhiều Feature đang hoạt động | đủ sáu file, đúng ID và dữ liệu |
| EXP02 | Tool/Feature retire, thiếu hoặc xung đột ID | không xuất bộ sai |
| EXP03 | Header chưa enrollment, pin sai, pending transaction | từ chối xuất |
| EXP04 | Gói ZIP lỗi, ghi dở, trùng đích | báo lỗi, không làm mất gói hợp lệ |
| EXP05 | Kiểm tra thành phần | không có khóa riêng, DLL, manifest, License hoặc database |
| QA01 | Build nguồn + Integration, không cài TidiDcad | PASS |
| QA02 | Build thiếu Gate DLL/manifest/License runtime | PASS |
| QA03 | Thiếu public header hoặc #error | Build FAIL |
| QA04 | Public key hoặc KeyId bị sửa | Build FAIL |
| QA05 | JSON/Header khớp nội bộ nhưng khác KeyId đã duyệt | Build FAIL |
| QA06 | Feature sai cú pháp, sai Tool, không đúng ánh xạ | Build FAIL |
| QA07 | Gọi build trực tiếp target, thiếu xác minh bắt buộc | Build FAIL |
| QA08 | SDK thiếu, sai target hoặc trộn root | Build FAIL |
| QA09 | Build/publish lỗi giữa chừng, output cũ hoặc đích sai | FAIL, không phát hành sai, bản cũ an toàn |
| QA10 | APPLOAD đúng CAD target | PASS |
| QA11 | License V2/V3 hợp lệ, đúng máy/Feature/thời hạn | ALLOW theo hợp đồng |
| QA12 | Thiếu License, sai chữ ký hoặc format | DENY |
| QA13 | License sai máy | DENY |
| QA14 | Hết hạn hoặc chưa đến hạn | DENY |
| QA15 | Không có Feature, quyền Feature khác | DENY |
| QA16 | Gate DLL bị sửa | DENY trước khi nạp DLL |
| QA17 | Manifest sai/thiếu, sai chữ ký, sai Host/hash/KeyId | DENY trước khi nạp DLL |
| QA18 | Gate thiếu, reparse point, lỗi đọc/khóa file | DENY |
| QA19 | Gate sai ABI/Policy hoặc thiếu export | DENY |
| QA20 | Module nạp không khớp file xác minh | DENY |
| QA21 | DUser thiếu/sai/xung đột; gọi lại sau thay đổi License | DENY đúng điều kiện, kiểm tra lại từng lần |
| QA22 | DENY bất kỳ entry point nào | không thực hiện nghiệp vụ được bảo vệ |
| QA23 | Nhiều Feature cấp quyền độc lập | chỉ Feature có quyền thực hiện |
| QA24 | Nghiệp vụ gốc khi ALLOW, dữ liệu và settings | không hồi quy |
| QA25 | Lỗi I/O/exception/mã kết quả không biết | DENY an toàn |

Các ca trên là **tiêu chí thiết kế**, chưa phải kết quả thực thi. Phiếu nghiệm thu dùng cột `Mã | Input | Expected | Actual | Status | Host | Evidence`. Kiểm thử an toàn trên môi trường cô lập; không sửa khóa hoặc dữ liệu thật chỉ để thử ca lỗi. Review độc lập AI-15, ma trận AI-16, test lại AI-17 và chốt AI-18 trước bàn giao.

## 11. Phân công trách nhiệm

| Việc | TidiDcad/đơn vị phát hành | Người tích hợp Add-in |
|---|---|---|
| Quản lý danh tính, Tool/Feature, phát hành Gate và License | Chịu trách nhiệm | Không tạo khóa |
| Xác minh/xuất public key và bộ Integration | Chịu trách nhiệm | Kiểm tra nguồn tin cậy |
| Xác định ánh xạ Feature → chức năng, CAD target | Phối hợp/phê duyệt | Khảo sát và thực hiện |
| Tích hợp client, project, Build_All, README | Cung cấp chuẩn | Chịu trách nhiệm |
| Kiểm thử build và CAD, hồi quy | Phối hợp cung cấp runtime/license QA | Chịu trách nhiệm và báo cáo |
| Nghiệm thu/phát hành chính thức | Phê duyệt | Bàn giao bằng chứng |

## 12. Căn cứ nguồn và giới hạn hiện tại

Đã kiểm tra trực tiếp ba tệp sau để lập đặc tả (SHA-256 giúp nhận diện nguồn, không phải chữ ký xác thực nguồn):

- `DJoin-V38 (1).zip`: `ae97bd564e9d8ff0f36daf3bc519b70d24ff8680e3b397686a3de4cbdc4b8043`. Tham chiếu Build_All.cmd; Build_Release.cmd; DJoin.vcxproj; Userdata/README.txt, QA-STATUS.txt; src/DJoin.cpp, SettingsNative.cpp, DJoinFeature.h, LicenseGate.*, GateReleaseVerifier.*, GateReleaseTrust.h; Buildtime/Check-ReleaseTrust.ps1, Sync-ReleaseTrust.ps1, ResolveC22SDK.ps1 và QA.
- `TidiDcad-System-V49.zip`: `2c805de957636add22d602778ad553c6ee69b16d7ed9d5890d7867605bb7ed5a`. Tham chiếu Source/ReleaseTrust/PortableSecurity.ps1 (public trust, KeyId, manifest) và Source/DcadLicenseGate/Source/DcadLicenseGate.h (API V6).
- `ToolCatalog.db`: `2100431221130d76bcb82af00e53c3505804aa328b104573bee6dcc8c2c08e5f`. Đã đọc schema tools/features và bản ghi Djoin.

Các phần chuẩn hóa mới: schema ToolInfo.json, API client tổng quát, chức năng xuất ZIP, preflight độc lập TidiDcad, nội dung Integration.txt và ma trận QA. Chưa thực hiện các thay đổi đó trong mã nguồn. Chưa có Public Key V49 thực tế đã duyệt để xuất; không dùng placeholder V38. QA-STATUS V38 ghi MSVCBuild, APPLOAD, Runtime là NotVerified; không dùng nguồn này làm chứng cứ đã nghiệm thu. Việc công bố quy ước này không thay thế nghiệm thu thực tế, không sửa source và không tạo khóa.