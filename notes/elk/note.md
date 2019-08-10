# Centos7

``` text
准备：
VMware 15
Centos 7 mini
```

## 1. 配置环境

### 1.1 首先配置静态网络地址

``` bash
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```

修改如下内容。

``` text
# 设置 IP 地址为静态 IP
BOOTPROTO=static
# 随 network 服务启动
ONBOOT=yes
# 指定静态 IP
IPADDR=192.168.16.17
# 设置子网掩码
NETMASK=255.255.255.0
# 设置网关
GATEWAY=192.168.16.2
# 设置DNS
DNS1=192.168.16.2
```

其中 *GATEWAY* 和 *DNS* 的地址为 **VMware** 中 *NAT* 模式设置的默认网关 IP， *IPADDR* 为指定子网的 IP。设置方法为 *编辑->虚拟网络编辑器* 然后在名称里选择 ***VMnet8 NAT模式***。就可以设置 *子网 IP* 和 *子网掩码*，在 *NAT设置* 中设置网关。

### 1.2 测试网络是否通畅

``` bash
# 重启网络
systemctl restart network
# 查看 ip，查看 ens33 端口IP是否出现
ip a
# 测试网络环境
ping www.baidu.com
```

### 1.3 安装完成

``` bash
# 保持系统最新版本
yum -y update && yum -y upgrade && yum -y autoremove
```

网络成功后，安装vim（个人喜好）。`yum -y install vim wget`。最好使用类似 *XShell* 的 *ssh* 终端连接操作。

### 1.4 配置 yum 源

``` text
前言：Linux 属于国外的系统，要想安装软件，通过系统自带的国外软件源，有的会感觉有些卡顿，所以有了国内软件源的出现。其中用的比较多的就是清华源、阿里源、中科大源等等，笔者最常用的就是清华源，当然了，下载一些 Linux 的镜像也会去下载，速度什么的还是不错的。
```

#### 1.4.1 [清华源](https://mirrors.tuna.tsinghua.edu.cn/)

在镜像站一般会有介绍如何使用该镜像源的方法（系统名称右上角有问号，点击即可），这里简单介绍下。

``` bash
# 备份原来的源
sudo mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
# 使用 vi ，创建一个新的文件
vi  /etc/yum.repos.d/CentOS-Base.repo
```

进入后，按 `i` 进入编辑模式，将以下内容复制进去。

``` bash
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the
# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-$releasever - Base
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/os/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#released updates
[updates]
name=CentOS-$releasever - Updates
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/updates/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/extras/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/centosplus/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

使用命令进行版本的修改。

``` bash
# 修改源系统版本
sudo sed -i `s/$releasever/7/g` /etc/yum.repos.d/CentOS-Base.repo
# 更新软件包缓存
sudo yum makecache
```

#### 1.4.2 [阿里源](https://opsx.alibaba.com/mirror)

阿里源配置首先也是要备份源的，执行命令。

``` bash
# 备份系统源
sudo mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak

# 下载新的源文件 Centos-Base.repo 到 /etc/yum.repo.d/ （这里只介绍 Centos7）
# 使用 wget ，首先要先安装，默认最小安装的系统不带 wget。
sudo yum -y install wget
# 下载阿里源文件
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# 或者
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# 修改源系统的版本
sudo sed -i `s$releasever/7/g` /etc/yum.repos.d/CentOS-Base.repo
# 生成缓存
sudo yum makecache
```

## 2. 搭建 ELK

> 简介：ELK 是一整套解决方案，是三个软件产品的首字母缩写， Elasticsearch、 Logstash 和 Kibana。这三款软件都是开源产品，通常是配合使用，简称为 *ELK 协议栈*。

日志主要包括系统日志、应用程序日志和安全日志。系统运维和开发可以通过日志了解服务器软硬件信息、检查配置过程中的错误击错误发生的原因。经常分析日志可以了解服务器的负荷，性能安全，从而及时采取措施纠正错误。

开源实时日志分析 ELK 平台由 Elasticsearch、 Logstash 和 Kibana 三个开源工具组成。现在新增一个 FileBeat，它是一个轻量级的日志收集处理工具（Agent），FileBeat 占用资源少，适合于在各个服务器上搜集日之后传输给 Logstash 。

+ **Elasticsearch** 是个开源分布式搜索引擎，提供搜集、分析、存储数据三大功能。它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，restful风格接口，多数据源，自动搜索负载等。

+ **Logstash** 主要是用来日志的搜集、分析、过滤日志的工具，支持大量的数据获取方式。一般工作方式为c/s架构，client端安装在需要收集日志的主机上，server端负责将收到的各节点日志进行过滤、修改等操作在一并发往elasticsearch上去。

+ **Kibana** 也是一个开源和免费的工具，Kibana可以为 Logstash 和 ElasticSearch 提供的日志分析友好的 Web 界面，可以帮助汇总、分析和搜索重要数据日志。

+ **FileBeat** 隶属于 Beats。目前 Beats 包括四种工具。
  + Packetbeat 搜集网络数据流量
  + Topbeat 搜集系统、进程和文件系统级别的 CPU 和内存使用情况等数据
  + Filebeat 搜集文件数据
  + Winlogbeat 搜集 Windows 事件日志数据

***工作原理*** ： 在需要收集日志的所有服务上部署logstash，作为logstash agent（logstash shipper）用于监控并过滤收集日志，将过滤后的内容发送到logstash indexer，logstash indexer将日志收集在一起交给全文搜索服务ElasticSearch，可以用ElasticSearch进行自定义搜索通过Kibana 来结合自定义搜索进行页面展示。

### 2.1 开源实时日志分析 ELK 平台部署流程

> 环境配置  
> 系统（3台） ：Centos 7  
> Beats ：filebeat(7.3.0)  
> ElasticSearch：7.3.0  
> Logstash：7.3.0  
> Kibana：7.3.0  
> Java（只需要 java 的运行环境，可以只安装 JRE ）：JDK-12.0.2

| 软件名称 | 安装方式 | 服务器数量 | IP |
| :--- | :--- | :--- | :--- |
| ElasticSearch | Cluster |  三台，1主两从 | Master：192.168.16.17 </br> data：192.168.16.3 </br> data：192.168.16.4 |
| Logstash | 单机 | 1台 | 192.168.16.3 |
| Kibana | 单机 | 1台 | 192.168.16.4 |
| Filebeat | 单机 | 所有被采集的服务器 |  |

#### 2.1.1 Selinux 和防火墙的设置

#### 2.1.2 安装 Logstash 依赖包 JDK

安装 `apt -y install jdk-12.0.2_linux-x64_bin.rpm` 可以将所缺的依赖一起安装。使用 `java --version` 检查 Java 是否安装成功。成功将返回 Java 的版本。

#### 2.1.3 Elasticsearch cluster 安装（三台）

> 注：所有节点都可以做 Master/data 节点，设置为 Master 说明该节点有作为 Master 的资格，默认设置为 true，默认集群中的第一台机器为 Master。data 节点存储数据，不设置默认为 true。

| 类型 | IP | 名称 |
| :---- | :---- | :---- |
| Master/data | 192.168.16.17 | elk-node1 |
| Master/data | 192.168.16.3 | elk-node2 |
| Master/data | 192.168.16.4 | elk-node3 |
|  |  |  |

##### a. 重要的 Elasticsearch 配置(https://cloud.tencent.com/developer/article/1454003)

+ `path.data` 和 `path.logs`  
如果正在使用 .zip 或 .tar.gz 文件归档，data 和 logs 目录在 $ES_HOME 下。如果这些重要文件夹保存在默认位置，则 ElasticSearch 升级到新版本时，很有可能被删除。所以在生产环境中，肯定要更改数据和日志文件夹的位置。

+ `cluster.name`  
某个节点只有和集群下的其他节点共享（使用相同的） `cluster.name` 才能加入到同一个集群。一定要确保不能在不同的环境中使用相同的集群名称，否则，节点可能会加入错误的集群中。

+ `node.name`  
默认情况下， ElasticSearch 将使用随机生成的 uuid 的前 7 个字符作为节点 id，请注意，节点 id 是持久化的，并且在节点重新启动时不会改变，因此默认节点名称不会更改。  
也可以使用服务器的 HOSTNAME 作为节点的名称。  
`node.name: ${HOSTNAME}`

设置完成的参数

``` bash
/_cat/allocation
/_cat/shards
/_cat/shards/{index}
/_cat/master
/_cat/nodes
/_cat/tasks
/_cat/indices
/_cat/indices/{index}
/_cat/segments
/_cat/segments/{index}
/_cat/count
/_cat/count/{index}
/_cat/recovery
/_cat/recovery/{index}
/_cat/health
/_cat/pending_tasks
/_cat/aliases
/_cat/aliases/{alias}
/_cat/thread_pool
/_cat/thread_pool/{thread_pools}
/_cat/plugins
/_cat/fielddata
/_cat/fielddata/{fields}
/_cat/nodeattrs
/_cat/repositories
/_cat/snapshots/{repository}
/_cat/templates
```

#### 2.1.4 Logstash 安装

#### 2.1.5 Kibana 安装

#### 2.1.6 Filebeat 安装

#### Nginx 代理
