# Kubernetes架构

	进行中

一个运行的Kubernetes集群包括节点上的agent（kubelet）和master组件（API、调度器等），它们建立在一个分布式存储方案之上。这张图显示了我们理想的最终状态，尽管我们还在做一些工作，比如让kubelet自身（实际上是我们的所有组件）在容器里运行，还有让调度器是100%可插拔的。

![架构图](../architecture.png?raw=true "架构总览")

## Kubernetes节点

当看一个系统的架构时，我们会把它分割成工作节点上运行的服务和组成群集级别的控制平面的服务。

Kubernetes节点负责运行其中有应用程序的容器，它们由master系统管理。

当然，每个节点都运行着Docker。Docker负责下载镜像和运行容器等细节。

### Kubelet

**Kubelet**管理[pods](../pods.md)和它们的容器、镜像、卷（volume）等等。

### Kube-Proxy

每个节点还运行了一个简单的网络代理和负载均衡器（参考[services FAQ](https://github.com/GoogleCloudPlatform/kubernetes/wiki/Services-FAQ)）。

Kube-Proxy反射出每个节点上Kubernetes API中定义的服务（service），并且它可以在一系列后端之间做简单的TCP和UDP流转发（轮询）。

服务终端现在是由DNS找到的，或者通过环境变量（[Docker-links-compatible](https://docs.docker.com/userguide/dockerlinks/)、Kubernetes {FOO}_SERVICE_HOST 、 {FOO}_SERVICE_PORT的都支持）。这些变量可以解出服务代理所管理的端口。

## Kubernetes控制面板

Kubernetes控制面板可分成一系列组件。现在它们都运行在一个_master_节点，但是为了支持高可用的集群，但是很快就会改变了。这些组件一起运行来提供一个集群的统一视图。

### etcd

所有持久化的master状态在一个etcd实例中储存。它提供了一个很好的可以可靠存放配置的方法。通过监视（watch）功能，变更可以非常迅速地通知到各个组件。

### Kubernetes API Server

Kubernetes的apiserver
The apiserver serves up the [Kubernetes API](../api.md). It is intended to be a CRUD-y server, with most/all business logic implemented in separate components or in plug-ins. It mainly processes REST operations, validates them, and updates the corresponding objects in `etcd` (and eventually other stores).

### Scheduler

The scheduler binds unscheduled pods to nodes via the `/binding` API. The scheduler is pluggable, and we expect to support multiple cluster schedulers and even user-provided schedulers in the future.

### Kubernetes Controller Manager Server

All other cluster-level functions are currently performed by the Controller Manager. For instance, `Endpoints` objects are created and updated by the endpoints controller, and nodes are discovered, managed, and monitored by the node controller. These could eventually be split into separate components to make them independently pluggable.

The [`replicationController`](../replication-controller.md) is a mechanism that is layered on top of the simple [`pod`](../pods.md) API. We eventually plan to port it to a generic plug-in mechanism, once one is implemented.


[![Analytics](https://kubernetes-site.appspot.com/UA-36037335-10/GitHub/docs/design/architecture.md?pixel)]()

[1]: architecture.png