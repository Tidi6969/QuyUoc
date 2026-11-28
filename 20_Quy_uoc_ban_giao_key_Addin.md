# DCAD-STD-20 — Quy ước bàn giao Key và tích hợp bản quyền Add-in DCAD

**Mã quy ước:** 20  
**Phiên bản:** 1.12  
**Ngày cập nhật:** 28/11/2026  
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

**Plan — Tổng thể hướng phát triển thực tế TidiDcad (cập nhật 28/11/2026)**

Plan là một phần của STD-20, được quản lý trực tiếp tại đây; không tạo quy ước, file kế hoạch hoặc dự án riêng. Đây là nơi ghi yêu cầu và tiến độ phát triển chính thức của TidiDcad; README DCAD chỉ là thông tin tổng hợp, không thay thế Plan. Plan không thay đổi hợp đồng tích hợp đang hiệu lực khi chưa có thiết kế và nghiệm thu được duyệt.

**Baseline ban đầu: V55. Baseline duy trì hiện tại: V76 (kế thừa V74/V75; chủ dự án chọn bản OK mới nhất ngày 28/11/2026)** — V59 là bản đã được chủ dự án xác nhận build OK và kiểm nghiệm các ca chính chống sửa Add-in với Dtscr V121 / AutoCAD 2022 ngày 26/11/2026. V68 Setup được chủ dự án xác nhận OK; ngày 28/11/2026 chủ dự án xác nhận hạng mục 08 SMBIOS 2/3 OK. Duy trì 01–06 và 08; 07 và 09 tạm dừng phát triển, System Check tạm hoãn. Xác nhận của chủ dự án không thay thế log kiểm thử riêng từng ca/host; các ca thiếu chứng cứ vẫn NotVerified. ZIP V68 trên GitHub và phát hành production chưa xác minh. V70 là mốc khảo sát của bộ nghiên cứu bảo vệ binary, không phải baseline tích hợp hiện tại. V74 hợp nhất năm tab và điều chỉnh khởi tạo dữ liệu; xác nhận OK của chủ dự án không thay thế log nghiệm thu chi tiết. Phát triển tiếp hạng mục 10 trên V76 sau khi đủ điều kiện và được duyệt; không mở lại 07, 09 hoặc System Check.

| STT | Hạng mục | Yêu cầu | Tiến độ |
|---|---|---|---|
| 01 | Chữ ký số License (LIC-04) | Giữ xác minh chữ ký và từ chối License không hợp lệ | V55 — giữ nguyên |
| 02 | Khóa bản quyền theo máy (LIC-02) | Giữ DeviceID và đối chiếu máy tại Gate | V55 — giữ nguyên |
| 03 | Phân quyền Feature (LIC-03) | Giữ định danh, quyền và thời hạn Feature | V55 — giữ nguyên |
| 04 | Chống thay thế Gate (SEC-02) | Giữ xác minh chữ ký/hash Gate DLL theo SEC06 | V55 — giữ nguyên |
| 05 | Ký số bộ bàn giao (SEC-03) | Giữ RSA-PSS Integration sáu file; tiếp nhận trực tiếp từ chủ dự án, kiểm tra nội bộ, không yêu cầu xác nhận ngoài ZIP | V76 — nguồn kiểm thử; chữ ký giữ nguyên, Windows NotVerified |
| 06 | Chống sửa Add-in (SEC-AT) | Ký số nhúng Authenticode cho thành phẩm; xác minh tính toàn vẹn và danh tính phát hành DCAD; từ chối nghiệp vụ trong ranh giới bảo vệ đã duyệt | V59 — đã triển khai; chủ dự án xác nhận các ca chính trên Dtscr V121 / AutoCAD 2022 |
| 07 | Chống lùi đồng hồ (SEC-07) | Bảo vệ thời hạn License offline, phát hiện lùi thời gian và xử lý hợp lệ | **Tạm dừng phát triển** theo quyết định chủ dự án 26/11/2026; chưa triển khai code |
| 08 | Cải tiến xác thực máy — Machine Identity V2 (LIC-02) | SMBIOS UUID + System Serial Number + Mainboard Serial Number; tối thiểu 2 giá trị hợp lệ khớp với License đã ký | Đã triển khai từ V60/V61, kế thừa V68; chủ dự án xác nhận OK ngày 28/11/2026 (UserReportedWorking=YES). Log từng ca Windows/CAD còn thiếu: NotVerified |
| 09 | Feature-Bound Execution (DCAD-FBE) — chống bỏ qua Gate | Ràng buộc Feature với thành phần thiết yếu của nghiệp vụ, không chỉ ALLOW/DENY; nghiên cứu độc lập, kiểm soát tác động, duyệt từng bước | **Tạm dừng tại 09.3** theo chủ dự án 28/11/2026; 09.1–09.2 đã duyệt hướng nghiên cứu, chưa duyệt thiết kế 09.3, chưa code/tích hợp |
| 10 | Mã hóa chống dịch ngược (Binary Encryption + Code Virtualization) | Nghiên cứu một công cụ bảo vệ dùng chung cho Gate và Add-in, nghiệm thu độc lập trước tích hợp hai luồng ký V76; chi tiết tại [STD-20.1](20.1_Ma_hoa_chong_dich_nguoc_Tam_thoi.md) | Lab V3/Handover V1; chủ dự án báo PE-QA-V1 thử thành công (10.3.2); tiếp theo 10.3.3; native post-build, tích hợp và production NotVerified |

```text
V55 — giữ nguyên 01–05
  ↓
V59 — 06 đã triển khai, kiểm nghiệm các ca chính
  ↓
Duy trì 01–06; 07 tạm dừng phát triển
  ↓
08 — Machine Identity V2 (LIC-02): V61 → V68; chủ dự án xác nhận OK
  ↓
V68 — baseline duy trì; Setup được chủ dự án xác nhận OK
  ↓
09 — DCAD-FBE: tạm dừng tại 09.3; chưa code/tích hợp
  ↓
V74 — baseline duy trì đã được chủ dự án xác nhận OK
  ↓
10 — Lab V3/Handover V1; đối chiếu V70 → V74; nghiên cứu và nghiệm thu độc lập trước tích hợp
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
| 08.9 | QA nguồn cuối, fixture/parser lỗi, ma trận xác thực, từ chối V1, DB backup/rollback nếu có thay đổi, hồi quy 01–06; Windows build, APPLOAD/Runtime theo host và đo hiệu năng | V61 rà soát tổng thể, Linux QA PASS; chủ dự án xác nhận SMBIOS 2/3 OK trên hệ thống hiện tại V68. Log riêng từng ca/host, đo hiệu năng và hồi quy đầy đủ: NotVerified |
| 08.10 | Chủ dự án nghiệm thu thực tế, phát hành theo STD-17, cập nhật Plan/chứng cứ; chỉ sửa hợp đồng tích hợp nếu thực sự cần và được duyệt | Chủ dự án xác nhận 08 OK ngày 28/11/2026; đã ghi nhận vào Plan. ZIP/phát hành GitHub và chứng cứ chi tiết từng host: NotVerified |

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

**Rà soát tổng thể V61 — 26/11/2026:** Chủ dự án yêu cầu xác nhận lại toàn bộ Build, liên kết và logic phần mềm, sửa điểm lỗi. Baseline nguồn V60 đã xác minh CRC/SHA256; last working baseline V59 giữ nguyên. V60 có lỗi thiếu dấu đóng namespace dcad tại Machine.cpp (tái hiện GCC với khai báo Win32 giả lập: expected '}' at end of input); sửa đúng một dấu đóng, không đổi thuật toán nhận diện. V60 còn công bố TidiDcad/DCAD_ID trước nhóm License nên lỗi ở bước sau có thể trộn giao thức V1/V2. V61 stage cả 5 module, chỉ publish All sau toàn bộ QA và xác minh Gate đã ký, dùng chung SEC06 lock/journal/rollback engine hiện có, 16 đích + alias tùy điều kiện nằm trong allowlist phục hồi. Ngoại lệ bootstrap chỉ riêng TidiDcad trên root thực sự chưa có Add-in identity state và License suite, không publish DCAD_ID/License; root hiện hữu thiếu trust dừng an toàn. Đây là thay đổi workflow do ràng buộc giao thức V2, không đổi cơ chế mật mã SEC06. Sửa tài liệu/hướng dẫn active, tên CMake V8, nhãn V61, tái tạo đúng mảng instructions embedded. Dreq 6, envelope 4/5, MachinePolicy 2, Gate V8 ABI 2, Add-in Policy 1, Integration 1.2, nguyên tắc 2/3, License gốc, không fallback, khóa và dependency/schema/Feature/time không đổi. Manifest SEC06 Policy=1 giữ nguyên, khác MachinePolicyVersion. Không sửa CAD nghiệp vụ, không tiếp tục 07/obfuscation.

QA nguồn V61: GCC/Clang ASan/UBSan 38/38 parser + 50.000 biến đổi mỗi compiler; 529/529 policy/format mỗi compiler; mới 13/13 ca trên chính Machine.cpp với Win32/CryptoAPI API double mỗi compiler, đối chiếu hash, lỗi đọc/băm, giải phóng tài nguyên; Request/Import/IssueCommand PASS, AddinContract 27/27 và 50.000 biến đổi mỗi compiler; Integration model 60/60 GCC; Gate contract/mock Policy 2, Inspector validation, Layout 12.000 stress mỗi compiler, FeatureWindows regression PASS. RunLinux-V61.sh exit 0. Static link graph, 4 Integration embedded exact bytes + SEC06 GUI script equality, 602/602 dependency hashes/membership, publisher order và allowlist, diff PASS; 827 file không đổi, 32 file sửa/thêm trước manifest diff, không xóa file. Bổ sung QA Windows bad Gate, locked later destination rollback và publish All/final hash. PowerShell/MSVC thực tế, native RSA-PSS/CryptoAPI/Authenticode, transaction/rollback/bootstrap Windows, GUI, APPLOAD/Runtime/latency CAD vẫn NotVerified. Build root dùng QA identity hiện hữu, không production; first build thiếu cache Licensecc/Boost có thể cần tải nguồn. ZIP V61 90.731.721 byte SHA-256 `f9527b09b522a61b17e044fcd823b1366c79ee6afb825886ff8efa9868c7f882`; CRC và byte hashes PASS. Bàn giao nguồn qua ChatGPT; ZIP chưa upload GitHub. Không thay STD-20 hợp đồng kỹ thuật mục 2 trở đi. V61 thay V60 làm ứng viên kiểm thử, chưa nghiệm thu 08.10. Tiếp theo backup, Windows Build_All V61, mới dreq/lic, Integration 1.2, build/ký lại Add-in và test AutoCAD 2022. Không dùng V60 để build.

**Ghi nhận hiện hành — 28/11/2026 (thay thế trạng thái tiến độ cũ, giữ nguyên lịch sử):** Chủ dự án đã xác nhận Setup V68 OK và xác nhận “Cải tiến xác thực máy — SMBIOS 2/3 cái này oke rồi nhé”. Hạng mục 08 được chốt theo xác nhận chủ dự án (`UserReportedWorking=YES`); không suy ra PASS từng ca 2/3, từng CAD host, hiệu năng hoặc kiểm thử hồi quy nếu thiếu log. Chọn V68 (Setup phát triển từ V61) làm baseline duy trì; giữ V55/V59/V61 làm lịch sử. Chủ dự án yêu cầu tạm dừng nghiên cứu chống bỏ qua Gate, quay lại hệ thống gốc; 09 dừng tại 09.3, không xóa kế hoạch, chưa duyệt thiết kế/code/tích hợp. 07 tiếp tục tạm dừng, System Check tạm hoãn. Chỉ đồng bộ metadata Plan, không sửa hợp đồng kỹ thuật, code, khóa hay tạo phiên bản phần mềm mới. Chưa xác minh ZIP V68 trên GitHub hoặc production release.

**Hạng mục 09 — DCAD Feature-Bound Execution (FBE), 27/11/2026**

**Quyết định của chủ dự án:** chọn hướng nghiên cứu Feature-Bound Execution nhằm tăng khả năng chống bỏ qua Gate; làm từng bước, kiểm tra đạt và chủ dự án xác nhận mới chuyển bước/tích hợp. Trước triển khai phải làm rõ lưu trình, phạm vi sửa, phần giữ nguyên, phụ thuộc và tác động tới ba vai trò. Bổ sung vào Plan hiện có; không tạo quy ước hay repo mới. System Check được tạm hoãn, không thuộc 09. Không tiếp tục 07 hoặc làm rối từ khóa.

**Baseline và ranh giới:** V59 vẫn là baseline đã có xác nhận sử dụng; V61 là ứng viên 08 (Gate V8, ABI 2, MachinePolicyVersion 2, Dreq 6, License envelope V4/V5, Integration 1.2, Add-in Policy 1, SEC06 manifest Policy 1). Chưa có xác nhận nghiệm thu Windows/CAD V61. Nghiên cứu/prototype 09 được cách ly hoàn toàn; không sửa hay phát hành V61, không tăng version chỉ vì Plan. Trước tích hợp phải nghiệm thu 08, xác minh đúng baseline nguồn và hợp đồng áp dụng. Không lấy mô tả giao thức lịch sử ở mục 2 trở đi thay cho nguồn V61 thực tế.

**Mục tiêu và mô hình nguy cơ:** FBE tạo sự phụ thuộc thực sự giữa Feature được cấp quyền và dữ liệu/hàm/module thiết yếu của nghiệp vụ. Giữ Gate làm cơ chế xác thực chính, không thay bằng Session/Token. Mục tiêu kiểm chứng là bỏ qua kiểm tra ALLOW/DENY đơn thuần không đủ để hoàn thành nghiệp vụ được chọn; tài nguyên không cấp quyền, sai Feature, sai phiên bản, bị sửa hoặc thiếu phải bị từ chối an toàn. Không tạo điều kiện giả hoặc cố tình làm sai kết quả hình học. Giới hạn: toàn bộ hệ thống offline chạy trên máy khách do khách hàng kiểm soát, mã/khóa/dữ liệu đã giải mã có thể bị phân tích hoặc trích xuất; không tuyên bố chống mọi bypass, chống trích xuất tuyệt đối, hoặc tương đương hệ thống thương mại có ranh giới giữ khóa độc lập. Không ước lượng tỷ lệ tăng bảo mật khi chưa có bằng chứng.

**Phạm vi nghiên cứu:** native C++ x64, offline, ưu tiên không thêm DLL/EXE runtime, không USB dongle/TPM/server/driver/obfuscation. Ưu tiên khảo sát Feature-Bound Data (dữ liệu thực sự thiết yếu, không công khai hay có bản plaintext dự phòng), sau đó Feature-Bound Function/Module nếu dữ liệu không phù hợp. Không đưa toàn bộ thuật toán CAD vào Gate trung tâm. Không mã hóa tùy tiện database được người dùng chỉnh sửa. Session chỉ là cơ chế quản lý quyền/vòng đời, không tự tạo ranh giới bảo mật; DeviceID, hash công khai và chữ ký License không phải bí mật để dùng trực tiếp làm khóa giải mã. DPAPI nếu khảo sát phải phân biệt phạm vi bảo vệ khi lưu trữ và khi chạy; không phụ thuộc tài khoản Windows hoặc thay chính sách mã máy hiện tại nếu chưa duyệt. Phương án khóa, thuật toán/định dạng, API và cách lưu tài nguyên chưa được chọn. Chỉ dùng primitive mật mã chuẩn và thư viện đã thẩm định; ưu tiên dùng lại dependency nếu phù hợp. Tách khóa ký và khóa giải mã; đánh giá phạm vi thiệt hại khi một khóa bị lộ, phân tách theo Feature/resource nếu khả thi; không đưa khóa riêng issuer sang máy build/khách hàng. Cần quyết định việc đóng gói, cấp phát/khôi phục/luân chuyển khóa, cấp lại License, tương thích cũ và quyền truy cập trước khi code tích hợp.

**Lưu trình hiện tại phải bảo toàn (tham chiếu ứng viên V61):**
- Quản trị: đăng ký Tool/Feature → xuất và ký Integration → tiếp nhận thành phẩm build → ký Authenticode và xác minh → phát hành; DCAD_ID tạo Dreq từ máy sử dụng → issuer cấp License đã ký theo Feature/Edition/thời hạn. Giữ nguyên quyền sở hữu khóa và thẩm quyền cấp phép.
- Phát triển: nhận Integration đã xác thực → ánh xạ Feature vào entry point nghiệp vụ → code/build đúng SDK → QA → bàn giao thành phẩm và bằng chứng. Không cần khóa riêng, tài khoản issuer hoặc License thật để biên dịch.
- Sử dụng: nhận Add-in đã ký, Gate + release manifest, License và tài nguyên được duyệt → APPLOAD → mỗi chức năng được bảo vệ gọi GateClient/Gate để kiểm tra đầy đủ → chỉ thực hiện khi hợp lệ.
- Hướng FBE dự kiến bổ sung bước bảo vệ tài nguyên cần thiết tại giai đoạn phát hành, trước ký Authenticode nếu tài nguyên được nhúng hoặc làm thay đổi PE; nếu tài nguyên rời phải xác thực và ràng buộc với Feature/version phù hợp. Runtime phải xác minh theo luồng hiện có trước khi cấp quyền sử dụng tài nguyên, sau đó nghiệp vụ mới được thực hiện. Không phát hành lại hay ký lại sau khi thay đổi nội dung đã ký mà chưa qua các cổng xác minh. Không tạo nhánh fallback plaintext hoặc quyền ALLOW cũ để vượt FBE đối với Feature đã áp dụng FBE.

**Ma trận ảnh hưởng sơ bộ — không phải danh sách file đã duyệt:**

| Thành phần | Định hướng | Điều kiện trước khi thay đổi |
|---|---|---|
| LIC-04, RSA-PSS, issuer/SEC06/Authenti­code signing identity | Giữ nguyên thuật toán, khóa và quy trình xác minh | Không dùng khóa ký làm khóa giải mã; kiểm thử hồi quy 01–06, 08 |
| LIC-02, SMBIOS 2/3, DCAD_ID/Dreq 6 | Giữ nhận diện, ngưỡng và đối chiếu License gốc; ưu tiên không sửa DCAD_ID/Dreq | Nếu cần enrollment khóa mới, phải trình tác động thiết bị, cấp phép và khả năng thay phần cứng trước |
| Tool/Feature/Edition/time | Giữ ID, mapping, phân quyền và thời hạn | Không mở rộng hoặc cấp chéo quyền; không khởi động lại 07 |
| License envelope V4/V5, Licensecc, LicenseccManager, DB, Inspector | Ưu tiên giữ định dạng, schema, thư viện; có thể bổ sung dữ liệu FBE nếu thật sự cần | Chốt tính tương thích, dữ liệu phải được xác thực/ràng buộc Feature, backup/rollback, cấp lại nếu cần; không tự đổi version |
| Gate V8, GateClient, ABI 2, MachinePolicy 2 | Gate/Client có khả năng tác động trực tiếp ở API FBE; giữ toàn bộ kiểm tra hiện hữu | Chốt API/lifetime/ownership, versioning, old/new compatibility, fail-closed, đo hiệu năng; không giả giữ ABI nếu không tương thích |
| Integration sáu file 1.2 | Ưu tiên giữ cấu trúc, quy trình ký và độc lập máy build; hợp đồng có thể phải mở rộng | Đánh giá tất cả file, chữ ký, pin, QA và nhu cầu build/ký lại; chỉ đổi hợp đồng sau khi được duyệt |
| TidiDcad và phát hành | Có thể thêm bảo vệ tài nguyên và quản lý khóa FBE | Duyệt quyền quản trị/phát triển/sử dụng, lưu trữ bí mật, lifecycle, xử lý khi phát hành dở |
| Build_All/Build_Release, ký số, packaging | Có thể thêm bước bảo vệ tài nguyên | Không làm mất khả năng build chuẩn; bảo vệ tài nguyên trước ký PE; kiểm tra đồng bộ, backup, rollback, không ghi đè bản đang dùng nếu FAIL |
| Add-in CAD | Chỉ thay điểm truy cập thành phần nghiệp vụ thiết yếu, không đổi thuật toán, UI, command, XData, settings hay target ngoài phạm vi | Chọn một Tool/Feature pilot có tài nguyên thực sự cần thiết; rà tất cả đường thực thi và chứng minh hình học trước/sau tương đương |
| Runtime/files | Ưu tiên giữ cấu trúc thư mục, số lượng module và offline | Chỉ thêm tài nguyên nếu cần và được duyệt, không private key, dữ liệu plaintext dự phòng, debug/QA hoặc dữ liệu quản trị ở khách hàng |
| Add-in/License hiện hữu | Chưa xác định khả năng tương thích | Lập ma trận phiên bản và kế hoạch triển khai, có cần cấp lại License/xuất Integration/build/ký lại không; không trộn phiên bản hoặc fallback cho Feature FBE |
| 07, System Check, mã hóa riêng Gate, obfuscation | Ngoài phạm vi 09 | Không triển khai khi chưa được giao riêng |

**Phân quyền và bàn giao dự kiến:** Quản trị giữ khóa ký riêng và quản lý khóa FBE/đóng gói nếu thiết kế cần; duyệt Tool/Feature, phát hành tài nguyên đã bảo vệ, ký thành phẩm và cấp License. Phát triển có mã nguồn và tài nguyên nghiệp vụ trong phạm vi được giao, Integration công khai đã xác thực, dữ liệu test/khóa QA cô lập và API để build/QA; không khóa riêng phát hành, không tự cấp License thật, không yêu cầu bí mật production lúc compile. Sử dụng chỉ nhận artifact đã ký, Gate/manifest, License của mình và tài nguyên bảo vệ đã duyệt; không issuer private key, source hoặc DB quản trị. Chốt cách bàn giao khóa/tài nguyên cụ thể ở 09.3–09.4, không mở rộng quyền mặc định.

**Lộ trình từng bước, xác nhận bắt buộc:**

| Bước | Công việc / đầu ra bắt buộc | Điều kiện đạt | Trạng thái |
|---|---|---|---|
| 09.1 | Mục tiêu, mô hình nguy cơ, sơ đồ luồng hiện tại và ma trận ảnh hưởng sơ bộ | Chủ dự án duyệt phạm vi, giới hạn, phần không được thay đổi | Đã xác nhận 27/11/2026: chủ dự án đồng ý đầy đủ 12 điều kiện và giao thực hiện 09.2 |
| 09.2 | Nghiên cứu tài liệu chính thức và mã nguồn tham khảo; rà nguồn baseline ở phạm vi liên quan, so sánh Data/Function/Module và giữ Gate hiện tại | Bảng phương án có căn cứ, phân biệt nguyên lý thương mại với khả năng offline DCAD; chọn ứng viên hoặc dừng | Đã duyệt 27/11/2026: ưu tiên FBE Data, tận dụng thư viện mật mã hiện có nếu phù hợp; giao thực hiện 09.3 |
| 09.3 | Thiết kế quản lý khóa: vị trí, phạm vi, cấp phát, lưu trữ, sử dụng, luân chuyển, khôi phục, rủi ro lộ khóa, phân quyền | Có phương án khả thi, không nhầm chữ ký/hash/DeviceID với bí mật; đánh giá rõ giới hạn runtime; nếu không đạt thì dừng | Tạm dừng theo yêu cầu chủ dự án 28/11/2026; giữ lịch sử khảo sát, chưa duyệt 09.3, chưa kiểm thử |
| 09.4 | Đặc tả Feature–Session–Resource, định dạng/API/version, vòng đời, trình tự build/sign/publish/runtime, quyền ba vai trò, danh sách file/module dự kiến sửa/giữ, ma trận tương thích và QA | Chủ dự án duyệt thiết kế, danh sách thay đổi và kế hoạch backup/rollback trước khi code | Chưa duyệt |
| 09.5 | Prototype C++ độc lập trong môi trường QA, không dùng khóa thật hoặc sửa sản phẩm | Build, test hợp lệ/không hợp lệ, bằng chứng trên nguồn cuối | Chưa code |
| 09.6 | So sánh prototype với Gate baseline: kiểm thử quyền, tính toàn vẹn, phụ thuộc thực sự, giới hạn lưu khóa/runtime, hiệu năng | Chứng minh lợi ích bổ sung trong mô hình nguy cơ đã duyệt; không chỉ thêm một kiểm tra boolean; nếu không đủ lợi ích thì dừng | Chưa thử |
| 09.7 | Chọn một Add-in pilot, rà source, xác định đúng Feature/resource thực sự thiết yếu; tích hợp trong bản thử riêng | Diff đúng phạm vi, không ảnh hưởng Add-in khác, bảo toàn nghiệp vụ và dữ liệu | Chưa chọn Tool |
| 09.8 | Build/APPLOAD/Runtime trên AutoCAD 2022 x64, đối chiếu hình học, đo thời gian, QA bảo mật, các trường hợp gián đoạn | Kết quả thực tế PASS theo tiêu chí được duyệt, chủ dự án nghiệm thu pilot; không suy rộng host khác | Chưa thử |
| 09.9 | Sau nghiệm thu 08 và pilot: chốt lại impact/diff/version/compatibility; tích hợp TidiDcad, Gate/Integration/issuer/build theo thiết kế đã duyệt | Các cổng 01–06 và 08 hồi quy; ký, phát hành đồng bộ, upgrade/rollback; không phát hành khi FAIL | Chưa triển khai |
| 09.10 | QA nguồn cuối, tài liệu, nghiệm thu và cập nhật Plan/hợp đồng đúng phạm vi đã duyệt | Chủ dự án xác nhận thực tế; version theo STD-17; bằng chứng Build/Load/Runtime riêng | Chưa nghiệm thu |

**QA tối thiểu phải thiết kế trước prototype/tích hợp:** hợp lệ và thiếu License, sai Feature/Edition, hết hạn, sai máy, Gate/Add-in bị sửa, quyền/Session không hợp lệ hoặc hết vòng đời, tài nguyên thiếu/sai Feature/sai version/bị sửa, lỗi giải mã/I/O, không có bản plaintext dự phòng, không tạo kết quả nghiệp vụ sai khi từ chối, không cho phép bỏ qua một kiểm tra đơn thuần mà vẫn đủ tài nguyên thiết yếu trong kịch bản kiểm thử đã duyệt. Đánh giá rủi ro trích xuất và phạm vi ảnh hưởng khi lộ khóa, không coi việc dữ liệu không đọc được lúc lưu là chống mọi can thiệp runtime. Kiểm thử tương thích cũ/mới, lỗi build/ký/publish dở, backup/rollback, hồi quy đầy đủ bảo mật và kết quả CAD. Chốt ngưỡng hiệu năng dựa trên baseline đo được trước khi thử. Mỗi ca ghi expected/actual/host/bằng chứng/PASS/FAIL/NotVerified theo STD-18 AI-14–AI-18; không lấy mock/Linux thay Windows/CAD. FAIL bắt buộc hoặc NotVerified trọng yếu thì không nghiệm thu, không tích hợp sản xuất. Không được giảm các lớp bảo vệ hiện hữu để đạt FBE.

**Nguồn tham khảo khởi điểm (cần đối chiếu sâu ở 09.2):** [Thales Sentinel Licensing API workflow](https://docs.sentinel.thalesgroup.com/softwareandservices/LDK/LDKdocs/SPNL/LDK_SLnP_Guide/Protection/Licensing_API_protection/Workflow.htm), [Sentinel decrypt](https://docs.sentinel.thalesgroup.com/ldk/LDKdocs/API-licensing/Licensing_API/hasp_decrypt.htm), [Wibu AxProtector](https://www.wibu.systems/protection/axprotector/), [Microsoft DPAPI](https://learn.microsoft.com/en-us/windows/win32/api/dpapi/nf-dpapi-cryptprotectdata), [Saltzer & Schroeder](https://web.mit.edu/Saltzer/www/publications/protection/) — nguyên tắc complete mediation, fail-safe defaults và economy of mechanism. Các liên kết là tài liệu để khảo sát, không chọn SDK hay khẳng định tương đương mức bảo mật. 09.2 cần ghi rõ nguồn thực sự đọc, license mã nguồn tham khảo nếu có và đánh đổi kỹ thuật.

**Trạng thái 27/11/2026:** Chủ dự án yêu cầu kiểm soát tác động và bổ sung FBE vào Plan GitHub, thực hiện từng bước và chỉ tích hợp sau xác nhận. Đã tổng hợp phạm vi và ma trận ảnh hưởng cấp kiến trúc; chưa đọc lại ZIP V61 trong lượt này, chưa chọn khóa/API/định dạng/resource/pilot, chưa quyết định tương thích, chưa code/build/test 09, chưa tăng version, chưa thay hợp đồng kỹ thuật STD-20. Bước tiếp theo: chốt 09.1, sau đó 09.2; có thể nghiên cứu độc lập trong khi 08 chờ nghiệm thu, không thay đổi sản phẩm đang dùng.

**Nghiệm thu phạm vi 09.1 — 27/11/2026:** Chủ dự án xác nhận đầy đủ 12 điều kiện: (1) offline; (2) không TPM/USB dongle/server/driver; (3) không đổi thuật toán và khóa ký hiện tại; (4) giữ SMBIOS 2/3; (5) giữ Tool ID, Feature ID, Edition; (6) không giảm lớp bảo mật hiện hữu; (7) không sai nghiệp vụ CAD; (8) không ảnh hưởng đáng kể tốc độ CAD; (9) ưu tiên không tăng DLL/EXE runtime; (10) developer không giữ khóa riêng production; (11) phát triển/kiểm thử độc lập, chưa sửa V61; (12) chứng minh hiệu quả trước tích hợp. Chủ dự án giao cập nhật Plan và thực hiện 09.2. Trạng thái 09.1: Approved; đây là duyệt yêu cầu, không phải PASS phần mềm.

**Kết quả khảo sát 09.2 — 27/11/2026 (chờ duyệt):**

- Đã mở ZIP TidiDcad-System-V61.zip trong nguồn bàn giao, 90.731.721 byte, CRC toàn archive PASS; SHA-256 f9527b09b522a61b17e044fcd823b1366c79ee6afb825886ff8efa9868c7f882 khớp Plan. V59 vẫn là baseline được người dùng xác nhận; V61 là ứng viên chưa nghiệm thu Windows/CAD. Không sửa ZIP.
- Đã đọc trực tiếp các thành phần V61 liên quan: Source/DcadLicenseGate/Source/DcadLicenseGate.h/.cpp, CMakeLists.txt; Source/TidiDcad/Integration/DcadGateClient.cpp/.h, Integration.txt; Source/LicenseccManager/Source/DcadLicenseSDK.cpp/.h, DcadSeal.cpp/.h, IssueCommand.h; Source/DcadLicenseRequest/Source/RequestFormat.h; Source/TidiDcad/Source/IntegrationExport.cpp, IntegrationModel.h, ReleaseSecurity.cpp và Build_All.cmd (đối chiếu điểm nối liên quan, không tuyên bố audit toàn bộ mã). Gate V8 ABI 2, Machine Policy 2, Add-in Policy 1; DcadGate_CheckSignedAddinW xác minh caller/module/Feature rồi gọi kiểm tra License; CheckLicenseDetailsW dùng SDK VerifyLicenseDetails, VerifyEnvelope kiểm tra V4/V5. GateClient kiểm tra manifest SEC06 và policy trước gọi Gate. API hiện trả kết quả cấp quyền/thông tin, chưa cung cấp resource nghiệp vụ. Dreq 6 hiện là yêu cầu chứa ID/fingerprint SMBIOS, không chứa public key giải mã riêng. Integration 1.2 xuất metadata và mã tích hợp; không mặc định tương thích với FBE. Gate CMake đã link bcrypt, crypt32, advapi32, LIEF static và Licensecc; ưu tiên khảo sát tận dụng bcrypt/crypt32 hiện hữu, không tự thêm dependency runtime.
- Tài liệu đã đối chiếu: [Thales Sentinel Decrypt API](https://docs.sentinel.thalesgroup.com/ldk/LDKdocs/API-licensing/Licensing_API/hasp_decrypt.htm) ràng buộc phiên/Feature với giải mã, khóa giữ trong protection key; [Thales attack/defense](https://docs.sentinel.thalesgroup.com/softwareandservices/ldk/LDKdocs/SPNL/LDK_SLnP_Guide/Protection/Protection_Strategies/TypesofAttack_Defense.htm) bảo vệ data/code, kênh bảo mật HL có điều kiện phần cứng; [Wibu CodeMeter Protection Suite](https://www.wibu.com.cn/magazine/keynote-articles/article/detail/codemeter-protection-suite-the-multitool-for-software-protection.html) và [CodeMeter Security](https://www.wibu.com.cn/cn/%E8%B5%84%E6%BA%90/%E5%A8%81%E6%AD%A5%E6%9D%82%E5%BF%97/keynote-articles/keynote/detail/codemeter-security.html) có container license phần mềm/phần cứng/cloud, mã hóa theo license và quản lý key; không sao chép SDK hay coi DCAD có cùng trust boundary.
- Mật mã tham khảo: [Microsoft BCryptEncrypt](https://learn.microsoft.com/en-us/windows/win32/api/bcrypt/nf-bcrypt-bcryptencrypt), [BCRYPT_AUTHENTICATED_CIPHER_MODE_INFO](https://learn.microsoft.com/en-us/windows/win32/api/bcrypt/ns-bcrypt-bcrypt_authenticated_cipher_mode_info) hỗ trợ mã hóa xác thực; [CryptProtectData](https://learn.microsoft.com/en-us/windows/win32/api/dpapi/nf-dpapi-cryptprotectdata) phạm vi máy có thể được mọi user trên máy giải mã, phạm vi user không phù hợp yêu cầu đa tài khoản; [Windows Key Storage and Retrieval](https://learn.microsoft.com/en-us/windows/win32/seccng/key-storage-and-retrieval) phần mềm KSP dùng vùng lưu khóa OS, không mặc định phù hợp quy ước folder/portable, không chọn NCrypt/KSP mới. Không thay hệ thống khóa ký hiện có.
- Repository tham khảo: [open-license-manager/licensecc](https://github.com/open-license-manager/licensecc) BSD-3-Clause, giữ thư viện đang dùng; [jedisct1/libsodium](https://github.com/jedisct1/libsodium) ISC, [AEAD](https://doc.libsodium.org/secret-key_cryptography/aead) và [sealed boxes](https://doc.libsodium.org/public-key_cryptography/sealed_boxes), [crypto_box_seal.c](https://github.com/jedisct1/libsodium/blob/master/src/libsodium/crypto_box/crypto_box_seal.c) để đối chiếu nguyên lý; [openssl/openssl](https://github.com/openssl/openssl) Apache-2.0 (nhánh hiện hành), đối chiếu primitive chuẩn. Đã khảo sát tài liệu/repo, chưa audit thư viện hoặc chọn commit/version/dependency mới. Sealed box không tự xác thực danh tính sender; vẫn phải xác minh quyền cấp phép bằng chữ ký hiện hữu. Chưa tìm được bằng chứng một thư viện open-source duy nhất cung cấp toàn bộ FBE với ranh giới an toàn độc lập đáp ứng mọi điều kiện DCAD.
- So sánh: giữ Gate hiện tại ít thay đổi nhất nhưng không tạo phụ thuộc nghiệp vụ; Session-only không đạt mục tiêu; FBE Data có phạm vi thay đổi nhỏ nhất nếu có tài nguyên thực sự cần và không có bản thay thế dễ dàng; Function/Module thay đổi code, loader, ABI/đóng gói và QA ObjectARX nhiều hơn; không triển khai mã hóa code/loader tùy biến ở giai đoạn này. Không mặc định mọi Add-in có resource phù hợp.
- Ứng viên ưu tiên đề xuất cho 09.3 (chưa duyệt): FBE Data với tài nguyên mã hóa có xác thực, khóa ngẫu nhiên tách theo Feature/resource, quyền sử dụng khóa do issuer cấp cho Feature hợp lệ và không có khóa giải mã chung nhúng ở Add-in/Gate. Khảo sát phương án cấp khóa qua License đã ký so với gói khóa được bọc cho public key của máy nhận; phương án sau có thể tăng bảo vệ khi sao chép dữ liệu ở trạng thái lưu trữ nhưng cần enrollment key riêng, khả năng thay đổi DCAD_ID/Dreq/issuer/License, khôi phục khi mất key/cài lại Windows và phân quyền đa tài khoản. Không lấy DeviceID/hash/chữ ký làm bí mật. Khóa trong License đọc được trực tiếp có lợi ích giới hạn; DPAPI LOCAL_MACHINE chỉ là bảo vệ lưu trữ, không phải ranh giới chống admin. Nếu khóa hoặc plaintext có thể bị trích xuất khi người dùng hợp lệ chạy Feature, có thể bị sao chép/chia sẻ; không cam kết ngăn mọi bypass. Mọi lựa chọn thuật toán, lưu khóa, định dạng và lifecycle chỉ chốt tại 09.3–09.4 sau duyệt, không mặc định cần đổi dreq/lic.
- Ảnh hưởng dự kiến cập nhật: Gate/Client/API resource, Integration metadata/export, bước bảo vệ resource trước ký PE, điểm sử dụng resource ở Add-in và QA bắt buộc nếu tiếp tục; DCAD_ID/RequestFormat, RequestImport, issuer/DcadSeal/SDK, Inspector, DB và phát hành đồng bộ chỉ sửa nếu mô hình cấp khóa được duyệt thực sự cần. Giữ Machine 2/3, Tool/Feature/Edition/time, Licensecc, RSA-PSS, SEC06, Authenticode, hình học/UI/settings/XData và cấu trúc thư mục; không tự thay đổi hợp đồng hoặc buộc thêm DLL runtime. Chưa xác định resource/pilot hay danh sách diff chính thức.
- Cổng quyết định: 09.2 khảo sát có căn cứ, đề xuất Data trước Function/Module, nhưng lợi ích còn phụ thuộc vào quản lý khóa và resource thật; nếu không có phương án tăng bảo mật đủ giá trị mà vẫn đáp ứng 12 yêu cầu thì dừng FBE, giữ Gate. Phải có kết quả prototype so sánh tại 09.6, không đánh PASS bảo mật theo nghiên cứu. Chưa code/build/runtime/test xâm nhập FBE; chưa lựa chọn khóa/API/format/thuật toán cuối; chưa nghiệm thu V61. Chờ chủ dự án duyệt 09.2 mới bắt đầu 09.3; không sửa V61 hay mục 2 trở đi của STD-20.

**Xác nhận 09.2 — 27/11/2026:** Chủ dự án đồng ý kết quả nghiên cứu, ưu tiên FBE Data và tận dụng thư viện mật mã hiện có nếu phù hợp; giao cập nhật Plan và thực hiện 09.3. Không cho phép code, tích hợp hoặc sửa V61. 09.2: Approved (duyệt hướng nghiên cứu, không phải nghiệm thu phần mềm).

**Kết quả nghiên cứu và thiết kế quản lý khóa 09.3 — 27/11/2026 (đề xuất, chờ duyệt):**

**Bằng chứng nguồn:** Đã đọc lại ZIP TidiDcad-System-V61.zip, SHA-256 f9527b09b522a61b17e044fcd823b1366c79ee6afb825886ff8efa9868c7f882, CRC toàn ZIP PASS. Đối chiếu Source/DcadLicenseRequest/Source/RequestFormat.h, Source/LicenseccManager/Source/{DcadSeal.h,DcadSeal.cpp,DcadLicenseSDK.h,DcadLicenseSDK.cpp,IssueCommand.h,RequestImport.h}, Source/DcadLicenseGate/Source/{DcadLicenseGate.h,DcadLicenseGate.cpp}, Source/DcadLicenseGate/CMakeLists.txt và Source/TidiDcad/Source/IntegrationModel.h. Dreq 6 là đúng 5 dòng ASCII/LF, giới hạn 1024 byte, chỉ requestId/machineId/policy marker/checksum; checksum SHA-256 không xác thực người gửi, không có recipient public key. DcadSeal V4/V5 là envelope canonical có chữ ký, chứa Licensecc payload, SMBIOS tuple, thông tin thời hạn; API hiện không mang key grant FBE. SDK VerifyLicenseDetails đọc/xác minh envelope, kiểm tra máy và quyền/thời hạn. Gate V8 ABI 2 có CheckSignedAddinW và CheckLicenseDetailsW nhưng không có API resource. CMake Gate đã link bcrypt/crypt32/advapi32; DcadSeal đã dùng BCrypt cho RSA-PSS. IntegrationModel tạo metadata 1.2. Không suy luận rằng định dạng hiện hữu cho phép tự thêm trường. Chưa audit toàn hệ thống hoặc chạy Windows/CAD; V59 giữ baseline đã xác nhận, V61 ứng viên 08.

**Nguồn kỹ thuật đã đối chiếu:** [NIST SP 800-57 Part 1 Rev.5](https://csrc.nist.gov/pubs/sp/800/57/pt1/r5/final), [Part 2 Rev.1](https://csrc.nist.gov/pubs/sp/800/57/pt2/r1/final) về phân mục đích, vòng đời và xử lý lộ khóa; [NIST SP 800-38D](https://csrc.nist.gov/pubs/sp/800/38/d/final) về AEAD GCM, yêu cầu IV duy nhất theo khóa; [RFC 8017](https://www.rfc-editor.org/rfc/rfc8017.html) về RSAES-OAEP; [Microsoft BCryptEncrypt](https://learn.microsoft.com/en-us/windows/win32/api/bcrypt/nf-bcrypt-bcryptencrypt), [BCryptGenRandom](https://learn.microsoft.com/en-us/windows/win32/api/bcrypt/nf-bcrypt-bcryptgenrandom), [BCryptGenerateKeyPair](https://learn.microsoft.com/en-us/windows/win32/api/bcrypt/nf-bcrypt-bcryptgeneratekeypair), [BCryptExportKey](https://learn.microsoft.com/en-us/windows/win32/api/bcrypt/nf-bcrypt-bcryptexportkey), [CryptProtectData](https://learn.microsoft.com/en-us/windows/win32/api/dpapi/nf-dpapi-cryptprotectdata). Microsoft xác nhận DPAPI LOCAL_MACHINE cho phép các user trên cùng máy giải mã nếu có blob; chỉ bảo vệ lưu trữ, không bảo đảm chống admin/runtime. BCrypt thuộc bộ primitive CNG đã dùng trong V61, không đề xuất hệ thống NCrypt/KSP mới. NIST/Windows/RFC không xác nhận mức bảo mật của DCAD-FBE.

**So sánh phương án:** khóa chung nhúng Gate/ARX: không chọn, bí mật chung nằm trong sản phẩm; DeviceID/hash/chữ ký làm khóa: loại, không phải bí mật; DEK plaintext trong License đã ký: chữ ký không che giấu, chỉ tạo phụ thuộc khi khách chưa có key và dễ chia sẻ, không chọn production; DPAPI machine-scope: chỉ bổ trợ lưu trữ, không giải quyết phân phối khóa; recipient public/private key riêng theo cài đặt + DEK ngẫu nhiên theo resource: ứng viên ưu tiên có điều kiện, cho phép issuer phân phối key được bọc offline đến đúng recipient nhưng thêm enrollment, dữ liệu lưu trữ và cấp lại; encrypt riêng toàn bộ resource cho từng khách: chi phí đóng gói/phát hành lớn, không ngăn chia sẻ plaintext, không ưu tiên; TPM/dongle/server/driver: ngoài phạm vi. Không mặc định cần hệ thống khóa phức tạp nếu lợi ích không đạt.

**Kiến trúc khóa đề xuất:** giữ nguyên tất cả khóa ký License/SEC06/Authenticode và Machine 2/3; không tái sử dụng chúng làm khóa giải mã. Quản trị sinh DEK ngẫu nhiên riêng cho (ToolId, FeatureId, ResourceId, resource revision/key generation) ở thời điểm bảo vệ tài nguyên; cùng một resource revision dùng cùng ciphertext/DEK cho các recipient được cấp (đây không phải DEK riêng từng khách), khóa khác nhau giữa resource/Feature và revision mới; không tạo master key giải mã toàn bộ ở Gate/Add-in. Chỉ quản trị/issuer được quản lý DEK production. Đề xuất AES-256-GCM, nonce 96 bit duy nhất theo key, tag 128 bit, CSPRNG BCryptGenRandom; giữ nội dung định danh và version trong metadata xác thực, ràng buộc hash tài nguyên mã hóa vào dữ liệu phát hành/cấp quyền có chữ ký tin cậy, không dựa riêng vào AEAD vì recipient hợp lệ biết DEK. Resource revision độc lập version ARX để không buộc cấp lại quyền FBE khi build lại mà resource không đổi. Khóa nhận: một cặp RSA 3072 bit riêng cho mỗi cài đặt được đăng ký, sinh cục bộ qua BCrypt, RSA-OAEP SHA-256 chỉ dùng bọc DEK. Key ID là fingerprint SHA-256 của public key canonical, không phải bí mật và không thay DeviceID. Định dạng chuẩn xác, AAD, giới hạn độ dài/số resource, schema và test vector sẽ chốt tại 09.4, không tự sáng chế primitive mật mã.

**Lưu trữ và phân quyền:** khách hàng sinh recipient key riêng; chỉ gửi public key/key ID, không gửi private key tới issuer. Private key này không phải issuer private key và không cấp được License. Đề xuất lưu blob recipient private key bảo vệ bởi DPAPI LOCAL_MACHINE trong DUser dùng chung đúng STD-03, quyền file tối thiểu phù hợp nhiều tài khoản Windows, không ghi plaintext/log, không ghi ProgramData hoặc tạo lưu khóa NCrypt/KSP riêng; tên file, ACL và đường dẫn chốt ở 09.4. DPAPI dùng trạng thái bảo mật của Windows, không thể bảo đảm portable chỉ bằng sao chép DUser, không gắn trực tiếp SMBIOS và không tuyên bố non-exportable. Quản trị lưu DEK theo danh mục resource trong vùng DData của issuer với kiểm soát truy cập và mã hóa lưu trữ; đề xuất DPAPI machine-scope kết hợp backup mã hóa offline do quản trị giữ khóa phục hồi độc lập. DPAPI đơn thuần không thay backup có thể phục hồi trên môi trường quản trị khác; cơ chế backup/khôi phục, quyền truy cập và kiểm thử phải được duyệt ở 09.4 trước production, không tự thêm master key runtime. Không đưa secret production vào GitHub, build log, QA, Integration hoặc máy developer. Developer dùng key/resource QA độc lập; có thể build không cần issuer hoặc khóa production.

**Lưu trình dự kiến (chỉ thiết kế):** (1) Quản trị chọn resource nghiệp vụ thật, tạo DEK, bảo vệ resource; ký PE sau mọi thay đổi PE, ký/ràng buộc resource rời nếu có. (2) DCAD_ID sinh/đọc recipient key của cài đặt, tạo request mang Machine 2/3 như cũ và recipient public key/key ID. Dreq 6 hiện không mang khóa và checksum không xác thực nguồn, nên cần request có phiên bản mới hoặc gói đăng ký bổ sung; đề xuất issuer xác minh public key fingerprint với khách đã xác thực qua kênh liên lạc tin cậy hiện có trước khi cấp. Proof-of-possession nếu cần chỉ chứng minh có private key, không chứng minh phần cứng/OS đáng tin cậy. Không tự chấp nhận thay public key hoặc coi checksum là xác thực. (3) Issuer kiểm tra quyền Feature/Edition/time và machine, bọc đúng DEK cho recipient public key bằng RSA-OAEP, ký cả grant có định danh recipient, Tool/Feature/Resource/revision, hash resource và liên kết chính xác License hiện có. Đề xuất production giữ một Dcad.lic nếu khả thi qua version envelope có chữ ký mới; không append tùy tiện V4/V5; prototype dùng grant QA độc lập, không sửa giao thức thực. (4) Khách nhận Dcad.lic và tài nguyên đã bảo vệ; Gate giữ mọi xác minh SEC06/SEC-AT/License/Machine/Feature/time, xác minh binding grant/recipient/resource, dùng private key được DPAPI bảo vệ để giải bọc DEK và AEAD giải mã, chỉ trả resource đã xác thực vào nghiệp vụ; không xuất key API cho Add-in, không tạo plaintext trên đĩa, quản lý bộ nhớ/zeroization best-effort, dừng an toàn trước mutation CAD nếu lỗi. Không cache quyền qua lần thực thi mới làm mất kiểm tra License hiện tại; policy session cụ thể tại 09.4. Không có fallback plaintext hay giải mã khi thiếu grant.

**Vòng đời và tương thích:** Tạo một lần recipient key ở lần đăng ký; không tự tạo mới khi key thiếu/hỏng/mất, không tự sửa machine tuple; nếu cài lại Windows/mất DPAPI hoặc recipient key, người dùng phải xin cấp lại grant/License sau xác minh của issuer, kể cả SMBIOS vẫn đủ 2/3. Thay linh kiện còn đạt 2/3 và recipient key vẫn dùng được: không cần cấp lại vì FBE; nếu không còn đạt hoặc key không đọc được, dừng và xử lý theo thẩm quyền issuer. Đổi user Windows trên cùng máy: dự kiến hoạt động với DPAPI machine-scope + quyền đọc blob; phải test. Sao chép riêng Add-in/resource/license sang môi trường Windows khác không có private key/DPAPI tương ứng dự kiến không giải mã được; clone đầy đủ OS/keys hoặc attacker có quyền admin không được coi là đã ngăn. Cấp lại/gia hạn cùng resource revision và recipient dùng grant mới do issuer ký; khi đổi resource revision/DEK phải cấp lại grant cho recipient có quyền, không tự hạ version. Nếu DEK lộ, resource revision dùng DEK đó cho mọi khách có thể bị đọc; tách key giảm blast radius, không thể thu hồi plaintext/DEK đã lộ hay ép mọi máy offline nhận bản mới. Nếu recipient key lộ, các grant bọc cho key đó có nguy cơ, issuer ngừng cấp cho key cũ, đăng ký mới và reissue; việc từ chối offline ngay lập tức không được bảo đảm. Mất kho DEK quản trị: phục hồi từ backup kiểm chứng hoặc bảo vệ lại resource và cấp lại grant, không phục hồi bằng khóa ký. Anti-clock 07 vẫn tạm dừng.

**Ảnh hưởng theo phương án này:** DCAD_ID/RequestFormat/RequestImport cần thêm enrollment nếu triển khai; LicenseccManager/issuer và DcadSeal/SDK phải xử lý key grant đã ký, DB lưu metadata recipient/key/resource/lifecycle nếu cần, Inspector cần kiểm tra/chẩn đoán không tiết lộ bí mật; ưu tiên giữ inner Licensecc payload, MachinePolicy2, Feature/Edition/time và RSA-PSS. Gate/Client cần API resource mới, Integration metadata/version và workflow phát hành cần xem xét; TidiDcad quản lý resource/DEK, Build phải bảo vệ trước ký PE, không thay SEC06/Authenti­code; module Add-in chỉ thay điểm dùng resource thiết yếu. Các thay đổi này chưa là diff được duyệt. Không tự giữ ABI/format khi thực tế không tương thích. Ưu tiên một License, không thêm DLL/EXE runtime ngoài hệ thống; có thể cần thêm blob private key DUser và resource được bảo vệ (nhúng/rời chốt 09.4). Add-in hiện hữu không tự hưởng FBE; License hiện hữu không có grant, Feature áp dụng FBE phải từ chối nếu thiếu grant. Quyết định tương thích cũ/mới, cấp lại Dreq/License, Integration/build/sign, chuyển version và rollback phải duyệt trước code. Giữ 01–06, 08 và các quy ước CAD, không sửa 07.

**Giới hạn / điều kiện tiếp tục:** phần mềm offline không có trust boundary độc lập; DPAPI machine-scope không bảo vệ chống người kiểm soát máy hoặc cùng user có blob; recipient private key, DEK và resource plaintext có thể bị lấy trong runtime, license expiration không thu hồi secret đã lấy. Resource chung bị lộ từ một máy hợp lệ có thể bị chia sẻ. Không cam kết tăng đáng kể đối với attacker có license hợp lệ, không đánh đồng AES/RSA mạnh với khả năng chống bypass. FBE có giá trị đối với sao chép artifact thiếu key và bypass boolean đơn thuần khi thiếu resource thật; nếu không có resource thiết yếu, không thể bảo vệ/lưu giữ khóa theo ràng buộc của chủ dự án hoặc lợi ích không đủ so với chi phí enrollment/reissue thì dừng FBE, giữ Gate. Prototype 09.5–09.6 phải so sánh baseline, kiểm tra đúng/sai key/recipient, grant bị sửa/thay thế, thiếu key/grant, sai Feature/resource/revision, ciphertext/tag/binding sai, renewal/rotation, mất key, clone/khác môi trường, user Windows khác, backup/restore, không fallback và tác động hiệu năng; lỗi dữ liệu không tạo hình sai. Các ca và giới hạn thực nghiệm chốt 09.4.

**Trạng thái 09.3:** khảo sát và thiết kế cấp kiến trúc đã thực hiện; phương án trên là đề xuất, chưa được chủ dự án duyệt. Đặc biệt cần xác nhận chấp nhận key nhận gắn với cài đặt Windows, việc phải cấp lại khi mất key dù SMBIOS 2/3 vẫn đạt, phạm vi lộ resource chung và thay đổi có phiên bản đối với Dreq/License. Chưa xác nhận resource/pilot, chưa duyệt schema/API/tên file/backup format, chưa code, chưa sinh khóa, chưa build/test FBE hoặc Windows/CAD. Không thay V61, không tăng version phần mềm. Chờ chủ dự án chốt 09.3 rồi mới 09.4; chỉ tích hợp sau nghiệm thu 08 và pilot.

**Điều chỉnh 09.3 — yêu cầu của chủ dự án, 27/11/2026:** Chủ dự án không đồng ý phương án khóa nhận phụ thuộc cài đặt Windows; yêu cầu nghiên cứu quản lý khóa FBE không phụ thuộc cài đặt Windows và giữ tính portable hiện tại. Toàn bộ đề xuất DPAPI/recipient key gắn cài đặt ở ghi nhận 09.3 trước là lịch sử khảo sát, KHÔNG phải thiết kế được duyệt, KHÔNG được triển khai. Giữ 09.1–09.2 Approved; 09.3 vẫn đang nghiên cứu/chờ duyệt; chưa được chuyển 09.4. Yêu cầu portable được hiểu: khi khôi phục đầy đủ bộ file DCAD, máy vẫn thỏa SMBIOS 2/3 và mọi điều kiện License khác, việc cài lại Windows/đổi tài khoản/thay ổ không tạo thêm yêu cầu cấp lại vì FBE. Không bảo đảm phục hồi file đã mất nếu không có backup; không mở quyền sang máy khác. Ưu tiên giữ luồng sử dụng không nhập mật khẩu như hiện tại (đề xuất cần xác nhận, không phải yêu cầu mới đã duyệt). Không giữ bí mật FBE dựa trên DPAPI, registry, Credential Manager, Windows certificate store, MachineGuid, OS installation identity, ProgramData hoặc NCrypt/KSP; không thêm TPM/dongle/server/driver. Dùng API mật mã BCrypt hiện có không đồng nghĩa ràng buộc khóa với cài đặt Windows.

**Cơ sở đánh giá lại:** [NIST SP 800-57 Part 1 Rev.5](https://csrc.nist.gov/pubs/sp/800/57/pt1/r5/final) phân tách mục đích, lưu trữ, phục hồi và xử lý lộ khóa; [NIST SP 800-38D](https://csrc.nist.gov/pubs/sp/800/38/d/final) AEAD; [Microsoft CryptProtectData](https://learn.microsoft.com/en-us/windows/win32/api/dpapi/nf-dpapi-cryptprotectdata) phụ thuộc môi trường máy và LOCAL_MACHINE cho phép user cùng máy giải mã; [RFC 9180](https://www.rfc-editor.org/rfc/rfc9180.html) bảo mật mã hóa khóa công khai phụ thuộc bảo vệ private key bên nhận, không tự giải quyết lưu khóa; [NIST SP 800-132](https://csrc.nist.gov/pubs/sp/800/132/final) và [RFC 9106](https://www.rfc-editor.org/rfc/rfc9106.html) dẫn xuất khóa từ mật khẩu/passphrase để bảo vệ lưu trữ; [Sentinel decrypt](https://docs.sentinel.thalesgroup.com/ldk/LDKdocs/API-licensing/Licensing_API/hasp_decrypt.htm) dùng protection key và session/Feature, [Sentinel clone protection](https://docs.sentinel.thalesgroup.com/softwareandservices/ldk/LDKdocs/SPNL/LDK_SLnP_Guide/Licensing/Managing_products/Clone_protection.htm) có giả định/phụ thuộc môi trường khác DCAD. Không suy diễn các công nghệ thương mại chứng minh chống mọi bypass trong mô hình portable.

**Giới hạn nền tảng:** Nếu mọi dữ liệu cần thiết để giải mã đều được sao lưu/copy, ứng dụng tự giải mã offline không cần bí mật ngoài bộ portable, và không có trust boundary độc lập, không thể bảo đảm bí mật trước người kiểm soát toàn bộ dữ liệu/runtime. Mã hóa với khóa nằm cạnh dữ liệu, chia nhỏ khóa, bọc bằng khóa chung nhúng Gate, hoặc dẫn xuất từ SMBIOS/hash/chữ ký công khai không tạo ra bảo mật giữ khóa độc lập. SMBIOS 2/3 giữ vai trò kiểm tra quyền sử dụng, không phải bí mật mật mã; không làm biến dạng chính sách để ép giải mã. RSA-PSS xác thực, không che giấu DEK. Phải phân biệt (a) người không có grant hợp lệ/không có DEK, (b) có bản sao License chứa DEK hoặc có License hợp lệ của Feature, (c) kiểm soát runtime/OS; (a) là phạm vi lợi ích chính có thể kiểm chứng, (b)/(c) không được tuyên bố bảo mật mạnh. Không dùng kết quả từ chối của Gate nguyên bản làm bằng chứng tài nguyên không thể giải mã độc lập khi key đã lộ.

**So sánh phương án portable:**
- P0 giữ Gate hiện tại: không thêm lợi ích FBE nhưng không phát sinh rủi ro/quản lý khóa; baseline đối chứng, chọn nếu FBE không đáng làm.
- P1 grant portable có chữ ký chứa DEK riêng theo Tool/Feature/Resource/revision: giữ Dreq 6/SMBIOS 2/3, không recipient key, không trạng thái OS hoặc mật khẩu khách; đơn giản nhất, phân phối key chỉ khi issuer cấp quyền. DEK là dữ liệu đọc được từ License/grant (encoding không phải mã hóa), người có bản sao grant/DEK hợp lệ có thể giải mã resource ngoài Gate dù khác máy hoặc hết hạn, và chia sẻ key/plaintext. Không tuyên bố chống copy toàn bộ license bằng mật mã, không bảo vệ trước khách hợp lệ trích xuất. Chữ ký và Machine 2/3 vẫn kiểm tra bình thường trong Add-in/Gate nguyên bản. Không ưu tiên production nếu mục tiêu cần chống cả người có License/DEK; đề xuất ứng viên nghiên cứu/prototype có điều kiện vì ít tác động nhất và cho phép đo lợi ích chính xác.
- P2 recipient key portable riêng lưu file DUser: có thể giữ nguyên khi cài lại Windows nếu backup đầy đủ, nhưng key tự giải mã/không được bảo vệ bởi bí mật bên ngoài sẽ sao chép được cùng bộ file; public-key wrapping chỉ bảo vệ khi đối phương có grant nhưng không có private key, không chống sao chép đầy đủ; thêm enrollment/Dreq, key blob, issuer, backup/reissue. Không ưu tiên vì tăng độ phức tạp nhưng không có trust boundary mới.
- P3 portable secret được bảo vệ bằng passphrase thực sự tách khỏi bộ file, chẳng hạn encrypted recipient key hoặc encrypted grant: tăng bảo mật lưu trữ đối với người chỉ có bản sao file nếu passphrase đủ mạnh; có thể khôi phục trên Windows mới với passphrase, nhưng thay luồng sử dụng, quản lý quên mật khẩu/backup và vẫn có rủi ro runtime/chia sẻ bí mật. Không lưu passphrase trong bộ DCAD; không dùng tự nhập/khôi phục tự động để tuyên bố bí mật độc lập. Chỉ xem xét nếu chủ dự án chấp thuận UX và quy trình phục hồi riêng. Không chọn mặc định, chưa duyệt KDF/format.
- P4 mã hóa khóa bằng DeviceID/SMBIOS 2/3 hoặc chia khóa nhiều mảnh từ các thông số công khai: loại, không có bí mật độc lập, tạo nguy cơ phá chính sách thay linh kiện. Mã hóa khóa bằng secret nhúng Gate/ARX hoặc ngụy trang khóa: không chọn. Không tái sử dụng private key ký License, SEC06, Authenticode cho giải mã. USB/TPM/server vẫn ngoài phạm vi.

**Ứng viên đề xuất sửa đổi 09.3 — Portable Feature Grant (P1), chỉ phạm vi nghiên cứu/QA, chưa duyệt:** quản trị sinh DEK ngẫu nhiên theo Tool/Feature/Resource/revision, AES-256-GCM ứng viên, nonce duy nhất theo key, ràng buộc metadata và hash ciphertext trong grant/danh tính phát hành có chữ ký. Resource thật thiết yếu, không có bản plaintext đi kèm. Issuer cấp DEK trong grant có chữ ký gắn đúng quyền Feature/Resource/revision, License hiện hành và định danh machine đã ký; không coi DEK trong grant là bí mật trước người nhận grant. Ưu tiên một Dcad.lic có envelope/version mới nếu cần; không append vào canonical V4/V5 hoặc tự đổi giao thức. Prototype dùng dữ liệu và khóa QA độc lập. Gate phải giữ toàn bộ xác thực trước khi đọc grant/giải mã tài nguyên; dữ liệu được kiểm tra trước khi thay đổi CAD; không fallback plaintext, không cache quyền qua lần gọi mới, không xuất API trả DEK, không ghi plaintext/log; đây là vệ sinh triển khai không phải ranh giới chống reverse engineering. Quản trị giữ DEK production trong DData có mã hóa lưu trữ và backup độc lập, ưu tiên passphrase phục hồi do quản trị nắm (không áp lên khách); chốt tại 09.4 nếu được duyệt. Không private key issuer hoặc secret chung giải mã toàn hệ thống ở máy khách/developer. Cùng resource/revision dùng cùng DEK, lộ một DEK ảnh hưởng mọi khách có resource đó; tạo revision/DEK mới và cấp grant mới khi cần, không thu hồi được dữ liệu/key đã phát tán offline. Không buộc cấp lại khi chỉ build lại ARX mà resource/revision và quyền giữ nguyên, cần xác minh tương thích ở 09.4.

**Tác động thay đổi so với đề xuất trước:** ưu tiên KHÔNG sửa DCAD_ID/RequestFormat/Dreq 6/RequestImport, không đăng ký recipient key, không thêm file private key DUser, không yêu cầu DPAPI/OS state hoặc reissue do cài Windows; giữ machine tuple, Tool/Feature/Edition/time, khóa ký và Licensecc inner payload. Có thể cần thay envelope có version, LicenseccManager/issuer/DcadSeal/SDK/Inspector, resource metadata/DB, Gate/Client/Integration, bước bảo vệ resource trước ký PE và điểm truy cập resource của Add-in. Chưa xác định diff, format hoặc chính sách tương thích chính thức; chỉ chốt sau khi duyệt 09.3. Nếu Feature không có tài nguyên thực sự thiết yếu, không áp dụng giả tạo. Không coi P1 là phương án bảo mật production đã chốt.

**Cổng kiểm chứng bổ sung:** prototype phải chứng minh không có DEK thì không khôi phục được resource từ artifact được phát hành và bypass boolean đơn thuần không đủ; đối chứng người có bản sao grant/DEK đọc được phải ghi đúng giới hạn, không thiết kế test chỉ dựa trên Gate từ chối. Test nguyên bản trên máy sai SMBIOS phải DENY; test với License hợp lệ đúng máy sau cài Windows sạch, đổi user, đổi ổ/khôi phục full DCAD, SMBIOS vẫn đạt 2/3, không cần reissue vì FBE. Kiểm tra thiếu/corrupt grant, sai binding/resource/version, lỗi I/O, không tạo CAD sai, renewal/rotation, backup, hiệu năng và hồi quy 01–06/08. Đo lợi ích theo đối thủ đã duyệt, không đặt điểm bảo mật giả. Nếu P1 chỉ tăng khó khăn với người chưa có License, trong khi mục tiêu thực tế là ngăn người có License chia sẻ/crack và chi phí tích hợp không xứng đáng, dừng FBE hoặc xem xét P3 khi có chấp thuận riêng. Không tự mở rộng sang obfuscation, FBE Function hoặc hệ thống khóa khác.

**Trạng thái:** đã đối chiếu Plan STD-20 và các tài liệu công khai nêu trên; sử dụng kết quả khảo sát nguồn V61 đã ghi ở 09.2/09.3, chưa mở lại ZIP trong lượt điều chỉnh, không tuyên bố kiểm tra lại CRC/build. 09.3 chưa được nghiệm thu; P1 chỉ là đề xuất ứng viên portable tối giản, P3 là phương án nếu chấp nhận passphrase; P0 vẫn hợp lệ. Chờ chủ dự án xác nhận đánh đổi trước 09.4. Không code, không tích hợp V61, không sinh khóa, không tăng version, không thay hợp đồng kỹ thuật mục 2 trở đi. V59 vẫn là baseline được xác nhận; V61 ứng viên 08 chờ nghiệm thu Windows/CAD.

**Điều chỉnh phạm vi 09 — 27/11/2026 (quyết định mới nhất của chủ dự án):** Mục tiêu chính giữ nguyên: Anti-Gate Bypass, không cho việc bỏ qua một bước kiểm tra quyền đơn thuần đủ để hoàn thành chức năng được bảo vệ. Mã hóa, Session, Resource Key và FBE Data/Function/Module là phương án, không phải mục tiêu bắt buộc. Chủ dự án yêu cầu cho phép tạo thêm một bộ chạy nền Windows độc lập, không cứng nhắc portable tuyệt đối mà hạn chế tối đa phụ thuộc/cài đặt. Đây là điều chỉnh riêng điều kiện portable và ưu tiên không tăng module runtime đã ghi trước; cho phép khảo sát một thành phần service dùng chung, không phải phê duyệt thiết kế cụ thể hay cho phép code. Các ghi nhận trước vẫn lưu lịch sử; phần cấm tuyệt đối thành phần chạy nền/đòi portable tuyệt đối trong phạm vi 09 được thay bằng quyết định này. Không tiếp tục 07 hoặc System Check.

**Định hướng nghiên cứu bổ sung (đề xuất, chưa duyệt kỹ thuật):** Windows Service native x64, tên làm việc DcadSecurityService.exe, tiến trình riêng do Windows Service Control Manager quản lý; ưu tiên user-mode, một service dùng chung, chạy offline, không bắt buộc thêm driver, USB dongle, TPM hoặc server/network. Gate/Client hiện hữu tiếp tục là điểm tích hợp Add-in, không tự bỏ các kiểm tra hiện hành. Chưa quyết định phân bổ chính xác xác thực License/Feature/machine/chữ ký giữa Gate và service hay thành phần nghiệp vụ thiết yếu được bảo vệ. Chưa chọn giao thức/API, không mặc định thêm khóa/mã hóa, không phê duyệt Protected Execution hoặc grant portable cũ. Khảo sát kênh IPC nội bộ Windows (ví dụ named pipe), phân quyền tối thiểu và xác thực đầu cuối; không cấp LocalSystem mặc định, không mở cổng mạng, không can thiệp tiến trình không thuộc DCAD, không chống gỡ cài đặt trái ý người dùng, không thay chính sách bảo mật toàn hệ điều hành. Thiếu service, không tương thích, lỗi IPC, timeout hoặc service không tin cậy phải xử lý an toàn, không fallback bỏ kiểm tra; không treo CAD. Service chỉ trả ALLOW/DENY vẫn không tự ngăn Add-in bỏ qua lệnh gọi. Phải nghiên cứu ràng buộc quyền với một khả năng/tài nguyên nghiệp vụ thật hoặc cơ chế cưỡng chế phù hợp và đánh giá lại lợi ích. Tách tiến trình tăng khả năng cách ly theo quyền hệ điều hành nhưng không tự tạo ranh giới chống administrator toàn quyền trên máy khách.

**Phạm vi giữ nguyên:** offline, SMBIOS 2/3, Tool ID/Feature ID/Edition/thời hạn, khóa ký hiện tại, SEC06/SEC-AT, Licensecc và kết quả hình học/UI/XData/settings. Ưu tiên giữ Dreq và Dcad.lic hiện hữu, không thêm key nhận gắn Windows (phương án DPAPI/recipient theo cài đặt trước vẫn không được duyệt; cho phép cài service không đồng nghĩa chấp nhận ràng buộc License vào Windows). Đề xuất cài lại service sau cài lại Windows mà không cấp lại License nếu dữ liệu DCAD và SMBIOS 2/3 vẫn hợp lệ; cần kiểm chứng. Ưu tiên file sản phẩm/dữ liệu đúng các thư mục STD-03, khảo sát cài/gỡ/nâng cấp service có quyền admin thông qua SCM, quản lý ACL, đường dẫn root, nhiều tài khoản/host CAD, ổ đĩa, lỗi/rollback; đăng ký SCM là phụ thuộc OS cần xem xét rõ, không tự thay STD-03 hoặc tạo lưu trữ ngoài phạm vi. Không yêu cầu developer có khóa riêng production. Chưa cho phép kernel driver, thay chính sách App Control toàn máy, hoặc phụ thuộc OS mới ngoài thành phần chạy nền đang nghiên cứu.

**Nguồn đối chiếu trong lần điều chỉnh:** [Thales Sentinel License Manager](https://docs.sentinel.thalesgroup.com/softwareandservices/ldk/LDKdocs/SPNL/LDK_SLnP_Guide/Distributing/License_Manager/020-License_Manager_Type.htm) mô tả các loại License Manager, bao gồm service cho Admin LM; [Microsoft Service Security and Access Rights](https://learn.microsoft.com/en-us/windows/win32/services/service-security-and-access-rights), [Service User Accounts](https://learn.microsoft.com/en-us/windows/win32/services/service-user-accounts), [Named Pipe Security](https://learn.microsoft.com/en-us/windows/win32/ipc/named-pipe-security-and-access-rights), [LocalService](https://learn.microsoft.com/en-us/windows/win32/services/localservice-account), [App Control](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/appcontrol) để phân biệt cách ly service với cưỡng chế mã thực thi. Không coi các nguồn xác nhận DCAD đã đạt bảo mật tương đương, không chọn SDK thương mại.

**Cổng kế tiếp 09.3 (phạm vi điều chỉnh, chưa chuyển 09.4):** đối chiếu giữ Gate hiện tại, Windows Service chỉ xác thực, Windows Service có phụ thuộc nghiệp vụ thực sự và các lựa chọn FBE Data/Execution đã khảo sát; xác định mô hình nguy cơ, phần nào cần tách, có thực sự cần quản lý khóa mới không, chi phí IPC/hiệu năng, phụ thuộc cài đặt, quyền, trạng thái mất service và tương thích V61. Trình phương án ít thay đổi nhất, ma trận ảnh hưởng các module, trách nhiệm admin/developer/user, lifecycle cài/gỡ/upgrade/restore, kế hoạch QA so sánh bypass đơn thuần và kiểm thử trên máy do khách kiểm soát. Nếu chỉ chuyển boolean sang service hoặc tăng phức tạp không đem lại đủ lợi ích thì không tích hợp. Chỉ chuyển 09.4 sau chủ dự án duyệt phương án 09.3; giữ nguyên các cổng prototype và pilot 09.5–09.10, nghiệm thu 08 trước tích hợp production. Đây là cập nhật yêu cầu và hướng khảo sát, chưa đọc lại ZIP V61 trong lượt này, chưa rà source service (chưa có), chưa code/build/test, không tăng version, V59 baseline đã xác nhận và V61 ứng viên chờ Windows/CAD.

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

**Hạng mục 10 — Mã hóa chống dịch ngược, tiếp nhận Handover V1 và đối chiếu V70 → V74 (28/11/2026)**

Chủ dự án yêu cầu bổ sung tiến trình vào Plan chính thức và lấy V74 làm baseline tích hợp. [STD-20.1 V1.2 (Tạm thời)](20.1_Ma_hoa_chong_dich_nguoc_Tam_thoi.md) vẫn là tài liệu nghiên cứu chi tiết, không phải hợp đồng production đã nghiệm thu. Việc ghi Plan không cấp quyền triển khai, không sửa hợp đồng tích hợp hiện hành.

**Căn cứ đã kiểm tra trực tiếp:** Ba ZIP đều CRC PASS; SHA-256 của V70 `f212eac53f083bec93d17cb0ab0d397b0517bb1909ab3c0a24e314aa6a7355e8`, V74 `5be6fbf8f78f3d0778dd88213229ed667a0793a20d2bc1f46b477d60c8d639b5`, Handover V1(1) `1c517a1dc638115d1c17337abfcaf841c80b2fd92269a18eae0af58133df06d0`. Manifest Handover 12/12 và Lab V3 25/25 khớp. Fingerprint V70 khớp tham chiếu trong STATUS.json của Handover. Đã đọc báo cáo đánh giá, ma trận tác động, QA và nguồn Lab V3. V70 chỉ là nguồn hệ thống được khảo sát, không có bằng chứng Gate/ARX thật đã được bảo vệ trên V70. Lab chạy độc lập: GCC/Clang mỗi bộ 5/5 theo log lưu sẵn; demo Windows Native = VM = AES-GCM + VM = 253 theo bản ghi chủ dự án. Chưa có log Windows V3 Build_All/Test_All đầy đủ, LabLoader và nghiệm thu CAD. Lab chỉ kiểm chứng bytecode số học mẫu và đọc cấu trúc PE; chưa có PE Rebuilder hoặc chuyển đổi native post-build. Không chạy lại build/runtime trong lần cập nhật Plan này.

**Đối chiếu nguồn payload V70 và V74:** V70 có 263 file, V74 có 267; 257 file giống byte, 6 thay đổi, 4 bổ sung, không xóa file. Các đường dẫn tương đối từ TidiDcad-Source/payload/.

| Phạm vi | Kết quả đối chiếu | Điều kiện tiếp nhận |
|---|---|---|
| Giao diện và điều phối | Source/TidiDcad/Source/Main.cpp, ReleaseSecurity.cpp/.h thay đổi | Dùng bố cục và vòng đời 5 tab V74; không chép GUI V70 đè lên |
| LicenseccManager | Source/LicenseccManager/Source/Main.cpp thay đổi; EmbeddedManager.h, EmbeddedReleaseReady.h bổ sung | Giữ khởi tạo và bảo toàn danh tính V74; không mở rộng phạm vi cấp phép |
| Build/QA | Build/TidiDcadCompileCache.ps1, Source/TidiDcad/Build.cmd thay đổi; Source/Common/NativeControls.h và Source/TidiDcad/QA/VerifyEmbeddedBootstrap.cpp bổ sung | Giữ kiểm thử khởi động, chuyển tab và bootstrap; chỉ sửa phần cần thiết |
| Ký Gate/Add-in, build và publish | PortableSecurity.ps1, PortableSecurityEmbedded.h, StreamBuild.ps1, PublishRootBinaries.ps1 không đổi byte | Dùng hợp đồng hiện hành; rà cả hai đường ký và giao dịch phát hành |
| Gate, Client, Inspector, Integration | Nguồn liên quan không đổi byte | Giữ ABI/policy, định dạng, danh tính khóa và sáu file Integration; kiểm thử lại trên V74 khi có thay đổi |
| Setup/dependency | Giữ cơ chế V74 | Chỉ cập nhật inventory, build và đóng gói khi có thành phần mới được duyệt |

**Phạm vi quản lý và trình tự:** Không tạo tab mới hoặc hệ thống bảo mật thứ hai. Dự kiến dùng chung công cụ bảo vệ sau khi được nghiệm thu, với phạm vi riêng cho Gate và Add-in. Hai luồng ký giữ thẩm quyền/danh tính độc lập. Mọi thay đổi binary phải hoàn tất trước ký; kiểm tra và ký đúng thành phẩm cuối; bảo toàn bản gốc, không tự công bố bản chưa bảo vệ dưới nhãn đã bảo vệ. Giữ nguyên các kiểm tra bảo mật Windows/CAD, không vô hiệu hóa cơ chế bảo vệ. Không tự thay đổi License, Dreq, SMBIOS 2/3, Feature, Gate ABI 2/Machine Policy 2/Add-in Policy 1, khóa, dữ liệu khách hàng hoặc nghiệp vụ CAD.

| Bước | Công việc / cổng nghiệm thu | Tiến độ |
|---|---|---|
| 10.1 | Tiếp nhận Handover, xác định baseline và đối chiếu V70/V74 | Đã kiểm tra CRC, manifest, đối chiếu payload; chủ dự án xác nhận V74 OK; khảo sát tĩnh, chưa nghiệm thu runtime |
| 10.2 | Hoàn thiện hồ sơ kiểm thử Lab, thẩm định thiết kế và phạm vi độc lập theo STD-20.1 | Demo mẫu đã xác nhận; Windows QA đầy đủ, thẩm định dependency và đặc tả còn thiếu |
| 10.3 | Đánh giá khả năng đáp ứng mục tiêu bảo vệ sở hữu trí tuệ và tương thích trên mẫu được phép; nghiệm thu độc lập công cụ trước tích hợp | 10.3.2: chủ dự án báo DCAD-10.3-PE-QA-V1 thử thành công; log chi tiết chưa đối chiếu. Tiếp theo 10.3.3: thẩm định công nghệ/dependency. Chưa đạt toàn bộ cổng |
| 10.4 | Khi đủ điều kiện và được duyệt, tích hợp chức năng bảo vệ/ ký Gate vào tab Quản lý tài khoản và khóa phát hành; rà cả luồng Build tự động | Chưa triển khai; giữ cơ chế SEC06 hiện hành |
| 10.5 | Khi đủ điều kiện và được duyệt, tích hợp chức năng bảo vệ/ ký Add-in vào tab Ký duyệt công cụ | Chưa triển khai; giữ Authenticode và Feature |
| 10.6 | Nghiệm thu riêng và kết hợp Gate/Add-in, hồi quy baseline V76, tính toàn vẹn, tương thích và hiệu năng; đánh giá độc lập | NotVerified; kiểm thử đúng thành phẩm và đúng host đã duyệt, không thay điều kiện PASS/DENY |
| 10.7 | Hoàn thiện Setup, Build_setup.cmd, hồ sơ QA và một ZIP hệ thống; chủ dự án duyệt phát hành | Chưa thực hiện; không tăng phiên bản phần mềm trong lần cập nhật tài liệu |

Bước 10.1–10.7 là các mốc quản lý tổng hợp, không thay lộ trình 11 bước nghiên cứu chi tiết của STD-20.1. Tám phạm vi công việc BM01–BM08 được theo dõi trong các mốc này: công cụ, giao diện, Gate, Add-in, Build tự động, QA, lưu trữ/phục hồi và Setup. Chỉ bổ sung thay đổi trực tiếp cần thiết, review ảnh hưởng và nghiệm thu theo STD-18/16 trước bàn giao. Kiểm thử đối chứng A (Gate gốc/Add-in gốc), B (Gate bảo vệ/Add-in gốc), C (Gate gốc/Add-in bảo vệ), D (cả hai bảo vệ); B/C đạt độc lập trước D. Không dùng kết quả Lab hoặc V70 làm PASS cho thành phẩm V74 mới. Tất cả các thay đổi tích hợp còn ở trạng thái NotVerified.

**Bước tiếp theo tại mốc V74 (lịch sử):** 10.2; hoàn thiện chứng cứ còn thiếu, đánh giá phạm vi và thiết kế trước khi chủ dự án duyệt triển khai. Chỉ cập nhật Plan STD-20 V1.4; không thay nội dung hợp đồng kỹ thuật hiện hành, không sửa V74, không tạo V75, không thay ZIP quy ước đã phát hành, không tiếp tục hạng mục 07/09.

**Quyết định V1.5 — 28/11/2026:** Chủ dự án duyệt bỏ thủ tục xác nhận ngoài ZIP khi trực tiếp bàn giao Integration. V76 lấy V75 làm nguồn chuẩn, giữ ký và xác minh SEC06 lúc xuất, thay thông báo và hướng dẫn người tích hợp. Người lập trình chịu trách nhiệm hoàn tất bốn file bảo mật trước bàn giao, không chuyển intake sang máy build. Không sửa Gate/License/Authenticode/Machine 2/3, không triển khai 10.3. V76 QA mô hình Linux 64 ca PASS, cross-build Setup x64 và 267 payload file khớp; Windows/MSVC/CAD NotVerified. Các ghi nhận lịch sử dưới Plan không thay thế INT-06 V1.5 hiện hành.

**Cập nhật tiến độ V1.6 — 28/11/2026:** Chủ dự án chọn bộ TidiDcad V76 là baseline OK mới nhất để phát triển tiếp, thay mốc V74; đây là xác nhận của chủ dự án, không phải nghiệm thu production hoặc chứng cứ từng ca Windows/CAD. Giữ V74/V75 và lịch sử trước đó. Không tạo phiên bản phần mềm mới trong lần cập nhật này.

- **Tồn đọng cần xử lý trong lần sửa nguồn tiếp theo:** hệ thống cấp và xác minh License DCAD-LIC-V4/V5 đúng theo hợp đồng API hiện hành; nguồn sinh ToolInfo.json (IntegrationModel.h) còn khai báo V2/V3. Sửa metadata tại nguồn xuất, đối chiếu hợp đồng Gate và tài liệu STD-20 hiện hành, bổ sung QA LicenseFormats, rồi xuất/ký lại Integration khi cần. Không chỉnh tay JSON đã ký, không đổi giao thức License, khóa, runtime hay cấp lại License chỉ vì lỗi metadata. Còn phải kiểm chứng trên nguồn cuối theo STD-18; chưa sửa ở lần cập nhật tiến độ này.
- **10.2:** Lab V3 Windows Build và CTest 8/8 đã được ghi nhận trong trao đổi kiểm thử trước đó; demo nguyên lý không chứng minh bảo vệ PE native. Thẩm định dependency, thiết kế và các điều kiện còn thiếu tiếp tục được theo dõi.
- **10.3.2:** Chủ dự án báo bộ DCAD-10.3-PE-QA-V1.zip đã thử thành công (UserReportedWorking=YES). Bộ kiểm tra chỉ đọc PE, không bảo vệ hoặc biến đổi binary. Chưa nhận/đối chiếu log từng ca, hash trước/sau và tên/hash mẫu trong lần cập nhật này; trạng thái chứng cứ chi tiết NotVerified.
- **Bước tiếp theo 10.3.3:** thẩm định ứng viên công nghệ và thư viện, phiên bản/commit, license, dependency, khả năng bảo trì, tương thích PE32+ x64, khả năng giữ ABI/metadata và giới hạn bảo vệ thực tế; chốt phương án và tiêu chí thử nghiệm trước khi phát triển hoặc chọn dependency. Đối chiếu log PE QA khi có. Không chuyển sang 10.4/10.5 trước nghiệm thu độc lập và quyết định chủ dự án; giữ 07, 09 và System Check tạm dừng. Rà lại đúng nguồn V76 trước tích hợp.
- Chỉ cập nhật Plan; hợp đồng INT-06 V1.5 và các điều khoản kỹ thuật hiện hành chưa thay đổi. Không sửa mã, ZIP đã phát hành hoặc ghi PASS cho bảo vệ binary thực tế.

**Cập nhật tiến độ V1.7 — 28/11/2026:** Chủ dự án duyệt tiếp tục nghiên cứu độc lập 10.3 theo STD-20.1. Baseline hệ thống V76 giữ nguyên; baseline Lab mới nhất đã kiểm thử Windows là V5. Không sửa V76 hoặc tạo V77, giữ tồn đọng LicenseFormats V4/V5, không mở lại 07/09/System Check.

- Lab V5 ZIP SHA-256 `90b8dd320d3580a4aba66e5c9826d7587de8e092f7740d25b116f75f1cfc0291`, CRC và manifest 37/37 PASS. Hai log Windows do chủ dự án cung cấp đã đọc: Build_All(20261128-164614).log SHA-256 `0d095a79122a6b6215a1b86fa4bde98c8a45b8484383b8e9b8a756054b025fde`; Test_All(1).log SHA-256 `2ef812c847892500818ffd77b10777759483b56765bfaea376e06dde0901cb07`. Cùng Run-848ad50416594948ad718bcd1b9cad7d. MSVC 19.51, LIEF SDK 593/593 file hash PASS, Build/Test mỗi lần 10/10 CTest, exit 0, publication Lab PASS. LIEF tái tạo EXE 79360 byte và DLL 9728 byte giống từng byte, kiểm thử lại EXE và DLL mẫu PASS. Bước kỹ thuật 03 đạt phép thử tái tạo nguyên trạng trên hai mẫu, không chứng minh rewrite native tổng quát, bảo vệ PE hay CAD.
- Lab V6 là bản nguồn thử nghiệm mới theo bước kỹ thuật 04, SHA-256 ZIP `7e9b69169a9e1912fa2525cf5f276bdaafa6cb72d1bc1119cb8dd4e676ee84c3`, CRC và manifest 44/44 PASS. Kế thừa V5, chỉ bổ sung QA mã hóa bản sao dữ liệu resource không thực thi của EXE/DLL mẫu trong RAM bằng AES-GCM hiện có; không sửa PE, không sinh thành phẩm bảo vệ, không nhận đường dẫn tùy ý, không thêm thư viện. GCC Release và Clang ASan/UBSan Linux mỗi bộ 7/7 CTest; fixture tổng hợp, boundary và 13 ca PE từ chối PASS. Windows V6/11 CTest, CNG và kiểm thử trên hai PE mẫu thật NotVerified. Không công bố đã đạt cổng 04 hoặc 10.3.
- Bước hiện tại: 10.3.4 trong phạm vi thử nghiệm dữ liệu mẫu độc lập, tiếp tục hoàn thiện chứng cứ 10.2/10.3.3 còn thiếu; chưa nghiệm thu 10.3.5. Thẩm định Zydis, các dependency/thiết kế và phạm vi bảo vệ native vẫn cần thực hiện trước khi mở rộng. Chưa tích hợp Gate/Add-in hoặc thay cơ chế ký; INT-06 và hợp đồng kỹ thuật hiện hành giữ nguyên. V6 chỉ dùng để kiểm thử; sau khi nhận log Windows mới đánh giá bước tiếp theo.

**Cập nhật tiến độ V1.8 — 29/11/2026:** Chủ dự án duyệt tiếp tục 10.3 theo STD-20.1. Hệ thống V76 giữ nguyên, baseline Lab đã kiểm thử Windows chuyển sang V6. Tồn đọng LicenseFormats V4/V5 giữ nguyên; không mở lại 07/09/System Check.

- V6 ZIP SHA-256 `7e9b69169a9e1912fa2525cf5f276bdaafa6cb72d1bc1119cb8dd4e676ee84c3`, CRC và manifest 44/44 PASS. Hai log Windows do chủ dự án cung cấp đã đối chiếu: Build_All(20261128-170227).log SHA-256 `712f5f43bb85bcaff6751dbc8e2ffd51ae97d3f7f037fef3b15f6a266b7f670d`; Test_All(3).log SHA-256 `d3a2cefa176a1ee88ced6087d2bb93741b1711e26354f0793858d30ba0529f23`. Cùng Run-066b668636c148429de8f8ece6742989; LIEF SDK 593/593, Build và Test mỗi lần 11/11 CTest, exit 0; PE_DATA_QA=PASS, ORIGINAL_FILES_UNCHANGED=YES; .rsrc 512 byte của EXE/DLL mẫu được mã hóa và giải mã trong RAM, 100 vòng. Không chứng minh bảo vệ mã máy hoặc CAD.
- V7 là bản nguồn thử nghiệm phân tích chỉ đọc, SHA-256 ZIP `2cb22c62d5d296058a36137aeab83100c7528258a4ceb94bcbc0ce34da4fbb7b`, CRC và manifest 56/56 PASS. Thẩm định Zydis 4.1.1 và Zycore 1.5.2, giấy phép MIT, commit pin trong STD-20.1 và hồ sơ. Chỉ bổ sung ca QA decoder trên fixture cố định và hàm C++ mẫu nội bộ, không chuyển đổi mã, không sửa PE, không nhận file tùy ý. Full checkout upstream chưa có trong môi trường; V7 yêu cầu chuẩn bị recursive checkout chính thức riêng, không tự tải trong Build_All. Linux GCC và Clang sanitizer mỗi bộ 7/7 hồi quy V6; configure từ chối thiếu Zydis, thiếu Zycore, sai commit. Full Zydis build và Windows V7 12/12 NotVerified, chưa nghiệm thu 10.3.
- Bước hiện tại tiếp tục 10.3.3/10.3.4 (thẩm định và thử nghiệm phân tích x64); chưa qua cổng 10.3.5, 10.4/10.5. Không sửa V76, Gate, Add-in, cơ chế ký hoặc hợp đồng INT-06; V7 chưa được chọn làm baseline nghiệm thu. Chờ chứng cứ upstream/full Lab QA trước khi đánh giá bước kế tiếp.

**Cập nhật tiến độ V1.9 — 29/11/2026:** Chủ dự án duyệt tiếp tục nghiên cứu 10.3. Baseline hệ thống TidiDcad V76 giữ nguyên; baseline Lab đã thử Windows là V8. Giữ tồn đọng LicenseFormats V4/V5; không mở lại 07/09/System Check.

- V8 ZIP SHA-256 `74aabd002c4d2400269a0d77e2dd7e22d2e2417c81fc9ba9e3b554fd2c554efb`; manifest 61/61 đối chiếu. Hai log Windows đã đọc: Build_All(20261128-172758).log SHA-256 `b0f7a9e969106a80920486dfc9ceb2d5e9522beeb145397dd0c9fb31a0376531`; Test_All(4).log SHA-256 `a6f8fcc1488ba354448831b3b55720b95c613c0e2f15a9a093eeee8565943096`. Cùng Run-8d05d5f53dd241a88de6c5a9379dc5e4, 12/12 CTest mỗi lần, exit 0. Kho chung D:/000/Dependencies/Libraries/Zydis-4.1.1 được chọn đúng; kiểm tra HEAD và trạng thái checkout PASS, Zydis/Zycore static build PASS, LIEF 593/593. Native decode chỉ đọc hàm mẫu MSVC 66 byte, 18 instruction, 128 đối chiếu PASS; không chứng minh bảo vệ native.
- V9 là bản nguồn thử nghiệm ABI/metadata, SHA-256 ZIP `6c14473b042d605ef33550c0f54f6924a6ae254ec8e1955de2c469a0a4569777`; CRC và manifest 69/69 PASS. Chỉ mở rộng QA hợp đồng C ABI mẫu (108 cặp giá trị biên/guard), báo cáo cấu trúc PE trên fixture và ba mẫu tự build Windows; không đổi mã lõi, decoder, thư viện hoặc PE gốc. Sửa nhãn V86 thành V76; giữ stderr PowerShell vì chưa xác minh RemoteException. GCC và Clang sanitizer Linux mỗi bộ 7/7, resolver 8/8, ba negative control PASS; Windows V9, full Zydis build V9, CAD và native protection chưa xác minh. Ma trận phạm vi và tiêu chí đánh giá trong Design-V9.md; chưa đặt ngưỡng hiệu năng hoặc kết luận hiệu quả chống dịch ngược.
- Tiếp tục 10.3.3/10.3.4 trong phạm vi kiểm tra tương thích độc lập. Chưa đạt 10.3.5, chưa chuyển 10.4/10.5, không sửa Gate, Add-in, khóa, cơ chế ký hay V76. V9 chỉ dùng để kiểm thử, không phát hành production.

**Cập nhật tiến độ V1.10 — 29/11/2026:** Chủ dự án duyệt bước thiết kế tiếp theo. Baseline Lab đã thử Windows là V9; TidiDcad V76 và tồn đọng LicenseFormats V4/V5 giữ nguyên, không mở lại 07/09/System Check.

- V9 ZIP SHA-256 `6c14473b042d605ef33550c0f54f6924a6ae254ec8e1955de2c469a0a4569777`, manifest 69/69 xác minh. Hai log Windows: Build_All(20261128-173553).log SHA-256 `afc5b557ad9545240eecb09756999f1e3987d2e312f1c90a75d2129a21877496`; Test_All(5).log SHA-256 `390a2d7b247ba2deefbc1c4acd492215a35a7265b7e59604b6daef8cf8b28c4b`. Cùng Run-d507b80adb0545d08563120be477a9f2, 12/12 CTest mỗi lần, exit 0; ABI 108 cặp, output guards nguyên, metadata 3 PE mẫu, decoder mẫu PASS, LIEF 593/593. Native protection chưa xác minh.
- Lab V10 bản nguồn thử nghiệm SHA-256 `db1ec0f2dcc97fc48fff636f5a9d73aea548d2d46c5ccd39b015b0f1baeef35e`, CRC/manifest 78/78. Hồ sơ kiến trúc, tài sản, threat model, giới hạn, quản lý khóa và cổng nghiệm thu đề xuất; chỉ thêm một CTest tính toàn vẹn hai file dữ liệu cố định không thực thi, 12 tình huống, không đổi binary/core/publish/keys. Linux GCC và Clang ASan/UBSan 8/8 mỗi lượt; resolver fixtures 8/8; negative control sai digest bị từ chối. Windows V10/13 CTest, chữ ký, khóa thật, CAD và native protection NotVerified. Không thêm dependency. Thiết kế production và ngưỡng hiệu năng chưa được duyệt; không coi hash là chữ ký.
- Tiếp tục nghiên cứu độc lập 10.3.3/10.3.4; chưa nghiệm thu 10.3.5, chưa chuyển 10.4/10.5. Không sửa V76, Gate, Add-in, License, cơ chế ký, INT-06 hoặc chính sách Windows/CAD. Không có thành phẩm native được bảo vệ.

**Cập nhật tiến độ V1.11 — 29/11/2026:** Chủ dự án duyệt cập nhật tiến độ và Lab V11. Baseline nghiên cứu đã kiểm thử Windows là V10; hệ thống TidiDcad V76, tồn đọng LicenseFormats V4/V5, 07/09/System Check giữ nguyên.

- V10 ZIP SHA-256 `db1ec0f2dcc97fc48fff636f5a9d73aea548d2d46c5ccd39b015b0f1baeef35e`, manifest 78/78 xác minh. Hai log Windows đã đọc: Build_All(20261128-174637).log SHA-256 `fe4028c8e8af08c12da5f63f24dbb6b04646a7f124ab600808df5c2797d2d115`; Test_All(6).log SHA-256 `1a0e95c549246005f118df6bd2db97832c9ea908d75d5c9103121a56074feb40`. Cùng Run-464996ae16ec45308e97104647533b2c, 13/13 CTest mỗi lần, exit 0, publication Lab và đối chiếu binary PASS; ReleaseDataIntegrity 12/12 tình huống trên dữ liệu mẫu. LIEF 593/593, Zydis/Zycore static, ABI/metadata/decoder hồi quy PASS. Chưa xác minh chữ ký production, native protection hoặc CAD.
- V11 source candidate SHA-256 `1de8d1d7564bdba0be4b3cc363898b9ebe7d3d38191487cac1cb65c08e1f7ee4`, CRC/manifest 89/89. Thêm duy nhất CTest ReleaseSignature, RSA-2048/SHA-256 PKCS#1 v1.5 trên chuỗi dữ liệu mẫu cố định; hai public key và chữ ký tạo độc lập, không lưu private key, không có chức năng ký hoặc nhận input bên ngoài. Linux OpenSSL hiện có, Windows CNG bcrypt (NotVerified). GCC và Clang ASan/UBSan Linux mỗi bộ 9/9, 13/13 ca signature, resolver 8/8, hai negative controls và đối chiếu public DER/CNG + chữ ký bằng OpenSSL CLI PASS. Không sửa core, PE, publish, thư viện hoặc khóa sản phẩm. Windows V11/14 CTest, Authenticode, chain/publisher, trust anchor production, key lifecycle, CAD, native protection NotVerified.
- Tổng hợp: PE Analyzer/Rebuilder, AES-GCM dữ liệu, VM số học mẫu, decoder x64, ABI/metadata, integrity đều có bằng chứng Windows trong phạm vi Lab V10. V11 bổ sung xác minh chữ ký dữ liệu mẫu trên Linux, chưa được chọn baseline Windows. Hồ sơ kiến trúc V10 là đề xuất, chưa phê duyệt production. Không có thành phẩm native được bảo vệ; chưa đủ bằng chứng về khả năng chống dịch ngược. Chưa đạt 10.3.5, chưa chuyển 10.4/10.5; giữ Gate/License/INT-06, Authenticode và Windows/CAD security, không sửa V76. Cảnh báo upstream/RemoteException còn mở.

**Cập nhật tiến độ V1.12 — 29/11/2026:** Chủ dự án duyệt Lab V12 và cập nhật tiến độ. Baseline nghiên cứu Windows là V11; TidiDcad V76, tồn đọng LicenseFormats V4/V5, 07/09/System Check giữ nguyên.

- V11 ZIP SHA-256 `1de8d1d7564bdba0be4b3cc363898b9ebe7d3d38191487cac1cb65c08e1f7ee4`, manifest 89/89 đối chiếu. Hai log Windows: Build_All(20261128-175312).log SHA-256 `685375398f9089bf6437e7c3654398b5949d4fc3affb34f2460090c5630a63ee`; Test_All(7).log SHA-256 `a021dbc214d4c0f443850bddeebd6dc4119e88ec5d7e0b5ef1c5d80a1e7ee69a`. Cùng Run-f50d8b7bb0c84bb387cf1b95fd20eb6a, 14/14 CTest mỗi lần, exit 0, Lab publish và đối chiếu binary PASS. RSA-2048/SHA-256 CNG 13/13 ca chữ ký mẫu; các ca hồi quy PASS. Production trust, Authenticode, native protection và CAD chưa xác minh.
- V12 source candidate SHA-256 `1684d528eb477e8d4c70adf26b08b27cc4b58df470b3fcbcf3cea46fd7f6bebd`, CRC/manifest 100/100. Thêm một CTest ReleasePackage tái dùng verifier V11; 20 tình huống trên hai file logic không thực thi cố định trong RAM. Chữ ký rời RSA-2048/SHA256, khóa public pin, schema/publisher/release mẫu, tập file và hash. Từ chối chữ ký không tin cậy, metadata sai dù ký hợp lệ, file bị tráo/thiếu/thừa/sửa. Không lưu private key, không ký, không đọc input ngoài, không thay PE/core/publish/dependencies/khóa sản phẩm. GCC Release và Clang ASan/UBSan Linux regression-only 10/10 mỗi bộ; package 20/20, resolver 8/8, ba negative controls và tám chữ ký mẫu kiểm chứng độc lập OpenSSL CLI PASS. Windows V12/15 CTest NotVerified.
- Tổng hợp: PE Analyzer/Rebuilder, AES-GCM dữ liệu, VM số học, decoder x64, ABI/metadata, integrity và RSA sample verification có bằng chứng Windows V11. V12 ghép kiểm tra bộ dữ liệu mẫu trên Linux, chưa chứng minh filesystem production, X.509 publisher, Authenticode, lifecycle, rollback thực tế hoặc khả năng chống dịch ngược. Hồ sơ kiến trúc V10 vẫn là đề xuất. Chưa có thành phẩm native được bảo vệ; chưa đạt 10.3.5, chưa chuyển 10.4/10.5. Giữ V76, Gate/License, INT-06, Authenticode và chính sách Windows/CAD. Cảnh báo upstream/RemoteException còn mở.

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

**Tại bên nhận:** bộ ZIP do chủ dự án trực tiếp giao là căn cứ tiếp nhận. Tính lại KeyId từ public header, đối chiếu JSON và kiểm tra chữ ký nội bộ theo INT-06. Không yêu cầu SHA-256 ZIP, KeyId hoặc bằng chứng xác nhận riêng. Chữ ký tự khớp với khóa trong ZIP không chứng minh độc lập nguồn phát hành; nếu đã có trust độc lập, đối chiếu với trust đó. KeyId thay đổi so với baseline phải được đánh giá trong phạm vi tích hợp đã giao; không tự đổi danh tính phát hành. Thay mật khẩu nhưng giữ KeyId không cần đổi public header. Phân biệt SEC06 ký Gate với khóa Licensecc cấp License.

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

Ghi ToolId, tên Tool, mô tả, toàn bộ Feature đang hoạt động, target, output, phiên bản Integration, Gate ABI, Policy, KeyId; danh sách sáu file; nguồn bàn giao trực tiếp và kết quả kiểm tra nội bộ theo INT-06. Xác định baseline Add-in, mục tiêu chỉ tích hợp bảo mật và những phần phải giữ nguyên. Nêu rõ target nào chưa hỗ trợ. Đọc STD-18 trước sửa.

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

### INT-06. Tiếp nhận và kiểm tra bộ bàn giao — V1.5 (thay thế thủ tục tiếp nhận V1.2)

1. TidiDcad giữ nguyên ký RSA-PSS SHA-256 salt 32 bằng PFX SEC06 hiện hành sau khi kiểm tra Tool/Feature, target, Header/Pin, trạng thái và snapshot; tự xác minh chữ ký, kiểm tra ZIP đọc lại trước công bố. Không tạo khóa mới; miền chữ ký riêng với Gate manifest. Hủy/sai tài khoản/pending/snapshot đổi/ký hoặc ghi lỗi thì bảo toàn ZIP cũ. Khi xuất thành công, chỉ cần giao ZIP cùng mã nguồn; SHA-256 toàn ZIP và kiểm tra theo Header/Pin tại TidiDcad vẫn xem được ở Kiểm tra ZIP, không phải đầu vào bắt buộc cho người lập trình.
2. Giữ đúng sáu file. `IntegrationVersion` giữ theo hợp đồng hiện hành (1.2 trên V76); bổ sung thành viên cuối `Certification` trong ToolInfo.json theo định dạng canonical:
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
3. Khi chủ dự án trực tiếp giao ZIP, đó là căn cứ tiếp nhận. Không hỏi lại SHA-256 ZIP, KeyId, nguồn phát hành hoặc yêu cầu file/script phê duyệt riêng. Người lập trình tự kiểm tra đúng sáu entry, không trùng/thoát thư mục, không rỗng/placeholder, ToolId, Feature, target, KeyId tính lại từ Header, ContentHash, PackageId và chữ ký RSA-PSS bằng public key trong gói. Nếu đã có trust độc lập, đối chiếu; tự khớp không phải xác thực độc lập nguồn gốc. Sai cấu trúc/hash/chữ ký/ánh xạ hoặc trust đã biết mâu thuẫn thì dừng, báo lỗi cụ thể. Nếu không được chủ dự án trực tiếp giao, phải có căn cứ tin cậy độc lập trước tiếp nhận. Gói chưa ký hoặc sai hợp đồng không được sử dụng.
4. Người lập trình hoàn thành tích hợp trước bàn giao: sao chép nguyên byte cả bốn file GateReleaseTrust.h, DcadLicenseGate.h, DcadGateClient.h, DcadGateClient.cpp vào nguồn (ví dụ src\Security), lưu ToolInfo.json trong project, cấu hình include/compile/link, ánh xạ từng Feature, rà điểm chặn và diff. Lưu ZIP gốc và kết quả tự kiểm tra. Kiểm tra đầy đủ file/tham chiếu project, khả năng build độc lập. Thiếu dữ liệu thiết yếu thì báo trước, không bàn giao nguồn thiếu file, không chuyển việc tích hợp sang chủ dự án. Tên ZIP nguồn theo STD-03/17, không tự thêm hậu tố SourceCandidate. Thiếu host kiểm thử ghi NotVerified trong QA, không coi là thiếu tích hợp và không tuyên bố PASS.
5. Không tạo ImportVerifiedIntegration.ps1 hoặc chuyển intake sang Build_All/Build_Release/MSBuild. Không yêu cầu ApprovedGateKeyId.txt, biến môi trường phê duyệt, Check-ReleaseTrust.ps1, verifier bổ sung ở máy build, SEC06 login, ExecutionPolicy mới, ZIP gốc, Gate DLL hoặc License khi compile. Bỏ các hook phê duyệt V1.1 còn sót; giữ các kiểm tra đầu vào build, SDK và mọi điểm chặn runtime. Nguồn bảo mật thay đổi phải được rà soát, kiểm tra lại trước build/phát hành; giữ nguyên byte bốn file từ ZIP trừ thay đổi hợp đồng được duyệt. Bảo toàn nghiệp vụ, settings, dữ liệu và quy trình ký thành phẩm.

### INT-07. Build_All.cmd — luồng bắt buộc

1. Xác định root từ script, target, đường dẫn SDK và output; tạo BuildRunId, log/binlog và staging riêng từng lần; không lấy log cũ. Nếu thiếu/sai đầu vào, dừng trước compile/publish, ghi lỗi riêng lần build. Không thao tác vào dữ liệu vận hành.
2. Kiểm tra nguồn/project đã tích hợp đầy đủ bốn file bảo mật, metadata, ID/Feature, KeyId và toolchain/SDK; không thực hiện lại intake hoặc yêu cầu ZIP/xác nhận riêng.
3. Gọi Build_Release.cmd cho từng target đã duyệt, truyền thông số chính xác, nhận exit code. Không lấy output cũ để bù khi lỗi; MSBuild nonzero phải dừng. Xác minh output tạo mới, không rỗng; không dùng verification build để hợp thức hóa artifact cũ.
4. Publish đúng tên thành phẩm theo quy ước, tạo thư mục nếu thiếu; kiểm tra nội dung file đích bằng SHA-256 hoặc so sánh byte. Không phá hủy bản đã phát hành chỉ vì build mới thất bại; dùng staging riêng, chỉ công bố kết quả đủ điều kiện.
5. Log chính ghi baseline, target, ToolId, Feature, KeyId, SDK root, toolset, bước build, output, kết quả và lỗi; log target tách riêng. Exit 0 chỉ khi build/publish đúng phạm vi; lỗi nonzero. Không coi exit 0 là APPLOAD/Runtime PASS.

**Khác mẫu V38:** bỏ phụ thuộc đồng bộ từ máy TidiDcad trong `Sync-ReleaseTrust.ps1` của Build_All khi bộ Integration đã chứa Public Key thật; kiểm tra và hoàn tất tích hợp trước bàn giao theo INT-06. V38 còn xóa output staging trước khi build và có trạng thái SETUP REQUIRED (exit 2); quy ước mới yêu cầu không mất bản phát hành cũ, thiếu khóa bàn giao phải FAIL rõ nguyên nhân. Không tạo PFX/khóa, không đăng nhập, không yêu cầu Dcad.lic, không build/copy DLL hoặc manifest Gate. Không tự chạy ứng dụng hoặc mở Explorer. Có thể dùng `/nopause` cho build tự động.

### INT-08. Build_Release.cmd và SDK

Ví dụ AutoCAD 2022: reference `D:\000\Reference\Autocad 2022`; ObjectARX 2022 x64; Visual Studio/MSBuild có MSVC x64, v142 ưu tiên. V38 hỗ trợ v143/v145 thay thế hoặc `DCAD_MSVC_TOOLSET`, nhưng phải nghiệm thu CAD trên toolset thực tế. Có thể dùng `DCAD_REFERENCE_ROOT` khi được phê duyệt; không dùng file SDK trộn từ nhiều root.

`Buildtime/ResolveC22SDK.ps1` V38 kiểm tra arx.props/dbx.props, header và library cần thiết; script là nguồn danh sách chính xác cho baseline, không áp đặt cho target khác. Tìm một root SDK đầy đủ, file không rỗng; nếu nhiều bộ đầy đủ nhưng nội dung khác thì dừng; nếu giống nhau từng byte thì có thể chọn nhất quán. Không mở hai luồng ghi đồng thời vào cùng log (V38 SDK resolver tự ghi log).

Lệnh mẫu V38:

```cmd
"%MSBUILD%" "%BASE%DJoin.vcxproj" /t:Rebuild /p:Configuration=Release /p:Platform=x64 "/p:PlatformToolset=%TOOLSET%" "/p:ARX2022_SDK=%SDKROOT%" /m:1 /nologo "/flp:LogFile=%LOG%;Verbosity=normal;Append"
```

Không đóng gói SDK dùng chung vào ZIP nguồn. Output V38: `bin\DJoin.arx`; intermediate `Buildtime\obj\`; staging root `Dprogram\AutoCAD2022\DJoin.arx`; log chính `log\Build_All.log`, log target `log\Build_AutoCAD2022.log` trong thư mục target. Giữ bố trí theo quy ước hiện hành khi thực hiện. Không liên kết nhầm SDK/host. Build_Release dùng trực tiếp nguồn đã tích hợp đầy đủ, không yêu cầu intake; giữ kiểm tra build gốc và mọi điểm chặn runtime.

### INT-09. Cập nhật README, BuildInfo và dữ liệu QA

README của Add-in phải có: tên/phiên bản/baseline; ToolId và Feature → command/action; CAD target; Security Contract (Gate version, ABI, Policy, License format); KeyId được nhúng, căn cứ tiếp nhận và kết quả kiểm tra nội bộ; các file tích hợp; điểm chặn; SDK/toolset/Windows SDK; cách chạy Build_All; output/log; cài đặt DProgram/DUser; hành vi ALLOW/DENY; cách xử lý thiếu file hoặc khóa không khớp; Known Issues và trạng thái Build/APPLOAD/Runtime riêng. Cập nhật BuildInfo, DataManifest, QA và lịch sử phiên bản theo STD-16/17 khi phù hợp. Không thêm README thứ hai vào ZIP Integration.

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
4. Báo rõ xuất thành công hay lỗi cụ thể (Tool đã retire, Feature thiếu, KeyId chưa duyệt, phiên bản client không phù hợp, v.v.). Khi xuất thành công, chỉ hướng dẫn giao ZIP cùng mã nguồn, không yêu cầu sao chép mã xác nhận. Xuất ZIP PASS chỉ là nghiệm thu bộ dữ liệu tích hợp, không phải nghiệm thu Add-in đích.

Không tạo thêm schema database hoặc phần mềm quản lý khóa; tái sử dụng khóa SEC06 hiện có, tách miền chữ ký bộ bàn giao. Nếu cần mở rộng dữ liệu CAD target, phải duyệt riêng phương án; V1.0 có thể lấy từ thông tin target đã xác nhận khi xuất.

## 10. Ma trận QA bắt buộc theo phạm vi

| Mã | Điều kiện | Expected |
|---|---|---|
| EXP01 | Tool có một/nhiều Feature đang hoạt động | đủ sáu file, đúng ID và dữ liệu |
| EXP02 | Tool/Feature retire, thiếu hoặc xung đột ID | không xuất bộ sai |
| EXP03 | Header chưa enrollment, pin sai, pending transaction | từ chối xuất |
| EXP04 | Gói ZIP lỗi, ghi dở, trùng đích | báo lỗi, không làm mất gói hợp lệ |
| EXP05 | Kiểm tra thành phần | không có khóa riêng, DLL, manifest, License hoặc database |
| QA01 | Build nguồn đã tích hợp hoàn chỉnh, không cần ZIP hoặc cài TidiDcad | PASS |
| QA02 | Build thiếu Gate DLL/manifest/License runtime | PASS |
| QA03 | Thiếu bất kỳ file bảo mật hoặc public header #error | Không bàn giao nguồn thiếu file; build FAIL nếu dữ liệu hỏng |
| QA04 | Public header/KeyId không khớp, hash hoặc chữ ký sai | Dừng tích hợp |
| QA05 | ZIP hợp lệ do chủ dự án trực tiếp giao, không có mã xác nhận ngoài ZIP | Tự kiểm tra và tích hợp; không yêu cầu xác nhận thêm; không tuyên bố xác thực độc lập |
| QA06 | Feature sai cú pháp, sai Tool, không đúng ánh xạ | Từ chối tích hợp |
| QA07 | Build trực tiếp nguồn đủ bốn file bảo mật, không có ZIP hoặc script/phê duyệt máy | Không cần intake; giữ kiểm tra build gốc |
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
| Xác minh/xuất public key và bộ Integration | Chịu trách nhiệm; chủ dự án trực tiếp giao ZIP | Tự kiểm tra nội bộ, hoàn thành tích hợp trước bàn giao; không yêu cầu xác nhận ngoài ZIP |
| Xác định ánh xạ Feature → chức năng, CAD target | Phối hợp/phê duyệt | Khảo sát và thực hiện |
| Tích hợp client, project, Build_All, README | Cung cấp chuẩn | Chịu trách nhiệm |
| Kiểm thử build và CAD, hồi quy | Phối hợp cung cấp runtime/license QA | Chịu trách nhiệm và báo cáo |
| Nghiệm thu/phát hành chính thức | Phê duyệt | Bàn giao bằng chứng |

## 11A. Nghiệm thu bổ sung INT-06 V1.5

| Mã | Điều kiện | Expected |
|---|---|---|
| CERT01 | Xuất hợp lệ với khóa SEC06 hiện hành | Sáu file, Certification hợp lệ, chữ ký đúng |
| CERT02 | Hủy/sai mật khẩu, khóa thiếu, pending hoặc snapshot đổi | Không công bố, bảo toàn ZIP cũ |
| CERT03 | Sửa bất kỳ entry, chữ ký, mã, thêm file/repack | Xác thực FAIL |
| CERT04 | Gói ký bằng danh tính mâu thuẫn với trust độc lập đã biết | Từ chối; không tự thay trust |
| CERT05 | Kiểm tra ZIP đúng bằng public trust hiện hành, không PFX | PASS |
| CERT06 | ZIP hợp lệ do chủ dự án trực tiếp giao, không kèm hash riêng | Tự kiểm tra; không yêu cầu xác nhận ngoài ZIP |
| CERT07 | Build nguồn đã tích hợp đầy đủ, không cần ZIP, file/script phê duyệt máy | Không cần intake/cấu hình Windows bổ sung; giữ luồng build gốc |
| CERT08 | Build/publish lỗi sau một lần PASS | Không dùng log/artifact cũ, bản cũ còn nguyên |

Tiêu chí trên chưa phải kết quả thực thi Windows/CAD. V55 dùng khóa SEC06 hiện có; không thay Gate/Licensecc, khóa, database và runtime. Phê duyệt máy build V1.1 hết hiệu lực; thủ tục tiếp nhận V1.2 được thay bằng INT-06 V1.5. Không tự xóa file cũ của người dùng.

## 12. Căn cứ nguồn và giới hạn hiện tại

Đã kiểm tra trực tiếp ba tệp sau để lập đặc tả (SHA-256 giúp nhận diện nguồn, không phải chữ ký xác thực nguồn):

- `DJoin-V38 (1).zip`: `ae97bd564e9d8ff0f36daf3bc519b70d24ff8680e3b397686a3de4cbdc4b8043`. Tham chiếu Build_All.cmd; Build_Release.cmd; DJoin.vcxproj; Userdata/README.txt, QA-STATUS.txt; src/DJoin.cpp, SettingsNative.cpp, DJoinFeature.h, LicenseGate.*, GateReleaseVerifier.*, GateReleaseTrust.h; Buildtime/Check-ReleaseTrust.ps1, Sync-ReleaseTrust.ps1, ResolveC22SDK.ps1 và QA.
- `TidiDcad-System-V49.zip`: `2c805de957636add22d602778ad553c6ee69b16d7ed9d5890d7867605bb7ed5a`. Tham chiếu Source/ReleaseTrust/PortableSecurity.ps1 (public trust, KeyId, manifest) và Source/DcadLicenseGate/Source/DcadLicenseGate.h (API V6).
- `ToolCatalog.db`: `2100431221130d76bcb82af00e53c3505804aa328b104573bee6dcc8c2c08e5f`. Đã đọc schema tools/features và bản ghi Djoin.

V1.2 thay cơ chế phê duyệt máy build V1.1 bằng chữ ký bộ bàn giao, triển khai trong nguồn TidiDcad V55. Không dùng placeholder làm khóa thật. QA-STATUS V38 ghi MSVCBuild, APPLOAD, Runtime là NotVerified; không dùng nguồn này làm chứng cứ đã nghiệm thu. Windows signing, GUI, Build_All và CAD runtime của V55 cần nghiệm thu riêng. Việc cập nhật quy ước không thay thế kiểm chứng phần mềm.
