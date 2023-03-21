# Phần 1: Cài đặt Prometheus

**Bước 1: Chuẩn bị file chạy Prometheus**
```
#1.Chuẩn bị user và thư mục
useradd --no-create-home --shell /bin/false prometheus
mkdir /etc/prometheus
mkdir /var/lib/prometheus
chown prometheus:prometheus /etc/prometheus
chown prometheus:prometheus /var/lib/prometheus

#2.Chuẩn bị file chạy 
wget https://github.com/prometheus/prometheus/releases/download/v2.42.0/prometheus-2.42.0.linux-amd64.tar.gz
tar -xvzf prometheus-2.42.0.linux-amd64.tar.gz
cd prometheus-2.42.0.linux-amd64
cp prometheus /usr/local/bin/
cp promtool /usr/local/bin/
chown prometheus:prometheus /usr/local/bin/prometheus
chown prometheus:prometheus /usr/local/bin/promtool

#3.Copy file config mẫu
cp prometheus.yml /etc/prometheus/prometheus.yml

#4.Chuẩn bị thư mục console, thư viện
cp -r consoles /etc/prometheus
cp -r console_libraries /etc/prometheus
chown -R prometheus:prometheus /etc/prometheus/consoles
chown -R prometheus:prometheus /etc/prometheus/console_libraries
```


**Bước 2: Chuẩn bị Systemd file**

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
**Bước 3: Start prometheus**
```
systemctl daemon-reload
systemctl start prometheus
```

**Bonus**

Để reload nhanh Prometheus, ta có thể bật option --web.enable-lifecycle và chạy lệnh curl
```
# curl -XPOST http://ip-của-prometheus:9090/-/reload 
```
Để kiểm tra config của prometheus có sai ở đâu không ta sử dụng lệnh
```
# promtool check config /etc/prometheus/prometheus.yml
```
