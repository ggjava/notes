之前在介绍netstat的时候说过，netstat是一个非常实用的socket查看命令。ss是Socket Statistics的缩写。顾名思义，ss命令可以用来获取socket统计信息，它可以显示和netstat类似的内容。但ss的优势在于它能够显示更多更详细的有关TCP和连接状态的信息，而且比netstat更快速更高效。

### 为什么使用ss

值得注意的是，几乎所有的linux系统都默认支持netstat命令，而并不一定支持ss，从这一点来说，netstat通常还是不二选择。但是不得不承认的是，ss命令更加快捷高效。

netstat从proc文件系统获取所需要的信息，而ss利用netlink机制，与内核通信，通过TCP 协议栈中 tcp_diag 模块获取第一手的内核信息。当然这些都不是我们关注的重点，我们来看看ss命令到底如何使用。

### 命令参数

-h, --help          帮助信息  
-V, --version       程序版本信息  
-n, --numeric       不解析服务名称  
-r, --resolve       解析主机名  
-a, --all           显示所有套接字（sockets）  
-l, --listening     显示监听状态的套接字（sockets）  
-o, --options       显示计时器信息  
-e, --extended      显示详细的套接字（sockets）信息  
-m, --memory        显示套接字（socket）的内存使用情况  
-p, --processes     显示使用套接字（socket）的进程  
-i, --info          显示 TCP内部信息  
-s, --summary       显示套接字（socket）使用概况  
-4, --ipv4          仅显示IPv4的套接字（sockets）  
-6, --ipv6          仅显示IPv6的套接字（sockets）  
-0, --packet        显示 PACKET 套接字（socket）  
-t, --tcp           仅显示 TCP套接字（sockets）  
-u, --udp           仅显示 UCP套接字（sockets）  
-d, --dccp          仅显示 DCCP套接字（sockets）  
-w, --raw           仅显示 RAW套接字（sockets）  
-x, --unix          仅显示 Unix套接字（sockets）  
-f, --family=FAMILY  显示 FAMILY类型的套接字（sockets），FAMILY可选，支持  unix, inet, inet6, link, netlink  
-A, --query=QUERY, --socket=QUERY
      QUERY := {all|inet|tcp|udp|raw|unix|packet|netlink}[,QUERY]
-D, --diag=FILE     将原始TCP套接字（sockets）信息转储到文件  
-F, --filter=FILE   从文件中都去过滤器信息  
       FILTER := [ state TCP-STATE ] [ EXPRESSION ]

### 查看TCP/UDP连接

使用-t（TCP）参数查看TCP连接，而使用-u(UDP)参数查看UDP socket：

```
$ ss -t
State       Recv-Q Send-Q                                     Local Address:Port                                                      Peer Address:Port
ESTAB       0      0                                          192.168.0.103:56296                                                   113.107.216.82:https
ESTAB       0      0                                          192.168.0.103:56540                                                  185.199.108.153:https
ESTAB       0      0                                              127.0.0.1:socks                                                        127.0.0.1:44452
ESTAB       0      0                                              127.0.0.1:42150                                                        127.0.0.1:9614
```

其中state显示了当前连接的状态，例如结果的第一行是ESTABLISHED状态，Local Address:port代表本地连接的ip和端口号。另外使用-n参数显示数字形式的ip和端口。

### 查看socket进程信息

查看到某个连接后，怎么知道是哪个进程的连接呢？使用-p（processes）即可，例如：

```
$ ss -tp
State       Recv-Q Send-Q                                     Local Address:Port                                                      Peer Address:Port
ESTAB       0      0                                              127.0.0.1:42150                                                        127.0.0.1:9614                  users:(("chrome",pid=2578,fd=347))
ESTAB       0      0                                              127.0.0.1:41910                                                        127.0.0.1:9614                  users:(("chrome",pid=2578,fd=383))
```

拖动滚动条到最后可以看到，-p参数显示了这条连接的进程信息，例如，对于第一条结果，可以看到，该进程是chrome，进程id为2578，并且这条连接的文件描述符为383。

users:(("chrome",pid=2578,fd=383))
查看处于特定状态的socket

我们知道，对于TCP连接来讲，在不同的阶段它的状态不同，常见状态有

* ESTABLISHED  已建立
* CLOSED   已关闭
* LISTENING 正在监听
* FIN-WAIT-2 等待连接关闭
* TIME-WAIT 等待足够时间，确保服务器正常关闭该连接
* ……

这里还有很多其他状态，我们会留到介绍TCP的时候展开。

如何查看处于特定状态的连接呢？例如，要查看处于LISTENING状态的连接：

```
$ ss -t state LISTENING
Recv-Q Send-Q                                          Local Address:Port                                                           Peer Address:Port
0      5                                                   127.0.1.1:domain                                                                    *:*
0      128                                                 127.0.0.1:5941                                                                      *:*
0      5                                                   127.0.0.1:ipp                                                                       *:*
```

使用state选项即可查看。当然对于LISTENING状态，也可以使用-l参数。

除此之外，还有以下参数，用于查看某类状态，例如：

* all 所有类型
* connected  除closed和listen状态以外已连接的状态
* synchronized  除了syn-sent外的状态

### 查看TCP相关定时器信息

我们知道在TCP中，有很多定时器，和netstat一样，可以使用-o参数显示定时器相关信息：

```
$ ss -to
State       Recv-Q Send-Q                                     Local Address:Port                                                      Peer Address:Port
ESTAB       0      0                                              127.0.0.1:44660                                                        127.0.0.1:socks                 timer:(keepalive,4min42sec,0)
ESTAB       0      0                                          192.168.0.103:60306                                                    203.208.41.37:https                 timer:(keepalive,9.956ms,0)
ESTAB       0      0  
```

例如上面显示的keepalive定时器剩余时间：

timer:(keepalive,9.956ms,0)

### 查看socket详细信息

如果想要查看连接更加详细信息呢？比如收到多少数据？上一个ACK是什么时候？mss是多大？拥塞窗口大小是多少？这些信息在分析理解TCP的时候非常有帮助，而查看这些信息只需要使用-i(information)参数即可：

```
$ ss -ti  #(内容很长，省略了很多信息，可执行尝试)
cubic wscale:7,7 rto:204 rtt:2.302/4.528 ato:40 mss:23488 cwnd:10 bytes_acked:1560 bytes_received:3907 segs_out:18 segs_in:20 send 816.3Mbps lastsnd:1384 lastrcv:1384 lastack:1384 pacing_rate 1632.1Mbps rcv_rtt:546 rcv_space:43690
```

由于显示的内容比较多，这里就不贴出来了，可自行尝试，里面展示了TCP很多关键信息。

### 查看socket内存使用情况

使用-m（memory）参数可以查看连接使用内存信息：

```
$ ss -tm  #只显示内存部分信息
skmem:(r0,rb374400,t0,tb46080,f0,w0,o0,bl0)
```

由于信息较多，这里只显示内存部分，括号内从左到右分别代表：

* 接收报文分配的内存
* 接收报文可分配的内存
* 发送报文分配的内存
* 发送报文可分配的内存
* socket使用的缓存
* 为将要发送的报文分配的内存
* 保存socket选项使用的内存
* 连接队列使用的内存

### 根据IP或端口过滤socket信息

你可以使用grep命令来过滤出你需要的信息，但是ss本身提供一些参数用来过滤信息。例如，查看本地ip为192.168.0.103的连接：

```
$ ss -t src 192.168.0.103
State       Recv-Q Send-Q                                     Local Address:Port                                                      Peer Address:Port
ESTAB       0      0                                          192.168.0.103:44528                                                  185.199.109.153:https  
$ ss -t src 192.168.0.103:35418
State       Recv-Q Send-Q                                     Local Address:Port                                                      Peer Address:Port
ESTAB       0      0                                          192.168.0.103:35418                                                  111.230.120.127:https
```

src后面跟本地ip:port，而也可以使用dst根据远端ip来过滤信息。

同样还可以根据协议类型（端口）来过滤，例如查看https socket信息：

```
$ ss -t '( dport = :https or sport = :https )'
State       Recv-Q Send-Q                                     Local Address:Port                                                      Peer Address:Port
ESTAB       0      0                                          192.168.0.103:44528                                                  185.199.109.153:https
ESTAB       0      0                                          192.168.0.103:35418                                                  111.230.120.127:https
$ ss -t dport = :https
State       Recv-Q Send-Q                                     Local Address:Port                                                      Peer Address:Port
CLOSE-WAIT  32     0                                          192.168.0.103:46626                                                   123.58.182.252:https
$ ss -t sport > :44550   #显示本地端口大于44550的连接
State       Recv-Q Send-Q                                     Local Address:Port                                                      Peer Address:Port
ESTAB       390    0                                              127.0.0.1:46468                                                        127.0.0.1:socks
ESTAB       0      0                                              127.0.0.1:46382                                                        127.0.0.1:socks
ESTAB       0      0                                              127.0.0.1:46490                                                        127.0.0.1:socks
```

其中dport，指定本地协议（应用 端口），sport指定远端协议（应用 端口）。

### 显示socket统计信息

使用-s(summary)查看整体统计信息。

```
$ ss -s
Total: 1379 (kernel 2907)
TCP:   68 (estab 58, closed 1, orphaned 0, synrecv 0, timewait 1/0), ports 0

Transport Total     IP        IPv6
*      2907      -         -
RAW      1         0         1
UDP      13        8         5
TCP      67        47        20
INET      81        55        26
FRAG      0         0         0
```

从统计结果中可以看到，共有67个TCP连接。

### 总结

本文介绍了ss命令一些实用的用法，为后面介绍网络编程相关内容打下基础，其他诸如指定ipv4或协议族等更多ss用法可查看帮助手册