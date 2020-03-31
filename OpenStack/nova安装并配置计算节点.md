这部分描述如何在计算节点上安装并配置计算服务。计算服务支持多种虚拟化方式 hypervisors to deploy instances or VMs. For simplicity, this configuration uses the QEMU hypervisor with the :term:`KVM <kernel-based VM (KVM)>`计算节点需支持对虚拟化的硬件加速。对于传统的硬件，本配置使用generic qumu的虚拟化方式。你可以根据这些说明进行细微的调整，或者使用额外的计算节点来横向扩展你的环境。

> 这部分假设你已经一步一步的按照之前的向导配置好了第一个计算节点。如果你想要配置额外的计算节点，像:ref:`example architectures <overview-example-architectures>`部分中第一个计算节点那样准备好。每个额外的计算节点都需要一个唯一的IP地址。

## 安装并配置组件

> 默认配置文件在各发行版本中可能不同。你可能需要添加这些部分，选项而不是修改已经存在的部分和选项。另外，在配置片段中的省略号(...)表示默认的配置选项你应该保留。

1. 安装软件包：

```
# yum install openstack-nova-compute
```

2. 编辑``/etc/nova/nova.conf``文件并完成下面的操作：

    * 在``[DEFAULT]`` 和 [oslo_messaging_rabbit]部分，配置``RabbitMQ``消息队列的连接：

    ```
    [DEFAULT]
    ...
    rpc_backend = rabbit

    [oslo_messaging_rabbit]
    ...
    rabbit_host = controller
    rabbit_userid = openstack
    rabbit_password = RABBIT_PASS
    ```

    用你在 “RabbitMQ” 中为 “openstack” 选择的密码替换 “RABBIT_PASS”。

    * 在 “[DEFAULT]” 和 “[keystone_authtoken]” 部分，配置认证服务访问：

    ```
    [DEFAULT]
    ...
    auth_strategy = keystone

    [keystone_authtoken]
    ...
    auth_uri = http://controller:5000
    auth_url = http://controller:35357
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = nova
    password = NOVA_PASS
    ```

    使用你在身份认证服务中设置的``nova`` 用户的密码替换``NOVA_PASS``。

    > 在 [keystone_authtoken] 中注释或者删除其他选项。

    * 在 [DEFAULT] 部分，配置 my_ip 选项：

    ```
    [DEFAULT]
    ...
    my_ip = MANAGEMENT_INTERFACE_IP_ADDRESS
    ```

    将其中的 MANAGEMENT_INTERFACE_IP_ADDRESS 替换为计算节点上的管理网络接口的IP 地址，例如 :ref:`example architecture <overview-example-architectures>`中所示的第一个节点 10.0.0.31 。

    * 在 [DEFAULT] 部分，使能 Networking 服务：

    ```
    [DEFAULT]
    ...
    use_neutron = True
    firewall_driver = nova.virt.firewall.NoopFirewallDriver
    ```

    > 缺省情况下，Compute 使用内置的防火墙服务。由于 Networking 包含了防火墙服务，所以你必须通过使用 nova.virt.firewall.NoopFirewallDriver 来去除 Compute 内置的防火墙服务。

    * 在``[vnc]``部分，启用并配置远程控制台访问：

    ```
    [vnc]
    ...
    enabled = True
    vncserver_listen = 0.0.0.0
    vncserver_proxyclient_address = $my_ip
    novncproxy_base_url = http://controller:6080/vnc_auto.html
    ```

    服务器组件监听所有的 IP 地址，而代理组件仅仅监听计算节点管理网络接口的 IP 地址。基本的 URL 指示您可以使用 web 浏览器访问位于该计算节点上实例的远程控制台的位置。

    > 如果你运行浏览器的主机无法解析``controller`` 主机名，你可以将 ``controller``替换为你控制节点管理网络的IP地址。

    * 在 [glance] 区域，配置镜像服务 API 的位置：

    ```
    [glance]
    ...
    api_servers = http://controller:9292
    在 [oslo_concurrency] 部分，配置锁路径：

    [oslo_concurrency]
    ...
    lock_path = /var/lib/nova/tmp
    ```

## 完成安装

1. 确定您的计算节点是否支持虚拟机的硬件加速。

```
$ egrep -c '(vmx|svm)' /proc/cpuinfo
```

如果这个命令返回了 one or greater 的值，那么你的计算节点支持硬件加速且不需要额外的配置。

如果这个命令返回了 zero 值，那么你的计算节点不支持硬件加速。你必须配置 libvirt 来使用 QEMU 去代替 KVM

在 /etc/nova/nova.conf 文件的 [libvirt] 区域做出如下的编辑：

```
[libvirt]
...
virt_type = qemu
```

2. 启动计算服务及其依赖，并将其配置为随系统自动启动：

```
# systemctl enable libvirtd.service openstack-nova-compute.service
# systemctl start libvirtd.service openstack-nova-compute.service
```
