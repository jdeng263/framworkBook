# 调用链接入说明

[TOC]

# 1、修改pom.xml文件
加入调用链的maven坐标

```

<dependency>
	<groupId>com.tc.mw</groupId>
	<artifactId>trident</artifactId>
	<version>1.1.1</version>
	<exclusions>
		<exclusion>
			<groupId>org.apache.activemq</groupId>
			<artifactId>activemq-all</artifactId>
		</exclusion>
	</exclusions>
</dependency>

```

注意：如果原来的项目使用了javassist，则要排除依赖，建议使用javassist-3.19.0-GA.jar这个版本（经测试）


# 2、修改env目录下面的的filter-dev、filter-test、filter-product文件

增加trident.broker.url配置项，该配置项的值为发送mq地址，如
trident.broker.url=localhost

注意：这里不用写端口号，默认端口号61613（使用stomp协议发送消息），低版本activemq需要修改配置文件开启stomp监听

# 3、新增trident.properties
在src\main\resources下新增trident.properties配置文件，内容如下：

```


trident.app.name=projectName
trident.broker.url=${trident.broker.url}
trident.broker.port=61613
trident.local.queue.size=20
trident.performance.queue.name=/queue/call-info
trident.status.queue.name=/queue/heartbeat-info

```

注意：trident.app.name要改为项目名称


# 4、dubbo方式接入
新增DubboTridentFilter类


一般在放在com.tianque.dubbo.filter包下,并修改类中的appName（改成自己的项目名称即可）

在META-INF目录下建立dubbo\internal\com.alibaba.dubbo.rpc.Filter路径和文件并增加配置项dubboTridentRPCFilter，如
dubboTridentRPCFilter=com.tianque.dubbo.filter.DubboTridentFilter
在提供dubbo服务配置的xml（如applicationContext-dubbo.xml）中,加入dubboTridentRPCFilter如

```



<dubbo:provider protocol="dubbo" timeout="5000" filter="dubboTridentRPCFilter,dubboRPCFilter"/>

```

注意：两边配置的路径中配置的完全一致


# 5、web接入方式
在web.xml中加入filter配置

```
<filter>
	<filter-name>statFilter</filter-name>
    <filter-class>com.tc.trident.web.StatFilter</filter-class>
    <init-param>
    	<param-name>statStore</param-name>
        <param-value>com.tc.trident.store.StompStatStore</param-value>
 	</init-param>
</filter>
<filter-mapping>
	<filter-name>statFilter</filter-name>
    <url-pattern>/sysadmin/*</url-pattern>
</filter-mapping>

```

注意：url-pattern根据具体需要修改


# 6、启动项目

使用tomcat8（tomcat7有问题）启动项目，增加启动参数：
-javaagent:D:\mvnRepo\com\tc\mw\trident\1.1.1\trident-1.1.1.jar -Xbootclasspath/a:D:\mvnRepo\org\javassist\javassist\3.19.0-GA\javassist-3.19.0-GA.jar -DtridentMonitorIsOpen=true

注意：修改成指定路径
