# Phần 8: Cài đặt giám sát trong kubernetes

## 8.1 Cài đặt kube-prometheus-stack
**Bước 1**: Cài đặt helm
```
wget https://get.helm.sh/helm-v3.10.3-linux-amd64.tar.gz
tar -zxvf helm-v3.10.3-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/
rm -rf helm-v3.10.3-linux-amd64.tar.gz linux-amd64
```

**Bước 2:** Cài prometheus stack
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm search repo prometheus-community/kube-prometheus-stack –versions
helm pull prometheus-community/kube-prometheus-stack --version 29.0.1
tar -xzf kube-prometheus-stack-29.0.1.tgz
cd kube-prometheus-stack
Ta sửa: values.yaml ta sửa   admissionWebhooks:     enable: false
helm install prometheus .
kubectl get ds
kubectl patch ds prometheus-prometheus-node-exporter --type "json" -p '[{"op": "remove", "path" : "/spec/template/spec/containers/0/volumeMounts/2/mountPropagation"}]'
```

**Bước 3**: Kiểm tra + giải thích chức năng deployment và sts
```
# k get deployments.apps
NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
prometheus-kube-prometheus-operator   1/1     1            1           103s
prometheus-kube-state-metrics         1/1     1            1           103s
prometheus-grafana                    1/1     1            1           103s
Chức năng của 3 deployment: 
prometheus-grafana - This is the grafana instance that gets deployed with the helm chart.
prometheus-kube-prometheus-operator - Responsible for deploying & managing the prometheus instance.
prometheus-kube-state-metrics - Collects cluster level metrics (pods, deployments, etc).

# kubectl get statefulset
NAME                                                   READY   AGE
alertmanager-prometheus-kube-prometheus-alertmanager   1/1     2m48s
prometheus-prometheus-kube-prometheus-prometheus       1/1     2m46s
Chức năng của 2 statefullset
prometheus-prometheus-kube-prometheus-prometheus - Tạo prometheus pod.
alertmanager-prometheus-kube-prometheus-alertmanager - Tạo alertmanager Pod
```
**Bước 4:** Mở port hoặc ingress truy cập vào Prometheus / Grafana

A~mở NodePort

```
# kubectl edit svc prometheus-kube-prometheus-prometheus
Đổi ClusterIP thành NodePort với nodePort: 31000
# k expose deployment prometheus-grafana --type=NodePort --port=3000 --target-port=3000 --name prometheus-grafana2
Truy cập vào grafana với mật khẩu như sau: admin / prom-operator
```
B~ Mở Ingress
```
ingress config here
```

## 8.2 Giám sát 1 dịch vụ application trong cụm k8s
Ví dụ như ta muốn theo dõi trạng thái của API Project1 ta cần làm các bước sau:
- 1 application chạy như java, tomcat
- Lên /metrics để giám sát tiến trình java, tomcat ở trên (giống dựng exporter)
- Tạo ServiceMonitor để pull metric về lưu vào prometheus


