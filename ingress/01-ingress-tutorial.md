# Ingress

### What is Kubernetes Ingress

![ingress](/shot_screen/ingress/ingress.png)

>It is the same in Kubernetes world as well. Ingress means the traffic that enters the cluster and egress is the traffic that exits the cluster.

![](/shot_screen/ingress/nodeport-service.png)

>Without Kubernetes ingress, to expose an application to the outside world, you will add a service Type Loadbalancer to the deployments. Here is how it looks. (I have shown the nodePort just to show the traffic flow)

![](/shot_screen/ingress/ingress-diagram.png)

>>Before the Kubernetes Ingress was stable, a custom Nginx or an HAproxy kubernetes deployment would be exposed as a Loadbalancer service for routing external traffic to the internal cluster services.

>>The routing rules are added as a configmap in the Nginx/HAProxy pods. 

>>Whenever there is a change in dns or a new route entry to be added, it gets updated in the configmap, and pod configs are reloaded, or it gets re-deployed.

>> Kubernetes ingress also follows a similar pattern by having the routing rules maintained as native Kubernetes ingress objects instead of a configmap.

### How Does Kubernetes Ingress work?

![ingress-concept](/shot_screen/ingress/ingress-concept.png)
**Kubernetes Ingress Resource**

>Kubernetes ingress resource is responsible for storing DNS routing rules in the cluster.

**Kubernetes Ingress Controller**

>Kubernetes ingress controllers (Nginx/HAProxy etc.) are responsible for routing by accessing the DNS rules applied through ingress resource.

### Kubernetes Ingress Resource

>The Kubernetes Ingress resource is a native kubernetes resource where you specify the DNS routing rules. Meaning, you map the external DNS traffic to the internal Kubernetes service endpoints.

>It requires an ingress controller for routing the rules specified in the ingress object. Let’s have a look at a very basic ingress resource.

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
  namespace: dev
spec:
  rules:
  - host: test.apps.example.com
    http:
      paths:
      - backend:
          serviceName: hello-service
          servicePort: 80
```

- test.apps.example.com should hit the service named hello-service residing in the dev namespace
- multiple routing endpoints for path-based routing, you can add TLS configuration, etc.

### Kubernetes Ingress Controller

>An ingress object requires an ingress controller for routing traffic.

>And most importantly, the external traffic does not hit the ingress API, instead, it will hit the ingress controller service endpoint configured directly with a load balancer.

>Ingress controller is not a native Kubernetes implementation. Meaning, It doesn’t come default in the cluster.

>An ingress controller is typically a reverse web proxy server implementation in the cluster.

### How Does an Ingress Controller Work?

>Nginx is one of the widely used ingress controllers

![ingress controller nginx](/shot_screen/ingress/ingress-controller-nginx.png)

>1.The nginx.conf file inside the Nginx controller pod is a lua template that can talk to Kubernetes ingress API and get the latest values for traffic routing in real-time. Here is the [template](https://github.com/kubernetes/ingress-nginx/blob/main/rootfs/etc/nginx/template/nginx.tmpl) file.

>2.The Nginx controller talks to Kubernetes ingress API to check if there is any rule created for traffic routing.

>3.If it finds any ingress rules, the Nginx controller generates a routing configuration inside /etc/nginx/conf.d location inside each nginx pod.

>4.For each ingress resource you create, nginx generates a configuration inside /etc/nginx/conf.d location.

>5.The main /etc/nginx/nginx.conf file containes all the configfurations from etc/nginx/conf.d.

>6.If you update the ingress object with new configurations, the nginx config gets updated again and does a graceful reload of configuration.

*If you connect to the Nginx ingress controller pod using exec and check the /etc/nginx/nginx.conf file, you can see all the rules specified in the ingress object applied in the conf file.*

### Ingress & Ingress Controller Architecture

**It shows ingress rules routing traffic to two payment & auth applications**

![ingress architecture diagram](/shot_screen/ingress/ingress-architecture-diagram.png)

### List of Kubernetes Ingress Controller

- Nginx Ingress Controller ([Community](https://github.com/kubernetes/ingress-nginx) & [From Nginx Inc](https://github.com/nginxinc/kubernetes-ingress))
- [Traefik](https://github.com/traefik/traefik)
- [HAproxy](https://www.haproxy.com/blog/dissecting-the-haproxy-kubernetes-ingress-controller/)
- [Contour](https://github.com/projectcontour/contour)
- [GKE Ingress Controller for GKE](https://github.com/kubernetes/ingress-gce)
- [AWS ALB Ingress Controller Fro AKS](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html)
- [Azure Application Gateway Ingress Controller](https://azure.github.io/application-gateway-kubernetes-ingress/)
  
[comparison document](https://docs.google.com/spreadsheets/d/191WWNpjJ2za6-nbG4ZoUMXMpUK8KlCIosvQB0f-oq3k/edit#gid=907731238)

### Kubernetes Ingress FAQs

**Is Ingress a load balancer?**

>Ingress is not a load balancer. It contains all the routing rules, custom headers, and TLS configurations. Ingress controller acts as a load balancer.

**Why do I need an ingress controller?**

>Ingress controller is responsible for the actual routing of external traffic to kubernetes service endpoints. Without an ingress controller, the routing rules added to ingress will not work.

**What is the difference between ingress and Nginx?**

>An ingress is a kubernetes object. Nginx is used as an ingress controller (Reverse proxy).

**Can we route traffic to multiple paths using ingress?**

>Yes. With a single ingress definition, you can add multiple path-based routing configurations.

**Does ingress support TLS configuration?**

>Yes. You can have TLS configurations in your ingress object definition. The TLS certification will be added as a Kubernetes secret and referred to in the ingress object.


**参考文档**

- [Kubernetes Ingress Tutorial For Beginners](https://devopscube.com/kubernetes-ingress-tutorial/)