使用[CirrOS](http://launchpad.net/cirros)对镜像服务进行验证，CirrOS是一个小型的Linux镜像可以用来帮助你进行 OpenStack部署测试。

关于如何下载和构建镜像的更多信息，参考[OpenStack Virtual Machine Image Guide](http://docs.openstack.org/image-guide/)。关于如何管理镜像的更多信息，参考[OpenStack用户手册](http://docs.openstack.org/user-guide/common/cli-manage-images.html)。

在控制节点上执行这些命令。

1. 获得 admin 凭证来获取只有管理员能执行的命令的访问权限：

```bash
$ . admin-openrc
```

2. 下载源镜像：

```bash
$ wget http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img
```

如果您的发行版里没有包含wget，请安装它

3. 使用 QCOW2 磁盘格式， bare 容器格式上传镜像到镜像服务并设置公共可见，这样所有的项目都可以访问它：

```
$ openstack image create "cirros" \
  --file cirros-0.3.4-x86_64-disk.img \
  --disk-format qcow2 --container-format bare \
  --public
+------------------+------------------------------------------------------+
| Property         | Value                                                |
+------------------+------------------------------------------------------+
| checksum         | 133eae9fb1c98f45894a4e60d8736619                     |
| container_format | bare                                                 |
| created_at       | 2015-03-26T16:52:10Z                                 |
| disk_format      | qcow2                                                |
| file             | /v2/images/cc5c6982-4910-471e-b864-1098015901b5/file |
| id               | cc5c6982-4910-471e-b864-1098015901b5                 |
| min_disk         | 0                                                    |
| min_ram          | 0                                                    |
| name             | cirros                                               |
| owner            | ae7a98326b9c455588edd2656d723b9d                     |
| protected        | False                                                |
| schema           | /v2/schemas/image                                    |
| size             | 13200896                                             |
| status           | active                                               |
| tags             |                                                      |
| updated_at       | 2015-03-26T16:52:10Z                                 |
| virtual_size     | None                                                 |
| visibility       | public                                               |
+------------------+------------------------------------------------------+
```

更多关于命令:`glance image-create`的参数信息，请参考[OpenStack Command-Line Interface Reference](http://docs.openstack.org/cli-reference/openstack.html#openstack-image-create)中的`Image service command-line client`部分。

更多镜像磁盘和容器格式信息，参考[OpenStack虚拟机镜像指南](http://docs.openstack.org/image-guide/image-formats.html)中的镜像的磁盘及容器格式部分。

OpenStack 是动态生成 ID 的，因此您看到的输出会与示例中的命令行输出不相同。

4. 确认镜像的上传并验证属性：

```
$ openstack image list
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| 38047887-61a7-41ea-9b49-27987d5e8bb9 | cirros | active |
+--------------------------------------+--------+--------+
```

