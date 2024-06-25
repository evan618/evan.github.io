# 本帖解决Openwrt的Docker下安装homeassistant接入Homekit二维码不成功 或者一直转圈的问题！
仅仅给那些跟我一样白的不能再白了的小白朋友们一个经验指引
通过查阅无数  youtube csdn 简书  知乎 百度  Google  github资料  本太白金星成功添加homekit桥设备

## 前提
X86软路由 openwrt（已经带好docker环境）
portiner 已拉取并安装（这一步可以忽略 不做也可以）
拉取并安装homeassistant到docker
hacs商店插件安装（YouTube上有详细教程，如果遇到问题可以给我留言，这一步也有一个很细的问题）
配置好了homeassistant以后可以访问到web界面


## 问题原因：
    因为docker下的默认网络环境是三种 bridge，host，none。一般大家按照各种大神的教程做的话默认的容器网络一般都是bridge。docker的网络是虚拟出来的并不是真正的物理网卡，所以homekit在连接的时      候就不是跟主路由在同一网段下，可以在ssh命令：docker network ls 查看。



## 解决方法思路：

开启混杂网络macvlan（具体原理查阅下面参看文案连接）
创建一个macvlan模式的docker网络模式
将新建的homeassistant容器用macvlan网络模式加载




##具体ssh命令操作！

第一步：ip link set br-lan promisc on

###开启混杂网络模式  
标出来的"br-lan"为你的openwrt主路由的网卡名称
查阅方法ssh命令：ip addr

会出现下图类似的很多个网卡，看图中的192.168.5.1，也就是openwrt的地址，我没有其他的旁路由，其网络名称为br-lan
所以上面的第一步命令就是 ip link set br-lan promisc on,如果你的是eth0，那么命令就是ip link set eht0 promisc on     学会贯通~

### 第二步：
docker network create -d macvlan --subnet=192.168.5.55/24 --ip-range=192.168.5.55/24 -o macvlan_mode=bridge -o parent=br-lan macvlan

#创建一个macvlan网络模式

#命令中的195.168.5.55/24是因为我上面选择br-lan的地址是192.168.5.1  所以分配一个跟你现有网络地址不冲突的在同一网段就可以，可以理解为一个虚拟网关。
  如果你的主路由地址为192.168.2.1 那么命令中的地址就可以是 192.168.2.X/24 自己改成自己的地址即可  
  后面的br-lan即为上面的网卡名称，如果你的是eth0 这里就改成eth0


第三步之前，要删除当前的homeassistant容器，因为大家都是小白，所以这里不用ssh命令，可以直接到openwrt-docker-容器 选中homeassistant容器，先停止，再删除（不用担心yaml文件和hacs源，这里资料不会丢失）

第三步：  

docker run -d --restart=always --network=macvlan --ip=192.168.5.5 --privileged --name=homeassistant  -v /opt/docker/homeassistant:/config homeassistant/home-assistant:latest


#创建一个新的homeassistant容器ip为192.168.5.5 容器名称为 homeassistant
--restart=always 是指当软路由重启或者docker重启的时候 该容器会自动启动
--ip=192.168.5.5 这条命令会改变你的homeassistant容器的访问地址，自己设定自己想要访问的地址即可，例：你的网关是192.168.2.1，那么命令可以是 --ip=192.168.2.x
--privileged  该容器获取真正的sudo权限
--name=homeassistant 为创建的容器自定义名称为homeassistant
-v /opt/docker/homeassistant:/config  这句意思是把homeassistant容器的config文件夹挂载到你的docker另一个文件目录
      
 如果你是看的IT conmmander的教程把软路由的剩余分区当成了docker的挂载盘的话就是这么操作的       如果你没有挂载，这一句可以不加直接删掉就就行 前提你的软路由硬盘大小够大！
#homeassistant/home-assistant:latest 是你的docker镜像标签名称  可以到openwrt-docker-镜像 找到你拉取的homeassistant的镜像的标签名称



后序：当你运行完了第三条命令以后，出现了一个容器id的结果  没有什么错误提示的话，那么恭喜你。已经成功创建了新的homeassistant容器！
                 此时，你可以到openwrt-docker-容器 看看里面还不是有一个网络为macvlan 192.168.x.x的容器


然后你就可以用这个新的ip:8123访问你的homeassistant

快快去添加你的homekit    用siri来命令一切其他品牌的只能家居吧！


### 参考文案链接：
[](https://blog.csdn.net/KEYMA/article/details/114114726)
[](https://www.cnblogs.com/hgdf/p/13812369.html)


