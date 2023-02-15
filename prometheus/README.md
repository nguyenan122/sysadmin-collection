Mục Lục

> [Phần 1: Cài đặt Prometheus](https://github.com/nguyenan122/sysadmin-collection/blob/main/prometheus/README.md#phan-1-cai-dat-prometheus)

> [Phần 2: Cài đặt Node_Exporter](https://github.com/nguyenan122/sysadmin-collection/blob/main/prometheus/README.md#phan-2-cai-dat-node-exporter)

> [Phần3: (Tùy chọn) SSL và Authen cho Prometheus + Node_Exporter](https://github.com/nguyenan122/sysadmin-collection/blob/main/prometheus/README.md#phan3-option-ssl-and-authen-prometheus--node_exporter)



# Phan 1: Cai dat Prometheus
```
#1.Chuẩn bị user và thư mục
useradd --no-create-home --shell /bin/false prometheus
mkdir /etc/prometheus
mkdir /var/lib/prometheus
chown prometheus:prometheus /etc/prometheus
chown prometheus:prometheus /var/lib/prometheus

#2.Chuẩn bị file chạy binary
wget https://github.com/prometheus/prometheus/releases/download/v2.41.0/prometheus-2.41.0.linux-amd64.tar.gz
tar -xvzf prometheus-2.41.0.linux-amd64.tar.gz
cd prometheus-2.41.0.linux-amd64
cp prometheus /usr/local/bin/
cp promtool /usr/local/bin/
chown prometheus:prometheus /usr/local/bin/prometheus
chown prometheus:prometheus /usr/local/bin/promtool

#3.Chuẩn bị thư mục console, thư viện
cp -r consoles /etc/prometheus
cp -r console_libraries /etc/prometheus
chown -R prometheus:prometheus /etc/prometheus/consoles
chown -R prometheus:prometheus /etc/prometheus/console_libraries

#4.Chuẩn bị file config
cp prometheus.yml /etc/prometheus/prometheus.yml


```
**Chuẩn bị Systemd file**

vi /etc/systemd/system/prometheus.service
```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries \
    --web.enable-lifecycle

[Install]
WantedBy=multi-user.target

```
**Khởi động lại**
```
systemctl daemon-reload
systemctl start prometheus
```


# Phan 2: Cai dat Node Exporter
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
tar -xvzf node_exporter-1.5.0.linux-amd64.tar.gz
cd node_exporter-1.5.0.linux-amd64
cp node_exporter /usr/local/bin/
useradd --no-create-home --shell /bin/false node_exporter
chown node_exporter:node_exporter /usr/local/bin/node_exporter
```
**Chuẩn bị Systemd file**
```
[Unit]
Description=Node Exporter 
Wants=network-online.target 
After=network-online.target

[Service] 
User=node_exporter 
Group=node_exporter 
Type=simple
ExecStart=/usr/local/bin/node_exporter
#ExecStart= /usr/local/bin/node_exporter --web.config.file=/etc/node_exporter/config.yml \
                                         --web.listen-address=:9100 \
                                         --web.telemetry-path="/metrics"

[Install]
WantedBy=multi-user.target
```
**Khởi động lại**
```
systemctl daemon-reload
systemctl start node_exporter
```


# Phan3: (Option) SSL and Authen Prometheus + Node_Exporter


**B1: Tạo mật khẩu authen và ssl**
```
# htpasswd -nBC 12 "" | tr -d ':\n'
Kết quả: $2y$12$Cc1OeSGajFbIHCP3YECjpeZusdmUNFU4xNdOoMjbT/yatqUpIouPu
mkdir /etc/node_exporter
cd /etc/node_exporter
openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout node_exporter.key -out node_exporter.crt -subj "/C=US/ST=VietNam/L=HaNoi/O=TuanDA/CN=localhost"
```
**B2: Sửa node_exporter config**
```
# cat /etc/node_exporter/config.yml
tls_server_config:
  cert_file: node_exporter.crt
  key_file: node_exporter.key
basic_auth_users:
  prometheus: $2y$12$Cc1OeSGajFbIHCP3YECjpeZusdmUNFU4xNdOoMjbT/yatqUpIouPu
```
**B3: Sửa prometheus config tại node1**
```
# vim /etc/prometheus/prometheus.yml  
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["192.168.88.111:9090"]
  - job_name: "node1"
    scheme: https
    basic_auth:
      username: prometheus
      password: 123
    tls_config:
      ca_file: /etc/prometheus/node_exporter.crt
      insecure_skip_verify: true
    static_configs:
      - targets: ["192.168.88.111:9100"]
```
**B4: Khởi động lại**
```
# systemctl restart node_exporter
# systemctl restart prometheus
```


# Phan8: Pushgateway
**B1: Cài đặt**
```
wget https://github.com/prometheus/pushgateway/releases/download/v1.5.1/pushgateway-1.5.1.linux-amd64.tar.gz
tar -xvzf pushgateway-1.5.1.linux-amd64.tar.gz
cd pushgateway-1.5.1.linux-amd64
useradd --no-create-home --shell /bin/false pushgateway
cp pushgateway /usr/local/bin/
chown pushgateway:pushgateway /usr/local/bin/pushgateway
```
**B2: Tạo systemd file**
```
vim /etc/systemd/system/pushgateway.service
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
**B3: Start pushgateway**
```
systemctl daemon-reload
systemctl restart pushgateway
systemctl enable pushgateway
```
**B4: Cấu hình prometheus.yml lấy data from pushgatway**
```
scrape_configs:
  - job_name: pushgateway
  honor_labels: true
  static_configs:
    - targets: ["192.168.1.168:9091"]
```


**B5: Push data into gateway**
```
VD1: Basic
echo "metric1_name 123456" | curl --data-binary @- http://<pushgateway_address>:<port>/metrics/job/<job_name>/<label1>/<value1>/<label2>/<value2>

VD2: Basic
cat <<EOF | curl --data-binary @-http://localhost:9091/metrics/job/job1/instance/instance1 
# TYPE my_job_duration gauge 
my_job_duration{label="val1"} 42
# TYPE another_metric counter
# HELP another_metric Just an example. 
another_metric 12
EOF
```

**Group data gửi sang pushgateway- dựa theo uri**
```
$ cat <<EOF | curl --data-binary @- http://localhost:9091/metrics/job/archive/db/mysql
# TYPE metric_one counter
metric_one{label="val1"} 11
# TYPE metric_two gauge
# HELP metric_two Just an example.
metric_two 100
EOF
$ cat <<EOF | curl --data-binary @- http://localhost:9091/metrics/job/archive/app/web
# TYPE metric_one counter 
metric_one{label="val1"} 22 
# TYPE metric_two gauge
# HELP metric_two Just an example.
metric_two 200 
EOF

Kết quả:
curl localhost:9091/metrics | grep archive
metric_one{db="mysql",instance="",job="archive",label="val1"} 11
metric_two{db="mysql",instance="",job="archive"} 100
metric_one{app="web",instance="",job="archive",label="val1"} 22
metric_two{app="web",instance="",job="archive"} 200
```

**Cập nhập - sửa đổi dữ liệu đang có trên pushgateway**
```
POST: Sửa không xóa
cat <<EOF | curl -XPOST --data-binary @- http://localhost:9091/metrics/job/archive/app/web 
# TYPE metric_one counter
metric_one{label="val1"} 33
EOF
kết quả metric_one thay từ 22 lên 33

PUT: Sửa có xóa (ghi đè)
cat <<EOF | curl –X PUT --data-binary @- http://localhost:9091/metrics/job/archive/app/web 
# TYPE metric_one counter
metric_one{label="val1"} 44 
EOF
kết quả là metric_one từ 22 thành 44, nhưng metric_two đã bị xóa

DELETE: xóa toàn bộ hoặc 1 phần nhỏ
curl –X DELETE http://localhost:9091/metrics/job/archive/app/web
kết quả toàn bộ /app/web đều bị xóa hết metrics
```
