# Ingress Controller

### Ingress & Nginx Ingress controller Architecture

![ingress control demo diagram](/shot_screen/ingress/ingress-demo-diagram.png)

[ingress controller demo](https://github.com/scriptcamp/nginx-ingress-controller/tree/main/manifests)

### Prerequisites

**1.Create a Namespace**

```bash
k create namesapce ingress-nginx
```

**2.Create Admission Controller Roles & Service Account**

```bash
k apply -f ./resource/ingress-controller/admission-controller-account.yaml
```

**3.Create Validating Webhook Configuration**

```bash
k apply -f ./resource/ingress-controller/validation-webhook.yaml
```

**4.Deploy Jobs To Update Webhook Certificates**

```bash
// 宿主机上替换镜像
sudo docker pull dyrnq/kube-webhook-certgen:v1.1.1
sudo docker tag dyrnq/kube-webhook-certgen:v1.1.1 k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1

k apply -f ./resource/ingress-controller/jobs.yaml
```

**5.Create Ingress Controller Roles & Service Account**

```bash
k apply -f ./resource/ingress-controller/ingress-service-account.yaml
```

**6.Create Configmap**

```bash
k apply -f ./resource/ingress-controller/configmap.yaml
```

**7.Create Ingress Controller & Admission ControllerServices**

```bash
k apply -f ./resource/ingress-controller/service.yaml
```

**8.Create Ingress Class**

```bash
k apply -f ./resource/ingress-controller/ingress-class.yaml
```

**9.Create Ingress Controller Deployment**

```bash
// 宿主机上替换镜像
sudo docker pull willdockerhub/ingress-nginx-controller:v1.1.1
sudo docker tag willdockerhub/ingress-nginx-controller:v1.1.1 k8s.gcr.io/ingress-nginx/controller:v1.1.1

k apply -f ./resource/ingress-controller/deployment.yaml
```

### Deploy Jobs To Update Webhook Certificates

>The ValidatingWebhookConfiguration works only over HTTPS. So it needs a CA bundle.

>We use [kube-webhook-certgen](https://github.com/jet/kube-webhook-certgen) to generate a CA cert bundle with the first job. The generated CA certs are stored in a secret named ingress-nginx-admission

>The second job patches the ValidatingWebhookConfiguration object with the CA bundle.

Once the jobs are executed, you can describe the ValidatingWebhookConfigurationand, you will see the patched bundle.

```bash
kubectl describe ValidatingWebhookConfiguration ingress-nginx-admission
```

### Create Configmap

With this configmap, you can customize the Nginx settings. For example, you can set custom headers and most of the Nginx settings. Please refer to the [official community documentation](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/) for all the supported configurations.

### Create Ingress Controller & Admission Controller Services

ingress-nginx-controller creates a Loadbalancer in the respective cloud platform you are deploying.

You can get the load balancer IP/DNS using the following command.

```bash
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
  annotations:
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  externalTrafficPolicy: Local
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - appProtocol: http
    name: http
    port: 80
    protocol: TCP
    targetPort: http
  - appProtocol: https
    name: https
    port: 443
    protocol: TCP
    targetPort: https
  selector:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
  ### Nodeport OR LoadBalancer  
  type: NodePort
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
  name: ingress-nginx-controller-admission
  namespace: ingress-nginx
spec:
  ports:
  - appProtocol: https
    name: https-webhook
    port: 443
    targetPort: webhook
  selector:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
  type: ClusterIP
```

```bash
kubectl --namespace ingress-nginx get services -o wide -w ingress-nginx-controller
```

### Create Ingress Controller Deployment

Create a file named deployment.yaml

### Map a Domain Name To Ingress Loadbalancer IP

The primary goal of Ingress is to receive external traffic to services running on Kubernetes. Ideally in projects, a DNS would be mapped to the ingress controller Loadbalancer IP.

This can be done via the respective DNS provider with the domain name you own.

>Info: For internet-facing apps, it will be public DNS pointing to the public IP of the load balancer. If it’s an internal app, it will be an organization’s private DNS mapped to a private load balancer IP.

#### Single DNS Mapping
You can map a single domain directly as an A record to the load balancer IP. Using this you can have only one domain for the ingress controller and multiple path-based traffic routing.

For example,

www.example.com --> Loadbalancer IP
You can also have path-based routing using this model.

Few examples

```bash
http://www.example.com/app1
http://www.example.com/app2
http://www.example.com/app1/api
http://www.example.com/app2/api
```

**Wildcard DNS Mapping**

If you map a wildcard DNS to the load balancer, you can have dynamic DNS endpoints through ingress.

Once you add the wildcard entry in the DNS records, you need to mention the required DNS in the ingress object and the Nginx ingress controller will take care of routing it to the required service endpoint.

For example, check the following two mappings.

```bash
*.example.com --> Loadbalancer IP
*.apps.example.com --> Loadbalancer IP 
```

### Deploy a Demo Application

**1.create a namespace named dev**

```bash
k create namespace ingress-nginx
```

**2.create hello app deployment**

```bash
k apply -f -<<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app
  namespace: ingress-nginx
spec:
  selector:
    matchLabels:
      app: hello
  replicas: 3
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
      - name: hello
        image: "nginx"
EOF
```

**3.create hello app service**

```bash
k apply -f -<<EOF
apiVersion: v1
kind: Service
metadata:
  name: hello-service
  namespace: ingress-nginx
  labels:
    app: hello
spec:
  type: ClusterIP
  selector:
    app: hello
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
EOF
```

**4.verify hello app service**

```bash
kubectl run -it --rm --restart=Never alpine --image=alpine sh

nslookup hello-service.ingress-nginx.svc.cluster.local
```

![hello-app-service](/shot_screen/ingress/hello-app-service.png)

### Create Ingress Object for Application

Now let’s create an ingress object to access our hello app using a DNS. An ingress object is nothing but a setup of routing rules.

If you are wondering how the ingress object is connected to the Nginx controller, the ingress controller pod connects to the Ingress API to check for rules and it updates its nginx.conf accordingly.

```bash
k apply -f -<<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
  namespace: ingress-nginx
spec:
  ingressClassName: nginx
  rules:
  - host: "demo.apps.mlopshub.com"
    http:
      paths:
        - pathType: Prefix
          path: "/"
          backend:
            service:
              name: hello-service
              port:
                number: 80
EOF
```

**verify ingress resource**

```bash
k get ingress
k get svc -n ingress-nginx
```

![ingress-verification](/shot_screen/ingress/ingress-verification.png)


**verify ingress dns**

- add DNS provider at raspberry 

```bash
vi /etc/hosts

--add new at the end
10.99.216.162   demo.apps.mlopshub.com
```

- verify *demo.apps.mlopshub.com* connection

```bash
curl -XGET demo.apps.mlopshub.com
```

![ingress-dns](/shot_screen/ingress/ingress-dns.png)


**参考文档**

- [How to Setup Nginx Ingress Controller On Kubernetes – Detailed Guide](https://devopscube.com/setup-ingress-kubernetes-nginx-controller/)
- [Nginx ingress controller by kubernetes community](https://github.com/kubernetes/ingress-nginx)
- [Nginx ingress controller by Nginx Inc](https://github.com/nginxinc/kubernetes-ingress)