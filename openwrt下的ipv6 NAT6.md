# openwrt下的ipv6 NAT6
> [源blog](https://github.com/lixingcong/my-hexo-blog2/blob/master/source/_posts/ipv6-nat-lede.md)

校园网的ipv6路由器做NAT6，使得路由器下级可以使用v6协议访问网站。
<!-- more -->
测试平台

	LEDE 17.01.1
	DNS: ss-tunnel

全文所有步骤均在LEDE上面操作

首先确保在LEDE上面能ping通ipv6的地址

	ping ipv6.google.com

安装ip6tables和nat6

	opkg install ip6tables kmod-ipt-nat6

## ULA prefix

LUCI -> Network -> Interface

NAT6转换的ULA网段要求是2fff::/64网段，因此把ULA前缀改为2fff::/64内的任意网段，比如以下都是合法的ULA

	2fff:3333:4444::/64
	2fff:cccc:dddd:eeee:/64
	2fff:1:1::/64

![](/images/ula.png)

如果不修改的话，也能正常使用，但是挂在路由器下客户端解析出来的IP结果很多是v4的，我希望尽量多的网站走v6，建议设置ULA为2fff::/64网段

## Announce default route

LUCI -> Network -> Interface -> LAN

宣告默认路由，勾选即可。

![](/images/announce.png)

## Default gateway

查看当前IPv6默认路由如下

	ip -6 route | grep "default from"

若结果是这样的

	default from (ipv6 range) via (gateway) dev (intf) proto static  metric 512

就需要向下一级宣告默认网关，括号的内容自行替换为上面结果。

	ip -6 r add default via (gateway) dev (intf)

## MASQUERADE(SNAT)

利用ip6tables进行内网NAT，括号内容为上面的结果

	ip6tables -t nat -A POSTROUTING -o (intf) -j MASQUERADE

这时候连接到路由器的计算机或者手机，估计可以访问IPv6了，可以chrome打开ipv6.google.com看看

## Auto script

（可选步骤）我将上面的过程写成脚本，每次连接ipv6后自动设置NAT。

	#!/bin/sh
	# filename: /etc/hotplug.d/iface/90-ipv6
	# please make sure this file has permission 644
	
	# check if it is the intf which has a public ipv6 address like "2001:da8:100d:aaaa:485c::1/64"
	interface_public="wan6"
	[ "$INTERFACE" = "$interface_public" ] || exit 0
	
	res=`ip -6 route | grep "default from"`
	gateway=`echo $res | awk '{print $5}'`
	interface=`echo $res | awk '{print $7}'`
	
	if [ "$ACTION" = ifup ]; then
		ip -6 r add default via $gateway dev $interface
		if !(ip6tables-save -t nat | grep -q "v6NAT"); then
			ip6tables -t nat -A POSTROUTING -o $interface -m comment --comment "v6NAT" -j MASQUERADE
		fi
	else
		ip6tables -t nat -D POSTROUTING -o $interface -m comment --comment "v6NAT" -j MASQUERADE
		ip -6 r del default via $gateway dev $interface
	fi

上面的脚本使用要注意，变量interface_pulbic是带有公网IPV6地址的接口地址，比如我的是这种情况，那么变量interface_pulbic是wan6

![](/images/intf_screen.png)

这脚本位置，还有加上权限

	chmod 644 /etc/hotplug.d/iface/90-ipv6

效果就是每次ipv6连接和断开就自动添加和删除路由规则(实际上，不知道为什么ip6tables规则无法移除，反正无影响)

## Reference

[清华大学 路由器作为 IPv6 网关的配置](https://github.com/tuna/ipv6.tsinghua.edu.cn/blob/master/openwrt.md)
[LEDE配置IPv6 NAT](https://blog.blacate.me/2017/04/09/ipv6-nat-on-openert-lede/)