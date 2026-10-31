# DCAD-STD-14 — Transaction, Undo và hiệu năng

**Phiên bản:** 1.5  
**Mức áp dụng:** Bắt buộc theo thao tác

## Transaction, DocumentLock, Undo, lifecycle, hiệu năng [CORE theo thao tác]

- Mọi sửa DWG thực hiện trong transaction; cân nhắc DocumentLock phù hợp host/ngữ cảnh. Không giữ transaction mở trong lúc modal Form kéo dài. Dùng một transaction/undo có kiểm soát cho một thao tác logic nếu khả thi.
- Khi phải chia batch lớn, chốt batch size, rollback, hành vi fail giữa chừng, thông báo partial và số lần Undo. Không commit “nửa vời” rồi báo thành công toàn bộ.
- Dispose resource đúng cách (transaction, result buffer, clone geometry, form, streams). Không tái tạo preview entities trên drawing rồi chỉ REGEN mới mất.
- Tránh scan toàn bản vẽ nếu workflow chỉ yêu cầu tập chọn; trì hoãn công việc nặng đến khi có phạm vi hợp lệ; tách parse/validation khỏi vẽ, tránh đọc file ở mỗi vòng lặp drawing. Không tối ưu thay logic gốc khi chưa có benchmark hoặc bằng chứng lỗi.
- Thực hiện regression trên CAD thật khi thay đổi selection/typed input/ESC/Undo/DrawJig, không chỉ static QA.
