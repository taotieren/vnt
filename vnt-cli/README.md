## 模块介绍
体积小，可以在服务器、路由器等环境使用
## 详细参数说明
### -k `<token>`
一个虚拟局域网的标识，在同一服务器下，相同token的设备会组建一个局域网
### -n `<name>`
设备名称，方便区分不同设备
### -d `<id>`
设备id，每台设备的唯一标识，注意不要重复
### -c
关闭控制台交互式命令，后台运行时可以加此参数
### -s `<server>`
注册和中继服务器地址，注册和转发数据
### -e `<stun-server>`
使用stun服务探测客户端NAT类型，不同类型有不同的打洞策略
### -a
加了此参数表示使用tap网卡，默认使用tun网卡，tun网卡效率更高
### -i `<in-ip>`、-o  `<out-ip>`

配置点对网(IP代理)时使用，例如A(虚拟ip:10.26.0.2)通过B(虚拟ip:10.26.0.3,本地出口ip:192.168.0.10)访问C(目标网段192.168.0.0/24)，

则在A配置 **'-i 192.168.0.0/24,10.26.0.3'** ,表示将192.168.0.0/24网段的数据都转发到10.26.0.3节点

在B配置 **'-o 192.168.0.0/24'**  ,表示允许将数据转发到 192.168.0.0/24 ,允许转发所有网段可以使用 **'-o 0.0.0.0/0'**

-i和-o参数均可使用多次，来指定不同网段，例如 **'-o 192.168.1.0/24 -o 192.168.2.0/24'** 表示允许转发目标为192.168.1.0/24或192.168.2.0/24这两个网段的数据

### -w `<password>`

提升通信安全性，使用该密码生成的密钥对客户端数据进行加密，并且服务端无法解密(包括中继数据)。使用相同密码的客户端才能通信

| 密码位数  | 加密算法       |  
|-------|------------| 
| 小于8   | AES128-GCM |
| 大于等于8 | AES256-GCM | 

### -W
开启和服务端通信的数据加密，采用rsa+aes256gcm加密客户端和服务端之间通信的数据，可以避免token泄漏、中间人攻击

注意：
1. -w `<password>`是用于客户端-客户端之间的加密，password不会传递到服务端，只添加这个参数不会加密客户端-服务端通信的数据
2. -W 用于开启客户端-服务端之间的加密

### -m
模拟组播，高频使用组播通信时，可以尝试开启此参数，默认情况下会把组播当作广播发给所有节点

默认情况(组播当广播发送)：稳定性好，使用组播频率低时更省流量

模拟组播：高频使用组播时防止广播泛洪，客户端和中继服务器会维护组播成员等信息，注意使用此选项时，虚拟网内所有成员都需要开启此选项

### -u `<mtu>`

设置虚拟网卡的mtu值，大多数情况下使用默认值效率会更高，也可根据实际情况微调这个值，不加密默认为1450，加密默认为1410

###  --tcp
和服务端使用tcp通信。有些网络提供商对UDP限制比较大，这个时候可以选择使用TCP模式，提高稳定性。一般来说udp延迟和消耗更低
### --ip `<IP>`
指定虚拟ip,指定的ip不能和其他设备重复,必须有效并且在服务端所属网段下,默认情况由服务端分配
### --par `<parallel>`
任务并行度(必须为正整数),默认值为1,该值表示处理网卡读写的任务数,组网设备数较多、处理延迟较大时可适当调大此值
### --model `<model>`
加密模式，可选值 aes_gcm/aes_cbc/aes_ecb/sm4_cbc，默认使用aes_gcm，通常情况aes_gcm安全性高、aes_ecb性能更好，但是在低性能设备上sm4_cbc也许速度会更快；


| 密码位数  | model   | 加密算法       |  
|-------|---------|------------|
| 1~8位  | aes_gcm | AES128-GCM |
| `>=`8 | aes_gcm | AES256-GCM |
| 1~8位  | aes_cbc | AES128-CBC |
| `>=`8 | aes_cbc | AES256-CBC |
| 1~8位  | aes_ecb | AES128-ECB |
| `>=`8 | aes_ecb | AES256-ECB |
| `>0`  | sm4_cbc | SM4-CBC    |
### --finger 

开启数据指纹校验，可增加安全性，如果服务端开启指纹校验，则客户端也必须开启，开启会损耗一部分性能

注意：默认情况下服务端不会对中转的数据做校验，如果要对中转的数据做校验，则需要客户端、服务端都开启此参数
### --punch `<punch>`
取值ipv4/ipv6，选择只使用ipv4打洞或者只使用ipv6打洞，默认两则都会使用
### --port `<port>`
取值0~65535，指定本地监听的端口，默认取随机端口
### --cmd
开启交互式命令，开启后可以直接在窗口下输入命令，如需后台运行请勿开启
### --no-proxy
关闭内置的ip代理，内置的代理较为简单，而且一般来说直接使用网卡NAT转发性能会更高，
有需要可以自行配置NAT转发，[可参考‘编译’小节中的NAT配置](https://github.com/lbl8603/vnt#%E7%BC%96%E8%AF%91)
### -f `<conf>`
指定配置文件
配置文件采用yaml格式，可参考：
```yaml
# 全部参数
tap: false #是否使用tap 
token: xxx #组网token
device_id: xxx #当前设备id
name: windows 11 #当前设备名称
server_address: ip:port #注册和中继服务器
stun_server:  #stun服务器
  - stun1.l.google.com:19302
  - stun2.l.google.com:19302
in_ips: #代理ip入站
  - 192.168.1.0/24,10.26.0.3
out_ips: #代理ip出站
  - 0.0.0.0/0
password: xxx #密码
simulate_multicast: false #模拟组播
mtu: 1420  #mtu
tcp: false #tcp模式
ip: 10.26.0.2 #指定虚拟ip
relay: false #中继模式
server_encrypt: true #服务端加密
parallel: 1 #任务并行度
cipher_model: aes_gcm #客户端加密算法
finger: false #关闭数据指纹
punch_model: ipv4 #打洞模式 
port: 0 #使用随机端口
cmd: false #关闭控制台输入
no_proxy: false #是否关闭内置代理，true为关闭
```

或者需要哪个配置就加哪个，当然token是必须的
```yaml
# 部分参数
token: xxx #组网token
```
### --relay
禁用p2p,在网络环境很差时，只使用服务器中转效果可能更好（可以配合--tcp参数一起使用）
### --list
在后台运行时,查看其他设备列表
### --all
在后台运行时,查看其他设备完整信息
### --info
在后台运行时,查看当前设备信息
### --route 
在后台运行时,查看数据转发路径
### --stop
停止后台运行
