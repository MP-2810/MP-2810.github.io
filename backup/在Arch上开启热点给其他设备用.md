## 1. 先确认安装了NetWorkManager
## 2. 再确认网卡名
`nmcli device status`
找到当前电脑使用的网络类型（WLAN或者网线）使用的网卡名字
例如如下输出：
```
DEVICE           TYPE      STATE         CONNECTION         
enp8s0           ethernet  已连接        Wired connection 1 
lo               loopback  连接（外部）  lo                 
wlp14s0          wifi      已断开        --                 
p2p-dev-wlp14s0  wifi-p2p  已断开        --  
```
当前使用有线网络，对应的网卡名字是`enp8s0`
而开启热点需要使用无线网卡来提供热点服务
可得无线网卡名字为`wlp14s0`
使用命令：
`nmcli device wifi hotspot ifname wlp14s0 ssid ArchHotspot password "12345678"`
开启热点服务
其中ssid是热点名字，password是密码（最少八位）
也可以使用GUi工具nm-connection-editor来配置修改
选择想要分享的网络
然后添加新链接，选择Wi-Fi即可添加修改热点