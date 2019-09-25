!>  **当前版本**：[UWS服务开发规范v1.3]() 

 **发布时间**：2018年9月3日


## 1引言

### 1.1执行摘要

?> UWS即U+ Web Service，是云平台的能力开放层

### 1.2编写目的

?> 为第三方应用程序开发提供基础和指导。

### 1.3术语和定义

?> 暂无

### 1.4参考资料

?> 本设计所参考的相关文档、标准等资料如下：

- 云平台UWS的签名规范
- 云平台运维规范.doc
- 云平台信息安全规范v1.3.pdf

###  1.5特殊记号格式说明

?> 本文档暂未使用其他特殊文档记号或格式。</br>
本文中的接口示例均使用的是示例数据并非真实数据。

## 2接口定义

### 2.1URI

**{协议}:**

`//{域名}/{app-name}/{version}/{domain}/{rest-convention}`

**{协议}：**

HTTPS为对外提供服务的统一使用协议，默认使用443端口， 在服务端使用TSL进行单向加解密处理。服务调用方在进行调用时，无需下载或安装证书。

特殊服务要使用http协议的在服务开发时商定。

**{域名}:**

国内服务用`uws.haier.net`，

国外北美服务用`uws-gea-us.haieriot.net`，

国外欧洲服务用`uws-euro.haieriot.net`；

**{app-name}:**

应用简称，代表发布的服务名称；

U+平台已有的uws服务应用名。（后续新增uws服务的应用名参见资源申请章节。）

|序号|	服务名称|	应用名|	备注|
:-:|:-:|:-:|:-: 
1|	[账户服务](zh-cn/Account)|	N/A	|集团690用户中心提供
2|	[设备管理](zh-cn/DeviceManage)	|uds、stdudse、udse|	设备注册、设备管理、设备控制
3|	[数据订阅](zh-cn/DataSubscription)|	无|	设备数据订阅、应用数据订阅
4|	[家庭模型](zh-cn/FamilyManage)	|ufm、ufme|	家庭成员管理、家庭设备管理
5|	[场景引擎](zh-cn/IFTTT)|	iftttscene|	场景创建、场景执行、场景日志管理
6|	[云定时](zh-cn/Scheduler)|	scheduler|	设备预约控制、场景预约控制  
7|	[设备影子](zh-cn/DevicesShadow)|	shadow|	设备云状态查询、预期设备控制管理
8|	[消息推送](zh-cn/MessagePush)|	ums、umse|	设备消息推送、APP消息推送、语音消息推送
9|	[设备资源云存储](zh-cn/CapacityService_DeviceCloudStorage)|	css|	设备图片/视频存储、查看 
10|[账户授权](zh-cn/Session)|uaccount|应用OAuth授权，用户授权管理


**{version}:**

代表api的版本信息；将版本号放入URL中，需要注意版本规划，以便以后API的升级和维护。不要发布无版本的API，版本使用”v+简单数字 ”进行标识，避免小数点。如：v1、v2等，不要使用v1.5或v2.0。

**{domain}：** 

是一个你可以用来定义任何技术的区域(例如：安全-允许指定的用户可以访问这个区域)或者业务上的原因(例如：同样的功能在同一个前缀之下)。

**{rest-convention}：**

 代表这个域(domain)下，约定的rest接口集合。


- 1.关于分隔符"/"的使用</br>
"/"分隔符一般用来对资源层级的划分。对于REST API来说，"/"只是一个分隔符，并无其他含义。为了避免混淆，"/"不应该出现在URL的末尾。

- 2.URI中尽量使用连字符"-"代替下划线"\_"的使用</br>
连字符 "-"一般用来分割URI中出现的字符串(单词)，来提高URI的可读性。使用下划线"_"来分割字符串(单词)可能会和链接的样式冲突重叠，而影响阅读性。
 
- 3.URI中统一使用小写字母</br>
根据RFC3986定义，URI是对大小写敏感的，所以为了避免歧义，我们尽量用小写字符。（平台已有带大写字母的接口服务URI维持不变，新开发服务按此规范执行。）

- 4.URI中不要包含文件(脚本)的扩展名</br>
例如 .php .json 之内的就不要出现了，对于接口来说没有任何实际的意义。如果是想对返回的数据内容格式标示的话，通过HTTP Header中的Content-Type字段更好一些。

- 5.URI中的名词表示资源集合，使用复数形式。

- 6.避免层级过深的URI</br>
在url中表达层级，用于按实体关联关系进行对象导航，一般根据id导航。过深的导航容易导致url膨胀，不易维护，如 GET /zoos/1/areas/3/animals/4，尽量使用查询参数代替路径中的实体导航，如GET /animals?zoo=1&area=3；

###  2.2HTTP动词

常用的 HTTP动词有GET、POST、PUT、DELETE，为了安全及后期服务监控、控制、取参的便利性，UWS接口统一使用 POST动词，所有参数放到 HEADER或 BODY中进行传递，不要放到 URL中进行传参。

### 2.3媒体类型

对于请求UWS规定用以下三种媒体类型：

#### 2.3.1 Content-Type: application/json

UWS服务与应用的交互数据接口统一为基于JSON的REST-RPC接口
```
POST /v1/animal HTTP/1.1
Host: uws.haier.net
Accept: application/json
Content-Type: application/json
Content-Length: 24

{   
  "name": "Gir",
  "animalType": "12"
}
```

#### 2.3.2 Content-Type: multipart/form-data

适用于表单文件上传

#### 2.3.3 Content-Type:application/octet-stream

适用于文件的上传下载


### 2.4 响应格式

UWS定义的接口统一使用JSON格式进行数据响应，在响应对象的属性中必须包含retCode（响应码）和retInfo（响应描述）信息。如下所示：
```
 HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: xxx

{
    "retCode": "00000",
    "retInfo": "成功",
    "employees": [
        {
            "firstName": "John",
            "lastName": "Doe"
        },
        {
            "firstName": "Anna",
            "lastName": "Smith"
        },
        {
            "firstName": "Peter",
            "lastName": "Jones"
        }
    ]
}
```

如上面的示例中所示，响应中必须包含retCode和retInfo属性，且这两个属性必须属于对象的直接子属性； JSON对象中其它属性和结构，各服务可根据自己情况自定义。

#### 2.4.1 请求响应成功

对请求正常处理的统一响应成功信息如下：

	"retCode":"00000"，"retInfo":"****成功"


#### 2.4.2 请求响应失败
对请求异常处理的响应对应的"retCode"错误码和"retInfo"错误说明。</br>
下附已定义的错误码，请在接口服务开发中选择使用；若错误码不满足实际的服务开发，**请向U+云平台智能服务团队（lijingtao@haier.com 李京涛）申请分配**（详见资源申请章节）。

**附件：**[U+云平台已定义错误码][ErrorCode]


#### 2.4.3 响应数据规范
##### 数据类型限定
限定类型|说明|格式|json示例
:-:|:-|:-|:-
DataTime|日期时间类型的字符串|`yyyy-MM-dd hh:mm:ss`|`{“lgTime”:“2013-10-08 08:00:00”}`
Data|日期类型的字符串|`yyyy-MM-dd hh:mm:ss`|`{“lgDate”:“2013-10-08”}`
String|字符串||`{“address”:“street 123”}`
Int|整型数字| |`{“age”:1234}`
Long|长整型数字| |`{“oid”:1234567890123}`
double|浮点型数字| |`{“price”:12.35}`
boolean|布尔型（true或false）| |`{“idOld”:true}`

###### null值说明

为避免解析错误， uws各接口的返回参数，不返回null值。

必填参数，无论输入还是输出，必须有值，不能为null。

非必填参数，则说明如下：


1.数值类型数据（int、long、double）只会返回数字，包括正数、零及负数；

布尔类型数据（boolean）只返回true和false；

以上基本类型本身不包含null值。

2.DateTime、Date、String及结构体类型的数据，如果为null时，所对应的属性将不返回。


例如：

**DataTime类型**

birthday为DateTime类型，不为null时：
	
`{"name":"Tom","age":23,"birthday":"2013-10-08 08:00:00","address":{"city":"beijing","street":"haidian"} }`

birthday为null时，则birthday属性不返回：
	
`{"name":"Tom","age":23, "address":{"city":"beijing","street":"haidian"} }`

**String类型**

name是String类型，不为null时：

`{"name":"Tom","age":23,"birthday":"2013-10-08 08:00:00","address":{"city":"beijing","street":"haidian"} }`

name为null时，则name属性不返回

`{"age":23,"birthday":"2013-10-08 08:00:00","address":{"city":"beijing","street":"haidian"} }`

**结构体类型**

address为结构体类型，不为null时：
	
`{"name":"Tom","age":23,"birthday":"2013-10-08 08:00:00","address":{"city":"beijing","street":"haidian"} }`

address为null时，address属性不返回
	
`{"name":"Tom","age":23,"birthday":"2013-10-08 08:00:00" }`


### 2.5 国际化

UWS接口服务设计时要考虑国际化，能够国际化部署。系统规划统一的时钟方案，支持多语言、不同时区的业务处理。

#### 2.5.1 请求响应码配置文件化

请求响应码retCode和响应描述retInfo配置文件化，不能直接写在程序代码中。</br>
retInfo字段保留，但从程序中返回的都是英文，国际化从错误码文档体现。

#### 2.5.2 支持国际化部署

服务能够通过简单调整配置方式来快速部署到新的数据中心，通过新域名访问服务。

### 2.6 接入OSS运营日志系统

为了更好地对UWS服务的支持运营，要求UWS服务接口需统一接入OSS运营日志系统 **(对于平台已有未接入oss的服务,需随着后续版本迭代尽快增加接入，具体任务排期由产品经理确定。）**</br>

具体服务如何接入OSS请参照“服务链式跟踪插件使用说明书V1.1.doc”。

**附件:**[UTrace链式跟踪系统插件使用说明书][PluginInstruction]

## 3 接口安全

### 3.1 公共参数
应用与uws交互中，应用需要在每个请求Header中传入一些固定的参数；uws的每个响应中也会包含固定的响应码，具体如下：

**输入参数**

|参数名|	类型|位置|是否必填|说明|  
|:-----:|:-----:|:-----:|:-----:|--|
|appId|	String|	Header	|必填	|应用ID40位以内字符,Haier uHome 云平台全局唯一。开发者通过海极网申请获得。|
|appVersion	|String	|Header	|必填	|应用版本32 位字符,Haier uHome 云平台全局唯一。|  
|clientId	|String	|Header	|必填	|客户端ID27 位字符,客户端机编码与客户端 MAC 地址 拼合成唯一的客户端标识。 主要用途为唯一标识客户端 (例如,手机)。手机机编码为 IMEI 码。 手机 MAC 为 12 位地址。命名规范:客户端机编码(15 位)-客户 端 MAC 地址(12 位)格式: XXXXXXXXXXXXXXX-XXXXXXXXXXXX 举例: 356877020056553-08002700DC94。APP端可调用usdk获取，其他服务端自定义标识，不能为空。 |
|sequenceId	|String	|Header|必填	|报文流水(客户端唯一)客户端交易流水号。20 位, 前 14 位时间戳（格式：yyyyMMddHHmmss）,后 6 位流水 号。交易发生时,根据交易 笔数自增量。App应用访问uws接口时必须确保每次请求唯一，不能重复。|
|accessToken	|String	|Header|必填（登录后不为空，登录前可为空）|安全令牌 token，30 位字符。 用户登录 Haier U+ 云平台,由系统创建。用户退出 Haier U+ 云平台,由系统销毁。未登录时，访问不需要登录的平台接口，仍然需要传入本参数，参数值可为空或任意值（不超过30字符）|
|sign	|String|	Header|	必填|对请求进行签名运算产生的签名,签名算法见附录。|
|timestamp	|String	|Header	|必填|	应传入用户所在地时间戳，long型时间戳,精确到毫秒|
|language	|String	|Header|	必填	|该参数为多语言版提供支持。默认填写zh-cn即可。|
|timezone	|String|	Header|	必填	|代表客户端使用的时区。传入用户所在时区ID，国内服务请填写"Asia/Shanghai"即可|
|Content-Type|String|	Header|	必填	|互联网媒体信息，填写为"application/json;charset=UTF-8" |



**输出参数，UWS响应中包含**

|参数名|	类型|位置|是否返回|说明|  
|:-----:|:-----:|:-----:|:-----:|--|
|retCode|	String	|Body|	是	|返回码（其中00000代表请求成功,其它代表错误，错误码及描述见附录错误码表）|
|retInfo	|String	|Body	|是	|返回信息（用于调试的返回信息）|


### 3.2 UAG使用

U+IOT云平台为便于各服务采用统一的安全、鉴权标准，开发了UAG插件用来提供鉴权服务，包括应用访问鉴权，应用接口地址免token校验和请求公共参数sign签名认证。</br>
各服务只需关注自己的业务开发即可，通过引入UAG拦截器配置来服务鉴权。</br>
服务开发前向U+云平台负责账号系统的**余少晨（yushaochen.uh@haier.com）**获取要使用UAG插件的版本，使用参照UAG使用文档

**附件:**[UAG使用说明][UAG]


### 3.3 接口服务授权
按照UAG的serviceType服务类型有2c客户级，2e企业级两类，针对2e企业级服务的授权，需要对发起调用接口服务的应用服务器进行安全鉴权：

- 1.IP白名单方式</br>
需要支持IP地址或IP地址段的设置；IP地址的个数默认支持10个，可针对胳臂接口服务调整参数

- 2.域名白名单方式</br>
支持顶级域名配置白名单方式。


> 鉴于安全考虑，默认情况下只开放IP白名单方式，特殊情况下（如开发者无法提供具体的IP地址或IP地址段）可审核授权配置域名白名单。


## 4 接口文档
### 4.1 文档名称

uws服务开发完成输出的服务文档名称为**“uws服务名称+版本号”**，版本号为小写v+3位数字组成，如：v1.0.1。版本号由开发按文档编写自定义生成。

### 4.2 文档输出

开放的uws服务文档，开发编写好后发给**U+云平台智能服务团队（lijingtao@haier.com 李京涛）**统一往外部输出。

### 4.3 文档内容

- 1)文档格式参照《海尔开放平台接口说明书模板及规范》；
- 2)文档中需对接口服务出现的对象有统一明确的说明：“公共结构”章节；
- 3)文档中需对每个接口服务进行清晰的接口定义、请求样例、错误码的描述；
- 4)文档评审：开发uws服务的接口文档需组织进行评审，人员包括产品经理、项目经理、技术经理、uws服务开发人员等；

**附件：** [海尔开放平台接口说明书模板及规范][InterfaceSpecification]；

## 5 环境使用规范

### 5.1 各套环境及用途
环境名称|标识|主要用途
:-|:-|:-
北京联调环境|IP为192.168.190.*，需使用证书连接|用于平台联调，只提供给需要与平台进行联调的项目。与平台无关的项目不能使用该环境
北京验收环境|IP为192.168.160.*，需使用证书连接|用于质量部验收
开发者环境|IP为192.168.160.*，需使用证书连接|供开发者开发测试使用
北京生产环境|IP为10.159.*.* ，需拨通北京数据中心VPN访问|正式生产环境
北京验证环境|IP为10.159.*.* ，需拨通北京数据中心VPN访问|主要用于平台功能验证、压力测试

**统一申请途径：**通过运维支撑系统向U+云平台运维组申请开发所需的应用服务器、数据库。运维支撑系统：http://203.187.186.135:5000/login


## 6 代码管理规范

### 6.1 源码管理

**使用SVN进行源码管理：**

开发所需的具体SVN地址通过U+云平台的运维支撑系统向运维人员**（xiaoxuesong.uh@haier.com 肖雪松）**提出申请。

运维支撑系统：http://203.187.186.135:5000/login

### 6.2 依赖管理方式

**使用Maven管理依赖**

开发所需的Maven本地仓库地址通过U+云平台的运维支撑系统向运维人员**（xiaoxuesong.uh@haier.com 肖雪松）**提出申请。

### 6.3 Java开发规范

请遵照各项目组制定的开发规范

### 6.4 数据库开发规范

请遵照数据开发使用规范文档，联系人DBA**（wangxianchao.xian@haier.com 王贤朝）**  


**附件:** [数据库开发使用规范][DatabaseDevelopmentSpecification]  


## 7 资源申请

### 7.1 错误码申请

各服务在开发业务过程中出现新增的错误信息，需定义好错误信息描述，以及出现此错误时的解决方案。按照如下表格填写申请单，邮件发送给**U+云平台智能服务团队（lijingtao@haier.com 李京涛）**进行申请错误码。

申请示例：

字系统名|错误描述（retInfo）|解决方案|分配错误码（retCode）
:-:|:-:|:-:|:-
uds|错误\*\*\* | \*\*\* | 由分配人员填写

### 7.2 应用名

后续新增的uws服务所叫的服务名称、应用名由U+IOT平台的产品经理进行定义，并标识在对应的产品需求文档中，开发人员根据文档的定义名称进行编制相关开发文档和服务开发。


### 7.3 SVN

#### SVN账号

开发所需使用的SVN账号通过U+云平台的运维支撑系统向运维人员**（xiaoxuesong.uh@haier.com 肖雪松）**提出申请。

运维支撑系统：http://203.187.186.135:5000/login

#### SVN地址

开发所需的具体SVN地址通过U+云平台的运维支撑系统向运维人员**（xiaoxuesong.uh@haier.com 肖雪松）**提出申请。

运维支撑系统：http://203.187.186.135:5000/login

#### Maven本地仓库地址

开发所需的Maven本地仓库地址通过U+云平台的运维支撑系统向运维人员**（xiaoxuesong.uh@haier.com 肖雪松）**提出申请。


## 8 服务发布

### 8.1 发布流程

**服务发布流程如图所示：**

![发布流程图][process_type]



### 8.2 服务发布域名说明

**国内环境域名：**

> 开发者环境uws服务域名使用：`dev-uws.haigeek.com`

> 生产环境uws服务域名使用：`uws.haier.net`


**国外环境域名：**

> 北美环境uws服务域名使用：`uws-gea-us.haieriot.net`

> 欧洲环境uws服务域名使用：`uws-gea-euro.haieriot.net`



## 9 附录

### 9.1 签名算法

#### 参数介绍

**待签名字符串为：**url字符串 + Body字符串+appId+appKey +timestamp；

**url 字符串：**指请求的接口地址去除https://uws.haier.net 后剩余的路径部分；

**Body字符串：**指应用发送请求的Body部分去除所有空白字符后的JSON字符串，没有body时为空字符串（不是null）;

**appId：**Header头中的属性（见公共部分说明）；

**appKey：**在海极网给应用申请的appKey，不能明文发送；

**timestamp：**Header头中的属性（见公共部分说明）；

#### 算法

签名算法就是对待签名字符串计算32位小写SHA-256值。

算法JAVA示例

```java
String getSign(String appId, String appKey, String timestamp, String body,String url){：
    URL urlObj = new URL(url);
    url=urlObj.getPath();
	appKey = appKey.trim();
	appKey = appKey.replaceAll("\"", "");
	if (body != null) {
		body = body.trim();
	}
	if (!body.equals("")) {
		body = body.replaceAll("", "");
		body = body.replaceAll("\t", "");
		body = body.replaceAll("\r", "");
		body = body.replaceAll("\n", "");
	}
	log.info("body:"+body);
	StringBuffer sb = new StringBuffer();
	sb.append(url).append(body).append(appId).append(appKey).append(timestamp);

	MessageDigest md = null;
	byte[] bytes = null;
	try {
		md = MessageDigest.getInstance("SHA-256");
		bytes = md.digest(sb.toString().getBytes("utf-8"));
	} catch (Exception e) {
		e.printStackTrace();
	}
	
	return BinaryToHexString(bytes);
}

String BinaryToHexString(byte[] bytes) {
	StringBuilder hex = new StringBuilder();
	String hexStr = "0123456789abcdef";
	for (int i = 0; i < bytes.length; i++) {		
		hex.append(String.valueOf(hexStr.charAt((bytes[i] & 0xF0) >> 4)));		
		hex.append(String.valueOf(hexStr.charAt(bytes[i] & 0x0F)));
	}
	return hex.toString();
}
```

### 9.2 国际化代码表


|语言编码|	英文名称|	中文名称|
|-----|----|----|
|af	|Afrikaans - South Africa	|南非荷兰语|
|ar-ae	|Arabic(U.A.E.)	|阿拉伯语 - 阿拉伯联合酋长国	|
|ar-bh	|Arabic(Bahrain)	|阿拉伯语 - 巴林	|
|ar-dz	|Arabic(Algeria)	|阿拉伯语 - 阿尔及利亚	|
|ar-eg	|Arabic(Egypt)	|阿拉伯语 - 埃及	|
|ar-iq	|Arabic(Iraq)	|阿拉伯语 - 伊拉克	|
|ar-jo	|Arabic(Jordan)	|阿拉伯语 - 约旦	|
|ar-kw	|Arabic(Kuwait)	|阿拉伯语 - 科威特	|
|ar-lb	|Arabic(Lebanon)	|阿拉伯语 - 黎巴嫩	|
|ar-ly	|Arabic(Libya)	|阿拉伯语 - 利比亚	|
|ar-ma	|Arabic(Morocco)	|阿拉伯语 - 摩洛哥	|
|ar-om	|Arabic(Oman)	|阿拉伯语 - 阿曼	|
|ar-qa	|Arabic(Qatar)	|阿拉伯语 - 卡塔尔	|
|ar-sa	|Arabic(Saudi Arabia)	|阿拉伯语 - 沙特阿拉伯	|
|ar-sy	|Arabic(Syria)	|阿拉伯语 - 叙利亚	|
|ar-tn	|Arabic(Tunisia)	|阿拉伯语 - 突尼斯	|
|ar-ye	|Arabic(Yemen)	|阿拉伯语 - 也门	|
|be	|Belarusian	|白俄罗斯语	|
|bg	|Bulgarian	|保加利亚语	|
|ca	|Catalan	|加泰罗尼亚语	|
|cs|	Czech	|捷克语|	
|da	|Danish	|丹麦语	|
|de	|German(Standard)	|德语 - 标准|	
|de-at	|German(Austrian)	|德语 - 奥地利	|
|de-ch	|German(Swiss)	|德语 - 瑞士|	
|de-li	|German(Liechtenstein)	|德语 - 列支敦士登|	
|de-lu	|German(Luxembourg)	|德语 - 卢森堡	|
|el	|Greek	|希腊语	|
|en	|English	|英语|	
|en-au	|English(Australian)	|英语 - 澳大利亚	|
|en-bz	|English(Belize)	|英语 - 伯利兹	|
|en-ca	|English(Canadian)	|英语 - 加拿大	|
|en-gb	|English(British)	|英语 - 英国	|
|en-ie	|English(Ireland)	|英语 - 爱尔兰|
|en-jm	|English(Jamaica)	|英语 - 牙买加	|
|en-nz	|English(New Zealand)	|英语 - 新西兰	|
|en-tt	|English(Trinidad)	|英语 - 特立尼达岛	|
|en-us	|English(United States)	|英语 - 美国	|
|en-za	|English(South Africa)	|英语 - 南非	|
|es	|Spanish(Spain - Modern Sort)	|西班牙语 - 标准|	



[^-^]:附件注释
[ErrorCode]:attachment/ErrorCode  
[PluginInstruction]:attachment/PluginInstruction  
[UAG]:attachment/UAG  
[DatabaseDevelopmentSpecification]:attachment/DatabaseDevelopmentSpecification  
[InterfaceSpecification]:attachment/InterfaceSpecification  
[process_type]:attachment\_media\process_type.png