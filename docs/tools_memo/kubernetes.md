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
* Deployment: 在 1.2 版本引入的新概念。Deployment 内部使用 Replica Set 实现，可以当做是Replica Set的升级版。Deployment 可以知道当前的部署进度、记录修改历史和回滚。
* Horizontal Pod Autoscaler: 即横向 Pod 自动扩展。通过监控 Pod 的负载变化来判断是否需要调整 Pod 的数量。HPA 以以下两种方式做作为 Pod 的负载指标：
    * CPUUtilizationPercentage: 是所有目标 Pod 的自身 CPU 的平均利用率。
    * 应用程序自定义指标。
* StatefulSet: 可以看作是 Deployment/Rc 的变种，为了适应有状态的服务它有如下特性：
    * StatefulSet 里的每个 Pod 都有稳定、唯一的网络标识。如果 StatefulSet 的名称是 kafka 那么第一个 Pod 叫 kafka-0，第二个叫 kafka-1，以此类推。
    * StatefulSet 控制的 Pod 副本的启动顺序是受控的。操作第 n 个 Pod 时，前 n-1 个 Pod 必定是准备好的。
    * StatefulSet 采用稳定的持久话存储卷，通过 PV 或 PVC 实现([PV、PVC、StorageClass讲解](https://www.cnblogs.com/rexcheny/p/10925464.html))，删除 Pod 时默认不会删除相关存储卷。
    * StatefulSet 要与 Headless Service 配合使用。Headless Service 没有 Cluster IP，如果解析 Headless Sercvice 的域名会得到 Service 的 Endpoint 列表。
* Service: 就是一这个真正的服务。在多端口的情况下要求给每个 Endpoint 定义一个名称来区分。
* Job: 并行或串行处理一些列任务后就结束的资源。
* Volume: 是 Pod 中能够被多个容器访问的共享目录。它的生命周期与 Pod 相同，和容器无关。Kubernetes 支持多种不同的 Volume 类型：
    * emptyDir: 初始内容为空，由 Kubernetes 自动分配的一个目录，在 Pod 分配到 Node 时创建的。当 Pod 从 Node 上移除时 emptyDir 的数据就会永久被删除。
    * hostPath: 挂载在宿主机上的文件或目录，可以永久保存。要注意以下两点：
        1. 在不同的Node上具有相同配置的Pod,可能会因为宿主机上的目录和文件不同而导致对 Volume 上目录和文件的访问结果不一致
        2. 如果使用了资源配额管理,则Kubernetes无法将hostPath在宿主机上使用的资源纳入管理。
    * gcePersistentDisk: 表示这种类型的 Volume 使用的是谷歌公有云提供的永久磁盘（Persostent Disk, PD）。PD 上的内容会永远保存，Pod 被删除时 PD 只会v被卸载。
    * awsElasticBlockStore： 与 GCE 类似，使用的是亚马逊公有云。
    * NFS 使用 NFS 网络文件系统提供共享目录。
    * 其他 Volume:
        * iscsi
        * flocker
        * glusterfs
        * rbd
        * gitRepo
        * secret
* Persistent Volume: PV 可以被理解成Kubernetes集群中的某个网络存储对应的一块存储,它与Volume类似,但有以下区别:
    * PV只能是网络存储,不属于任何Node,但可以在每个Node上访问。
    * PV并不是被定义在Pod上的,而是独立于Pod之外定义的。
    * [目前支持的持久卷类型](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes)。
* Namespace: 可以将资源分配到不同的 Namespaces，从而形成逻辑上的资源隔离。
* Annotation: 和 Label 类似，也是 key/value 的形式。Annotation 用于定义各类附加信息。Kubernetes 的模块自身会通过Annotation标记资源对象的一些特殊信息。
* ConfigMap: 是一个以 key/value 形式保存在 etcd 中的 Map 对象。ConfigMap 通过 API 给相关组件 CRUD。
