# BÁO CÁO BÀI THỰC HÀNH LAB 8: KIỂM THỬ HIỆU NĂNG VỚI APACHE JMETER


## 1. MỤC TIÊU BÀI LAB
- **Về kiến thức:** Nắm vững khái niệm, nguyên lý và tầm quan trọng của kiểm thử hiệu năng (Performance Testing/Load Testing) trong quy trình phát triển và vận hành phần mềm.
- **Về công cụ:** Thành thạo cách cài đặt, cấu hình kịch bản và sử dụng các thành phần cốt lõi của công cụ Apache JMeter (Thread Group, Sampler, Listeners).
- **Về kỹ năng:** Có khả năng tự xây dựng kịch bản kiểm thử tải độc lập, giả lập lượng người dùng lớn truy cập đồng thời vào hệ thống API, từ đó biết cách đọc hiểu và phân tích các chỉ số hiệu năng (Response Time, Error Rate, Throughput) để đánh giá sức chịu tải của hệ thống.

---

## 2. KỊCH BẢN KIỂM TRA (TEST SCENARIO)
Để đảm bảo tính độc lập và không trùng lặp đề tài, bài kịch bản lựa chọn kiểm thử hiệu năng trên hệ thống Open API quốc tế cung cấp dữ liệu Pokémon (**PokeAPI**).

- **Đối tượng kiểm thử (API Endpoint):** `https://pokeapi.co/api/v2/pokemon?limit=100`
- **Mục đích kịch bản:** Giả lập hành vi của một lượng lớn người dùng cùng lúc gửi yêu cầu truy vấn thông tin dữ liệu của 100 loài Pokémon để kiểm tra năng suất phản hồi và độ ổn định của Server Backend.
- **Thông số cấu hình tải (Thread Group):**
  - **Number of Threads (Số người dùng giả lập đồng thời):** 100 users.
  - **Ramp-up Period (Thời gian để kích hoạt toàn bộ 100 users):** 10 giây (tăng tải tuyến tính trung bình 10 users mỗi giây).
  - **Loop Count (Số vòng lặp thực thi của mỗi user):** 1 lần.
  - **Tổng số Request dự kiến gửi đi:** 100 Requests.

---

## 3. KẾT QUẢ KIỂM TRA (TEST RESULTS)

Quá trình thực thi kịch bản tải trên Apache JMeter được ghi nhận và giám sát trực quan qua 2 bộ đọc kết quả (Listeners) dưới đây:

### 3.1. Kết quả chi tiết từng Request (View Results Tree)
- **Mục tiêu giám sát:** Theo dõi trạng thái Sống/Chết và mã phản hồi HTTP của từng request riêng lẻ dưới áp lực tăng tải.
- **Minh chứng thực tế:** Toàn bộ 100 request đều được hệ thống đánh dấu tích xanh thành công, trả về mã phản hồi chuẩn `HTTP 200 OK` và không có request nào bị gián đoạn hay ngắt kết nối giữa chừng.
![View Results Tree](./images/view-results-tree.png)

### 3.2. Bảng thống kê hiệu năng tổng quan (Summary Report)
- **Mục tiêu giám sát:** Thu thập các số liệu tính toán thô về thời gian phản hồi hệ thống và năng suất xử lý dữ liệu.
- **Minh chứng thực tế:** Bảng số liệu nhảy đầy đủ thông tin sau khi kết thúc 10 giây chạy tải:
![Summary Report](./images/summary-report.png)

---

## 4. PHÂN TÍCH KẾT QUẢ KIỂM TRA
Dựa trên các số liệu kỹ thuật thực tế thu được từ bảng **Summary Report**, hiệu năng của hệ thống PokeAPI được phân tích chi tiết qua 3 chỉ số cốt lõi sau:

- **Tỷ lệ lỗi hệ thống (Error %):** Đạt mức **0.00%** tuyệt đối. Điều này chứng minh máy chủ Backend của PokeAPI có cấu hình hạ tầng mạng và bộ nhớ đệm (Cache) cực kỳ tốt. Dù bị 100 người dùng dồn tải liên tục trong thời gian ngắn để lấy khối lượng dữ liệu tương đối lớn (100 đối tượng JSON Pokémon), hệ thống vẫn xử lý mượt mà, không sập luồng và không từ chối bất kỳ yêu cầu nào.
- **Thời gian phản hồi (Response Time):**
  - **Thời gian phản hồi trung bình (Average):** Đạt **4053 ms** (~4.05 giây). Đối với một Open API công cộng phục vụ miễn phí toàn cầu, tốc độ xử lý hơn 4 giây dưới áp lực tải đồng thời ở mức chấp nhận được, đảm bảo ứng dụng phía Client không bị rơi vào trạng thái đơ/treo quá lâu.
  - **Thời gian phản hồi nhanh nhất (Min):** Đạt **159 ms** (thường rơi vào các request đầu tiên khi hệ thống chưa chịu áp lực tải).
  - **Thời gian phản hồi chậm nhất (Max):** Lên tới **8841 ms** (~8.8 giây), cho thấy khi lượng người dùng được đẩy lên mức đỉnh điểm (giây thứ 9 và thứ 10), hàng đợi xử lý của Server bắt đầu bị quá tải nhẹ, dẫn đến độ trễ mạng tăng lên rõ rệt.
- **Năng suất xử lý của Server (Throughput):** Đạt tốc độ **9.4 request/giây**. Chỉ số này phản ánh khả năng giải phóng băng thông và phân phối gói tin JSON của máy chủ ở mức ổn định dưới điều kiện kiểm thử.

---

## 5. KẾT LUẬN
- **Về kết quả bài thực hành:** Bài Lab 8 đã hoàn thành chính xác và đầy đủ tất cả các nội dung yêu cầu. Kịch bản kiểm thử hiệu năng hoạt động đúng thiết kế, số liệu thu thập trung thực và minh chứng trực quan rõ ràng.
- **Bài học và giá trị thực tiễn:** Qua bài lab này, sinh viên đã hiểu sâu sắc rằng một phần mềm chạy đúng chức năng (ở mức kiểm thử chức năng như Postman) là chưa đủ, mà còn cần phải chạy khỏe, chạy nhanh khi có nhiều người dùng thực tế truy cập. Việc thành thạo kỹ năng sử dụng Apache JMeter giúp sinh viên có tư duy tối ưu hóa hệ thống, biết cách tìm ra "điểm nghẽn cổ chai" (Bottleneck) của Backend để từ đó đề xuất các giải pháp nâng cấp phần cứng hoặc tối ưu mã nguồn trước khi triển khai sản phẩm lên môi trường thực tế (Production).
