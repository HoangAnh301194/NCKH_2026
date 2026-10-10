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

*Lưu ý:* Các công trình YOLO dưới đây chủ yếu đánh giá góc của **rotated bounding box (OBB)**, thường mang tính đối xứng 180°, còn Leanbot cần **directed heading 360°**. Chỉ số mAP của OBB không phải sai số góc (MAE). **Các quartile dưới đây dùng dữ liệu năm 2025:** **SJR** = phân hạng SCImago dựa trên Scopus; **JCR (JIF quartile)** = phân hạng Journal Citation Reports của Clarivate dựa trên Web of Science. Một tạp chí có thể thuộc nhiều ngành với Q khác nhau. JCR 2025 được phát hành năm 2026. Không đồng nhất SJR Q với Scopus CiteScore Q hay JCR Q; các xếp hạng bên dưới có liên kết nguồn kiểm tra. Nếu không có dữ liệu JCR đáng tin cậy thì ghi rõ thay vì suy đoán.

**1. Rotated Object Detection Using Adaptive Angle Classification and Dynamic Sample Matching (2026)**
- **Xuất bản:** Liu Han và cộng sự, *Chinese Journal of Engineering*, 48(3), 586–598 (2026). [DOI](https://doi.org/10.13374/j.issn2095-9389.2025.06.09.006). **SJR 2025: Q2** (*Engineering*; SJR ≈ 0,386). **JCR 2025: Không có JIF quartile được xác nhận** (chưa ghi nhận trong WoS Core Collection). [SJR/coverage](https://journalindexes.com/gongcheng-kexue-xuebao-chinese-journal-of-engineering-20959389/) · [Nhà xuất bản](https://journal.ustb.edu.cn/Journals/index.htm).
- **Bài toán:** Phát hiện đối tượng quay trong ảnh viễn thám, giảm sai số gián đoạn góc và cải thiện matching đối tượng.
- **Phương pháp:** YOLOv8, *shape-aware adaptive angle classification* dùng nhãn mềm Gaussian tuần hoàn với độ rộng điều chỉnh theo tỷ lệ hình dạng; thêm dynamic sample matching.
- **Kết quả:** Tác giả báo cáo mAP **78,6% trên DOTA** và **92,4% trên dữ liệu ký tự công nghiệp**; đây là chỉ số phát hiện OBB, không phải heading MAE.
- **Liên hệ:** Gần Soft Angular BCE của Leanbot; đáng kiểm tra việc dùng \(\sigma\) cố định so với thích nghi.

**2. ODC-YOLO: An Optimized YOLOv5 Method for Detecting Objects in Remote Sensing Images (2025)**
- **Xuất bản:** Qing Liu và cộng sự, *Remote Sensing Letters*, 16(10), 1120–1130 (2025). [DOI](https://doi.org/10.1080/2150704X.2025.2529599). **SJR 2025: Q2** (*Electrical and Electronic Engineering*, SJR ≈ 0,434). **JCR 2025: Q4** (*Remote Sensing* và *Imaging Science & Photographic Technology*, JIF 1,5). [SJR](https://www.journalsbase.com/journals/remote-sensing-letters) · [JCR 2025](https://apa.letpub.com/index.php?journalid=8680&page=journalapp&view=detail). **Lưu ý:** Một số trang vẫn hiển thị **JCR Q3 năm 2024**, không phải năm 2025.
- **Bài toán:** Phát hiện vật thể nhỏ, hướng quay bất kỳ, phân bố dày đặc trong ảnh viễn thám.
- **Phương pháp:** Cải tiến YOLOv5 bằng ODConv/Res2Net, M-RFB và Circular Smooth Labels (CSL) cho góc của box.
- **Kết quả:** Tác giả báo cáo cải thiện **3,45 điểm phần trăm mAP so với YOLOv10n trên DOTA**, kết quả phụ thuộc cấu hình đánh giá.
- **Liên hệ:** Cho thấy tích hợp CSL vào YOLO không còn là ý tưởng mới riêng biệt.

**3. Rotating-YOLO: A Novel YOLO Model for Remote Sensing Rotating Object Detection (2025)**
- **Xuất bản:** Zhiguo Liu, Yuqi Chen, Yuan Gao, *Image and Vision Computing*, 154, 105397 (2025). [DOI](https://doi.org/10.1016/j.imavis.2024.105397). **SJR 2025: Q1** (*Computer Vision and Pattern Recognition*, SJR ≈ 0,881). **JCR 2025: Q2** (*Computer Science, Artificial Intelligence*) **hoặc Q1** (*Computer Science, Software Engineering*), JIF 5,0. [SJR](https://scienceaijournal.com/journals/image-and-vision-computing-0262-8856) · [JCR theo ngành](https://www.akaturk.com/journals/25549?lang=en).
- **Bài toán:** Phát hiện các đối tượng nhỏ, xoay nhiều hướng với mô hình gọn.
- **Phương pháp:** Cải tiến YOLOv8; biểu diễn hình học rotated box bằng Gaussian và dùng Gaussian loss, kết hợp fusion/attention.
- **Kết quả:** Tác giả báo cáo **giảm 33,25% số tham số** và **tăng 1,4 điểm mAP** so với YOLOv8 baseline.
- **Liên hệ:** Gaussian trên hình học OBB khác Gaussian soft labels cho 24 lớp directed heading của Leanbot.

**4. Detection of Objects in Satellite and Aerial Imagery Using Channel and Spatially Attentive YOLO-CSL for Surveillance (2024)**
- **Xuất bản:** Divyansh Chaurasia, B. D. K. Patro, *Image and Vision Computing*, 147, 105070 (2024). [DOI](https://doi.org/10.1016/j.imavis.2024.105070). **SJR 2025: Q1** (*Computer Vision and Pattern Recognition*, SJR ≈ 0,881). **JCR 2025: Q2** (*Computer Science, Artificial Intelligence*) **hoặc Q1** (*Computer Science, Software Engineering*), JIF 5,0. [SJR](https://scienceaijournal.com/journals/image-and-vision-computing-0262-8856) · [JCR theo ngành](https://www.akaturk.com/journals/25549?lang=en).
- **Bài toán:** Oriented object detection trong ảnh vệ tinh/hàng không.
- **Phương pháp:** YOLOv5 với nhánh góc riêng, Circular Smooth Labels và BCEWithLogits; thêm channel/spatial attention.
- **Kết quả:** Báo cáo **mAP 57,86 trên DOTA-v2**, so sánh với các detector trên cùng dataset; khoảng **25 triệu tham số, 54 GFLOPs**.
- **Liên hệ:** Rất gần ở cấp **CSL + YOLO + BCE**, nhưng dự đoán angle branch cho OBB thay vì dùng class heading của Leanbot.

**5. Rotated Object Detection with Circular Gaussian Distribution (2023)**
- **Xuất bản:** Hang Xu và cộng sự, *Electronics*, 12(15), 3265 (2023). [DOI](https://doi.org/10.3390/electronics12153265). **SJR 2025: Q2** (*Control and Systems Engineering*, SJR ≈ 0,623). **JCR 2025: Q2** (*Engineering, Electrical & Electronic*, JIF 2,9); **Q3** ở *Computer Science, Information Systems* và *Physics, Applied*. [SJR](https://scienceaijournal.com/journals/electronics-2079-9292) · [JCR – MDPI](https://www.mdpi.com/journal/electronics/stats).
- **Bài toán:** Góc rotated box có tính tuần hoàn và bị gián đoạn gần biên.
- **Phương pháp:** Circular Gaussian Distribution (CGD), loss theo KL divergence; dựa trên CenterNet-FPN.
- **Kết quả:** Trên HRSC2016, cấu hình R-50-FPN, **CGD mAP07 90,52 so với CSL 89,98**; với mAP12, **CGD 97,76 so với CSL 95,13**.
- **Liên hệ:** Gợi ý so sánh huấn luyện phân bố góc với soft-target BCE hiện tại.

**6. Object Detection of Flexible Objects with Arbitrary Orientation Based on Rotation-Adaptive YOLOv5 (2023)**
- **Xuất bản:** Jiajun Wu và cộng sự, *Sensors*, 23(10), 4925 (2023). [DOI](https://doi.org/10.3390/s23104925). **SJR 2025: Q1** (*Instrumentation*, SJR ≈ 0,802; một số ngành khác Q2). **JCR 2025: Q2** (*Instruments & Instrumentation*, JIF 4,0). [SJR](https://scienceaijournal.com/journals/sensors-1424-8220) · [JCR – MDPI](https://www.mdpi.com/journal/sensors/stats).
- **Bài toán:** Phát hiện vật thể mềm/dài có góc quay bất kỳ.
- **Phương pháp:** YOLOv5 nhận diện rotated box; mã hóa góc bằng vector Gaussian và tối ưu loss.
- **Kết quả:** Trên dữ liệu FO của tác giả, mAP **47,7% (YOLOv5s) lên 57,9% (R-YOLOv5s)**; FPS **44,1 xuống 41,9**.
- **Liên hệ:** Gaussian-coded angle trong YOLO đã có tiền lệ; cần đối chiếu cơ chế 24-class của Leanbot.

**7. Arbitrary-Oriented Object Detection with Circular Smooth Label (2020) — Nghiên cứu nền tảng**
- **Xuất bản:** Xue Yang, Junchi Yan, *ECCV 2020* (hội nghị; **SJR-Journal Q: không áp dụng; JCR-Journal Q: không áp dụng**; cần đánh giá theo xếp hạng hội nghị riêng). [DOI](https://doi.org/10.1007/978-3-030-58598-3_40).
- **Bài toán:** Khắc phục gián đoạn góc khi oriented bounding box quay qua biên góc.
- **Phương pháp:** Chuyển hồi quy góc sang classification và dùng Circular Smooth Labels; khảo sát các hàm window, trong đó có Gaussian.
- **Kết quả:** So sánh với các detector/biểu diễn góc trên DOTA, HRSC2016, ICDAR2015 và MLT.
- **Liên hệ:** Soft Angular BCE của Leanbot cần được phân biệt rõ với CSL; không nên tự nhận Gaussian circular label là đóng góp mới.

**8. Biternion Nets: Continuous Head Pose Regression from Discrete Training Labels (2015) — Baseline 360°**
- **Xuất bản:** Lucas Beyer, Alexander Hermans, Bastian Leibe, *GCPR 2015*, LNCS 9358, tr. 157–168 (hội nghị; **SJR-Journal Q: không áp dụng; JCR-Journal Q: không áp dụng**; cần đánh giá theo xếp hạng hội nghị riêng). [DOI](https://doi.org/10.1007/978-3-319-24947-6_13).
- **Bài toán:** Ước lượng hướng liên tục 360° từ nhãn góc rời rạc.
- **Phương pháp:** CNN hồi quy trực tiếp vector \((\cos\theta,\sin\theta)\), tránh gián đoạn 0°/360°.
- **Kết quả:** Đánh giá với các baseline regression/classification cho hướng đầu; cần đọc bảng metric gốc trước khi chuyển số liệu vào báo cáo.
- **Liên hệ:** Baseline trực tiếp cho bài toán Leanbot; cần thử nghiệm với cùng dataset và sai số circular MAE.

**Nhận định bước đầu:** Circular Gaussian soft labels và việc tích hợp vào YOLO đã được nghiên cứu; weighted circular mean và các kỹ thuật suy ra góc liên tục cũng cần được đối chiếu thêm. Điểm có thể đào sâu là **giám sát 24 lớp heading 360° ngay trong đầu ra detection**, xử lý score trước NMS và hiệu quả khi nhãn thưa/dữ liệu ít. Chưa có bằng chứng đủ để kết luận phương pháp hiện tại tối ưu hoặc có novelty độc lập.

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