# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Trần Minh Nhật`
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

Bổ sung của nhóm (nếu có): `Chỉ gán các xe bốn bánh di chuyển thực tế trên lòng đường, bỏ qua các phương tiện là hình ảnh phản chiếu trên kính của tòa nhà hoặc hình vẽ quảng cáo.`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Do camera quay cố định, xe đi khuất sau cột điện hoặc biển báo rồi lộ diện lại rất nhanh, việc giữ nguyên ID giúp biểu diễn liên tục quỹ đạo của một vật thể đơn lẻ. |
| Xe bị che lâu hơn ngưỡng trên | Cắt track và gán **ID mới** | Vượt quá thời gian ước lượng của mắt thường và tracker, không thể đảm bảo chắc chắn đó có phải là chính chiếc xe đó hay không. |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Tuân thủ quy định chuẩn của MOT 1.1 khi một vật thể tái xuất hiện sau khi đã đi ra ngoài biên hình ảnh hoàn toàn. |
| Hai xe cắt nhau / chồng lên nhau | Duy trì ID riêng biệt cho từng xe | Tránh lỗi merge ID hay ID Switch làm suy giảm nghiêm trọng chỉ số IDF1 và AssA. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: **Khi xe đạt kích thước chiều rộng tối thiểu khoảng 10-15 pixels và nhìn rõ cabin/bánh xe** |
| Xe đang đỗ, không di chuyển | Gán đầy đủ BBox và duy trì một ID cố định duy nhất suốt quá trình đứng im |
| Keyframe đặt dày ở đâu | Đặt keyframe dày (khoảng **3-5 frames/lần**) tại các góc cua, khi xe đi qua giao lộ đổi hướng đột ngột, hoặc khi xe bị che khuất một phần (occlusion) |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / frame 105 - 106 / ID 6`
- Tình huống: Chiếc xe buýt lớn đi ngang qua ngã tư và che lấp một phần các xe nhỏ phía sau.
- Quyết định: Vẽ BBox khít theo mép ngoài nhìn thấy của xe buýt, xe nhỏ phía sau khi bị che hoàn toàn thì đánh dấu `outside=1`.
- Lý do: Đảm bảo độ khít sát (tightness) cực cao và không sinh ra bounding box rỗng kéo trôi khi bị khuất.

### Ca 2
- Clip / frame / ID: `clip_01 / frame 79 / ID 5`
- Tình huống: Xe ô tô con màu trắng di chuyển rất chậm ở phía làn đường xa bên trái góc camera.
- Quyết định: Phải gán nhãn đầy đủ cho xe này dù kích thước rất nhỏ và mờ.
- Lý do: Nhãn Gold gán rất kỹ ở làn xe phía xa này, bỏ sót sẽ làm tăng lỗi FN rất cao của nhãn tay.

### Ca 3
- Clip / frame / ID: `clip_01 / frame 149 / ID 4`
- Tình huống: Xe chuẩn bị đi ra ngoài rìa phải màn hình và khuất dần.
- Quyết định: Gán Outside chính xác ngay tại frame 150 khi toàn bộ thân xe vừa vượt qua mép ảnh.
- Lý do: Tránh lỗi "treo khung" ở các frame 151, 152 dẫn tới tăng FP.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **Quy tắc bấm Outside đúng Frame biến mất (Bắt buộc):** Khi xe biến mất khỏi khung hình, ngay tại frame tiếp theo phải đánh trạng thái Outside lập tức, tuyệt đối không để dư thừa BBox treo ở rìa.
- **Bổ sung keyframe tại các đoạn gia tốc hoặc chuyển hướng:** Khi xe chuyển hướng rẽ tại ngã tư, không được để khoảng cách keyframe quá xa (phải đặt 3-5 frames một lần) để tránh nội suy tuyến tính làm lệch BBox.
