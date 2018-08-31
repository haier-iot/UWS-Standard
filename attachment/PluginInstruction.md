# 基础日志插件使用说明书  


!>  **当前版本**：基础日志插件使用说明书 V1.3.0   
 **更新时间**：2018-05-28  

## 引言  
### 目的  

?> 本说明书简单介绍了插件的使用方式，主要用于调用者写业务日志。  

## 特别说明  

?> 用户隐私信息禁止收集。若出现问题，由用户隐私信息的日志产生方承担相关责任。  

## 插件使用前提  

?> 开发的服务系统在使用日志插件前，请先与负责日志插件使用的<font color=FF0000>**龚学纲（gongxuegang.uh@haier.com）**</font>进行联系沟通获取日志版本以及日志插件的版本等内容：  

- **日志schema版本**  
>  具体使用的日志schema版本与格式定义；  
 
- **插接版本**  
>  具体使用基础日志插件jar包的版本；  

- **日志输出规则**   
>  应用系统接入日志插件，输出日志的打点粒度、文件命名规则、输出路径等输出规则；  


- **业务代码登记**   
>  各系统根据自身的业务需求，自定义业务代码及代码的含义，在运营支撑系统进行登记；   


## 插件使用示例说明  
### 基本信息说明  
?> **日志实体类：**`LogEntity`  
  **涉及到的枚举字段：**  
	`Sys：m2m, AGS, UWS, CD, MP, OSS, IFTTT`    
	`Frm：haier, ali, jd, tx`    
	`Prot：Http, dubbo, db, kafka, app、rpc、local、thrift、mqtt、wifi、uwt、coap`    
	`RqMod：get, post, put, delete`    
	`Step：CS, SR, SS, CR, DI`    
 **枚举字段简介：**  
1、 Sys字段代表系统名称、Frm文件子系统名称，具体传入参数可根据各个系统的实际情况。
2、 Step字段记录流程的各个节点，现在对应5个枚举变量。其中各变量代表的具体内容为：  

- CS（Client Send）客户端发起请求，生成调用上下文；  
- SR（Server Recieve）服务端接收请求，并生成上下文；  
- DI业务端自定义埋点自己感兴趣的数据；  
- SS（Server Send）服务端返回请求结果，归档上下文；  
- CR（Client Receive）客户端接收返回结果，归档上下文。  
 
3、Port字段包含各种传输协议，具体请根据各系统的具体协议来传相应的参数值。  
4、日志监控的其他详细字段请见“日志Schema”。  


### maven配置信息{index4.2}

?> Maven项目首先配置pom.xml，添加基础日志插件依赖，目前版本1.0.9：  
   
```java   
	<dependency>
    	<groupId>com.hshbic.cloud.log</groupId>
        <artifactId>base-log-plugin</artifactId>
        <version>1.0.9</version>
	</dependency>

```    

### 实例配置信息{index4.3} 

?> 配置base-log-plugin.xml，通过spring bean方式定义和配置插件，并配置依赖关系：  
注：红色标注为注释，黄色标注为各位根据需求需要修改的值，其他信息均不用变动。  

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
		xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
		xsi:schemaLocation="http://www.springframework.org/schema/beans classpath:/org/springframework/beans/factory/xml/spring-beans-4.0.xsd
					http://www.springframework.org/schema/context classpath:/org/springframework/context/config/spring-context-4.0.xsd
					http://www.springframework.org/schema/aop classpath:/org/springframework/aop/config/spring-aop-4.0.xsd
					http://www.springframework.org/schema/tx classpath:/org/springframework/transaction/config/spring-tx-4.0.xsd">
																								
		<!-- 自动扫描 ,将带有注解的类 纳入spring容器管理 -->
		<context:component-scan base-package="com.hshbic.cloud.log" />
		
		<!-- 枚举类型转换 -->
		<!-- 字段Sys（系统）为枚举类型
			枚举类:com.hshbic.cloud.audit.Sys
			枚举值：m2m, AGS, UWS, CD, MP, OSS, IFTTT
			各位根据需求自行配置
		-->
		<bean id="m2m" class="org.springframework.beans.factory.config.FieldRetrievingFactoryBean">    
		<property name="staticField" value="com.hshbic.cloud.log.Sys.m2m" />    
		</bean>
		<!-- 字段Frm(日志源)为枚举类型
			枚举类：com.hshbic.cloud.audit.Frm
			枚举值：haier, ali, jd, tx
			各位根据需求自行配置	
		 -->
		<bean id="haier" class="org.springframework.beans.factory.config.FieldRetrievingFactoryBean">    
		<property name="staticField" value="com.hshbic.cloud.log.Frm.haier" />    
		</bean>
	
		<!-- 其中Sys和Frm的ref引用上面的转换bean，ver根据自身需求进行配置 -->
		<bean id="LogPlugin" class="com.hshbic.cloud.log.LogPlugin"> 
			<property name="Sys" ref="m2m"></property> 
			<property name="ver" value="v1.0.2"></property> 
			<property name="Frm" ref="haier"></property> 
		</bean>
	</beans>  

### Web应用容器配置{index4.4}  
?> 配置web应用的web.xml：  

```java  

<!-- 声明WEB应用上下文初始化参数，参数定义了要装入的配置文件 -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:base-log-plugin.xml</param-value>
	</context-param>
<!-- 监听器自动装配Spring相关的配置信息 -->
	<listener>  
		<listener-class>
			org.springframework.web.context.ContextLoaderListener
		</listener-class>
	</listener>


```  


## 使用方法  

### 使用Spring的工程  

#### 步骤一：进行maven配置  

具体信息见 [maven配置信息](index4.2)

#### 步骤二：配置文件装载方法  

##### 方法一  

直接将base-log-plugin.xml里面定义的bean拷贝到自己工程下的applicationContext.xml下，（即本工程已经加载的配置文件中）  

##### 方法二  

1、在web.xml中配置文件装载，见[Web应用容器配置](index4.4)  
2、在resources文件夹下添加base-log-plugin.xml文件，配置信息如[实例配置信息](index4.3)

#### 步骤三：方法的使用  

##### 方法的引入  

a)	通过属性引用方式：    

	<bean id="xxxService" 
		class="com.hshbic.service.client.impl.xxxServiceImpl">
		<property name="logplugint" ref="LogPlugin"/>
	</bean>  

b)	通过注解方式：  

	@Autowired
	private LogPlugin logplugint;  


##### 方法的调用  

	LogEntity entity = new LogEntity();
	entity.setnId("1111111111");
	entity.setProt(Prot.valueOf("dubbo"));
	.
	.
	.
	String json = logplugint.formatLog(entity);

注意：请大家务必按照文档中的要求set字段，类型、必填项等


#### 步骤四：日志文件的写入  

日志文件名规范：`oss-%subSys-%d{yyyy-MM-dd}.%i.json.log`  
1)	`%subSys`：子系统名，区分不同应用，尽量避免同一系统存在不同应用同时写入的相互冲突  
2)	`%d{yyyy-MM-dd}`：日期格式  
3)	`%i`：文件大小每百兆自增序号  
日志文件目录：`/export/logs/oss/`  

##### Log4j  

1)、	log4j的配置  

```java
log4j.logger.baseLog =info , baseLog  
log4j.appender.baseLog =org.apache.log4j.RollingFileAppender  
log4j.appender.baseLog.File= oss-%subSys-%d{yyyy-MM-dd}.%i.json.log  
log4j.appender.baseLog.MaxFileSize=100M  
log4j.appender.baseLog.layout=org.apache.log4j.PatternLayout  
log4j.appender.baseLog.layout.ConversionPattern=%m%n  
```
2)、	调用方式  

```java  
final static Logger auditLogger = Logger.getLogger("baseLog");
LogEntity entity = new LogEntity();
entity.setnId("1111111111");
entity.setProt(Prot.valueOf("dubbo"));
.
.
.
String json = logplugint.formatLog(entity);
auditLogger.info(json);
```  
##### logback  

使用logback开源组件进行日志收集，logback.xml配置信息如下：  

```java  

<appender name="baseLog" 
		class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy 
		class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <fileNamePattern>
               /export/logs/oss/oss-%subSys-%d{yyyy-MM-dd}.%i.json.log
		</fileNamePattern>
            <maxHistory>15</maxHistory>
            <timeBasedFileNamingAndTriggeringPolicy      					  		 class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
            	<maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <Pattern>%msg%n</Pattern>
        </layout>
</appender>
<root level="info">
<appender-ref ref="baseLog" />
</root>

```  
### 未使用Spring的工程  

```java  

//使用构造函数
//LogPlugin(Sys sys, String ver, Frm frm, String nId, String subSys, String nIP)
LogPlugin plgin = new LogPlugin(Sys.valueOf("m2m"), "v1.2.3", Frm.valueOf("haier"), "123.4.12.3", "uds", "127.0.0.1");
//基础信息类
LogObj log = new LogObj();
log.setdId("diddddd");
.
.
.
String json = plgin.format2Log(log);
```  





##  日志schema  


<table>
<tr>
    <td>区域</td>
    <td>文字段名称</td>
    <td>字段含义</td>
	<td>是否必填</td>
	<td>数据范围</td>
</tr>
<tr>
    <td rowspan="27">基本信息</td>
    <td>sn</td>
	<td>流水号</td>
	<td>必填，不能填空串</td>
	<td>字符型，最多64位</td>
</tr>
<tr>
    <td>sys</td>
	<td>所属系统</td>
	<td>必填，不能填空串</td>
	<td>字符型，最多64位</td>
</tr>
<tr>
    <td>ts</td>
	<td>时间戳</td>
	<td>必填</td>
	<td>整数型</td>
</tr>
<tr>
    <td>type</td>
	<td>日志类别</td>
	<td>必填</td>
	<td>字符型</td>
</tr>
<tr>
    <td>ver</td>
	<td>日志版本</td>
	<td>必填，不能填空串</td>
	<td>字符型，最多16位</td>
</tr>
<tr>
    <td>key</td>
	<td>日志Id</td>
	<td>必填</td>
	<td>字符型，最多64位</td>
</tr>
<tr>
    <td>frm</td>
    <td>日志源</td>
	<td>必填，不能填空串</td>
    <td>字符型</td>
</tr>
<tr>
    <td>code</td>
    <td>返回码</td>
	<td>必填，不能填空串</td>
    <td>字符型，最多10位</td>
</tr>
<tr>
    <td>span</td>
    <td>处理时间</td>
	<td>不必填</td>
    <td>整数型</td>
</tr>
<tr>
    <td>aId</td>
    <td>应用Id</td>
	<td>必填，可以填空串</td>
    <td>字符型，最多32位</td>
</tr>
<tr>
    <td>uId</td>
    <td>用户Id</td>
	<td>必填，可以填空串</td>
    <td>字符型，最多33位</td>
</tr>
<tr>
    <td>tk</td>
    <td>token</td>
	<td>必填，可以填空串</td>
    <td>字符型，最多64位</td>
</tr>

<tr>
    <td>bName</td>
    <td>业务(接口)名称</td>
	<td>必填，可以填空串</td>
    <td>字符型，最多64位</td>
</tr>

<tr>
    <td>dId</td>
    <td>设备Id</td>
	<td>必填，可以填空串</td>
    <td>字符型，最多32位</td>
</tr>

<tr>
    <td>nId</td>
    <td>节点信息</td>
	<td>必填，可以填空串</td>
    <td>字符型，最多32位</td>
</tr>

<tr>
    <td>step</td>
    <td>处理阶段</td>
	<td>必填，可以填空串</td>
    <td>字符型，最多8位</td>
</tr>

<tr>
    <td>subSys</td>
    <td>子系统名称</td>
	<td>必填，可以填空串</td>
    <td>字符型，最多16位</td>
</tr>

<tr>
    <td><s>rqMod</s></td>
    <td><s>请求方式</s></td>
	<td><s>不必填，不能填空串</s></td>
    <td><s>字符型，最多8位</s></td>
</tr>

<tr>
    <td>typeId</td>
    <td>wifitype</td>
	<td>不必填</td>
    <td>字符型，最多128位</td>
</tr>

<tr>
    <td>dType</td>
    <td>八位码</td>
	<td>不必填</td>
    <td>字符型，最多8位</td>
</tr>

<tr>
    <td>bId</td>
    <td>关键业务代码</td>
	<td>必填，可以填空串</td>
    <td>字符型，最多4位</td>
</tr>

<tr>
    <td>prot</td>
    <td>协议类型</td>
	<td>必填，可以填空串</td>
    <td>字符型，最多8位</td>
</tr>

<tr>
    <td>nIP</td>
    <td>应用所在服务器的IP</td>
	<td>必填，可以填空串</td>
    <td>字符型，最多32位</td>
</tr>

<tr>
    <td>tags</td>
    <td>日志标签</td>
	<td>不必填</td>
    <td>字符型，最多256位</td>
</tr>

<tr>
    <td>ipm</td>
    <td>输入参数</td>
	<td>不必填</td>
    <td>字符型，最多512位</td>
</tr>

<tr>
    <td>rrt</td>
    <td>返回结果</td>
	<td>不必填</td>
    <td>字符型，最多512位</td>
</tr>


<tr>
    <td>exp</td>
    <td>异常</td>
	<td>不必填</td>
    <td>字符型，最多512位</td>
</tr>

<tr>
	<td rowspan="2">高级信息</td>
    <td>args</td>
    <td>可选参数</td>
	<td>不必填</td>
    <td>对象类型</td>
</tr>
<tr>
    <td>desc</td>
    <td>日志记录描述</td>
	<td>不必填</td>
    <td>字符型，最多65535位</td>
</tr>
</table>

























































































