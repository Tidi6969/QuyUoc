# DCAD-STD-20 — Quy ước bàn giao Key và tích hợp bản quyền Add-in DCAD

**Mã quy ước:** 20  
**Phiên bản:** 1.3  
**Ngày cập nhật:** 26/11/2026  
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

**Plan — Tổng thể hướng phát triển thực tế TidiDcad (cập nhật 26/11/2026)**

Plan là một phần của STD-20, được quản lý trực tiếp tại đây; không tạo quy ước, file kế hoạch hoặc dự án riêng. Đây là nơi ghi yêu cầu và tiến độ phát triển chính thức của TidiDcad; README DCAD chỉ là thông tin tổng hợp, không thay thế Plan. Plan không thay đổi hợp đồng tích hợp đang hiệu lực khi chưa có thiết kế và nghiệm thu được duyệt.

**Baseline ban đầu: V55. Baseline hiện tại: V59** — chủ dự án xác nhận build OK và kiểm nghiệm chống sửa Add-in với Dtscr V121 trên AutoCAD 2022 ngày 26/11/2026. Giữ nguyên 01–05 trên V59; 06 đã triển khai và kiểm nghiệm các ca chính; 07 tạm dừng phát triển; 08 đã thống nhất nguyên tắc nhận diện máy SMBIOS 2/3, chưa triển khai.

| STT | Hạng mục | Yêu cầu | Tiến độ |
|---|---|---|---|
| 01 | Chữ ký số License (LIC-04) | Giữ xác minh chữ ký và từ chối License không hợp lệ | V55 — giữ nguyên |
| 02 | Khóa bản quyền theo máy (LIC-02) | Giữ DeviceID và đối chiếu máy tại Gate | V55 — giữ nguyên |
| 03 | Phân quyền Feature (LIC-03) | Giữ định danh, quyền và thời hạn Feature | V55 — giữ nguyên |
| 04 | Chống thay thế Gate (SEC-02) | Giữ xác minh chữ ký/hash Gate DLL theo SEC06 | V55 — giữ nguyên |
| 05 | Ký số bộ bàn giao (SEC-03) | Giữ RSA-PSS Integration sáu file và xác thực khi tiếp nhận theo STD-20 | V55 — giữ nguyên |
| 06 | Chống sửa Add-in (SEC-AT) | Ký số nhúng Authenticode cho thành phẩm; xác minh tính toàn vẹn và danh tính phát hành DCAD; từ chối nghiệp vụ trong ranh giới bảo vệ đã duyệt | V59 — đã triển khai; chủ dự án xác nhận các ca chính trên Dtscr V121 / AutoCAD 2022 |
| 07 | Chống lùi đồng hồ (SEC-07) | Bảo vệ thời hạn License offline, phát hiện lùi thời gian và xử lý hợp lệ | **Tạm dừng phát triển** theo quyết định chủ dự án 26/11/2026; chưa triển khai code |
| 08 | Cải tiến xác thực máy — Machine Identity V2 (LIC-02) | SMBIOS UUID + System Serial Number + Mainboard Serial Number; tối thiểu 2 giá trị hợp lệ khớp với License đã ký | Baseline V59; nguồn ứng viên V60 đã tích hợp 08.6–08.8; Linux QA PASS; Windows/CAD NotVerified, chưa nghiệm thu |

```text
V55 — giữ nguyên 01–05
  ↓
V59 — 06 đã triển khai, kiểm nghiệm các ca chính
  ↓
Duy trì 01–06; 07 tạm dừng phát triển
  ↓
08 — Machine Identity V2 (LIC-02): SMBIOS 2/3; chưa triển khai
```

**Yêu cầu và tiến độ từng bước của hạng mục hiện tại:**

| Bước | Yêu cầu / điều kiện hoàn thành | Tiến độ |
|---|---|---|
| 06.1 | Xác định baseline V55, đọc nguồn thực tế và cơ chế SEC06; xác định phạm vi ảnh hưởng, phần giữ nguyên | Đã rà soát nguồn V55, chủ dự án cho phép triển khai 06 |
| 06.2 | Lựa chọn công nghệ ký số thành phẩm Add-in | Chủ dự án đã chọn Authenticode (26/11/2026); không tiếp tục so sánh công nghệ ký số |
| 06.3 | Thiết kế Authenticode độc lập theo quy trình build → ký → phát hành → xác minh; không ràng buộc máy build/ký, không phụ thuộc Windows trust; xác định danh tính ký, nhận diện Add-in và ranh giới cưỡng chế | Đã triển khai trong V56–V59; phạm vi kiểm tra tại Gate, không bảo đảm cưỡng chế độc lập trước nạp |
| 06.4 | Lập trình đúng phạm vi, rà soát và kiểm thử theo STD-18; giữ nguyên năm hạng mục V55 và nghiệp vụ CAD | V59: đã sửa, QA nguồn; chủ dự án báo build OK |
| 06.5 | Tích hợp thử Add-in native AutoCAD 2022 x64; kiểm tra Build/APPLOAD/Runtime, phát hiện sửa đổi và hồi quy | Dtscr V121 được dùng kiểm nghiệm thực tế theo yêu cầu chủ dự án; DJoin V42 chưa xác minh Runtime |
| 06.6 | Chủ dự án nghiệm thu theo đúng ranh giới bảo vệ đã kiểm chứng; cập nhật Plan | Chủ dự án xác nhận các ca chính Dtscr V121 OK; ca build/ký mới giữ Gate/License được phản hồi OK khi chuyển chủ đề nhưng chưa có log riêng; không tuyên bố nghiệm thu mọi host hoặc production |

Hạng mục 07 **tạm dừng phát triển**, không bị xóa khỏi Plan. Giữ nguyên kiểm tra thời hạn License hiện hành; không bổ sung TPM, đồng hồ ảo, trạng thái thời gian, dịch vụ nền hoặc thay đổi code. Chỉ tiếp tục khi chủ dự án yêu cầu lại, xác định yêu cầu và tiêu chí nghiệm thu trước khi code.

**Hạng mục 08 — DCAD Machine Identity V2 (LIC-02), 26/11/2026**

Chủ dự án thống nhất tối thiểu 2 trong 3 thông số: ID01 SMBIOS UUID (Type 1), ID02 System Serial Number (Type 1), ID03 Mainboard Serial Number (Type 2). Đây là cải tiến của LIC-02, không thay thế 01–06 và không tiếp tục 07. Baseline V59; phát triển native C++ x64, offline; không bổ sung TPM, CPU, MAC, disk, MachineGuid, WMI runtime, driver, Internet hoặc SDK thương mại cho V2. Không thay khóa/chữ ký hiện tại, Feature, Edition, thời hạn, SEC06/SEC-AT hoặc nghiệp vụ CAD. Ưu tiên giữ API Gate/Client, Integration sáu file và Add-in đã phát hành; phải xác minh khả thi trên nguồn, không mặc định tương thích.

**Nguyên tắc thiết kế:** Thu thập một bảng SMBIOS bằng API Win32; chuẩn hóa và băm SHA-256 từng thành phần riêng, có ID/phiên bản miền băm xác định; không gộp ba giá trị vào một hash dùng để so khớp tuyệt đối. Dreq → issuer kiểm tra → License ký chứa phiên bản/chính sách/ngưỡng cố định và các giá trị hợp lệ → Gate đọc lại và kiểm tra riêng từng trường theo thứ tự cố định UUID → System Serial → Mainboard Serial; không đối chiếu chéo; mỗi trường hợp lệ khớp được 1 điểm, kiểm tra đủ ba trường rồi tổng hợp. Chỉ đạt kiểm tra máy khi có ít nhất 2 thành phần hợp lệ khớp với dữ liệu gốc đã ký; các điều kiện cấp phép khác vẫn bắt buộc. Nếu cấp với 3 giá trị hợp lệ, chấp nhận 2/3; nếu chỉ 2 hợp lệ, bắt buộc 2/2; dưới 2 không cấp V2. Không tính giá trị thiếu/lỗi/default là điểm; không hạ ngưỡng, không tự ghi đè fingerprint gốc khi chạy, không fallback V1 khi V2 lỗi. Chủ dự án xác nhận chưa phát hành chính thức: V2 là phương pháp duy nhất trong bản mới; loại bỏ xác thực V1, từ chối dreq/lic V1, không fallback, không cần chuyển đổi License V1; tạo lại key thử nghiệm theo V2. Giữ V59 làm baseline/lịch sử, giữ trường phiên bản giao thức để quản lý phát triển. Chính sách cấp lại và xử lý dữ liệu thử nghiệm/DB, phát hành Gate cần chốt tại 08.5–08.6. Gate luôn so với A/B/C gốc trong License đã ký; chỉ issuer được cấp lại License. Không hiển thị raw serial, chi tiết từng trường khớp/sai hoặc logic xác thực cho khách hàng; chỉ kết quả chung, chẩn đoán chi tiết nội bộ nếu cần. Không coi giữ kín logic là lớp bảo mật mật mã.

| Bước | Nội dung / điều kiện hoàn thành | Tiến độ |
|---|---|---|
| 08.1 | Chốt ba thông số và ngưỡng tối thiểu 2/3, xác định phạm vi giữ nguyên | Chủ dự án đã chốt phương án điều chỉnh: 2/3, đúng trường/thứ tự, License gốc, V2 duy nhất, không quản lý trùng chuyên sâu; chưa duyệt đặc tả byte-level |
| 08.2 | Rà toàn bộ nguồn V59: luồng đọc máy, MachineProtocol, DCAD_ID, dreq, issuer, Licensecc, DB, Gate, Inspector, build/phát hành/QA; xác định chính xác phạm vi tác động và tương thích | Đã xác minh ZIP V59 và rà luồng chính, phát hiện GateClient khóa MachinePolicyVersion=1; chưa hoàn tất rà mọi nhánh build/publish |
| 08.3 | Bộ thu thập native độc lập, đọc 3 trường, kiểm tra biên dữ liệu, chuẩn hóa ứng viên, trạng thái valid/missing/invalid/ambiguous, không sửa License/Gate | Nguồn ứng viên 08.3; Linux GCC/Clang 38/38 mỗi compiler + 50.000 bảng biến đổi mỗi compiler; chủ dự án chạy EXE trên Windows thành công, chi tiết log build NotVerified |
| 08.4 | Kiểm nghiệm trên máy đồng bộ, laptop, máy tự lắp, máy ảo; đối chiếu PowerShell/WMI chỉ để QA; thống kê tỷ lệ đủ 2/3, placeholder, trùng, hai serial giống nhau, độ ổn định khi khởi động/cập nhật/nâng cấp | Chủ dự án xác nhận máy thật 3/3, hai serial khác nhau; quyết định dừng test thêm và chuyển bước; các máy khác và độ ổn định dài hạn NotVerified |
| 08.5 | Duyệt đặc tả byte-level: parser, UUID byte order, encoding/case/trim, danh sách invalid, Type 2 nhiều board, phân biệt lỗi đọc; vector hash; chính sách trùng tối giản, reissue, V2 duy nhất và không hạ ngưỡng | Chủ dự án cho phép code; đặc tả 08.5 được dùng cho ứng viên V60 |
| 08.6 | Thiết kế dreq/lic/DB có phiên bản, ràng buộc trong chữ ký, schema/backup/rollback nếu cần cho DB hiện hữu, không chuyển đổi License V1; kiểm tra giới hạn Licensecc và ABI; chốt kế hoạch phát hành | V60 dùng tuple 3 slot, dreq 6, envelope 4/5; ABI 2 giữ nguyên, policy 2; chưa nghiệm thu |
| 08.7 | Tích hợp DCAD_ID, issuer/LicenseccManager và Inspector; dùng chung logic định danh, kiểm tra input; cảnh báo trùng bằng DB hiện có nếu đơn giản/khả thi, không xây hệ thống quản lý trùng chuyên sâu; phát hành thử | Đã code ứng viên V60; không bổ sung cảnh báo trùng tùy chọn; Windows NotVerified |
| 08.8 | Tích hợp Gate 2/3, kiểm tra phiên bản/chính sách đã ký, từ chối sai định dạng/thiếu điểm, không fallback, không tự đổi fingerprint | Đã code Gate V8/SDK/Integration 1.2 trong V60; Windows NotVerified |
| 08.9 | QA nguồn cuối, fixture/parser lỗi, ma trận xác thực, từ chối V1, DB backup/rollback nếu có thay đổi, hồi quy 01–06; Windows build, APPLOAD/Runtime theo host và đo hiệu năng | Linux parser 38/38, policy 529/529 mỗi GCC/Clang; Integration 60/60; Windows/CAD NotVerified |
| 08.10 | Chủ dự án nghiệm thu thực tế, phát hành theo STD-17, cập nhật Plan/chứng cứ; chỉ sửa hợp đồng tích hợp nếu thực sự cần và được duyệt | Chưa triển khai |

**Cơ sở tham khảo và dependency dự kiến:** [DMTF DSP0134 3.8.0, 05/08/2024, mục 6.1.2–6.1.3, 7.2–7.3](https://www.dmtf.org/sites/default/files/standards/documents/DSP0134_3.8.0.pdf) là đặc tả bố cục trường (Type 1 serial offset 07h, UUID offset 08h dài 16 byte; Type 2 serial offset 07h là chỉ số chuỗi); [Microsoft GetSystemFirmwareTable](https://learn.microsoft.com/windows/win32/api/sysinfoapi/nf-sysinfoapi-getsystemfirmwaretable) đọc RSMB, Windows SDK Kernel32.lib/Kernel32.dll; [Microsoft BCrypt](https://learn.microsoft.com/en-us/windows/win32/api/bcrypt/nf-bcrypt-bcrypthashdata) SHA-256, Bcrypt.lib/Bcrypt.dll; C++ standard library/MSVC. [DeviceId](https://github.com/MatthewKing/DeviceId) tham khảo cách tổ chức thành phần/phiên bản và tương thích; [Thales Sentinel LDK](https://docs.sentinel.thalesgroup.com/softwareandservices/ldk/LDKdocs/ldk-ems/SPNL/LDK_SLnP_Guide/A-CloneDetection/CloneDetection_Phy.htm) tham khảo nguyên lý fingerprint/dung sai, không sao chép chính sách hay khẳng định 2/3 được hãng chứng nhận. [Microsoft ComputerSystemProduct](https://learn.microsoft.com/en-us/windows/win32/cimwin32prov/win32-computersystemproduct) và [BaseBoard](https://learn.microsoft.com/en-us/windows/win32/cimwin32prov/win32-baseboard) dùng đối chiếu QA. Dự kiến không thêm dependency bên thứ ba; xác nhận ở 08.2.

**Rủi ro và đề xuất cần đánh giá trước code tích hợp:** (a) Ba trường cùng firmware, không độc lập; nếu hai serial có cùng nội dung thì có thể chỉ là một bằng chứng thực chất, đề xuất cảnh báo/đánh giá chính sách khi cấp trước khi chốt, không tự đổi quy tắc 2/3; (b) loại UUID toàn 00/FF, serial rỗng/mặc định/placeholder; tránh chuẩn hóa quá mức làm trùng serial khác nhau; (c) parser phải giới hạn kích thước, kiểm tra header/length, chỉ số chuỗi, double-NUL, UUID byte order, Type 1 trùng và Type 2 nhiều board; không tùy tiện chọn bản ghi đầu tiên khi mơ hồ; (d) đối chiếu theo từng trường; cảnh báo trùng cặp trong DB issuer hiện có nếu đơn giản/khả thi, không bắt buộc xây quản lý trùng chuyên sâu; không tuyên bố phát hiện toàn bộ máy offline, không tự chặn mọi VM; (e) dữ liệu có thể giả mạo đồng thời, máy clone có thể giữ 2/3; thay mainboard/cập nhật firmware có thể đổi nhiều trường; chính sách cấp lại cần được duyệt, không tự chuyển License; (f) một lỗi đọc SMBIOS có thể làm cả ba trường mất; từ chối nếu không đủ hai, thông báo chẩn đoán không lộ dữ liệu raw; (g) V2 duy nhất, từ chối V1 và không fallback; xác minh giới hạn Licensecc khi mang nhiều hash đã ký, không thêm dữ liệu ngoài chữ ký; (h) DB backup/migration có transaction, tránh trùng thiết bị do dùng hash tổng hợp làm khóa bản ghi, không ghi đè fingerprint cũ; ưu tiên không lưu serial raw thường trực; (i) đọc một lần mỗi phiên kiểm tra phù hợp với Gate, không tạo tiến trình/WMI cho từng lệnh, đo thời gian thực tế, không cache kết quả cấp quyền vượt chính sách hiện hành.

**QA tối thiểu:** đủ 3/3 và mọi tổ hợp 2/3 → đạt kiểm tra máy; mọi tổ hợp 1/3 và 0/3 → từ chối; cấp 2 giá trị chỉ chấp nhận đủ 2/2; thiếu/rỗng/default/ambiguous không tính điểm; dữ liệu bảng cắt cụt, độ dài sai, index sai, nhiều Type 2, khác byte order/encoding, lỗi API → xử lý an toàn; sửa version/ngưỡng/hash trong License, trường trùng loại, V2 lỗi không fallback, License sao chép khi không đủ 2 khớp → từ chối; dreq/lic V1 bị từ chối ở hệ thống mới; Gate cũ không được dùng để cấp quyền V2 nếu không hiểu chính sách; cài Windows/thay CPU/RAM/GPU/disk/NIC không mất quyền nếu đủ 2 khớp; thay mainboard tùy kết quả; QA cảnh báo trùng nếu có, DB backup/rollback nếu có thay đổi và hồi quy chữ ký License, Feature, Edition, hết hạn, SEC06, Integration, Authenticode; kiểm thử CAD thực tế và hiệu năng. Không đánh dấu PASS khi chưa có log/chứng cứ.

**Trạng thái 08:** Chủ dự án đã chốt phương án điều chỉnh ngày 26/11/2026: kiểm tra riêng theo thứ tự, ngưỡng 2/3, luôn đối chiếu License gốc, không công khai chi tiết xác thực cho khách hàng, không quản lý trùng chuyên sâu, không hỗ trợ V1 do chưa phát hành chính thức. Quyết định này thay thế yêu cầu tương thích V1 trong đề xuất 08 trước đó; không thay các hạng mục bảo mật khác. Các chi tiết triển khai, dependency và xử lý hai serial giống nhau là đề xuất cần xác nhận theo từng bước. Tại thời điểm quyết định thiết kế chưa đọc lại ZIP V59, chưa code/build/test. Ghi nhận tiến độ mới dưới đây thay thế trạng thái này.


**Tiến độ 08.2–08.3 — 26/11/2026:** Chủ dự án cho phép code theo STD-18 và loại bỏ làm rối từ khóa. Đã xác minh ZIP V59 CRC PASS, SHA-256 `26281931c5facde26b03fff5119182063eded6c029c68cce1c1063003233b8c3`; giữ nguyên ZIP. Rà trực tiếp Machine.cpp/h, MachineProtocol, RequestFormat, DCAD_ID, RequestImport, Manager/IssueCommand/SQLite history, DcadSeal/SDK, Gate, Inspector, IntegrationModel/GateClient và build/QA liên quan. V59 dùng CryptoAPI SHA-256 (advapi32), không cần chuyển BCrypt riêng; fingerprint hiện nằm trong RSA-PSS envelope ngoài payload Licensecc, cần chốt định dạng mới và capacity. GateClient kiểm cứng machinePolicy()==1, contract khai báo MachinePolicyVersion=1; không thể giả giữ 1 để che thay đổi, cần chốt đồng bộ Integration/client/Gate version và khả năng build/ký lại Add-in tại 08.5–08.6. Chưa rà hết nhánh build/publish; chưa sửa hệ thống chính.

Đã tạo gói nguồn QA độc lập `DCAD-MachineIdentity-V2-08.3-QA.zip` SHA-256 `08eadfc5339ba8f0ae3e9cf6378d96a10be06bb5e1037d822ca6054639d64e09`, gồm parser, Win32 probe, SHA-256 trích từ V59, fixture QA, Build.cmd, hướng dẫn, phân tích phạm vi. Không đưa serial raw ra ngoài; hash từng trường chỉ có tùy chọn QA nội bộ. Chính sách ứng viên (raw UUID, ASCII printable, trim space, giới hạn serial 256 byte, Type127 nghiêm ngặt, chọn board duy nhất/hosting motherboard, blacklist, miền hash) cần kiểm nghiệm và duyệt tại 08.5; không phải đặc tả sản xuất. GCC/Clang ASan/UBSan 38/38 mỗi compiler, 50.000 bảng biến đổi mỗi compiler, vector SHA-256 đối chiếu độc lập Python; Linux dùng OpenSSL chỉ làm backend QA, không thêm dependency Windows. ZIP CRC PASS. Windows MSVC/CryptoAPI/firmware thật, CAD và tích hợp License/Gate NotVerified. Không phát hành V60, không tăng phiên bản sản phẩm, không sửa hợp đồng kỹ thuật hay hạng mục 01–07, không bổ sung obfuscation. Gói bàn giao nguồn thử nghiệm nội bộ qua ChatGPT, không tuyên bố đã upload ZIP lên GitHub. Bước tiếp: chủ dự án build/chạy probe và xác nhận kết quả trên máy thật (08.4); hoàn thiện rà soát 08.2 và duyệt đặc tả trước tích hợp. Không tự đánh dấu nghiệm thu.

**Tiến độ 08.4 và đề xuất 08.5 — 26/11/2026:** Chủ dự án cung cấp kết quả chạy MachineIdentityProbe.exe trên Windows: SMBIOS=3.3, TYPE1=1, TYPE2=1, UUID=VALID, SYSTEM_SERIAL=VALID, BOARD_SERIAL=VALID, SERIALS_EQUAL=NO, VALID_COUNT=3, ELIGIBLE_CANDIDATE=YES. Probe đã đọc và băm thành công ba trường theo luồng chương trình; không có log MSVC riêng, hash thực tế không được cung cấp. Chủ dự án xác nhận OK, không cần test thêm và yêu cầu chuyển bước. Dừng thử nghiệm mở rộng probe; các máy khác, độ ổn định khi reboot/nâng cấp, đối chiếu WMI và CAD vẫn NotVerified. Không coi đây là nghiệm thu toàn hệ thống.

**Đặc tả ứng viên 08.5:** Giữ parser và CryptoAPI SHA-256 đã có trong probe, đọc RSMB một lần cho mỗi lần thu thập; giữ UUID 16 byte raw không đảo endian. Serial đọc đúng string index, trim ASCII space đầu/cuối, giữ case và khoảng trắng giữa, ASCII printable tối đa 256 byte; kiểm placeholder không phân biệt case, theo danh sách trong probe. UUID toàn 00/FF, serial rỗng/default, index sai, dữ liệu thiếu/không rõ không tính điểm. Giữ kiểm tra biên, Type127 và quy tắc chọn Type2 của probe: ưu tiên motherboard hosting duy nhất, rồi motherboard duy nhất; nếu chỉ một Type2 chưa rõ loại thì ứng viên; trường hợp mơ hồ không tính điểm. Các giới hạn này là quyết định kỹ thuật đề xuất, có thể từ chối firmware không chuẩn. Hash chính xác: ASCII domain `DCAD-MACHINE-V2-UUID`, `DCAD-MACHINE-V2-SYSTEM`, `DCAD-MACHINE-V2-BOARD` tương ứng, theo sau một byte NUL và dữ liệu đã chuẩn hóa; SHA-256 biểu diễn lowercase 64 hex. Giữ phân biệt ID01/02/03. Đề xuất hai serial giống nhau nhưng hợp lệ vẫn đối chiếu đúng trường theo chính sách 2/3 đã chốt; ghi nhận nội bộ, không tự thay ngưỡng, chấp nhận tương quan nguồn. Không hiển thị raw/điểm thành phần ở bản khách hàng. Lỗi đọc toàn bảng hoặc lỗi băm kỹ thuật thì từ chối; thiếu trường hợp lệ được biểu diễn rõ, không dùng hash rỗng như điểm.

**Hướng 08.6 cần duyệt:** ưu tiên giữ framing dreq 5 dòng ASCII/LF/checksum và DB hardware_id TEXT hiện có; đề xuất phiên bản request mới và tuple 3 slot theo thứ tự cố định, mỗi slot 64 lowercase hex hoặc dấu '-' cho trường không hợp lệ, phân cách ':', tối thiểu 2 slot hợp lệ. Chỉ dùng tuple để vận chuyển/lưu, không băm gộp hoặc so sánh toàn tuple. Dùng envelope RSA-PSS hiện có để ký policy V2 và tuple; giữ payload Licensecc và khóa, Feature/time. Đổi định dạng envelope có phiên bản để bản cũ không chấp nhận V2; giữ hai biến thể tương ứng với tính năng V59 sau khi rà soát. Đồng bộ MachinePolicyVersion=2 ở Gate/Integration/Client, cần build/ký lại Add-in nghiệm thu do client cũ khóa 1. Tạo lại dreq/lic test; không xóa lịch sử DB cũ hay chuyển đổi license V1. Chưa duyệt tên header/chính xác layout envelope/capacity/build/publish; 08.6 sẽ chốt sau rà soát. Không phát sinh dependency mới, không sửa code V59 trong bước này. Quy ước 18 áp dụng; QA tích hợp và hồi quy cuối vẫn bắt buộc.

**Tiến độ V60 — 26/11/2026:** Chủ dự án xác nhận “Oke code” sau đề xuất 08.5. Đã tạo ZIP nguồn đầy đủ TidiDcad-System-V60 từ V59 CRC/SHA đã xác minh; SHA-256 V60 `b09e4766c300b4ba5442e30379a700e191187b75596b1040098818e702889504`, 90.714.077 byte; CRC PASS. 08.6–08.8: đã triển khai nguồn ứng viên V2 duy nhất: parser kế thừa probe, CryptoAPI SHA-256, raw UUID/trim serial, domain tách riêng, tuple ba slot 64 lowercase hex hoặc '-' phân cách ':', tối thiểu hai slot; kiểm tra đúng thứ tự, ngưỡng cố định 2, đối chiếu License gốc. Dreq DCAD-REQUEST-6; envelope DCAD-LIC-V4/V5 tương ứng common/per-feature UTC, trường ký DCAD_MACHINE_V2; version/field ký xác định chính sách cố định 2 (không thêm ngưỡng tùy chỉnh). Licensecc inner payload, RSA-PSS và khóa hiện tại giữ nguyên. DB hardware_id TEXT, không migration schema hoặc xóa lịch sử. Không triển khai cảnh báo trùng tùy chọn hay obfuscation. Hai serial hợp lệ giống nhau vẫn đối chiếu riêng; không tuyên bố độc lập nguồn. Gate V8, ABI 2 giữ, MachinePolicyVersion 2, Add-in Policy 1 giữ, Integration 1.2, sáu file và cơ chế chứng nhận giữ; cập nhật Inspector, mock, build và embedded. Add-in client Policy 1 cần Integration mới, build/ký lại; không sửa nghiệp vụ CAD, chưa tạo Add-in mới. Không thay SEC06/SEC-AT, dependency, issuer identity, các quyền và thời hạn. Không tiếp tục 07.

08.9: nguồn cuối Linux GCC/Clang ASan/UBSan: parser 38/38 mỗi compiler và 50.000 bảng biến đổi mỗi compiler; 529 ca policy/format mỗi compiler, request/import/issue command PASS; AddinContract 27/27 mỗi compiler; Integration model GCC 60/60; bốn embedded byte-match, 602/602 dependency hash, ZIP CRC PASS. QA backend OpenSSL chỉ trên Linux, không thêm dependency Windows. Review riêng phát hiện và sửa dấu ngoặc thừa và fixture cũ chỉ đổi một mã (không còn phải DENY nếu đủ 3 trường); Windows signed QA bổ sung một trường đổi, toàn bộ trường đổi, sửa tuple, header/field V1, giữ các ca hồi quy. MSVC full Build, RSA-PSS/CryptoAPI thật V60, PowerShell, GUI, Gate load, Authenticode, CAD, hiệu năng và rollback NotVerified; chưa nghiệm thu 08.10, không công bố production. Gói nguồn được bàn giao trong ChatGPT, chưa upload ZIP lên GitHub. Baseline duy trì vẫn V59 đến khi V60 được nghiệm thu. Giữ hợp đồng kỹ thuật STD-20 mục 2 trở đi chưa đổi trong đợt này. Bước tiếp: backup, build V60 trên Windows, tạo dreq/lic mới, xuất Integration 1.2 và nghiệm thu Add-in AutoCAD 2022 sau khi build/ký lại; không trộn thành phẩm V59/V60.

**Quản lý và cập nhật Plan (bắt buộc):**

- Trước mỗi phiên phát triển TidiDcad, đọc lại STD-20/Plan và STD-18; xác định đúng baseline, bước hiện tại, yêu cầu đã thống nhất, phần phải giữ nguyên và điều kiện chuyển bước.
- Khi chủ dự án giao yêu cầu mới hoặc xác nhận thay đổi, cập nhật trực tiếp yêu cầu, phạm vi và tiêu chí nghiệm thu của bước liên quan trong Plan. Không tự coi phương án đang nghiên cứu là đã chốt.
- Sau mỗi bước thực tế, cập nhật ngay trạng thái, phiên bản nguồn, kết quả thực hiện, bằng chứng (commit/log/test nếu có), hạn chế hoặc blocker và hành động tiếp theo. Nếu chưa có bằng chứng thì ghi rõ Chưa xác minh/NotVerified; không tự đánh dấu PASS.
- Chỉ triển khai bước tiếp theo khi bước trước đủ điều kiện và chủ dự án đã xác nhận chuyển bước. Khi có lỗi, xác định nguyên nhân và cập nhật bước cần sửa lại; không bỏ qua bước chưa đạt hoặc tự thay đổi các hạng mục đã chốt.
- Mỗi lần cập nhật ghi ngày, bước, nội dung yêu cầu/thay đổi, trạng thái trước/sau, bằng chứng/xác nhận và việc tiếp theo; lưu lịch sử qua Git. Chỉ thay đổi hợp đồng kỹ thuật STD-20 khi phương án mới đã được duyệt theo phạm vi tương ứng.
- Không mở rộng sang chống dịch ngược chuyên sâu, mã hóa code, obfuscation, bảo vệ RAM chuyên sâu, USB dongle, cloud hay hạng mục khác nếu chưa được giao. Không tăng version phần mềm chỉ vì cập nhật Plan.

**Ghi nhận tiến độ:** 26/11/2026 — V55 được chủ dự án xác nhận OK; chốt giữ 01–05, thực hiện 06 trước 07. Đã nghiên cứu sơ bộ phương án ký Add-in; chưa đọc trực tiếp nguồn V55 trong đợt nghiên cứu này, chưa chốt thiết kế, chưa code hoặc nghiệm thu 06. Chưa xác minh độc lập ZIP nguồn trên GitHub, log MSVC, kiểm thử PFX và Build/APPLOAD/Runtime DJoin/Dtscr của V55. Chữ ký số và kiểm tra trong cùng tiến trình không tự chứng minh khả năng chặn mọi Add-in bị sửa; phải đánh giá ranh giới cưỡng chế trước khi nghiệm thu. Bước tiếp theo: 06.1.


**Cập nhật 06.1 — 26/11/2026:** Đã đọc ZIP nguồn TidiDcad-System-V55 (SHA-256 `089342d6274ac5b78909c395eb16ebca1136ec0b11af1a5dfd05f3aa4e598875`), DJoin-V41 và Dtscr-Integration(2); CRC ba ZIP đạt. Rà tĩnh Gate V6/API 2, SDK xác minh License, GateClient và luồng lệnh DJoin; GateClient ba gói trùng SHA-256. Kết luận: SEC06 xác minh Gate, không xác minh tính toàn vẹn của chính Add-in; hợp đồng Gate hiện tại không ràng buộc danh tính/thành phẩm Add-in đã ký. Chữ ký bộ Integration không bao phủ thành phẩm build sau đó. Do vậy V55 không bảo đảm phát hiện Add-in bị sửa. Giữ nguyên 01–05; hạng mục 06 cần xác định cơ chế xác minh thành phẩm và ranh giới cưỡng chế. Chưa có kiểm thử sửa binary trên Windows/CAD, Build/APPLOAD/Runtime NotVerified; DJoin V41 chỉ dùng làm nguồn đối chiếu, không coi là bản đã nghiệm thu tích hợp V55. Không sửa code, không tăng phiên bản phần mềm, chưa chốt công nghệ. Trạng thái 06.1: đã rà tĩnh, chờ chủ dự án xác nhận; sau xác nhận mới chuyển 06.2.

**Nghiên cứu 06.2 — 26/11/2026 (chưa chuyển cổng 06.1):** Đối chiếu [OpenSSL RSA-PSS](https://docs.openssl.org/3.1/man7/RSA-PSS/), [TUF](https://theupdateframework.org/docs/metadata/) ([CNCF Graduated](https://www.cncf.io/projects/the-update-framework-tuf/), [audit công khai](https://theupdateframework.io/docs/security/audits/)), [osslsigncode](https://github.com/mtrojnar/osslsigncode), [Sigstore Cosign](https://github.com/sigstore/cosign), [Microsoft WinVerifyTrust](https://learn.microsoft.com/en-us/windows/win32/api/wintrust/nf-wintrust-winverifytrust), [App Control](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/appcontrol-and-applocker-overview) và [Autodesk ký ObjectARX](https://help.autodesk.com/cloudhelp/2023/ENU/AutoCAD-Customization/files/GUID-3DA95353-9EF3-4E29-9671-6AEB7704EBE6.htm). Kiểm tra lại DcadGateClient.cpp V55: đã có BCrypt SHA-256/RSA-PSS. Đề xuất nghiên cứu ưu tiên tận dụng cơ chế SEC06 hiện có, tham khảo mô hình danh mục hash có chữ ký của TUF để quản lý nhiều Add-in; Authenticode là lựa chọn đối chiếu nếu cần chữ ký nhúng. Chữ ký/danh mục phát hiện thay đổi, không tự bảo đảm cưỡng chế trước nạp; App Control là lựa chọn kiểm soát độc lập có điều kiện, chưa duyệt áp dụng tại khách hàng. Không mặc định thêm dependency, khóa, file runtime hoặc đổi hợp đồng. Chưa build/test ứng viên, chưa thử ARX trên Windows/CAD, chưa xác nhận khả năng cưỡng chế và chưa chốt phương án. Không thay trạng thái nghiệm thu 06.1, không sửa code hay tăng phiên bản.

**Quyết định 06.2 — 26/11/2026 (chủ dự án):** Chọn Authenticode theo tiêu chuẩn Microsoft làm công nghệ ký số nhúng cho thành phẩm Add-in ARX/DLL. Quyết định này thay thế đề xuất ưu tiên RSA-PSS + danh mục kiểu TUF trong ghi nhận nghiên cứu 06.2 trước đó; dừng so sánh công nghệ ký số, không tự xây định dạng chữ ký nhúng riêng. Dự kiến sử dụng SignTool để ký thành phẩm sau khi build và API WinVerifyTrust để xác minh, kèm kiểm tra danh tính nhà phát hành DCAD tin cậy độc lập. Cần xác định chứng thư ký mã phù hợp, khả năng dùng lại danh tính SEC06, chính sách xác minh offline, vị trí và hiệu lực cưỡng chế kiểm tra, tương thích ObjectARX và tiêu chí nghiệm thu tại 06.3. Không mặc định chữ ký hợp lệ là đủ để chặn Add-in bị sửa trước nạp, không tự thay chính sách Windows/CAD khách hàng. Giữ nguyên 01–05 và Integration sáu file; chưa quyết định thêm khóa, dependency hoặc thay hợp đồng. Đã chốt công nghệ ở 06.2, chưa xác nhận nghiệm thu 06.1 riêng, chưa duyệt thiết kế 06.3, chưa ký thử ARX hoặc Build/APPLOAD/Runtime, chưa sửa code, giữ baseline V55.

**Yêu cầu bổ sung 06.3 — 26/11/2026:** Chủ dự án muốn hệ thống ký/xác minh Authenticode của TidiDcad tách riêng hoàn toàn khỏi hệ thống quản lý bảo mật Windows. Thiết kế cần ưu tiên tự quản lý danh tính/chứng thư và public trust DCAD, không bắt cài chứng thư vào Windows Certificate Store, không phụ thuộc Windows trust policy, Internet hay CA bên ngoài để xác minh nội bộ; khảo sát khả năng ký và xác minh không dùng SignTool/WinVerifyTrust. Ứng viên tham khảo: [osslsigncode](https://github.com/mtrojnar/osslsigncode) (ký/xác minh Authenticode độc lập), [LIEF](https://lief.re/doc/stable/tutorials/13_pe_authenticode.html) (C++ xác minh PE Authenticode không cần WinTrust, hỗ trợ kiểm tra trust riêng). Đây là yêu cầu thiết kế và ứng viên, chưa duyệt dependency, cách quản lý khóa/chứng thư, vị trí cưỡng chế hoặc tích hợp. Không thay năm hạng mục V55, không đổi cấu hình Windows/CAD khách hàng. Cần kiểm tra riêng việc nạp ARX trong AutoCAD theo chính sách host hiện hành; xác minh độc lập không đồng nghĩa cưỡng chế độc lập trước thực thi. Chưa thử nghiệm/chưa sửa code.

**Yêu cầu và quy trình 06.3 — độc lập máy tính (26/11/2026):** Chủ dự án yêu cầu loại bỏ ràng buộc máy khỏi thiết kế ngay từ đầu, không biến việc ký trên máy A và xác minh trên máy B/C thành bước phát triển hay thử nghiệm riêng. Mọi máy có môi trường phát triển phù hợp đều build được; mọi máy có TidiDcad và danh tính ký DCAD hợp lệ do quản trị viên mở khóa đều có thể ký, kể cả cùng máy build. Không có máy ký cố định, đăng ký/phê duyệt máy, mã máy/SID/tài khoản Windows làm điều kiện build hoặc ký, khóa ký ràng buộc phần cứng hay kho chứng thư Windows, cấu hình trust Windows/Registry riêng, CA bên ngoài hay Internet bắt buộc. Không đưa lại phê duyệt máy build V1.1. Không bỏ kiểm tra DeviceID của License khách hàng.

**Lưu trình triển khai chi tiết 06.3 — Chống sửa Add-in**

Chỉ triển khai phạm vi chống sửa Add-in. Yêu cầu đã chốt: Authenticode nhúng vào thành phẩm sau build; độc lập máy build/máy ký, tự quản lý trust DCAD, xác minh offline, không cài chứng thư vào Windows, không phát sinh file chữ ký riêng cho từng Add-in. Không thêm giai đoạn thử nghiệm riêng theo máy.

| Công việc | Yêu cầu thực hiện | Đầu ra / điều kiện hoàn thành | Tiến độ |
|---|---|---|---|
| 06.3.1 — Danh tính ký | TidiDcad quản lý chứng thư và private key di chuyển/khôi phục được; dùng thông tin xác thực quản trị, không dùng mã máy, SID, TPM hoặc kho chứng thư Windows làm điều kiện. Đối chiếu khả năng tái sử dụng danh tính SEC06 trước khi quyết định tạo danh tính mới. Đề xuất PFX mã hóa bằng mật khẩu; public trust phải được cung cấp độc lập với Add-in đang kiểm tra. | Thiết kế lưu, mở khóa, sao lưu/khôi phục và phân phối public trust; không lộ private key vào source, Integration hoặc máy khách. | Đã đối chiếu cơ chế PFX portable V55; chờ chốt danh tính ký và phạm vi tác động |
| 06.3.2 — Ký Add-in | Build độc lập với chức năng ký. TidiDcad tiếp nhận thành phẩm cuối, ký Authenticode nhúng bằng công cụ độc lập (đề xuất osslsigncode), xác minh lại chữ ký và danh tính DCAD trước phát hành. Không ký tự động mọi file bất kỳ mà không có xác nhận phát hành. Lỗi thì không công bố, giữ bản hợp lệ trước đó. | Một file ARX/DLL đã ký, không thêm file chữ ký đi kèm; xác định công cụ, dependency và điều kiện phát hành. | Chưa triển khai |
| 06.3.3 — Xác minh Add-in | Đề xuất Gate dùng LIEF C++ để xác minh cấu trúc chữ ký Authenticode, hash nội dung được bảo vệ, chữ ký mật mã và danh tính ký DCAD theo public trust riêng; không tin chứng thư chỉ vì nó nằm trong Add-in. Ràng buộc đúng Add-in thực tế với Tool/Feature/host đã được phát hành; xác định hợp đồng nhận diện và metadata được ký, không chỉ tin đường dẫn hoặc ID caller tự khai. | Luồng xác minh rõ ràng, từ chối chữ ký thiếu/sai, file bị sửa trong vùng ký, signer khác hoặc Add-in không tương ứng; không cần Windows trust hay Internet. | Chưa triển khai |
| 06.3.4 — Hiệu lực kiểm tra | Thiết kế kiểm tra trước khi Gate cấp quyền, xử lý lỗi theo hướng DENY. Xác định ranh giới bảo vệ và cách nhận diện module thực tế; Gate không mặc nhiên là cơ chế cưỡng chế trước nạp độc lập. Không tự thay chính sách Windows/CAD khách hàng. Nếu không thể bảo đảm chặn Add-in đã sửa không gọi Gate trong các ràng buộc hiện tại, ghi blocker và trình chủ dự án duyệt phạm vi bảo vệ thực tế; không tuyên bố chống sửa tuyệt đối. | Hợp đồng Gate/Client, vị trí kiểm tra, phạm vi bảo vệ và tiêu chí nghiệm thu được chủ dự án duyệt trước 06.4. | Chưa chốt |

**Luồng vận hành:** Build Add-in chưa ký → TidiDcad mở khóa danh tính DCAD → ký Authenticode nhúng → xác minh lại và phát hành file đã ký → Gate xác minh Add-in thực tế bằng public trust DCAD → PASS mới tiếp tục xét cấp quyền; lỗi thì DENY. Cùng một máy hoặc các máy khác nhau đều được, không có máy cố định.

**Tiêu chí nghiệm thu của phạm vi này (thực hiện trong 06.4–06.6, không thêm giai đoạn riêng):** ký và xác minh thành phẩm hợp lệ; từ chối thiếu chữ ký, sửa nội dung được Authenticode bảo vệ, chữ ký sai, signer khác, không đúng Add-in/Tool/host; không phát hành nếu ký/xác minh lỗi; không cần private key lúc build, không cần cài chứng thư Windows hoặc Internet cho xác minh nội bộ; xác nhận tương thích APPLOAD/Runtime của DJoin AutoCAD 2022 x64. Kiểm tra theo STD-18 trên nguồn cuối cùng. Kiểm tra chữ ký Authenticode không bao phủ mọi byte PE và không chứng minh toàn vẹn RAM; từ chối cấp quyền tại Gate không đồng nghĩa chặn mọi mã trước thực thi.

**Rà soát 06.3.1 — 26/11/2026:** Đọc `Source/ReleaseTrust/PortableSecurity.ps1` (Open-SecPfx, Open-SecApproved, New-SecPfx, Backup/Restore/ChangePassword), `Source/TidiDcad/Source/ReleaseSecurity.cpp` và `SEC06-README.md` của ZIP V55. Đã có PFX portable, tài khoản DCAD, mật khẩu, KeyId/pin, sao lưu .dbackup, khôi phục và đổi mật khẩu; mở PFX bằng EphemeralKeySet, không đăng ký Certificate Store. Cơ chế SEC06 hiện dùng API mật mã Windows/.NET, nhưng không ràng buộc máy hoặc tài khoản Windows; giữ nguyên. New-SecPfx sinh chứng thư tự ký RSA 3072 làm container khóa, không khai báo EKU Code Signing; chưa xác nhận PFX thực tế hoặc tương thích Authenticode. Đề xuất tái sử dụng cơ chế quản lý, ưu tiên không đổi khóa/chứng thư SEC06 hiện hữu; quyết định dùng chung khóa hay danh tính ký mã riêng cần đánh giá tương thích và phân tách mục đích, chủ dự án duyệt trước code. Nếu cần danh tính riêng, dùng cùng giao diện/quy trình quản lý portable, không thay đổi danh tính SEC06, không thêm đăng ký máy. Phạm vi dự kiến chỉ ReleaseSecurity/PortableSecurity và dữ liệu trust liên quan nếu thực sự cần; phần xác minh Gate/Client thuộc 06.3.3–06.3.4. Đây là kết quả đọc nguồn, chưa thay đổi source hay khóa, chưa nghiệm thu 06.3.1.

**Tổng hợp tạm thời 06.3 — phạm vi sửa và tác động (26/11/2026):** Chỉ hạng mục chống sửa Add-in, baseline V55. Đây là thiết kế đề xuất chờ duyệt, không phải kết quả triển khai.

| Thành phần | Thay đổi dự kiến một lần | Khi build Add-in phiên bản mới |
|---|---|---|
| Danh tính ký Add-in | Ưu tiên danh tính Code Signing riêng, tận dụng cơ chế PFX portable/mật khẩu/backup hiện có; chưa duyệt tạo khóa | Giữ nguyên khóa và danh tính |
| TidiDcad | Thêm chức năng tiếp nhận thành phẩm, ký Authenticode nhúng bằng công cụ độc lập (đề xuất osslsigncode), xác minh và phát hành; tác động ReleaseSecurity.cpp/.h, PortableSecurity.ps1 và PortableSecurityEmbedded.h nếu cần; rà build/phát hành liên quan | Chọn và ký file thành phẩm mới; không build lại TidiDcad |
| DcadLicenseGate.dll | Bổ sung public trust ký Add-in và bộ xác minh Authenticode (đề xuất LIEF C++); ưu tiên liên kết tĩnh, không tạo Gate DLL khác. Tác động DcadLicenseGate.cpp/.h và build/QA trực tiếp nếu cần | Dùng lại Gate đã nâng cấp khi khóa, hợp đồng và chính sách không đổi |
| GateClient / Integration | Xác định hợp đồng nhận diện module thực tế, dữ liệu định danh Tool/Feature/host được bảo vệ; cập nhật DcadGateClient.cpp/.h, DcadLicenseGate.h và IntegrationExport/IntegrationEmbedded nếu hợp đồng thay đổi, ưu tiên giữ sáu file | Dùng lại Integration tương thích |
| Add-in | Tích hợp client tương thích, build rồi ký trực tiếp vào thành phẩm | Build rồi ký lại; không tạo file chữ ký bên ngoài |
| License .lic | Không thay đổi định dạng, khóa cấp hay quyền đã cấp; kiểm tra hồi quy | Giữ nguyên theo điều kiện License hiện hành |

Luồng ký: build thành phẩm cuối → tính hash Authenticode → ký dữ liệu chứa hash bằng private key DCAD → nhúng chữ ký → kiểm tra lại → phát hành. Luồng xác minh: Gate xác định đúng module yêu cầu → xác minh chữ ký và khóa tin cậy độc lập → tính lại hash vùng được bảo vệ → so sánh → chỉ tiếp tục xét cấp quyền nếu đạt. Không yêu cầu biết trước hash từng phiên bản, không ràng buộc máy build/ký, không cài chứng thư vào Windows để xác minh nội bộ. Gate V55 chưa có tính năng mới; cần xác định chuyển đổi Add-in chưa ký/Gate cũ, không mặc định tương thích hoặc mở đường bỏ qua kiểm tra. Gate DENY chỉ có hiệu lực với luồng đi qua Gate; chưa chốt cưỡng chế độc lập, nhận diện module, chính sách phiên bản/chuyển đổi, cách lưu danh tính và dependency. Chưa sửa code, chưa chốt thiết kế, chưa nghiệm thu; cập nhật Plan để chủ dự án xác nhận.

**Trạng thái:** 26/11/2026 — đã lập lưu trình chi tiết theo yêu cầu đã thống nhất; PFX, osslsigncode, LIEF và hợp đồng nhận diện/cưỡng chế vẫn là thiết kế đề xuất, chưa duyệt hoàn toàn. Chưa sửa source V55, chưa nghiệm thu; không tự chuyển 06.4 trước khi chủ dự án duyệt.


**Tiến độ V56 — 26/11/2026:** Chủ dự án đã cho phép code. Đã tạo bản nguồn TidiDcad V56 và DJoin V42 (C22), Gate V7, Integration 1.1. 06.3.1–06.3.4 và 06.4/06.5: nguồn ứng viên đã triển khai, không phải nghiệm thu. Danh tính Code Signing riêng RSA3072/PFX portable, pin SHA-256 chứng thư; giữ nguyên SEC06/Licensecc. Ký Authenticode bằng osslsigncode, xác minh LIEF tĩnh, GateClient không fallback, xuất thành phẩm cùng tên vào DProgram khác không ghi đè. QA Linux: 27/27 ca cấu trúc Add-in trên GCC/Clang, 50.000 mẫu biến đổi với ASan/UBSan; Integration 60/60 trên GCC/Clang; Gate mock/ABI, Catalog/layout, JoinMath PASS; rà soát byte nguồn ngoài phạm vi và source nhúng PASS. Windows/MSVC, ký/xác minh Authenticode thực, Gate module caller, DJoin Build/APPLOAD/Runtime đều NotVerified do môi trường thiếu Windows/CAD/dependency. Không có công cụ bên ngoài hay khóa riêng kèm ZIP; DJoin yêu cầu Integration thực do issuer xuất. API legacy còn cho tương thích; Gate không cưỡng chế mã bị sửa bỏ qua Gate, không chứng minh RAM/pre-load; chưa chốt chấp nhận giới hạn này. 06.6 chưa nghiệm thu, không production. SHA256 nguồn V56: 8c8ccb1ff64b74c4bc56679ac2fe5afb8a999c92fbc35af31e138c62c3dbeddf; DJoin V42: cf20efefbbccb7398cc262d8a00ae422a29db518f55ee8a6ac2614f076667ba5. Hướng dẫn và QA nằm trong ZIP.

**Tiến độ V57 — 26/11/2026:** Tích hợp bộ DCAD-Dependencies-Win64.zip do chủ dự án cung cấp: LIEF 1.0.0 static x64 /MD và osslsigncode 2.15 portable, cùng nguồn upstream/notices. Chỉ sửa đóng gói/build/phát hành dependency, không sửa logic bảo mật V56; Gate V7, Integration 1.1, DJoin V42 giữ nguyên. Build tự xác minh manifest SHA256, dùng SDK trong ZIP, xuất signer/notices qua transaction hiện có; không tải/cài thêm hai dependency. Kiểm tra 602/602 hash và ZIP PASS; Linux GCC AddinContract 27/27 (50.000 biến đổi), Integration 60/60, đối chiếu phạm vi PASS. Log collector Windows MSVC 19.51 xác nhận compile/link/load probe V56 và osslsigncode version; full Windows V57 build, Authenticode thực và CAD NotVerified, chưa nghiệm thu. Cần toolchain tương thích MSVC 19.51, các điều kiện build cũ không đổi. Chỉ bàn giao kiểm nghiệm nội bộ, chưa duyệt phân phối thương mại dependency. SHA256 ZIP V57: 396c64cf763323b776c159cc083441f892a6e0d8a9835b5f051391775ccd89c7.

**Tiến độ V58 — 26/11/2026:** Hai log Windows V57 xác nhận TidiDcad/DCAD_ID build và publish PASS, root dừng ADDIN_PUBLIC_TRUST_MISSING; Gate/License group chưa chạy. Rà nguồn phát hiện New-SecPfx -Addin có ba X509ExtensionCollection.Add trả Int32 vào pipeline; sửa bỏ output thừa, kiểm tra kiểu byte[], đồng bộ mã nhúng. Thêm QA danh tính biệt lập trước live trust preflight (tạo, kiểm tra, sai mật khẩu, lặp setup, sao lưu, khôi phục cùng pin). Chưa có log GUI nên chưa xác nhận lỗi thực tế chính là lỗi nguồn này. Diff đảo ngược, đồng bộ nguồn nhúng, 602/602 hash dependency, ZIP PASS trên Linux; Windows V58 QA/Build/GUI/Authenticode/CAD NotVerified. Giữ nguyên Gate, Licensecc, SEC06, Integration và dependency. ZIP V58 SHA256: 40020ec0664ef096137ba00a74430bc1049d2798700a49bbb64e8c7c4a757c5d. Chưa nghiệm thu.

**Tiến độ V59 — 26/11/2026:** Log V58 đã vượt live Add-in trust preflight đến SEC06 QA; dừng do fixture bundle thiếu osslsigncode.exe. Đối chiếu publisher xác nhận còn thiếu hai notice và DcadAddinVerify.exe ở hai QA root; recovery allowlist chưa có ba signer outputs. V59 sửa fixture bằng dependency thực đã pin hash, bổ sung verifier giả lập, ca sai hash/rollback/recovery/final verification; đồng bộ embedded, không sửa publisher/thuật toán ký/xác minh/Gate/Licensecc/Integration. Linux đối chiếu đường dẫn, 602/602 hash, phạm vi diff, embedded và ZIP PASS; AddinContract 27/27 (50.000 mẫu), Integration model 60/60 PASS. Windows SEC06 QA mới, full Build/Authenticode/CAD NotVerified. ZIP SHA256 26281931c5facde26b03fff5119182063eded6c029c68cce1c1063003233b8c3. Chưa nghiệm thu production.

**Quyết định và trạng thái hiện hành — 26/11/2026:** Chủ dự án xác nhận V59 build OK; Dtscr V121 (Tool 019, Feature 019-01, Gate V7, Integration 1.1, AutoCAD 2022 x64) build, ký và chạy OK theo phản hồi; ARX chưa ký bị Gate từ chối `Gate status=9: ADDIN_PE_INVALID`; ARX đã ký bị sửa một byte trong vùng bảo vệ bị AutoCAD cảnh báo Invalid Signature và Gate từ chối `Gate status=9: ADDIN_SIGNATURE_INVALID`; khôi phục bản nguyên gốc được chủ dự án xác nhận OK. Phản hồi cuối “oke” khi chuyển sang 07 không kèm log riêng cho ca build/ký lại giữ nguyên Gate/License. Ghi nhận kết quả trong phạm vi thực tế, không khẳng định chống sửa tuyệt đối hoặc nghiệm thu mọi host. Chọn V59 làm baseline duy trì; 07 đã khảo sát nhưng **tạm dừng phát triển** theo yêu cầu chủ dự án vì chi phí/phức tạp và giới hạn bảo vệ offline; chưa code, giữ kiểm tra hết hạn hiện có. Không thay hợp đồng kỹ thuật STD-20, không tạo phiên bản phần mềm mới; ZIP V59/V121 trên GitHub chưa xác minh. Các ghi nhận trước đây là lịch sử, trạng thái này thay thế trạng thái cũ.

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

**Tại bên nhận:** xác thực chữ ký bộ bàn giao theo INT-06, dựa vào danh tính SEC06 đã được tin cậy độc lập. Không dùng khóa trong chính ZIP để tự chứng minh nguồn gốc. KeyId thay đổi phải được duyệt và tích hợp/build lại các Add-in liên quan. Thay mật khẩu nhưng giữ KeyId không cần đổi public header. Phân biệt SEC06 ký Gate với khóa Licensecc cấp License.

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

### INT-06. Xác thực bộ bàn giao — V1.2 (thay thế toàn bộ cơ chế phê duyệt máy build V1.1)

1. TidiDcad kiểm tra Tool/Feature, target, Header/Pin và trạng thái SEC06, mở khóa PFX bằng tài khoản phát hành hiện hành, ký RSA-PSS SHA-256 salt 32. Không tạo khóa khác; ký bộ bàn giao là một miền chữ ký độc lập với Gate manifest. TidiDcad phải xác minh chữ ký và kiểm tra ZIP đọc lại trước khi công bố. Hủy, thiếu/sai tài khoản, đổi snapshot, pending hoặc ký/ghi lỗi thì không công bố, bảo toàn ZIP cũ.
2. Giữ đúng sáu file. `IntegrationVersion` vẫn là "1.0" (mở rộng tương thích); bổ sung thành viên cuối `Certification` trong ToolInfo.json theo định dạng canonical:
```json
"Certification": {"Issuer": "TidiDcad", "Format": "DCAD-INTEGRATION-V1", "Algorithm": "RSA-PSS-SHA256", "KeyId": "<64hex>", "PackageId": "<64hex>", "ContentHash": "<64hex>", "SignatureHex": "<768-1024 hex>"}
```
   `ContentHash` là SHA256 của ZIP canonical unsigned: bỏ phần Certification cuối gồm dấu phẩy và LF trước nó, phục hồi đúng LF + } + LF kết thúc ToolInfo.json, giữ nguyên byte của năm file còn lại. Thứ tự sáu file theo mục 2; ZIP stored không nén, DOS timestamp 1980-01-01 00:00:00, flags 0, version needed/made 20, không extra/comment, các header local/central/EOCD, CRC32, size, offset chuẩn, thuộc tính khác bằng 0. PackageId = ContentHash (cùng nội dung cho cùng mã, không phải số thứ tự lần xuất). Dữ liệu ký ASCII có LF cuối:
```text
DCAD-INTEGRATION-V1
KeyId=<64hex>
ContentHash=<64hex>
```
   SignatureHex dạng hex thường, kích thước tương ứng modulus RSA 384–512 byte. Xác thực tất cả sáu entry, reject unsigned/tamper/extra/repack; không trộn chữ ký Gate với chữ ký bộ bàn giao.
3. Xác thực khi bàn giao, không lặp tại mỗi build. TidiDcad V55 có nút Kiểm tra ZIP dùng Header/Pin của issuer hiện hành, không lấy trust từ ZIP. Khi xuất, ứng dụng tự xác minh và hiển thị SHA-256 của ZIP thực tế; nếu máy nhận khác, đối chiếu SHA-256 do quản trị viên xác nhận qua kênh tin cậy (ví dụ `certutil -hashfile "Djoin-Integration.zip" SHA256`). Có thể kiểm lại ZIP nguyên byte tại TidiDcad khi cần, hoặc dùng verifier đã có public key đáng tin cậy từ trước; không yêu cầu cài TidiDcad ở máy build. Chữ ký tự khớp với khóa trong ZIP không đủ chứng minh nguồn gốc. Không tích hợp khi không xác thực được nguồn, hash hoặc chữ ký sai.
4. Người nhận đối chiếu ToolId, Feature, target, ánh xạ nghiệp vụ với yêu cầu đã duyệt; lưu ZIP gốc và bằng chứng xác thực, giữ đúng byte bốn file bảo mật. Khi source bảo mật bị thay đổi, phải xác thực lại trước build/phát hành. Thiếu public header hoặc sai ánh xạ phải dừng. Không cần phê duyệt máy build; bỏ riêng ApprovedGateKeyId.txt, hai biến môi trường phê duyệt, Check-ReleaseTrust.ps1 và hook phê duyệt trong CMD/project khi chuyển từ V1.1. Không bỏ các kiểm tra SDK, build và runtime của baseline. Không yêu cầu đổi ExecutionPolicy, thêm EXE, tài khoản SEC06 hoặc Gate/License khi compile. Build_All/Build_Release/MSBuild dùng nguồn đã xác thực, giữ quy trình build gốc. Gói cũ chưa ký phải xuất lại cho lần tích hợp mới.

### INT-07. Build_All.cmd — luồng bắt buộc

1. Xác định root từ script, target, đường dẫn SDK và output; tạo BuildRunId, log/binlog và staging riêng từng lần; không lấy log cũ. Nếu thiếu/sai đầu vào, dừng trước compile/publish, ghi lỗi riêng lần build. Không thao tác vào dữ liệu vận hành.
2. Kiểm tra nguồn và project, sáu file tích hợp; kiểm tra ID/Feature, khóa tin cậy đã duyệt; kiểm tra toolchain/SDK.
3. Gọi Build_Release.cmd cho từng target đã duyệt, truyền thông số chính xác, nhận exit code. Không lấy output cũ để bù khi lỗi; MSBuild nonzero phải dừng. Xác minh output tạo mới, không rỗng; không dùng verification build để hợp thức hóa artifact cũ.
4. Publish đúng tên thành phẩm theo quy ước, tạo thư mục nếu thiếu; kiểm tra nội dung file đích bằng SHA-256 hoặc so sánh byte. Không phá hủy bản đã phát hành chỉ vì build mới thất bại; dùng staging riêng, chỉ công bố kết quả đủ điều kiện.
5. Log chính ghi baseline, target, ToolId, Feature, KeyId, SDK root, toolset, bước build, output, kết quả và lỗi; log target tách riêng. Exit 0 chỉ khi build/publish đúng phạm vi; lỗi nonzero. Không coi exit 0 là APPLOAD/Runtime PASS.

**Khác mẫu V38:** bỏ phụ thuộc đồng bộ từ máy TidiDcad trong `Sync-ReleaseTrust.ps1` của Build_All khi bộ Integration đã chứa Public Key thật; xác thực bộ bàn giao khi tiếp nhận theo INT-06. V38 còn xóa output staging trước khi build và có trạng thái SETUP REQUIRED (exit 2); quy ước mới yêu cầu không mất bản phát hành cũ, thiếu khóa bàn giao phải FAIL rõ nguyên nhân. Không tạo PFX/khóa, không đăng nhập, không yêu cầu Dcad.lic, không build/copy DLL hoặc manifest Gate. Không tự chạy ứng dụng hoặc mở Explorer. Có thể dùng `/nopause` cho build tự động.

### INT-08. Build_Release.cmd và SDK

Ví dụ AutoCAD 2022: reference `D:\000\Reference\Autocad 2022`; ObjectARX 2022 x64; Visual Studio/MSBuild có MSVC x64, v142 ưu tiên. V38 hỗ trợ v143/v145 thay thế hoặc `DCAD_MSVC_TOOLSET`, nhưng phải nghiệm thu CAD trên toolset thực tế. Có thể dùng `DCAD_REFERENCE_ROOT` khi được phê duyệt; không dùng file SDK trộn từ nhiều root.

`Buildtime/ResolveC22SDK.ps1` V38 kiểm tra arx.props/dbx.props, header và library cần thiết; script là nguồn danh sách chính xác cho baseline, không áp đặt cho target khác. Tìm một root SDK đầy đủ, file không rỗng; nếu nhiều bộ đầy đủ nhưng nội dung khác thì dừng; nếu giống nhau từng byte thì có thể chọn nhất quán. Không mở hai luồng ghi đồng thời vào cùng log (V38 SDK resolver tự ghi log).

Lệnh mẫu V38:

```cmd
"%MSBUILD%" "%BASE%DJoin.vcxproj" /t:Rebuild /p:Configuration=Release /p:Platform=x64 "/p:PlatformToolset=%TOOLSET%" "/p:ARX2022_SDK=%SDKROOT%" /m:1 /nologo "/flp:LogFile=%LOG%;Verbosity=normal;Append"
```

Không đóng gói SDK dùng chung vào ZIP nguồn. Output V38: `bin\DJoin.arx`; intermediate `Buildtime\obj\`; staging root `Dprogram\AutoCAD2022\DJoin.arx`; log chính `log\Build_All.log`, log target `log\Build_AutoCAD2022.log` trong thư mục target. Giữ bố trí theo quy ước hiện hành khi thực hiện. Không liên kết nhầm SDK/host. Build_Release dùng nguồn đã xác thực khi bàn giao, giữ kiểm tra build gốc và mọi điểm chặn runtime.

### INT-09. Cập nhật README, BuildInfo và dữ liệu QA

README của Add-in phải có: tên/phiên bản/baseline; ToolId và Feature → command/action; CAD target; Security Contract (Gate version, ABI, Policy, License format); KeyId được nhúng và nguồn duyệt và bằng chứng xác thực ZIP; các file tích hợp; điểm chặn; SDK/toolset/Windows SDK; cách chạy Build_All; output/log; cài đặt DProgram/DUser; hành vi ALLOW/DENY; cách xử lý thiếu file hoặc khóa không khớp; Known Issues và trạng thái Build/APPLOAD/Runtime riêng. Cập nhật BuildInfo, DataManifest, QA và lịch sử phiên bản theo STD-16/17 khi phù hợp. Không thêm README thứ hai vào ZIP Integration.

### INT-10. Ma trận kiểm thử bắt buộc

Xem mục 10. Mỗi ca có đầu vào/điều kiện, Expected, Actual, PASS/FAIL/NotVerified hoặc N/A có lý do, host và bằng chứng. DJoin V38 có `Buildtime/QA/VerifySEC06.cpp` và `MockGate.cpp` làm mẫu kiểm thử verifier; phải kiểm thử lại client chuẩn trên nguồn cuối cùng. Mock Gate chỉ trong môi trường QA cô lập, không phát hành hoặc tích hợp vào runtime thực tế.

### INT-11. Phát hành và quản lý thay đổi

SEC06 ký Gate không thay cho ký chính ARX. Nếu hợp đồng production yêu cầu ký ARX, phải ký/verify bằng chứng thư đã duyệt trước publish; chưa có hợp đồng hoặc chứng thư thì ghi NotVerified và không tuyên bố production. Đồng bộ version trong README, BuildInfo và chuỗi log/command. Không tự xây cơ chế ký mới.

Một Tool xuất một ZIP; một build chỉ dùng danh tính đã được phê duyệt. Khi Gate DLL được cập nhật với cùng KeyId và hợp đồng tương thích, bộ runtime mới vẫn cần nghiệm thu tương thích, không mặc định phải build lại mọi Add-in. Khi public key, ABI, Policy, manifest format hoặc target contract đổi, phải đánh giá ảnh hưởng, xuất bộ phù hợp, build và nghiệm thu lại các Tool liên quan; không tự đổi hợp đồng. Không tự đổi ID đã cấp. Không ghi đè dữ liệu người dùng. Ghi chứng cứ trên bản nguồn cuối cùng, không thay đổi ngoài phạm vi.

### INT-12. Chốt bàn giao và phân loại trạng thái

Build PASS ≠ APPLOAD PASS ≠ Runtime PASS. Chỉ phát hành bản nghiệm thu khi toàn bộ cổng áp dụng đã đạt theo STD-18; thiếu host phải ghi NotVerified, không giả lập thành PASS. Nếu người quản trị cho phép bàn giao bản nguồn để test, ghi rõ chưa nghiệm thu. Không ghi nhận production/OK vào hệ thống quản lý khi chưa được xác nhận. Integration.txt phải giải thích cách nghiệm thu cả Tool có một Feature và nhiều Feature.

## 9. Quy trình xuất ZIP tại TidiDcad (V55 — nguồn chờ nghiệm thu Windows)

1. Chọn Tool đã đăng ký → Xuất bộ tích hợp bản quyền. Đọc snapshot Tool/Feature đang hoạt động, xác nhận target và ánh xạ theo dữ liệu được duyệt. Hiển thị tên, mã, Feature, ABI, Policy, KeyId, sáu file.
2. Đọc public trust đã duyệt, kiểm tra trạng thái giao dịch, tính nhất quán; không cần xuất private key. Lấy API Header đúng phiên bản và client đã chuẩn hóa/kiểm thử, không dùng code do AI sinh tùy ý từng lần xuất.
3. Tạo ToolInfo và Integration.txt với thông tin thật, chép đúng các header/client; kiểm tra dữ liệu không thiếu, không placeholder, không có bí mật. Ký nội dung sáu file theo INT-06, xác minh chữ ký bằng danh tính SEC06 hiện hành; kiểm tra ZIP đúng sáu file, đọc lại nội dung, đối chiếu hash file nguồn/đích. Nếu đích đã tồn tại, yêu cầu xác nhận trước thay thế; không để ZIP dang dở khi lỗi.
4. Báo rõ xuất thành công hay lỗi cụ thể (Tool đã retire, Feature thiếu, KeyId chưa duyệt, phiên bản client không phù hợp, v.v.). Xuất ZIP PASS chỉ là nghiệm thu bộ dữ liệu tích hợp, không phải nghiệm thu Add-in đích.

Không tạo thêm schema database hoặc phần mềm quản lý khóa; tái sử dụng khóa SEC06 hiện có, tách miền chữ ký bộ bàn giao. Nếu cần mở rộng dữ liệu CAD target, phải duyệt riêng phương án; V1.0 có thể lấy từ thông tin target đã xác nhận khi xuất.

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
| QA04 | Public key hoặc KeyId bị sửa trong ZIP | Xác thực FAIL |
| QA05 | JSON/Header tự khớp nhưng ký bằng danh tính khác | Xác thực FAIL |
| QA06 | Feature sai cú pháp, sai Tool, không đúng ánh xạ | Từ chối tích hợp |
| QA07 | Build trực tiếp nguồn đã xác thực, không có file/script phê duyệt máy | Không phụ thuộc phê duyệt máy; giữ kiểm tra build gốc |
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

## 11A. Nghiệm thu bổ sung V1.2

| Mã | Điều kiện | Expected |
|---|---|---|
| CERT01 | Xuất hợp lệ với khóa SEC06 hiện hành | Sáu file, Certification hợp lệ, chữ ký đúng |
| CERT02 | Hủy/sai mật khẩu, khóa thiếu, pending hoặc snapshot đổi | Không công bố, bảo toàn ZIP cũ |
| CERT03 | Sửa bất kỳ entry, chữ ký, mã, thêm file/repack | Xác thực FAIL |
| CERT04 | Gói tự ký bằng danh tính khác | Xác thực FAIL |
| CERT05 | Kiểm tra ZIP đúng bằng public trust hiện hành, không PFX | PASS |
| CERT06 | Nhận ZIP khác với hash được xác nhận | Không tích hợp |
| CERT07 | Build nguồn đã xác thực không có file/script phê duyệt máy | Không cần cấu hình Windows bổ sung; giữ luồng build gốc |
| CERT08 | Build/publish lỗi sau một lần PASS | Không dùng log/artifact cũ, bản cũ còn nguyên |

Tiêu chí trên chưa phải kết quả thực thi Windows/CAD. V55 dùng khóa SEC06 hiện có; không thay Gate/Licensecc, khóa, database và runtime. Phê duyệt máy build V1.1 hết hiệu lực với bộ bàn giao V1.2. Không tự xóa file cũ của người dùng.

## 12. Căn cứ nguồn và giới hạn hiện tại

Đã kiểm tra trực tiếp ba tệp sau để lập đặc tả (SHA-256 giúp nhận diện nguồn, không phải chữ ký xác thực nguồn):

- `DJoin-V38 (1).zip`: `ae97bd564e9d8ff0f36daf3bc519b70d24ff8680e3b397686a3de4cbdc4b8043`. Tham chiếu Build_All.cmd; Build_Release.cmd; DJoin.vcxproj; Userdata/README.txt, QA-STATUS.txt; src/DJoin.cpp, SettingsNative.cpp, DJoinFeature.h, LicenseGate.*, GateReleaseVerifier.*, GateReleaseTrust.h; Buildtime/Check-ReleaseTrust.ps1, Sync-ReleaseTrust.ps1, ResolveC22SDK.ps1 và QA.
- `TidiDcad-System-V49.zip`: `2c805de957636add22d602778ad553c6ee69b16d7ed9d5890d7867605bb7ed5a`. Tham chiếu Source/ReleaseTrust/PortableSecurity.ps1 (public trust, KeyId, manifest) và Source/DcadLicenseGate/Source/DcadLicenseGate.h (API V6).
- `ToolCatalog.db`: `2100431221130d76bcb82af00e53c3505804aa328b104573bee6dcc8c2c08e5f`. Đã đọc schema tools/features và bản ghi Djoin.

V1.2 thay cơ chế phê duyệt máy build V1.1 bằng chữ ký bộ bàn giao, triển khai trong nguồn TidiDcad V55. Không dùng placeholder làm khóa thật. QA-STATUS V38 ghi MSVCBuild, APPLOAD, Runtime là NotVerified; không dùng nguồn này làm chứng cứ đã nghiệm thu. Windows signing, GUI, Build_All và CAD runtime của V55 cần nghiệm thu riêng. Việc cập nhật quy ước không thay thế kiểm chứng phần mềm.
