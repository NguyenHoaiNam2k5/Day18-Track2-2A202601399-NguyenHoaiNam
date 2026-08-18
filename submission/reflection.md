# Reflection — Lakehouse Anti-Pattern

Anti-pattern em có nguy cơ gặp nhất là **bỏ qua OPTIMIZE, dẫn đến small-file problem**. Pipeline LLM observability nhận dữ liệu liên tục từ nhiều model và endpoint. Nếu mỗi micro-batch tạo một file Parquet riêng, các tầng Bronze và Silver sẽ nhanh chóng chứa hàng nghìn file nhỏ. Dữ liệu vẫn đúng nhưng hệ thống phải tốn nhiều thao tác liệt kê metadata, mở file và lập kế hoạch truy vấn. Vì vậy dashboard có thể chậm rõ rệt dù tổng dung lượng dữ liệu chưa lớn.

Nguy cơ này dễ bị bỏ qua vì pipeline thường chạy tốt ở giai đoạn thử nghiệm. Trong NB2, compaction làm giảm mạnh số file, còn Z-ORDER giúp loại bỏ phần lớn file không liên quan khi lọc theo `user_id`. NB6 cũng cho thấy maintenance không chỉ là chạy `VACUUM`; file orphan chưa từng được commit có thể không xuất hiện trong transaction log.

Em sẽ theo dõi số file và kích thước file trung bình, chạy OPTIMIZE theo lịch hằng ngày, Z-ORDER theo các cột lọc phổ biến, đồng thời quét orphan định kỳ. Cảnh báo phải dựa trên các chỉ số này thay vì chờ truy vấn chậm hoặc chi phí lưu trữ tăng mới xử lý.
