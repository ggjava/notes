这个部分将描述如何在控制节点上安装和配置 Compute 服务，即 nova。

## 先决条件

在安装和配置 Compute 服务前，你必须创建数据库服务的凭据以及 API endpoints。

1. 为了创建数据库，必须完成这些步骤：

    * 用数据库连接客户端以 root 用户连接到数据库服务器：

    ```sql
    $ mysql -u root -p
    ```

    * 创建 nova_api 和 nova 数据库：

    ```sql
    CREATE DATABASE nova_api;
    CREATE DATABASE nova;
    ```

    * 对数据库进行正确的授权：

    ```sql
    GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
    IDENTIFIED BY 'NOVA_DBPASS';
    GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \
    IDENTIFIED BY 'NOVA_DBPASS';
    GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
    IDENTIFIED BY 'NOVA_DBPASS';
    GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
    IDENTIFIED BY 'NOVA_DBPASS';
    ```

    用合适的密码代替 NOVA_DBPASS。

    * 退出数据库客户端。

2. 获得 admin 凭证来获取只有管理员能执行的命令的访问权限：

```
$ . admin-openrc
```

3. 创建服务证书，完成这些步骤：

    * 创建 nova 用户：

    ```
    $ openstack user create --domain default \
    --password-prompt nova
    User Password:
    Repeat User Password:
    +-----------+----------------------------------+
    | Field     | Value                            |
    +-----------+----------------------------------+
    | domain_id | e0353a670a9e496da891347c589539e9 |
    | enabled   | True                             |
    | id        | 8c46e4760902464b889293a74a0c90a8 |
    | name      | nova                             |
    +-----------+----------------------------------+
    ```

    * 给 nova 用户添加 admin 角色：

    ```
    $ openstack role add --project service --user nova admin
    ```

    * 创建 nova 服务实体：

    ```
    $ openstack service create --name nova \
    --description "OpenStack Compute" compute
    +-------------+----------------------------------+
    | Field       | Value                            |
    +-------------+----------------------------------+
    | description | OpenStack Compute                |
    | enabled     | True                             |
    | id          | 060d59eac51b4594815603d75a00aba2 |
    | name        | nova                             |
    | type        | compute                          |
    +-------------+----------------------------------+
    ```

4. 创建 Compute 服务 API 端点 ：

```
$ openstack endpoint create --region RegionOne \
compute public http://controller:8774/v2.1/%\(tenant_id\)s
+--------------+-------------------------------------------+
| Field        | Value                                     |
+--------------+-------------------------------------------+
| enabled      | True                                      |
| id           | 3c1caa473bfe4390a11e7177894bcc7b          |
| interface    | public                                    |
| region       | RegionOne                                 |
| region_id    | RegionOne                                 |
| service_id   | e702f6f497ed42e6a8ae3ba2e5871c78          |
| service_name | nova                                      |
| service_type | compute                                   |
| url          | http://controller:8774/v2.1/%(tenant_id)s |
+--------------+-------------------------------------------+

$ openstack endpoint create --region RegionOne \
compute internal http://controller:8774/v2.1/%\(tenant_id\)s
+--------------+-------------------------------------------+
| Field        | Value                                     |
+--------------+-------------------------------------------+
| enabled      | True                                      |
| id           | e3c918de680746a586eac1f2d9bc10ab          |
| interface    | internal                                  |
| region       | RegionOne                                 |
| region_id    | RegionOne                                 |
| service_id   | e702f6f497ed42e6a8ae3ba2e5871c78          |
| service_name | nova                                      |
| service_type | compute                                   |
| url          | http://controller:8774/v2.1/%(tenant_id)s |
+--------------+-------------------------------------------+

$ openstack endpoint create --region RegionOne \
compute admin http://controller:8774/v2.1/%\(tenant_id\)s
+--------------+-------------------------------------------+
| Field        | Value                                     |
+--------------+-------------------------------------------+
| enabled      | True                                      |
| id           | 38f7af91666a47cfb97b4dc790b94424          |
| interface    | admin                                     |
| region       | RegionOne                                 |
| region_id    | RegionOne                                 |
| service_id   | e702f6f497ed42e6a8ae3ba2e5871c78          |
| service_name | nova                                      |
| service_type | compute                                   |
| url          | http://controller:8774/v2.1/%(tenant_id)s |
+--------------+-------------------------------------------+
```

## 安全并配置组件

> 默认配置文件在各发行版本中可能不同。你可能需要添加这些部分，选项而不是修改已经存在的部分和选项。另外，在配置片段中的省略号(...)表示默认的配置选项你应该保留。

1. 安装软件包：

```
# yum install openstack-nova-api openstack-nova-conductor \
  openstack-nova-console openstack-nova-novncproxy \
  openstack-nova-scheduler
```

2. 编辑``/etc/nova/nova.conf``文件并完成下面的操作：

    * 在``[DEFAULT]``部分，只启用计算和元数据API：

    ```
    [DEFAULT]
    ...
    enabled_apis = osapi_compute,metadata
    ```

    * 在``[api_database]``和``[database]``部分，配置数据库的连接：

    ```
    [api_database]
    ...
    connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova_api

    [database]
    ...
    connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova
    ```

    用你为 Compute 数据库选择的密码来代替 NOVA_DBPASS。

    * 在 “[DEFAULT]” 和 “[oslo_messaging_rabbit]”部分，配置 “RabbitMQ” 消息队列访问：

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

    * 在 [DEFAULT 部分，配置``my_ip`` 来使用控制节点的管理接口的IP 地址。

    ```
    [DEFAULT]
    ...
    my_ip = 10.0.0.11
    ```

    * 在 [DEFAULT] 部分，使能 Networking 服务：

    ```
    [DEFAULT]
    ...
    use_neutron = True
    firewall_driver = nova.virt.firewall.NoopFirewallDriver
    ```

    > 默认情况下，计算服务使用内置的防火墙服务。由于网络服务包含了防火墙服务，你必须使用``nova.virt.firewall.NoopFirewallDriver``防火墙服务来禁用掉计算服务内置的防火墙服务

    * 在``[vnc]``部分，配置VNC代理使用控制节点的管理接口IP地址 ：

    ```
    [vnc]
    ...
    vncserver_listen = $my_ip
    vncserver_proxyclient_address = $my_ip
    ```

    * 在 [glance] 区域，配置镜像服务 API 的位置：

    ```
    [glance]
    ...
    api_servers = http://controller:9292
    ```

    * 在 [oslo_concurrency] 部分，配置锁路径：

    ```
    [oslo_concurrency]
    ...
    lock_path = /var/lib/nova/tmp
    ```

3. 同步Compute 数据库：

```
# su -s /bin/sh -c "nova-manage api_db sync" nova
# su -s /bin/sh -c "nova-manage db sync" nova
```

## 完成安装

启动 Compute 服务并将其设置为随系统启动：

```
# systemctl enable openstack-nova-api.service \
  openstack-nova-consoleauth.service openstack-nova-scheduler.service \
  openstack-nova-conductor.service openstack-nova-novncproxy.service
# systemctl start openstack-nova-api.service \
  openstack-nova-consoleauth.service openstack-nova-scheduler.service \
  openstack-nova-conductor.service openstack-nova-novncproxy.service
```