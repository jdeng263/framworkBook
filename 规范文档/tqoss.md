# 背景

早期，天阙的文件存储在共享NFS或是MFS分布式文件系统中，各个系统的文件都是在各自的项目中实现，管理分散，维护困难，项目上云之后开始出现一些商用的OSS存储，导致在项目迁移过程中，需要进行大面积的改动，每个厂商提供的对接方式不同，兼容性无法保证。也没有安全策略。
解决：
> 1.对内接口一致，透明   
> 2.对外兼容各种厂商的方案   
> 3.安全的文件访问方案  
> 4.统一的运维界面  
> 5.文件的跟踪   
> 6.图片格式的转换   
> 7.流量的分析和统计   
> 8.ngnix映射服务  

# 产品简介
## 产品概述
天阙文件存储服务（Tian Que Object Storage Service，简称OSS），是对第三方提供文件统一管理的服务。用户可以通过调用API实现文件的上传和下载数据对象，并且提供图片预览裁剪等其他功能。文件存储目前支持阿里云的存储、分布式文件系统、共享存储emc三种主流结构。

## 产品架构

![](/assets/oss/oss.png)

## 产品场景
TQOSS作为对象存储提供商，可以广泛应用于各种场景。适用于图片、音视频、日志等海量文件的存储，支持Web网站程序直接向OSS写入或读取数据，支持流式写入和文件写入两种方式，如图所示： 

## 快速入门
1.2.1. 开始使用天阙OSS
天阙TQOSS（Tian Que Object Storage Service）为您提供基于网络的数据存取服务。通过使用TQOSS，您可以通过网络随时存储和调用包括文本、图片、音频、和视频等在内的各种结构化或非结构化数据文件。

## 基本概念介绍
##### 存储空间（Bucket）
存储空间是您用于存储对象（Object）的容器，所有的对象都必须隶属于某个存储空间。对象存储底层为阿里oss时，概念与之一致。而当对象存储在共享存储是，则是一个目录。

##### 项目名（Project）
项目名称，你申请存储的单个项目名称。一个项目只能存在于同一个Bucket下，而一	个Bucket下可以有多个项目。

##### 对象/文件（Object）
对象是TQOSS存储数据的基本单元，也被称为TQOSS的文件。对象上传理论上目前支	持5GB。

##### 访问域名（Endpoint）
Endpoint表示TQOSS对外服务的访问域名。内网Endpoint是internalEndpoint。

##### 访问密钥（AccessKey）
指的是访问身份验证中用到的AccessKey和AccessKeySecret。TQOSS通过使用	AccessKey和AccessKeySecret来验证某个请求的发送者身份。

##### 存储协议（ossProtocol）
指的是在文件存储的协议，目前分为两种OSS（阿里oss）和TQFS（共享存储）

## 申请TQOSS服务秘钥
通过web界面，申请对应 Access Key和Access Secret，申请成功后，客户端需记录项目名project、Access Key和Access Secret，同时还会提供一个burket用来调用sdk。
![](/assets/oss/1.png)


## 阿里oss存储
在使用过程中如果采用阿里oss，那么还需要另外提供阿里的AccessKeyId和AccessKeySecret，还有存储的burket。其中burket与TQOSS的burket是一致的。

## Bucket管理
#### 创建Bucket
Bucket分TQFS和OSS，在选择OSS时需要填写对应阿里oss的相关信息。

#### 删除Bucket
删除Bucket，会将改Bucket里的文件全部删除，所以晋谨慎使用。

#### Bucket属性管理
1.5.3.1. 读写权限
Bucket的读写权限分私有、公共读和公共读写

##### 静态网站
默认404页面。填写本Bucket的一个文件相关路径，仅支持html、jpg、png、bmp、webp格式的文件，如果为空则不启用默认404页设置。

##### 日志管理
默认不存储，如果需要存储日志，会将文件访问的信息保存在想要的文件内。

#####  防盗链
防盗链设置，针对页面访问，通过sdk访问并不会进行校验。
目前支持*和?的形式：*后N位的任意匹配，?后一个字母的任意匹配。

##### 生命周期
对象的生命周期设置，设置后期用，会自动删除过期对象，不能回复。

## java-sdk使用
1.6.1. 安裝
环境准备：适用于jdk1.6及以上版本。
在maven环境引用：
```
    <dependency>
        <groupId>com.tianque</groupId>
        <artifactId>tq-oss-sdk</artifactId>
        <version>0.1.0-SNAPSHOT</version>
    </dependency>
```
非maven环境引入:
1. 下载Java SDK开发包版本号 version：javatq-oss-sdkversion.zip；
2. 解压该开发包；
3. 将解压后文件夹中的文件： tq-oss-sdk-.jar 以及lib文件夹下的所有文件拷贝到您的项目中；
4. 在Eclipse中选择您的工程，右击 -> Properties -> Java Build Path -> Add JARs,选中您在第三步拷贝的所有JAR文件；

#### 初始化Client
endpoint和internalEndpoint：由天阙提供服务地址，下面以192.168.1.1:8080为例。
project、accessKey、accessSecret和bucket：在申请TQOSS服务后获得
ossProtocol：确定存储协议

#### 构造函数创建
``` java
String endpoint = "192.168.1.1:8080";
String internalEndpoint = "192.168.1.1:8080";
String accessKey = "<yourAccessKey>";
String accessKeySecret = "<yourAccessKeySecret>";
String project = "test";
OssProtocol ossProtocol = OssProtocol.TQFS;
String bucket = "bucket"

ClientConfiguration clientConfiguration = new ClientConfiguration(endpoint, internalEndpoint, project, bucket,accessKey, accessSecret, ossProtocol);
TQOSSClient tqOssClient = new TQOSSClient(clientConfiguration);
```
#### 配置文件创建
此时此时需要在项目classes根目录下创建一个tqoss.properties：
```java
tqoss.project=test
tqoss.ossProtocol=TQFS
tqoss.endpoint=192.168.1.1:8080
tqoss.burket=burket
tqoss.accessKey=<yourAccessKey>
tqoss.accessSecret=<yourAccessKeySecret>
```
构造函数与配置文件的优先级，如果同时存在以构造函数的参数为准，参数无效再获取配置文件信息（此时若无配置文件则会抛出异常）。 使用ClientConfiguration设置OSSClient参数代码如下：
```java
ClientConfiguration clientConfiguration = new ClientConfiguration();

// 设置OSSClient使用的最大连接数，默认1024
clientConfiguration .setMaxConnections(200);
// 设置请求超时时间，默认50秒
clientConfiguration .setSocketTimeout(10000);
```

#### 上传文件
上传文件主要分两种：普通文件上传和临时文件上传。区别在于临时文件作用于临时存储，具有时效性超过一定时间就会删除，通常用于表单提交分布操作过程中，在表单正式提交时再将临时文件提交到正式目录。

#### 普通上传文件
```
//文件形式
tqOssClient.putObject(new FIle("..//test.txt"),"test.txt");

//上传网络流
URL url = new URL("http://www.wocloud.cn/u/cms/www/201404/10092746o3z3.png");
url.openConnection().getContentLength();
InputStream inputStream = url.openStream();
//类似网络流的形式必须要提供大小（注：不要用available，值不一定是不对的）
tqOssClient.putObject(inputStream, "test.png", url.openConnection().getContentLength());

//上传文件流
tqOssClient.putObject(new FileInputStream("..//test.txt"),"test.txt",new FIle("..//test.txt").length());
上传文件返回结果

Result [
//相对路径
ossUri=png/2016/8/24/174951224745.png, 
//绝对路径
ossUrl=TQFS://burket/test/png/2016/8/24/174951224745.png, 
//唯一标示
etag=VFFGUzovL3Rlc3QvemhlamlhbmdzaGVndWFuL3BuZy8yMDE2LzgvMjQvMTc0OTUxMjI0NzQ1LnBuZw==, 
//文件大小
size=10694]
1.6.3.2. 临时文件上传
putTmpObject与普通文件上传形式一样，返回结果也一致。
submitTmpObject(String uri)：uri为相对目录。
```

#### 文件下载
```
getObjectByUri(String uri)：根据相对路径获取
getObjectByUri(String uri)：根据绝对路径获取
返回文件流和文件的相关信息。
```

#### 文件拷贝

```
copyObject(String fromUri)：uri为相对目录，将该目录的文件拷贝一份返回拷贝后的文件信息。

```

#### 文件删除

```
deleteObject(String uri)：uri为相对目录
```

#### 图片裁剪

```
ImageOperationParam imageOperationParam=new ImageOperationParam(imgWidth, imgHeight, cropX, cropY, cropWidth, cropHeight);
PutObjectResult result = tqOssClient.clip(uri, imageOperationParam);
result=tqOssClient.submitTmpObject(result.getOssUri(),"headerImg");
1.6.8. url访问
http://192.168.1.1:8080/tqOssManager/getObjectByUri/png/2016/8/24/174951224745.png?auther=<message>  
auther通过clientConfiguration.getAutherHeader()编码获得。
```

#### 图片的特殊处理
正常图片访问
```java
http://192.168.1.1:8080/tqOssManager/image/view/518aa339babd035251.jpg?auther=<message>

```
![](/assets/oss/2.png)
将图缩略成宽度为100，高度为100，按短边边优先，将图片保存成jpg格式
```
http://192.168.1.1:8080/tqOssManager/image/view/518aa339babd035251.jpg?w=200&h=200&auther=<message>
```
![](/assets/oss/3.png)

旋转,ro为顺时针的翻转度数
```
http://192.168.1.1:8080/tqOssManager/image/view/518aa339babd035251.jpg?ro=90&w=500&h=500&auther=<message>
```
![](/assets/oss/4.png)

#### 文字水印
```
http://localhost:8080/tqOssManager/image/view/518aa339babd035251.jpg?wm=天阙科技,40,255-255-0,2-2&ro=90&w=500&h=500&auther=<message>
```
#### wm水印参数
第一个参数水印的文字，如天阙科技
第二个参数文字的大小，如40
第三个参数颜色rgb，如255-255-0
第四个参数位置，如2-2（分别对应水平的左中右，纵向的上中下，此时为右下角）
![](/assets/oss/5.png)