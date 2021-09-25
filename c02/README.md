# 必备概念：Kubernetes 的核心架构原则

- Kubernetes 解决了哪些业务痛点
- Kubernetes 的对象管理：
    - 命令式系统
    - 声明式系统

- 理解 kubernetes 的核心组件：
              
  master节点：
    - apiserver
    - controller manager
    - schedule
    - etcd
                      
  node节点：
    - kubelet
    - proxy
    - Container Runtime


## 一：Kubernetes 解决了什么业务痛点？

kubernetes 说白了就是自动化管理 Docker 容器组的一种集群工具。
如果已经使用镜像打包应用，可以方便的进行部署上线动作，不用去考虑不同环境的差异。          
但是真实的业务系统，通常还需要考虑到容器的访问，弹性伸缩，容灾，机器升级等因素。         
类似上述的问题，就是所谓的容器编排和调度。            

如以下一些常见场景:        
1. 告警系统检测到流量持续增大时，传统解决是再部署几台服务器，通过负载均衡均摊流量。Kubernetes 可以根据不同维度（如pod中容器cpu，内存使用率，请求访问数等）去动态扩缩容。
2. 应用自检和修复能力，当有实例挂掉时，服务发现自动剔除挂掉的实例，重启成功后，又自动加入到服务发现当中。

## 二：Kubernetes 的对象管理

### 理解 k8s 对象
在系统中，创建对象，就是告诉集群我想要我的应用怎么样运行。kubernetes 对象是持久化的一个实体，这些实体通常有如下描述信息：         
1. 哪些节点上有哪些容器化的应用在运行
1. 应用运行时表现的策略，如重启策略、升级策略，以及容错策略
1. 应用可以使用哪些资源，如cpu、内存、volume


如对象创建的过程（假设集群有3个节点，node1，node2，node3）,用大白话可以描述成：       
1. 将nginx镜像启动3个pod实例，并且只部署在node1和node2。
1. 将nginx部署10份，并且使用滚动升级策略，限制最大资源为0.1核cpu，100Mi内存。
1. 将java程序镜像部署30个pod实例，并且每15秒自检一次8080端口有没有被监听，每个pod分配2核cpu，4Gi内存。

当然，大白话说出来是给我们听的，机器和程序并不懂。
这时候就需要一种对象管理的手段。

###  k8s 对象管理
kubernetes 中的对象管理方式，可以根据对象配置信息的位置不同分为两大类。

- 命令式：对象的参数通过命令直接指定
- 配置式：对象的参数通过 YAML 配置文件指定

其中，对于配置式对象管理的方式，根据在执行 kubectl 命令时是否指定具体操作，又分为两类：

- 命令式对象配置：命令中指定具体操作，对象配置信息在配置文件中指定
- 声明式对象配置：命令中不指定具体操作，通过 kubectl 自动检测对象并进行创建、更新和删除操作。

#### 命令式（常见）
操作以及对象定义都可以直接在 kubectl 命令行中通过参数指定，这时候采用的对象管理方式就是命令式。例如：

```shell
# 使用 nginx 镜像运行一个pod
kubectl run nginx --image nginx
# 使用 nginx 镜像创建一个deployment 对象
kubectl create deployment nginx --image nginx
```

#### 命令式对象配置
命令行煮给你指定了操作和定义对象的文件

```shell
# 创建 nginx.yaml 文件中指定的对象
kubectl create -f nginx.yaml
# 删除 nginx.yaml 文件中指定的对象
kubectl delete -f nginx.yaml
```

#### 声明式对象配置（只有一个命令apply）
声明式对象配置保留其它对象所做的更改，即时这些更改为记录到对象的配置文件中。

```powershell
#可以是创建、修改
kubectl apply -f  nginx.yaml
```



