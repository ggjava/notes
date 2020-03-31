MegaCli是一款管理维护硬件RAID软件，可以查看当前raid卡的所有信息：raid卡的型号，raid的阵列类型，raid的磁盘状态；可以对raid进行管理：在线添加磁盘，创建磁盘阵列、删除阵列等。MegaCli是LSI公司官方提供的SCSI卡管理工具。由于被收购变成了现在的Broadcom，所以现在想下载MegaCli，需要去Broadcom官网查找Legacy产品支持，搜索MegaRAID。

## MegaCli安装

```bash
wget ftp://download2.boulder.ibm.com/ecc/sar/CMA/XSA/ibm_utl_sraidmr_megacli-8.00.48_linux_32-64.zip
unzip ibm_utl_sraidmr_megacli-8.00.48_linux_32-64.zip
cd linux
rpm -ivh Lib_Utils-1.00-09.noarch.rpm  MegaCli-8.00.48-1.i386.rpm
或
rpm -ivh Lib_Utils-1.00-09.noarch.rpm  MegaCli-8.00.48-1.i386.rpm --replacefiles
```

## 常见参数含义

一般通过MegaCli的Media Error Count 、Other Error Count、Predictive Failure Count来确定阵列中磁盘是否有问题

* Medai Error Count 不为0，表示磁盘可能错误，可能是磁盘有坏道，数值越大，危险系数越高
* Other Error Count 不为0，表示磁盘可能存在松动，可能需要重新再插入
* Predictive Failure Count：表示监控硬盘的预报错误数量，不为0要更换
* Slot Number：slot号，应该跟机器外观上的标识一致。（磁盘位置）
* Inquiry Data: 磁盘的序列号，跟磁盘标签上一致。（磁盘标签需要拔盘才能看到）
* Firmware state: 这磁盘的状态，Online是最好的状态，除此之外还有 Unconfigured Offline Failed
* Last Predictive Failure Event Seq Number：最后一条预警的时间序列号
* Raw Size：磁盘大小
* Firmware state：磁盘目前的状态

## 磁盘状态

* Unconfigured Good ：未配置好。 RAID控制器可访问的驱动器，但未配置为虚拟驱动器或热备分
* Online：在线
* Rebuild ：重建。写入数据的驱动器，以恢复虚拟驱动器的完全冗余
* Failed ：失败
* Unconfigured Bad：未配置的坏-驱动器上的固件检测不可恢复的错误；驱动器无法初始化Unconfigured Good或驱动器
* Missing：失踪。在线驱动，但已从其位置移除
* Offline：脱机-驱动器是虚拟驱动器的一部分，但在RAID中具有无效数据或未配置
* Hot Spare：热备份
* None：具有不支持标志集的驱动器。具有未配置的良好或离线驱动器，完成了搬迁作业的准备工作

## RAID Level对应关系

* RAID Level : Primary-1, Secondary-0, RAID Level Qualifier-0 RAID 1
* RAID Level : Primary-0, Secondary-0, RAID Level Qualifier-0 RAID 0
* RAID Level : Primary-5, Secondary-0, RAID Level Qualifier-3 RAID 5
* RAID Level : Primary-1, Secondary-3, RAID Level Qualifier-0 RAID 10

## MegaCli常用命令

查raid级别

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -LDInfo -Lall -aALL
```

查raid卡信息

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -AdpAllInfo -aALL
```

查看硬盘信息

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -PDList -aALL
```

查看电池信息

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -AdpBbuCmd -aAll
```

查看raid卡日志

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -FwTermLog -Dsply -aALL
```

显示适配器个数

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -adpCount
```

显示适配器时间

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -AdpGetTime –aALL
```

显示所有适配器信息

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -AdpAllInfo -aAll
```

显示所有逻辑磁盘组信息

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -LDInfo -LALL -aAll
```

显示所有的物理信息

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -PDList -aAll
```

查看充电状态

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -AdpBbuCmd -GetBbuStatus -aALL |grep ‘Charger Status’
```

显示BBU状态信息

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -AdpBbuCmd -GetBbuStatus -aALL
```

显示BBU容量信息

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -AdpBbuCmd -GetBbuCapacityInfo -aALL
```

显示BBU设计参数

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -AdpBbuCmd -GetBbuDesignInfo -aALL
```

显示当前BBU属性

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -AdpBbuCmd -GetBbuProperties -aALL
```

显示Raid卡型号，Raid设置，Disk相关信息

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -cfgdsply -aALL
```

### 磁带状态的变化，从拔盘，到插盘的过程中

```bash
Device |Normal|Damage|Rebuild|Normal
Virtual Drive |Optimal|Degraded|Degraded|Optimal
Physical Drive |Online|Failed –> Unconfigured|Rebuild|Online
```

### 查看磁盘缓存策略

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -LDGetProp -Cache -L0 -a0
/opt/MegaRAID/MegaCli/MegaCli64 -LDGetProp -Cache -L1 -a0
/opt/MegaRAID/MegaCli/MegaCli64 -LDGetProp -Cache -LALL -a0
/opt/MegaRAID/MegaCli/MegaCli64 -LDGetProp -Cache -LALL -aALL
/opt/MegaRAID/MegaCli/MegaCli64 -LDGetProp -DskCache -LALL -aALL
```

### 设置磁盘缓存策略

缓存策略解释：
WT (Write through)
WB (Write back)
NORA (No read ahead)
RA (Read ahead)
ADRA (Adaptive read ahead)
Cached
Direct

## dell服务器raid的备注

DELL服务器的Riad卡都有可充电池的特性，这块可充电电池，在不使用时，也会有微弱的放电现象，当它的电量放电到低到一定程度时，Raid卡控制器就会对电池进行一次“放电”，将剩余的电量放掉，然后再进行一次“充电”，放电后，Raid卡默认电池不可用，为了保护磁盘数据，会将Raid状态改为no write cache :

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -LDGetProp -Cache -LAll -aAll
Adapter 0-VD 0(target id: 0): Cache Policy:WriteBack, ReadAdaptive, Direct, No Write Cache if Bad BBU #电池不可用时直接写入硬盘
```

直接写入硬盘会导致磁盘IO性能大为下降，对于高IO的应用会造成很大的压力，所以需要修改成：

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -LDSetProp CachedBadBBU -lall -a0
/opt/MegaRAID/MegaCli/MegaCli64 -LDGetProp -Cache -LAll -aAll
Adapter 0-VD 0(target id: 0): Cache Policy:WriteBack, ReadAdaptive, Direct, Write Cache OK #手动修改缓存策略，恢复IO性能
```

以下不常用:
创建一个 raid5 阵列，由物理盘 2,3,4 构成，该阵列的热备盘是物理盘 5

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -CfgLdAdd -r5 [1:2,1:3,1:4] WB Direct -Hsp[1:5] -a0
```

创建阵列，不指定热备

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -CfgLdAdd -r5 [1:2,1:3,1:4] WB Direct -a0
```

删除阵列

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -CfgLdDel -L1 -a0
```

在线添加磁盘

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -LDRecon -Start -r5 -Add -PhysDrv[1:4] -L1 -a0
```

阵列创建完后，会有一个初始化同步块的过程，可以看看其进度

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -LDInit -ShowProg -LALL -aALL
```

或者以动态可视化文字界面显示

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -LDInit -ProgDsply -LALL -aALL
```

查看阵列后台初始化进度

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -LDBI -ShowProg -LALL -aALL
```

或者以动态可视化文字界面显示

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -LDBI -ProgDsply -LALL -aALL
```

指定第 5 块盘作为全局热备

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -PDHSP -Set [-EnclAffinity] [-nonRevertible] -PhysDrv[1:5] -a0
```

指定为某个阵列的专用热备

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -PDHSP -Set [-Dedicated [-Array1]] [-EnclAffinity] [-nonRevertible] -PhysDrv[1:5] -a0
```

删除全局热备

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -PDHSP -Rmv -PhysDrv[1:5] -a0
```

将某块物理盘下线/上线

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -PDOffline -PhysDrv [1:4] -a0
/opt/MegaRAID/MegaCli/MegaCli64 -PDOnline -PhysDrv [1:4] -a0
```

查看物理磁盘重建进度

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -PDRbld -ShowProg -PhysDrv [1:5] -a0
```

或者以动态可视化文字界面显示

```bash
/opt/MegaRAID/MegaCli/MegaCli64 -PDRbld -ProgDsply -PhysDrv [1:5] -a0
```
