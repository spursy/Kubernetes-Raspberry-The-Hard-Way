# Prometheus

### 创建 monitoring 空间

```bash
kubectl create namespace monitoring
```

### 创建 cluster role

```bash
k apply -f ./resource/prometheus/promethes-cluster-role.yaml
```

### 创建 prometheus config map

[prometheus config map 在线链接](https://raw.githubusercontent.com/bibinwilson/kubernetes-prometheus/master/config-map.yaml)

```bash
k apply -f ./resource/prometheus/prometheus-config-map.yaml
```

**prometheus.yaml**

>This is the main Prometheus configuration which holds all the scrape configs, service discovery details, storage locations, data retention configs, etc)

>kubernetes-apiservers: It gets all the metrics from the API servers

>kubernetes-nodes: It collects all the kubernetes node metrics

>kubernetes-pods: All the pod metrics get discovered if the pod metadata is annotated with prometheus.io/scrape and prometheus.io/port annotations
>kubernetes-cadvisor: Collects all cAdvisor metrics

>kubernetes-service-endpoints: All the Service endpoints are scrapped if the service metadata is annotated with prometheus.io/scrape and prometheus.io/port annotations. It can be used for black-box monitoring

**prometheus.rules**
>This file contains all the Prometheus alerting rules

### 创建 prometheus deployment

```bash
k apply -f ./resource/prometheus/prometheus-deployment.yaml
```

**临时通过 pod forward 实现外部访问**

```bash
k get po -n monitoring | grep prometheus

kubectl port-forward prometheus-monitoring-XXXXX 8080:9090 -n monitoring
```

### 创建 prometheus nodeport service

```bash
k apply -f ./resource/prometheus/prometheus-service.yaml
```

**点击导航栏的 status 下的 target 查看到所有的 endpoint 都已经绑到 prometheus 上**

![promethes 页面](/shot_screen/monitoring/prometheus-web.png)

**查看指标 container_cpu_usage_seconds_total 的数据**

![promethes container_cpu_usage_seconds_total 指标](/shot_screen/monitoring/prometheus-container-cpu-usage-metric.png)


**参照文档**

- [setup prometheus monitoring on kubernetes](https://devopscube.com/setup-prometheus-monitoring-on-kubernetes/)


*下一篇文章[Setup Kube State Metrics](/monitor/02-setup-kube-state-metrics.md)*