# KCP Tube Parameters

|  Name   | Value  | Require |Note|
|  ----  | ----  | :----: | ---- |
| mode  | client<br>server<br>relay |Yes|Client Mode<br>Server Mode<br>Relay Node Mode|
| listen_on | domain name or IP address |No|domain name / IP address only. Multiple addresses should be comma-separated.|
| listen_port | 1 - 65535 |Yes|Port ranges can be specified when running as a server mode|
| destination_port | 1 - 65535 |Yes|Port ranges can be specified when running as a client mode|
| destination_address  | IP address, domain name |Yes|Brackets are not required when filling in an IPv6 address. Multiple addresses should be comma-separated.|
| dport_refresh  | 20 - 65535 |No|The unit is ‘second’. Not writting this option means using the default value of 60 seconds. <br>1 to 20 is treated as 20 seconds; greater than 32767 is treated as 32767 seconds. <br>Set to 0 means disable this option.|
| encryption_algorithm | XOR<br>AES-GCM<br>AES-OCB<br>chacha20<br>xchacha20<br>none |No    |XOR Only<br>AES-256-GCM-AEAD<br>AES-256-OCB-AEAD<br>ChaCha20-Poly1305<br>XChaCha20-Poly1305<br>No Encryption |
| encryption_password  | Any character |Depends…|…on the setting of encryption_algorithm, if the value is set and it is neither `none` nor `XOR`, it is required|
| udp_timeout  | 0 - 65535 |No|The unit is ‘second’. The default value is 180 seconds, set to 0 to use the default value<br>This option represents the timeout setting between UDP application ↔ kcptube|
| keep_alive  | 0 - 65535 |No | The unit is ‘second’. The default value is 0, which means that Keep Alive is disabled. This option refers to Keep Alive between two KCP endpoints.<br>Can be enabled on any side. If no response is received after 30 seconds, the channel will be closed.|
| mux_tunnels  | 0 - 65535 |No | The default value is 0, which means that multiplexing is disabled. This option means how many multiplexing tunnels between two KCP endpoints.<br>Client Mode only.|
| stun_server  | STUN Server's address |No| Cannot be used if listen_port option is port range mode|
| log_path  | The directory where the Logs are stored |No|Cannot point to the file itself|
| fec  | uint8:uint8 |No|The format is `fec=D:R, for example `fec=20:4`. <br>Note: The maximum total value of D + R is 255 and cannot exceed this number.<br>A value of 0 on either side of the colon indicates that the option is not used. Must be the same value on both side.<br>Please refer to [The Usage of FEC](fec_en.md)|
| mtu  | Positive Integer |No|MTU Value of current network, is to automatically calculate the value of `kcp_mtu`|
| kcp_mtu  | Positive Integer |No|This option refers to the length of the data content within a UDP packet. <br>The value set for this option refers to the value set by calling ikcp_setmtu(). <br>Default value is 1440.|
| kcp  | manual<br>fast1 - 6<br>regular1 - 5<br> &nbsp; |Yes|Setup Manually<br>Fast Modes<br>Regular Speeds<br>(the number at the end: the smaller the value, the faster the speed)|
| kcp_sndwnd  | Positive Integer |No|See the table below for default values, which can be overridden individually|
| kcp_rcvwnd  | Positive Integer |No|See the table below for default values, which can be overridden individually|
| kcp_nodelay  | Positive Integer |Depends…|…on the setting of ‘kcp=’, if if the value is set as ‘kcp==manual’, this option is required. See the table below for default values.|
| kcp_interval  | Positive Integer |Depends…|…on the setting of ‘kcp=’, if if the value is set as ‘kcp==manual’, this option is required. See the table below for default values.|
| kcp_resend  | Positive Integer |Depends…|…on the setting of ‘kcp=’, if if the value is set as ‘kcp==manual’, this option is required. See the table below for default values.|
| kcp_nc  | yes<br>true<br>1<br>no<br>false<br>0 |Depends…|…on the setting of ‘kcp=’, if if the value is set as ‘kcp==manual’, this option is required. See the table below for default values.|
| outbound_bandwidth | Positive Integer |No|Outbound bandwidth, used to dynamically update the value of kcp_sndwnd during communication|
| inbound_bandwidth | Positive Integer |No|Inbound bandwidth, used to dynamically update the value of kcp_rcvwnd during communication|
| ipv4_only | yes<br>true<br>1<br>no<br>false<br>0 |No|If the system disables IPv6, this option must be enabled and set to yes or true or 1|
| ipv6_only | yes<br>true<br>1<br>no<br>false<br>0 |No|Ignore IPv4 address|
| blast | yes<br>true<br>1<br>no<br>false<br>0 |No|Packets are forwarded as quickly as possible regardless of KCP flow control settings. May lead to overload.|
| \[listener\] | N/A |Yes<br>(Relay Mode only)|Section Name of Relay Mode, KCP settings for specifying the listening mode<br>This tag represents data exchanged with the client|
| \[forwarder\] | N/A  |Yes<br>(Relay Mode only)|Section Name of Relay Mode, KCP settings for specifying the forwarding mode<br>This tag represents data exchanged with the server|
| \[custom_input\] | N/A  |No| Section Name of Custom-IP-Mapping Mode, please refer to [The Usage of Custom IP Mappings](custom_ip_mappings_en.md)|
| \[custom_input_tcp\] | N/A  |No| Section Name of Custom-IP-Mapping Mode, please refer to [The Usage of Custom IP Mappings](custom_ip_mappings_en.md)|
| \[custom_input_udp\] | N/A  |No| Section Name of Custom-IP-Mapping Mode, please refer to [The Usage of Custom IP Mappings](custom_ip_mappings_en.md)|

Note: `encryption_algorithm` and `encryption_password` must be consistent at both ends of the communication.

## FreeBSD only
|  Name   | Value  | Require |Note|
|  ----  | ----  | :----: | ---- |
| fib_ingress  | 0 - 65535 |No|FIB for ingress connections|
| fib_egress | 0 - 65535 |No|FIB for egress connections|

## outbound_bandwidth and inbound_bandwidth
Available suffixes: K / M / G

The suffix is case-sensitive, uppercase is calculated as binary (1024), lowercase is calculated as decimal (1000).

- Entering 1000 represents a bandwidth of 1000 bps

- Entering 100k represents a bandwidth of 100 kbps (100000 bps)

- Entering 100K represents a bandwidth of 100 Kbps (102400 bps)

- Entering 100M represents a bandwidth of 100 Mbps (102400 Kbps)

- Entering 1G represents a bandwidth of 1 Gbps (1024 Mbps)

Please note that it is bps (Bits Per Second), not Bps (Bytes Per Second).

This bandwidth values should not larger than your actual bandwidth, otherwise this will cause the sending window to be congested and cause blocking.

**Important Notice**:<br>
KCPTube will calculate and set the KCP sending window size based on the delay value of the handshake packet and the values of outbound_bandwidth and inbound_bandwidth about 5 seconds after the KCP link is established. Within a period of time after the setup is completed, there is a high chance that the traffic will fluctuate significantly, or even the traffic may suddenly drop to 0, and it will take several seconds to recover.

## KCP Mode Default Values
| Fast Mode    | kcp_sndwnd | kcp_rcvwnd|kcp_nodelay|kcp_interval|kcp_resend|kcp_nc |
|  ----        | :----:     | :----:    | :----:    | :----:     | :----:   |:----: |
| fast1        | 2048       |   2048    |      1    |   1        |   2      |   1   |
| fast2        | 2048       |   2048    |      2    |   1        |   2      |   1   |
| fast3        | 2048       |   2048    |      1    |   1        |   3      |   1   |
| fast4        | 2048       |   2048    |      2    |   1        |   3      |   1   |
| fast5        | 2048       |   2048    |      1    |   1        |   4      |   1   |
| fast6        | 2048       |   2048    |      2    |   1        |   4      |   1   |

| Regular Mode | kcp_sndwnd | kcp_rcvwnd|kcp_nodelay|kcp_interval|kcp_resend|kcp_nc |
|  ----        | :----:     | :----:    | :----:    | :----:     | :----:   |:----: |
| regular1     | 1024       |   1024    |      1    |   1        |   5      |   1   |
| regular2     | 1024       |   1024    |      2    |   1        |   5      |   1   |
| regular3     | 1024       |   1024    |      0    |   1        |   2      |   1   |
| regular4     | 1024       |   1024    |      0    |   15       |   2      |   1   |
| regular5     | 1024       |   1024    |      0    |   30       |   2      |   1   |

Note: If the packet loss rate is high enough (higner than 10%), kcp_nodelay=1 may better than kcp_nodelay=2. If the packet loss rate is not too high, kcp_nodelay=2 can make the network latency jitter smoother.

### High-traffic transmission
For low packet loss environments, each mode is suitable for use. The difference lies only in the amount of wasted traffic and the slightly different upper limit of the highest speed.<br />Among them, regular3 wastes less traffic.<br />It is recommended to enable the `blast=1` setting at the same time.

For high packet loss environments, consider using FEC settings at the same time. For more details, please refer to [The Usage of FEC](fec_en.md).

#### Pattern Interpretation (Simplified Version)
First, the conclusion: the lower the number of the pattern, the faster the response speed. The fast mode is slightly different, please continue reading.

`kcp_sndwnd` refers to the size of the sending buffer, and `kcp_rcvwnd` refers to the size of the receiving buffer. These two factors will affect the transmission rate.

`kcp_nodelay` refers to the `nodelay` variable of KCP, which is used to select the growth rate of the waiting time for timeout retransmission. Originally, there were only two options, 0 and 1, where 0 means not using KCP's own fast retransmission and 1 means enabling it. 

- When the value is 0, the new time is simply multiplied by 2 based on the previous time.<br /> 
- When the value is 1, the new waiting time is only 1.5 times the previous time, not 2 times.<br /> 
- In May 2023, the original author of KCP added a value of 2, similar to value 1. The difference is that value 1 uses the ‘previous time’ calculated by the current packet itself, while value 2 uses the average delay time calculated separately within KCP.<br />
That's why this page was mentioned earlier that ‘kcp_nodelay=2 can make the network latency jitter smoother.’

`kcp_interval` refers to the `internal` variable, the update interval of KCP.

`kcp_resend` refers to the value of the `fastresend` variable within KCP. If the value is 0, it means that the fast retransmission function is turned off. If the value is not 0, it means that after crossing the specified number of times, it will no longer wait and will directly transmit again.

`kcp_nc` refers to the last parameter `nc` of `ikcp_nodelay()`, where 0 means not to close the flow control and 1 means to close the flow control. It should be set to 1, otherwise the transmission speed will be very slow.

## Encryption and Data verification
Since TCP data transmission is required, data verification cannot be ignored, just like TCP itself.

Regardless of whether encryption is enabled or not, this program will reduce the MTU by 2 bytes and append 2-byte data at the end.

If the encryption option is used, then the 2-byte data appended at the end will be a temporarily generated IV.

If the encryption feature is not selected, the 2-byte data appended at the end will be the checksum, consisting of two different 8-bit checksums:

- Longitudinal Redundancy Check (LRC)
- 8-bit checksum

This is because the Botan library used in this program does not come with a 16-bit checksum algorithm. Therefore, this program simultaneously utilizes these two 8-bit checksums.

The calculation speed of these two checksums is fast enough, concise and practical, and is not an obscure calculation method. For example, Modbus uses LRC.

It should be reminded that using two checksums still cannot completely avoid content errors, just like TCP itself. If you really need accuracy, please enable the encryption option.

## Multiplexing (mux_tunnels=N)
The function of multiplexing is not automatically enabled by default. Without using this feature, for each incoming connection accepted, a corresponding outgoing connection is created.

The reason is to avoid the QoS of operators. Once a port number is affected by QoS in multiplexing mode, other sessions sharing the same port number will also be blocked until the port number is changed.

The connections are independent of each other. Even if a port number is affected by QoS, only this session will be affected, not other sessions.

Unless the carried program generates many independent connections. In this case, KCP Tube will create many KCP channels and consume more CPU resources during communication.

If you really need to use the ‘multiplexing’ function, you can refer to the following classifications:

- Scenarios suitable for using multiplexing:
    - Proxy forwarding programs, such as Shadowsocks

- Scenarios that do not require using multiplexing:
    - VPN, such as
        - OpenVPN
        - Wireguard

When "Multiplexing" is enabled, KCPTube will pre-create N links, and all inbound new connections will transmit data from the existing links instead of creating new links separately. At this time, the KCP channel timeout is 30 seconds.

In most cases, setting `mux_tunnels` to 3 - 10 is enough, and there is no need to set an excessively high value.

# Log File
After obtaining the IP address and port after NAT hole punching for the first time, and after the IP address and port of NAT hole punching change, an ip_address.txt file will be created in the Log directory (overwrite if it exists), and the IP address and port will be written in.

The obtained NAT hole punching address will be displayed on the console at the same time.

# STUN Servers
The STUN servers obtained from [NatTypeTeste](https://github.com/HMBSbige/NatTypeTester):
- stun.syncthing.net
- stun.qq.com
- stun.miwifi.com
- stun.bige0.com
- stun.stunprotocol.org

The STUN servers obtained from [Natter](https://github.com/MikeWang000000/Natter):

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

Other STUN Servers: [public-stun-list.txt](https://gist.github.com/mondain/b0ec1cf5f60ae726202e)
