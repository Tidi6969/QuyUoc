---
name: d17
description: "Quy ước D17 cho mọi tác vụ tạo/sửa/build/đóng gói/phát hành module DCAD và đồng bộ GitHub; đọc toàn văn STD-17 mới nhất trước khi làm."
---

# D17 — DCAD Autoupdate (thực thi Quy ước 17)

**Nguồn chuẩn duy nhất:** https://github.com/Tidi6969/QuyUoc/blob/main/17_Autoupdate.md  
**Nội dung raw:** https://raw.githubusercontent.com/Tidi6969/QuyUoc/main/17_Autoupdate.md

## Kích hoạt

Áp dụng khi người dùng yêu cầu tạo, sửa, build, đóng gói, phát hành, lưu phiên bản hoặc xác nhận nghiệm thu một module DCAD. Với mọi nhiệm vụ viết/sửa/build code DCAD, phải đọc và tuân thủ đồng thời **D17 và D18**. Nội dung của file Skill này là **cơ chế dẫn tới nguồn chuẩn và kiểm soát**, không phải bản thay thế hay bản rút gọn của quy ước.

## Bước bắt buộc trước khi thao tác

1. Trong **lượt làm việc hiện tại**, dùng GitHub connector (ưu tiên) để đọc **toàn bộ** `17_Autoupdate.md` từ nhánh `main` của `Tidi6969/QuyUoc`; đồng thời đọc **toàn bộ** `18_Karpathy_Guidelines_cho_AI_lap_trinh.md`. Đọc thêm nguyên văn các quy ước khác được viện dẫn và áp dụng, nếu cần để hoàn tất kiểm chứng; không mặc định tóm tắt là đủ.
2. Xác minh đúng repository/module, baseline, nguồn thực tế, version đích, CAD target, quyền thay đổi, giới hạn và tiêu chí hoàn thành. Khi người dùng không chỉ định baseline/version, áp dụng quy tắc chọn **V<n> lớn nhất có source/ZIP thực và xác minh được**, version mới = **max + 1**. Chỉ định cụ thể của người dùng có ưu tiên.
3. Không tự chạy script đáng ngờ trong ZIP; không xóa/ghi đè nguồn đã lưu; không tự mở rộng phạm vi hoặc tạo bản ngoài yêu cầu.
4. Không đọc được nguồn mới nhất, không xác minh được baseline, xung đột version/nhánh hoặc thiếu dữ liệu thiết yếu → **HARD STOP** và báo rõ.

## Quy trình thực hiện

- Đọc và đối chiếu source/project/SDK/Reference thực tế; chỉ sửa đúng phần được giao; giữ nguyên chức năng ngoài phạm vi.
- Chuẩn hóa cấu trúc theo các tiêu chuẩn liên quan; không trộn SDK giữa các CAD host. Build Release độc lập theo host được yêu cầu; kiểm tra log, exit code và DLL vừa tạo.
- Kiểm tra QA, hồi quy, NETLOAD/smoke test theo yêu cầu; ghi riêng `PASS`, `FAIL`, `NotVerified` cho từng CAD và từng loại kiểm chứng. Không suy ra Runtime PASS từ Build PASS.
- Chỉ khi thỏa cổng phát hành: đóng gói ZIP đầy đủ source/project/script/tài nguyên/Reference được phép phân phối/DLL mới đã build; mở lại kiểm tra ZIP; viết changelog và BuildInfo; commit/push lên repository module.
- Đọc lại blob và đường dẫn sau upload; đồng bộ README/manifest của repository module và `Tidi6969/DCAD`; đọc lại và đối chiếu hai nơi.
- Khi người dùng nói OK/chạy ngon/đạt cho bản xác định được: **trong lượt đó**, chỉ cập nhật metadata nghiệm thu tại repository module và DCAD, đúng host/phạm vi người dùng xác nhận; không tự tăng version, không viết đè ZIP và không gán PASS ngoài bằng chứng.
- Trọn vẹn PASS: chỉ trả **một URL GitHub dạng /blob/<branch>/<path-to-ZIP>** theo quy ước gốc. Không đạt/chưa xác minh: báo `FAIL` hoặc `NotVerified` với cổng bị chặn, **không** gửi link ZIP giả hay công bố hoàn tất.

## Cổng kiểm chứng

Phải đối chiếu **toàn bộ AUTO-01 đến AUTO-16** trong văn bản STD-17 mới nhất; mỗi tiêu chí áp dụng cần bằng chứng/trạng thái chính xác. Mọi điều kiện phát hành bắt buộc không đạt → **HARD STOP**. Không được tự nhận đã build, chạy CAD, upload hoặc đồng bộ khi chưa kiểm chứng.

## Quy tắc cập nhật

Mỗi lần kích hoạt đều đọc **lại nội dung mới nhất trên GitHub**, không dựa vào bản copy cũ, tóm tắt cuộc hội thoại hay phiên bản file Skill đã lưu. Nếu STD-17 đổi trên GitHub, thực thi bản mới ngay trong lượt đọc tiếp theo. **Cập nhật GitHub không đồng nghĩa tự cập nhật Skill đã được cài trong giao diện ChatGPT**; việc cài/cập nhật Skill trong tài khoản chỉ thực hiện khi có quyền và công cụ hỗ trợ.
