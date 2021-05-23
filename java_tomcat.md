JAVA:
	包含四个独立却又彼此相关的技术
		java程序设计语言
		java API
		java Class文件格式
		JVM：java virtual machine

============================================
	java程序语言	  java API
			  |	 /
           1.java
			  |	
		java complier	
			  |
			1.class   公共类库(Class loader)  
			  |       /
			JVM <————/  		
============================================
	web container:

	     .jsp---------> .java -------> (JVM).class
	    	|	           |              |
	      JSP(jsper)    servlet      JDK:javac,.java-->.class	

====================================================================

 Sun公司创建了第一个Servlet容器，即Java Web Server,但fws只是为了演示Servlet的相应功能，所以其很不稳定。与此同时，ASF创建了JServ项目，一个能够与apache整合起来的servlet容器。1999年，Sun反JWS捐给了ASF，于是两个项目被命名为Catalina。

 Java SE包含了Java二进制程序（如JVM和Java字节码编译器）和Java的核心代码库，而Java EE标准则包含了一组适用于创建企业级Web应用程序的API，Java EE建立在Java SE的基础上，并依赖于Java SE才能正常工作。当然，任何级别的应用程序均能从Java EE中获益，但Java EE却更适合解决大型软件系统设计中的问题。

	  *Java SE APIs:
	   ·JNDI(Java Naming and Directory Interface)
	   ·JAXP(Java API for XML Pressing)
	
	  JAVA EE包含多个独立的API，Servlet和JSP就是其中的两个，而JAVA EE中著名的API中还包含如下的几个：
	  *JAVA EE APIs
	   ·EJB(Enterprise JavaBeans):
	   ·JMS(Java Message Service):
	   ·JMX(Java Management Extensions):
	   ·JTA(Java transcation API):
	   ·JavaMail:

	 JAVA EE Application Servers:
	 	·Websphere
	  	·Weblogic
	  	·oc4j
	  	·JBoss
	  	·JOnAS
	  	·Geronimo
	  	·Glassfish 
	 开源实现:
		Tomcat
		Jetty
		Resin


   安装JDK
	#二进制格式直接解压使用或rpm安装	
	yum list all | grep openjdk
	yum instal java(java-1.8.0-openjdk)

	配置JAVA环境变量：vi /etc/profile.d/java.sh
					JAVA_HOME=/usr/java
					PATH=$PATH:$JAVA_HOME/bin

	Sun JDK监控故障处理工具：
		·jps（JVM Precess Status Tool）:显示指定系统内所有的HotSpot虚拟机进程的信息
		·jstat(JVM Statistice Monitoring Tool):显示HotSpot虚拟机运行状态
		·jinfo:显示HotSpot虚拟机配置信息
		·jmap：生成某HotSpot

 TOMCAT:是J2EE的一种不完整实现
   安装:
	#二进制格式直接解压使用或rpm安装
	yum list all | grep tomcat // tar xf apache-tomcat-7.0.33.tar.gz /usr/local
	yum install tomcat		   // ln -sv /usr/local/apache-tomcat-7.0.33 /usr/local/tomcat	
			
   tomcat 自带的应用程序：
	manager app:webapp管理工具
	host manager:Virtual Host管理工具

  Web container:

   JAVA WebAPP 组织结构：
	有特定的组织形式、层次型的目录结构：主要包含了servlet代码文件、JSP页面文件、类文件、部署描述符文件等：
		/usr/local/tomcat/webapps/app1/
			/:webapp的根目录；
			WEB-INF/:当前webapp的私有资源，通常存放当前webapp自用的web.xml;
			META-INF/:当前webapp的私有资源，通常存放当前webapp自用的context.xml;
			classes/:此webapp的私有类；
			lib/:此webapp的私有类，被打包为jar格式类；
			index.jsp:webapp的主页

		webapp归档格式：
			.war webapp; .jar EJB的类; .rar 资源适配器; .ear 企业级应用程序

	手动添加一个测试应用程序：
		1、创建webapp特有的目录结构：
			mkdir -pv myapp/{lib,classes,WEB-INF,META-INF}
		2、提供webapp各文件：
			myapp/index.jsp
				<%@ page language="java" %>
				<%@ page import="java.util.*" %>
				<html>
					<head>
						<title>JSP Test Page</title>
					</head>
				<body>
					<% out.println("Hello,world.") %>
				</body>
			</html>

  部署（deployment)webapp相关的操作：
	deploy:部署，将webapp的源文件旋转于目标目录、配置tomcat服务器能够基于context.xml文件中定义的路径来访问此webapp; 将其特有类通过class loader装载到tomcat;
		有两种方式：自动部署；手动部署
			自动部署: auto deploy
			手动部署
				1、冷部署: 把webapp复制到指定位置,而后才启动tomcat
				2、热部署: 在不停止tomcat的前提下进行的部署
					部署工具:manager、ant脚本、tcd(tomcat client deployer)等


   tomcat的主配置文件结构：
	<server>			//一个运行的tomcat实例
		<service>		//用于关联engine和connector 
			<connector />	//负责连接客户端请求到Servlet容器内的Web应用程序;默认http连接器
			<connector />
			<engine >			
				<host name="">		//多个host用来提供不同的虚拟主机
					<context />		//用来定义部署多个webapp应用程序
					<context />
				</host>
			</engine>
		</service>
	</server>

   #connector组件
	 负责连接客户端请求到Servlet容器内的Web应用程序;
     server.xml中的连接器通常有4种;定义连接器可以使用多种属性,有些属性也只适用于某特定的连接器类型。
	   1.HTTP/1.1连接器,默认;  2.SSL连接器; 3.AJP/1.3连接器

     可定义的连接器属性有:
	  1）address：指定连接器监听的地址，默认为所有地址，即0.0.0.0；
	  2）maxThreads：支持的最大并发连接数，默认为200；
	  3）port：监听的端口，默认为0；
	  4）protocol：连接器使用的协议，默认为HTTP/1.1,定义AJP协议时通常为AJP/1.3;
	  5）redirectPort：如果某连接器支持的协议是HTTP,当接收客户发来的HTTPS请求时，则转发至此属性定义的端口；
	  6）connectionTimeout：等待客户端发送请求的超时时间，单位为毫秒，默认为60000，即1分钟；
	  7）enableLookups：是否通过request.getRemoteHost()进行DNS查询以获取客户端的主机名；默认为true; 
	  8）acceptCont：设置等待队列的最大长度；通常在tomcat所有处理线程均处于繁忙状态时，新发来的请求将被放置于等待队列中；

	下面是一个定义了多个属性的SSL连接器：
	 <Connector port="8443"
		maxThreads="150" minSpareThreads="25" maxSpareThreads="75"
		enableLookups="false" acceptCount="100" debug="0" scheme="https" secure="true"
		clientAuth="false" sslProtocol="TLS" />

	 #Engine组件常用的属性定义：
	  1）defaultHost:
	  2）name
	  3）jvmRoute=

 	 #Host组件常用属性：
	  1）appBase：此Host的webapps目录
	  2）autoDeploy:false :是否自动部署
	  3）uppackWars:true :是否自动展开war文档

	  虚拟主机定义示例：
	  	<Engine name="Catalina" defaultHost="localhost">
		  <Host name="localhost" appBase="webapps">
			<Context path="" docBase="ROOT"/>
			<Context path="/bbs" docBase="/web/bss"
			  reloadable="true" crossContext="true"/>
		  </Host>

		  <Host name="mail.magedu.com" appBase="/web/mail">
			<Context path="/" docBase="ROOT"/>
		  </Host>
		</Enging>

	主机别名定义：
	 如果一个主机有两个或两个以上的主机名，额外的名称均可以以别名的形式进行定义，如下：
	<Host name="www.magedu.com" appBase="webapps" uppackWARs="true">
	  <Alias>magedu.com</Alias>
	</Host>

	 #Context组件：
	  Context在某些意义上类似于apache中的路径别名，一个Context定义用于标识tomcat实例中的一个web应用程序：
		 <!-- Tomcat Root Context -->
		 <Context path="/URL/路径" docBase="/系统/路径"/>

		 <!-- buzzin webapp -->
		 <Context path="/bbs"
			docBase="/web/threads/bbs"
			reloadable="true">		//是否支持重新装载
		 </Context>

	  #Valve组件：
		Valve类似于过滤器，它可以工作于Engine和Host/Context之间、Host和Context之间以及Context和Web应用程序的某资源之间。一个容器内可以建立多个Valve，而且Valve定义的次序也决定了它们生效的次序，Tomcat6中实现了多种不同的Valve:
		 ·AccessLogValve：访问日志Valve
		 ·ExtendedAccessValve：扩展功能的访问日志Valve
		 ·JDBCAccessLogValve：通过JDBC将访问日志信息发送到数据库中；
		 ·RequestDumperValve：请求转储Valve;
		 ·RemoteAddrValve：基于远程主机名称的访问控制；
		 ·RemoteHostValve：基于远程主机名称的访问控制；
		 ·SemaphoreValve：用于控制Tomcat主机上任何容器上的并发访问数量；
		 ·JvmRouteBinderValve：在配置多个Tomcat为以Apache通过mod_proxy或mod_jk作为前端的集群架构中，当期望停止某节点时，可以通过此      	Valve将？请求定向至备用节点；使用此Valve，必须使用JvmRouteSessionIDBinderListener;
		 ·ReplicationValve：专用于Tomcat集群架构中，可以在某个请求的session信息发生更改时触发session数据在各节点间进行复制
		 ·SongleleSignOn：将两个或多个需要对用户进行认证webapp在认证用户时连接在一起，即一次认证即可访问所有连接在一起的webappp;
		 ·ClusterSignleSingOn：对SingleSignOn的扩展专用于Tomcat集群当中，需要结合ClusterSingleSignOnListener进行工作；

		*e.g.
		  RemoteHostValve和RemoteAddrValve可以分别用来实现基于主机名称和基于IP地址的访问控制，控制本身可以通过allow或deny来定义
		  这有点类似于Apache的访问控制功能；如下面的Valve则实现了仅允许本机访问/probe;
			<Context path="/probe" docBase="probe">
			  <Valve className="org.apache.catalina.valves.RemoteAddrVlave"
			  allow="127\.0\.0\.1"/>
			</Context>

			·其中相关属性定义有：
			 1）className：相关的java实现的类名，相应于分别应该为org.apache.catalina.valves.RemoteHostValve或org.apache.catalina.valves.RemoteAddrValve;
			 2）allow：以逗号分开的允许访问的IP地址列表，支持正则表达式，因此，点号“.”用于IP地址时需要转义；仅定义allow项时，非明确  	allow的地址均被deny；
			 3）deny：以逗号分开的禁止访问的IP地址列表，用法同allow;

  LNMT:NGINX做为后端tomcat的反向代理
	client--> nginx--> reverse_proxy--> http--> tomcat(http connector)

	-location / {
    -}
	  ##动静分离##
	-location ~* \.(jsp|do)$ {
	-	proxy_pass http://web.magedu.com:8080;
	-}
	注：path给定的路径不能以“/”结尾；

  LAMT:用apache做反代时一般采用LNAMT构架，(apache的作用类似于作为tomcat的http连接器)由apache负责接收和处理来自前端NGINX的请求,并反代		至后端tomcat；
	配置apache将访问请求反代至后端：
	 #1.proxy_module_http; 2.proxy_module_ajp   #基于http或ajp协议反代至后端
		<VirtualHost *:80>
			ServerName web1.magedu.com
			ProxyVia On
			ProxyRequests Off
			ProxyPreserveHost On
			<Proxy *>
				Require all granted
			</Proxy>
			ProxyPass /status !		##为自己提供status页面
			ProxyPass / {http|ajp}://172.16.1.20:{8080|8009}/	##基于http或ajp协议
			ProxyPassReverse / {http|ajp}://172.16.1.20:{8080|8009}/  ##基于http或ajp协议
			<Location />
				Reauire all granted
			</Location>
		</VirtualHost>


  ####部署shopxx在线商城