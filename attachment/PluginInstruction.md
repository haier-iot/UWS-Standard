# UTrace链式跟踪系统插件使用说明书  

!> **当前版本**：服务链式跟踪插件使用说明书 V1.1.0   
 **更新时间**：2018-05-04  


## 引言  
### 目的  

?> 本说明书详细介绍了UTrace Client插件的详细配置和使用说明，为插件部署提供方便。    


### 说明  

?> UTrace Client 是用来装备 Java 程序的类库，提供了面向 Standard Servlet、Http Client、Dubbo、Db 等接口的装备能力，可以通过编写简单的配置和代码，让基于这些框架构建的应用可以向 Utrace 系统报告数据。  

## UTrace Client组成  

![UTrace Client组成图片][UTraceClient]   


?> **UTrace Client** 主要由utrace配置信息、Core核心包、Component组件库。
**utrace配置信息** 是运行环境的配置文件，对对象定义、节点信息、httpclient配置、名单、请求和响应规则解析配置。  
**Core** 核心包定义了在各种通讯协议下，CS、CR、SR 、SS埋点的核心逻辑，该包中的方法，主要给Component中的各种Filter、Interceptor调用对外提供服务，定义默认的埋点日志输出方式(Log)。  
**Component** 提供了为各种框架构建的应用程序进行埋点的中间件，应用程序直接引用这些中间件，并编写简单的配置和代码，即可完成埋点和数据上报。  
  

## UTrace client插件配置说明  
### maven配置信息  

?> Maven项目首先配置pom.xml，添加utrace插件依赖：  


	<dependencies>
		<dependency>
			<groupId>com.hshbic.utrace</groupId>
			<artifactId>utrace-web-servlet-filter</artifactId>
			<version>1.1.5</version>
		</dependency>
		<dependency>
			<groupId>com.hshbic.cloud.m2m</groupId>
			<artifactId>m2m-common-rest-client</artifactId>
			<version>1.1.5-SNAPSHOT</version>
		</dependency>
		<dependency>
			<groupId>com.hshbic.utrace</groupId>
			<artifactId>utrace-httpclient</artifactId>
			<version>1.1.5</version>
		</dependency>
		<dependency>
			<groupId>com.hshbic.utrace</groupId>
			<artifactId>utrace-dubbo</artifactId>
			<version>1.1.5</version>
		</dependency>
	<dependency>
			<groupId>com.hshbic.utrace</groupId>
			<artifactId>utrace-mybatis-interceptor</artifactId>
			<version>1.1.5</version>
		</dependency>
	</dependencies>

 

### 实例配置信息

?> 配置utrace.xml，通过spring bean方式定义和配置utrace核心包和插件，并配置依赖关系：  

 
	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	   	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
	    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
	<!-- 自动扫描 ,将带有注解的类 纳入spring容器管理 -->
	<context:component-scan base-package="com.hshbic.utrace" />

	<!-- utrace核心类定义，注：id必须为uTrace -->
	<bean id="uTrace" class="com.hshbic.utrace.UTrace" scope="singleton"
		init-method="init">
		<!-- web应用使用下面配置 -->
		<property name="location" value="classpath:utrace_extractor.xml" />
		<!-- paas平台下dubbo使用下面配置 -->
		<property name="config" value="/utrace_extractor.xml" />
	</bean>

	<!-- dubbo -->
	<bean class="com.hshbic.utrace.dubbo.UTraceDubboManagementBean"
		scope="singleton">
		<property name="uTrace" ref="uTrace"></property>
	</bean>

	<!-- httpclient -->
	<bean id="uTraceHttpClient" 
			class="com.hshbic.utrace.http.UTraceHttpClientImpl"
		scope="singleton" init-method="init">
		<property name="uTrace" ref="uTrace"></property>
	</bean>
	</beans>  

<font color=FF0000><b>注：以上定义的核心类UTrace的id、组件中的属性和依赖的名称是不能变的。</b></font>

### Web应用容器配置 

?> 配置web应用的web.xml：    

	<!-- 声明WEB应用上下文初始化参数，参数定义了要装入的 utrace 配置文件 -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:utrace.xml</param-value>
	</context-param>

	<!-- 监听器自动装配Spring相关的配置信息 -->
	<listener>
		<listener-class>
			org.springframework.web.context.ContextLoaderListener
		</listener-class>
	</listener>



### utrace节点详细配置  
?> 应用插件时在resources添加utrace_extractor.xml文件，配置信息包含节点信息、请求和响应规则解析配置；    

#### 基础配置信息  
![基础配置信息图片][ConfigurationInformation]  
	<!-- 服务节点IP -->
	<pair key="serverIp" value="10.159.32.166"/>

#### Httpclient插件配置信息

![插件配置信息图片][Httpclient]  


#### 名单配置信息  

![黑名单配置信息图片][blacklist]  

?> Pair value中配置具体采用名单策略(`all、whitelist、blacklist`)  
`all`:全部处理，不过滤 ；  
`whitelist`：白名称，只处理名单列表中配置的地址；  
`blacklist`：黑名单，不处理名单列表中配置的地址；  
配置名单中对应的值包含以下两种：  
`http`： `url`中包含动态参数的用`{0}`替换；`value.method`为请求方式  				`value.method`配置不为空时，请求地址和请求方式都要验证；  
		`value.method`没有配置或配置为空时，只验证请求地址；  
`dubbo`：包名.类名.方法名；  

#### 请求数据解析规则配置  

![数据解析图片][DataParsing]   

?> 请求地址配置规则示例：  

1)、	地址相似的，动态参数有规律的：  
`/commonapp/users/{userId}`  
`/commonapp/users/{userId}/mobile`  
`/commonapp/users/{userId}/profile`  
`/commonapp/users/{userId}/originalMessages`  
`/commonapp/users/{userId}/messages/unreadCount`  
`/commonapp/users/{userId}/messages/confirm`  
`/commonapp/users/{userId}/devices/{deviceId}/name`  

根据以上url，其中有相同规则可以按下面方式配置  

![url配置图片][urlConf]  


<font color=FF0000><b>注：url长的放在前面</b></font>

2)、	地址相似的，参数无规则的：    
`/commonapp/resources/types/12345/info`   
`/commonapp/resources/12345`    
以上两个地址相似，配置时要配置两个解析规则如下：  

![url配置图片2][urlConf2]   

如果按以上解析规则配置，如果请求地址是  
`/commonapp/resources/types/12345/info`根据配置规则的先后顺序，首先匹配到规则是/`commonapp/resources/{0}`，然后进行规则解析最后得到的地址是`/commonapp/resources/{0}/12345/info`，匹配结果错误,正确的配置是：  
![url配置图片3][urlConf3]     

说明：以上配置中出现的字段属性具体含义可见[链式跟踪日志schema](#index1) ；    
1)、	request节点表示为请求节点；  
2)、	外层`element`节点 `key`的值为链式跟踪日志对应的信息域；  
3)、	内层`element`节点 `key`的值为链式跟踪日志对应的日志信息，`append`属性表示为日志信息追加，`replace`表示匹配规则计算后的值是否替换日志属性值，`notation`表示动态参数要替换的值，`split`表示多匹配值时是否进行拆分；  
4)、	`filte`r节点 表示过滤掉能匹配到自定义函数但不包含动态参数的；  
5)、	`value`节点`format`为匹配规则，method为匹配的方式，`value`节点中的内容是要根据匹配规则和方式进行计算后要赋值的日志属性；  
匹配方式：`regex (正则)，json`  
实例：  

*   `reqHttp.reqUrl=/v1/users/10001395736615537/devices/C8934640948A`   
    `value.format=/v1/users/{0}/devices/{0}`  
    `value.method=regex`  
	`value.content= ,reqMAC`  
 执行过程：  
`reqUrl`通过正则计算后取得  
`10001395736615537,C8934640948A`  
value.content内容按动态参数出现的顺序进行配置，多动态参数时用逗号分隔；   
(动态参数的值在服务链式跟踪日志中不赋值时填空)  
最后更新value.content中配置的属性对应的值  
 结果：  
	`reqHttp.reqUrl=/v1/users/{0}/devices/{0}`  
	`reqMAC.reqMAC=C8934640948A`

*   `reqHttp.reqUrl=/v1/protected/C8934640948A`  
	`value.format=/v1/protected/{0}`   
	`value.method=regex`  
	`value.content= reqMAC`  
   执行过程：  
`reqUrl`通过正则计算后取得  
`C8934640948A`
最后更新`value.content`中配置的属性对应的值  
   结果：  
	`reqHttp.reqUrl=/v1/users/{0}/devices/{0}`  
	`reqHttp.reqMAC=C8934640948A` 
*   `reqHttp.reqUrl=/v1/protected/C8934640948A/isOnline`  
	`value.format=/v1/protected/{0}/*`  
	`value.method=regex`  
	`value.content=reqMAC`  
   执行过程：  
`reqUrl`通过正则计算后取得    
`C8934640948AZZZZ`  
   最后更新`value.content`中配置的属性对应的值  
   结果：  
	`reqHttp.reqUrl=/v1/protected/{0}/isOnline`
	`reqHttp.reqMAC=C8934640948A`  
  	注：同样规则的的url地址可以使用星号  
	`/v1/protected/C8934640948A/isOnline`  
	`/v1/protected/C8934640948A/aliasName`  
	`value.format=/v1/protected/{0}/*`  

*   `reqHttp.reqBody={'deviceId':'C8934640948A','name':'空调'}`  
	`value.format=deviceId`  
	`value.method=json`  
	`value.content= reqMAC`  
   执行过程：  
`reqUrl`通过函数`json`进行解析取得  
`C8934640948A`  
最后更新`value.content`中配置的属性对应的值  
   结果：  
`reqHttp.reqMAC=C8934640948A`  

*   `reqHttp.reqBody={'devices':[`  
`{'id':'A8934640948B','name':'空调A'},`  
`{'id':'C8934640948B','name':'空调B'}`  
`]}`  
`value.format= devices[id]`  
`value.method=json`  		
`value.content= reqMAC`  
   执行过程：  
`reqUrl`通过函数`json`进行解析结果取得  
`A8934640948B C8934640948B`  
结果出现多值时使用逗号连接,最后更新value.content中配置的属性对应的值。  
  结果：  
`reqHttp.reqMAC= A8934640948B,C8934640948B`  
注：  
`value.format=devices[id]` 表示`json`中出现数组集合;  
`devices`表示集合的key值;  
`id`表示集合内属性;  

#### 响应数据解析规则配置  

![url配置图片4][urlConf4]  

1)、	response节点表示为响应节点；  
2)、	element节点 key的值为服务类型(serviceType)；  
3)、	bean节点 key的值响应的数据类型；  
4)、	bean节点下的value元素；  

`value`节点 `format`为匹配规则，method为匹配的方式，value节点中的内容是要根据匹配规则和方式进行计算后要赋值的日志属性  
说明：没有配置响应规则的按默认规格解析  
a)	出现异常的`retCode=-1`  
b)	无异常的 `retCode=00000`  
注：  
`value.format`根据实际响应类型中的属性进行配置；  
规则说明：  

*   `bean.key=java.lang.Integer` (返回值为数值型)  
`value.format=`匹配的值  
`value.method=计算公式(=、!=、>、>=、<、<=)`  
实例：  
![url配置图片5][urlConf5] 
响应值`respInt = 2`   
`if(respInt > format){`  
	`retCode=00000;`  
`}else{`  
	`retCode-1;`  
`}`  
 计算结果返回`true`时返回码为`00000`,为`false`时为`-1`；  
说明：`=(等于) 、!=(不等于)、>(大于)、>=(大于等于)、<(小于) 、<= (小于等于)`  


*   `bean.key=java.lang.String (返回值为字符型)`  
`value.format=匹配的值`  
`value.method=计算公式(equals、startsWith、endsWith、contains、notEmpty、empty)`  
实例：  
![url配置图片6][urlConf6]  
响应值`respStr = “ACCCCA”`  
`if(respStr.equals (format)){`  
	`retCode=00000;`  
`}else{`  
	`retCode-1;`  
`}`  
说明：`equals`(等于) 、`startsWith`(以`**`开头)、`endsWith`(以`**`结尾)、`contains`(包含)、`notEmpty`(不为空,`value.format`不配置) 、`empty` (为空, `value.format`不配置)  

*   `bean.key=java.lang.Boolean (返回值为布尔型)`  
`value.format=匹配的值`  
`value.method=计算公式(==、!=)`  
实例：  
![url配置图片7][urlConf7]  
响应值`respStr = true`  
`if(respStr == format){`  
	`retCode=00000;`  
`}else{`  
	`retCode-1;`  
`}`
说明：`==` (等于) 、`!=`(不等于)  


*   `bean.key=自定义对象`  
![url配置图片8][urlConf8]  
`bean.method=json`，转换的格式(不可更改，否则解析失败) 按`json`对自定义对象进行转换   
`value.format`：转换后的`json`里面属性`key`   
`value.svalue：`   
1)、取得`value.format`配置的`json`属性`key`对应的值与`value.svalue`的值进行判断，相同则`value.content=00000`，不同`value.content=`解析后的`key`对应的属性值；  
2)、`value.svalue`不配置时则将`json`属性`key`对应的值赋值给`value.content`    
`value.content`：日志中的属性；  
实例：  
1)、  
![url配置图片9][urlConf9]  
`ServiceException:{'resCode':'G0000','resInfo':'成功'}`  
`retCode=00000`  
`retInfo=成功`  
2)、  
![url配置图片10][urlConf10]  
`ServiceException:{'resCode':'G0000','resInfo':'成功'}`  
`retCode=G0000`  
`retInfo=成功`  

#### 业务规则配置  
![url配置图片11][urlConf11]  
按请求地址配置对应业务地址;  

#### 日志配置  

参见[日志收集](#index2)章节



##  Component组件集成应用  

?> 提供了对Standard Servlet、Http Client、Dubbo、Db框架支持的中间件。

### 组件应用  

#### utrace-web-servlet-filter  

1)、	对maven项目中需要在pom.xml中配置UTrace Client插件所需要的依赖包；  

	<dependencies>
		<dependency>
			<groupId>com.hshbic.utrace</groupId>
			<artifactId>utrace-web-servlet-filter</artifactId>
			<version>1.1.5</version>
		</dependency>
	</dependencies>


2)、	对于应用spring开源框架的项目中需要对UTrace Client插件中应用的类进行初始化，并加载需要的配置信息，应用插件时在resources添加utrace.xml文件，配置信息如下：  

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
		xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

		<!-- 自动扫描 ,将带有注解的类 纳入spring容器管理 -->
		<context:component-scan base-package="com.hshbic.utrace" />

		<!-- utrace核心类定义，注：id必须为uTrace -->
		<bean id="uTrace" class="com.hshbic.utrace.UTrace" scope="singleton"
			init-method="init">
			<property name="location" value="classpath:utrace_extractor.xml" />
		</bean>

	</beans>
 
3)、	应用插件时在resources添加utrace_extractor.xml文件，配置信息包含节点信息、请求和响应规则解析配置，配置信息如下:  

```    
<?xml version="1.0" encoding="UTF-8"?>  
<configuration>

	<!-- 系统名称 -->
	<pair key="sysName" value="UWS" />
	<!-- 服务名称 -->
	<pair key="serviceName" value="uds" />
	<!-- 应用所在节点标识,全局唯一(一个应用可部署于多个节点) -->
	<pair key="nodeIdentifier" value="9789f6b082214463ab70cb4d22158b2f" />
<!-- 服务节点IP -->
<pair key="serverIp" value="10.159.32.166"/>
	<!-- 是否开户链路跟踪,值为on/off(空值为on) -->
	<pair key="switch" value="on" />
	<!-- 链路节点状态，根据各应用实际情况选择配置 -->
	<!--S0:起始节点状态,接收到服务请求,请求头包含uTraceId使用此值并且不验证uStatus然后根据名单过滤, 请求头不包含uTraceId的自动生成然后根据名单过滤 
		S1:起始节点状态,接收到服务请求,请求头包含uTraceId使用此值并且不验证uStatus然后根据名单过滤, 请求头不包含uTraceId的不生成日志 
		S2:起始节点状态,接收到服务请求,请求头包含uTraceId使用此值并且验证uStatus, 请求头不包含uTraceId的自动生成然后根据名单过滤 
		S3:起始节点状态,接收到服务请求,请求头包含uTraceId使用此值并且验证uStatus, 请求头不包含uTraceId的自动生成，名单配置无效 
		C0:起始节点状态,客户端发送请求,需要根据名单过滤
		D0:默认节点状态,可不配置,接收到服务请求,请求头包含uTraceId使用此值并且验证uStatus,请求头不包含uTraceId不生成日志,名单配置无效 -->
	<pair key="nodeStatus" value="XX" />

	<!-- 名单管理 -->
	<pair key="uTraceUrlFilter" value="blacklist" />
	<nlist>
		<value>/v1/protected/devices/{0}</value>
		<value>/uds/v1/protected/bindDevice</value>
	</nlist>

	<request>
		<element key="reqHttp">
			<element key="reqUrl" append="/" replace="true" notation="{0}" split="true">
				<value format="/v1/users/{0}/devices/{0}" method="regex">,reqMAC</value>
				<value format="/v1/protected/{0}/*" method="regex">reqMAC</value>
				<filter>
					<value>/uds/v1/protected/bindDevice</value>
				</filter>
			</element>
			<element key="reqBody">
				<value format="deviceId" method="json">reqMAC</value>
				<value format="devices[id]" method="json">reqMAC</value>
			</element>
		</element>
	</request>

	<!-- 响应实例规则 -->
	<response>
		<element key="http">
			<bean key="java.lang.String" method="json">
				<value format="retCode">retCode</value>
				<value format="retInfo">retInfo</value>
			</bean>
		</element>
</response>

	<!-- 业务信息 -->
	<business>
		<value format="bind" method="POST">/commonapp/users/{0}/devices</value>
		<value format="unBind" method="delete">/commonapp/users/{0}/devices/{0}</value>
</business>

</configuration>

```  

4)、	在web应用容器web.xml中配置UTraceWebServletFilter过滤器，如下：    

	<!-- 声明WEB应用上下文初始化参数，参数定义了要装入的 utrace 配置文件 -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:utrace.xml</param-value>
	</context-param>
	<!-- 监听器装配ApplicationContext的配置信息 -->
	<listener>
	<listener-class>
		org.springframework.web.context.ContextLoaderListener
	</listener-class>
	</listener>
	<filter>
		<filter-name>uTraceWebServletFilter</filter-name>
		<filter-class>
			com.hshbic.utrace.servlet.UTraceWebServletFilter
		</filter-class>
	</filter>
	<filter-mapping>
		<filter-name>uTraceWebServletFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>   

#### utrace-dubbo   

1)、	对maven项目中需要在pom.xml中配置UTrace Client插件所需要的依赖包；  


	<dependencies>
		<dependency>
			<groupId>com.hshbic.utrace</groupId>
			<artifactId>utrace-dubbo</artifactId>
			<version>1.1.5</version>
		</dependency>
	</dependencies>



2)、	配置信息包含以下两种：  
a)	web应用使用spring开源框架的项目中需要对UTrace Client插件中应用的类进行初始化，并加载需要的配置信息，应用插件时在resources添加utrace.xml文件；  

b)	java程序启动时将utrace.xml加载到classpath中；并在启动方法通过下面方法内加载内容：`new ClassPathXmlApplicationContext("utrace.xml")`;  
配置内容如下：  

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
		xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
		<!-- 自动扫描 ,将带有注解的类 纳入spring容器管理 -->
		<context:component-scan base-package="com.hshbic.utrace" />
		<!-- utrace核心类定义，注：id必须为uTrace -->
		<!-- utrace核心类定义，注：id必须为uTrace -->
		<bean id="uTrace" class="com.hshbic.utrace.UTrace" scope="singleton"
			init-method="init">
			<!-- web应用使用下面配置/java程序启动 -->
			<property name="location" value="classpath:utrace_extractor.xml" />
			<!-- paas平台下dubbo使用下面配置 -->
			<property name="config" value="/utrace_extractor.xml" />
		</bean>
		<!-- dubbo 组件定义-->
		<bean class="com.hshbic.utrace.dubbo.UTraceDubboManagementBean"
			scope="singleton">
			<property name="uTrace" ref="uTrace"/>
		</bean>
	</beans>



3)、	应用插件时在classpath目录下添加utrace_extractor.xml文件，配置信息包含节点信息、请求和响应规则解析配置，配置信息如下：  

 
	<?xml version="1.0" encoding="UTF-8"?>  
	<configuration>
		<!-- 系统名称 -->
		<pair key="sysName" value="UWS" />
		<!-- 服务名称 -->
		<pair key="serviceName" value="uds" />
		<!-- 应用所在节点标识,全局唯一(一个应用可部署于多个节点) -->
		<pair key="nodeIdentifier" value="9789f6b082214463ab70cb4d22158b2f" />
		<!-- 服务节点IP -->
		<pair key="serverIp" value="10.159.32.166"/>
		<!-- 是否开户链路跟踪,值为on/off(空值为on) -->
		<pair key="switch" value="on" />
		<!-- 链路节点状态，根据各应用实际情况选择配置 -->
		<!--S0:起始节点状态,接收到服务请求,请求头包含uTraceId使用此值并且不验证uStatus然后根据名单过滤, 请求头不包含uTraceId的自动生成然后根据名单过滤 
		S1:起始节点状态,接收到服务请求,请求头包含uTraceId使用此值并且不验证uStatus然后根据名单过滤, 请求头不包含uTraceId的不生成日志 
		S2:起始节点状态,接收到服务请求,请求头包含uTraceId使用此值并且验证uStatus, 请求头不包含uTraceId的自动生成然后根据名单过滤 
		S3:起始节点状态,接收到服务请求,请求头包含uTraceId使用此值并且验证uStatus, 请求头不包含uTraceId的自动生成，名单配置无效 
		C0:起始节点状态,客户端发送请求,需要根据名单过滤
		D0:默认节点状态,可不配置,接收到服务请求,请求头包含uTraceId使用此值并且验证uStatus,请求头不包含uTraceId不生成日志,名单配置无效 -->
		<pair key="nodeStatus" value="XX" />	
		<!-- 名单管理 -->
		<pair key="uTraceUrlFilter" value="all" />
		<nlist>
		</nlist>	
		<!-- 响应实例规则 -->
		<response>
			<element key="dubbo">
				<bean key="com.hshbic.cloud.exception.ServiceException" method="json">
					<value format="resCode">retCode</value>
					<value format="resInfo">retInfo</value>
				</bean>
				<bean key="com.hshbic.cloud.m2m.domain.common.util.Result"
				method="json">
					<value format="code">retCode</value>
					<value format="info">retInfo</value>
				</bean>
				<bean key="java.lang.Integer">
					<value format="1" method=">">retCode</value>
				</bean>
				<bean key="java.lang.String">
					<value format="utrace" method="contains">retCode</value>
				</bean>
				<bean key="java.lang.Boolean">
					<value format="true" method="==">retCode</value>
				</bean>
			</element>
		</response>
		<!-- 业务信息 -->
		<business>	
		</business>
	</configuration>



4)、	客户请求传递UTrace  

`dubbo`消费者提供了三个方法，如下：   
a)	`UTraceDubboContextApi.addUTraceContextRequest()`;自动生成`uTraceId(UUID)`，`uSpanId(0.1)`，`uStatus(00)`添加到请求上下文中；    
b)	`UTraceDubboContextApi.addUTraceContextRequest(String uTraceId)`;传递`uTraceId`到请求上下文中，`uSpanId(0.1)`自动生成，`uStatus(00)`；   
c)	`UTraceDubboContextApi.addUTraceContextRequest(String uTraceId`, `String  uStatus)`;传递`uTraceId，uStatus`到请求上下文件中。    
d)	`UTraceDubboContextApi.addUTraceContextRequest(String uTraceId, String uSpanId,String  uStatus)`;传递`uTraceId`，`uSpanId`，`uStatus`到请求上下文件中。  



#### utrace-httpclient  

1)、	对maven项目中需要在pom.xml中配置UTrace Client插件所需要的依赖包；  

	<dependencies>
		<dependency>
			<groupId>com.hshbic.utrace</groupId>
			<artifactId>utrace-httpclient</artifactId>
			<version>1.1.5</version>
		</dependency>
	</dependencies>

2)、	对于应用spring开源框架的项目中需要对UTrace Client插件中应用的类进行初始化，并加载需要的配置信息，应用插件时在resources添加utrace.xml文件，配置信息如下：  

 
	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
		xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
		<!-- 自动扫描 ,将带有注解的类 纳入spring容器管理 -->
		<context:component-scan base-package="com.hshbic.utrace" />

		<!-- utrace核心类定义，注：id必须为uTrace -->
		<bean id="uTrace" class="com.hshbic.utrace.UTrace" scope="singleton"
			init-method="init">
			<!-- web应用使用下面配置 -->
			<property name="location" value="classpath:utrace_extractor.xml" />
			<!-- paas平台下dubbo使用下面配置 -->
			<property name="config" value="/utrace_extractor.xml" />

		</bean>

		<!-- httpclient -->
		<bean id="uTraceHttpClient" 
				class="com.hshbic.utrace.http.UTraceHttpClientImpl"
			scope="singleton" init-method="init">
			<property name="uTrace" ref="uTrace" />
		</bean>

	</beans>  

3)、	应用插件时在resources添加utrace_extractor.xml文件，配置信息包含节点信息、请求和响应规则解析配置，配置信息如下：  

	<?xml version="1.0" encoding="UTF-8"?>  
	<configuration>

	<!-- 系统名称 -->
	<pair key="sysName" value="UWS" />
	<!-- 服务名称 -->
	<pair key="serviceName" value="uds" />
	<!-- 应用所在节点标识,全局唯一(一个应用可部署于多个节点) -->
	<pair key="nodeIdentifier" value="9789f6b082214463ab70cb4d22158b2f" />
	<!-- 服务节点IP -->
	<pair key="serverIp" value="10.159.32.166"/>
	<!-- 是否开户链路跟踪,值为on/off(空值为on) -->
	<pair key="switch" value="on" />
	<!-- 链路节点状态，根据各应用实际情况选择配置 -->
	<!--S0:起始节点状态,接收到服务请求,请求头包含uTraceId使用此值并且不验证uStatus然后根据名单过滤, 请求头不包含uTraceId的自动生成然后根据名单过滤 
		S1:起始节点状态,接收到服务请求,请求头包含uTraceId使用此值并且不验证uStatus然后根据名单过滤, 请求头不包含uTraceId的不生成日志 
		S2:起始节点状态,接收到服务请求,请求头包含uTraceId使用此值并且验证uStatus, 请求头不包含uTraceId的自动生成然后根据名单过滤 
		S3:起始节点状态,接收到服务请求,请求头包含uTraceId使用此值并且验证uStatus, 请求头不包含uTraceId的自动生成，名单配置无效 
		C0:起始节点状态,客户端发送请求,需要根据名单过滤
		D0:默认节点状态,可不配置,接收到服务请求,请求头包含uTraceId使用此值并且验证uStatus,请求头不包含uTraceId不生成日志,名单配置无效 -->
	<pair key="nodeStatus" value="XX" />

	<!-- httpclient配置 -->
	<pair key="httpclient.maxConnTotal" value="5000" />
	<pair key="httpclient.maxConnPerRoute" value="5000" />
	<pair key="httpclient.connectTimeout" value="30000" />
	<pair key="httpclient.socketTimeout" value="30000" />
	

	<!-- 名单管理 -->
	<pair key="uTraceUrlFilter" value="blacklist" />
	<nlist>
		<value method="GET">/uds/v1/protected/devices/{0}</value>
		<value>/uds/v1/protected/bindDevice</value>
	</nlist>

	<request>
		<element key="reqHttp">
			<element key="reqUrl" append="/" replace="true" notation="{0}"  split="true">
				<value format="/uds/v1/users/{0}/devices/{0}" method="regex">,reqMAC</value>
				<value format="/uds/v1/protected/{0}/*" method="regex">reqMAC</value>
				<filter>
					<value>/uds/v1/protected/bindDevice</value>
				</filter>
			</element>
			<element key="reqBody">
				<value format="deviceId" method="json">reqMAC</value>
				<value format="devices[id]" method="json">reqMAC</value>
			</element>
		</element>
	</request>

	<!-- 响应实例规则 -->
	<response>
		<element key="http">
			<bean key="java.lang.String" method="json">
				<value format="retCode">retCode</value>
				<value format="retInfo">retInfo</value>
			</bean>
		</element>
	</response>

	<!-- 业务信息 -->
	<business>
	</business>

	</configuration>  



  
4)、	插件类实例引用  

a)	通过属性引用方式：  

	<bean id="xxxService" 
		class="com.hshbic.service.client.impl.xxxServiceImpl">  
		<property name="uTraceHttpClient" ref="uTraceHttpClient"/>  
	</bean>  

b)	通过注解方式：  

	@Autowired
	private UTraceHttpClient uTraceHttpClient;  

5)、	utrace-httpclient实例请求应用；  

插件包含get、post、put、delete方法  

![url配置图片12][urlConf12]  

方法返回的UTraceHttpClientResponse其中包含status(状态码)、headers(响应头)、body(返回结果体)；  



#### m2m-common-rest-client  

注：此插件在一些老项目中在使用，为减少引用插件所引发的问题，在原来的插件基础上配置拦截器；  
1)、	对maven项目中需要在pom.xml中配置UTrace Client插件所需要的依赖包；  
 
	<dependencies>
		<dependency>
			<groupId>com.hshbic.cloud.m2m</groupId>
			<artifactId>m2m-common-rest-client</artifactId>
			<version>1.1.5-SNAPSHOT</version>
		</dependency>
	</dependencies>  

2)、	对于应用spring开源框架的项目中需要对UTrace Client插件中核心类进行初始化，并加载需要的配置信息,；  

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
		xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

		<!-- 自动扫描 ,将带有注解的类 纳入spring容器管理 -->
		<context:component-scan base-package="com.hshbic.utrace" />

		<!-- utrace核心类定义，注：id必须为uTrace -->
		<bean id="uTrace" class="com.hshbic.utrace.UTrace" 	scope="singleton"
			init-method="init">
			<property name="location" 	value="classpath:utrace_extractor.xml" />
		</bean>
	</beans>  

3)、	应用插件时在resources添加utrace_extractor.xml文件，配置信息包含节点信息、请求和响应规则解析配置，配置信息如下：  



	<?xml version="1.0" encoding="UTF-8"?>  
	<configuration>
		<!-- 系统名称 -->
		<pair key="sysName" value="UWS" />
		<!-- 服务名称 -->
		<pair key="serviceName" value="uds" />
		<!-- 应用所在节点标识,全局唯一(一个应用可部署于多个节点) -->
		<pair key="nodeIdentifier" value="9789f6b082214463ab70cb4d22158b2f" />
		<!-- 服务节点IP -->
		<pair key="serverIp" value="10.159.32.166"/>
		<!-- 是否开户链路跟踪,值为on/off(空值为on) -->
		<pair key="switch" value="on" />
		<!-- 链路节点状态，根据各应用实际情况选择配置 -->
		<!--S0:起始节点状态,接收到服务请求,请求头包含uTraceId使用此值并且不验证uStatus然后根据名单过滤, 请求头不包含uTraceId的自动生成然后根据名单过滤 
		S1:起始节点状态,接收到服务请求,请求头包含uTraceId使用此值并且不验证uStatus然后根据名单过滤, 请求头不包含uTraceId的不生成日志 
		S2:起始节点状态,接收到服务请求,请求头包含uTraceId使用此值并且验证uStatus, 请求头不包含uTraceId的自动生成然后根据名单过滤 
		S3:起始节点状态,接收到服务请求,请求头包含uTraceId使用此值并且验证uStatus, 请求头不包含uTraceId的自动生成，名单配置无效 
		C0:起始节点状态,客户端发送请求,需要根据名单过滤
		D0:默认节点状态,可不配置,接收到服务请求,请求头包含uTraceId使用此值并且验证uStatus,请求头不包含uTraceId不生成日志,名单配置无效 -->
		<pair key="nodeStatus" value="XX" />	
		<!-- 名单管理 -->
		<pair key="uTraceUrlFilter" value="blacklist" />
		<nlist>
			<value method="GET">/uds/v1/protected/devices/{0}</value>
			<value>/uds/v1/protected/bindDevice</value>
		</nlist>
		<request>
			<element key="reqHttp">
				<element key="reqUrl" append="/" replace="true" notation="{0}"	split="true">
					<value format="/uds/v1/users/{0}/devices/{0}" method="regex">,reqMAC</value>
					<value format="/uds/v1/protected/{0}" method="regex">reqMAC</value>
					<filter>
						<value>/uds/v1/protected/bindDevice</value>
					</filter>
				</element>
				<element key="reqBody">
					<value format="deviceId" method="json">reqMAC</value>
					<value format="devices[id]" method="json">reqMAC</value>
				</element>
			</element>
		</request>
		<!-- 响应实例规则 -->
		<response>
			<element key="http">
				<bean key="java.lang.String" method="json">
				<value format="retCode">retCode</value>
				<value format="retInfo">retInfo</value>
				</bean>
			</element>
		</response>
		<!-- 业务信息 -->
		<business>
		</business>
	</configuration>
 


4)、	在spring配置文件中配置restClient实例，服务引用restClient实例时，通过属性引用方式：  

	<bean id="restClient"
	class="com.hshbic.m2m.rest.impl.SimpleJsonRestClient" 
	init-method="init">
			<property name="uTrace" ref="uTrace"/>
		</bean>

	<bean id="xxxService" 
		class="com.hshbic.service.client.impl.xxxServiceImpl">
			<property name="restClient" ref="restClient"/>
	</bean>   

#### utrace-mybatis-interceptor  

1)、	对maven项目中需要在pom.xml中配置UTrace Client插件所需要的依赖包；  

 

	<dependencies>
		<dependency>
			<groupId>com.hshbic.utrace</groupId>
			<artifactId>utrace-mybatis-interceptor</artifactId>
			<version>1.1.5</version>
		</dependency>
	</dependencies>


2)、	对于应用spring开源框架的项目中需要对UTrace Client插件中应用的类进行初始化，并加载需要的配置信息，应用插件时在resources添加utrace.xml文件，配置信息如下：  

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
		xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

		<!-- 自动扫描 ,将带有注解的类 纳入spring容器管理 -->
		<context:component-scan base-package="com.hshbic.utrace" />

		<!-- utrace核心类定义，注：id必须为uTrace -->
		<bean id="uTrace" class="com.hshbic.utrace.UTrace" scope="singleton"
			init-method="init">
			<property name="location" value="classpath:utrace_extractor.xml" />
		</bean>

	</beans>

3)、	应用插件时在resources添加utrace_extractor.xml文件，配置信息包含节点信息、请求和响应规则解析配置，配置信息如下：  

   
	<?xml version="1.0" encoding="UTF-8"?>  
	<configuration>
		<!-- 系统名称 -->
		<pair key="sysName" value="UWS" />
		<!-- 服务名称 -->
		<pair key="serviceName" value="uds" />
		<!-- 应用所在节点标识,全局唯一(一个应用可部署于多个节点) -->
		<pair key="nodeIdentifier" value="9789f6b082214463ab70cb4d22158b2f" />
		<!-- 服务节点IP -->
		<pair key="serverIp" value="10.159.32.166"/>
		<!-- 是否开户链路跟踪,值为on/off(空值为on) -->
		<pair key="switch" value="on" />
		<!-- 链路节点状态，根据各应用实际情况选择配置 -->
		<!--S0:起始节点状态,接收到服务请求,请求头包含uTraceId使用此值并且不验证uStatus然后根据名单过滤, 请求头不包含uTraceId的自动生成然后根据名单过滤 
		S1:起始节点状态,接收到服务请求,请求头包含uTraceId使用此值并且不验证uStatus然后根据名单过滤, 请求头不包含uTraceId的不生成日志 
		S2:起始节点状态,接收到服务请求,请求头包含uTraceId使用此值并且验证uStatus, 请求头不包含uTraceId的自动生成然后根据名单过滤 
		S3:起始节点状态,接收到服务请求,请求头包含uTraceId使用此值并且验证uStatus, 请求头不包含uTraceId的自动生成，名单配置无效 
		C0:起始节点状态,客户端发送请求,需要根据名单过滤
		D0:默认节点状态,可不配置,接收到服务请求,请求头包含uTraceId使用此值并且验证uStatus,请求头不包含uTraceId不生成日志,名单配置无效 -->
		<pair key="nodeStatus" value="XX" />
	</configuration>
 
4)、	通过spring集成mybatis创建session工厂，增加拦截器插件：  

	<bean id="sqlSessionFactory" 
			class="org.mybatis.spring.SqlSessionFactoryBean">
	    <!-- SqlSessionFactory的数据源 -->
	    <property name="dataSource" ref="dataSource" />
	    <!-- 配置Mybatis的插件plugin-->
	    <property name="plugins">
			<array>
				<bean class="com.hshbic.utrace.mybatis.UTraceMybatisPlugin">
					<constructor-arg ref="uTrace"></constructor-arg>
				</bean>
			</array>
		</property>  
	</bean>



#### utrace-independence  

1)、	对maven项目中需要在pom.xml中配置UTrace Client插件所需要的依赖包；  

	<dependencies>
		<dependency>
			<groupId>com.hshbic.utrace</groupId>
			<artifactId>utrace-independence</artifactId>
			<version>1.1.5</version>
		</dependency>
	</dependencies>  

2)、	utrace独立组件，不依赖其它核心类和插件；  

`dubbo`消费者提供了三个方法，如下：  
e)	`UTraceDubboContextApi.addUTraceContextRequest()`;自动生成`uTraceId(UUID)`，`uSpanId(0.1)`，`uStatus(00)`添加到请求上下文中；  
f)	`UTraceDubboContextApi.addUTraceContextRequest(String uTraceId)`;传递`uTraceId`到请求上下文中，`uSpanId(0.1)`自动生成，`uStatus(00)`；  
g)	`UTraceDubboContextApi.addUTraceContextRequest(String uTraceId, String  uStatus)`;传递`uTraceId`，`uStatus`到请求上下文件中。  
h)	`UTraceDubboContextApi.addUTraceContextRequest(String uTraceId, String uSpanId,String  uStatus)`;传递`uTraceId`，`uSpanId`，`uStatus`到请求上下文件中。  


### 日志收集

?> 使用logback开源组件进行日志收集，logback.xml配置信息如下：  


	<property name="APP_NAME" value="uds"/>
 		<!-- 服务链式跟踪日志 start -->
		<appender name="UTraceLoggingReporter" class="ch.qos.logback.core.rolling.RollingFileAppender">
        	<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            	<fileNamePattern>
				/export/logs/oss/oss-${APP_NAME}-%d{yyyy-MM-dd}.%i.json.log
				</fileNamePattern>
           		<maxHistory>7</maxHistory>
            	<timeBasedFileNamingAndTriggeringPolicy            class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                	<maxFileSize>100MB</maxFileSize>
           		</timeBasedFileNamingAndTriggeringPolicy>
        	</rollingPolicy>
        	<layout class="ch.qos.logback.classic.PatternLayout">
            	<Pattern>%msg%n</Pattern>
        	</layout>
    	</appender>
		<appender name="AsyncUTraceLoggingReporter" 
			class="ch.qos.logback.classic.AsyncAppender">
			<!-- 不丢失日志.默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志 -->
			<discardingThreshold>0</discardingThreshold>
			<!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->
			<queueSize>100000</queueSize>
			<!-- 添加附加的appender,最多只能添加一个 -->
			<appender-ref ref="UTraceLoggingReporter" />
		</appender>
		<logger name="com.hshbic.utrace.reporter.UTraceLoggingReporter" additivity="false">
			<level value="info" />
			<appender-ref ref="AsyncUTraceLoggingReporter" />
		</logger>
		<!-- 服务链式跟踪日志 end -->


注：property配置的APP_NAME属性的值与utrace.xml中配置的serviceName相同；或不配置property直接使用serviceName配置的值。  

### 获取uTrace相关信息  

首先服务端要配置相关的组件`(utrace-web-servlet-filter、utrace-dubbo)`；  
`String uTraceId = UTraceDataOpen.getUTraceId()`;  
`String uSpanId = UTraceDataOpen.getUSpanId()`;  
`String sn = UTraceDataOpen.getSn()`;  
注：  
如果配置服务端组件，以上方法是取得服务端请求的sn信息；  
没有配置服务端组件，以上方法是取得最后请求接口的sn信息；  



## 链式跟踪日志schema  

<table>
<tr>
    <td bgcolor="#90EE90"><b>信息域</b></td>
 	<td bgcolor="#90EE90"><b>日志信息</b></td>
	<td bgcolor="#90EE90"><b>审计schema</b></td>
</tr>
<tr>
    <td rowspan="4">链路信息（base）</td>
    <td>uBId</td>
	<td>bId</td>
</tr>
<tr>
    <td>uTraceId</td>
	<td><font color=FF0000>tId</font></td>
</tr>
<tr>
    <td>uSpanId</td>
	<td><font color=FF0000>sId</font></td>
</tr>
<tr>
    <td>uFromType</td>
	<td>step</td>
</tr>
<tr>
    <td rowspan="6">节点信息（node）</td>
    <td>sysName</td>
	<td>sys</td>
</tr>
<tr>
    <td>serviceName</td>
	<td>subSys</td>
</tr>
<tr>
    <td>nodeIdentifier</td>
	<td>nId</td>
</tr>
<tr>
    <td>serviceType</td>
	<td>prot</td>
</tr>
<tr>
    <td>serverIp</td>
	<td>nIP</td>
</tr>
<tr>
    <td>appId</td>
	<td>aId</td>
</tr>
<tr>
    <td rowspan="3" >时间信息（time）</td>  
    <td>reqTime</td>
	<td  rowspan="2">ts</td>
</tr>
<tr>
    <td>respTime</td>
</tr>
<tr>
    <td>duration</td>
	<td>span</td>
</tr>

<tr>
    <td rowspan="9">http请求（reqHttp）</td>
    <td>reqBytes</td>
	<td><font color=FF0000>rqBt</font></td>
</tr>
<tr>
    <td>reqIP</td>
	<td><font color=FF0000>rqBt</font></td>
</tr>
<tr>
    <td>reqHeader</td>
	<td><font color=FF0000>rqHd</font></td>
</tr>
<tr>
    <td>reqBody</td>
	<td rowspan="2">ipm</td>
</tr>
<tr>
    <td>reqParams</td>
</tr>
<tr>
    <td>reqUrl</td>
	<td>bName</td>
</tr>
<tr>
    <td>reqMode</td>
	<td> </td>
</tr>

<tr>
    <td>reqUserId</td>
	<td>uId</td>
</tr>

<tr>
    <td>reqMAC</td>
	<td>dId</td>
</tr>
<tr>
    <td rowspan="3">Dubbo请求（reqRPC）</td>
    <td>reqMethod</td>
	<td>bName</td>
</tr>
<tr>
    <td>reqParams</td>
	<td>ipm</td>
</tr>
<tr>
    <td>reqContext</td>
	<td><font color=FF0000>rqCt</font></td>
</tr>

<tr>
    <td rowspan="2">数据库请求（reqDB）</td>
    <td>sql</td>
	<td><font color=FF0000>sql</font></td>
</tr>
<tr>
    <td>dbParams</td>
	<td>ipm</td>
</tr>
<tr>
    <td rowspan="6">请求响应（response）</td>
    <td>respBytes</td>
	<td><font color=FF0000>rpBt</font></td>
</tr>
<tr>
    <td>respBody</td>
	<td>rrt</td>
</tr>
<tr>
    <td>retCode</td>
	<td>code</td>
</tr>  
<tr>
    <td>retInfo</td>
	<td>desc</td>
</tr>
<tr>
    <td>exception</td>
	<td>exp</td>
</tr>  

<tr>
    <td>status</td>
	<td><font color=FF0000>status</font></td>
</tr>

</table>



##  插件使用注意说明  

?> 引入UTrace client插件的，pom.xml文件中可以不引入审计插件base-log-plugin*.jar；  




    


[^-^]:常用图片注释  
[UTraceClient]:_media/_attachment/UTraceClient.jpg 
[ConfigurationInformation]:_media/_attachment/ConfigurationInformation.jpg  
[Httpclient]:_media/_attachment/Httpclien.jpg 
[blacklist]:_media/_attachment/blacklist.jpg  
[DataParsing]:_media/_attachment/DataParsing.jpg  
[urlConf]:_media/_attachment/urlConf.jpg 
[urlConf2]:_media/_attachment/urlConf2.jpg     
[urlConf3]:_media/_attachment/urlConf3.jpg
[urlConf4]:_media/_attachment/urlConf4.png 
[urlConf5]:_media/_attachment/urlConf5.png 
[urlConf6]:_media/_attachment/urlConf6.png 
[urlConf7]:_media/_attachment/urlConf7.png
[urlConf8]:_media/_attachment/urlConf8.png
[urlConf9]:_media/_attachment/urlConf9.png
[urlConf10]:_media/_attachment/urlConf10.png
[urlConf11]:_media/_attachment/urlConf11.png 
[urlConf12]:_media/_attachment/urlConf12.png