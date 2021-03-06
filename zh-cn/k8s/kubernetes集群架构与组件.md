# Kubernetes集群架构与组件

![kubernetes架构图](../../_media/components-of-kubernetes.svg)

**组件作用**

1. **Master节点组件**

| 组件名称                | 作用                                                                                                                                             |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| kube-apiserver          | Kubernetes API，集群的统一入口，各组件协调者，以RESTful API提供接口服务，所有对象资源的增删改查和监听操作都交给APIServer处理后再提交给Etcd存储。 |
| kube-controller-manager | 处理集群中常规后台任务，一个资源对应一个控制器，而 ControllerManager就是负责管理这些控制器的。                                                   |
| kube-scheduler          | 根据调度算法为新创建的Pod选择一个Node节点，可以任意部署, 可以部署在同一个节点上,也可以部署在不同的节点上。                                       |
| etcd                    | 分布式键值存储系统。用于保存集群状态数据，比如Pod、Service 等对象信息。                                                                          |

2. **Node节点组件**

| 组件名称       | 作用                                                                                                                                                                 |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| kubelet        | kubelet是Master在Node节点上的Agent，管理本机运行容器的生命周 期，比如创建容器、Pod挂载数据卷、下载secret、获取容器和节点状态等工作。kubelet将每个Pod转换成一组容器。 |
| kube-proxy     | 在Node节点上实现Pod网络代理，维护网络规则和四层负载均衡工作。                                                                                                        |
| docker或rocket | 容器引擎，运行容器。                                                                                                                                                 |