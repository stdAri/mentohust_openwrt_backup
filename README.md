# mentohust_openwrt_backup  

备份自己刷路由器的过程及相关软件。

## 文件说明  

|-firmware wndr4300v1 router firmware   
|-mentohust-ar71xx  mentohust ipk for Target ar71xx  
|-minieap  minieap ipk for Target ar71xx  

>**备份两个连接**   
>[【openwrt/LEDE】mentohust--校园网锐捷认证(minieap测试) ](https://www.right.com.cn/forum/forum.php?mod=viewthread&tid=196317&extra=page%3D1&page=1)   
>[【4.25更新】MiniEAP(多架构支持)——MentoHUST替代插件，厦门理工锐捷认证工具](https://www.right.com.cn/forum/thread-4106567-1-1.html)  

## 刷机教程

准备工作：打开Windows功能中的tftp功能。

![tftp](./tftp.png)

首先，电脑打开cmd 输入`ping 192.168.1.1 -t`

1. 电脑改为固定IP（*`192.168.1.2/255.255.255.0`*）
2. 把电脑拉出来的线接到路由lan1，（*路由器的剩下的lan口的网线最好都拔掉*）
3. 断电，就是按掉路由器的那个电源开关
4. 用尖针按住路由后面的复位键，千万不要松手
5. 通电。**等待电源灯 从 黄灯→变为绿灯，并且绿灯一直在闪烁**。然后松开复位键。
6. 打开powershell 输入`tftp -i 192.168.1.1 PUT img_name.img`
7. 等待3到5分钟。
8. 注意看cmd中界面192.168.1.1-t 是否一直能ping通。 持续10秒以上
9. 最后一步很重要，断电（不然的话会没有5G信号，某位大神说的）。步骤如下：
   1. 直接拔掉电源插座，等待5秒左右。（先拔掉电源插座，然后关掉路由器上的开关）
   2. 插上电源插座，等待5秒左右。打开路由器上的开关。


刷机完毕了。把猫的线插到wan口。

web网页在进入192.168.1.1，默认 账号：root 默认密码：无。可以自己设置。

电脑改为自动获取ip。

**至此，路由器已经成功安装openwrt固件**

## 安装mentohust

利用winscp或者其他软件将`mentohust-ar71xx`文件夹中的四个ipk传至路由器`/root/`路径（也可自己选择其他路径）

``` bash
opkg update
opkg install libpcap_1.8.1-1_mips_24kc.ipk
opkg install luci-app-mentohust_2.1-1_all.ipk
opkg install luci-i18n-mentohust-zh-cn_2.1-1_all.ipk
opkg install mentohust_0.3.1-2_mips_24kc.ipk
```

然后就可以使用网页或者命令行，通过mentohust进行锐捷认证链接校园网了。

## mentohust认证

```bash
mentohust -h
用法:   mentohust [-选项][参数]
选项:   -h 显示本帮助信息
        -k -k(退出程序) 其他(重启程序)
        -w 保存参数到配置文件
        -u 用户名
        -p 密码
        -n 网卡名
        -i IP[默认本机IP]
        -m 子网掩码[默认本机掩码]
        -g 网关[默认0.0.0.0]
        -s DNS[默认0.0.0.0]
        -o Ping主机[默认0.0.0.0，表示关闭该功能]
        -t 认证超时(秒)[默认8]
        -e 响应间隔(秒)[默认30]
        -r 失败等待(秒)[默认15]
        -l 允许失败次数[默认0，表示无限制]
        -a 组播地址: 0(标准) 1(锐捷) 2(赛尔) [默认0]
        -d DHCP方式: 0(不使用) 1(二次认证) 2(认证后) 3(认证前) [默认0]
        -b 是否后台运行: 0(否) 1(是，关闭输出) 2(是，保留输出) 3(是，输出到文件) [默认0]
        -v 客户端版本号[默认0.00表示兼容xrgsu]
        -f 自定义数据文件[默认不使用]
        -c DHCP脚本[默认udhcpc -i]
例如:   mentohust -uusername -ppassword -neth0 -i192.168.0.1 -m255.255.255.0 -g0.0.0.0 -s0.0.0.0 -o0.0.0.0 -t8 -e30 -r15 -a0 -d1 -b0 -v4.10 -fdefault.mpf -cudhcpc -i
注意：使用时请确保是以root权限运行！
```
## mentohust.conf 

```
# MentoHUST for Linux By HustMoon Studio
#
# 配置文件名称必须是小写/etc/mentohust.conf，编码格式建议是UTF-8
# 配置文件中一行开头的空格和Tab会被忽略，其他的会视为参数一部分
# 配置文件中Section与Key不区分大小写，以#或;开头的行视为注释
# 同一个参数若在命令行参数和配置文件中均有设置，使用命令行参数
# 命令行中使用参数-h或-?可查看详细参数信息，建议通过命令行设置参数
# 命令行中使用参数-w可将配置更新到/etc/mentohust.conf

[MentoHUST]
;用户名，长度不超过64
Username=
;密码(简单加密)
EncodePass=
;网卡
Nic=
;静态IP用户可以使用非本机IP
IP=
;掩码，无关紧要
Mask=
;网关，如果指定了就会监视网关ARP信息
Gateway=
;DNS服务器，无关紧要
DNS=
;Ping主机，用于掉线检测，0.0.0.0表示关闭该功能
PingHost=
;每次发包超时时间（秒）
Timeout=
;发送Echo包的间隔（秒）
EchoInterval=
;失败等待（秒）认证失败后等待RestartWait秒或者服务器请求后重启认证
RestartWait=
;寻找服务器时的组播地址类型 0标准 1锐捷 2将MentoHUST用于赛尔认证
StartMode=
;DHCP方式 0(不使用) 1(二次认证) 2(认证后) 3(认证前)
DhcpMode=
;是否后台运行: 0(否) 1(是，关闭输出) 2(是，保留输出) 3(是，输出到文件/tmp/mentohust.log)
DaemonMode=
;是否显示通知： 0(否) 1~20(是)
ShowNotify=
;客户端版本号，如果未开启客户端校验但对版本号有要求，可以在此指定，形如3.30
Version=
;认证数据文件，如果需要校验客户端，就需要正确设置
DataFile=
;进行DHCP的脚本
DhcpScript=
```


**需要注意的点**

1. 需要选对网卡，如果不确定具体选择那个网卡，利用`ifconfig`查看网卡进行选择
2. 组播地址：1锐捷私有，DHCP方式：1二次认证

## 开机自启动与断线重连

经过上述的安装之后,在luci界面勾选上开机启动(随路由器启动mentohust)好像并不能成功的开机启动mentohust,所以通过手动配置的方式配置开机自启动同时配置断线重连.

**开机自启动**只需要在`/etc/rc.local`里面添加下面这句即可

``` bash
mentohust 
```

**断线重连**没有采用corntab和mentohust自带的方式(好像有点问题),断线后也是不能自动重连,所以采用脚本的方式,通过不断地`ping`来监测网络通断进而重启`mentohust`.脚本代码如下

```bash
#!/bin/sh
server="114.114.114.114" #不要是本地网关,容易被ban

while true
do
    ping -c 2 $server
    if [ $? != 0 ]
    then
        echo "#### Ping Failed, Retry"
        mentohust -k
        sleep 5
        mentohust -c /etc/mentohust.conf > /root/mentohust.log
        sleep 5
    else
        echo "#### The Network Is Fine"
    fi
    sleep 5
done
```

之后将脚本代码加入`/etc/rc.local`来实现开机自启动与断线重连

## 问题记录

### 网页打开luci-mentohust时报错module ‘luci.cbi‘ not found。
解决办法,分别运行以下两行命令：
```bash
opkg update
opkg install luci luci-base luci-compat
```

### 连接时无法获得ip地址
检查网卡发现网卡选择错误，利用`ifconfig`查看每个网卡ip地址重新选择了网卡为`eth0.2`，之后就可以重新认证了。
