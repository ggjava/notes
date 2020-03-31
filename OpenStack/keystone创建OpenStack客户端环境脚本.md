
我们可以使用环境变量和命令选项的组合通过``openstack``客户端与身份认证服务交互。为了提升客户端操作的效率，OpenStack支持简单的客户端环境变量脚本即OpenRC 文件。这些脚本通常包含客户端所有常见的选项，当然也支持独特的选项。更多信息，请参考[OpenStack End User Guide](http://docs.openstack.org/user-guide/common/cli_set_environment_variables_using_openstack_rc.html)。

### 创建脚本

创建 admin 和 ``demo``项目和用户创建客户端环境变量脚本。本指南的接下来的部分会引用这些脚本，为客户端操作加载合适的的凭证。

1. 编辑文件 admin-openrc 并添加如下内容：

```
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_AUTH_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

将 ADMIN_PASS 替换为你在认证服务中为 admin 用户选择的密码。

2. 编辑文件 demo-openrc 并添加如下内容：

```
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=DEMO_PASS
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

将 DEMO_PASS 替换为你在认证服务中为 demo 用户选择的密码。

### 使用脚本

使用特定租户和用户运行客户端，你可以在运行之前简单地加载相关客户端脚本。例如：

1. 加载``admin-openrc``文件来身份认证服务的环境变量位置和``admin``项目和用户证书：

```
$ . admin-openrc
```

2. 请求认证令牌:

```
$ openstack token issue
+------------+-----------------------------------------------------------------+
| Field      | Value                                                           |
+------------+-----------------------------------------------------------------+
| expires    | 2016-02-12T20:44:35.659723Z                                     |
| id         | gAAAAABWvjYj-Zjfg8WXFaQnUd1DMYTBVrKw4h3fIagi5NoEmh21U72SrRv2trl |
|            | JWFYhLi2_uPR31Igf6A8mH2Rw9kv_bxNo1jbLNPLGzW_u5FC7InFqx0yYtTwa1e |
|            | eq2b0f6-18KZyQhs7F3teAta143kJEWuNEYET-y7u29y0be1_64KYkM7E       |
| project_id | 343d245e850143a096806dfaefa9afdc                                |
| user_id    | ac3377633149401296f6c0d92d79dc16                                |
+------------+-----------------------------------------------------------------+
```