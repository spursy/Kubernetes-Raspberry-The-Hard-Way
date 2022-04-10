# Setup Kube State Metrics

### 
### Kube State Metrics

>Kube State metrics is a service that talks to the Kubernetes API server to get all the details about all the API objects like deployments, pods, daemonsets, Statefulsets, etc.

- Node status, node capacity (CPU and memory)
- Replica-set compliance (desired/available/unavailable/updated status of replicas per deployment)
- Pod status (waiting, running, ready, etc)
- Ingress metrics
- PV, PVC metrics
- Daemonset & Statefulset metrics
- Resource requests and limits
- Job & Cronjob metrics

[prometheus 支持哪些 metrics](https://github.com/kubernetes/kube-state-metrics/tree/master/docs)

### Setup Kube State Metrics

- A Service Account

```bash
k apply -f ./resource/kube-state-metrics/kube-state-metrics-service-account.yaml
```

- Cluster Role – For kube state metrics to access all the Kubernetes API objects

```bash
k apply ./resource/kube-state-metrics/kube-state-metrics-cluster-role.yaml
```

- Cluster Role Binding – Binds the service account with the cluster role

```bash
k apply ./resource/kube-state-metrics/kube-state-metrics-cluster-role-binding.yaml
```

- Kube State Metrics Deployment

```bash
// 下载国内镜像 
sudo docker pull kubesphere/kube-state-metrics:v2.3.0
sudo docker tag kubesphere/kube-state-metrics:v2.3.0 k8s.gcr.io/kube-state-metrics/kube-state-metrics:v2.4.2

k apply ./resource/kube-state-metrics/kube-state-metrics-deployment.yaml
```

- Service – To expose the metrics

```bash
k apply -f ./resource/kube-state-metrics/kube-state-metrics-service.yaml
```

- add prometheus metrics config
  
```
- job_name: 'kube-state-metrics'
  static_configs:
    - targets: ['kube-state-metrics.kube-system.svc.cluster.local:8080']
```


**参考文档**

- [How To Setup Kube State Metrics on Kubernetes](https://devopscube.com/setup-kube-state-metrics/)
- [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics/tree/master/examples/standard)


*下一篇文章[Alertmanager](/monitor/03-alertmanager.md]*