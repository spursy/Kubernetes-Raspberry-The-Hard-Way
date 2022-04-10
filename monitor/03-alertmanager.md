# AlertManager

### A config map for AlertManager configuration

- Prometheus should have the correct alert manager service endpoint in its config.yaml as shown below to send the alert to Alert Manager

```bash
alerting:
   alertmanagers:
      - scheme: http
        static_configs:
        - targets:
          - "alertmanager.monitoring.svc:9093"
```

- All the alerting rules have to be present on Prometheus config based on your needs

```bash
rule_files:
      - /etc/prometheus/prometheus.rules
```

- Set alert template path, email, and other alert receiving configurations

```bash
k apply -f ./resource/alertmanager/alertmanager-configmap.yaml
```

### A config Map for AlertManager alert templates

```bash
k apply -f ./resource/alertmanager/alert-template-configmap.yaml
```

### Alert Manager Kubernetes Deployment

```bash
k apply -f ./resource/alertmanager/alertmanager-deployment.yaml
```

### Alert Manager service to access the web UI

```bash
k apply -f ./resource/alertmanager/alertmanager-service.yaml
```

### verification web

![alertmanager web](/shot_screen/monitoring/alertmanager-web.png)


**参考文档**

- [kubernetes-alert-manager code](https://github.com/bibinwilson/kubernetes-alert-manager)
- [Setting Up Alert Manager on Kubernetes – Beginners Guide](https://devopscube.com/alert-manager-kubernetes-guide/)


*下一篇文章[Grfana](/monitor/04-grafana.md]*