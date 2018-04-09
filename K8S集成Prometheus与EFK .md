## K8S集成Prometheus与EFK 

### 背景

```
最近在做K8S的私有云项目建设，集群搭建功能基本完善，但是监控系统一直空缺，主要包括资源和日志两部分。看到部分私有云厂商使用了Prometheus来做资源的监控，包括网络、CPU、Memory等资源，所以私下也搭建了Prometheus集成K8S。
```

### 前置条件

```
K8S集群搭建完成，服务均正常运行
```

### 一、Prometheus 进程方式

#### 1.Prometheus 安装

```
进入官网https://prometheus.io/download/，下载解压
配置文件，即配置每个job的metric数据源
启动，可以直接使用./prometheus启动或者使用nohup ./prometheus &启动
默认启动端口为9090，输入ip：port的形式访问，查看是否安装成功
```

#### 2.安装Grafana

```
Prometheus是用来收集数据的，内部的图表显示功能不够完善，所以使用Grafana作为图表可视化工具
进入http://grafana.org/download/官网，使用wget下载rpm包
wget https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-5.0.4-1.x86_64.rpm
yum localinstall grafana-5.0.4-1.x86_64.rpm
systemctl start grafana-server，至此安装完成
```

#### 3.使用

```
1.进入Grafana，默认Grafana端口为3000，默认用户名密码是admin/admin,配置Data Sources数据源，将Prometheus的http地址加载到数据源中；
2.替换dashboards
Grafana默认的dashboards显示不健全，可以下载和K8S比较匹配的dashboard
下载模版https://grafana.com/dashboards/315，导入到Grafana内，并修改Prometheus的yml配置
3.重新启动Grafana，systemctl restart grafana-server，以下是修改后的Prometheus的配置
```

```
global:
  scrape_interval:     10s
  evaluation_interval: 10s
scrape_configs:
  - job_name: node
    static_configs:
      - targets: ['172.20.0.63:9100','172.20.0.99:9100','172.20.0.102:9100','172.20.0.131:9100']
        labels:
          instance: node
  - job_name: kubernetes-nodes-cadvisor
    static_configs:
      - targets: ['172.20.0.63:4194','172.20.0.99:4194', '172.20.0.102:4194', '172.20.0.131:4194']
        labels:
          instance: kubernetes-nodes-cadvisor
    kubernetes_sd_configs:
      - role: node
    relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
    metric_relabel_configs:
      - action: replace
        source_labels: [id]
        regex: '^/machine\.slice/machine-rkt\\x2d([^\\]+)\\.+/([^/]+)\.service$'
        target_label: rkt_container_name
        replacement: '${2}-${1}'
      - action: replace
        source_labels: [id]
        regex: '^/system\.slice/(.+)\.service$'
        target_label: systemd_service_name
        replacement: '${1}'
```



### 二、Prometheus 容器方式

#### 1.安装Prometheus 

```
1.创建命名空间monitoring
kubectl create -f namespace.yaml
2.创建权限
kubectl create -f rbac.yaml
2.创建 node-exporter
kubectl create -f prometheus-node-exporter-daemonset.yaml
kubectl create -f prometheus-node-exporter-service.yaml
3.创建 kube-state-metrics
kubectl create -f kube-state-metrics-deployment.yaml
kubectl create -f kube-state-metrics-service.yaml
4.创建 node-directory-size-metrics
kubectl create -f node-directory-size-metrics-daemonset.yaml
5.创建 prometheus
kubectl create -f prometheus-core-configmap.yaml
kubectl create -f prometheus-core-deployment.yaml
kubectl create -f prometheus-core-service.yaml
kubectl create -f prometheus-rules-configmap.yaml
```

#### 2.安装Grafana

```
1.安装grafana service
kubectl create -f grafana-svc.yaml
2.导入configmap
kubectl create configmap "grafana-etc" --from-file=grafana.ini --namespace=monitoring
3.创建gragana deployment
kubectl create -f grafana-deployment.yaml
```



### 三、EFK（Elasticsearch + Fluentd + Kibana）

```
EFK的安装方式参考 https://github.com/gjmzj/kubeasz/tree/master/manifests/efk，唯一需要注意的是Docker启动方式中，log-driver需要改成json-file的格式。
另外，安装Fluentd的时候，需要设置label，
kubectl label nodes XXXX beta.kubernetes.io/fluentd-ds-ready=true
```



### 相关文档：

https://www.cnblogs.com/163yun/p/7716253.html

https://www.cnblogs.com/sfnz/p/6566951.html

https://blog.csdn.net/wenwst/article/details/76624019

https://prometheus.io/docs/

https://grafana.com

https://grafana.com/dashboards

https://github.com/gjmzj/kubeasz

