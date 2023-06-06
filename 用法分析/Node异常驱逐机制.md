## Node异常，容器的驱逐机制由kube-controller-manager实现

kube-controller-manager实现的eviction
kube-controller-manager主要由多个控制器构成，而eviction的功能主要由node controller这个控制器实现。该Eviction会周期性检查所有节点状态，当节点处于NotReady状态超过一段时间后，驱逐该节点上所有pod。

kube-controller-manager提供了以下启动参数控制eviction：

- node-monitor-grace-period duration 默认40s, 多久node没有响应认为node为unhealthy
- pod-eviction-timeout：即当节点宕机该时间间隔后，开始eviction机制，驱赶宕机节点上的Pod，默认为5min。
- node-eviction-rate：驱赶速率，即驱赶Node的速率，由令牌桶流控算法实现，默认为0.1，即每秒驱赶0.1个节点，注意这里不是驱赶Pod的速率，而是驱赶节点的速率。相当于每隔10s，清空一个节点。
- secondary-node-eviction-rate：二级驱赶速率，当集群中宕机节点过多时，相应的驱赶速率也降低，默认为0.01。
- unhealthy-zone-threshold：不健康zone阈值，会影响什么时候开启二级驱赶速率，默认为0.55，即当该zone中节点宕机数目超过55%，而认为该zone不健康。
- large-cluster-size-threshold：大集群阈值，默认为50。当该zone的节点多于该阈值时，则认为该zone是一个大集群。大集群节点宕机数目超过55%时，则将驱赶速率降为0.01，假如是小集群，则将速率直接降为0。


默认配置情况下，驱逐的机制结论：
1. 节点异常40s后（未上报心跳）标识为NotReady，5分钟后开始驱逐Pod
2. 多节点异常的情况下，每10s清空一个节点。大集群（节点数>=50个）中大量节点（超过55%节点）异常，驱逐速率降低，每100s清空一个节点；小集群（节点数<50），中大量节点（超过55%节点）异常，驱逐速率降为0，不再驱逐。



### 问题：如果Kubelet失联，节点上的容器无法Terminating无法驱逐，新的容器能产生么？（漂移能成功么?）

- 对于Deployment，容器Terminating后，会有相应的副本马上产生，如下例，两个副本是Terminating状态。新的Pod已产生。

```
NAME                      READY   STATUS        RESTARTS   AGE     IP        
deploy-7fd668d948-86hpp   1/1     Terminating   0          3d19h   8.0.2.216 
deploy-7fd668d948-cpr4c   0/1     Pending       0          3m23s   <none>    
deploy-7fd668d948-tk4ht   0/1     Pending       0          3m23s   <none>    
deploy-7fd668d948-wwl9n   1/1     Terminating   0          3d19h   8.0.2.214 
```

- 对于StatefulSet
- 

- 对于DaemonSet，不受影响，容器状态仍是Running状态


### 失联恢复
kubelet上报心跳后，Node状态即恢复。Terminating的容器真正进行删除。
