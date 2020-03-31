使用OpenStack计算服务来托管和管理云计算系统。OpenStack计算服务是基础设施即服务(IaaS)系统的主要部分，模块主要由Python实现。

OpenStack计算组件请求OpenStack Identity服务进行认证；请求OpenStack Image服务提供磁盘镜像；为OpenStack dashboard提供用户与管理员接口。磁盘镜像访问限制在项目与用户上；配额以每个项目进行设定（例如，每个项目下可以创建多少实例）。OpenStack组件可以在标准硬件上水平大规模扩展，并且下载磁盘镜像启动虚拟机实例。

OpenStack计算服务由下列组件所构成：

### nova-api 服务

接收和响应来自最终用户的计算API请求。此服务支持OpenStack计算服务API，Amazon EC2 API，以及特殊的管理API用于赋予用户做一些管理的操作。它会强制实施一些规则，发起多数的编排活动，例如运行一个实例。

### nova-api-metadata 服务

接受来自虚拟机发送的元数据请求。``nova-api-metadata``服务一般在安装``nova-network``服务的多主机模式下使用。更详细的信息，请参考OpenStack管理员手册中的链接[Metadata service](http://docs.openstack.org/admin-guide/compute-networking-nova.html#metadata-service)in the OpenStack Administrator Guide。

### ``nova-compute``服务

一个持续工作的守护进程，通过Hypervior的API来创建和销毁虚拟机实例。例如：

* XenServer/XCP 的 XenAPI
* KVM 或 QEMU 的 libvirt
* VMware 的 VMwareAPI

过程是蛮复杂的。最为基本的，守护进程同意了来自队列的动作请求，转换为一系列的系统命令如启动一个KVM实例，然后，到数据库中更新它的状态。

### ``nova-scheduler``服务

拿到一个来自队列请求虚拟机实例，然后决定那台计算服务器主机来运行它。

### ``nova-conductor``模块

媒介作用于``nova-compute``服务与数据库之间。它排除了由``nova-compute``服务对云数据库的直接访问。nova-conductor模块可以水平扩展。但是，不要将它部署在运行``nova-compute``服务的主机节点上。参考[Configuration Reference Guide](http://docs.openstack.org/mitaka/config-reference/compute/conductor.html)。

### ``nova-cert``模块

服务器守护进程向Nova Cert服务提供X509证书。用来为``euca-bundle-image``生成证书。仅仅是在EC2 API的请求中使用

### nova-network worker 守护进程

与``nova-compute``服务类似，从队列中接受网络任务，并且操作网络。执行任务例如创建桥接的接口或者改变IPtables的规则。

### nova-consoleauth 守护进程

授权控制台代理所提供的用户令牌。详情可查看``nova-novncproxy``和 nova-xvpvncproxy。该服务必须为控制台代理运行才可奏效。在集群配置中你可以运行二者中任一代理服务而非仅运行一个nova-consoleauth服务。更多关于nova-consoleauth的信息，请查看[About nova-consoleauth](http://docs.openstack.org/admin-guide/compute-remote-console-access.html#about-nova-consoleauth)。

### nova-novncproxy 守护进程

提供一个代理，用于访问正在运行的实例，通过VNC协议，支持基于浏览器的novnc客户端。

### nova-spicehtml5proxy 守护进程

提供一个代理，用于访问正在运行的实例，通过 SPICE 协议，支持基于浏览器的 HTML5 客户端。

### nova-xvpvncproxy 守护进程

提供一个代理，用于访问正在运行的实例，通过VNC协议，支持OpenStack特定的Java客户端。

### nova-cert 守护进程

X509 证书。

### ``nova``客户端

用于用户作为租户管理员或最终用户来提交命令。

### 队列
一个在守护进程间传递消息的中央集线器。常见实现有[RabbitMQ](http://www.rabbitmq.com), 以及如[Zero MQ](http://www.zeromq.org/)等AMQP消息队列。

### SQL数据库
存储构建时和运行时的状态，为云基础设施，包括有：

* 可用实例类型
* 使用中的实例
* 可用网络
* 项目

理论上，OpenStack计算可以支持任何和SQL-Alchemy所支持的后端数据库，通常使用SQLite3来做测试可开发工作，MySQL和PostgreSQL 作生产环境。