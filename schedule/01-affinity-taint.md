# Affinity & Taints

>nodeSelector — This is a simple Pod scheduling feature that allows scheduling a Pod onto a node whose labels match the nodeSelector labels specified by the user

>Node Affinity — This is the enhanced version of the nodeSelector introduced in Kubernetes 1.4 in beta. It offers a more expressive syntax for fine-grained control of how Pods are scheduled to specific nodes

>Inter-Pod Affinity — This feature addresses the third scenario above. Inter-Pod affinity allows co-location by scheduling Pods onto nodes that already have specific Pods running

>spec.affinity.podAntiAffinity field of the Pod spec. There, we can specify a list of labels which will be compared with the labels of Pods running on the node. If the labels match, the Pod won’t be placed alongside with the Pod having this label and running on the node

>Taints and tolerations, which ensure that a given Pod does not end up on the inappropriate node. If the taint is applied to a node, only those Pods that have tolerations for this taint can be scheduled onto that node

### nodeSelector

#### node label

```bash
k get nodes --show-labels

k label nodes m1 disktype=ssd
```

**default label attached to Kubernetes nodes**

- beta.kubernetes.io/arch
- kubernetes.io/hostname
- failure-domain.beta.kubernetes.io/zone
- failure-domain.beta.kubernetes.io/region
- beta.kubernetes.io/instance-type
- beta.kubernetes.io/os
- beta.kubernetes.io/arch


#### node selector demo

```bash
k apply -f -<<EOF
apiVersion: v1
kind: Pod
metadata:
  name: httpd
  labels:
    env: prod
spec:
  containers:
  - name: httpd
    image: httpd
    imagePullPolicy: IfNotPresent
  nodeSelector:
    kubernetes.io/hostname: m1
EOF
```

### Node and Pod Affinity and Anti-Affinity

>Users can now “soft” scheduling rules. If the “soft” rule is not met, the scheduler can still schedule a Pod onto a specific node.

>New affinity feature supports Pod co-location. Users can constraint a Pod against the label of another Pod running on a specific node rather than against node labels. With this feature, users can control what Pods end up on the same node and which one don’t. This feature is called “inter-Pod affinity/anti-affinity.

#### Node Affinity

Node affinity allows scheduling Pods to specific nodes. There are a number of use cases for node affinity, including the following:

- Spreading Pods across different availablity zones to improve resilience and availability of applications in the cluster
- Allocating nodes for memory-intensive Pods. In this case, you can have a few nodes dedicated to less compute-intensive Pods and one or two nodes with enough CPU and RAM dedicated to memory-intensive Pods

**Kubernetes support for "hard" and "soft" node affinity**

- "hard" affinity: set a precise rule that should be met in order for a Pod to be scheduled on a node (requiredDuringSchedulingIgnoredDuringExecution)
- "soft" affinity: less strict,you can ask the scheduler to try to run the set of Pod in availability zone XYZ,but if it's impossible, allow some of those Pods to run in other Availability Zone (preferredDuringSchedulingIgnoredDuringExecution)


```bash
k apply -f -<<EOF
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/e2e-az-name
            operator: In
            values:
            - e2e-az1
            - e2e-az2
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: custom-key
            operator: In
            values:
            - custom-value
  containers:
  - name: with-node-affinity
    image: k8s.gcr.io/pause:2.0
EOF
```

#### Inter-Pod Affinity/Anti-Affinity

- Spread the Pods of a service accross nodes and/or availability zones to reduce correlated failuers. 
- Give a Pod "exclusive" accross to a node to guarantee resource isolation
- Don't schedule the Pods of a particular service on the same nodes as Pods of another service that may interface with the performance of the Pods of the first service

The Pod affinity/anti-affinity may be formalized as follows. "This Pod should or should node run in an X node if that X node is already running one or more pods that meet rule Y"

**Similarly to node affinity, Pod affinity and anti-affinity support "hard" and "soft" rule**



```bash
k apply -f -<<EOF
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: security
            operator: In
            values:
            - S1
        topologyKey: failure-domain.beta.kubernetes.io/zone
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: security
              operator: In
              values:
              - S2
          topologyKey: kubernetes.io/hostname
  containers:
  - name: with-pod-affinity
    image: k8s.gcr.io/pause:2.0
EOF
```


**experiment**

- create one pod with label *security:s1*

```bash
apiVersion: v1
kind: Pod
metadata:
  name: s1
  labels:
    security: s1
spec:
  containers:
  - name: bear
    image: supergiantkir/animals:bear
```

- create second pod with Pod anti-affinity rule

```bash
k apply -f -<<EOF
apiVersion: v1
kind: Pod
metadata:
  name: pod-s2
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: security
            operator: In
            values:
            - s1
        topologyKey: kubernetes.io/hostname
  containers:
  - name: pod-antiaffinity
    image: supergiantkir/animals:hare
EOF
```

**Pod affinity and anti-affinity rules in deployment**

```bash
k apply -f -<<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-server
spec:
  selector:
    matchLabels:
      app: web-store
  replicas: 3
  template:
    metadata:
      labels:
        app: web-store
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - web-store
            topologyKey: "kubernetes.io/hostname"
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - store
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: web-app
        image: nginx:1.12-alpine
EOF
```