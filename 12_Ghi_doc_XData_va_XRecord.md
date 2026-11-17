# DCAD-STD-12 — Ghi, đọc XData/XRecord và khôi phục hình học dùng chung

**Phiên bản:** 1.6  
**Ngày cập nhật:** 17/11/2026  
**Mức áp dụng:** BẮT BUỘC khi module ghi/đọc XData hoặc tham gia khôi phục bằng Drestore.  
**Nguồn thiết kế:** MASTER XData Dtscr1 V84 (hợp đồng TSCR0 schema 4); tham chiếu DPunch V381/V382 ở Phụ lục A.  
**Trạng thái áp dụng:** Chuẩn cho các writer/reader **được triển khai mới**. Việc cập nhật tài liệu không tự nâng schema trong DLL/ARX hiện có, không sửa DWG cũ.

## 1. Mục tiêu và bất biến

1. Lấy mô hình **PRIMARY → SCHEMA → META → PROJECTION → ROW** của Dtscr1 làm chuẩn đóng gói MASTER. Phần riêng của DPunch phải được đặt trong META/EXT có đặc tả, không ép sai nghĩa trường.
2. Mỗi loại chi tiết có định danh **Tok0 + Tok1** riêng. Dtscr1: `3 TSCR0` (và `3 TSCR` khi reader đã hỗ trợ); DPunch AP/BP: `3 ap1…ap12`, `3 bp1/bp2`; CP1/CP4 dùng `7 CP1/CP4` theo hợp đồng subtype. Không dùng `3 TSCR0` cho DPunch.
3. RegApp XData: `1001 MY_TAG` (so sánh RegApp theo hành vi API CAD); các bản ghi MASTER theo cặp `1000 NAME` rồi `1000 <payload>`. Đăng ký RegApp trước khi ghi, bảo toàn mọi XData khác của đối tượng.
4. **Chỉ dòng lỗ hợp lệ và đã vẽ thành công mới được ghi vào MASTER**. Dòng trống/thiếu/sai/bị bỏ qua không phát sinh ROW hoặc EXT. Giữ RowIndex/STT gốc; `ValidRows` phải bằng số ROW đã lưu.
5. XData trên từng **lỗ gia công thật** và MASTER có vai trò khác nhau. Không lấy nét mô tả/preview/hình chiếu làm một lỗ gia công; không xóa XData lỗ riêng của DPunch khi chuẩn hóa MASTER.
6. **Writer mới** phát ra một schema chuẩn đã được nghiệm thu; **reader** giữ nhánh nhận diện V1–V4 của Dtscr1 và các dữ liệu V381/legacy của DPunch đúng phiên bản. Đọc/khôi phục không sửa tự động đối tượng nguồn hoặc nâng cấp DWG.
7. Màu, Linetype, layer, hình học, thông số kỹ thuật phải cùng quy tắc và cùng nguồn thuật toán với add-in gốc. Chỉ đổi cấu trúc XData khi có yêu cầu riêng và phép thử hồi quy.

## 2. Bố cục MASTER theo thứ tự

| Thứ tự | Loại payload `NAME` | Vai trò | Bắt buộc |
|---|---|---|---|
| 1 | PRIMARY: `<Tok0> <Tok1> ...` | Nhận diện đúng loại part/entity và thông số chính | Có |
| 2 | `DCAD_SCHEMA ...` | Schema chung, creator, part, số dòng | Có với writer chuẩn mới |
| 3 | `<PART>_META ...` | Thông số đặc thù của loại chi tiết | Theo part |
| 4 | `PROJECTION ...` | Cấu hình hình chiếu | Khi part hỗ trợ/đòi hình chiếu |
| 5 | `DCAD_ROW ...` | Một dòng lỗ hợp lệ | 0 hoặc nhiều |
| 6 | `<PART>_ROW_EXT ...` | Thông số không nằm trong ROW chung | Theo part, liên kết RowIndex |

Các bản ghi được sắp xếp ổn định. Không được nhầm số token/ý nghĩa giữa PRIMARY, ROW và payload của Hole riêng. Không bắt toàn bộ PRIMARY dài 14 token: Dtscr1 hiện có hình dạng riêng; CP1 hiện có đúng 7 token trong V381.

### 2.1. PRIMARY — định danh

- Dtscr1 giữ nguyên định danh MASTER `Tok0=3, Tok1=TSCR0`; không đổi thành `DCAD` hoặc đổi Tok1 trong bản vẽ cũ.
- Dtscr1 PRIMARY ví dụ: `3 TSCR0 10.00 20.00 0.000 0.00 0.00 .. ..`. Token 2 là M danh nghĩa, token 3 là L; các token khác diễn giải theo parser TSCR0 thực tế.
- DPunch AP/BP giữ Tok0=3 và part code theo subtype; thông số D/L/P/B/H/T/dx/W/R/K/ProfileAngleDeg vẫn thuộc hợp đồng riêng, không ép thành M/L của Dtscr1.
- DPunch CP1/CP4 giữ Tok0=7 của dữ liệu cũ và adapter/schema riêng; không sửa công thức STT, quyền XRecord hoặc quan hệ với lỗ thật chỉ để thống nhất tên record.
- Reader nhận diện bằng `RegApp + Tok0 + Tok1 + loại entity + schema + dữ liệu liên quan`. Chỉ thấy `Tok0=3` hoặc tên layer chưa đủ để kết luận là MASTER hợp lệ.

### 2.2. DCAD_SCHEMA — 14 token

| Tok | Trường | Ràng buộc |
|---|---|---|
| 0 | `DCAD_SCHEMA` | Marker cố định |
| 1 | `CommonSchema` | Bắt đầu ở `1` cho hợp đồng chung |
| 2 | `Creator` | `DTSCR1`, `DPUNCH`, ...; mã ASCII chính thức |
| 3 | `PartType` | Phải khớp PRIMARY.Tok1 |
| 4 | `PartSchema` | Phiên bản schema riêng của part; không phải số version ứng dụng |
| 5 | `TotalRows` | Số dòng bảng nguồn, nguyên không âm |
| 6 | `ValidRows` | Đếm đúng số `DCAD_ROW`, nguyên không âm |
| 7–13 | Reserved | `.` |

Ví dụ **dạng đề xuất khi TSCR0 writer được nâng cấp**, không phải định dạng hiện hành của V84:

`DCAD_SCHEMA 1 DTSCR1 TSCR0 5 5 3 . . . . . . .`

Ở đây `PartSchema=5` là phiên bản *dự kiến* cho writer TSCR0 áp dụng hợp đồng chung; Dtscr1 V84 vẫn dùng `TSCR0_SCHEMA 4`. Không tự đổi dữ liệu phát hành trước đó. Reader phải từ chối schema không hỗ trợ, Creator/PartType mâu thuẫn, số ROW sai, RowIndex trùng hoặc vượt [0,TotalRows).

### 2.3. META — thông số riêng của part

Dtscr1 V84 sử dụng `TSCR0_META M10 20.0 MANUAL 1 0 1.5 . . . . . . .` (14 token). Những trường hiện có của TSCR0 META giữ đúng ý nghĩa và cách kiểm tra trong mã nguồn TSCR0.

Writer chung dùng `<PART>_META` với hợp đồng token xác định riêng theo part (ví dụ AP/BP/CP). Không dùng các vị trí META của TSCR0 để diễn giải tham số góc mũi, biên dạng, K/R/W hay đặc tính CP4 của DPunch. Nếu thiếu hợp đồng subtype thì **chưa cho phép phát hành writer subtype mới**.

### 2.4. PROJECTION — cấu trúc Dtscr1 làm mẫu

Ví dụ `PROJECTION UP 0 BU 1 . . . . . . . . .` với Tok0=PROJECTION, Tok1=tấm gốc, Tok2=StartDepth, Tok3=hướng `TD/BU`, Tok4=DistanceRatio, Tok5–13 dự phòng.

- Chỉ được dùng nếu loại chi tiết có hình chiếu hợp lệ theo thuật toán gốc.
- DPunch V381 có record PROJECTION riêng; adapter tương thích phải diễn giải theo phiên bản nguồn.
- Nếu schema của part yêu cầu PROJECTION nhưng dữ liệu sai/thiếu thì từ chối MASTER, không đoán theo hình học/layer.

### 2.5. DCAD_ROW — 14 token

| Tok | Trường | Yêu cầu |
|---|---|---|
| 0 | `DCAD_ROW` | Marker |
| 1 | `RowIndex` | STT gốc trong bảng, nguyên không âm, không trùng |
| 2 | `Plate` | Mã tấm/layer hợp lệ |
| 3 | `HoleType` | Mã loại lỗ đã được xác thực |
| 4 | `Depth` | Chiều sâu, có thể `.` nếu subtype cho phép |
| 5 | `Gap` | Khe hở, có thể `.` khi không áp dụng |
| 6 | `Thickness` | Chiều dày tấm có nguồn hợp lệ |
| 7 | `Note` | Ghi chú có codec theo schema |
| 8 | `PlateP` | Cờ `_P` (0/1), theo ngữ nghĩa Dtscr1 |
| 9 | `Offset` | Dịch chuyển nếu có |
| 10 | `SecondaryType` | Phân loại phụ nếu có |
| 11–13 | Reserved | `.` |

Ví dụ: `DCAD_ROW 0 UP fix -25 . 35 . 0 . . . . . .`

`.` không mặc định là lỗi; giá trị `0` không tự chứng minh hàng hợp lệ. Với mỗi subtype, phải có danh sách trường bắt buộc và luật hình học rõ. Không tự chuyển dữ liệu lỗi thành số 0 để lách kiểm tra. Giữ nguyên **RowIndex gốc** khi bỏ qua hàng khác. Writer chỉ ghi các ROW đã xác nhận thành công trong cùng transaction tạo hình.

### 2.6. EXT — dữ liệu mở rộng

Đề xuất tên `DPUNCH_ROW_EXT` cho những hàng DPunch có thông số mà ROW chung không bao quát: process, PointAngle, CutEdge, ReliefAngle, SpacingAfter, vai trò lỗ, tham chiếu biên dạng, v.v. **Đây là danh sách trường cần kiểm toán, không phải token map đã nghiệm thu.**

- EXT bắt buộc ghi RowIndex gốc để tham chiếu một ROW thực có; không có ROW tương ứng thì không được ghi EXT.
- Mỗi subtype phải có phiên bản và token map riêng trước khi triển khai; không cắt bớt CP4 16 token hoặc các trường đặc biệt cho vừa ROW chung.
- Giữ HOLE_TYPE, Process và dữ liệu trên entity lỗ thật theo hợp đồng của nguồn; `NAME.Tok1` trên Hole DPunch không phải canonical HoleType.
- Lỗ Circle Tok0=1/Tok2=đường kính; Profile Tok0=5/Tok2=STT theo V381. Không áp quy tắc RowIndex của MASTER lên NAME của entity lỗ.

## 3. Quy trình writer — vẽ và lưu

1. Đọc và kiểm tra dữ liệu đầu vào, phân loại từng dòng độc lập.
2. Dùng **chính thuật toán của add-in gốc** để xác định hàng hợp lệ và dựng hình; hàng không thể vẽ thì bỏ qua, không chặn các hàng hợp lệ khác.
3. Chỉ sau khi tạo thành công mới đưa hàng vào danh sách ROW/EXT của MASTER. Đếm `ValidRows` từ danh sách thực lưu, không lấy tổng hàng bảng.
4. Sinh PRIMARY, SCHEMA, META, PROJECTION và ROW/EXT bởi các builder tương ứng; mã hóa dữ liệu/giới hạn độ dài, kiểm tra toàn bộ trước khi commit.
5. Gắn XData với RegApp đã đăng ký, không phá XData khác; dùng transaction/Undo và bảo đảm không để hình nửa chừng nếu MASTER không ghi được.
6. Đọc lại để kiểm chứng round-trip. Không ghi XData Hole lên đường mô tả, Preview hoặc hình chiếu không phải lỗ gia công thật.

**Giới hạn kỹ thuật:** mỗi giá trị DXF 1000 tối đa 255 byte; kiểm tổng dung lượng XData theo giới hạn CAD, không cắt chuỗi. XData quá lớn phải báo FAIL và dùng XRecord chỉ sau khi có hợp đồng reader/writer/owner/priority riêng. Không tự viết trùng dữ liệu MY_TAG vào XRecord để vượt giới hạn mà không có migration. Dữ liệu UTF-8 phải dùng codec versioned và giải mã đối xứng; **không áp một codec lên mọi token**: TSCR0 và DPunch V381 có các quy tắc Note khác nhau (`$`, `&`, dấu `.`).

## 4. Quy trình reader và Drestore

`Raw XData → Parse RegApp/NAME → PRIMARY (Tok0/Tok1) → Detect schema → Validate envelope → Parse META/PROJECTION/ROW/EXT → Validate subtype → Geometry model → Restore`.

- MASTER sai định danh, sai schema, sai số ROW, thiếu metadata toàn cục: từ chối toàn bộ; không tự đổi lỗi thành mặc định và không dựng hình nửa vời.
- ROW độc lập sai hoặc thiếu thông số: bỏ qua ROW đó cùng EXT của nó, không chặn các ROW hợp lệ khác; báo số đã đọc/đã bỏ qua.
- Không dùng fallback legacy để lách validation của một schema mới bị hỏng.
- Drestore không chỉnh sửa đối tượng nguồn; tạo hình ở vị trí mới; hỗ trợ chế độ `H=Lỗ,P=Hình chiếu` **chỉ cho subtype có bộ dựng tương ứng**.
- Drestore chọn bộ giải mã và bộ dựng hình theo loại part. Thuật toán hình học, màu, nét, layer và dữ liệu kỹ thuật phải dùng **cùng nguồn được version hóa** với add-in tạo hình, không tự chép rồi sửa độc lập.
- Nếu `Ti.dat`, `color.dat`, `Shared_data.Set` thay đổi, việc khôi phục theo cấu hình hiện hành có thể không giống trạng thái tại lúc vẽ. Muốn khôi phục tuyệt đối phải có snapshot thông số hiệu dụng / cấu hình tương ứng; không giả định đã đủ.

### 4.1. Khả năng nhận diện trước/sau chuẩn hóa

| Dữ liệu | Reader | Writer mới khi đã triển khai |
|---|---|---|
| Dtscr1 MASTER `3 TSCR0`, `TSCR0_SCHEMA` V1–V4 | Giữ parser legacy đúng phiên bản | `3 TSCR0` + `DCAD_SCHEMA`/ROW theo PartSchema mới |
| Alias `3 TSCR` | Chỉ chấp nhận nếu schema TSCR hợp lệ như reader đã công bố | Không tự thay `TSCR0` hiện hữu |
| DPunch AP/BP `3 ap*/bp*` V381 | Giữ reader V381 và alias legacy | PRIMARY riêng + META/ROW/EXT chuẩn sau khi audit |
| DPunch CP1/CP4 `7 CP1/CP4` | Adapter theo contract CP/XRecord thực tế | Không tự thay Tok0, STT, quyền ưu tiên XRecord |
| Hole Circle/Profile Tok0=1/5 | Reader của entity hole đúng loại, không coi là MASTER | Giữ Hole NAME/HOLE_TYPE riêng theo hợp đồng subtype |

**XRecord:** Với phiên bản thực sự đã dùng `DCAD_DPUNCH`, kiểm tra OWNER/SCHEMA/COUNT và quy tắc ưu tiên đã công bố; nếu XRecord hiện diện nhưng lỗi, không âm thầm lấy MY_TAG để lách validation. Việc DPunch V381 có XRecord không được tự giả định (xem Phụ lục A).

## 5. Nghiệm thu bắt buộc khi chuyển DPunch sang chuẩn Dtscr1

1. Lập bảng mapping từ từng payload V381/legacy sang PRIMARY/META/ROW/EXT, chỉ dùng nguồn mã DPunch đã kiểm tra; giữ 17 HoleType, Process, PointAngle, CutEdge, ReliefAngle, CP1/CP4 và STT theo subtype.
2. Thực hiện **dual reader / single new writer** cho mỗi subtype khi đã có round-trip đủ và permission thay đổi; không tự migration DWG cũ.
3. So hình gốc với hình khôi phục về số đối tượng, kích thước, màu, Linetype, layer, định hướng, Undo và XData; ghi riêng kết quả từng subtype AP1–AP12/BP1–BP2/CP1/CP4 và từng host thực hiện.
4. Test dòng trống/lỗi, ROW hợp lệ đan xen, sai schema/count/index/EXT, vượt 255 byte, giới hạn XData, Unicode và legacy. Xác nhận không ghi ROW lỗi, không chặn ROW hợp lệ, không xóa gốc.
5. Build PASS/NETLOAD PASS/runtime PASS phải tách trạng thái; chưa build CAD thực ghi `NotVerified`. Tuân thủ STD-18: sửa tối thiểu đúng phạm vi, đối chiếu diff.

**Phụ lục A dưới đây là hợp đồng DPunch V381 đã phát hành, giữ lại để reader tương thích.** Các code/editor hiện còn ở schema cũ chỉ được xem là legacy, không tuyên bố đã dùng chuẩn mới khi chưa thay mã nguồn và kiểm thử.

---

## Phụ lục A — Hợp đồng DPunch V381 đã phát hành (reader tương thích)

### Phạm vi schema lịch sử

- Các quy tắc bên dưới mô tả **hợp đồng XData DPunch-V381** để ứng dụng CAD khác ghi/đọc tương thích.
- **Writer DPunch V381** dùng canonical schema V381 vào thời điểm phát hành; reader mới phải giữ nhánh đọc phiên bản này. Không tiếp tục phát sinh V381 sau khi writer mới đã được chuyển đổi và nghiệm thu theo hợp đồng chung.
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
