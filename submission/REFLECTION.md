Đối với ứng dụng Gia sư AI lưu trữ thông tin học sinh và năng lực học sinh, dự án dễ mắc phải **Anti-pattern #2: Partition theo high-cardinality (như `student_id`)** nhất.

Vì ứng dụng chủ yếu truy vấn dữ liệu theo từng học sinh để cá nhân hóa lộ trình, nhóm phát triển rất dễ đi theo lối mòn phân vùng bảng (partition) theo `student_id` hoặc `user_id`. Do số lượng học sinh lớn và tăng dần theo thời gian, việc này sẽ tạo ra hàng triệu thư mục con (partition) cực kỳ nhỏ trên Object Storage. Hậu quả là làm phình to metadata, gây thắt nút cổ chai hiệu năng khi đọc/ghi (file-skipping mất tác dụng) và tăng chi phí API gọi storage.

**Giải pháp:** Nhóm sẽ không partition theo `student_id`. Thay vào đó, phân vùng theo các trường có cardinality thấp như `date` hoặc `grade_level` (khối lớp), và áp dụng **`Z-ORDER BY student_id`** để tối ưu hóa việc bỏ qua file (data skipping) khi truy vấn thông tin học sinh cụ thể.
