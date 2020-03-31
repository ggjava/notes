身份认证服务为每个OpenStack服务提供认证服务。认证服务使用 T domains， projects (tenants)， :term:`users<user>`和 :term:`roles<role>`的组合。

1. 创建域``default``：

```
$ openstack domain create --description "Default Domain" default
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Default Domain                   |
| enabled     | True                             |
| id          | e0353a670a9e496da891347c589539e9 |
| name        | default                          |
+-------------+----------------------------------+
```

2. 在你的环境中，为进行管理操作，创建管理的项目、用户和角色：

    * 创建 admin 项目：

    ```
    $ openstack project create --domain default \
    --description "Admin Project" admin
    +-------------+----------------------------------+
    | Field       | Value                            |
    +-------------+----------------------------------+
    | description | Admin Project                    |
    | domain_id   | e0353a670a9e496da891347c589539e9 |
    | enabled     | True                             |
    | id          | 343d245e850143a096806dfaefa9afdc |
    | is_domain   | False                            |
    | name        | admin                            |
    | parent_id   | None                             |
    +-------------+----------------------------------+
    ```

    > OpenStack 是动态生成 ID 的，因此您看到的输出会与示例中的命令行输出不相同。

    * 创建 admin 用户：

    ```
    $ openstack user create --domain default \
    --password-prompt admin
    User Password:
    Repeat User Password:
    +-----------+----------------------------------+
    | Field     | Value                            |
    +-----------+----------------------------------+
    | domain_id | e0353a670a9e496da891347c589539e9 |
    | enabled   | True                             |
    | id        | ac3377633149401296f6c0d92d79dc16 |
    | name      | admin                            |
    +-----------+----------------------------------+
    ```

    * 创建 admin 角色：

    ```
    $ openstack role create admin
    +-----------+----------------------------------+
    | Field     | Value                            |
    +-----------+----------------------------------+
    | domain_id | None                             |
    | id        | cd2cb9a39e874ea69e5d4b896eb16128 |
    | name      | admin                            |
    +-----------+----------------------------------+
    ```

    * 添加``admin`` 角色到 admin 项目和用户上：

    ```
    $ openstack role add --project admin --user admin admin
    ```
    这个命令执行后没有输出。

    > 注解
    > 你创建的任何角色必须映射到每个OpenStack服务配置文件目录下的``policy.json`` 文件中。默认策略是给予“admin“角色大部分服务的管理访问权限。更多信息，参考[Operations Guide - Managing Projects and Users](http://docs.openstack.org/ops-guide/ops-projects-users.html)。

3. 本指南使用一个你添加到你的环境中每个服务包含独有用户的service 项目。创建``service``项目：

```
$ openstack project create --domain default \
  --description "Service Project" service
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Service Project                  |
| domain_id   | e0353a670a9e496da891347c589539e9 |
| enabled     | True                             |
| id          | 894cdfa366d34e9d835d3de01e752262 |
| is_domain   | False                            |
| name        | service                          |
| parent_id   | None                             |
+-------------+----------------------------------+
```

4. 常规（非管理）任务应该使用无特权的项目和用户。作为例子，本指南创建 demo 项目和用户。

    * 创建``demo`` 项目

    ```
    $ openstack project create --domain default \
    --description "Demo Project" demo
    +-------------+----------------------------------+
    | Field       | Value                            |
    +-------------+----------------------------------+
    | description | Demo Project                     |
    | domain_id   | e0353a670a9e496da891347c589539e9 |
    | enabled     | True                             |
    | id          | ed0b60bf607743088218b0a533d5943f |
    | is_domain   | False                            |
    | name        | demo                             |
    | parent_id   | None                             |
    +-------------+----------------------------------+
    ```

    当为这个项目创建额外用户时，不要重复这一步

    * 创建``demo`` 用户：

    ```
    $ openstack user create --domain default \
    --password-prompt demo
    User Password:
    Repeat User Password:
    +-----------+----------------------------------+
    | Field     | Value                            |
    +-----------+----------------------------------+
    | domain_id | e0353a670a9e496da891347c589539e9 |
    | enabled   | True                             |
    | id        | 58126687cbcc4888bfa9ab73a2256f27 |
    | name      | demo                             |
    +-----------+----------------------------------+
    ```

    * 创建 user 角色

    ```
    $ openstack role create user
    +-----------+----------------------------------+
    | Field     | Value                            |
    +-----------+----------------------------------+
    | domain_id | None                             |
    | id        | 997ce8d05fc143ac97d83fdfb5998552 |
    | name      | user                             |
    +-----------+----------------------------------+
    ```

    * 添加 user``角色到 ``demo 项目和用户：

    ```
    $ openstack role add --project demo --user demo user
    ```

> 你可以重复此过程来创建额外的项目和用户。
