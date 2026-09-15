# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Hoàng Kim Thiện (2A202602293)`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): Bỏ qua xe cứu hộ hoặc xe công trình nếu chỉ xuất hiện dưới dạng tĩnh và bị che khuất > 80% thời lượng.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (2 giây @ 12.5 fps) | Đảm bảo tính liên tục của track (trajectory continuity) khi xe di chuyển bình thường qua chướng ngại vật |
| Xe bị che lâu hơn ngưỡng trên | Tạo track mới với ID mới | Khi bị che quá lâu, quỹ đạo chuyển động không còn chắc chắn thuộc cùng một đối tượng |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Tránh nhầm lẫn danh tính khi xe đã ra khỏi trường nhìn của camera |
| Hai xe cắt nhau / chồng lên nhau | Duy trì riêng biệt ID của từng xe; xe ở trước giữ bbox bình thường, xe ở sau chỉ ôm phần nhìn thấy | Tránh switch ID giữa 2 xe đan xen |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** (visible box) |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh (tối thiểu nhìn rõ 2 đặc điểm như đèn, kính hoặc bánh xe; chiều cao > 20px) |
| Xe đang đỗ, không di chuyển | Gán bbox tĩnh, đặt keyframe đầu và cuối, chỉ cập nhật nếu góc camera thay đổi |
| Keyframe đặt dày ở đâu | Đặt dày (mỗi 5 - 10 frame) ở các đoạn xe chuyển làn, cua gấp hoặc thay đổi gia tốc nhanh |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / frame 1-15 / ID 1`
- Tình huống: Xe xuất hiện ở góc xa đường chân trời, kích thước rất nhỏ và có độ tương phản thấp.
- Quyết định: Chờ đến khi xe tiến lại gần ở frame 1 và xác định rõ hình khối xe con mới bắt đầu track.
- Lý do: Tránh gán nhầm bóng râm hoặc xe máy ở khoảng cách cực xa thành FP.

### Ca 2
- Clip / frame / ID: `clip_01 / frame 85-95 / ID 5`
- Tình huống: Xe chuyển làn và bị khuất một phần góc bánh sau bởi xe khác.
- Quyết định: Chỉ khoanh vùng thân xe nhìn thấy được, bổ sung keyframe ở frame 86 và 91.
- Lý do: Tránh bbox nội suy bị trôi (drift) và tuân thủ đúng nguyên tắc visible box.

### Ca 3
- Clip / frame / ID: `clip_01 / frame 185-190 / ID 1`
- Tình huống: Xe chuẩn bị thoát khỏi khung hình ở mép dưới bên phải.
- Quyết định: Bấm thuộc tính outside ngay khi mép trước của xe vừa trượt hoàn toàn ra khỏi khung hình.
- Lý do: Ngăn chặn việc tồn tại bounding box rác đứng yên ở rìa ảnh.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Quy định rõ hơn về khoảng cách đặt keyframe: không để 2 keyframe cách nhau quá 15 frame ở các đoạn xe thay đổi vận tốc để tránh lỗi loose box.
- Thống nhất nguyên tắc cắt phẳng viền bbox tại các mép ảnh khi xe chạm biên.
