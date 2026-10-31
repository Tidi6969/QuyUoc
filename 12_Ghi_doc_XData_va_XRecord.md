# DCAD-STD-12 — Ghi, đọc XData và XRecord

**Phiên bản:** 1.5  
**Mức áp dụng:** Theo tính năng

## Phạm vi schema

- Các quy tắc bên dưới mô tả **hợp đồng XData DPunch-V381** để ứng dụng CAD khác ghi/đọc tương thích.
- **Writer mới** dùng canonical schema V381; **reader** được phép đọc dữ liệu cũ bằng nhánh legacy riêng, không viết lại DWG khi chỉ đọc.
- Mọi module khác phải tự xác định RegApp, owner, carrier, schema/version và quyền ưu tiên XData/XRecord; không mặc định đã triển khai giống DPunch.

### 1. Hợp đồng `MY_TAG`, `NAME` và `HOLE_TYPE` của DPunch-V381

| Nội dung | Cách lưu | Quy tắc |
|---|---|---|
| RegApp | `1001 MY_TAG` | Đăng ký trong `RegAppTable` trước khi ghi |
| `NAME` | `1000 NAME` rồi **một** `1000 <payload>` | Có thể có nhiều cặp trên MASTER |
| `HOLE_TYPE` | **Một** `1000 "HOLE_TYPE <process> <canonicalHoleType> <PointAngle>"` | Không tách marker và payload thành 2 giá trị 1000 |
| `MATER`, `LENGTH` | Marker + payload | Khi business object cho phép |
| `CUTEDGE` cũ | Marker + payload | **Reader legacy**; writer lỗ V381 dùng NAME Tok14/Tok15 |

**Ba chiều phân loại không được nhầm:** `NAME.Tok0` = nhóm entity/record; **`NAME.Tok1` của MASTER** = mã chốt/part; **`NAME.Tok1` của Hole** = mã quy trình gia công; **`HOLE_TYPE.Tok2`** mới là canonical HoleType. Không phân loại chỉ theo layer hoặc Tok1.

### 2. Nhóm entity theo `NAME.Tok0` trong V381

| Tok0 | Nhóm | Tok2 có ý nghĩa gì? | Lưu ý |
|---|---|---|---|
| `1` | Circle Hole (Circle/Arc) | **Đường kính**, số dương | Không được coi Tok2 là STT |
| `3` | Standard Part / MASTER AP-BP / metadata phụ | Theo từng loại bản ghi | Không nhất thiết là entity chi tiết mới |
| `5` | Profile Hole | **STT**, số nguyên ≥1 | Có thể dùng để quét STT CP khi đúng `CP1` |
| `7` | Insert (CP1/CP4) | Theo contract của subtype | CP1 là MASTER, CP4 có schema riêng |
| `8` | Plate | Hằng số có khai báo | **Chưa xác nhận DPunch-V381 tự ghi** entity tấm kiểu này |

**Reader tương thích:** DPunch-V356 dùng Tok0=5 cho các Hole nói chung, nhóm phụ MASTER `10/11`; V381 phân biệt **1=Circle**, **5=Profile**, và writer MASTER dùng **`HOLE_TABLE`/`PROJECTION`** có tên. Reader V381 có nhánh đọc 10/11 legacy, nhưng writer mới **không phát sinh** các nhóm phụ số này.

### 3. Mã chốt, carrier và MASTER

- MASTER AP: `Tok0=3`, `Tok1=ap1`…`ap12`; BP: `Tok1=bp1`/`bp2`. Những mã `ap-a1`, `bp-b1`, `ap-sh1`, `ap-st1`, `AP31`… chỉ phục vụ **reader legacy**; không phát lại khi writer V381 ghi mới.
- MASTER AP/BP: `Tok2=D`, `Tok3=L`, `Tok4=P`, `Tok5=B`, `Tok6=H`, `Tok7=T`, `Tok8=dx`, `Tok9=W`, `Tok10=R`, `Tok11=K`, `Tok12=ProfileAngleDeg`, từ `Tok13` là dự phòng, **đệm tối thiểu 14 token**. Các trường không thuộc đúng `PunchFieldSchema` của biến thể phải ghi `.`; không suy từ tên cột UI.
- MASTER CP1: **đúng 7 token**: `7 CP1 <STT> <L> 0.00 . <Plate>*<StartDepth>`; **Tok5 là `.`**; legacy reader có thể gặp dữ liệu cũ khác format. Không đệm CP lên 14/16 token. Chốt CP MASTER Tok0=7 **không phải** entity dùng để quét STT CP theo V381.
- Thân bạc CP4 dùng Tok0=7, Tok1=CP4, schema **16 token riêng**; lỗ gia công bạc là entity độc lập. Không tự ép CP4 thành HoleType hoặc schema chung của mọi command.
- MASTER lưu `HOLE_TABLE` và `PROJECTION` bằng các **cặp `NAME`/payload**. Reader V381 yêu cầu có bản ghi chốt **và** `PROJECTION` (hoặc projection legacy 11) mới nhận là MASTER hoàn chỉnh. Chỉ thấy `3 ap1 ...` thì **không** kết luận MASTER V381 đã hợp lệ.
- `HOLE_TABLE`: `Tok0=HOLE_TABLE`, Tok1 Layer, Tok2 canonical HoleType, Tok3 DepthRaw, Tok4 GapRaw, Tok5 ThicknessRaw, Tok6 Note, Tok7 SpacingAfter, Tok8 CutEdge, Tok9 ReliefAngle, Tok10–13 `.`. `PROJECTION`: `Tok0=PROJECTION`, Tok1 BasePlate, Tok2 StartDepth, Tok3 `TD/BU`, Tok4 DistanceRatio, Tok5–13 `.`. Chúng là **snapshot cấu hình**, không phải lỗ gia công thật. `EJECTROD0`/`SET_SCREW=M...` Tok0=3 trong MASTER là metadata phụ, không tự suy ra một entity vật lý.

### 4. `NAME` của một lỗ V381 — chuẩn 16 token

| Token | Ý nghĩa V381 | Điều kiện |
|---|---|---|
| Tok0 | `1` Circle/Arc hoặc `5` Profile | Theo hình học entity |
| Tok1 | Process/family token gia công | Sinh từ role, Gap, Depth và mã nguồn Punch, không phải HoleType |
| Tok2 | **Circle: đường kính** / **Profile: STT** | Không dùng chung parser ý nghĩa |
| Tok3 | Depth | Số invariant, theo writer |
| Tok4 | Gap/khe hở/cỡ ren | `.` nếu không có |
| Tok5 | Note | Khoảng trắng ASCII **`' '` → `$`**; không có → `.` |
| Tok6 | Tương thích CP1 | Profile CP1: `&` + Tok4; khác: `.` |
| Tok7 | Dự phòng | `.` |
| Tok8 | Tương thích lỗ không phải CP1 | `&` + Tok4; CP1 Profile: `.` |
| Tok9 | Dự phòng | `.` |
| Tok10 | Dung sai trên | `.` nếu chưa có nguồn |
| Tok11 | Dung sai dưới | `.` nếu chưa có nguồn |
| Tok12 | Dự phòng | `.` |
| Tok13 | PointAngle | **Phải khớp HOLE_TYPE.Tok3**, sai lệch > 1e-6: reader từ chối |
| Tok14 | CutEdge | `.` trừ dạng có CutEdge, ví dụ `user.slug_angle` |
| Tok15 | ReliefAngle | `.` trừ dạng có relief; **khác PointAngle** |

**Codec chú ý:** `EncodeNoteToken` hiện chỉ chuyển dấu cách ASCII thành `$`, và khi đọc `$` thành dấu cách; dấu `$` nguyên gốc và Note chỉ bằng dấu `.` có hạn chế round-trip; không tự thay codec khi chưa có quy tắc migration. Đối với **các token thông thường** (không phải Note), quy tắc encode whitespace dùng `&`; không lấy một codec áp cho mọi field. Không tự sửa codec lịch sử khi chưa có hợp đồng di trú.

### 5. Vai trò `HOLE_TYPE` và Token1 gia công

- Chuỗi chuẩn: `HOLE_TYPE <process> <17 canonical key> <PointAngle>` trên **một** giá trị DXF 1000; HoleType thực ở vị trí **Tok2** của tag, không phải Tok1 của `NAME`.
- Quy tắc writer: `thread/mscrew` dùng token ren động; các type `head.sink`/`head_profile.sink`/`head.sink_profile` dùng `g_sink`; type khác có Depth hoặc PointAngle dương dùng `f_sink`; còn lại theo catalog/fallback của entity. `HOLE_TYPE.Tok1` **không bắt buộc** bằng `NAME.Tok1`.
- `NAME.Tok1` gia công: Gap có hậu tố `W/T` → tiền tố `pos`, hậu tố `R` → `genr`, còn lại → `gen`; sau đó role/chiều sâu quyết định `G_sink`, `F_sink`, `b_sink`, `s_sink`, `+`, v.v. `CP1` là ngoại lệ ưu tiên giữ đúng `CP1`. **Dùng builder chuẩn** thay vì nối chuỗi từ ví dụ.
- Reader hiện tại kiểm tra `NAME.Tok13 == HOLE_TYPE.Tok3` trong tolerance `1e-6`. Với `user.slug_angle`, **một** lỗ thật có XData, outline mặt đối diện không tính thành Hole thứ hai. CutEdge/ReliefAngle nằm ở `NAME.Tok14/15`; `CUTEDGE` riêng là **reader legacy**.

### 6. Quy trình Write / Read, XRecord và giới hạn kiểm toán

1. **Write:** phân loại entity + geometry → normalize chốt/HoleType → validate typed model → xây payload từ **một builder** → đăng ký RegApp và gắn XData trong transaction/DocumentLock thích hợp → đọc lại round-trip. Không tạo XData Hole trên geometry mô tả/preview.
2. **Read:** `RawXData → ParseEnvelope → DetectSchema → Validate → NormalizeLegacy (chỉ khi schema cũ hợp lệ) → BusinessObject`. Với lỗ **đang ở đúng shape V381 (16 token, Tok0=1/5)** nhưng sai Tok2/Tok13/Tok14/Tok15, **reject**, không fallback xuống legacy để lách validation.
3. **Compatibility reader-only:** giữ alias HoleType và mã AP/BP cũ; không nâng cấp DWG hàng loạt, không viết legacy mới.
4. **XRecord:** Hợp đồng DPunch-V381 được mô tả ở đây sử dụng XData `MY_TAG`; không có chứng cứ trong phạm vi schema này cho writer/reader XRecord `DCAD_DPUNCH`. giao thức xác nhận ở đây là XData `MY_TAG`. Khả năng có XRecord trong module/version khác **cần audit riêng**. Không mặc định `DCAD_DPUNCH` là nguồn ưu tiên của V381 hoặc mặc định rằng toàn DCAD không dùng XRecord.
5. **STT CP:** giữ nguyên luật V381 — chỉ quét `NAME` của lỗ **Profile `5 CP1`** trong layout và lấy lớn nhất + 1; **không** mở rộng quét MASTER `7 CP1` khi chưa có yêu cầu riêng.

**Kiểm thử XData cần thực hiện:** T01–T20 gồm kiểm tra AP1–AP12/BP1–BP2/CP1/CP4; Circle Tok0=1 và Profile Tok0=5; 17 keys; Tok13 đồng nhất PointAngle; codec `$`/`&`; HOLE_TABLE/PROJECTION; legacy; XRecord claim; round-trip trên từng host. Không ghi `PASS` nếu chưa kiểm thử trên CAD thực tế.

**Ví dụ shape theo chuẩn XData V381 (tách các ví dụ độc lập, không tự nối thành một entity):**

```text
NAME = 1 genAP 10.407 0.000 0.2W . . . &0.2W . . . . 0 . .
HOLE_TYPE <process> <canonicalHoleType> <PointAngle>
```

Dòng `NAME` là **ví dụ test của BuildHole**, không phải khẳng định Tok1 khi vẽ AP V381. Dòng `HOLE_TYPE` minh họa cấu trúc, không phải tag được tạo từ dòng NAME phía trên. **Trong sản phẩm thật, phải để builder xây hai trường từ cùng một model và kiểm tra PointAngle đồng nhất.**
