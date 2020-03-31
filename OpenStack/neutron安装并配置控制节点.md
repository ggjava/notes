## 先决条件

在你配置OpenStack网络（neutron）服务之前，你必须为其创建一个数据库，服务凭证和API端点。

1. 完成下面的步骤以创建数据库：

    * 用数据库连接客户端以 root 用户连接到数据库服务器：

    ```
    $ mysql -u root -p
    ```

    * 创建``neutron`` 数据库：

    ```
    CREATE DATABASE neutron;
    ```

    * 对``neutron`` 数据库授予合适的访问权限，使用合适的密码替换``NEUTRON_DBPASS``：

    ```
    GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' \
    IDENTIFIED BY 'NEUTRON_DBPASS';
    GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' \
    IDENTIFIED BY 'NEUTRON_DBPASS';
    ```

    * 退出数据库客户端。

2. 获得 admin 凭证来获取只有管理员能执行的命令的访问权限：

```
$ . admin-openrc
```

3. 要创建服务证书，完成这些步骤：


    * 创建``neutron``用户：

    ```
    $ openstack user create --domain default --password-prompt neutron
    User Password:
    Repeat User Password:
    +-----------+----------------------------------+
    | Field     | Value                            |
    +-----------+----------------------------------+
    | domain_id | e0353a670a9e496da891347c589539e9 |
    | enabled   | True                             |
    | id        | b20a6692f77b4258926881bf831eb683 |
    | name      | neutron                          |
    +-----------+----------------------------------+
    ```
    
    * 添加``admin`` 角色到``neutron`` 用户：

    ```
    $ openstack role add --project service --user neutron admin
    ```

    * 创建``neutron``服务实体：

    ```
    $ openstack service create --name neutron \
    --description "OpenStack Networking" network
    +-------------+----------------------------------+
    | Field       | Value                            |
    +-------------+----------------------------------+
    | description | OpenStack Networking             |
    | enabled     | True                             |
    | id          | f71529314dab4a4d8eca427e701d209e |
    | name        | neutron                          |
    | type        | network                          |
    +-------------+----------------------------------+
    ```

4. 创建网络服务API端点：

```
$ openstack endpoint create --region RegionOne \
  network public http://controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 85d80a6d02fc4b7683f611d7fc1493a3 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | f71529314dab4a4d8eca427e701d209e |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+

$ openstack endpoint create --region RegionOne \
  network internal http://controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 09753b537ac74422a68d2d791cf3714f |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | f71529314dab4a4d8eca427e701d209e |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+

$ openstack endpoint create --region RegionOne \
  network admin http://controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 1ee14289c9374dffb5db92a5c112fc4e |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | f71529314dab4a4d8eca427e701d209e |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+
```

## 配置网络选项

您可以部署网络服务使用选项1和选项2两种架构中的一种来部署网络服务。

选项1采用尽可能简单的架构进行部署，只支持实例连接到公有网络（外部网络）。没有私有网络（个人网络），路由器以及浮动IP地址。只有``admin``或者其他特权用户才可以管理公有网络

选项2在选项1的基础上多了layer－3服务，支持实例连接到私有网络。``demo``或者其他没有特权的用户可以管理自己的私有网络，包含连接公网和私网的路由器。另外，浮动IP地址可以让实例使用私有网络连接到外部网络，例如互联网

典型的私有网络一般使用覆盖网络。覆盖网络，例如VXLAN包含了额外的数据头，这些数据头增加了开销，减少了有效内容和用户数据的可用空间。在不了解虚拟网络架构的情况下，实例尝试用以太网 最大传输单元 (MTU) 1500字节发送数据包。网络服务会自动给实例提供正确的MTU的值通过DHCP的方式。但是，一些云镜像并没有使用DHCP或者忽视了DHCP MTU选项，要求使用元数据或者脚本来进行配置

> 选项2同样支持实例连接到公共网络

从以下的网络选项中选择一个来配置网络服务。之后，返回到这里，进行下一步:ref:neutron-controller-metadata-agent。

* 网络选项1：公共网络
* 网络选项2：私有网络

## 配置元数据代理

The :term:`metadata agent <Metadata agent>`负责提供配置信息，例如：访问实例的凭证

编辑``/etc/neutron/metadata_agent.ini``文件并完成以下操作：

在``[DEFAULT]`` 部分，配置元数据主机以及共享密码：

```
[DEFAULT]
...
nova_metadata_ip = controller
metadata_proxy_shared_secret = METADATA_SECRET
```

用你为元数据代理设置的密码替换 METADATA_SECRET。

## 为计算节点配置网络服务

编辑``/etc/nova/nova.conf``文件并完成以下操作：

在``[neutron]``部分，配置访问参数，启用元数据代理并设置密码：

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

service_metadata_proxy = True
metadata_proxy_shared_secret = METADATA_SECRET
```

将 NEUTRON_PASS 替换为你在认证服务中为 neutron 用户选择的密码。

使用你为元数据代理设置的密码替换``METADATA_SECRET``

## 完成安装

1. 网络服务初始化脚本需要一个超链接 /etc/neutron/plugin.ini``指向ML2插件配置文件/etc/neutron/plugins/ml2/ml2_conf.ini``。如果超链接不存在，使用下面的命令创建它：

```
# ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```

2. 同步数据库：

```
# su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```

> 数据库的同步发生在 Networking 之后，因为脚本需要完成服务器和插件的配置文件。

3. 重启计算API 服务：

```
# systemctl restart openstack-nova-api.service
```

4. 当系统启动时，启动 Networking 服务并配置它启动。

对于两种网络选项：

```
# systemctl enable neutron-server.service \
  neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
  neutron-metadata-agent.service
# systemctl start neutron-server.service \
  neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
  neutron-metadata-agent.service
```

对于网络选项2，同样启用layer－3服务并设置其随系统自启动

```
# systemctl enable neutron-l3-agent.service
# systemctl start neutron-l3-agent.service
```