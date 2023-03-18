# Phần 10: AlertManager

## 10.1 Rules (tạo recording rule và alerting rule)
Rule có chức năng định nghĩa khi nào cảnh báo. Sau đó Alertmanager sẽ lấy rule này và đem bắn cảnh báo lên mail/chat.

**Bước 1:** Enable rule trong prometheus
```
#vim prometheus.yml
rule_files:
   - "/etc/prometheus/rules/first_rules.yml"
   #- "second_rules.yml"
```
**Bước 2:** Khai báo rules
```
#vim /etc/prometheus/rules/first_rules.yml
groups:
  - name: node
    interval: 15s
    rules:
      #1 ví dụ về recording rule, hỗ trợ khai báo expr alert ở #3
      - record: node_memory_memFree_percent
        expr: 100 - (100 * node_memory_MemFree_bytes{job="node"} / node_memory_MemTotal_bytes{job="node"})

      #2 record rule ở trên có thể đc tận dụng làm biến ở record rule khác
      - record: node_filesystem_free_percent_avg
        expr: avg by(instance)(node_filesystem_free_percent)

      #3 Ví dụ về alert rule dựa trên recording rule.
      - alert: LowMemory
        expr: node_memory_memFree_percent < 20

      #4 Ví dụ về alert rule sử dụng trực tiếp. Có thêm nhãn severity
      - alert: Node down
        expr: up{job="node"} == 0
        labels:
          severity: warning
      - alert: Multiple Nodes down
        expr: avg without(instance)(up{job="node"}) <= 0.5
        labels:
          severity: critical

	  #5 Sử dụng For để delay cảnh báo
      - alert: Node down
        expr: up{job="node"} == 0
		for: 5m\
		
      #6 Ví dụ về annotation giúp hiển thị chú thích cho alert rule
      - alert: node_filesystem_free_percent
        expr: 100 * node_filesystem_free_bytes{job="node"} / node_filesystem_size_bytes{job="node"} < 70
        annotations:
          description: "filesystem {{.Labels.device}} on {{.Labels.instance}} is low on space, current available space is {{.Value}}"

      - alert: LowDiskSpace
        expr: 100 * node_filesystem_free_bytes{job="nodes"} / node_filesystem_size_bytes{job="nodes"} < 10
        labels:
          severity: warning
          environment: prod
        annotations:
          message: "node {{.Labels.instance}} is down"

```

## 10.2. Cài đặt AlertManager

**Bước 1: Cài đặt**
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

**Bước 2: Sửa systemd file**
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


**Bước 3: Sửa config prometheus**
```
# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - 192.168.88.111:9093
            - alertmanager2:9093
```

**Bước 4: Cấu hình Alertmanager**

- Default Route 

- Route

- Sub-route

- Continue

- Multi routes - multi reveivers



# 10.3