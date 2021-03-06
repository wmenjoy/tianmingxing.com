---
title: MongoDB安装
date: 2017-11-18 10:55:32
tags:
- mongodb
- nosql数据库
- mongodb安装
categories:
- 数据库
---

# 简介

MongoDB是一个使用C++编写的、开源的、面向文档的NoSQL（Not Only SQL）数据库，也是当前最热门的NoSql数据库之一。

## NoSQL简介

NoSQL的意思是“不仅仅是SQL”，是目前流行的“非关系型数据库”的统称。常见的NoSQL数据库如：Redis、CouchDB、MongoDB、HBase、Cassandra等。

* 出现NoSQL的原因：为解决在Web2.0时代出现的**三高**要求：
  1. 对数据库高并发读写的需求
  1. 对海量数据的高效率存储和访问的需求
  1. 对数据库的高可扩展性和高可用性的需求
  1. 而RDB里面的一些特性，在web2.0里面往往变得不那么重要，比如：
    * 数据库事务一致性
    * 数据库的实时读写
    * 复杂的SQL查询，特别是多表关联查询

从第4点可以看出MongoDB的使用场景，但具体的我在后面重点介绍。
<!-- more -->
## CAP定理

又被称作布鲁尔定理（Eric Brewer）它指出对于一个分布式计算系统来说，不可能同时满足以下三点：

1. 强一致性（Consistency）：系统在执行某项操作后数据状态仍然处于一致，例如在分布式系统中，更新操作执行成功后所有的用户都应该读取到最新的值，这样的系统被认为具有强一致性。
1. 可用性（Availability）：每一个操作总是能够在一定的时间内返回结果。
1. 分区容错性（Partition tolerance）：单个节点故障不应导致整个系统崩溃，也就是说尽管网络在节点之间丢弃（或延迟）任意数量的消息，但是系统继续操作。

根据CAP原理将数据库分成了满足CA原则、满足CP原则和满足AP原则三大类

1. CA：单点集群，满足一致性，可用性，通常在可扩展性上不太强大，比如RDB
1. CP：满足一致性和分区容错性，通常性能不是特别高，如分布式数据库
1. AP：满足可用性和分区容错性，通常可能对一致性要求低一些，如大多数的NoSQL</li>

## BASE（Basically Available，Soft-state，Eventual consistency）

eBay的架构师Dan Pritchett源于对大规模分布式系统的实践总结，在ACM上发表文章提出BASE理论，BASE理论是对CAP理论的延伸，核心思想是即使无法做到强一致性（Strong Consistency，CAP的一致性就是强一致性），但应用可以采用适合的方式达到最终一致性（Eventual Consitency）。

* 基本可用（Basically Available）：系统能够基本运行并一直提供服务
* 软状态（Soft-state）：系统不要求一直保持强一致状态
* 最终一致性（Eventual consistency）：系统需要在某一时刻后达到一致性要求

## NoSQL的特点

* 优点
  1. 扩展简单方便，尤其是水平横向扩展（纵向扩展是指用更强的机器；横向扩展是指把数据分散到多个机器）
  1. 读写快速高效，多数都会映射到内存操作
  1. 成本低廉，用普通机器，分布式集群即可
  1. 数据模型灵活，没有固定的数据模型
* 缺点
  1. 不提供对SQL的支持
  1. 对事务操作的支持较弱

# MongoDB使用场景

相信有很多人了解之后都会觉得，nosql好是好但我该在哪些情况下使用，又如何与实际项目结合呢？接下来我举几个例子，大家自己体会体会。

1. 游戏场景，使用 MongoDB 存储游戏用户信息，用户的装备、积分等直接以内嵌文档的形式存储，方便查询、更新
1. 物流场景，使用 MongoDB 存储订单信息，订单状态在运送过程中会不断更新，以 MongoDB 内嵌数组的形式来存储，一次查询就能将订单所有的变更读取出来。
1. 社交场景，使用 MongoDB 存储存储用户信息，以及用户发表的朋友圈信息，通过地理位置索引实现附近的人、地点等功能
1. 物联网场景，使用 MongoDB 存储所有接入的智能设备信息，以及设备汇报的日志信息，并对这些信息进行多维度的分析
1. 视频直播，使用 MongoDB 存储用户信息、礼物信息等

并没有某个业务场景必须要使用 MongoDB才能解决，但使用 MongoDB 通常能让你以更低的成本解决问题（包括学习、开发、运维等成本），下面是 MongoDB 的主要特性，大家可以对照自己的业务需求看看，匹配的越多，用 MongoDB 就越合适。

|MongoDB 特性 |优势 |
|----------------------|-------|
|事务支持 |MongoDB 目前只支持单文档事务，需要复杂事务支持的场景暂时不适合 |
|灵活的文档模型 |JSON 格式存储最接近真实对象模型，对开发者友好，方便快速开发迭代 |
|高可用复制集 |满足数据高可靠、服务高可用的需求，运维简单，故障自动切换 |
|可扩展分片集群 |海量数据存储，服务能力水平扩展 |
|高性能 |mmapv1、wiredtiger、mongorocks（rocksdb）、in-memory 等多引擎支持满足各种场景需求 |
|大的索引支持 |地理位置索引可用于构建 各种 O2O 应用、文本索引解决搜索的需求、TTL索引解决历史数据自动过期的需求 |
|Gridfs |解决文件存储的需求 |
|aggregation & mapreduce |解决数据分析场景需求，用户可以自己写查询语句或脚本，将请求都分发到 MongoDB 上完成 |

# MongoDB安装

> mongodb的安装方式比较简单，下面我演示在CentOS7上用源码和yum两种方式安装。


## yum安装

> 在实际生产环境中服务器上的软件通常都由运维人员安装，我们通过此种傻瓜式安装是为了方便现在学习。

* 整个mongodb（社区版）包含如下软件
  1. mongodb-org-server 包含mongod守护程序和关联的配置和init脚本
  1. mongodb-org-mongos 包含mongos守护程序
  1. mongodb-org-shell 包含mongo shell，它是一个连接mongodb的命令行客户端，允许用户直接输入nosql语法管理数据库。
  1. mongodb-org-tools 包含以下工具的MongoDB：数据导入、导出、备份、恢复等等
* 创建yum源文件 `vim /etc/yum.repos.d/mongodb-org-3.4.repo`
* 把下面的内容复制到上面的文件中
    ```shell
    [mongodb-org-3.4]
    name=MongoDB Repository
    baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.4/x86_64/
    gpgcheck=1
    enabled=1
    gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc
    ```
* 启动yum命令开始安装 `yum install -y mongodb-org`
* 如果使用SELinux，则必须配置SELinux，以允许在基于Red Hat Linux的系统（Red Hat Enterprise Linux或CentOS Linux）上启动MongoDB。
    ```shell
    vim /etc/selinux/config
    #在打开的文件中将值设置为disabled
    SELINUX=disabled
    ```
* 启动mongodb
    ```
    #centos6用这种方式
    service mongod start

    #centos7用这种方式
    systemctl start mongod
    ```
* 检查进程是否成功启动，可以通过查看mongodb的日志文件来判断，打开 `/var/log/mongodb/mongod.log` 查看里面有无内容。我们可以将mongodb服务设置为开机就启动 `chkconfig mongod on`。
* 其它控制命令
    ```
    #停止mongodb服务
    service mongod stop
    #重启mongodb
    service mongod restart
    ```

## 命令行客户端连接到MongoDB服务

```shell
./bin/mongo
#上面的命令输入后直接回车，你应该可以看到和我相同的提示

root@bogon mongodb-linux-x86_64-rhel70-3.4.1]# ./bin/mongo
MongoDB shell version v3.4.1
connecting to: mongodb://127.0.0.1:27017
MongoDB server version: 3.4.1
Welcome to the MongoDB shell.

#我们可以输入一个命令测试一下，现在教你一个最简单的命令，查看当前数据库有哪些
> show dbs
admin  0.000GB
local  0.000GB
#可以看到有两个默认的数据库，如果要退出客户端可以输入
> exit
bye
```
到这里第一种方式的安装就结束了，整个过程比较简单，可以动手尝试一下好好体会。

## 使用源码安装

运维人员都比较喜欢源码安装，他们觉得可控性会比较强，但现在中小公司项目中普遍使用云服务，所以作为开发者直接连接上就可以用了。我这里使用的源码包实际上是二进制包，也就是说已经被官方编译过了，所以下载二进制包的时候一定要确定你用的操作系统，否则包是不可用的。

* 下载二进制安装包并解压
    ```shell
    cd /usr/local/src
    wget -c https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-3.4.1.tgz
    gunzip mongodb-linux-x86_64-rhel70-3.4.1.tgz
    tar -xvf mongodb-linux-x86_64-rhel70-3.4.1.tar

    #你现在应该可以看到和我相同的文件目录
    .
    ├── bin
    │   ├── bsondump
    │   ├── mongo
    │   ├── mongod
    │   ├── mongodump
    │   ├── mongoexport
    │   ├── mongofiles
    │   ├── mongoimport
    │   ├── mongooplog
    │   ├── mongoperf
    │   ├── mongoreplay
    │   ├── mongorestore
    │   ├── mongos
    │   ├── mongostat
    │   └── mongotop
    ├── GNU-AGPL-3.0
    ├── MPL-2
    ├── README
    └── THIRD-PARTY-NOTICES
    ```
* 创建数据存储目录，默认是在 `/data/db`，如果这个目录不存在则启动时报错。你可以自由创建一个目录 `mkdir -p data/db1`，在启动时指定即可。
* 开始启动服务 `./bin/mongod --dbpath=data/db1`
* 你应该可以看到和我相同的结果
    ```shell
    017-01-08T13:54:55.378+0800 I CONTROL  [initandlisten]
    2017-01-08T13:54:55.378+0800 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
    2017-01-08T13:54:55.378+0800 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
    2017-01-08T13:54:55.378+0800 I CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not recommended.
    2017-01-08T13:54:55.378+0800 I CONTROL  [initandlisten]
    2017-01-08T13:54:55.378+0800 I CONTROL  [initandlisten]
    2017-01-08T13:54:55.378+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/enabled is 'always'.
    2017-01-08T13:54:55.378+0800 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
    2017-01-08T13:54:55.378+0800 I CONTROL  [initandlisten]
    2017-01-08T13:54:55.378+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/defrag is 'always'.
    2017-01-08T13:54:55.379+0800 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
    2017-01-08T13:54:55.379+0800 I CONTROL  [initandlisten]
    2017-01-08T13:54:55.446+0800 I FTDC     [initandlisten] Initializing full-time diagnostic data capture with directory 'data/db1/diagnostic.data'
    2017-01-08T13:54:55.447+0800 I NETWORK  [thread1] waiting for connections on port 27017
    ```
* 细心的你应该发现一个问题，当前服务是前台启动的，如果你想做后续操作那必须再开启一个终端。这样肯定麻烦，那我们也可以采用后台启动的，在启动时必须指定一个日志文件。
    ```shell
    mkdir logs
    touch logs/db1.log
    ./bin/mongod --dbpath=data/db1 --fork --logpath=logs/db1.log
    ```
    

到这里源码安装并启动mongodb就结束了，下面对启动时的可配置参数作一个介绍。

* 基本配置
    ```shell
    --quiet # 安静输出
    --port arg # 指定服务端口号，默认端口27017
    --bind_ip arg # 绑定服务IP，若绑定127.0.0.1，则只能本机访问，不指定默认本地所有IP
    --logpath arg # 指定MongoDB日志文件，注意是指定文件不是目录
    --logappend # 使用追加的方式写日志
    --pidfilepath arg # PID File 的完整路径，如果没有设置，则没有PID文件
    --keyFile arg # 集群的私钥的完整路径，只对于Replica Set 架构有效
    --unixSocketPrefix arg # UNIX域套接字替代目录,(默认为 /tmp)
    --fork # 以守护进程的方式运行MongoDB，创建服务器进程
    --auth # 启用验证
    --cpu # 定期显示CPU的CPU利用率和iowait
    --dbpath arg # 指定数据库路径
    --diaglog arg # diaglog选项 0=off 1=W 2=R 3=both 7=W+some reads
    --directoryperdb # 设置每个数据库将被保存在一个单独的目录
    --journal # 启用日志选项，MongoDB的数据操作将会写入到journal文件夹的文件里
    --journalOptions arg # 启用日志诊断选项
    --ipv6 # 启用IPv6选项
    --jsonp # 允许JSONP形式通过HTTP访问（有安全影响）
    --maxConns arg # 最大同时连接数 默认2000
    --noauth # 不启用验证
    --nohttpinterface # 关闭http接口，默认关闭27018端口访问
    --noprealloc # 禁用数据文件预分配(往往影响性能)
    --noscripting # 禁用脚本引擎
    --notablescan # 不允许表扫描
    --nounixsocket # 禁用Unix套接字监听
    --nssize arg (=16) # 设置信数据库.ns文件大小(MB)
    --objcheck # 在收到客户数据,检查的有效性，
    --profile arg # 档案参数 0=off 1=slow, 2=all
    --quota # 限制每个数据库的文件数，设置默认为8
    --quotaFiles arg # number of files allower per db, requires --quota
    --rest # 开启简单的rest API
    --repair # 修复所有数据库run repair on all dbs
    --repairpath arg # 修复库生成的文件的目录,默认为目录名称dbpath
    --slowms arg (=100) # value of slow for profile and console log
    --smallfiles # 使用较小的默认文件
    --syncdelay arg (=60) # 数据写入磁盘的时间秒数(0=never,不推荐)
    --sysinfo # 打印一些诊断系统信息
    --upgrade # 如果需要升级数据库
    ```
* Replicaton 参数
    ```shell
    --fastsync # 从一个dbpath里启用从库复制服务，该dbpath的数据库是主库的快照，可用于快速启用同步
    --autoresync # 如果从库与主库同步数据差得多，自动重新同步，
    --oplogSize arg # 设置oplog的大小(MB)
    ```
* 主、从参数
    ```shell
    --master # 主库模式
    --slave # 从库模式
    --source arg # 从库 端口号
    --only arg # 指定单一的数据库复制
    --slavedelay arg # 设置从库同步主库的延迟时间
    ```
* 副本集 `--replSet arg # 设置副本集名称`
* 分片
    ```shell
    --configsvr # 声明这是一个集群的config服务,默认端口27019，默认目录
    /data/configdb
    --shardsvr # 声明这是一个集群的分片,默认端口27018
    --noMoveParanoia # 关闭偏执为moveChunk数据保存
    ```
* 卸载
  1. 如果你想卸载上面安装mongodb，那么可以参考下面的步骤。当然这个需求在实际部署时不会有，但是作为我们练习也是有用的。
  1. 要从系统中完全删除MongoDB，你必须删除MongoDB应用程序本身，配置文件和任何包含数据和日志的目录。此过程不可逆，因此请确保在继续之前备份所有配置和数据。
    ```shell
    yum erase $(rpm -qa | grep mongodb-org)
    rm -r /var/log/mongodb
    rm -r /var/lib/mongo
    ```
  1. 如果是采用源码（二进制）包安装，删除解压出来的目录即可（因为我们上面把数据和日志都放进去了），如果你指定在其它位置，那找到相应位置删除即可。
