# OpenYurt开箱测评｜一键让原生k8s集群具备边缘计算能力

自 OpenYurt 开源以来受到了开发者的关注，今天这篇文章将带大家快速上手 OpenYurt ，介绍如何使用 OpenYurt 提供的命令行管理工具 Yurtctl， 高效快速地部署 OpenYurt 集群。


## 1）OpenYurt介绍
OpenYurt 主打“云边一体化”概念，依托 Kubernetes 强大的容器应用编排能力，满足了云边一体化的应用分发、交付、和管控的诉求。相较于其他基于 Kubernetes 的边缘计算框架，OpenYurt 秉持着“最小修改”原则，通过在边缘节点安装 Yurthub 组件，和在云端部署 Yurt-controller-manager，保证了在对 Kubernetes 零侵入的情况下，提供管理边缘计算应用所需的相关能力。


OpenYurt 能帮用户解决在海量边、端资源上完成大规模应用交付、运维、管控的问题，并提供中心服务下沉通道，实现和边缘计算应用的无缝对接。在设计 OpenYurt 之初，我们就非常强调保持用户体验的一致性，不增加用户运维负担，让用户真正方便地 “Extending your native kubernetes to edge”。


## 2）Yurtctl：一键让原生k8s集群具备边缘计算能力

为了让原生 K8s 集群具备边缘计算能力，OpenYurt 以 addon 为载体，非侵入式给原生 K8s 增强了如下能力：
- 边缘自治能力（YurtHub：已开源），保证在弱网或者重启节点的情况下，部署在边缘节点上的应用也能正常运行；
- 云边协同能力（待开源），通过云边运维通道解决边缘的运维需求，同时提供云边协同能力；
- 单元化管理能力（待开源），为分散的边缘节点，边缘应用，应用间流量提供单元化闭环管理能力；

基于过往ACK@Edge的线上运维经验，我们开源了Yurtctl命令行工具，帮助实现了原生Kubernetes和OpenYurt之间的无缝转换以及对OpenYurt相关组件的高效运维。


### 2.1）Yurtctl的工作原理


Yurtctl是一个中心化的管控工具。在 OpenYurt云边一体的架构里，Yurtctl 将直接与 APIServer 进行交互。它借助原生 Kubernetes的Job workload对每个node进行运维操作。如图 1 所示，在执行转换（convert）操作时，Yurtctl 会通过 Job 将一个 servant Pod 部署到用户指定的边缘节点上。

servant Pod 里的容器执行的具体操作请参考：



由于 servant Pod 需要直接操作节点 root 用户的文件系统（例如将 yurthub 配置文件放置于 /etc/kubernetes/manifests 目录下），并且需要重置系统管理程序（kubelet.service），servant Pod 中的 container 将被赋予 privileged 权限，允许其与节点共享 pid namespace，并将借由 nsenter 命令进入节点主命名空间完成相关操作。当 servant Job 成功执行后，Job 会自动删除。如果失败，Job 则会被保留，方便运维人员排查错误原因。借由该机制，Yurtctl 还可对 Yurthub 进行更新或者删除。


#### 案例：一键转换OpenYurt集群
1）获取yurtctl
OpenYurt github 仓库包括了 yurtctl 的源码，下载 OpenYurt 仓库之后，即可通过编译获得 yurtctl，具体命令如下：

```
$ make build WHAT=cmd/yurtctl
hack/make-rules/build.sh cmd/yurtctl
Building cmd/yurtctl
```
编译成功之后，yurtctl 可执行文件就可以在 _output/bin/ 目录下找到。 

2）将Kubernetes转换为OpenYurt
yurtctl convert


3）将OpenYurt转换回Kubernetes
yurtctl revert



