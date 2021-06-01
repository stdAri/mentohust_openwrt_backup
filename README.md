# mentohust_openwrt_backup
备份自己刷路由器的过程及相关软件。

wndr4300v1为路由器固件
两个ipk文件为mentohust相关文件

**备份两个连接**  
[【openwrt/LEDE】mentohust--校园网锐捷认证(minieap测试) ](https://www.right.com.cn/forum/forum.php?mod=viewthread&tid=196317&extra=page%3D1&page=1)  
[【4.25更新】MiniEAP(多架构支持)——MentoHUST替代插件，厦门理工锐捷认证工具](https://www.right.com.cn/forum/thread-4106567-1-1.html)

# 安装mentohust

[下载链接](https://github.com/viseator/mentohust_for_ar71xx)


注意先要安装官方源中的`libpcap`（`openwrt`本身应处于联网状态）：

```bash
opkg update
opkg install libpcap
```

再安装`libsodium_1.0.12-1_ar71xx.ipk`：

```bash
opkg install libsodium_1.0.12-1_ar71xx.ipk
```

再安装`mentohust`：

```bash
opkg install mentohust_0.3.1-1_ar71xx.ipk
```

安装`luci`界面：

```bash
opkg install luci-app-mentohust_trunk+svn-1_ar71xx.ipk 
```

之后使用时一定要选对网卡，打开`luci`界面的`enable`。
