# Báo cáo tiến độ nghiên cứu ngày 09/10/2026

**Đề tài ( tên dự kiến):** Ước lượng góc định hướng 0-360° của robot Leanbot từ camera cố định bằng CNN-based Object Detection.
## A. Tổng quan dự án
- Hiện tại dự án em nghiên cứu và thực hiện trên Công Ty DTT do Thầy Quảng hướng dẫn trực tiếp đã hoàn thành và giải quyết một số nội dung bài toán nhưu sau ạ : 

1. **Bài toán:** 
    - Nhận diện, phát hiện vị trí và ước lượng góc định hướng của robot Leanbot trên mặt phẳng 2D bằng một camera cố định ( đặt chéo 45 độ ở một phía sa bàn) 
    - Thông qua vị trí và góc định hướng ,hệ thống máy chủ sẽ điều hướng Leanbot tới vị trí yêu cầu trên sa bàn thôgn qua hình ảnh từ Camera. 

2. **Mục tiêu:**
    - Ước lượng hướng quay từ 0-360°, phân biệt góc dương/âm mà không cần dùng mô hình các mô hình keypoint pose estimation cần dữ liệu huấn luyện và quy trình thu thập dữ liệu phức tạp
    - Áp dụng bộ điều khiển PID tính toán lệnh điều khiển, gửi lệnh điều khiển thôgn qua BLE để Leanbot di chuyển tới vị trí yêu cầu

    - Ảnh ví dụ minh họa khi nhận diện góc Leanbot (Leanbot_m75 nghĩa là minus 75 degree : -75 độ):

    ![alt text](image.png)

    - Ảnh thực tế triển khai khi điều khiển : 

    ![alt text](image-1.png)

    - Ảnh đường quỹ đạo di chuyển : 

    ![alt text](image-3.png)

    - Ảnh đồ thị các dữ liệu quan sát : 

    ![alt text](image-2.png)
3. **Một số kết quả kỹ thuật bước đầu:**
    - Đã xây dựng được mô hình phát hiện Leanbot và ước lượng góc định hướng bao phủ toàn bộ 360°, bao gồm các góc có dấu trong bốn góc phần tư đường tròn lượng giác. 
    - Bộ dữ liệu của phiên bản huấn luyện ngày gần nhất ( 11/09/2026) gồm 204 ảnh, 24 lớp góc 
    - Đã triển khai phương pháp kết hợp Soft Angular BCE và Weighted Circular Mean nhằm khai thác quan hệ tuần hoàn giữa các lớp góc, từ đó suy ra góc liên tục.
    - Đã tích hợp mô hình vào hệ thống nhận diện, theo dõi và điều khiển chuyển động của Leanbot bằng PID controller với camera cố định.

## B. Nội dung trọng tâm và kết quả hiện tại

1. **Hệ thống thu thập dữ liệu huấn luyện (dataset):** Đã triển khai bộ công cụ, quy trình thu thập ảnh, tách nền và tự động tạo nhãn bounding box
2. **Huấn luyện mô hình:** Sử dụng **YOLO11n** và triển khai chỉnh sửa hàm Loss function thành **Soft Angular BCE** nhằm xét đến tính tuần hoàn và liên hệ giữa các class góc. 
3. **Suy ra góc liên tục:** Dùng **weighted circular mean** trên điểm dự đoán (confidence) của các lớp góc để tính toán ra góc (có dấu) trong miền [−180°, 180°].
4. **Triển khai inference thực nghiệm :** Đã triển khai, export sang mô hình dạng **OpenVINO FP16** với 2 chế độ **Full Detection 640×640** và **ROI Tracking 160×160**, có cơ chế chuyển về tìm kiếm toàn ảnh nếu không phát hiện được Leanbot. 


- Theo như gợi ý của Thầy hôm qua thì em chia dự án thành các nội dung chính có thể đào sâu để viết bài publication như sau ạ : 

### 1. Phương pháp xác định góc xuay của vật thể từ camera cố định ứng dụng mô hình Yolo detection 
- **Điểm đóng góp chính** : 
    - Đưa ra công thức tính hàm mất mát (Loss function) cho bài toán : Soft Angular BCE Loss . 
        - Mục đích thay thế BCE (Binary Cross Entropy loss) mặc định của YOLO model , vốn chỉ xem mỗi nhãn là độc lập, khôgn có mối quan hệ gì với nhau ( ví dụ Leanbot 0 độ và Leanbot 15 độ được xem là độc lập hoàn toàn không có sự tương đồng nào gần nhau)
        - Tuy nhiên với bài toán xác định góc thì các nhãn có mối quan hệ với nhau ( ví dụ Leanbot 0 độ, nhìn gần giống với Leanbot 15 độ) 
        - Từ các mối quan hệ mờ này ta có thể ước lượng được góc giữa các nhãn gần nhau thông qua cơ chế cộng vector Confidence ( trọng số dự đoán ) của các nhãn. 
- Phân tích dữ liệu raw của model Yolo trước khi đi qua lớp lọc NMS ( lớp lọc giữ lại dự đoán có confidence cao nhất ) :
    - Dữ liệu trước lớp NMS là dữ liệu thô output trực tiếp từ model, nó chứa toàn bộ trọng số dự đoán của các class góc của Leanbot. Thôgn qua dữ liệu này để tính toán vector tổng hợp trọng số confidence để tính toán ra góc ước lượng 
- **Điểm đóng góp bổ sung ( thực nghiệm)** : 
    - Cơ chế thu thập dữ liệu ; đánh nhãn tự động và tự động tạo dataset cho mô hình huấn luyện 
    - Các cơ chế làm mịn dữ liệu thô bị nhiễu sau khi ước lượng góc được áp dụng để tăng ổn định và độ chính xác output: 
        - **Temporal Angle Smoothing**: Làm mượt chuỗi góc dự đoán theo thời gian bằng cách unwrap góc và hồi quy đa thức bậc nhất trên cửa sổ dữ liệu trượt, hạn chế dao động giữa các frame.
        - **Trajectory-based Heading Estimation**: Ước lượng hướng chuyển động dựa trên quỹ đạo tọa độ tâm robot ((x,y)) quan sát được qua nhiều frame.
        - **Velocity-Adaptive Angle Fusion**: Kết hợp góc từ mô hình CNN và hướng quỹ đạo với trọng số thay đổi theo vận tốc chuyển động : Khi robot gần đứng yên, ưu tiên góc CNN model detect ; khi robot chuyển động rõ ràng, tăng trọng số của hướng quỹ đạo.

**Kết quả khảo sát sơ bộ về CNN/YOLO và ước lượng góc tuần hoàn**

**Cách đọc và sử dụng tài liệu:** Phải phân biệt **directed heading 360°** (cần phân biệt đầu/đuôi Leanbot) với **orientation của rotated bounding box (OBB, thường là 180°)**. Bài OBB được dùng cho *prior work* về loss/encoding chứ **không thể so mAP OBB trực tiếp với sai số góc heading của Leanbot**. "**Có**" = phần trùng về bài toán/kỹ thuật; "**chưa có/chưa đánh giá**" = không được trình bày hay đo lường trong công trình **theo phạm vi tài liệu đã xem**, không đủ để tự kết luận một research gap mới. Quartile SJR/JCR dưới đây lấy từ bảng tổng hợp trước và **cần đối chiếu nguồn chính thức, năm và nhóm ngành khi nộp báo**.

**Mức độ trích dẫn trong bài báo của mình** có nghĩa là *độ ưu tiên sử dụng tài liệu*, **không phải số lần bài đó được người khác trích dẫn**:
- **Rất cao:** bắt buộc đối chiếu trong Related Work/Problem Formulation; cân nhắc tái hiện phương pháp cốt lõi trên cùng dataset Leanbot.
- **Cao:** dùng để so sánh khác biệt thuật toán, thiết kế ablation hoặc giải thích tại sao cần bổ sung thí nghiệm.
- **Trung bình:** dùng làm bằng chứng phương pháp đã tồn tại, đối chiếu hiệu năng/triển khai; không nhất thiết tái hiện toàn bộ.
- **Thấp–trung bình:** trích dẫn cho bối cảnh; không dùng làm đối thủ benchmark chính.

**1. Rotated Object Detection Using Adaptive Angle Classification and Dynamic Sample Matching (2026)**
- **Thông tin bài báo (SJR/JCR):** Liu Han và cộng sự, *Chinese Journal of Engineering*, 48(3), 586–598 (2026). [DOI](https://doi.org/10.13374/j.issn2095-9389.2025.06.09.006). **SJR 2025: Q2** (*Engineering*; SJR ≈ 0,386). **JCR 2025: Không có JIF quartile được xác nhận** (chưa ghi nhận trong WoS Core Collection). [SJR/coverage](https://journalindexes.com/gongcheng-kexue-xuebao-chinese-journal-of-engineering-20959389/) · [Nhà xuất bản](https://journal.ustb.edu.cn/Journals/index.htm).
- **Bài toán và phương pháp:** Phát hiện rotated object trên DOTA và dữ liệu ký tự công nghiệp; dùng YOLOv8, shape-aware adaptive angle classification (SA-ASL) với Gaussian soft label có độ rộng phụ thuộc hình dạng, kết hợp dynamic sample matching.
- **So sánh và kết quả:** mAP **78,6% (DOTA)** và **92,4% (ký tự công nghiệp)**; bài có ablation hai module, tác giả nêu mức cải thiện **4,3% trên các nhóm nhạy với góc**. Không phải sai số heading 360°.
- **Liên hệ với Leanbot — điểm có và chưa có:** **Có:** classification góc tuần hoàn, Gaussian soft labels, detector YOLO. **Chưa đánh giá theo mục tiêu Leanbot:** hướng đầu–đuôi 360°, chỉ 24 class góc, weighted circular mean từ candidate scores trước NMS, huấn luyện vài trăm ảnh và PID.
- **Vai trò trích dẫn/benchmark:** **Cao — Prior work sát nhất về cải tiến loss; gợi ý ablation**, đặc biệt so sánh σ=15° cố định với σ thích nghi. Không dùng mAP DOTA để benchmark trực tiếp circular MAE của Leanbot.

**2. ODC-YOLO: An Optimized YOLOv5 Method for Detecting Objects in Remote Sensing Images (2025)**
- **Thông tin bài báo (SJR/JCR):** Qing Liu và cộng sự, *Remote Sensing Letters*, 16(10), 1120–1130 (2025). [DOI](https://doi.org/10.1080/2150704X.2025.2529599). **SJR 2025: Q2** (*Electrical and Electronic Engineering*, SJR ≈ 0,434). **JCR 2025: Q4** (*Remote Sensing* và *Imaging Science & Photographic Technology*, JIF 1,5). [SJR](https://www.journalsbase.com/journals/remote-sensing-letters) · [JCR 2025](https://apa.letpub.com/index.php?journalid=8680&page=journalapp&view=detail). **Lưu ý:** Một số trang vẫn hiển thị **JCR Q3 năm 2024**, không phải năm 2025.
- **Bài toán và phương pháp:** Phát hiện vật thể nhỏ xoay nhiều hướng trong ảnh viễn thám; YOLOv5 cải tiến ODConv/Res2Net/M-RFB và dùng CSL để chuyển góc OBB từ regression sang classification.
- **So sánh và kết quả:** Theo abstract, cải thiện **3,45% mAP trên DOTA so với YOLOv10n**; mAP đối tượng nhỏ **26,5%**. Kết quả là năng lực phát hiện OBB.
- **Liên hệ với Leanbot — điểm có và chưa có:** **Có:** YOLO + circular smooth labels, giải quyết sự gần nhau và tính chu kỳ của các lớp góc. **Chưa thể hiện:** nhãn directed heading đủ 360°, không tách góc theo box geometry, weighted circular mean từ raw detection scores, test góc nội suy và vài trăm ảnh.
- **Vai trò trích dẫn/benchmark:** **Trung bình — Related Work / dẫn chứng triển khai CSL vào YOLO**; hữu ích để bác bỏ giả thiết 'CSL + YOLO là hoàn toàn mới', không cần tái triển khai toàn bộ ODConv/Res2Net làm baseline góc.

**3. Rotating-YOLO: A Novel YOLO Model for Remote Sensing Rotating Object Detection (2025)**
- **Thông tin bài báo (SJR/JCR):** Zhiguo Liu, Yuqi Chen, Yuan Gao, *Image and Vision Computing*, 154, 105397 (2025). [DOI](https://doi.org/10.1016/j.imavis.2024.105397). **SJR 2025: Q1** (*Computer Vision and Pattern Recognition*, SJR ≈ 0,881). **JCR 2025: Q2** (*Computer Science, Artificial Intelligence*) **hoặc Q1** (*Computer Science, Software Engineering*), JIF 5,0. [SJR](https://scienceaijournal.com/journals/image-and-vision-computing-0262-8856) · [JCR theo ngành](https://www.akaturk.com/journals/25549?lang=en).
- **Bài toán và phương pháp:** Oriented detection trong ảnh viễn thám; cải tiến YOLOv8 bằng feature fusion/attention, rotated bounding boxes và GaussianLoss cho hình học box.
- **So sánh và kết quả:** Theo abstract, so với **YOLOv8 baseline**, **giảm 33,25% tham số** và **tăng 1,4% mAP**. Hiệu năng đo detection, không có circular heading MAE tương ứng.
- **Liên hệ với Leanbot — điểm có và chưa có:** **Có:** YOLO, tính chất góc, tối ưu chi phí mô hình, sử dụng Gaussian cho rotated box. **Khác/Chưa có trong phạm vi báo cáo:** Gaussian class target BCE cho 24 heading classes, directed heading 360°, giải mã score thành hướng liên tục và thí nghiệm dữ liệu rất ít.
- **Vai trò trích dẫn/benchmark:** **Thấp–trung bình — Related Work về efficient oriented detection**; không phải baseline trực tiếp cho Soft Angular BCE vì loss làm việc trên biểu diễn Gaussian của box thay vì class heading.

**4. Detection of Objects in Satellite and Aerial Imagery Using Channel and Spatially Attentive YOLO-CSL for Surveillance (2024)**
- **Thông tin bài báo (SJR/JCR):** Divyansh Chaurasia, B. D. K. Patro, *Image and Vision Computing*, 147, 105070 (2024). [DOI](https://doi.org/10.1016/j.imavis.2024.105070). **SJR 2025: Q1** (*Computer Vision and Pattern Recognition*, SJR ≈ 0,881). **JCR 2025: Q2** (*Computer Science, Artificial Intelligence*) **hoặc Q1** (*Computer Science, Software Engineering*), JIF 5,0. [SJR](https://scienceaijournal.com/journals/image-and-vision-computing-0262-8856) · [JCR theo ngành](https://www.akaturk.com/journals/25549?lang=en).
- **Bài toán và phương pháp:** Phát hiện đối tượng OBB ở ảnh vệ tinh/hàng không bằng **YOLOv5 + nhánh angle riêng + Circular Smooth Labels (CSL) và BCEWithLogits**; bổ sung channel/spatial attention.
- **So sánh và kết quả:** Theo nhà xuất bản, đạt **57,86 mAP trên DOTA-v2**, hơn kết quả tốt thứ hai trong so sánh của tác giả **0,20 điểm mAP**; **25 triệu tham số, 54 GFLOPs**.
- **Liên hệ với Leanbot — điểm có và chưa có:** **Có:** gần nhất ở mức YOLO + CSL + BCE, học phân lớp góc theo chu kỳ, có so sánh chi phí tính toán. **Khác:** nhánh góc OBB riêng, không phải 24 class heading nằm trong nhãn object detection; **chưa báo cáo:** directed heading 360°, circular-mean decoding pre-NMS, few-image evaluation.
- **Vai trò trích dẫn/benchmark:** **Rất cao — Prior work bắt buộc và baseline kiến trúc/labeling**. Có thể tái hiện **YOLO + nhánh CSL angle** trên cùng Leanbot dataset nếu đủ nguồn lực; không so trực tiếp 57,86 mAP DOTA với heading MAE.

**5. Rotated Object Detection with Circular Gaussian Distribution (2023)**
- **Thông tin bài báo (SJR/JCR):** Hang Xu và cộng sự, *Electronics*, 12(15), 3265 (2023). [DOI](https://doi.org/10.3390/electronics12153265). **SJR 2025: Q2** (*Control and Systems Engineering*, SJR ≈ 0,623). **JCR 2025: Q2** (*Engineering, Electrical & Electronic*, JIF 2,9); **Q3** ở *Computer Science, Information Systems* và *Physics, Applied*. [SJR](https://scienceaijournal.com/journals/electronics-2079-9292) · [JCR – MDPI](https://www.mdpi.com/journal/electronics/stats).
- **Bài toán và phương pháp:** Xử lý gián đoạn và đa nghiệm của góc rotated box; detector dựa trên CenterNet-FPN, mã hóa góc bằng Circular Gaussian Distribution (CGD) và loss KL divergence.
- **So sánh và kết quả:** Trên **HRSC2016**, cùng backbone R-50-FPN: **CGD mAP07 90,52 so với CSL 89,98**; **mAP12 97,76 so với CSL 95,13**. Kết quả là OBB mAP.
- **Liên hệ với Leanbot — điểm có và chưa có:** **Có:** mô hình hóa phân bố xác suất góc tuần hoàn, tránh coi bin độc lập, khảo sát độ rộng Gaussian/loss. **Khác:** dùng KL-distribution loss và OBB theo chu kỳ 180°, không phải BCE mục tiêu mềm trên class heading 360° hoặc weighted circular mean pre-NMS.
- **Vai trò trích dẫn/benchmark:** **Cao — Đối chứng loss / ý tưởng ablation**, ví dụ hard BCE vs Soft Angular BCE vs normalized circular distribution + KL trên cùng dữ liệu Leanbot. Không cần coi toàn bộ CenterNet-FPN là benchmark bắt buộc.

**6. Object Detection of Flexible Objects with Arbitrary Orientation Based on Rotation-Adaptive YOLOv5 (2023)**
- **Thông tin bài báo (SJR/JCR):** Jiajun Wu và cộng sự, *Sensors*, 23(10), 4925 (2023). [DOI](https://doi.org/10.3390/s23104925). **SJR 2025: Q1** (*Instrumentation*, SJR ≈ 0,802; một số ngành khác Q2). **JCR 2025: Q2** (*Instruments & Instrumentation*, JIF 4,0). [SJR](https://scienceaijournal.com/journals/sensors-1424-8220) · [JCR – MDPI](https://www.mdpi.com/journal/sensors/stats).
- **Bài toán và phương pháp:** Rotated bounding-box detection cho vật thể dài/mềm tại công trường điện; thay YOLOv5 thông thường bằng rotated YOLOv5, thêm nhánh 180-bin Gaussian angle encoding.
- **So sánh và kết quả:** Trên tập **FO** của tác giả, **YOLOv5s 47,7% mAP và 44,1 FPS → R_YOLOv5s 57,9% mAP và 41,9 FPS**; thể hiện đánh đổi giữa detection accuracy và tốc độ.
- **Liên hệ với Leanbot — điểm có và chưa có:** **Có:** một detector YOLO với vector nhãn góc Gaussian và kết quả tốc độ/độ chính xác, tương tự mục tiêu gọn nhẹ. **Không giống:** nhánh angle OBB 180 bin, không dự đoán directed heading 360° bằng 24 class dùng chung detection label; chưa có đánh giá ước lượng liên tục từ raw scores.
- **Vai trò trích dẫn/benchmark:** **Trung bình — Prior work hỗ trợ và đối chứng trade-off FPS/mAP**; nếu khảo sát loss là chính, trích dẫn trong Related Work thay vì tái hiện tất cả R_YOLOv5.

**7. Arbitrary-Oriented Object Detection with Circular Smooth Label (2020) — Nghiên cứu nền tảng**
- **Thông tin bài báo (SJR/JCR):** Xue Yang, Junchi Yan, *ECCV 2020* (hội nghị; **SJR-Journal Q: không áp dụng; JCR-Journal Q: không áp dụng**; cần đánh giá theo xếp hạng hội nghị riêng). [DOI](https://doi.org/10.1007/978-3-030-58598-3_40).
- **Bài toán và phương pháp:** Khắc phục lỗi biên khi hồi quy rotated box; chuyển sang phân loại góc và tạo **Circular Smooth Label** với nhiều window functions, gồm Gaussian.
- **So sánh và kết quả:** ECCV 2020 kiểm chứng trên **DOTA, HRSC2016, ICDAR2015, MLT**, so sánh regression/classification và các radius/window; không sử dụng mAP của bài để đo sai số heading trên Leanbot.
- **Liên hệ với Leanbot — điểm có và chưa có:** **Có:** nền tảng lý thuyết quan trọng nhất cho tính tuần hoàn, khoảng cách bin góc và Gaussian circular labels. **Không giải quyết trực tiếp:** phân biệt đầu–đuôi 360° của robot, 24-class YOLO11n heading, circular mean trên pre-NMS scores, tập vài trăm ảnh và vòng điều khiển.
- **Vai trò trích dẫn/benchmark:** **Rất cao — Trích dẫn bắt buộc (seminal prior work); baseline hard labels vs CSL**. Khi trình bày công thức Soft Angular BCE phải ghi rõ kế thừa ý tưởng circular smoothing; kiểm tra điểm khác thực sự ở IoU weighting và inference.

**8. Biternion Nets: Continuous Head Pose Regression from Discrete Training Labels (2015) — Baseline 360°**
- **Thông tin bài báo (SJR/JCR):** Lucas Beyer, Alexander Hermans, Bastian Leibe, *GCPR 2015*, LNCS 9358, tr. 157–168 (hội nghị; **SJR-Journal Q: không áp dụng; JCR-Journal Q: không áp dụng**; cần đánh giá theo xếp hạng hội nghị riêng). [DOI](https://doi.org/10.1007/978-3-319-24947-6_13).
- **Bài toán và phương pháp:** Ước lượng **head pose có hướng liên tục 360°** bằng CNN, dù nhãn huấn luyện là các góc rời rạc; hồi quy biểu diễn biternion (cos θ, sin θ) để tránh biên chu kỳ.
- **So sánh và kết quả:** Các benchmark hướng đầu (Tosato, TownCentre...) đánh giá sai số góc và so với các kiểu regression/classification; trang tác giả xác nhận dự đoán liên tục từ coarse labels, **chưa chép số MAE do cần kiểm tra đúng bảng gốc**.
- **Liên hệ với Leanbot — điểm có và chưa có:** **Có:** sát nhất ở mục tiêu directed heading 360°, nhãn góc thưa và đầu ra góc liên tục, không cần keypoint pose. **Khác:** không dùng detector tích hợp với 24 class/Soft Angular BCE, không xử lý pre-NMS candidates hay closed-loop camera–robot.
- **Vai trò trích dẫn/benchmark:** **Rất cao — Baseline bắt buộc về phương pháp dự đoán góc**: CNN (hoặc cùng backbone và crop Leanbot) với đầu ra sin/cos, train/test trên cùng split và cùng circular MAE; tuyệt đối không chỉ so khác dataset.

**9. A Deep Learning Framework for Accurate Vehicle Yaw Angle Estimation from a Monocular Camera Based on Part Arrangement (2022) — Công trình gần directed heading**
- **Thông tin bài báo (SJR/JCR):** Wenjun Huang và cộng sự, *Sensors*, 22(20), 8027 (2022). [DOI](https://doi.org/10.3390/s22208027). **SJR 2025: Q1 (Instrumentation)**, **JCR 2025: Q2 (Instruments & Instrumentation)** — xếp hạng tạp chí của năm 2025, không phải đánh giá bài năm 2022.
- **Bài toán và phương pháp:** Dự đoán **yaw có hướng** của ô tô từ monocular RGB; YAEN dùng YOLOv5s nhận diện xe và các bộ phận (đèn/gương), rồi part-encoding và CNN decoder dự đoán yaw; tác giả nghiên cứu loss xét tính chu kỳ/góc có dấu.
- **So sánh và kết quả:** Trên tập yaw do tác giả xây dựng với **17.258 ảnh**, mean error báo cáo **dưới 3,1°**, **96,45%** dự đoán sai số dưới 10° và **97 FPS trên RTX 2070 Super**. Đây là kết quả từ hệ thống và dữ liệu riêng, **không phải baseline Leanbot**.
- **Liên hệ với Leanbot — điểm có và chưa có:** **Có:** mục tiêu yaw/heading thật, CNN nhẹ, monocular camera, quan tâm dấu và periodic loss, có thí nghiệm sai số góc. **Khác:** YAEN cần phát hiện **bộ phận xe** để suy ra hướng; Leanbot dùng 24 class heading trực tiếp, không gán nhãn bộ phận hay keypoint. Bài không dùng weighted circular mean từ detection scores; dữ liệu huấn luyện lớn hơn nhiều.
- **Vai trò trích dẫn/benchmark:** **Rất cao — prior work trực tiếp về hướng 360°**; tham khảo protocol đo góc và ground truth, đối chiếu yêu cầu nhãn bộ phận; **có thể** xây dựng một baseline part-based nhẹ nếu Leanbot có bộ phận phân biệt rõ, nhưng không bắt buộc phải tái hiện kiến trúc YAEN.

**Tổng hợp so sánh với Leanbot:**

| Thành phần của bài toán Leanbot | Công trình gần nhất | Kết luận sơ bộ | Thử nghiệm cần làm |
|---|---|---|---|
| Nhãn mềm Gaussian tuần hoàn | CSL 2020, YOLO-CSL 2024, Adaptive 2026 | **Đã có prior work mạnh** | Hard BCE vs CSL/Soft Angular BCE; thử σ khác nhau |
| Kết hợp detection và phân loại góc | YOLO-CSL 2024, Rotation-Adaptive YOLOv5 2023 | **Đã được triển khai**, nhưng đa số cho OBB | Nhánh angle riêng vs 24 class heading chung detector |
| Heading có hướng 360°, góc liên tục từ nhãn thưa | Biternion 2015, YAEN 2022 | **Không mới nếu chỉ tuyên bố dự đoán liên tục 360°** | Sin/cos regression vs 24-class circular classification trên cùng split |
| Hợp nhất class/candidate scores trước NMS | **Chưa xác minh trong các bài trên** | Chỉ là **khoảng trống khảo sát hiện tại**, chưa phải novelty | Argmax vs circular mean một candidate vs grouping nhiều candidate |
| Dataset ít và closed-loop camera robot | Các bài ở trên chưa đánh giá trực tiếp cùng bài toán | **Cần kiểm chứng ưu thế bằng benchmark** | Learning curve 25/50/75/100% data; circular MAE; inference latency; heading control error |

**Thứ tự ưu tiên khi viết bài:** (1) *Problem formulation*: CSL, Biternion, YAEN; (2) *Related Work*: YOLO-CSL, Adaptive Angle 2026, CGD; (3) *Ablation/baselines trên cùng Leanbot dataset*: hard-label YOLO11n, Soft Angular BCE, sin/cos regression, argmax/circular mean; (4) *Discussion về OBB và hiệu năng*: ODC-YOLO, R-YOLOv5, Rotating-YOLO. **Việc trích dẫn một bài trong Related Work không có nghĩa phải tái hiện nó làm baseline.** Tránh dùng số mAP và FPS từ các bộ dữ liệu/phần cứng khác nhau để tuyên bố phương pháp hiện tại vượt trội.

### 2. Toàn bộ hệ thống của bài toán (tính ứng dụng)
- Tối ưu bài toán cho hệ thống máy chủ tính toán yếu : 
    - Tối giản mô hình : YYOLO11n quantization FP16 , OPenvino runtime ,....
    - Cơ chế tracking và ROI ( Region Of Interest) : Giúp giảm kích thước ảnh đầu vào khi thực hiện inference , chỉ tập trugn nhận diện và phân tích gócLeanbot trong vùng quan tâm tracking theo Leanbot để tăng tốc độ xử lý.  
- Hệ thống robot di động Leanbot điều hướng thông qua camera cố định : 
    - Ước lượng được góc Leanbot đang nhìn thấy 
    - Sử dụng PID điều khiển Leanbot ( có thể sử dụng bộ điều khiển khác và so sánh với PID để đối chứng nếu lựa chọn bộ điều khiển khác .)
    - Hệ thống giao tiếp, điều khiển thôgn qua BLE communication với thiết bị chấp hành ( Leanbot )

## C. Khó khăn
- Không
## D. Công việc tiếp theo
- Khảo sát sâu thêm về các bài báo liên quan tới nội dung Orientation Estimation base-on CNN Architecture 
- Chạy lại thực nghiệm và thu thập dữ liệu với góc quay xác thực để lấy kết quả đánh giá, so sánh với các bài báo đã khảo sát 
- Em xin phép nhận thêm ý kiến , đề xuất hướng đi tiếp theo ạ .