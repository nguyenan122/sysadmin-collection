# Phần 6: PromQL

### 6.1 Đọc hiểu metric

![iamge1](https://github.com/nguyenan122/sysadmin-collection/blob/main/prometheus/06.PromQL/images/01.metric-type.JPG)
Các loại metric:
 - **Counter**: Bộ đếm số lần xuất hiện (tăng dần/không giảm). Ví dụ: total request, total exception, total job execute...
 - **Gauge**: Hiển thị giá trị current - đồng hồ đo (có thể tăng/giảm). VD: CPU util, free memory, concurent request...
 - **Histogram**: Hiển thị theo dạng đồ thị phân bố đều trên 1 khoảng thời gian
 - **Summary**: giống với histogram


