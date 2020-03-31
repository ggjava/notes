计算节点处理实例的连接和 安全组 。

## 安装组件

```
# yum install openstack-neutron-linuxbridge ebtables ipset
```

## 配置通用组件

Networking 通用组件的配置包括认证机制、消息队列和插件。

> 默认配置文件在各发行版本中可能不同。你可能需要添加这些部分，选项而不是修改已经存在的部分和选项。另外，在配置片段中的省略号(...)表示默认的配置选项你应该保留。

* 编辑``/etc/neutron/neutron.conf`` 文件并完成如下操作：

    * 在``[database]`` 部分，注释所有``connection`` 项，因为计算节点不直接访问数据库。

    * 在 “[DEFAULT]” 和 “[oslo_messaging_rabbit]”部分，配置 “RabbitMQ” 消息队列的连接：

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

    用你在RabbitMQ中为``openstack``选择的密码替换 “RABBIT_PASS”。

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
    username = neutron
    password = NEUTRON_PASS
    ```

    将 NEUTRON_PASS 替换为你在认证服务中为 neutron 用户选择的密码。

    > 在 [keystone_authtoken] 中注释或者删除其他选项。

    * 在 [oslo_concurrency] 部分，配置锁路径：

    ```
    [oslo_concurrency]
    ...
    lock_path = /var/lib/neutron/tmp
    ```

## 配置网络选项

选择与您之前在控制节点上选择的相同的网络选项。之后，回到这里并进行下一步：为计算节点配置网络服务。

* 网络选项1：公共网络
* 网络选项2：私有网络

## 为计算节点配置网络服务

* 编辑``/etc/nova/nova.conf``文件并完成下面的操作：

    * 在``[neutron]`` 部分，配置访问参数：

    ```
    [neutron]
    ...
    url = http://controller:9696
    auth_url = http://controller:35357
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    region_name = RegionOne
    project_name = service
    username = neutron
    password = NEUTRON_PASS
    ```

    将 NEUTRON_PASS 替换为你在认证服务中为 neutron 用户选择的密码。

## 完成安装

重启计算服务：

```
# systemctl restart openstack-nova-compute.service
```

启动Linuxbridge代理并配置它开机自启动：

```
# systemctl enable neutron-linuxbridge-agent.service
# systemctl start neutron-linuxbridge-agent.service
```
