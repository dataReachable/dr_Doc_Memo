# Kubernetes

## 相关概念

* Master: 集群的控制节点，所有的控制命令都发给它。主要有以下进程:
    * Kubernetes API Server(kube-apiserver): 提供 HTTP Rest 接口的进程，是资源操作的入口。
    * Kubernetes Controller Manager(kube-controller-manager): 所有资源对象的自动化控制中心。
    * Kubernetes Scheduler(kueb-scheduler): 负责资源调度(Pod 调度)的进程。
    * Master上通常还部署了 etcd 服务，kubernetes 上所有资源对象都保存在 etcd 中。
* Node: 除 Master 外的所有其他机器。主要有以下进程:
    * kubelet: 负责 Pod 上的容器创建、开启和停止等任务，并配合 Master 实现集群管理的基本功能。
    * kube-proxy: 实现 Kubernetes Service 的通信与负载均衡的组件。
    * Docker Engine: 容器引擎，负责本机的容器创建和管理。[Deprecate Dockershim #94624](https://github.com/kubernetes/kubernetes/pull/94624): k8s 官方将在未来弃用 Dockershim，现在官方推荐使用 containerd。这并非是不能使用 docker 进行开发，而是 k8s 更换了自身的运行时，对开发无太大影响。[官方 Blog 的说明](https://kubernetes.io/zh/blog/2020/12/02/dont-panic-kubernetes-and-docker/)。
* Pod: 是由一个 Pause 容器和多个业务容器组成。用 Pause 的状态代表整个 Pod 的状态，同时同一个 Pod 中的所有容器都共享 Pause 挂接的 Volume。Kuberbetes 给每个Pod 分配了唯一 IP，一个 Pod 中的多个容器共享同一个 Pod IP。集群内任意两个 Pod 都能通过 TCP/IP 直接通信。
    * 静态 Pod: 存放在 Node 中的某个文件中，且只能由此 Node 启动。
    * 普通 Pod: 创建后存放在 etcd 中，由 Kubernetes Master 调度到具体的 Node 上运行。
* Label： 一个键值对，可以在各类资源上添加任意个数。
* Replication Controller(RC): 控制 Pod 副本数量为设置的期望值，即在副本数量不足的时候根据 Template 创建 Pod 到预期的数量，过多时停掉 Pod。**删除 RC 时不会删除创建的 Pod**，可以将期望数量设置为 0。在 Kubernetes 1.2 中升级为一个新的概念 Replica Sets。Replica Sets 和 RC 的唯一区别就是 Replica Sets 支持基于集合的 Label selector。
* Deployment: 在 1.2 版本