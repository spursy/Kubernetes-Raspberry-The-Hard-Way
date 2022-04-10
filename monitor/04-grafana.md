# Grafana

### Create grafana database configmap

```bash
k apply -f ./resource/grafana/grafana-database-config.yaml
```

### Deploy grafana po

```bash
k apply -f ./resource/grafana/grafana-deployment.yaml
```

### Create grafana service

```bash
k apply -f ./resource/grafana/grafana-service.yaml
```

![grafana-web](/shot_screen/monitoring/grafana-web.png)

### load dashboard template

- get grafana dashboard id
[grafana official web](https://grafana.com/grafana/dashboards/?search=kubernetes)

- load grafana dashboard template
![load grafana dashboard template](/shot_screen/monitoring/grafana-lod-bashboard-template.png)



**参考文档**

- [grafana code](https://github.com/bibinwilson/kubernetes-grafana)
- [How To Setup Grafana On Kubernetes](https://devopscube.com/setup-grafana-kubernetes/)