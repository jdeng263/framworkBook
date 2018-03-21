# 背景

各级部门分别有各自独立的系统，虽拥有众多有价值的数据，但因数据分散与重复、建设标准的不统一、更新机制的不完善，增加了数据利用的难度。为更有效的发挥数据对工作能力的提升，建立数据中心，打破信息封闭，消除信息”荒岛”和”孤岛”。

# 架构

采用分布式集群的方式搭建的平台，分为ETL采集端、中心管理系统、订阅服务端三层

![](/assets/datacenter/datacenter.png)


# 产品特点

> 1.数据交换  
> 2.数据清洗  
> 3.数据整合    
> 4.数据专题  
> 5.数据服务  
> 6.实例管理   
> 7.血缘关系  
> 8.数据管控  
> 9.安全审计  


# 快速入门

### 项目依赖

> 1.jdk 8
>
> 2.memcached
>
> 3.tomcat 8
>
> 4.zookeeper
>
> 5.activeMQ
>
> 6.oracle 11
>
> 7.mongodb

### 内部项目依赖

> 1.配置中心（项目）（可选）
>
> 2.发布平台（项目）（可选）
>
> 3.TQ-OSS文件服务（项目）

### 数据中心服务包

> datacenter-manager-2.x（数据中心配置管理系统）
>
> datacenter-etcl-2.x（数据中心采集节点）
>
> datacenter-dataGateway-2.x（数据中心数据网关）

## 系统部署

### datacenter-manager部署

1、 下载datacenter-manager源代码，通过并修改`filter-product.properties` 配置文件，按需修改。

```properties
## 配置中心
## Use tq-config service, set <code> zk.config.ingore=true </code> to unuse it.
zk.config.ingore=true
## The tq-config service's zookeeper address and port, 
##		use it like <code> zk.config.hosts=192.168.1.1:2181 </code>,
##		if zookeeper is multi cloud, use ',' separate configuration.
zk.config.hosts=10.65.1.34:2181
## The node configuration of this project on the tq-config service.
zk.config.node.path=/PRODUCT/datacenter/1.0

## 文件下载的临时存储
local_stored_file_root=/home/admin/dataCenterAttachmentFiles

## 缓存
memcached.servers=localhost:11211

## Zookeeper 注册地址，内部服务使用
etcl.dubbo.zkserver=192.168.1.160:2181?backup=192.168.1.161:2181,192.168.1.162:2181

## Zookeeper 注册地址，提供给云搜索使用（可以与etcl服务的不同一个注册中心）
search.dubbo.zkserver=192.168.1.160:2181,192.168.1.161:2181,192.168.1.162:2181
## 首页云搜索按钮链接
cloud.search.url=http://localhost:8211/gotoCloudSearchPage

## ActiveMQ 消息队列配置
activeMQ.address=tcp://localhost:61616
activeMQ.producerWindowSize=102400000

## SMS 与短信发送代理之间的通信地址.
## Receiving SMS sending status, use this project deploy IP and give a port.
sms.accept.port=29098
## SMS sending server IP
sms.message.ip=
## SMS sending server Port
sms.message.port=29099

## 数据中心主应用数据存储
jdbc.driver=oracle.jdbc.driver.OracleDriver
jdbc.url=jdbc:oracle:thin:@localhost:1521:zjyun
jdbc.username=datacenter
jdbc.password=datacenter

## 数据中心前置到数据中心数据存储配置信息
jdbc.marketdata.driver=oracle.jdbc.driver.OracleDriver
jdbc.marketdata.url=jdbc:oracle:thin:@localhost:1521:zjyun
jdbc.marketdata.username=datamarket
jdbc.marketdata.password=datamarket

## 数据中心标准数据配置信息
jdbc.centerdata.driver=oracle.jdbc.driver.OracleDriver
jdbc.centerdata.url=jdbc:oracle:thin:@localhost:1521:zjyun
jdbc.centerdata.username=datarepository
jdbc.centerdata.password=datarepository

## TQOSS 文件系统，配置服务地址、burket名称以及访问秘钥，联系tqoss索取
tqoss.project=zjdatacenter
tqoss.ossProtocol=TQFS
tqoss.endpoint=10.65.1.37:20000
tqoss.burket=zjdatacenter
tqoss.accessKey=ITUgYvKHp9FhdA9m
tqoss.accessSecret=JH2AWdo9ftd5nBP6DoI7mGZVPXEhDHIo

## mongodb 服务地址，集群以','分隔
mongodb.replicaSet=192.168.1.101:27017,192.168.1.102:27017,192.168.1.103:27017

```

2、初始化项目，通过以下脚本完成应用管理节点的初始化

```shell
$ mvn clean install -Pproduct
$ mvn exec:exec -PinitApp
```

3、启动项目

```shell
$ mvn tomcat7:run -Pproduct -Dport=8080
```

4、在项目启动后，可在节点管理中绑定etcl执行节点

### datacenter-etcl部署

1、 下载datacenter-manager源代码，通过并修改`filter-xxx.properties` 配置文件，按需修改。

​	*注：xxx为如果有多个etcl节点，需要拷贝其中一个并修改名称和配置信息，因为etcl执行节点并非集群化部署，而是以不同的单节点的形式独立存在。

```properties
## The name of group for <dubbo:service group/>, each datacenter-etcl service must have a different name.
etcl.dubbo.group=xxx

## ActiveMQ 消息队列配置，与manager节点异步通信
activeMQ.address=tcp://localhost:61616

## 前置到数据中心数据存储，配置同manager节点的`jdbc.marketdata.*`一致
jdbc.market.*

## 标准数据存储库，配置同manager节点的`jdbc.centerdata.*`一致
jdbc.repository.*

## TQOSS 同manager节点一致
tqoss.*

## mongodb 配置同manager节点一致
mongodb.replicaSet=
```

2、初始化依赖配置表，标准数据存储库中，建表文件位于项目位置，`target\classes\database\tables\003_datarepository.sql` ，复制脚本并手动执行。

3、启动项目

```shell
$ mvn tomcat7:run -Pproduct -Dport=8101
```

4、在manager管理页面中新增节点"**基础配置  > 节点配置  > 管理节点 > 新增**",选择当前etcl节点对应的group，完成新增。

### datacenter-dataGateway部署

1、 下载datacenter-manager源代码，通过并修改`filter-product.properties` 配置文件，按需修改。

```properties
## 缓存
memcached.servers=localhost:11211

## ActiveMQ 与manager节点通信
activeMQ.address=tcp://localhost:61616

## 数据中心主应用数据存储，与manager节点配置一致，读取相关统计配置信息
jdbc.driver=oracle.jdbc.driver.OracleDriver
jdbc.url=jdbc:oracle:thin:@localhost:1521:zjyun
jdbc.username=datacenter
jdbc.password=datacenter

## 标准数据存储库，配置同manager节点的`jdbc.centerdata.*`一致
jdbc.centerdata.*
```

2、启动项目

```shell
$ mvn tomcat7:run -Pproduct -Dport=8090
```  