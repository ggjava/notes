
v1.0.5版本磁盘清理由从串行变为并行模式，每块磁盘删除都需要经过三个步。

### 查询磁盘信息

> lsblk命令用来查看块设备的信息，块设备有硬盘，闪存盘，cd-ROM等等，我们主要用于获取disk相关信息，包括name、model、size等信息.

```bash
lsblk -Pbdi -o KNAME,MODEL,SIZE,ROTA,TYPE,TRAN
```

## 1. 格式化

### 列出块设备的信息

```bash
lsblk ${device_name} -rnpf -o NAME,FSTYPE
```

### 销毁文件系统

销毁磁盘文件系统。如果设备有逻辑分区，分区上的文件系统也将被销毁。

```bash
mkfs.ext4 -F -T largefile ${part_dev}
```

## 2.shred擦除块设备

支持 low,normal,high 三种模式

* low 擦除前100M, 擦除 1 次
* normal 全部擦除, 擦除 1 次
* high 全部擦除, 擦除 3 次

```bash
shred --force --zero --verbose --iterations{$teration} --size={$size} ${device_name}
```

## 3.销毁节点磁盘上的元数据结构

首先使用wipefs命令尝试强制擦除磁盘信息

```bash
wipefs --force --all ${device_name}
```

如果--force不支持，去掉--force

```bash
wipefs --all ${device_name}
```

```bash
dd bs=512 if=/dev/zero ${dd_device} count=33
dd bs=512 if=/dev/zero ${dd_device} count=33 ${dd_seek}
```

sgdisk擦除分区数据

```bash
sgdisk --zap-all --clear --mbrtogpt ${device_name}
```
