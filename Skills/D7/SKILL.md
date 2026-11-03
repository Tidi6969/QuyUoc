---
name: d7
description: "DCAD D7: áp dụng Quy ước 07 về đa ngôn ngữ, Unicode và font; đọc toàn văn mới nhất trên GitHub. Khi code luôn áp dụng đồng thời D17 và D18."
---

# D7 — Quy ước DCAD số 7

**Nguồn chuẩn mới nhất:** https://github.com/Tidi6969/QuyUoc/blob/main/07_Da_ngon_ngu_va_Unicode.md

**Raw:** https://raw.githubusercontent.com/Tidi6969/QuyUoc/main/07_Da_ngon_ngu_va_Unicode.md

**Bản lưu chụp nguyên văn:** `references/STANDARD.md` (snapshot; không thay thế nguồn mới nhất).

## Khi nào sử dụng

Sử dụng D7 cho các công việc DCAD liên quan đến đa ngôn ngữ, Unicode và font. Cũng kích hoạt khi một quy ước áp dụng khác viện dẫn nội dung của D7. Với tất cả yêu cầu viết, sửa hoặc build code DCAD, luôn thực thi **D17 + D18** cùng các Skill chuyên môn cần thiết.

## Quy trình bắt buộc

1. Trước khi hành động, đọc **đầy đủ toàn văn** quy ước số 7 mới nhất trong repo `Tidi6969/QuyUoc`, nhánh `main`. Không dùng riêng snapshot hoặc tóm tắt để thay bản gốc. Nếu nguồn mới nhất không truy cập/kiểm chứng được → **HARD STOP**.
2. Xác định đúng module, baseline, phạm vi, CAD host, source và tài liệu liên quan; đọc thêm các quy ước được viện dẫn. Khi code, đọc và thực thi toàn văn D17 và D18. Không tự suy diễn luật nghiệp vụ hoặc sửa vượt ủy quyền.
3. Chuyển **tất cả** điều khoản kỹ thuật, ngoại lệ, cổng kiểm tra, ví dụ và tiêu chí QA áp dụng của tài liệu gốc thành checklist có bằng chứng. Không được bỏ bất kỳ điều kiện bắt buộc nào.
4. Chỉ sửa đúng yêu cầu; giữ nguyên chức năng, schema và dữ liệu ngoài phạm vi. Kiểm tra diff, test hồi quy và build riêng đúng CAD target khi cần.
5. Ghi kết quả theo từng tiêu chí và host: `PASS`, `FAIL`, `NotVerified` hoặc `N/A` đúng bằng chứng. Không biến suy đoán thành kết quả kiểm thử.
6. Nếu một cổng bắt buộc không đạt, baseline/SDK không thể xác minh, hoặc có nguy cơ hỏng dữ liệu → **HARD STOP**, báo rõ lý do; không công bố bản phát hành hoàn chỉnh. Bàn giao, đóng gói và đồng bộ GitHub theo D17 nếu đủ điều kiện.

## Nguồn và cập nhật

Github gốc là nguồn chính. Mỗi lần được gọi phải đọc lại quy ước mới nhất. Bản snapshot `references/STANDARD.md` phục vụ đối chiếu lịch sử và dùng cho bộ cài offline, không chứng minh là bản mới nhất. Skill nằm trên GitHub **không tự đồng nghĩa đã được cài** vào tài khoản ChatGPT.
