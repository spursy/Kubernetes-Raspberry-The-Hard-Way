# Service Account Role

>In this blog, you will learn how to create kubernetes role for a service account and use it with the pods, deployments, and cronjobs

**1. Create webapps Namespaces**

```bash
k create namespace webapps
```

**2. Create Kubernetes Service Account**

Create na service account named *app-service-account* that bounds to *webapps* namespace

```bash
k apply -f -<<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-service-account
  namespace: webapps
EOF
```

**3. Validate Kubernetes Role Permissions**

```bash
k get po -n webapps --as=system:serviceaccount:webapps:default
```

![k8s-sa-forbid](/shot_screen/security/k8s-sa-forbid.png)

**4. Create a Role For API Access**

Specify the list of API access for Kubernetes resources

```bash
k apply -f -<<EOF
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: webapps
rules:
  - apiGroups:
        - ""
        - apps
        - autoscaling
        - batch
        - extensions
        - policy
        - rbac.authorization.k8s.io
    resources:
      - pods
      - componentstatuses
      - configmaps
      - daemonsets
      - deployments
      - events
      - endpoints
      - horizontalpodautoscalers
      - ingress
      - jobs
      - limitranges
      - namespaces
      - nodes
      - pods
      - persistentvolumes
      - persistentvolumeclaims
      - resourcequotas
      - replicasets
      - replicationcontrollers
      - serviceaccounts
      - services
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
EOF
```

**5. Create a Rolebinding(Attaching Role to ServiceAccount)**

```bash
k apply -f -<<EOF
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: webapps 
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-role 
subjects:
- namespace: webapps 
  kind: ServiceAccount
  name: app-service-account 
EOF
```

**5. Validate Kubernetes Role Permissions Again**

```bash
k get po -n webapps --as=system:serviceaccount:webapps:default
```

![k8s-sa-access](/shot_screen/security/k8s-sa-access.png)

**6. Create Kubernete Deployment Use Sepcial SA**

```bash
k apply -f -<<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: webapps
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      serviceAccountName: app-service-account
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
EOF
```


