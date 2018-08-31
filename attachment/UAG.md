# UAG使用文档  

!>  **当前版本**：UAG使用文档 V1.3.0   
 **更新时间**：2018-05-28  

### 介绍  

?> U+IOT云平台为便于各服务采用统一的安全、鉴权标准，开发了UAG插件用来提供鉴权服务，包括应用访问鉴权，应用接口地址免token校验和请求公共参数sign签名认证。  

### 使用说明  

?> 各服务只需关注自己的业务开发即可，通过引入UAG拦截器配置来服务鉴权。  

通常情况下，经过下面几步完成对UAG的引用：  
Uag 5.0版本更新使用：  

- **第一步：添加POM引用  （配置样例）**    

```java  
<dependency>
  <groupId>com.hshbic.cloud.security</groupId>
  <artifactId>security-uag</artifactId>
  <version>5.0.0-RELEASE</version>
</dependency>
```


- **第二步：添加Web.xml配置 （配置样例）**  

```java  
<filter>
        <filter-name>uagfilter</filter-name>
        <filter-class>com.hshbic.security.uag.SecurityFilter</filter-class>
        <init-param>
         	<param-name>serverId</param-name>
         	<param-value>uws-server-0001</param-value>
        </init-param>
        <init-param>
         	<param-name>serviceType</param-name>
            <param-value>2C</param-value>
         </init-param>
<init-param>
	<param-name>noBodyUrl</param-name>
	<param-value>/css/v1/anonymous,/v1/uploadFile</param-value>
</init-param>
         <init-param>
                   <param-name>signType</param-name>
                   <param-value>sha256</param-value>
          </init-param>
          <init-param>
                     <param-name>retCodeType</param-name>
                     <param-value>uws</param-value>
           </init-param>
    </filter>
    <filter-mapping>
     		<filter-name>uagfilter</filter-name>
        	<url-pattern>/v1/*</url-pattern>
</filter-mapping>

```  

?> **参数说明：**  

| 参数名       | 参数说明        | 默认值  |  
| ：-------------：|:-------------:|    
|serviceType  | 企业版服务类型 | 2c为appId， 2e为systemId，默认为2c |    
|noBodyUrl|	特殊接口，签名排除body|	接口地址，如：资源上传接接口|  
|signType|	兼容各版本uag版本号|	md5，sha256，不配置不使用签名验证|  
|retCodeType|	兼容错误码|	uws，openapi, eur，不配置为uws|  

?> **返回值（在http response header中返回）：：**  

| Key       | 参数说明        |  &emsp;  |  
| ：-------------：|:-------------:|    
| uagUserId| 	平台账号ID|  &emsp; |  
| uagAppId	| 开发者应用ID|  &emsp; |  
| uagAppKey	| 开发者应用ID key|  &emsp; |  
| uagSystemId	| 开发者应用系统ID|  &emsp; |  
| uagTenantId| 	租户ID|  &emsp; |  

 
 
