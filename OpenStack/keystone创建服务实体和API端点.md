身份认证服务提供服务的目录和他们的位置。每个你添加到OpenStack环境中的服务在目录中需要一个 service 实体和一些 API endpoints 。

### 先决条件

默认情况下，身份认证服务数据库不包含支持传统认证和目录服务的信息。你必须使用:doc:keystone-install 章节中为身份认证服务创建的临时身份验证令牌用来初始化的服务实体和API端点。

你必须使用``–os-token``参数将认证令牌的值传递给:command:openstack 命令。类似的，你必须使用``–os-url`` 参数将身份认证服务的 URL传递给 openstack 命令或者设置OS_URL环境变量。本指南使用环境变量以缩短命令行的长度。

> 警告  
> 因为安全的原因，，除非做必须的认证服务初始化，不要长时间使用临时认证令牌。

1. 配置认证令牌：

```bash
$ export OS_TOKEN=ADMIN_TOKEN
```

将``ADMIN_TOKEN``替换为你在 :doc:`keystone-install`章节中生成的认证令牌。例如：

```bash
$ export OS_TOKEN=294a4c8a8a475f9b9836
```

2. 配置端点URL：

```bash
$ export OS_URL=http://controller:35357/v3
```

3. 配置认证 API 版本：

```bash
$ export OS_IDENTITY_API_VERSION=3
```

### 创建服务实体和API端点

1. 在你的Openstack环境中，认证服务管理服务目录。服务使用这个目录来决定您的环境中可用的服务。

创建服务实体和身份认证服务：

```
$ openstack service create \
  --name keystone --description "OpenStack Identity" identity
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Identity               |
| enabled     | True                             |
| id          | 4ddaae90388b4ebc9d252ec2252d8d10 |
| name        | keystone                         |
| type        | identity                         |
+-------------+----------------------------------+
```

> 注解  
> OpenStack 是动态生成 ID 的，因此您看到的输出会与示例中的命令行输出不相同。

2. 身份认证服务管理了一个与您环境相关的 API 端点的目录。服务使用这个目录来决定如何与您环境中的其他服务进行通信。

OpenStack使用三个API端点变种代表每种服务：admin，internal和public。默认情况下，管理API端点允许修改用户和租户而公共和内部APIs不允许这些操作。在生产环境中，处于安全原因，变种为了服务不同类型的用户可能驻留在单独的网络上。对实例而言，公共API网络为了让顾客管理他们自己的云在互联网上是可见的。管理API网络在管理云基础设施的组织中操作也是有所限制的。内部API网络可能会被限制在包含OpenStack服务的主机上。此外，OpenStack支持可伸缩性的多区域。为了简单起见，本指南为所有端点变种和默认``RegionOne``区域都使用管理网络。

创建认证服务的 API 端点：

```
$ openstack endpoint create --region RegionOne \
  identity public http://controller:5000/v3
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 30fff543e7dc4b7d9a0fb13791b78bf4 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 8c8c0927262a45ad9066cfe70d46892c |
| service_name | keystone                         |
| service_type | identity                         |
| url          | http://controller:5000/v3        |
+--------------+----------------------------------+

$ openstack endpoint create --region RegionOne \
  identity internal http://controller:5000/v3
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 57cfa543e7dc4b712c0ab137911bc4fe |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 6f8de927262ac12f6066cfe70d99ac51 |
| service_name | keystone                         |
| service_type | identity                         |
| url          | http://controller:5000/v3        |
+--------------+----------------------------------+

$ openstack endpoint create --region RegionOne \
  identity admin http://controller:35357/v3
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 78c3dfa3e7dc44c98ab1b1379122ecb1 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 34ab3d27262ac449cba6cfe704dbc11f |
| service_name | keystone                         |
| service_type | identity                         |
| url          | http://controller:35357/v3       |
+--------------+----------------------------------+
```

> 注解  
>  每个添加到OpenStack环境中的服务要求一个或多个服务实体和三个认证服务中的API 端点变种。

