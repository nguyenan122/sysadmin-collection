# Phần 3: Cài đặt Pushgateway và đẩy Metric lên PushGateway

## 3.1 Cài đặt
**Bước 1: Cài đặt**
```
wget https://github.com/prometheus/pushgateway/releases/download/v1.5.1/pushgateway-1.5.1.linux-amd64.tar.gz
tar -xvzf pushgateway-1.5.1.linux-amd64.tar.gz
cd pushgateway-1.5.1.linux-amd64
useradd --no-create-home --shell /bin/false pushgateway
cp pushgateway /usr/local/bin/
chown pushgateway:pushgateway /usr/local/bin/pushgateway
```
**Bước 2: Tạo systemd file**

vim /etc/systemd/system/pushgateway.service

```
[Unit]
Description=Prometheus Pushgateway 
Wants=network-online.target 
After=network-online.target

[Service] 
User=pushgateway 
Group=pushgateway 
Type=simple
ExecStart=/usr/local/bin/pushgateway

[Install]
WantedBy=multi-user.target
```
**Bước 3: Start pushgateway**
```
systemctl daemon-reload
systemctl restart pushgateway
systemctl enable pushgateway
```
Kiểm tra web pushgateway: http://192.168.88.12:9091/

![pushgateway01](/prometheus/03.pushgateway/images/01.pushgateway.PNG)


**Bước 4: Cấu hình prometheus.yml lấy data from pushgatway**

vim /etc/prometheus/prometheus.yml  
```
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["192.168.88.12:9090"]
  - job_name: "worker-node1"
    scheme: http
    static_configs:
      - targets: ["192.168.88.13:9100"]
  #thêm push gateway  
  - job_name: pushgateway
    honor_labels: true
    static_configs:
      - targets: ["192.168.88.13:9091"]
```


**Bước 5: Đẩy dữ liệu lên gateway**
```
VD1: Basic
Khung mẫu: echo "metric1_name 123456" | curl --data-binary @- http://<pushgateway_address>:<port>/metrics/job/<job_name>/<label1>/<value1>/<label2>/<value2>
Lệnh chạy thực tế: echo "metric1_name 123456" | curl --data-binary @- http://192.168.88.12:9091/metrics/job/app



VD2: Basic
cat <<EOF | curl --data-binary @- http://192.168.88.12:9091/metrics/job/job1/instance/instance1 
# TYPE my_job_duration gauge
my_job_duration{label="val1"} 42
# TYPE another_metric counter
# HELP another_metric Just an example. 
another_metric 12
EOF
```
Kết quả: 

![pushgateway01](/prometheus/03.pushgateway/images/02.pushgateway.PNG)



## 3.2 Hướng dẫn gửi metric lên PushGateway
**Group data gửi sang pushgateway- dựa theo uri**
```
$ cat <<EOF | curl --data-binary @- http://192.168.88.12:9091/metrics/job/archive/db/mysql
# TYPE metric_one counter
metric_one{label="val1"} 11
# TYPE metric_two gauge
# HELP metric_two Just an example.
metric_two 100
EOF

$ cat <<EOF | curl --data-binary @- http://192.168.88.12:9091/metrics/job/archive/app/web
# TYPE metric_one counter
metric_one{label="val1"} 22
# TYPE metric_two gauge
# HELP metric_two Just an example.
metric_two 200
EOF

Kết quả:
curl http://192.168.88.12:9091/metrics | grep archive
metric_one{db="mysql",instance="",job="archive",label="val1"} 11
metric_two{db="mysql",instance="",job="archive"} 100
metric_one{app="web",instance="",job="archive",label="val1"} 22
metric_two{app="web",instance="",job="archive"} 200
```

**Cập nhập - sửa đổi dữ liệu đang có trên pushgateway**
```
POST: Sửa không xóa
cat <<EOF | curl -XPOST --data-binary @- http://192.168.88.12:9091/metrics/job/archive/app/web 
# TYPE metric_one counter
metric_one{label="val1"} 33
EOF
kết quả metric_one thay từ 22 lên 33

PUT: Sửa có xóa (ghi đè)
cat <<EOF | curl -XPUT --data-binary @- http://192.168.88.12:9091/metrics/job/archive/app/web 
# TYPE metric_one counter
metric_one{label="val1"} 44
EOF
kết quả là metric_one từ 22 thành 44, nhưng metric_two đã bị xóa

DELETE: xóa toàn bộ hoặc 1 phần nhỏ
curl –X DELETE http://192.168.88.12:9091/metrics/job/archive/app/web
kết quả toàn bộ /app/web đều bị xóa hết metrics
```
