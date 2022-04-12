# Service Account

>This tutorial will guide you through the process of creating the service account, role and role binding to have API access to kubernetes cluster

>The best and recommended way to allow API access to kubernetes cluster is through service accounts following the principle of least privilege(PoLP)

### Basic Service Account

**1.Create service account in a namespace**

```bash
k create namespace devops-tools

k create serviceaccount api-service-account -n devops-tools

or

k apply -f -<<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: api-service-account
  namespace: devops-tools
EOF  
```

**2.Create cluster role**

```bash
k apply -f -<<EOF
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: api-cluster-role
  namespace: devops-tools
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

**3.Create a ClusterRole Binding**

```bash
k apply -f -<<EOF
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: api-cluster-role-binding
subjects:
- namespace: devops-tools 
  kind: ServiceAccount
  name: api-service-account 
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: api-cluster-role 
EOF
```

**4.Validate Service Account Accrss Using Kubectl**

- The following command checks if the api-service-account in the devops-tools namespaces can list the pods
  
```bash
 k auth can-i get pods --as=system:serviceaccount:devops-tools:api-service-account
```

- The following command checks if service account has permissions to delete deployments

```bash
k auth can-i delete deployments --as=system:serviceaccount:devops-tools:api-service-account
```

**5.Validate Service Account Access Using API Call**

```bash
// get k8s dns host
k8s_host=$(kubectl get endpoints | grep kubernetes | awk '{print $2}')

// get secrets name of sa
service_account_secrets=$(k get serviceaccount api-service-account  -n devops-tools -o=jsonpath='{.secrets[0].name}')

// get decode token of sa secrets
decoded_secrets_token=$(k get secrets $service_account_secrets -n devops-tools -o=jsonpath='{.data.token}' | base64 -d)

// '-k or -key' can ingnore ssl
curl -k  -H "Authorization: Bearer $decoded_secrets_token" "https://$k8s_host/api/v1/namespaces"
```

### Default Service Accoubt

**1.Create namespaces**

```bash
k create namespace devops-tools-2

k get sa -n devops-tools-2
```

**2.Get k8s api host url**

```bash
kubectl config view -o jsonpath='{.clusters[0].cluster.server}'
```

![k8s-host-url](/shot_screen/security/k8s-host-url.png)

**3.Create Test Pod**

```bash
k run -i --tty --rm curl-tns --image=curlimages/curl:latest -n devops-tools -n devops-tools-2  /bin/sh
```

**4.Validate Service Account Access Using API Call**

```bash
// set env variable in the pod of step2
CA_CERT=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
NAMESPACE=$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace)

curl --cacert $CA_CERT -H "Authorization: Bearer $TOKEN" "https://192.168.20.1:6443/api/v1/namespaces/$NAMESPACE/services/"
```

![k8s-forbidden-api-resource](/shot_screen/security/k8s-forbidden-api-resource.png)

**5.Assign Cluster Role To SA**

```bash
kubectl create rolebinding default-view \
  --clusterrole=view \
  --serviceaccount=devops-tools-2:default \
  --namespace=devops-tools-2
```

**6.Validate Service Account Access Using API Call Again**

```bash
curl --cacert $CA_CERT -H "Authorization: Bearer $TOKEN" "https://192.168.20.1:6443/api/v1/namespaces/$NAMESPACE/pods/"
```

![k8s-access-api-resource](/shot_screen/security/k8s-access-api-resource.png)


**参考文档**

- [How To Create Kubernetes Service Account For API Access](https://devopscube.com/kubernetes-api-access-service-account/)
- [权限控制：探索Kubernetes的Service Accounts](https://juejin.cn/post/6844903952870293511#comment)