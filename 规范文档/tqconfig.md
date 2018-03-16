
<html>
<H1>置中心开发者说明</H1>
</html>



****

[TOC]

## 配置中心介绍


Distributed Configuration Management Platform(分布式配置管理平台)

专注于各种「分布式系统配置管理」的「通用组件」和「通用平台」, 提供统一的「配置管理服务」

![](/assets/config/config.jpg)

### 主要目标：
> 部署极其简单：同一个上线包，无须改动配置，即可在 多个环境中(RD/QA/PRODUCTION) 上线   
> 部署动态化：更改配置，无需重新打包或重启，即可 实时生效
> 统一管理：提供web平台，统一管理 多个环境(RD/QA/PRODUCTION)、多个产品 的所有配置   
> 核心目标：一个jar包，到处运行  

1. 配置中心分为“开发”，“测试”，“生产” 3类环境， 每个 project只能属于一个环境.   
2. oject会有多个版本的配置属性列表。 
3. 属性，以及 TqConfigFactoryBean的localPropertyPath对于的本地配置文件的属性，优先级别高于配置中心值。 
4. 接入方通过ZkConfigProperty获取的属性，能够实时监听来自配置中心值的动态变化。 
5. spring 配置文件中注入的属性，目前不自从变更监听， 只有在应用重启是才重新获取。 
6. 说明： 配置中心分为“开发”，“测试”，“生产” 3类环境， 每个 project只能属于一个环境. 

## 接入步骤 

### 1.接入的应用，需要应用jar: 
``` 
<dependency>
   <groupId>com.tianque.tqconfig</groupId>
   <artifactId>tqconfig-client</artifactId>
   <version>0.0.1</version>
</dependency>
			
```
### 2.spring 环境添加 tqConfigFactoryBean和 PropertyPlaceholderConfigurer 
```    
<bean id="tqConfigFactoryBean" class="com.tianque.tqconfig.client.TqConfigFactoryBean">
	<property name="zkConfigHosts" value="{zk.config.hosts}" ></property>
	<property name="zkConfigNodePath" value="{zk.config.node.path}" ></property>
	<property name="localPropertyPath" value="{zk.config.localproperty}" ></property>	
</bean>	
		
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
	  <property name="properties">
<bean id="tqConfigPropertys" class="java.util.Properties"								factory-bean="tqConfigFactoryBean" factory-method="getProperties">
</bean>
</property>
</bean>
				

```
如上可以看到3个配置属性 ： 
~~~~
1）zk.config.hosts， 代表 zk服务器的地址 ， 
2）zk.config.node.path 代表 zk上配置数据的经典： 格式 ：/{env}/{projectname}/{version} ,例如zk.config.node.path=/dev/zhejiang-alicloud/1.0，表示加载开发环境的zhejiang-alicloud项目的版本号码“1.0”的所有配置数据 
3）zk.config.localproperty 代表引用本地classPath下的属性文件路径，例如： 
~~~~


### 3.filtet-xxx.properties 中需要添加的属性
至少包含下面3个属性, 其他属性都可以在配置中心获取。 

```
zk.config.node.path=/dev/zhejiang-alicloud/1.0
zk.config.hosts=127.0.0.1:2181
zk.config.localproperty=grid.properties

```
     
					
				


### 4.在启动java进程的脚本中设置系统参数
zkConfigNodePath 时， 那么这个 zkConfigNodePath的值将优先与 、filtet-xxx.properties中的zk.config.node.path   
例如 java Test.java -DzkConfigNodePath=/dev/zhejiang-alicloud/1.1 


### 5.动态日志级别临时调整
~~~~
a) 目标：原来依赖配置中心应用，无需添加任何额外配置项，都可以在线修改日志级别 
b) 应用重新启动是不受临时级别的影响，只有在重新修改配置中心值时，才会充分日志级别临时改动 
c)目前支持的框架为slf4j： 支持的日志实现有logback 和log 4j 两个日志实现。 
d) 属性格式 KEY:“tll@$packagename ”, VALUE: “$LEVEL@$ipArray” 其中tll前缀的含义是：“tmp log level” 
e)举例: 
No	key	value	描述  
例1	tll@com.tianque	DEBUG#192.168.1.1, 192.168.1.2	动态修改 com.tianque 包下日志级别为DEGUG，影响范围机器192.168.1.1，　192.168.1.２  
例2	tll@	INFO#192.168.1.1, 192.168.1.2	动态修改根日志级别为INFO，影响范围该应用部署的机器192.168.1.1，　192.168.1.２
例3	tll@com.tianque	DEBUG	动态修改 com.tianque 包下日志级别为DEGUG，影响范围该应用的所有部分机器
~~~~

