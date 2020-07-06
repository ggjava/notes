## RAID概述

独立硬盘冗余阵列（Redundant Array of Independent Disks，RAID），简称磁盘阵列。其基本思想就是把多个相对便宜的硬盘组合起来，成为一个磁盘阵列。根据RAID级别不同，磁盘阵列有着比单块硬盘更强的数据集成度、容错功能、更高的处理量或容量。

RAID层级的命名会以RAID开头并带数字，例如：RAID 0、RAID 1、RAID 5、RAID 6、RAID 7、RAID 01、RAID 10、RAID 50、RAID 60。每种等级都有其理论上的优缺点，不同的等级在增加数据可靠性以及增加存储器（群）读写性能之间平衡。

另外，磁盘阵列对于服务器来说，看起来就像一个单独的硬盘或逻辑存储单元。

根据实现模式，分为软件磁盘阵列（Software RAID）和硬件磁盘阵列（Hardware RAID）两种。软件磁盘阵列主要由CPU处理数据存储作业，缺点为耗损较多CPU资源运算RAID；硬件磁盘阵列RAID卡上内置处理器，不需要服务器的CPU运算。优点是读写性能最快，不占用服务器资源。

## 常用RAID级别

### RAID 0

是种简单的、无数据校验的数据条带化技术，它并不提供冗余和容错能力。RAID0 将数据分散存储在所有磁盘中，以独立访问方式实现多块磁盘同时读写操作。所以 RAID0 是所有 RAID 级别中性能最高的。

需要磁盘数量：n ≥ 1

磁盘空间利用率：100%

优点：

* 读写可以并行。
* 理论上读写速率可达单个磁盘的 N 倍（N 为组成 RAID 0 的磁盘个数），但实际上受限于文件大小、文件系统大小等多种因素。

缺点：

* 没有数据冗余，即便只有单个磁盘损坏，在最严重的情况下也有可能导致所有数据的丢失。

### RAID 1

称为镜像，它将数据完全一致地分别写到【工作磁盘】和【镜像磁盘】在数据写入时，响应时间会有所影响，但是读数据的时候没有影响。
提供了最佳的数据保护，一旦工作磁盘发生故障，系统自动从镜像磁盘读取数据，不会影响用户工作。

需要磁盘数量：2

磁盘空间利用率：50%

优点：

* 读取速度快。

* 数据可靠性较高，单个磁盘的损坏不会导致数据的不可修复。

缺点：

* 磁盘利用率低。

* 写入速度受限于单个磁盘的写入速度。

### RAID 5

RAID5 是一种存储性能、数据安全和存储成本兼顾的存储解决方案，数据以块为单位分布到各个硬盘上。RAID5 不对数据进行备份，而是把数据和与对应的奇偶校验信息存储到各个磁盘上，并且数据和校验信息存储在不同磁盘。当一个磁盘数据损坏后，利用剩下的数据和校验信息去恢复数据。

需要磁盘数量：n ≥ 3
磁盘空间利用率：(n-1)/n %

优点：

* RAID 5可以理解为是RAID 0和RAID 1的折衷方案。RAID 5可以为系统提供数据安全保障，但保障程度要比镜像低而磁盘空间利用率要比镜像高。
* RAID 5具有和RAID 0相近似的数据读取速度，写入数据的速度相对单独写入一块硬盘的速度略慢。
* RAID 5的磁盘空间利用率要比RAID 1高，存储成本相对较便宜。

缺点：

* 只允许单盘故障，一盘出现故障得尽快处理。有盘坏情况下，raid5 IO/CPU性能狂跌，此时性能变的非常差。

### RAID 10

先用多个盘构建成 RAID 1，再用多个 RAID 1 构建成 RAID 0。

需要磁盘数量：n ≥ 4

磁盘空间利用率：50%

优点：

* 兼顾安全性和速度。基础4盘的情况下，raid10允许对柜盘2块故障，随着硬盘数量的提示，容错量也会相对应提升。这是raid5无法做到的。

缺点：

* 对盘的数量要求稍高，磁盘使用率为一半。

## 常见RAID卡

目前常见的RAID卡类型：

* LSI SAS2208
* LSI SAS2308
* LSI SAS3008
* LSI SAS3108

## RAID卡管理工具

一般RAID卡使用官方提供的MegaCli、storcli64、sas2ircu、sas3ircu等工具来管理，根据不同的RAID卡类型选择对应的管理工具。不支持RAID 5的卡，称为SAS卡，使用lsiutil工具来管理。  
HP的服务器则使用其特有的hpacucli、ssacli工具来管理。

### MegaCli

steel-server-agent提供Adapter：MegaCli64Adapter

支持RAID卡类型：

* MegaRAID SAS-3 3008/3108

* LSI Logic / Symbios Logic MegaRAID SAS-3 3108

支持机型：

* MANUFACTURER = "Dell Inc."  
  PRODUCT_NAME = "PowerEdge R730xd ("SKU=NotProvided;ModelName=PowerEdge R730XD)"

* MANUFACTURER = "Dell Inc."  
  PRODUCT_NAME = "PowerEdge R630 (SKU=NotProvided;ModelName=PowerEdge R630)"

* MANUFACTURER = "Dell Inc."  
  PRODUCT_NAME = "PowerEdge R740xd (SKU=NotProvided;ModelName=PowerEdge R740xd)"

* MANUFACTURER = "Dell Inc."  
  PRODUCT_NAME = "PowerEdge R740 (SKU=NotProvided;ModelName=PowerEdge R740)"

* MANUFACTURER = "FiberHome"  
  PRODUCT_NAME = "FitServer R2200 (Default string)"

* MANUFACTURER = "Huawei"  
  PRODUCT_NAME = "RH1288 V3 (Type1Sku0)"

* MANUFACTURER = "Lenovo"  
  PRODUCT_NAME = "ThinkSystem SR630 -[7X02CTO1WW]- (none)"

* MANUFACTURER = "LENOVO"  
  PRODUCT_NAME = "System x3650 M5: -[8871AC1]- ((none))"

* MANUFACTURER = "Lenovo"  
  PRODUCT_NAME = "ThinkSystem SR650 -[7X06CTO1WW]- (7X06CTO1WW)"

* MANUFACTURER = "Sugon"  
  PRODUCT_NAME = "I620-G30 (98001156)"
  
* MANUFACTURER = "Sugon"  
  PRODUCT_NAME = "I840-G30 (98001181)"

#### MegaCli常用命令

1. 查看磁盘信息

```bash
/usr/sbin/MegaCli64 -PDList -aALL
```

2. 查看逻辑磁盘信息

```bash
/usr/sbin/MegaCli64 -LDInfo -LALL -aALL
```

3. 查raid卡信息

```bash
/usr/sbin/MegaCli64 -AdpAllInfo -aALL
```

4. 做raid0

```bash
/usr/sbin/MegaCli64 -CfgLdAdd -r0 [2:0, 2:1] WB Direct CachedBadBBU -a0
```

5. 做raid10

```bash
/usr/sbin/MegaCli64 -CfgLdAdd -r1s0 -Array0[252:0,252:1] Array1[252:2,252:3] WB Direct CachedBadBBU -a0
```

6. 清除raid配置

```bash
/usr/sbin/MegaCli64 -CfgLdDel -Lall force -aALL
/usr/sbin/MegaCli64 -CfgForeign -Scan -a0
/usr/sbin/MegaCli64 -CfgForeign -Clear -a0
```

7. 关闭JBOD mode

```bash
/usr/sbin/MegaCli64 -AdpSetProp -EnableJBOD -0 -a0
```

### storcli64

steel-server-agent提供Adapter：StorcliAdapter

支持RAID卡类型：MegaRAID SAS-3 3108

支持机型：

* MANUFACTURER = "Huawei"  
  PRODUCT_NAME = "2288H V5 (Type1Sku0)"

* MANUFACTURER = "Huawei"  
  PRODUCT_NAME = "2288H V5 (Purley)"

* MANUFACTURER = "Huawei"  
  PRODUCT_NAME = "G560 V5 (Purley)"

#### storcli64常用命令

1. 查看第一块RAID卡版本、功能、状态、以及raid卡下的物理盘、逻辑盘信息。c0代表第一块raid卡。

```bash
/usr/sbin/storcli64 /c0 show all
```

2. 查看第一块RAID卡上物理磁盘详细信息

```bash
/usr/sbin/storcli64 c0 /eall /sall show all
```

3. 做raid0

```bash
/usr/sbin/storcli64 c0 add vd type=raid0 drives=252:0,1,2,3 wb cached force
```

4. 做raid10

```bash
/usr/sbin/storcli64 /c0 add vd type=raid10 drives=252:0-3 pdperarray=2 wb cached
```

5. 清除raid配置

```bash
# clean raid config
/usr/sbin/storcli64 /c0 /vall del force
# clean foreign config
/usr/sbin/storcli64 /c0 /fall del
```

6. 关闭JBOD mode

```bash
/usr/sbin/storcli64 /c0 set jbod=off force
```

### ssacli

steel-server-agent提供Adapter：SsacliAdapter

支持RAID卡类型：

* S Series
   * HPE Smart Array S100i SR Gen10 SW RAID   ProLiant、Apollo、Synergy、BladeSystem

* E Series (RAID/HBA)
   * HPE Smart Array E208i-a SR Gen10         ProLiant and Apollo
   * HPE Smart Array E208i-p SR Gen10         ProLiant and Apollo
   * HPE Smart Array E208e-p SR Gen10         ProLiant and Apollo
   * HPE Smart Array E208i-c SR Gen10         Synergy

* P Series (RAID/HBA)
   * HPE Smart Array P408i-a SR Gen10         ProLiant and Apollo
   * HPE Smart Array P408i-p SR Gen10         ProLiant and Apollo
   * HPE Smart Array P408e-p SR Gen10         ProLiant and Apollo
   * HPE Smart Array P816i-a SR Gen10         ProLiant and Apollo
   * HPE Smart Array P204i-c SR Gen10         Synergy
   * HPE Smart Array P408i-c SR Gen10         Synergy
   * HPE Smart Array P416ie-m SR Gen10        Synergy
   * HPE Smart Array P408e-m SR Gen10         BladeSystem
   * HPE Smart Array P204i-b SR Gen10         BladeSystem

支持机型：

* MANUFACTURER = "HPE"  
  PRODUCT_NAME = "ProLiant DL560 Gen10 (841730-B21)"

* MANUFACTURER = "HPE"  
  PRODUCT_NAME = "ProLiant DL380 Gen10 (868703-B21)"

#### ssacli常用命令

1. 查看raid卡信息

```bash
/usr/sbin/ssacli ctrl all show status
```

2. 查看物理盘信息，slot=0代表第一块raid卡

```bash
/usr/sbin/ssacli ctrl slot=0 array A pd all show
```

3. 查看逻辑盘信息，slot=0代表第一块raid卡

```bash
/usr/sbin/ssacli ctrl slot=0 array A ld all show
```

4. 做raid1

```bash
/usr/sbin/ssacli ctrl slot=0 create ld drives=1I:3:1,1I:3:2 raid=1 forced
```

5. 做raid10

```bash
/usr/sbin/ssacli ctrl slot=0 create ld drives=1I:3:1,1I:3:21I:3:3,1I:3:4 raid=1+0 forced
```

5. 清除raid配置

```bash
/usr/sbin/ssacli ctrl slot=0 ld all delete forced
/usr/sbin/ssacli ctrl slot=0 ld all remove spares=all forced
```

6. 关闭JBOD mode

```bash
/usr/sbin/ssacli ctrl slot=0 modify hbamode=off
```

### sas2ircu

steel-server-agent提供Adapter：SAS2IRCUAdapter

支持RAID卡类型：SAS2308

支持机型：

* MANUFACTURER = "Huawei Technologies Co., Ltd."  
  PRODUCT_NAME = "RH1288 V2-8S (Type1Sku0)"

#### sas2ircu常用命令

1. 查看所有RAID控制器信息

```bash
/usr/sbin/sas2ircu list
```

2. 查看第一块RAID控制器、物理磁盘、逻辑磁盘等详细信息，0代表RAID卡编号，

```bash
/usr/sbin/sas2ircu 0 display
```

3. 查看第一块RAID控制器所有逻辑磁盘信息

```bash
/usr/sbin/sas2ircu 0 status
```

4. 做raid1

```bash
/usr/sbin/sas2ircu 0 create raid 1 MAX 252:0 252:1 noprompt
```

5. 做raid10

```bash
/usr/sbin/sas2ircu 0 create raid 1 MAX 252:0 252:1 252:2 252:3 noprompt
```

6. 清除raid配置

```bash
/usr/sbin/sas2ircu 0 delete noprompt
```

### sas3ircu

steel-server-agent提供Adapter：SAS3IRCUAdapter

支持RAID卡类型：SAS3008

支持机型：

* MANUFACTURER = "Dell Inc."  
  PRODUCT_NAME = "PowerEdge R740xd (SKU=NotProvided;ModelName=PowerEdge R740xd)"

* MANUFACTURER = "Sugon"  
  PRODUCT_NAME = "W580-G20 (Default string)"

#### sas3ircu常用命令

1. 查看所有RAID控制器信息

```bash
/usr/sbin/sas3ircu list
```

2. 查看第一块RAID控制器、volume、物理磁盘等信息。0代表第一块RAID卡

```bash
/usr/sbin/sas3ircu 0 display
```

3. 查看第一块RAID控制器所有逻辑磁盘信息

```bash
/usr/sbin/sas3ircu 0 status
```

4. 做raid1

```bash
/usr/sbin/sas3ircu 0 create raid 1 MAX 252:0 252:1 noprompt
```

5. 做raid10

```bash
/usr/sbin/sas3ircu 0 create raid 1 MAX 252:0 252:1 252:2 252:3 noprompt
```

6. 清除raid配置

```bash
/usr/sbin/sas3ircu 0 delete noprompt
```