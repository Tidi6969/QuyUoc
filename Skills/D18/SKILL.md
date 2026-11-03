---
name: d18
description: "Quy ước D18 bảo vệ chất lượng AI lập trình DCAD: đọc toàn văn STD-18 mới nhất, sửa tối thiểu, kiểm chứng thực tế và HARD STOP khi không đạt."
---

# D18 — DCAD AI Coding Guard (thực thi Quy ước 18)

**Nguồn chuẩn duy nhất:** https://github.com/Tidi6969/QuyUoc/blob/main/18_Karpathy_Guidelines_cho_AI_lap_trinh.md  
**Nội dung raw:** https://raw.githubusercontent.com/Tidi6969/QuyUoc/main/18_Karpathy_Guidelines_cho_AI_lap_trinh.md

## Kích hoạt

Áp dụng cho **mọi** yêu cầu phân tích, viết, sửa, gỡ lỗi, rà soát và build mã nguồn DCAD. Khi viết/sửa/build, kích hoạt **đồng thời D17 và D18**. File Skill này chỉ dẫn cách lấy nguồn và kiểm chứng, **không thay thế** nội dung quy ước gốc.

## Bắt buộc trước khi code

1. Trong lượt hiện tại, qua GitHub connector đọc **toàn bộ** `18_Karpathy_Guidelines_cho_AI_lap_trinh.md` và `17_Autoupdate.md` mới nhất từ `Tidi6969/QuyUoc`, nhánh `main`; áp dụng đầy đủ mọi điều kiện liên quan được viện dẫn. Không dùng trí nhớ hoặc bản tóm tắt thay thế văn bản mới nhất.
2. Kiểm tra nguồn thực, baseline, scope, quyền sửa, CAD targets, ảnh hưởng Geometry/XData/XRecord/Settings/DUser/DCL/UI/đa ngôn ngữ/Support Path, rủi ro hồi quy. Nêu mục tiêu, phần giữ nguyên và tiêu chí đo được trước khi sửa.
3. Thực hiện theo nguyên tắc: **THINK BEFORE CODING → SIMPLICITY FIRST → SURGICAL CHANGES → GOAL-DRIVEN EXECUTION**. Không tự suy diễn nghiệp vụ, thêm tính năng/framework/abstraction không được giao, refactor lan rộng hay bỏ cơ chế an toàn.
4. Khi thiếu nguồn thiết yếu, sai host/schema, xung đột phiên bản hoặc chưa giải quyết được bất định trọng yếu: **HARD STOP**, không sửa mù.

## Quy trình bắt buộc

1. **Hiểu:** đọc đúng baseline, source, yêu cầu, giới hạn và quy ước.
2. **Đặt tiêu chí:** xác định input/expected/actual, ca lỗi-biên, chức năng giữ nguyên, trạng thái build/NETLOAD/runtime theo CAD host.
3. **Chọn phương án:** giải pháp đơn giản, ít thay đổi; mọi phần thêm mới phải phục vụ yêu cầu.
4. **Sửa:** chỉ thay đổi dòng/hàm/file/phụ thuộc cần thiết; giữ nguyên code ngoài phạm vi và dữ liệu; kiểm tra từng thay đổi trong diff.
5. **Kiểm tra:** tái hiện lỗi khi khả thi, chạy test tĩnh/hồi quy, build Release đúng từng CAD host; tìm nguyên nhân khi FAIL, không vá mù.
6. **Bàn giao:** đánh dấu từng chứng cứ `PASS`, `FAIL`, `NotVerified` hoặc `N/A` khi phù hợp. Không tự nhận đã build, NETLOAD, runtime, push GitHub hoặc nghiệm thu nếu không có bằng chứng.

## Cổng kiểm chứng

Đối chiếu **toàn bộ AI-01 đến AI-12** trong STD-18 mới nhất và các điều kiện tương ứng STD-17. `Build PASS` **không** có nghĩa `NETLOAD PASS` hoặc `Runtime PASS`. Khi không xác minh baseline, có nguy cơ ghi đè, sai schema/host, bug lặp chưa rõ nguyên nhân hoặc cổng kiểm thử bắt buộc FAIL → **HARD STOP** theo văn bản gốc.

## Quy tắc cập nhật

Mỗi lượt kích hoạt đều tải và áp dụng **nội dung mới nhất trên GitHub**; thay đổi của quy ước có hiệu lực ở lần đọc sau, không cần sửa nội dung tóm tắt của Skill. Không nhầm khả năng đọc live với khả năng tự cập nhật một Skill đã cài trên tài khoản ChatGPT.
