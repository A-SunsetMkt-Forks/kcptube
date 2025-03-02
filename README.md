# KCP Tube

**[Click Here for English Version](README_EN.md)**

## 简单介绍
但凡使用过三大运营商的家用宽带，并且需要家宽互联，那么几乎都会体验到 UDP 被限速的情况。为了躲避三大运营商针对 UDP 的 QoS，我制作了另一个工具，叫做 [UDP Hop](https://github.com/cnbatch/udphop)。原理是定期建立新连接（更换端口号及连到新地址）。

只不过，UDP Hop 只支持转发 UDP 流量。为了能够利用 UDP 转发 TCP 流量，因此就有了KCP Tube。利用 KCP 的可靠重传保证转发的 TCP 不会丢包。

制作 KCP Tube 的另一个原因是，其它 KCP 转发工具只能转发 TCP 流量，但我又需要用 KCP 转发 UDP 流量。主要是为了方便玩游戏。

当然了，其实 udphop 以及 kcptube 都是同时构想出来的。所以为了方便起见，先做好了 KCP Tube 搭好框架，接着再在 KCP Tube 的基础上裁剪成 UDP Hop。然后再把 UDP Hop 的修补代码反向合并回 KCP Tube。

为了方便家宽 Full Cone NAT 用户使用，KCP Tube 以服务端基本模式运行的时候可以利用 STUN 打洞，同时支持 IPv4 与 IPv6。

正如 [KCP](https://github.com/skywind3000/kcp) 本身的用途一样，KCP Tube 的主要目标是降低延迟，而不是偏向于传输超大流量。那么能不能传输超大流量呢？能，只是效果未必比得上现有的 TCP-KCP 转发工具。

### 支持的模式
目前支持 3 种模式：
- 客户端模式
- 服务端模式
- 中继节点模式

## 用法

**注意**，客户端的时间与服务端的时间务必同步，时间相差不能大于 255 秒。

### 全部用法
请前往 [Wiki 页面](https://github.com/cnbatch/kcptube/wiki)，或前往[文档页面](docs/README.md)。

若需要配置文件生成器，请前往此处：[KCPTube Generator](https://github.com/cnbatch/KCPTubeGenerator)

### 基本用法
`kcptube config.conf`

客户端模式示例：
```
mode=client
kcp=regular3
inbound_bandwidth=500M
outbound_bandwidth=50M
listen_port=59000
destination_port=3000
destination_address=123.45.67.89
encryption_password=qwerty1234
encryption_algorithm=AES-GCM
```

服务端模式示例：
```
mode=server
kcp=regular3
inbound_bandwidth=1G
outbound_bandwidth=1G
listen_port=3000
destination_port=59000
destination_address=::1
encryption_password=qwerty1234
encryption_algorithm=AES-GCM
stun_server=stun.qq.com
log_path=./
```

备注：初次连接时，服务端会向客户端告知自己的端口范围，因此客户端模式的 `listen_port` 不一定要等于服务端模式的 `destination_port`，两边的端口可以不一致，但客户端所写的端口号范围不能超出服务端的范围，以免客户端选错端口连接不上。

如果要指定侦听的网卡，那就指定该网卡的 IP 地址，加一行即可
```
listen_on=192.168.1.1
```

如果想要侦听多个端口、多个网卡，那就分开多个配置文件

```
kcptube config1.conf config2.conf
```

如果想在连接前测试一下连接是否畅通，可以加上 ``--try`` 选项

```
kcptube --try config1.conf
```
或
```
kcptube config1.conf --try
```

#### 验证配置文件
使用 ``--check-config`` 选项即可验证配置文件是否正确：
```
kcptube --check-config config1.conf
```
或
```
kcptube config1.conf --check-config
```

### 更灵活用法——服务端模式动态端口

客户端模式示例：
```
mode=client
kcp=regular3
inbound_bandwidth=500M
outbound_bandwidth=50M
listen_port=6000
destination_port=3000-4000
destination_address=123.45.67.89
dport_refresh=600
encryption_password=qwerty1234
encryption_algorithm=AES-GCM
```

服务端模式示例：
```
mode=server
kcp=regular3
inbound_bandwidth=1G
outbound_bandwidth=1G
listen_port=3000-4000
destination_port=6000
destination_address=::1
encryption_password=qwerty1234
encryption_algorithm=AES-GCM
```

### 自己指定 KCP 选项

客户端模式示例：
```
mode=client
kcp=manual
kcp_mtu=1400
kcp_sndwnd=512
kcp_rcvwnd=2048
kcp_nodelay=1
kcp_interval=10
kcp_resend=2
kcp_nc=true
udp_timeout=300
listen_port=6000
destination_port=3000-4000
destination_address=123.45.67.89
dport_refresh=600
encryption_password=qwerty1234
encryption_algorithm=AES-GCM
```

服务端模式示例：
```
mode=server
kcp=manual
kcp_mtu=1400
kcp_sndwnd=512
kcp_rcvwnd=2048
kcp_nodelay=1
kcp_interval=10
kcp_resend=2
kcp_nc=true
udp_timeout=300
listen_port=3000-4000
destination_port=6000
destination_address=::1
encryption_password=qwerty1234
encryption_algorithm=AES-GCM
```

### 参数介绍

|  名称   | 可设置值  | 必填 |备注|
|  ----  | ----  | :----: | ---- |
| mode  | client<br>server<br>relay |是|客户端<br>服务端<br>中继节点|
| listen_on | 域名或 IP 地址 |否|只能填写域名或 IP 地址。多个地址请用逗号分隔|
| listen_port | 1 - 65535 |是|以服务端运行时可以指定端口范围|
| destination_port | 1 - 65535 |是|以客户端运行时可以指定端口范围。多个地址请用逗号分隔|
| destination_address  | IP地址、域名 |是|填入 IPv6 地址时不需要中括号|
| dport_refresh  | 0 - 32767 |否|单位“秒”。不填写表示使用预设值 60 秒。<br>1 至 20 按 20 秒算，大于 32767 按 32767 秒算。<br>设为 0 表示禁用。|
| encryption_algorithm | XOR<br>AES-GCM<br>AES-OCB<br>chacha20<br>xchacha20<br>none |否    |单纯异或运算<br>AES-256-GCM-AEAD<br>AES-256-OCB-AEAD<br>ChaCha20-Poly1305<br>XChaCha20-Poly1305<br>不加密 |
| encryption_password  | 任意字符            |视情况|设置了 encryption_algorithm 使用加密时必填，none与XOR除外|
| udp_timeout  | 0 - 65535 |否|单位“秒”。预设值 180 秒，设为 0 则使用预设值<br>该选项表示的是，UDP 应用程序 ↔ kcptube 之间的超时设置|
| keep_alive  | 0 - 65535 |否 |单位“秒”。预设值为 0，等于停用 Keep Alive<br>该选项是指两个KCP端之间的Keep Alive<br>可单方面启用，用于检测通道是否停止响应。若超过30秒仍未有回应，就关闭通道。|
| mux_tunnels  | 0 - 65535 |否 | 预设值为 0，等于不使用多路复用通道<br>该选项是指两个KCP端之间的多路复用通道数<br>仅限客户端启用|
| stun_server  | STUN 服务器地址 |否|listen_port 为端口范围模式时不可使用|
| log_path  | 存放 Log 的目录 |否|不能指向文件本身|
| fec  | uint8:uint8 |否|格式为 `fec=D:R`，例如可以填入 `fec=20:3`。<br>注意：D + R 的总数最大值为 255，不能超过这个数。<br>冒号两侧任意一个值为 0 表示不使用该选项。两端的设置必须相同。<br>详情请参考 [FEC使用介绍](docs/fec_zh-hans.md)|
| mtu  | 正整数 |否|当前网络 MTU 数值，用以自动计算 kcp_mtu|
| kcp_mtu  | 正整数 |否|预设值1440。调用 ikcp_setmtu() 设置的值，亦即 UDP 数据包内数据内容的长度|
| kcp  | manual<br>fast1 - 6<br>regular1 - 5<br> &nbsp; |是|手动设置<br>快速<br>常速<br>(末尾数字：数值越小，速度越快)|
| kcp_sndwnd  | 正整数 |否|预设值见下表，可以单独覆盖|
| kcp_rcvwnd  | 正整数 |否|预设值见下表，可以单独覆盖|
| kcp_nodelay  | 正整数 |视情况|kcp=manual 时必填，预设值见下表|
| kcp_interval  | 正整数 |视情况|kcp=manual 时必填，预设值见下表|
| kcp_resend  | 正整数 |视情况|kcp=manual 时必填，预设值见下表|
| kcp_nc  | yes<br>true<br>1<br>no<br>false<br>0 |视情况|kcp=manual 时必填，预设值见下表|
| outbound_bandwidth | 正整数 |否|出站带宽，用于通讯过程中动态更新 kcp_sndwnd 的值|
| inbound_bandwidth | 正整数 |否|入站带宽，用于通讯过程中动态更新 kcp_rcvwnd 的值|
| ipv4_only | yes<br>true<br>1<br>no<br>false<br>0 |否|若系统禁用了 IPv6，须启用该选项并设为 yes 或 true 或 1|
| ipv6_only | yes<br>true<br>1<br>no<br>false<br>0 |否|忽略 IPv4 地址|
| blast | yes<br>true<br>1<br>no<br>false<br>0 |否|尝试忽略 KCP 流控设置，尽可能迅速地转发数据包。可能会导致负载过大|
| \[listener\] | N/A |是<br>(仅限中继模式)|中继模式的标签，用于指定监听模式的 KCP 设置<br>该标签表示与客户端交互数据|
| \[forwarder\] | N/A  |是<br>(仅限中继模式)|中继模式的标签，用于指定转运模式的 KCP 设置<br>该标签表示与服务端交互数据|
| \[custom_input\] | N/A  |否|自定义映射模式的标签，使用方法请参考 [自定义映射使用方法](docs/custom_ip_mappings_zh-hans.md)|
| \[custom_input_tcp\] | N/A  |否|自定义映射模式的标签，使用方法请参考 [自定义映射使用方法](docs/custom_ip_mappings_zh-hans.md)|
| \[custom_input_udp\] | N/A  |否|自定义映射模式的标签，使用方法请参考 [自定义映射使用方法](docs/custom_ip_mappings_zh-hans.md)|

其中，`encryption_algorithm` 以及 `encryption_password` 在通讯的两端必须保持一致。

#### FreeBSD only
|  名称   | 可设置值  | 必填 |备注|
|  ----  | ----  | :----: | ---- |
| fib_ingress  | 0 - 65535 |否|入站连接使用的FIB|
| fib_egress | 0 - 65535 |否|出站连接使用的FIB|

#### outbound_bandwidth 与 inbound_bandwidth
可用后缀：K / M / G

后缀区分大小写，大写按二进制 (1024) 计算，小写按十进制 (1000) 计算。

- 填入 1000，表示带宽为 1000 bps

- 填入 100k，表示带宽为 100 kbps (100000 bps)

- 填入 100K，表示带宽为 100 Kbps (102400 bps)

- 填入 100M，表示带宽为 100 Mbps (102400 Kbps)

- 填入 1G，表示带宽为 1 Gbps (1024 Mbps)

注意，是 bps (Bits Per Second)，不是 Bps (Bytes Per Second)。

需要提醒的是，填写的带宽值不应超出实际带宽，以免造成发送窗口拥堵导致阻塞。

**重要提示**：<br>
KCPTube 会在 KCP 链路建立后的 5 秒左右，根据握手包的延迟值以及 outbound_bandwidth 与 inbound_bandwidth 的数值，计算并设置 KCP 的发送窗口大小。设置完成后的一段时间内，有很大机率出现流量大幅度波动的情况，甚至会出现流量突然降至 0，需要好几秒才能恢复。

#### KCP 模式预设值
| 快速模式      | kcp_sndwnd | kcp_rcvwnd|kcp_nodelay|kcp_interval|kcp_resend|kcp_nc |
|  ----        | :----:     | :----:    | :----:    | :----:     | :----:   |:----: |
| fast1        | 2048       |   2048    |      1    |   1        |   2      |   1   |
| fast2        | 2048       |   2048    |      2    |   1        |   2      |   1   |
| fast3        | 2048       |   2048    |      1    |   1        |   3      |   1   |
| fast4        | 2048       |   2048    |      2    |   1        |   3      |   1   |
| fast5        | 2048       |   2048    |      1    |   1        |   4      |   1   |
| fast6        | 2048       |   2048    |      2    |   1        |   4      |   1   |

| 常速模式      | kcp_sndwnd | kcp_rcvwnd|kcp_nodelay|kcp_interval|kcp_resend|kcp_nc |
|  ----        | :----:     | :----:    | :----:    | :----:     | :----:   |:----: |
| regular1     | 1024       |   1024    |      1    |   1        |   5      |   1   |
| regular2     | 1024       |   1024    |      2    |   1        |   5      |   1   |
| regular3     | 1024       |   1024    |      0    |   1        |   2      |   1   |
| regular4     | 1024       |   1024    |      0    |   15       |   2      |   1   |
| regular5     | 1024       |   1024    |      0    |   30       |   2      |   1   |

其中，丢包率越高（高于 10%），kcp_nodelay=1 就比 kcp_nodelay=2 越有优势。在丢包率不特别高的情况下，kcp_nodelay=2 可使延迟抖动更为平滑。

### 大流量传输
对于低丢包环境，每个模式都适合使用，区别只在于浪费的流量是多还是少，以及最高速的上限有所不同。其中 regular3 浪费的流量没那么多。<br />
建议同时开启 `blast=1` 设置。

对于高丢包环境，请考虑叠加使用 FEC 设置。详情请参考 [FEC使用介绍](docs/fec_zh-hans.md)

更多详解，请见[参数列表](docs/parameters_zh-hans.md)。

### Log 文件
在首次获取打洞后的 IP 地址与端口后，以及打洞的 IP 地址与端口发生变化后，会向 Log 目录创建 ip_address.txt 文件（若存在就覆盖），将 IP 地址与端口写进去。

获取到的打洞地址会同时显示在控制台当中。

`log_path=` 必须指向目录，不能指向文件本身。

如果不需要写入 Log 文件，那就删除 `log_path` 这一行。

### STUN Servers
从[NatTypeTeste](https://github.com/HMBSbige/NatTypeTester)找到的普通 STUN 服务器：
- stun.syncthing.net
- stun.qq.com
- stun.miwifi.com
- stun.bige0.com
- stun.stunprotocol.org

从[Natter](https://github.com/MikeWang000000/Natter)找到的STUN 服务器：

- fwa.lifesizecloud.com
- stun.isp.net.au
- stun.freeswitch.org
- stun.voip.blackberry.com
- stun.nextcloud.com
- stun.stunprotocol.org
- stun.sipnet.com
- stun.radiojar.com
- stun.sonetel.com
- stun.voipgate.com

其它 STUN 服务器：[public-stun-list.txt](https://gist.github.com/mondain/b0ec1cf5f60ae726202e)

---

## 预编译二进制
为了方便使用，目前已经提供了多个平台的二进制可执行文件：
- Windows
- FreeBSD
- Linux

预编译的二进制文件全部都是静态编译。Linux 版本基本上都是静态编译，但 libc 除外，因此准备了两个版本，一个用于 glibc (2.31)，另一个用于 musl。

### Docker 镜像

对于 Linux 环境，另有提供 Docker 镜像（目前仅限 x64），下载 kcptube_docker_image.zip 并解压，再使用 `docker load -i kcptube_docker.tar` 导入。

导入后，使用方式为：
```
docker run -v /path/to/config_file.conf:/config_file.conf kcptube config_file.conf
```

例如：
```
docker run -v /home/someone/config1.conf:/config1.conf kcptube config1.conf
```

---

## 建立服务
### FreeBSD

FreeBSD 用户可将下载好的二进制文件复制到 `/usr/local/bin/`，然后运行命令
```
chmod +x /usr/local/bin/kcptube
```

本项目的 `service` 目录已经准备好相应服务文件。

1. 找到 kcptube 文件，复制到 `/usr/local/etc/rc.d/`
2. 运行命令 `chmod +x /usr/local/etc/rc.d/kcptube`
3. 把配置文件复制到 `/usr/local/etc/kcptube/`
    - 记得把配置文件命名为 `config.conf`
        - 完整的路径名：`/usr/local/etc/kcptube/config.conf`
4. 在 `/etc/rc.conf` 加一行 `kcptube_enable="YES"`

最后，运行 `service kcptube start` 即可启动服务

---

## 编译
编译器须支持 C++20

依赖库：

- [asio](https://github.com/chriskohlhoff/asio) ≥ 1.18.2
- [botan3](https://github.com/randombit/botan)

### Windows
请事先使用 vcpkg 安装依赖包 `asio` 与 `botan`，命令如下：

```
vcpkg install asio:x64-windows asio:x64-windows-static
vcpkg install botan:x64-windows botan:x64-windows-static
```
（如果需要 ARM 或者 32 位 x86 版本，请自行调整选项）

然后用 Visual Studio 打开 `sln\kcptube.sln` 自行编译

### FreeBSD
同样，请先安装依赖项 asio 以及 botan3，另外还需要 cmake，用系统自带 pkg 即可安装：

```
pkg install asio botan3 cmake
```
接着在 build 目录当中构建
```
mkdir build
cd build
cmake ..
make
```

### 其它 BSD
步骤与 FreeBSD 类似，NetBSD 请使用 [pkgin](https://www.netbsd.org/docs/pkgsrc/using.html) 安装依赖项与 cmake：
```
pkgin install asio
pkgin install cmake
```
OpenBSD 请使用 `pkg_add` 安装上述两个依赖性。DragonflyBSD 请使用 `pkg`，用法与 FreeBSD 相同。

由于 botan-3 仍未被这几个 BSD 系统收录，须自行编译 botan-3。

剩余的构建步骤请参考上述的 FreeBSD。

注意，由于这几个 BSD 自带的编译器版本较低，请事先额外安装高版本 GCC。

### Linux
步骤与 FreeBSD 类似，请用发行版自带的包管理器安装 asio 与 botan3 以及 cmake。

#### Alpine
````
apk add asio botan3-libs cmake
````
接着在 build 目录当中构建
```
mkdir build
cd build
cmake ..
make
```

#### 静态编译注意事项
有两种做法

- **做法1**

    按照正常流程编译好，删除刚刚生成的 kcptube 二进制文件，并运行命令
    ```
    make VERBOSE=1
    ```
    再从输出的内容提取出最后一条 C++ 链接命令，把中间的 `-lbotan-3` 改成 libbotan-3.a 的**完整路径**，例如 `/usr/lib/x86_64-linux-gnu/libbotan-3.a`。


- **做法2**

    打开 src/CMakeLists.txt，把 `target_link_libraries(${PROJECT_NAME} PRIVATE botan-3)` 改成 `target_link_libraries(${PROJECT_NAME} PRIVATE botan-3 -static)`

    然后即可正常编译。注意，如果系统使用 glibc 的话，这样会连同 glibc 一并静态编译，从而会跳出有关 getaddrinfo 的警告。

### macOS
我没苹果电脑，所有步骤请自行解决。

---

## 对 UDP 传输性能的改善
增加接收缓存可以改善 UDP 传输性能
### FreeBSD
可以使用命令 `sysctl kern.ipc.maxsockbuf` 查看缓存大小。如果需要调整，请运行命令（数字改为想要的数值）：
```
sysctl -w kern.ipc.maxsockbuf=33554434
```
或者在 `/etc/sysctl.conf` 写入 
```
kern.ipc.maxsockbuf=33554434
```
### NetBSD & OpenBSD
可以使用命令 `sysctl net.inet.udp.recvspace` 查看接收缓存大小。如果需要调整，请运行命令（数字改为想要的数值）：
```
sysctl -w net.inet.udp.recvspace=33554434
```
或者在 `/etc/sysctl.conf` 写入 
```
net.inet.udp.recvspace=33554434
```
若有必要，可以同时调整 `net.inet.udp.sendspace` 的数值。这是发送缓存的设置。
### Linux
对于接收缓存，可以使用命令 `sysctl net.core.rmem_max` 及 `sysctl net.core.rmem_default` 查看接收缓存大小。

如果需要调整，请运行命令（数字改为想要的数值）：
```
sysctl -w net.core.rmem_max=33554434
sysctl -w net.core.rmem_default=33554434
```
或者在 `/etc/sysctl.conf` 写入 
```
net.core.rmem_max=33554434
net.core.rmem_default=33554434
```
若有必要，可以同时调整 `net.core.wmem_max` 及 `net.core.wmem_default` 的数值。这是发送缓存的设置。

## IPv4 映射 IPv6
由于 kcptube 内部使用的是 IPv6 单栈 + 开启 IPv4 映射地址（IPv4-mapped IPv6）来同时使用 IPv4 与 IPv6 网络，因此请确保 v6only 选项的值为 0。

**正常情况下不需要任何额外设置，FreeBSD 与 Linux 以及 Windows 都默认允许 IPv4 地址映射到 IPv6。**

如果系统不支持 IPv6，或者禁用了 IPv6，请在配置文件中设置 ipv4_only=true，这样 kcptube 会退回到使用 IPv4 单栈模式。

## 其它注意事项

### NetBSD
使用命令
```
sysctl -w net.inet6.ip6.v6only=0
```
设置后，单栈+映射地址模式可以侦听双栈。

但由于未知的原因，可能无法主动连接 IPv4 映射地址。

### OpenBSD
因为 OpenBSD 彻底屏蔽了 IPv4 映射地址，所以在 OpenBSD 平台使用双栈的话，需要将配置文件保存成两个，其中一个启用 ipv4_only=1，然后在使用 kcptube 时同时载入两个配置文件。

### 多种系统都遇到的 Too Many Open Files
大多数情况下，这种提示只会在服务器端遇到，不会在客户端遇到。

如果确实在客户端遇到了，请检查 `mux_tunnels` 的数值是否过高（请顺便参考“多路复用 (mux_tunnels=N)”段落）。
#### GhostBSD
一般情况下，绝大多数 BSD 系统都不会遇到这种事，只有 2023 年下半年更新后的 GhostBSD 才会遇到这种现象。

这是因为 GhostBSD 在 `/etc/sysctl.conf` 当中加了这一行：
```
kern.maxfiles=100000
```
这一行缩减了上限，远低于原版 FreeBSD 的对应数值。

解决办法很简单，删掉这一行即可。注释掉也可以。<br />
还可以使用命令 `sysctl kern.maxfiles=300000` 临时修改上限值。

#### Linux
由于 Linux 系统的 Open Files 数量限制为 1024，所以很容易会遇到这种问题。

临时解决办法：
1. 运行命令 `ulimit -n`，查看输出的数值
2. 如果数值确实只有 1024，请运行命令 `ulimit -n 300000`

永久解决办法：<br />
编辑 /etc/security/limits.conf，在末尾加上

```
*         hard    nofile       300000
*         soft    nofile       300000
root      hard    nofile       300000
root      soft    nofile       300000
```

## 加密与数据校验
由于需要传送 TCP 数据，因此数据校验是不可忽略的，正如 TCP 本身那样。

无论是否加密，kcptube 都会将 MTU 缩小 2 个字节，尾附 2 字节的数据。

如果已经使用了加密选项，那么尾附的 2 字节数据就是临时生成的IV。

如果选择不使用加密功能，那么尾附的 2 字节数据就是校验码，是由 CRC32 高低位异或而成。

需要提醒的是，使用校验码仍然无法 100% 避免内容错误，TCP 本身也是一样。如果确实需要精确无误，请启用加密选项。

## 多路复用 (mux_tunnels=N)
KCP Tube 虽然有“多路复用”的功能，但默认并不主动打开。在不使用该功能的情况下，每接受一个入站连接，就会创建一个对应的出站连接。

原因是为了躲避运营商的 QoS。多路复用状态下，一旦某个端口号被 QoS，就会导致共用端口号的其它会话同时受阻，直到更换端口号为止。

连接之间相互独立，即使某个端口号被 QoS，受影响的仅仅只是这一路会话，不影响其他会话。

除非被承载的程序会产生大量独立连接。在这种情况下，KCP Tube 会创建大量 KCP 通道，在通讯过程中会消耗较多的CPU资源。

如果确实要用“多路复用”功能，可以参考以下分类：

- 适合使用“多路复用”的场景：
    - 代理转发程序，例如 Shadowsocks

- 不必使用“多路复用”的场景：
    - VPN，例如
        - OpenVPN
        - Wireguard

启用“多路复用”后，KCPTube 会预创建 N 条链路，所有入站新连接都会从已有链路中传送数据，而不再单独创建新链路。此时 KCP 通道的超时时间为 30 秒。

一般来说，`mux_tunnels 设置成 3 ~ 10 就够用了，不需要设置过高的数值。

## 关于代码
### TCP
为了降低延迟，kcptube 启用了 TCP_NODELAY 选项。对于某些大流量应用场景，可能会造成 TCP 数据传输量减少。

### KCP
在原版 [KCP](https://github.com/skywind3000/kcp) 的基础上，作了以下修改：

1. 原版的“已发送数据包缓存”采用的是队列，修改后的版本改成使用 std::map，共三个映射表：总队列，按包号排序；两个待重发队列，一个按时间排序，另一个按丢包跳跃次数排序。
2. 原版的 `flush()` 是先把待发送数据转移到发送队列后，在同一个循环重做完“发送新数据包”、“数据包重发”、“ACK包发送”三件事。修改后的版本变为先做“数据包重发”、“ACK包发送”，然后再做“待发送数据转移到发送队列”，在转移期间顺便发送。
3. 原版的 `check()` 每次都会重新遍历一遍发送队列，查找已到点的重传时间戳。修改后的版本变为：从已经排好序的映射表读取第一个时间戳，免除查找步骤。

除此之外，原版的存在“bug”，kcptube 也会有。例如：

* [如何避免缓存积累延迟的问题](https://github.com/skywind3000/kcp/issues/175)
* [求助：一些压测出现的问题， 发大包后不断累积](https://github.com/skywind3000/kcp/issues/243)

于是 kcptube 设置了较为明显的暂停方案。对于 TCP 数据，在达到接收限制时（队列满额），会暂停接收 TCP 数据，直到有空位再恢复；对于 UDP 数据，在达到接收限制时就直接丢包。

这个限制对于传输量不大的应用场景基本上不会造成影响。

### 线程池
kcptube 使用的线程池来自于 [BS::thread_pool](https://github.com/bshoshany/thread-pool)，另外再做了些许修改，用于多连接时的并行加解密处理。

### 版面
代码写得很随意，想到哪写到哪，因此版面混乱。准确来说，是十分混乱。

其中有一些代码行长得像竹竿，主要是写的时候为了顺着思路所以懒得换行。毕竟我又不用 vim / emacs。我用 IDE 时，IDE 的代码区设置的文字大小不同于其他区域的文字大小，甚至字体都不一样，帮我缓解了混乱问题。

至于阅读者的感受嘛…… 那肯定会不爽。不关我事，不管了。