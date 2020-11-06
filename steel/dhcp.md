## DHCP简介

DHCP（Dynamic Host Configuration Protocol，动态主机配置协议）用来为网络设备动态地分配IP地址等网络配置参数。  
DHCP采用客户端/服务器通信模式，由客户端向服务器提出请求分配网络配置参数的申请，服务器返回为客户端分配的IP地址等配置信息，以实现IP地址等信息的动态配置。

## DHCP的IP地址分配

### IP地址分配策略

针对客户端的不同需求，DHCP提供三种IP地址分配策略：

* 手工分配地址：由管理员为少数特定客户端（如WWW服务器等）静态绑定固定的IP地址。通过DHCP将配置的固定IP地址分配给客户端。

* 自动分配地址：DHCP为客户端分配租期为无限长的IP地址。

* 动态分配地址：DHCP为客户端分配具有一定有效期限的IP地址，到达使用期限后，客户端需要重新申请地址。绝大多数客户端得到的都是这种动态分配的地址。

### IP地址获取过程

DHCP客户端从DHCP服务器获取IP地址，主要通过四个阶段进行：

1. 发现阶段，即DHCP客户端寻找DHCP服务器的阶段。客户端以广播方式发送DHCP-DISCOVER报文。

2. 提供阶段，即DHCP服务器提供IP地址的阶段。DHCP服务器接收到客户端的DHCP-DISCOVER报文后，根据IP地址分配的优先次序选出一个IP地址，与其他参数一起通过DHCP-OFFER报文发送给客户端。

3. 选择阶段，即DHCP客户端选择IP地址的阶段。如果有多台DHCP服务器向该客户端发来DHCP-OFFER报文，客户端只接受第一个收到的DHCP-OFFER报文，然后以广播方式发送DHCP-REQUEST报文，该报文中包含DHCP服务器在DHCP-OFFER报文中分配的IP地址。

4. 确认阶段，即DHCP服务器确认IP地址的阶段。DHCP服务器收到DHCP客户端发来的DHCP-REQUEST报文后，只有DHCP客户端选择的服务器会进行如下操作：如果确认将地址分配给该客户端，则返回DHCP-ACK报文；否则返回DHCP-NAK报文，表明地址不能分配给该客户端。

![IP地址动态获取过程](/images/dhcp.png)

客户端收到服务器返回的DHCP-ACK确认报文后，会以广播的方式发送免费ARP报文，探测是否有主机使用服务器分配的IP地址，如果在规定的时间内没有收到回应，并且客户端上不存在与该地址同网段的其他地址时，客户端才使用此地址。否则，客户端会发送DHCP-DECLINE报文给DHCP服务器，并重新申请IP地址。

如果网络中存在多个DHCP服务器，除DHCP客户端选中的服务器外，其它DHCP服务器中本次未分配出的IP地址仍可分配给其他客户端。

### IP地址的租约更新

DHCP服务器分配给客户端的IP地址具有一定的租借期限（除自动分配的IP地址），该租借期限称为租约。当租借期满后服务器会收回该IP地址。如果DHCP客户端希望继续使用该地址，则DHCP客户端需要申请延长IP地址租约。

在DHCP客户端的IP地址租约期限达到一半左右时间时，DHCP客户端会向为它分配IP地址的DHCP服务器单播发送DHCP-REQUEST报文，以进行IP租约的更新。如果客户端可以继续使用此IP地址，则DHCP服务器回应DHCP-ACK报文，通知DHCP客户端已经获得新IP租约；如果此IP地址不可以再分配给该客户端，则DHCP服务器回应DHCP-NAK报文，通知DHCP客户端不能获得新的租约。

如果在租约的一半左右时间进行的续约操作失败，DHCP客户端会在租约期限达到7/8时，广播发送DHCP-REQUEST报文进行续约。DHCP服务器的处理方式同上，不再赘述。

## DHCP报文

### DHCP报文类型

DHCP有8种类型报文。

* DHCP DISCOVER

DHCP客户端请求地址时，并不知道DHCP服务器的位置，因此DHCP客户端会在本地网络内以广播方式发送请求报文，这个报文成为Discover报文，目的是发现网络中的DHCP服务器，所有收到Discover报文的DHCP服务器都会发送回应报文，DHCP客户端据此可以知道网络中存在的DHCP服务器的位置。

* DHCP OFFER

DHCP服务器收到Discover报文后，就会在所配置的地址池中查找一个合适的IP地址，加上相应的租约期限和其他配置信息（如网关、DNS服务器等），构造一个Offer报文，发送给用户，告知用户本服务器可以为其提供IP地址。< 只是告诉client可以提供，是预分配，还需要client通过ARP检测该IP是否重复>

* DHCP REQUEST

DHCP客户端可能会收到多个Offer，所以必须在这些回应中选择一个。Client通常选择第一个回应Offer报文的服务器作为自己的目标服务器，并回应一个广播Request报文，通告选择的服务器。DHCP客户端成功获取IP地址后，在地址使用租期过去1/2时，会向DHCP服务器发送单播Request报文续延租期，如果没有收到DHCP ACK报文，在租期过去3/4时，发送广播Request报文续延租期。

* DHCP ACK

DHCP服务器收到Request报文后，根据Request报文中携带的用户MAC来查找有没有相应的租约记录，如果有则发送ACK报文作为回应，通知用户可以使用分配的IP地址。

* DHCP NAK

如果DHCP服务器收到Request报文后，没有发现有相应的租约记录或者由于某些原因无法正常分配IP地址，则发送NAK报文作为回应，通知用户无法分配合适的IP地址。

* DHCP DECLINE

当用户不再需要使用分配IP地址时，就会主动向DHCP服务器发送Release报文，告知服务器用户不再需要分配IP地址，DHCP服务器会释放被绑定的租约。

* DHCP RELEASE

DHCP客户端收到DHCP服务器回应的ACK报文后，通过地址冲突检测发现服务器分配的地址冲突或者由于其他原因导致不能使用，则发送Decline报文，通知服务器所分配的IP地址不可用。

* DHCP INFORM

DHCP客户端如果需要从DHCP服务器端获取更为详细的配置信息，则发送Inform报文向服务器进行请求，服务器收到该报文后，将根据租约进行查找，找到相应的配置信息后，发送ACK报文回应DHCP客户端。<极少用到>

### DHCP报文格式

DHCP8种类型的报文，每种报文的格式都相同，只是某些字段的取值不同。DHCP的报文格式如下图所示，括号中的数字表示该字段所占的字节。

![DHCP报文格式](/images/dhcp_message.png)

字段解释：

* op：报文的操作类型，分为请求报文和响应报文，1为请求报文；2为响应报文。具体的报文类型在options字段中标识。

* htype、hlen：DHCP客户端的硬件地址类型及长度。

* hops：DHCP报文经过的DHCP中继的数目。DHCP请求报文每经过一个DHCP中继，该字段就会增加1。

* xid：客户端发起一次请求时选择的随机数，用来标识一次地址请求过程。

* secs：DHCP客户端开始DHCP请求后所经过的时间。目前没有使用，固定为0。

* flags：第一个比特为广播响应标识位，用来标识DHCP服务器响应报文是采用单播还是广播方式发送，0表示采用单播方式，1表示采用广播方式。其余比特保留不用。

* ciaddr：DHCP客户端的IP地址。如果客户端有合法和可用的IP地址，则将其添加到此字段，否则字段设置为0。此字段不用于客户端申请某个特定的IP地址。

* yiaddr：DHCP服务器分配给客户端的IP地址。

* siaddr：DHCP客户端获取启动配置信息的服务器IP地址。

* giaddr：DHCP客户端发出请求报文后经过的第一个DHCP中继的IP地址。

* chaddr：DHCP客户端的硬件地址。

* sname：DHCP客户端获取启动配置信息的服务器名称。

* file：DHCP服务器为DHCP客户端指定的启动配置文件名称及路径信息。

* options：可选变长选项字段，包含报文的类型、有效租期、DNS服务器的IP地址、WINS服务器的IP地址等配置信息。

## DHCP选项

### DHCP选项简介

为了与BOOTP（Bootstrap Protocol，自举协议）兼容，DHCP保留了BOOTP的消息格式。DHCP和BOOTP消息的不同主要体现在选项（Options）字段。DHCP在BOOTP基础上增加的功能，通过Options字段来实现。

DHCP利用Options字段传递控制信息和网络配置参数，实现地址动态分配的同时，为客户端提供更加丰富的网络配置信息。

DHCP选项的格式下图所示：

![DHCP选项格式](/images/dhcp_option.png)

### DHCP常用选项介绍

常见的DHCP选项有：

* Option 3：路由器选项，用来指定为客户端分配的网关地址。

* Option 6：DNS服务器选项，用来指定为客户端分配的DNS服务器地址。

* Option 33：静态路由选项。该选项中包含一组有分类静态路由（即目的网络地址的掩码固定为自然掩码，不能划分子网），客户端收到该选项后，将在路由表中添加这些静态路由。如果Option 33和Option 121同时存在，则忽略Option 33。

* Option 51：IP地址租约选项。

* Option 53：DHCP消息类型选项，标识DHCP消息的类型。

* Option 55：请求参数列表选项。客户端利用该选项指明需要从服务器获取哪些网络配置参数。该选项内容为客户端请求的参数对应的选项值。

* Option 60：厂商标识选项。客户端利用该选项标识自己所属的厂商；DHCP服务器可以根据该选项区分客户端所属的厂商，并为其分配特定范围的IP地址。

* Option 66：TFTP服务器名选项，用来指定为客户端分配的TFTP服务器的域名。

* Option 67：启动文件名选项，用来指定为客户端分配的启动文件名。

* Option 121：无分类路由选项。该选项中包含一组无分类静态路由（即目的网络地址的掩码为任意值，可以通过掩码来划分子网），客户端收到该选项后，将在路由表中添加这些静态路由。如果Option 33和Option 121同时存在，则忽略Option 33。

* Option 150：TFTP服务器地址选项，用来指定为客户端分配的TFTP服务器的地址。

更多DHCP选项的介绍，请参见RFC 2132和RFC 3442。

## 协议规范

与DHCP相关的协议规范：

* RFC 2131：Dynamic Host Configuration Protocol

* RFC 2132：DHCP Options and BOOTP Vendor Extensions

* RFC 1542：Clarifications and Extensions for the Bootstrap Protocol

* RFC 3046：DHCP Relay Agent Information Option

* RFC 3442：The Classless Static Route Option for Dynamic Host Configuration Protocol (DHCP) version 4
