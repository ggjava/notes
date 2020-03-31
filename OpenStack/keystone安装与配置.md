这一章描述如何在控制节点上安装和配置OpenStack身份认证服务，代码名称keystone。出于性能原因，这个配置部署Fernet令牌和Apache HTTP服务处理请求。

### 先决条件

在你配置 OpenStack 身份认证服务前，你必须创建一个数据库和管理员令牌。

1. 完成下面的步骤以创建数据库：
    * 用数据库连接客户端以 root 用户连接到数据库服务器：

    ```sql
    mysql -u root -p
    ```

    * 创建 keystone 数据库：

    ```sql
    CREATE DATABASE keystone;
    ```

    *  对keystone数据库授予恰当的权限：

    ```sql
    GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
    IDENTIFIED BY 'KEYSTONE_DBPASS';
    GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
    IDENTIFIED BY 'KEYSTONE_DBPASS';
    ```

    用合适的密码替换 KEYSTONE_DBPASS 。

    * 退出数据库客户端。

2. 生成一个随机值在初始的配置中作为管理员的令牌。

```
$ openssl rand -hex 10
```

### 安全并配置组件

> 默认配置文件在各发行版本中可能不同。你可能需要添加这些部分，选项而不是修改已经存在的部分和选项。另外，在配置片段中的省略号(...)表示默认的配置选项你应该保留。

> 教程使用带有``mod_wsgi``的Apache HTTP服务器来服务认证服务请求，端口为5000和35357。缺省情况下，Kestone服务仍然监听这些端口。然而，本教程手动禁用keystone服务。

1. 运行以下命令来安装包。

```bash
yum install openstack-keystone httpd mod_wsgi
```

2. 编辑文件 /etc/keystone/keystone.conf 并完成如下动作：

    * 在``[DEFAULT]``部分，定义初始管理令牌的值：

    ```
    [DEFAULT]
    ...
    admin_token = ADMIN_TOKEN
    ```

    使用前面步骤生成的随机数替换``ADMIN_TOKEN`` 值。

    * 在 [database] 部分，配置数据库访问：

    ```
    [database]
    ...
    connection = mysql+pymysql://keystone:KEYSTONE_DBPASS@controller/keystone
    ```

    将``KEYSTONE_DBPASS``替换为你为数据库选择的密码。

    * 在``[token]``部分，配置Fernet UUID令牌的提供者。

    ```
    [token]
    ...
    provider = fernet
    ```

3. 初始化身份认证服务的数据库

```bash
su -s /bin/sh -c "keystone-manage db_sync" keystone
```

4. 初始化Fernet keys：

```bash
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
```

### 配置 Apache HTTP 服务器

1. 编辑``/etc/httpd/conf/httpd.conf`` 文件，配置``ServerName`` 选项为控制节点：

```
ServerName controller
```

2. 用下面的内容创建文件 /etc/httpd/conf.d/wsgi-keystone.conf。

```
Listen 5000
Listen 35357

<VirtualHost *:5000>
    WSGIDaemonProcess keystone-public processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-public
    WSGIScriptAlias / /usr/bin/keystone-wsgi-public
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat "%{cu}t %M"
    ErrorLog /var/log/httpd/keystone-error.log
    CustomLog /var/log/httpd/keystone-access.log combined

    <Directory /usr/bin>
        Require all granted
    </Directory>
</VirtualHost>

<VirtualHost *:35357>
    WSGIDaemonProcess keystone-admin processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-admin
    WSGIScriptAlias / /usr/bin/keystone-wsgi-admin
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat "%{cu}t %M"
    ErrorLog /var/log/httpd/keystone-error.log
    CustomLog /var/log/httpd/keystone-access.log combined

    <Directory /usr/bin>
        Require all granted
    </Directory>
</VirtualHost>
```

### 完成安装

启动 Apache HTTP 服务并配置其随系统启动：

```
systemctl enable httpd.service
systemctl start httpd.service
```