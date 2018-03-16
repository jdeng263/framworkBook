# 万达城网关平台对接文档

## 一、 概述

*  说明:

    >  网关后台账号的授权：在用户中心统一授权。
    >  账号的服务权限： 网关平台的账号同资源市场保持一致，  当然也在网关这边提供了单独的添加账号的功能。
    >  使用SHA1WithRSA校验，可用支付宝工具生成RSA 公钥、私钥，  将公钥提交到天阙网关平台。如使用MD5, SHA1校验仅需约定加密KEY即可
    >  网关的应用添加，服务的注册，服务的订阅，都在资源市场进行。
    >  网关后台页提供了应用的添加和服务的注册功能（如果同资源市场服务定义的apppath+grouppath+methodpaht不一致，那么在资源市场上该服务修改后，会覆盖网关的定义）。
    
## 本文主要讲访问的服务的调用，后台配置管理。 不涉及资源市场。
    



## 二、 接口规范

### 接口URL： 
http://xxx/api/${apppaht}/${grouppaht}/${methodName}.${format}


路径参数 | 说明 
------------ | ------------- 
appname | 请求的模块名称
groupname | 请求的接口名称
methodName | 请求的接口的具体方法
format |  返回数据的格式， 默认为json,

**Coding，eg**


### 接口参数
参数名 |  类型	| 是否必填 | 最大长度 | 描述 |  示例值
--------|--------|--------|--------|--------|--------
requestId | String |  否  | 100 | 调用方请求的标识,用于日志记录和跟踪 |  cj123324234  
**appKey** | String  | 是  | 100 | 调用方申请的应用编号 |  tianqueapp1
**sign** | String | 是  | 100 | 签名字段 |  MbFTAA+vZBRdViffAQSgXAKAq6Ye/8J0=
version | String  | 否  | 100 | 接口版本号 |  1.0 
callType | String  | 否  | 100 | 内部调用类型（可忽略） |  http, dubbo
charset | String  | 否  | 100 | 返回的字符集编码，默认UTF-8, 目前只支持UTF-8 |  UTF-8
timestamp | long  | 否  | 100 | 距离1970年1月1日0点0分0秒的毫秒数 |  1495525700681
notifyUrl | String  | 否  | 100 | Third Header |  接口调用成功后回调通知的地址（需要业务接口支持）| http:/abc.com.cn/test.html
**bizContent**  | String  | 否  | 不限 | **业务请求参数的集合，最大长度不限，除公共参数外所有请求参数都必须放在这个参数中传递, 格式为json的字符串** | {id:1213,"name":"dsfafd"}

### sign 生成规则： 
 
 加密方式  |  是否推荐 | 算法描述
--------|--------|--------
MD5| String |  否  |  MD5(appKey=${appKey}&bizContent=${bizContent}${用户密钥}) 
SHA1| String |  否  | SHA1(appKey=${appKey}&bizContent=${bizContent}${用户密钥})
SHA1WithRSA| 是 |   200 |  RSA（SHA1(appKey=${appKey}&bizContent=${bizContent})，${用户私钥}）， 平台使用用户公钥解密

RSA工具可以在此下载：https://doc.open.alipay.com/docs/doc.htm?spm=a219a.7386797.0.0.D5ngWa&treeId=291&articleId=106097&docType=1


### 返回值
在出错时，返回格式统一为  {success:false,errorCode:xxx, errorInfo:xxx}
如果成功，返回数据为服务提供者的数据的原样格式。

 错误码     |  描述   
------------|--------------
000| url出错
001| 缺少网关参数，如appkey, sign
002| sign 校验出错
003| 服务无权限
004| 服务不存在
005| 服务已关闭
006| 只支持post
007| 网关服务定义出错
101| 参数长度出错
102| 产生格式出错
103| 缺少业务参数
201| 内容服务调用异常，  这类错误都是服务提供者的错误。
202| 网关未知异常



## 三、 后台服务设置

* 说明

    >  服务的注册包含：服务的定义，和参数的定义两个部分。
    >  服务的支持的提供方协议为两种http 和dubbo .

服务添加的入口在菜单“接口管理”， 添加服务的第一步是选择服务的协议。

### 3.1 为http服务提供者添加网关接口。 （包括http get , http post , http body）

##### 3.1.1 服务添加表单说明
字段名    |  描述   
------------|--------------
服务名| 
应用code| 提供者的应用code,用于生成网关接口url 路径
服务分组| 提供者的应用下的分组,用于生成网关接口url 路径
方法名| 提供者的应用下的方法名e,用于生成网关接口url 路径
服务主机| 服务提供者的主机地址，可多个,逗号分隔，网关采用随机方式选取。例如（192.168.1.2:8080,192.168.1.3:8080 ）
服务地址| 提供者的服务地址（不包含ip和端口）例如：   /clue/addclue，   服务主机+服务地址，组合就是提供服务的内部url.
响应时间| 提供者的服务超过一定实际无响应的话，网关自动断开同提供者之间的连接请求
缓存时间| 对同一调用者，相同接口，相同参数的结果缓存的实际，默认无缓存。  主要：缓存生效，会导致实际提供者的接口不被调用，而直接以网关缓存结果代替
脱免字段| 过滤掉提供者返回值中的响应字段
服务等级| 默认3， 在网关服务压力过大时，会限制低等级的就服务。
重试次数| 在网关服务调用提供者服务出错时，尝试的次数，
   
##### 3.1.2 服务参数添加

字段名     |  描述   
------------|--------------
参数名| 需要同提供者上的实际参数名保持一致
参数类型| 参数类型包括6种： String, Int,Long, Double,Float,Boolean 
参数位置| 包括query（表单参数）,body（非表单提交时参数，只支持json解析）,header(http头参数),path(url 路径上的参数)  
次序|  参数次序，对提供者是http服务无效
最大值 | 仅仅对数值类型参数有效 
最小值 | 仅仅对数值类型参数有效
最大长度：| 仅仅对字符串型参数有效
参数校验：|正则表达式校验， 仅仅对字符串型参数有效
json验证：：| 仅仅对字符串型参数并且诶json 格式的，可以使用json validate schema定义


### 3.2 为dubbo服务提供者添加网关接口。
##### 3.2.1  服务添加表单说明
字段名    |  描述   
------------|--------------
服务主机| 同http服务添加的唯一区别，这里选择的是zookeeper服务中心的地址。例如（192.168.1.2:2181,192.168.1.3:2181 ）
…… | ……


##### 3.2.2 服务添加表单说明
        
字段名     |  描述   
-----------|--------------
参数名| 不需要同dubbo就上的实际参数名保持一致
参数类型| 参数类型包括6种： String, Int,Long, Double,Float,Boolean ， 党"dubbo参数类型"为pojo或list时，这里的参数类型需要设置为String
Dubbo参数类型|必须为包装类型， 如果不填写那么同参数类型。 这里只能填写 6种基本类型 + java.util.Date+pojo
是否为数组| 表示参数是否为数组
次序|  参数次序，这个特别重要     
        

		1、当接口协议为dubbo 时:参数类型说明：

			a、dubbo参数为java基本对象， 请使用包装类型，	   包括： Integer,Float Boolean,Double,Long。 
			
			b、dubbo接口中的char 类型请使用String 代替,不支持枚举类型。
			
			c、dubbo接口中的参数为集合类型，只支持java.util.List, list 中对象可以是基本对象，  Date或pojo.  不支持set 和map类型的参数，也不支持list 套list.
			
			d、dubbo接口中的参数为pojo对象时，对象属性可以是另外一个pojo
			
			e、dubbo接口参数日期类型,暴露http时全部使用时间戳格式
			
			f、dubbo接口参数为pojo,或list其通过网关映射出去的http参数类型都是String 
			
			g、参数的次序需要严格匹配
			
		举例： dubbo接口有3个产生 ：
		    
		    createDate(javaUtilDate),
		    
		    age(java.lang.Integer),  
		    
		    user(com.tianquer.UserVO), 
		    
			eventList(java.util.List,  list中元素为com.tianque.EventVO)
			
		则 :
		
	    	       参数1 createDate:  参数类型  Long,  Dubbo对象类名：java.util.Date,  是否数组为否。
	    	       
	    	       参数2 age:         参数类型  Int,   Dubbo对象类名可为空 （表示同参数类型相同），是否数组为否。
	    	       
	    	       参数3 user:        参数类型  String,Dubbo对象类名 com.tianquer.UserVO, 是否数组为否。
	    	       
	    	       参数4 eventList:      参数类型  String,Dubbo对象类名 com.tianque.EventVO, 是否数组为是。
	    