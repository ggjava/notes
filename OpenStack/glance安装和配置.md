这个部分描述如何在控制节点上安装和配置镜像服务，即 glance。简单来说，这个配置将镜像保存在本地文件系统中。

## 先决条件

安装和配置镜像服务之前，你必须创建创建一个数据库、服务凭证和API端点。

1. 完成下面的步骤以创建数据库：

    * 用数据库连接客户端以 root 用户连接到数据库服务器：

    ```sql
    $ mysql -u root -p
    ```

    * 创建 glance 数据库：

    ```sql
    CREATE DATABASE glance;
    ```

    * 对``glance``数据库授予恰当的权限：

    ```sql
    GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
    IDENTIFIED BY 'GLANCE_DBPASS';
    GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
    IDENTIFIED BY 'GLANCE_DBPASS';
    ```

    用一个合适的密码替换 GLANCE_DBPASS。

    * 退出数据库客户端。

2. 获得 admin 凭证来获取只有管理员能执行的命令的访问权限：

```
$ . admin-openrc
```

3. 要创建服务证书，完成这些步骤：

    * 创建 glance 用户：

    ```
    $ openstack user create --domain default --password-prompt glance
    User Password:
    Repeat User Password:
    +-----------+----------------------------------+
    | Field     | Value                            |
    +-----------+----------------------------------+
    | domain_id | e0353a670a9e496da891347c589539e9 |
    | enabled   | True                             |
    | id        | e38230eeff474607805b596c91fa15d9 |
    | name      | glance                           |
    +-----------+----------------------------------+
    ```
    
    * 添加 admin 角色到 glance 用户和 service 项目上。

    ```
    $ openstack role add --project service --user glance admin
    ```

    * 创建``glance``服务实体：

    ```
    $ openstack service create --name glance \
    --description "OpenStack Image" image
    +-------------+----------------------------------+
    | Field       | Value                            |
    +-------------+----------------------------------+
    | description | OpenStack Image                  |
    | enabled     | True                             |
    | id          | 8c2c7f1b9b5049ea9e63757b5533e6d2 |
    | name        | glance                           |
    | type        | image                            |
    +-------------+----------------------------------+
    ```

4. 创建镜像服务的 API 端点：

```
$ openstack endpoint create --region RegionOne \
  image public http://controller:9292
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 340be3625e9b4239a6415d034e98aace |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 8c2c7f1b9b5049ea9e63757b5533e6d2 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+

$ openstack endpoint create --region RegionOne \
  image internal http://controller:9292
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | a6e4b153c2ae4c919eccfdbb7dceb5d2 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 8c2c7f1b9b5049ea9e63757b5533e6d2 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+

$ openstack endpoint create --region RegionOne \
  image admin http://controller:9292
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 0c37ed58103f4300a84ff125a539032d |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 8c2c7f1b9b5049ea9e63757b5533e6d2 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+
```

## 安全并配置组件

> 默认配置文件在各发行版本中可能不同。你可能需要添加这些部分，选项而不是修改已经存在的部分和选项。另外，在配置片段中的省略号(...)表示默认的配置选项你应该保留。

1. 安装软件包：

```bash
# yum install openstack-glance
```

2. 编辑文件 /etc/glance/glance-api.conf 并完成如下动作：

    * 在 [database] 部分，配置数据库访问：

    ```
    [database]
    ...
    connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance
    ```

    将``GLANCE_DBPASS`` 替换为你为镜像服务选择的密码。

    * 在 [keystone_authtoken] 和 [paste_deploy] 部分，配置认证服务访问：

    ```
    [keystone_authtoken]
    ...
    auth_uri = http://controller:5000
    auth_url = http://controller:35357
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = glance
    password = GLANCE_PASS

    [paste_deploy]
    ...
    flavor = keystone
    ```
    将 GLANCE_PASS 替换为你为认证服务中你为 glance 用户选择的密码。

    > 在 [keystone_authtoken] 中注释或者删除其他选项。

    * 在 [glance_store] 部分，配置本地文件系统存储和镜像文件位置：

    ```
    [glance_store]
    ...
    stores = file,http
    default_store = file
    filesystem_store_datadir = /var/lib/glance/images/
    ```

3. 编辑文件 ``/etc/glance/glance-registry.conf``并完成如下动作：

    * 在 [database] 部分，配置数据库访问：

    ```
    [database]
    ...
    connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance
    ```

    将``GLANCE_DBPASS`` 替换为你为镜像服务选择的密码。

    * 在 [keystone_authtoken] 和 [paste_deploy] 部分，配置认证服务访问：

    ```
    [keystone_authtoken]
    ...
    auth_uri = http://controller:5000
    auth_url = http://controller:35357
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = glance
    password = GLANCE_PASS

    [paste_deploy]
    ...
    flavor = keystone
    ```

    将 GLANCE_PASS 替换为你为认证服务中你为 glance 用户选择的密码。

    > 在 [keystone_authtoken] 中注释或者删除其他选项。

4. 写入镜像服务数据库：

```bash
# su -s /bin/sh -c "glance-manage db_sync" glance
```

> 忽略输出中任何不推荐使用的信息。

## 完成安装

* 启动镜像服务、配置他们随机启动：

```bash
# systemctl enable openstack-glance-api.service \
  openstack-glance-registry.service
# systemctl start openstack-glance-api.service \
  openstack-glance-registry.service
```

