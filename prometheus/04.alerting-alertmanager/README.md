# Phan10: AlertManager
**B1: Cài đặt**
```
useradd --no-create-home --shell /bin/false alertmanager
wget https://github.com/prometheus/alertmanager/releases/download/v0.25.0/alertmanager-0.25.0.linux-amd64.tar.gz
tar -xvzf alertmanager-0.25.0.linux-amd64.tar.gz
chown -R alertmanager:alertmanager alertmanager-0.25.0.linux-amd64
cd alertmanager-0.25.0.linux-amd64

mkdir /etc/alertmanager /var/lib/alertmanager
mv alertmanager.yml /etc/alertmanager
chown -R alertmanager:alertmanager /etc/alertmanager
chown -R alertmanager:alertmanager /var/lib/alertmanager
cp alertmanager /usr/local/bin/
cp amtool /usr/local/bin/
```

**B2: Sửa systemd file**
```
vi /etc/systemd/system/alertmanager.service
[Unit]
Description=Alert Manager 
Wants=networkonline.target 
After=networkonline.target

[Service]
Type=simple 
User=alertmanager 
Group=alertmanager
ExecStart=/usr/local/bin/alertmanager \
          --config.file=/etc/alertmanager/alertmanager.yml \
          --storage.path=/var/lib/alertmanager 
Restart=always

[Install]
WantedBy=multi-user.target

- Khởi động Alertmanager lên và truy cập port 9093
systemctl daemon-reload
systemctl start alertmanager
systemctl enable alertmanager
```


**B3: Sửa config prometheus**
```
# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - 192.168.88.111:9093
            - alertmanager2:9093
```

**B4: Cấu hình Alertmanager**
