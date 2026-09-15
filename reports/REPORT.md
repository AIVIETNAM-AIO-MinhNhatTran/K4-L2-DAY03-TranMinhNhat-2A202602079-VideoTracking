# BÁO CÁO PHÂN TÍCH CHẤT LƯỢNG TRACKING (REPORT.md)

## I. Bảng tổng hợp kết quả (Metrics Summary)

| Cấu hình đánh giá | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **nhãn của bạn vs gold** | 0.806 | 0.790 | 0.824 | 0.883 | 0.954 | 0.904 | 0.871 | 52 | 3 | 0 |
| **ByteTrack vs gold** | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| **BoT-SORT + ReID vs gold**| 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| **ReID vs nhãn của bạn** | 0.751 | 0.694 | 0.814 | 0.889 | 0.881 | 0.764 | 0.876 | 80 | 64 | 3 |

*Cổng đánh giá nhãn của bạn so với Gold:* **ĐẠT** (IDF1 = 0.954 >= 0.80, MOTA = 0.904 >= 0.75, MOTP = 0.871 >= 0.70).

---

## II. Trả lời các câu hỏi phân tích

### 1. MOTA của bạn cao hơn hay thấp hơn IDF1? Giải thích lý do vì sao MOTA không phạt nặng lỗi ID Switch?
* **Kết quả thực tế:** MOTA đạt **0.904**, thấp hơn một chút so với IDF1 là **0.954**.
* **Giải thích lý do:** 
  - **IDF1 (Identity F1-score)** tập trung đo lường độ chính xác và nhất quán trong việc bảo toàn định danh (Identity) của toàn bộ quỹ đạo vật thể xuyên suốt thời gian thực của nó. Chỉ cần một đối tượng bị nhảy ID giữa chừng, toàn bộ mapping của track đó sẽ bị ảnh hưởng nặng nề làm kéo giảm IDF1.
  - Ngược lại, **MOTA (Multiple Object Tracking Accuracy)** là chỉ số tối ưu hóa cho bài toán phát hiện tích lũy và kết hợp sai số. Công thức của MOTA chỉ phạt lỗi ID Switch trực tiếp tại đúng frame xảy ra sự chuyển đổi (mỗi lần đổi ID chỉ tính là +1 IDSW). Do đó, nếu một xe đi qua 100 frames và chỉ bị đổi ID duy nhất 1 lần ở giữa, MOTA chỉ phạt duy nhất 1 lỗi IDSW trên tổng số 100 detections, khiến điểm số MOTA vẫn ở mức cực kỳ cao (~0.99), trong khi IDF1 sẽ bị phạt nặng do chia cắt một thực thể dài thành hai nửa không trùng khớp định danh.

### 2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA, IDSW?
* **So sánh số liệu:**
  - **IDF1:** BoT-SORT + ReID đạt **0.900**, cải thiện đáng kể so với ByteTrack (**0.875**).
  - **AssA (Association Accuracy):** BoT-SORT + ReID đạt **0.820** so với ByteTrack (**0.776**).
  - **IDSW:** Cả hai mô hình đều giữ ở mức **2 lần** đổi ID.
* **Phân tích chi tiết:**
  - Mặc dù số lần đổi ID (IDSW) là bằng nhau (đều bằng 2), BoT-SORT + ReID tối ưu hóa vượt trội về khả năng kết nối chính xác và tái định danh (Re-identification) các mảnh quỹ đạo bị chia cắt. Nhờ có vector đặc trưng ngoại hình (appearance embeddings), mô hình ReID có thể liên kết mượt mà lại các phương tiện sau khi bị che khuất hoặc khi đi sát nhau, giúp giữ track liền mạch dài hơn, phản ánh trực tiếp qua chỉ số AssA tăng mạnh (từ 0.776 lên 0.820) và kéo theo sự tăng trưởng của IDF1.

### 3. DetA, FP và FN thay đổi như thế nào? Lỗi còn lại chủ yếu do Detector hay do Association?
* **So sánh số liệu:**
  - **DetA:** BoT-SORT + ReID đạt **0.711** (tăng mạnh từ **0.649** của ByteTrack).
  - **FP (False Positives):** BoT-SORT + ReID có **91 FP** (tăng nhẹ từ **88 FP** của ByteTrack).
  - **FN (False Negatives):** BoT-SORT + ReID giảm sâu xuống còn **26 FN** (so với **54 FN** của ByteTrack).
* **Phân tích:**
  - Chỉ số DetA cải thiện rất nhiều và FN giảm hơn một nửa (từ 54 xuống 26) cho thấy thuật toán so khớp của BoT-SORT giúp giữ lại các detection điểm thấp cực kỳ tốt mà không làm bùng nổ lỗi nhận diện nhầm (FP chỉ tăng nhẹ từ 88 lên 91).
  - **Kết luận lỗi còn lại:** Lỗi lớn nhất hiện tại vẫn nằm ở khâu **Detector (phát hiện)** do mô hình YOLO26n có kích thước nhỏ, dễ bỏ sót hoặc nhận diện sai các phương tiện nằm ở rìa xa hoặc bị che khuất sâu, thể hiện qua việc lượng FP (91) và FN (26) còn tương đối lớn so với số lượng ID Switch rất thấp (chỉ 2).

### 4. Tìm một vị trí bạn đúng/ReID sai và ngược lại
* **Trường hợp Bạn đúng, ReID sai:**
  - **Vị trí:** Phân cảnh chiếc xe buýt lớn đi ngang qua ngã tư (Frames 105 - 106).
  - **Mô tả:** ReID treatment (đỏ T#) bị bắt trùng lặp và khoanh thừa BBox chồng lấn lên nhau (sinh ra FP dư thừa), trong khi nhãn của bạn (xanh B#) đã gán cực kỳ chuẩn xác và khít sát theo biên dạng thực tế của chiếc xe buýt.
* **Trường hợp ReID đúng, Bạn cần xem lại:**
  - **Vị trí:** Khu vực làn đường phía xa bên trái ngã tư (Frames 79, 91).
  - **Mô tả:** Có những phương tiện xe ô tô nhỏ màu trắng di chuyển chậm ở phần xa của camera, người gán nhãn đã bỏ sót không vẽ khung (lỗi FN của nhãn tay), nhưng mô hình ReID đã phát hiện cực kỳ nhạy bén và theo vết chuẩn xác.

### 5. Đề xuất sửa đổi GUIDELINE_MINI.md
* Cần bổ sung thêm ví dụ trực quan về việc bắt buộc phải vẽ nhãn các xe ở làn xe phía xa (kích thước nhỏ) nếu chúng vẫn nằm trong phạm vi quan sát rõ ràng của camera.
* Đưa quy định cứng về việc đặt keyframe sát nhau (~3 frames) khi xe đi qua vùng giao cắt giao thông phức tạp.
