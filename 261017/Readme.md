# Báo cáo nghiên cứu ngày 17/10/2026

**Bản dịch tiếng Việt và hình minh họa bài IJCV 2022:** [BAN_DICH_IJCV_2022_TIEN_G_VIET.md](./BAN_DICH_IJCV_2022_TIEN_G_VIET.md) — giữ bố cục các mục 1–5, công thức và số liệu chính; hình gốc từ repository tác giả.


## A. Công việc đã làm

### 1. Khảo sát nội dung và đóng góp của bài báo IJCV 2022
**Bài báo** :  [On the Arbitrary-Oriented Object Detection: Classification Based Approaches Revisited, Yang & Yan, IJCV 2022](./2003.05597v4.pdf)

**Bài toán:** Hồi quy góc hộp giới hạn xoay (*oriented bounding box*, OBB) có thể tạo ra lỗi gián đoạn gần biên miền góc hoặc khi đổi thứ tự cạnh/đỉnh. Các tác giả chuyển dự đoán góc thành bài toán phân loại các khoảng góc; các lớp góc phải giữ tính tuần hoàn.

**Các thành phần cốt lõi:**

- **Circular Smooth Label (CSL) — Mục 3.4, Eq. (3)–(4):** Làm mềm nhãn góc dựa trên khoảng cách tuần hoàn. Bài báo thử bốn hàm cửa sổ: pulse, rectangular, triangular và **Gaussian**. Nhãn mềm cho phép các góc lân cận không bị phạt giống như một góc xa. CSL xử lý chu kỳ góc, còn vấn đề hoán đổi cạnh của OBB cũng phụ thuộc cách quy ước góc.
- **Densely Coded Label (DCL) — Mục 3.5:** Giảm số chiều mã góc, sử dụng **Binary Coded Label (BCL)** hoặc **Gray Coded Label (GCL)**. Ví dụ với miền 180° và độ phân giải 1°, CSL dùng 180 giá trị trong khi mã nhị phân/Gray chỉ cần 8 bit. Điều này khắc phục nhược điểm chi phí tính toán khi cần nhiều angle bins.
- **Angle Fine-Tuning — Mục 3.6, Eq. (10)–(11):** Bổ sung một nhánh hồi quy phần dư nhỏ trong mỗi bin nhằm giảm sai số lượng tử hóa. **Không thể tuyên bố dòng nghiên cứu CSL chỉ biết xuất góc rời rạc**, vì bản IJCV đã có cơ chế tinh chỉnh liên tục.
- **Object Heading Detector (OHDet) — Mục 3.7:** Thay vì chỉ dùng OBB, tác giả thêm một nhánh **phân loại bốn vị trí/cạnh của đầu vật thể** để xác định hướng đầu–đuôi tàu, máy bay và phương tiện. Có dataset **OHD-SJTU**. Vì vậy **directed heading đã được nghiên cứu trong chính họ công trình CSL**.
- **Thực nghiệm — Mục 4:** Báo cáo mAP rotated detection và các chỉ số **OBB mAP, OHD mAP, Head Accuracy**. Trong Mục 4.4, Head Accuracy tại cột IoU50:95 là khoảng **94,25%** (OHD-SJTU-S) và **64,12%** (OHD-SJTU-L). Các con số này **không phải circular MAE** và không được dùng so sánh trực tiếp với sai số góc Leanbot.

### 2. So sánh trực tiếp với phương pháp ứng dụng cho Leanbot

| Nội dung | CSL / DCL / OHDet trong IJCV 2022 | Bài toán ứng dụng cho Leanbot hiện tại | Nhận xét về đóng góp |
|---|---|---|---|
| Bài toán | Phát hiện vật thể với OBB; OHDet thêm hướng đầu vật thể | Phát hiện Leanbot bằng hộp ngang và suy ra heading 360° | Khác mục tiêu và cách biểu diễn, **không chứng minh tính mới** |
| Kiến trúc | Object class, OBB angle và heading được tổ chức thành các đầu ra/nhánh riêng | YOLO11n dùng **24 detection classes** chính là 24 nhãn hướng, không có angle head độc lập | Khác cách tích hợp head; cần baseline cùng backbone |
| Miền góc | CSL cho OBB thường dùng chu kỳ 180° theo quy ước; OHDet xác định hướng đầu | 24 hướng trải toàn 360°, mỗi bin 15° | Heading 360° không phải điểm mới riêng |
| Nhãn học góc | CSL có Gaussian circular targets và các cửa sổ khác | Gaussian circular targets, `sigma=15°` | **Giống nguyên lý**, không nên gọi loss Gaussian là phát minh mới |
| Hàm loss | Loss riêng cho angle classification (focal loss hoặc sigmoid BCE tùy detector) và các mục tiêu khác | Thay mục tiêu đưa vào BCE của **YOLO classification head** bằng nhãn Gaussian đã cân theo target score | Là sự khác biệt trong tích hợp giám sát |
| Độ dài đầu ra | CSL phụ thuộc số angle bins; DCL giảm mạnh kích thước mã góc | 24 class scores, tức 24 giá trị cho toàn vòng 360° | Thiết kế gọn do bin thưa; chưa chứng minh chính xác/hiệu quả hơn |
| Giải mã | CSL cơ sở dùng argmax; IJCV thêm **residual angle fine-tuning** | Tổng vector của **24 scores** rồi lấy atan2 để suy ra góc liên tục | Khác chiến lược giải mã, cần đối chứng argmax và fine-tuning |
| Hướng đầu–đuôi | OHDet suy ra đầu đối tượng từ rotated box + nhánh 4-way head | Heading được học **trực tiếp** qua nhãn class robot | Khác thiết kế; cần so số tham số/MAE/độ tin cậy |
| Trước/sau NMS | Detector có chọn/lọc candidate nhưng không phải cơ chế tổng hợp vector đang triển khai ở Leanbot | Có phân tích raw pre-NMS, gom các candidate IoU cao rồi hợp nhất góc | Hướng kiểm chứng tiềm năng; chưa thể khẳng định mới |
| Thước đo | OBB/OHD mAP và Head Accuracy | Cần **circular MAE/median/P90** theo góc thực, đồng thời detection accuracy và latency | Không so trực tiếp kết quả từ các dataset khác nhau |

### 3. Đối chiếu công thức toán học và mã nguồn thực tế

**3.1. CSL trong bài báo (Mục 3.4)**

CSL xây dựng nhãn bằng hàm cửa sổ có bán kính `r`, tuần hoàn theo số lượng angle bins; các điểm ngoài cửa sổ được gán target bằng 0. Gaussian là một lựa chọn của hàm cửa sổ, **không phải toàn bộ thuật toán CSL**. Phần giải mã CSL cơ sở sử dụng bin có score cao nhất (Eq. 4).

**3.2. Soft Angular BCE trong Leanbot**


$$
d_{ij}=\min\left(|\theta_i-\theta_j|,\;360^\circ-|\theta_i-\theta_j|\right)
$$

$$
y_{ij}^{LB}=a_i\exp\left(-\frac{d_{ij}^{2}}{2\sigma^2}\right),
\quad \theta_j=15^\circ j,\quad j=0,\ldots,23,\quad \sigma=15^\circ.
$$

Trong đó `a_i` là **trọng số target score do cơ chế gán mẫu của YOLO tạo ra**. Gaussian được nhân với `a_i` rồi truyền cho BCE classification loss; mẫu nền vẫn giữ targets bằng 0. Khác với định nghĩa CSL có cửa sổ hữu hạn, Gaussian trong code hiện tại **không bị cắt theo bán kính** (các bin xa nhận target rất nhỏ, không chính xác bằng 0).

**3.3. Weighted Circular Mean — giai đoạn suy luận, không phải loss**

$$
C_i=\sum_{j=0}^{23}s_{ij}\cos\theta_j,\quad
S_i=\sum_{j=0}^{23}s_{ij}\sin\theta_j,
\quad
\hat{\theta}_i=\operatorname{atan2}(S_i,C_i).
$$

`s_{ij}` là class scores của YOLO cho một candidate. Phương pháp này có thể cho đầu ra góc **không trùng các bội số 15°**, nhưng không đảm bảo nội suy đúng góc thực nếu scores bị lệch hoặc đa đỉnh. Các scores sigmoid của detection head **không mặc nhiên là phân bố xác suất chuẩn hóa**.

Có thể khảo sát độ tập trung chuẩn hóa:

$$
R_i=\frac{\sqrt{C_i^2+S_i^2}}{\sum_j s_{ij}}\in[0,1]
\qquad (\sum_j s_{ij}>0).
$$

`R_i` thấp báo hiệu các hướng phân tán hoặc triệt tiêu, **không phải xác suất dự đoán sai/đúng**. Cần đánh giá thực nghiệm khả năng dùng nó để phát hiện dự đoán không đáng tin.

**3.4. Phân tích nhiều candidates trước NMS**

Script hiện đã chọn tối đa **200 candidates** theo score lớn nhất, nhóm các box dựa trên **IoU mặc định 0,7** và cộng các vector góc của những candidate cùng nhóm. Đây là hướng nghiên cứu khả thi hơn việc chỉ thay Gaussian, vì có thể so sánh **một candidate sau NMS** và **nhiều candidates trước NMS** mà giữ nguyên model weights. Tuy nhiên, nhiều candidates gần nhau có thể cùng mắc lỗi nên cũng có nguy cơ khuếch đại sai lệch.

### 4. Hạn chế đã thấy trong IJCV 2022 và khoảng trống liên quan Leanbot

**Hạn chế/điều kiện được bài báo thể hiện:**

- CSL với góc rất mịn cần nhiều output channels; DCL được phát triển để giảm chi phí này (Mục 3.5).
- Argmax trên các angle bins có sai số lượng tử hóa; tác giả đã đề xuất residual fine-tuning (Mục 3.6).
- OHDet gặp khó khi đầu và đuôi gần đối xứng, vật thể nhỏ/mờ, đông đúc hoặc số mẫu ít. Trong Mục 4.4, Head Accuracy thay đổi rõ rệt giữa OHD-SJTU-S và OHD-SJTU-L.

**Những nội dung chưa được kiểm chứng trực tiếp trong bài IJCV, chứ không phải khuyết điểm của tác giả:**

- Độ chính xác *continuous 360° robot heading* từ các **class scores của một YOLO detector** chỉ dùng 24 lớp hướng.
- Nội suy góc giữa các nhãn 15° khi huấn luyện ít ảnh, đánh giá bằng **ground truth góc liên tục độc lập**.
- Lợi ích/hạn chế của **weighted circular mean và grouping các candidates pre-NMS** trong điều kiện camera cố định nhìn chéo, giới hạn tài nguyên tính toán.
- Khả năng hiệu chuẩn độ tin cậy của góc dự đoán trong các tình huống mơ hồ.

**Các giả thuyết nghiên cứu có thể kiểm tra:**

- **H1:** Gaussian soft BCE cải thiện circular MAE ở góc **không xuất hiện trong train** so với hard BCE.
- **H2:** Với cùng model weights, weighted circular mean có tốt hơn argmax và nội suy lân cận/angle fine-tuning không?
- **H3:** Hợp nhất các candidates pre-NMS có giảm sai số/độ dao động mà không làm tăng quá nhiều latency không?
- **H4:** Lợi ích còn ổn định khi giảm số session train, thay đổi ánh sáng, góc camera hoặc kích thước ROI không?
- **H5:** Có thể dùng độ tập trung scores hoặc mức đồng thuận candidates để nhận biết dự đoán heading không đáng tin cậy không?

## B. Khó khăn

1. **Tính mới của loss:** Công thức Gaussian circular labels đã có trong CSL 2020 và IJCV 2022. Việc chỉ cho thấy Soft Angular BCE hơn hard BCE sẽ chủ yếu chứng minh ứng dụng prior work vào Leanbot.
2. **Tránh chọn sai research gap:** IJCV đã có **angle fine-tuning** và **OHDet**, vì vậy không được viết “CSL không suy ra được góc liên tục” hoặc “CSL không giải quyết heading đầu–đuôi”.
3. **Thiếu phép đo chuẩn cho heading liên tục:** Dataset huấn luyện phiên bản 11/09/2026 gồm **204 ảnh, 24 lớp góc** theo tài liệu dự án. Cần tạo tập test khác với góc được đo độc lập, bao gồm các góc 7°, 22°, 38°... không nằm trên nhãn 15°.
4. **Độ tin cậy của benchmark:** Notebook hiện chia train/val/test ngẫu nhiên theo ảnh thuộc từng class; ảnh gần trùng từ cùng session có thể gây **data leakage**. Bên cạnh đó, `degrees=10.0` thực hiện augmentation xoay ảnh **nhưng không cập nhật class heading tương ứng**, có nguy cơ gây nhiễu nhãn.
5. **Vấn đề hình học camera:** Camera Leanbot đặt chéo khoảng 45°; góc biểu kiến trên ảnh không mặc nhiên bằng yaw trong hệ tọa độ sa bàn. Cần chuẩn hóa quy ước đo và kiểm tra ảnh hưởng hiệu chỉnh phối cảnh.
6. **Đánh giá công bằng:** IJCV báo cáo OBB/OHD mAP và Head Accuracy, trong khi đề tài cần circular MAE. Không được so các trị số này trực tiếp hoặc so tốc độ trên phần cứng khác nhau.
7. **Đầu ra score có thể không đáng tin:** Gaussian smoothing và circular mean vẫn có thể sai khi scores đa đỉnh, hướng 0°/180° có score tương đương hoặc bị ảnh hưởng bởi các candidate trùng lặp.

## C. Công việc tiếp theo

### 1. Chuẩn hóa dữ liệu và đánh giá

- Chia dữ liệu theo **session/ngày thu thập**, không chỉ chia ngẫu nhiên từng ảnh; dùng buổi chụp độc lập cho test.
- Thiết lập ground truth góc thực trong **hệ tọa độ sa bàn**; lấy góc không trùng bội số 15°, chú trọng vùng biên 0°/360° và các trường hợp tĩnh/quay tại chỗ.
- Tắt `degrees=10.0` và kiểm tra mọi augmentation ảnh có thể làm thay đổi heading mà không cập nhật nhãn.
- Báo cáo **Circular MAE, median, P90, tỷ lệ sai số dưới 5°/10°/15°**, chất lượng detection, thời gian inference cùng phần cứng; nếu đủ dữ liệu, chạy nhiều seed.

### 2. Thực hiện ma trận benchmark/ablation

| ID | Training | Giải mã/postprocessing | Mục tiêu |
|---|---|---|---|
| A | YOLO11n + Hard BCE | Argmax | Baseline |
| B | **Giữ weights A** | Circular mean (một candidate) | Đo riêng lợi ích decoder |
| C | YOLO11n + Soft Angular BCE | Argmax | Đo lợi ích của nhãn mềm |
| D | **Giữ weights C** | Circular mean (một candidate) | Đo riêng lợi ích decoder dưới soft labels |
| E | **Giữ weights C** | Nội suy từ bin lân cận có score cao | Baseline giải mã liên tục khác |
| F | **Giữ weights C** | Circular mean + nhóm candidate pre-NMS | Đo lợi ích multi-candidate fusion |
| G (mở rộng) | Detector có **angle head riêng + CSL** | Argmax / residual fine-tuning | Đối thủ kiến trúc gần IJCV/YOLO-CSL |
| H (mở rộng) | CNN/YOLO + sin/cos regression | Atan2 | Đối thủ Biternion |

Sau thí nghiệm nền: khảo sát `sigma` = 7,5° / 15° / 22,5° / 30°, ngưỡng IoU, số candidates, độ tập trung scores và ảnh hưởng số session training.

### 3. Định hướng publication tạm thời

**Chủ đề gợi ý:** *Continuous 360° Robot Heading Estimation from Sparse Angular Labels via Circular Score Decoding.*

**Câu hỏi khoa học:** Liệu một YOLO detector học trực tiếp 24 nhãn heading tuần hoàn, kết hợp chiến lược giải mã/đồng thuận scores, có dự đoán heading 360° liên tục chính xác và đáng tin cậy hơn argmax, CSL có angle branch hoặc sin/cos regression **trong cùng một điều kiện thí nghiệm** hay không?

**Điều kiện để công bố:** Chỉ kết luận có đóng góp thuật toán mới sau khi chứng minh một cải tiến cụ thể so với prior work (đặc biệt CSL/DCL/fine-tuning/OHDet), có giải thích phương pháp, baseline và ablation đầy đủ. Nếu chỉ xác nhận Soft Angular BCE tốt hơn hard BCE thì nên mô tả là **ứng dụng CSL** vào bài toán Leanbot, không phải sáng tạo loss mới.

### Tài liệu cần tiếp tục đọc

- [IJCV 2022 — PDF trong thư mục này](./2003.05597v4.pdf), ưu tiên Mục 3.4, 3.5, 3.6, 3.7 và 4.4.
- [CSL — ECCV 2020](https://www.ecva.net/papers/eccv_2020/papers_ECCV/papers/123530664.pdf).
- [Leanbot: notebook SoftBCE](https://github.com/HoangAnh301194/Leanbot_orientation_estimation-CNN_base/blob/master/Leanbot_Camera_AI_Control_FINAL/tools/Leanbot_Train_SoftBCE.ipynb).
- [Leanbot: phân tích raw scores](https://github.com/HoangAnh301194/Leanbot_orientation_estimation-CNN_base/blob/master/Leanbot_Camera_AI_Control_FINAL/tools/check_confidence.py).
