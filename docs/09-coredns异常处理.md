# coredns 异常处理

## connection timed out; no servers could be reached

### 1. 创建一个 alpine pod 后并检查该 pod 中 /etc/resolv.conf 文件

```
kubectl run -it --rm --restart=Never alpine --image=alpine sh

// 检查 dns 的配置
## cat /etc/resolv.conf
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```
- nameserver: 用来配置 dns 服务的地址，可以指定多个地址
- domain: 用来指定本地的域名，没有配置 search 的情况下，search 默认值为 domain 的值
- search: 指定多个域名，用空格分隔。当访问的域名无法被 DNS 服务器解析时， resolver 会将该域名后加上 search 的值，重新请求 DNS，直到被正确解决或列表循环结束
  

### 2. 检查 DNS 的 pod 是否运行

```
kubectl get pods --namespace=kube-system -l k8s-app=kube-dns
```

### 3. 检查 DNS 的 service 是否正常运行

```
kubectl get svc -n kube-system
```

### 4. 检查 DNS 的 endpoint 是否正常

```
kubectl get ep kube-dns --namespace=kube-system
```

### 5. 检查 COREDNS 的 pod 是否有正常的日志

- 在 coredns configmap 中的 Corefile 中添加 log

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        log   // 添加的 log
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          pods insecure
          upstream
          fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
```


- 监听日志

```
kubectl logs --namespace=kube-system -l k8s-app=kube-dns
```

### 2. 检查多节点的 node 的 ip 地址

- 检查 node 的 ip 地址是否是我们原先的设定的 ip 地址
```
kubectl get node -owide
```

- 若不是可通过 node-ip 参数
```
cat /etc/systemd/system/kubelet.service

--- ---
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=docker.service
Requires=docker.service
[Service]
ExecStart=/usr/local/bin/kubelet \
  --config=/var/lib/kubelet/kubelet-config.yaml \
  --docker=unix:///var/run/docker.sock \
  --docker-endpoint=unix:///var/run/docker.sock \
  --image-pull-progress-deadline=2m \
  --kubeconfig=/var/lib/kubelet/kubeconfig \
  --network-plugin=cni \
  --register-node=true \
  --node-ip=192.168.56.126 \
  --v=2
Restart=on-failure
RestartSec=5
[Install]
WantedBy=multi-user.target
```

### 3. 检查 flannel ds pod 绑定物理的网络设备

```
// 检查 flannel ds 的日志
kubectl logs -n kube-system -l app=flannel

kubectl logs -f pods/kube-flannel-ds-54d4g -n kube-system
```

**<u>需要确保绑定网络设备是 eth0</u>**

### 4. 若发现上面都没问题

- 在副本节点上安装 dnsutils
  
```
sudo apt-get install dnsutils
```

- 使用 nslookup 检查

```
nslookup baidu.com

--- ---
nslookup baidu.com
Server:		192.168.20.1
Address:	192.168.20.1#53

Non-authoritative answer:
Name:	baidu.com
Address: 220.181.38.148
Name:	baidu.com
Address: 39.156.69.79
```



## 2. coredns crashes with error “Failed to list *v1.Service: Get https://10.96.0.1:443/api/v1/****: dial tcp 10.96.0.1:443: connect: no route to host”

```
sudo systemctl stop kubelet
sudo systemctl stop docker
sudo iptables --flush
sudo iptables -tnat --flush
sudo systemctl start kubelet
sudo systemctl start docker
```





**参照文献：**
- [Debugging networking issues with multi-node Kubernetes on VirtualBox](https://www.jeffgeerling.com/blog/2019/debugging-networking-issues-multi-node-kubernetes-on-virtualbox)
- [调试 DNS 问题](https://kubernetes.io/zh/docs/tasks/administer-cluster/dns-debugging-resolution/)
- [How to specify Internal-IP for kubernetes worker node](https://medium.com/@kanrangsan/how-to-specify-internal-ip-for-kubernetes-worker-node-24790b2884fd)
- [linux系统dig和nslookup的安装
](https://blog.csdn.net/bjbs_270/article/details/7003088)
- [coredns-crashes-with-error-failed-to-list-v1-service](https://stackoverflow.com/questions/61315967/coredns-crashes-with-error-failed-to-list-v1-service-get-https-10-96-0-144)
- [Kubernetes on (vanilla) Raspbian Lite](https://github.com/teamserverless/k8s-on-raspbian/blob/master/GUIDE.md)