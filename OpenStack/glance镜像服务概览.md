镜像服务 (glance) 允许用户发现、注册和获取虚拟机镜像。它提供了一个 REST API，允许您查询虚拟机镜像的 metadata 并获取一个现存的镜像。您可以将虚拟机镜像存储到各种位置，从简单的文件系统到对象存储系统—-例如 OpenStack 对象存储, 并通过镜像服务使用。

> 重要  
> 简单来说，本指南描述了使用`file``作为后端配置镜像服务，能够上传并存储在一个托管镜像服务的控制节点目录中。默认情况下，这个目录是 /var/lib/glance/images/。  
> 继续进行之前，确认控制节点的该目录有至少几千兆字节的可用空间。  
> 关于后端要求的更多信息，参考 [配置参考](http://docs.openstack.org/mitaka/config-reference/image-service.html)。

OpenStack镜像服务是IaaS的核心服务，如同 :ref:`get_started_conceptual_architecture`所示。它接受磁盘镜像或服务器镜像API请求，和来自终端用户或OpenStack计算组件的元数据定义。它也支持包括OpenStack对象存储在内的多种类型仓库上的磁盘镜像或服务器镜像存储。

大量周期性进程运行于OpenStack镜像服务上以支持缓存。同步复制（Replication）服务保证集群中的一致性和可用性。其它周期性进程包括auditors, updaters, 和 reapers。

OpenStack镜像服务包括以下组件：

### glance-api

接收镜像API的调用，诸如镜像发现、恢复、存储。

### glance-registry

存储、处理和恢复镜像的元数据，元数据包括项诸如大小和类型。

> glance-registry是私有内部服务，用于服务OpenStack Image服务。不要向用户暴露该服务

### 数据库

存放镜像元数据，用户是可以依据个人喜好选择数据库的，多数的部署使用MySQL或SQLite。

### 镜像文件的存储仓库

支持多种类型的仓库，它们有普通文件系统、对象存储、RADOS块设备、HTTP、以及亚马逊S3。记住，其中一些仓库仅支持只读方式使用。

### 元数据定义服务

通用的API，是用于为厂商，管理员，服务，以及用户自定义元数据。这种元数据可用于不同的资源，例如镜像，工件，卷，配额以及集合。一个定义包括了新属性的键，描述，约束以及可以与之关联的资源的类型。