# 本帖解决Openwrt的Docker下安装homeassistant接入Homekit二维码不成功 或者一直转圈的问题！


## 前提
X86软路由 openwrt（已经带好docker环境）
portiner 已拉取并安装（这一步可以忽略 不做也可以）
拉取并安装homeassistant到docker
hacs商店插件安装（YouTube上有详细教程，如果遇到问题可以给我留言，这一步也有一个很细的问题）
配置好了homeassistant以后可以访问到web界面


## 问题原因：

因为docker下的默认网络环境是三种 bridge，host，none。一般大家按照各种大神的教程做的话默认的容器网络一般都是bridge。docker的网络是虚拟出来的并不是真正的物理网卡，所以homekit在连接的时      候就不是跟主路由在同一网段下


## 使用macvlan解决

查看docker 网络情况
`docker network ls`


解决方法思路：

开启混杂网络macvlan（具体原理查阅下面参看文案连接）
创建一个macvlan模式的docker网络模式
将新建的homeassistant容器用macvlan网络模式加载

查看网卡

`ip addr`

开启混杂网络模式  

`ip link set br-lan promisc on`

标出来的"br-lan"为你的openwrt主路由的网卡名称，_一般情况下双网口的设备为eth0 三网口及以上为 br-lan_

会出现下图类似的很多个网卡，看图中的192.168.5.1，也就是openwrt的地址，我没有其他的旁路由，其网络名称为br-lan
所以上面的第一步命令就是 ip link set br-lan promisc on ,如果你的是eth0，那么命令就是ip link set eht0 promisc on     学会贯通~

创建 macvlan 网卡桥接

`docker network create -d macvlan --subnet=192.168.5.55/24 --ip-range=192.168.5.55/24 -o macvlan_mode=bridge -o parent=br-lan macvlan`

命令中的195.168.5.55/24是因为我上面选择br-lan的地址是192.168.5.1  所以分配一个跟你现有网络地址不冲突的在同一网段就可以，可以理解为一个虚拟网关。
如果你的主路由地址为192.168.2.1 那么命令中的地址就可以是 192.168.2.X/24 自己改成自己的地址即可  
后面的br-lan即为上面的网卡名称，如果你的是eth0 这里就改成eth0

创建一个新的 homeassistant 容器ip为192.168.5.5 容器名称为 homeassistant

`docker run -d --restart=always --network=macvlan --ip=192.168.5.5 --privileged --name=hasstant -v /opt/docker/homeassistant:/config homeassistant/home-assistant:latest`

|参数|注释|
|-----|-----|
|--restart=always|自动启动|
|--ip=192.168.5.5|homeassistant容器访问地址|
|--privileged|最高权限运行|
|--name=homeassistant|容器名|
|-v /opt/docker/homeassistant:/config|这句意思是把homeassistant容器的config文件夹挂载到你的docker另一个文件目录|
|homeassistant/home-assistant:latest|是你的docker镜像标签名称|


## 修改 Network Adapter 方法

在homeassistant中打开高级设置，然后在配置 -> 系统 -> 网络中修改 “Network Adapter“就可以解决问题了，不需要使用macvlan

<img width="1208" alt="180143q71qx1x1zla1q7i1" src="https://github.com/evan618/evan.github.io/assets/20328741/015ee61a-4041-4ed7-b75d-45e3858cdf9d">

### 参考文案链接：
https://blog.csdn.net/KEYMA/article/details/114114726
https://www.cnblogs.com/hgdf/p/13812369.html


