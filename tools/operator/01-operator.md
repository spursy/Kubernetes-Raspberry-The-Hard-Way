## Operator 

#### 简介

- Operator 是一种封装、部署和管理Kubernetes的方法，是针对最复杂的有状态应用取封装运维能力的解决方案
- Operator 自 Kubernetes 1.7 开始支持自定义资源（Custom Resource Definitions - CRD）
- Operator 要求开发者自己实现一个专门针对自定义资源的控制器，在控制器中维护自定义资源的期望状态（Helm 和 Kustomize 最总仍然是依靠 Kubernetes 的内置资源，来跟 Kubernetes 打交道）




