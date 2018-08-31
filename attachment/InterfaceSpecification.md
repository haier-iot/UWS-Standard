# *******服务  

## 引言  
### 执行摘要  

为Haier U+云平台提供用户********************************************服务的指导说明。  
作为第三方应用程序开发的依据和输入

### 编写目的  

为第三方应用程序开发提供基础和指导。

### 术语和定义  

Haier U+云平台，为物联网云平台，提供设备和应用的接入服务、数据分析服务等。  
USDK，APP开发包，符合E++协议，具有发现设备、配置设备上网、与设备通信等功能。  


### 参考资料  

本设计所参考的相关文档、标准等资料如下：  

### 特殊记号格式说明  

本文档暂未使用其他特殊文档记号或格式。  
本文中的接口示例均使用的是示例数据并非真实数据。  

## 公共说明  

### 基本介绍  

                                                                                            

### 调用流程  
                                                                                                                 


### 接入地址  

本文档提供的所有接口仅支持https协议请求。  
在开发、联调时，应用开发者应该连接开发者环境，通过配置应用开发者访问外网的路由器（设置路由器的dns），可以连接到不同的开发环境。  
部署生产环境：  

| 接口分类       | 接入地址        | 开发者环境DNS配置  |  
| ------------- |:----------:|:-----:|     
| **服务 | 	https://uws.haier.net | 210.51.17.146 |  

### 接口公共部分  

应用与uws交互中，应用需要在每个请求Header中传入一些固定的参数；uws的每个响应中也会包含固定的响应码，具体如下：  

**输入参数，请求中需要传入**  

|参数名|	类型|位置|是否必填|说明|  
|:-----:|:-----:|:-----:|:-----:|--|
|appId|	String|	Header	|必填	|应用ID,40位以内字符,Haier uHome 云平台全局唯一。开发者通过海极网申请获得。<font color=FF0000>本服务appId要传平台上申请的systemId。</font>|
|appVersion	|String	|Header	|必填	|应用版本32 位字符,Haier uHome 云平台全局唯一。|
|clientId	|String	|Header	|必填	|客户端ID27 位字符,客户端机编码与客户端 MAC 地址 拼合成唯一的客户端标识。 主要用途为唯一标识客户端 (例如,手机)。手机机编码为 IMEI 码。 手机 MAC 为 12 位地址。命名规范:客户端机编码(15 位)-客户 端 MAC 地址(12 位)格式: XXXXXXXXXXXXXXX-XXXXXXXXXXXX 举例: 356877020056553-08002700DC94 |
|sequenceId	|String	|Header|必填	|报文流水(客户端唯一)客户端交易流水号。20 位, 前 14 位时间戳（格式：yyyyMMddHHmmss）,后 6 位流水 号。交易发生时,根据交易 笔数自增量。App应用访问uws接口时必须确保每次请求唯一，不能重复。|
|accessToken	|String	|Header|必填|（登录后不为空，登录前可为空）	请求令牌（用户登陆后）安全令牌 token。30 位字符。<font color=FF0000>本服务3.3-3.8仍然需要传入本参数，参数值可为空或任意值（不超过30字符）,3.9-3.12要传拥有设备控制权限的用户token。</font>|
|sign	|String|	Header|	必填|（登录后不为空，登录前可为空）	详见签名认证章节|
|timestamp	|String	|Header	|必填|	long型时间戳,精确到毫秒，该参数为多国家地区提供支持。应传入用户所在地时间戳。|
|language	|String	|Header|	必填	|该参数为多语言版提供支持。默认填写zh-cn即可。|
|timezone	|String|	Header|	必填	|时区， -11 至 13。传入用户所在时区，默认填写8即可。|
|Content-Type|String|	Header|	必填	|该参数不同的服务会有所不同，一般为"application/json;charset=UTF-8" 具体参照媒体类型|  

**输出参数，uws响应中会包含**

|参数名|	类型|位置|是否返回|说明|  
|:-----:|:-----:|:-----:|:-----:|--|
|retCode|	String	|Body|	是	|返回码（其中00000代表请求成功,其它代表错误，错误码及描述见附录错误码表）|
|retInfo	|String	|Body	|是	|返回信息（用于调试的返回信息，不支持国际化，也不能直接显示在UI上）|

### 签名认证  

#### 说明  

调用方需要对发送到uws的请求进行签名，执行签名计算的签名值需要赋值到Header头中的sign属性（见公共部分说明），以便服务端进行签名验证。  

#### 参数介绍  

**待签名字符串为：**url字符串 + Body字符串+appId+appKey +timestamp；  
**url 字符串：**指请求的接口地址去除https://域名+端口 后剩余的路径部分；  
**Body字符串：**指应用发送请求的Body部分去除所有空白字符后的JSON字符串，没有body时为空字符串（不是null）。  
**appId：**Header头中的属性（见公共部分说明）；  
**appKey：**在海极网给应用申请的appKey，不能明文发送；  
**timestamp：**Header头中的属性（见公共部分说明）；  


#### 算法  

签名算法就是对待签名字符串计算32位小写SHA-256值，算法示例见附录。  


### 国际化处理  

对接口响应返回的retCode和retInfo不做国际化处理，由接口调用方处理。  
对于接口涉及业务数据的国际化通过在header中传递language参数来定义，具体的国际化语言代码见附录。  


### 数据类型限定  

#### 字段类型说明  


|限定类型|	说明|	格式|	json示例|
|---|---|---|---|
|DateTime	|日期时间类型的字符串|	`yyyy-MM-dd hh:mm:ss`| `	{“lgTime”:“2013-10-08 08:00:00”}`|
|Date	|日期类型的字符串	|`yyyy-MM-dd` 	|`{“lgDate”:“2013-10-08”}`|
|String	|字符串		||`{“address”:“street 123”}`|
|int	|整形数字	||	`{“age”:1234}`|
|long	|长整形数字	||	`{“oid”:1234567890123}`|
|double	|浮点数数字	||	`{“price”:12.35}`|
|boolean	|布尔型（true或false）	||	`{“idOld”:true}`|

#### null值说明  

为避免解析错误， uws各接口的返回参数，不返回null值。
必填参数，无论输入还是输出，必须有值，不能为null
非必填参数，则说明如下：
数值类型数据（int、long、double）只会返回数字，包括正数、零及负数。
布尔类型数据（boolean）只返回true和false
以上基本类型本身不包含null值。

DateTime、Date、String及结构体类型的数据，如果为null时，所对应的属性将不返回。

**DateTime类型**  

birthday为DateTime类型，不为null时：

    {"name":"Tom","age":23,"birthday":"2013-10-08 08:00:00","address":{"city":"beijing","street":"haidian"} }

birthday为null时，则birthday属性不返回

    {"name":"Tom","age":23, "address":{"city":"beijing","street":"haidian"} }  

**String类型**  

name是String类型，不为null时：

    {"name":"Tom","age":23,"birthday":"2013-10-08 08:00:00","address":{"city":"beijing","street":"haidian"} }

name为null时，则name属性不返回

    {"age":23,"birthday":"2013-10-08 08:00:00","address":{"city":"beijing","street":"haidian"} }


**结构体类型**

address为结构体类型，不为null时：
  
    {"name":"Tom","age":23,"birthday":"2013-10-08 08:00:00","address":{"city":"beijing","street":"haidian"} }

address为null时，address属性不返回

	{"name":"Tom","age":23,"birthday":"2013-10-08 08:00:00" }


## ****服务 

### 公共说明  

*************************************************

### 公共结构  

#### ******  

<table>
<tr>
    <td  >名称</td>
    <td colspan="2">******</td>
  
	<td>****</td>
</tr>
<tr>
    <td >字段名</td>
    <td>类型</td>
	<td>说明</td>
	<td>备注</td>
</tr>
<tr>
	<td >******</td>
    <td>******</td>
	<td>******</td>
	<td>******</td>
</tr>
<tr>
	<td >******</td>
    <td>******</td>
	<td>******</td>
	<td>******</td>
</tr>
<tr>
	<td >******</td>
    <td>******</td>
	<td>******</td>
	<td>******</td>
</tr>
</table>


### ****接口***  

#### 接口定义  

![接口定义图片][interface_temp]  


#### 请求样例  

*********************************  

#### 错误码  

*********************************  

## 附录  

### 公共错误码  

|retCode错误码|	retInfo描述|
|---|---|---|---|
|A00001|	服务不可用                                    |
|A00002|	网络异常                                      |
|A00003|	访问或操作超时                                |
|A00004|	系统内部错误                                  |
|A00005|	数据库访问异常                                |
|A00006|	未知异常                                      |
|A00007|	邮件服务异常                                  |
|A00008|	邮件发送失败                                  |
|A00009|	邮件发送次数超限                              |
|B00001|	缺少必填参数                                  |
|B00002|	参数类型错误                                  |
|B00003|	参数数值超出值域或不是枚举值                  |
|B00004|	参数不符合规则要求                            |
|B00006|	参数长度错误                                  |
|B00007|	参数与接口定义不匹配                          |
|C00001|	appId与appKey验证失败                         |
|C00002|	appServer无访问授权                           |
|C00003|	访问权限不足                                  |
|C00004|	操作权限不足                                  |
|C00005|	重复请求                                      |
|C00006|	未知设备类型                                  |
|C00007|	appId配置信息为空                             |
|C00008|	appKey为空                                    |
|D00001|	sign签名错误                                  |
|D00002|	账号或密码错误                                |
|D00003|	Token不存在，未通过token验证                  |
|D00004|	Token已过期，未通过token验证。                |
|D00005|	Token不是由此应用创建，未通过token验证。      |
|D00006|	会话失效                                      |
|D00007|	不是系统内部用户                              |
|D00008|	用户不合法                                    |  

### 国际化语言代码表  

|语言编码|	英文名称|	中文名称|	是否支持|
|-----|----|----|----|
|af	|Afrikaans - South Africa	|南非荷兰语|	否|
|ar-ae	|Arabic(U.A.E.)	|阿拉伯语 - 阿拉伯联合酋长国	|否|
|ar-bh	|Arabic(Bahrain)	|阿拉伯语 - 巴林	|否|
|ar-dz	|Arabic(Algeria)	|阿拉伯语 - 阿尔及利亚	|否|
|ar-eg	|Arabic(Egypt)	|阿拉伯语 - 埃及	|否|
|ar-iq	|Arabic(Iraq)	|阿拉伯语 - 伊拉克	|否|
|ar-jo	|Arabic(Jordan)	|阿拉伯语 - 约旦	|否|
|ar-kw	|Arabic(Kuwait)	|阿拉伯语 - 科威特	|否|
|ar-lb	|Arabic(Lebanon)	|阿拉伯语 - 黎巴嫩	|否|
|ar-ly	|Arabic(Libya)	|阿拉伯语 - 利比亚	|否|
|ar-ma	|Arabic(Morocco)	|阿拉伯语 - 摩洛哥	|否|
|ar-om	|Arabic(Oman)	|阿拉伯语 - 阿曼	|否|
|ar-qa	|Arabic(Qatar)	|阿拉伯语 - 卡塔尔	|否|
|ar-sa	|Arabic(Saudi Arabia)	|阿拉伯语 - 沙特阿拉伯	|否|
|ar-sy	|Arabic(Syria)	|阿拉伯语 - 叙利亚	|否|
|ar-tn	|Arabic(Tunisia)	|阿拉伯语 - 突尼斯	|否|
|ar-ye	|Arabic(Yemen)	|阿拉伯语 - 也门	|否|
|be	|Belarusian	|白俄罗斯语	|否|
|bg	|Bulgarian	|保加利亚语	|否|
|ca	|Catalan	|加泰罗尼亚语	|否|
|cs|	Czech	|捷克语|	否|
|da	|Danish	|丹麦语	|否|
|de	|German(Standard)	|德语 - 标准|	否|
|de-at	|German(Austrian)	|德语 - 奥地利	|否|
|de-ch	|German(Swiss)	|德语 - 瑞士|	否|
|de-li	|German(Liechtenstein)	|德语 - 列支敦士登|	否|
|de-lu	|German(Luxembourg)	|德语 - 卢森堡	|否|
|el	|Greek	|希腊语	|否|
|en	|English	|英语|	是|
|en-au	|English(Australian)	|英语 - 澳大利亚	|否|
|en-bz	|English(Belize)	|英语 - 伯利兹	|否|
|en-ca	|English(Canadian)	|英语 - 加拿大	|否|
|en-gb	|English(British)	|英语 - 英国	|否|
|en-ie	|English(Ireland)	|英语 - 爱尔兰|	否|
|en-jm	|English(Jamaica)	|英语 - 牙买加	|否|
|en-nz	|English(New Zealand)	|英语 - 新西兰	|否|
|en-tt	|English(Trinidad)	|英语 - 特立尼达岛	|否|
|en-us	|English(United States)	|英语 - 美国	|否|
|en-za	|English(South Africa)	|英语 - 南非	|否|
|es	|Spanish(Spain - Modern Sort)	|西班牙语 - 标准|	否|  

### 签名算法示例  

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


[interface_temp]:_media/_attachment/interface_temp.png