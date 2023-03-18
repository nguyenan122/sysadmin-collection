# Phần 6: PromQL

### 6.1 Đọc hiểu metric

![iamge1](/prometheus/06.PromQL/images/01.metric-type.JPG)


1~Các loại metric:
 - **Counter**: *Bộ đếm số lần xuất hiện (tăng dần/không giảm). Ví dụ: total request, total exception, total job execute...*
 - **Gauge**: *Hiển thị giá trị current - dạng đồng hồ đo (có thể tăng/giảm). VD: CPU util, free memory, concurent request...*
 - **Histogram**: *Hiển thị theo dạng đồ thị phân bố đều trên 1 khoảng thời gian*
 - **Summary**: *giống với histogram*

2~Đọc hiểu metric:
```
http_request_total_count{instance="node01:3000", method="POST"} = 500.000
```
Giải thích:
- **http_request_total_count**: là TÊN METRIC Name.
- **instance="node01:3000", method="POST"**: được gọi là NHÃN label.
- **500.000**: là Value của METRIC.


### 6.2 Làm quen với PromQL
