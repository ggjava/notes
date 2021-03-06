OpenStack Networking（neutron），允许创建、插入接口设备，这些设备由其他的OpenStack服务管理。插件式的实现可以容纳不同的网络设备和软件，为OpenStack架构与部署提供了灵活性。

它包含下列组件：

* neutron-server

接收和路由API请求到合适的OpenStack网络插件，以达到预想的目的。

* OpenStack网络插件和代理

插拔端口，创建网络和子网，以及提供IP地址，这些插件和代理依赖于供应商和技术而不同，OpenStack网络基于插件和代理为Cisco 虚拟和物理交换机、NEC OpenFlow产品，Open vSwitch,Linux bridging以及VMware NSX 产品穿线搭桥。

常见的代理L3(3层)，DHCP(动态主机IP地址)，以及插件代理。

* 消息队列

大多数的OpenStack Networking安装都会用到，用于在neutron-server和各种各样的代理进程间路由信息。也为某些特定的插件扮演数据库的角色，以存储网络状态

OpenStack网络主要和OpenStack计算交互，以提供网络连接到它的实例。

### 网络概念

OpenStack网络（neutron）管理OpenStack环境中所有虚拟网络基础设施（VNI），物理网络基础设施（PNI）的接入层。OpenStack网络允许租户创建包括像 firewall， :term:`load balancer`和 :term:`virtual private network (VPN)`等这样的高级虚拟网络拓扑。

网络服务提供网络，子网以及路由这些对象的抽象概念。每个抽象概念都有自己的功能，可以模拟对应的物理设备：网络包括子网，路由在不同的子网和网络间进行路由转发。

对于任意一个给定的网络都必须包含至少一个外部网络。不像其他的网络那样，外部网络不仅仅是一个定义的虚拟网络。相反，它代表了一种OpenStack安装之外的能从物理的，外部的网络访问的视图。外部网络上的IP地址可供外部网络上的任意的物理设备所访问

外部网络之外，任何 Networking 设置拥有一个或多个内部网络。这些软件定义的网络直接连接到虚拟机。仅仅在给定网络上的虚拟机，或那些在通过接口连接到相近路由的子网上的虚拟机，能直接访问连接到那个网络上的虚拟机。

如果外部网络想要访问实例或者相反实例想要访问外部网络，那么网络之间的路由就是必要的了。每一个路由都配有一个网关用于连接到外部网络，以及一个或多个连接到内部网络的接口。就像一个物理路由一样，子网可以访问同一个路由上其他子网中的机器，并且机器也可以访问路由的网关访问外部网络。

另外，你可以将外部网络的IP地址分配给内部网络的端口。不管什么时候一旦有连接连接到子网，那个连接被称作端口。你可以给实例的端口分配外部网络的IP地址。通过这种方式，外部网络上的实体可以访问实例.

网络服务同样支持安全组。安全组允许管理员在安全组中定义防火墙规则。一个实例可以属于一个或多个安全组，网络为这个实例配置这些安全组中的规则，阻止或者开启端口，端口范围或者通信类型。

每一个Networking使用的插件都有其自有的概念。虽然对操作VNI和OpenStack环境不是至关重要的，但理解这些概念能帮助你设置Networking。所有的Networking安装使用了一个核心插件和一个安全组插件(或仅是空操作安全组插件)。另外，防火墙即服务(FWaaS)和负载均衡即服务(LBaaS)插件是可用的。