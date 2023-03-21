# Phần 4: AlertManager

## 10.1 Rules (tạo recording rule và alerting rule)
Rule có chức năng định nghĩa khi nào cảnh báo. Sau đó Alertmanager sẽ lấy rule này và đem bắn cảnh báo lên mail/chat.

**Bước 1:** Enable rule trong prometheus

vim /etc/prometheus/prometheus.yml
```
rule_files:
   - "/etc/prometheus/rules/first_rules.yml"
   #- "second_rules.yml"
```
**Bước 2:** Khai báo rules

mkdir /etc/prometheus/rules/
vim /etc/prometheus/rules/first_rules.yml
```
groups:
  - name: node
    interval: 15s
    rules:
      #1 ví dụ về recording rule, hỗ trợ khai báo expr alert ở #3
      - record: node_memory_memFree_percent
        expr: 100 - (100 * node_memory_MemFree_bytes{job="worker-node1"} / node_memory_MemTotal_bytes{job="worker-node1"})

      #2 record rule ở trên có thể đc tận dụng làm biến ở record rule khác
      - record: node_filesystem_free_percent_avg
        expr: avg by(instance)(node_filesystem_free_percent)

      #3 Ví dụ về alert rule dựa trên recording rule.
      - alert: LowMemory
        expr: node_memory_memFree_percent < 20

      #4 Ví dụ về alert rule sử dụng trực tiếp. Có thêm nhãn severity
      - alert: Node down
        expr: up{job="worker-node1"} == 0
        labels:
          severity: warning
      - alert: Multiple Nodes down
        expr: avg without(instance)(up{job="worker-node1"}) <= 0.5
        labels:
          severity: critical

      #5 Sử dụng For để delay cảnh báo
      - alert: Node down
        expr: up{job="worker-node1"} == 0
        for: 5m

      #6 Ví dụ về annotation giúp hiển thị chú thích cho alert rule
      - alert: node_filesystem_free_percent
        expr: 100 * node_filesystem_free_bytes{job="worker-node1"} / node_filesystem_size_bytes{job="worker-node1"} < 70
        annotations:
          description: "filesystem {{.Labels.device}} on {{.Labels.instance}} is low on space, current available space is {{.Value}}"

      - alert: LowDiskSpace
        expr: 100 * node_filesystem_free_bytes{job="worker-node1"} / node_filesystem_size_bytes{job="worker-node1"} < 10
        labels:
          severity: warning
          environment: prod
        annotations:
          message: "node {{.Labels.instance}} is down"

      #7 Tạo 1 rule giả để test AlertManager
      - alert: Node down TEST
        expr: up{job="worker-node1"} == 1
        labels:
          severity: test-only
```
Thực hiện kiểm tra và reload config
```
# promtool check config /etc/prometheus/prometheus.yml
# curl -XPOST http://192.168.88.12:9090/-/reload
```
Kiểm tra thấy rule đã được apply như 2 hình dưới: (hình 1)


![rule01](/prometheus/04.alerting-alertmanager/images/01.rule.PNG)

(hình 2)

![rule01](/prometheus/04.alerting-alertmanager/images/02.alert.PNG)


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

vim /etc/systemd/system/alertmanager.service
```
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
ps aux | grep alertmanager
```


**Bước 3: Sửa config prometheus**

vim /etc/prometheus/prometheus.yml
```
# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - 192.168.88.12:9093
```
Kiểm tra AlertManager đã nhận rule trên prometheus chưa: http://192.168.88.12:9093/#/alerts
![alertmanager03](/prometheus/04.alerting-alertmanager/images/03.alert-manager.PNG)


**Bước 4: Cấu hình Alertmanager**

- Default Route 

- Route

- Sub-route

- Continue

- Multi routes - multi reveivers



# 10.3