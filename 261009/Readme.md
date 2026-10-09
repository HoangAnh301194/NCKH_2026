# Báo cáo tiến độ nghiên cứu ngày 09/10/2026

**Đề tài:** Ước lượng góc định hướng 360° của robot Leanbot từ camera cố định bằng CNN-based Object Detection.

**Mã nguồn tham khảo:** [Leanbot_orientation_estimation-CNN_base](https://github.com/HoangAnh301194/Leanbot_orientation_estimation-CNN_base)

## A. Tổng quan dự án

1. **Bài toán:** Phát hiện vị trí và ước lượng góc định hướng của Leanbot trên mặt phẳng 2D bằng một camera cố định.
2. **Mục tiêu:** Ước lượng hướng quay đủ 360°, phân biệt góc dương/âm mà không cần dùng mô hình keypoint pose estimation.
3. **Giải pháp:** Sử dụng mô hình object detection nhỏ (YOLO11n), kết hợp biểu diễn góc tuần hoàn để suy ra góc định hướng.

## B. Nội dung trọng tâm và kết quả bước đầu

1. **Xây dựng dữ liệu:** Đã triển khai quy trình thu thập ảnh, tách nền và tự động tạo nhãn bounding box. Bộ dữ liệu của phiên bản huấn luyện ngày 11/09/2026 gồm **204 ảnh** (192 ảnh có Leanbot, 12 ảnh nền âm), **1.728 bounding box**.
2. **Thiết kế nhãn góc:** Biểu diễn hướng quay thành **24 lớp**, cách nhau **15°**, bao phủ toàn bộ 360°.
3. **Huấn luyện mô hình:** Sử dụng **YOLO11n** và **Soft Angular BCE** nhằm xét đến tính tuần hoàn và khoảng cách giữa các lớp góc.
4. **Suy ra góc liên tục:** Dùng **weighted circular mean** trên điểm dự đoán của các lớp để nhận góc có dấu trong miền [−180°, 180°].
5. **Triển khai inference:** Đã chuẩn bị mô hình **OpenVINO FP16** với chế độ **Full Detection 640×640** và **ROI Tracking 160×160**, có cơ chế chuyển về tìm kiếm toàn ảnh khi mất bám.
6. **Kết quả chức năng:** Đã xây dựng được pipeline phát hiện Leanbot và suy ra góc định hướng ở cả bốn góc phần tư. **Độ chính xác góc và tốc độ thực thi chưa được kết luận định lượng trong báo cáo này.**

## C. Khó khăn và hạn chế

1. Số lượng ảnh gốc còn ít, chủ yếu thu thập trong điều kiện camera và môi trường tương đối cố định.
2. Chưa kiểm chứng đầy đủ sai số tại các góc trung gian giữa hai lớp nhãn 15°.
3. Đầu ra góc có thể dao động hoặc không chắc chắn khi thay đổi ánh sáng, vị trí robot hay xảy ra mất bám.
4. Chưa có tập kiểm thử với góc tham chiếu độc lập để lượng hóa sai số của hệ thống.

## D. Công việc tiếp theo

1. Chuẩn hóa bộ dữ liệu và xây dựng tập kiểm thử với ground truth góc độc lập.
2. Đo sai số góc tuần hoàn, độ ổn định dự đoán và thời gian inference thực tế.
3. Bổ sung dữ liệu cho các trường hợp dự đoán lỗi hoặc mất tracking.
4. Sau khi có kết quả cơ sở, xây dựng thí nghiệm so sánh và xác định hướng phát triển bài báo.

> **Phạm vi báo cáo:** Tổng hợp nội dung đã thực hiện và khó khăn hiện tại; chưa trình bày khảo sát tài liệu, baseline hay kết luận về tính mới của phương pháp.
