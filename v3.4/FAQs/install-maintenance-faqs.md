---
title: 平台安装，维护-常见问题
summary: 云帮安装、维护相关的常见问题。
toc: false
---

<div id="toc"></div>

### 云帮目前只支持CentOS 7.x 是否有支撑其他操作系统的计划？

> 目前我们在在CentOS 7.x 系统下做了严格的测试，并且具备了丰富的生产环境使用经验。因此建议用户也选用基于CentOS 7.3的系统安装云帮。后续会加入Debian/Ubuntu系统的支持。


### 云帮所需要的分布式存储是否需要提前部署好？
> A：
> 如果有分布式存储，可以将分布式存储挂载到所有节点的 `/grdata` 目录；如果没有分布式存储，在执行安装脚本的时候会自动安装nfs服务，并在所有节点挂载。
>
> demo或者测试环境，使用nfs实现分布式存储就可以了，如果是生产环境，建议使用[GlusterFS](/docs/stable/platform-maintenance/distributed-storage/GlusterFS/install.html)或者CEPH。


### 在阿里云等公有云环境部署云帮后，应用域名解析为内网ip，如何更改默认域名解析设置？

> 云帮平台上运行起来的应用，如果需要外部访问，都需要一个域名与之对应。这个域名是应用的标示，同时也是平台负载均衡区分后端服务的方法。
> 云帮安装程序默认会自动注册一个`*.<random>.goodrain.org` 的泛域名并进行dns的解析工作

```
# 查看当前解析记录
dig  *.y5vhe.goodrain.org | grep -A 2 "ANSWER SECTION"
;; ANSWER SECTION:
*.y5vhe.goodrain.org.	60	IN	A	10.26.98.37
# 更改当前解析记录
grctl domain --ip 59.110.8.184 --domain y5vhe.goodrain.org # domain 参数可选
# 稍等3分钟左右等待解析生效后再次查看
dig *.y5vhe.goodrain.org | grep -A 2 "ANSWER SECTION"
;; ANSWER SECTION:
*.y5vhe.goodrain.org.	60	IN	A	59.110.8.184
```

### 云帮支持集群管理吗？下图怎么实现的，用Docker编排工具？

> <img src="https://static.goodrain.com/images/acp/docs/faqs/rain-usage-faq.png" width="100%"/>
> **A：**

> 云帮是以应用为中心的PaaS，没有普通机器管理的集群概念。
>
> 云帮平台底层是基于Kubernetes的，集群状态是自动维护的。

### 云帮的日志下载功能怎样配置？

> **A：**

> 应用的输出日志会按天保存在分布式存储的 `/grdata/downloads/log`目录下。为方便从控制台 "日志" 标签页进行下载，管理节点的[rbd-proxy](/docs/stable/platform-maintenance/add-management-node/component-introduction/rbd-proxy.html)组件提供了文件下载服务。

> **创建日志目录**

> ```
> mkdir /grdata/downloads
> chown rain.rain  /grdata/downloads
> ```

> **配置rbd-proxy服务**

> 配置文件 ：/etc/nginx/sites-enabled/download

> 默认监听了 `<local_ip>:8083`, 用户可以修改成适合自己的, 比如本机公网IP

> 改完以后重启rbd-proxy

> ```
> dc-compose restart rbd-proxy
> ```

> **修改web控制台的配置文件**

> 配置文件：`/etc/goodrain/console.py`找到 LOG_DOMAIN 字段, 修改对应的地址

### 在多个容器节点的集群环境，如果某个容器节点挂了，该节点上的容器会在其他节点上自动重启吗？

> 
> 会自动重启，无论是云帮平台，还是Kubernetes的设计，都可以保证这一点。

> 自动重启一般会在数秒内完成，用户几乎感觉不到影响。

### 如何访问云帮内部的私有docker镜像仓库？

> 云帮通过[rbd-registry](/docs/stable/platform-maintenance/add-management-node/component-introduction/rbd-registry.html)组件实现docker镜像仓库，goodrain.me的域名就是本地的镜像仓库。

> 私有docker镜像仓库路径：

> ```
> <管理节点>: /etc/goodrain/tengine/sites/registry
>```

> 通过`dc-compose ps | grep registry`可以看到容器服务

>compose的配置文件在`/etc/goodrain/docker-compose.yaml`

### 3.4版本如何升级到3.4.1版本

```
wget repo.goodrain.com/release/3.4.1/gaops/jobs/update/update.sh -O /root/update_version.sh
chmod +x /root/update_version.sh
bash /root/update_version.sh
```

### 如何查看安装日志

```
安装日志默认 /var/log/event/
```

### 3.4.1 版本安装过程中可能遇到到问题

```
Q:安装任务卡住或者停止了,如何处理
A:新开一个终端，执行systemctl restart rainbond-node
```

### 机器重启后发生无法访问控制台

```
1. 检查机器ip
ip r
2. 检查机器dns
cat /etc/resolv.conf
<dns 应该和机器ip一致>
3. 如果不一致，检查相关配置
grctl configs get
确定ip具体是何值
4. 修改相关配置以grctl configs get获取的值为准
/etc/resolv.conf 和 /etc/sysconfig/network-scripts/ifcfg-eth0

绝大数可能是网卡获取ip是dhcp导致的。
cat /etc/sysconfig/network-scripts/ifcfg-eth0 # 具体网卡具体对待

如下静态ip demo：

TYPE=Ethernet
BOOTPROTO=static
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
NAME=eno1
UUID=ea4bbbab-1804-46df-b8f9-1451e5a710fc
DEVICE=eno1
ONBOOT=yes
IPADDR=192.168.1.45
GATEWAY=192.168.1.1
NETMASK=255.255.255.0

说明：
BOOTPROTO=static                  #使用static配置
ONBOOT=yes                        #开机启用本配置
IPADDR=192.168.1.45               #静态IP
GATEWAY=192.168.1.1               #默认网关
NETMASK=255.255.255.0             #子网掩码

最后，修改完重启一下网络
systemctl restart network

dc-compose stop
cclear
dc-compose up -d
```