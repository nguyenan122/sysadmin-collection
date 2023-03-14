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