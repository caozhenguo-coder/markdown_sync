Nginx的配置：
	main: 全局配置段
		worker_processes 2;
		worker_cpu_affinity 0010 0100; 配置worker进程与cpu的亲缘性
		worker_priority -10;  配置worker进程的优先级(nice值:-20到20）
		worker_rlimit_nofile NUMBER;  worker进程能打开文件数量上限，最大65535
		event{}: 定义event模型工作特性

	http{  ： 定义http协议相关的配置
		 配置指令：要以分号结尾,语法格式:
		 directive value1 [value2...]
			upstream{
				...
			}
			server{
				listen
				server_name
				location URL{
						if{
						}
					}
				}
				server{
				}
			}
		
	ngx_http_proxy_module模块：
			server{
		 		listen
				server_name
				location / {
					proxy_pass http://192.168.1.10:80/;  **如果URI做模式匹配 则代理路径后不能带目录**
				}
			}

	location的正则表达式匹配：
		^~： 对URI的最左部分做匹配检查，不区分字符大小写
		~：  对URI做正则表达式模式匹配，区分字符大小写
		~*： 对URI做正则表达式模式匹配，不区分字符大小写
		=   完全匹配
		匹配优先级从高到低：
			=，^~，~/~*，不带符号

	  try_files file ... uri;
	  try_files file ... =code;
		location /images/ {
				try_files $uri /images/default.gif;
		}
		location / {
			try_files $uri $uri/index.html $uri.html =404;
		}
		 按顺序检查文件是否存在，返回第一个找到的文件或文件夹（结尾加斜线表示为文件夹），如果所有的文件或文件夹都找不到，会进行一个内部重定向到最后一个参数。只有最后一个参数可以引起一个内部重定向，之前的参数只设置内部URI的指向。最重一个参数是回退URI且必须存在，否则会出现内部500错误

	nginx做反向代理:(可异步请求)
		proxy-set-header:x-real-ip		//-->做反向代理时接收请求向后端代理时添加的请求报文首部
						 x-forward-for
		add-header:x-via					//<--做反向代理时响应客户端请求时向响应报文首部添加的信息
				  x-cache	

========================================================================================
  
				#   HTTPD

  http协议:[http客户端工具：curl;elinks]
		http/0.9; http/1.0; http/1.1; http/2.0
	

 
   MPM:multi process module
	  prefork：多进程模型，一个进程响应一个请求；
	  worker：多线程模型(多进程生成，一个进程生成多个线程),一个线程响应一个请求；
	  event：事件驱动模型，一个线程响应多个请求；

   常用配置：
	配置格式：
		directive value
			directive：不区分大小写；
			value：不区分大小写，但为路径时取决于文件系统；
	
	keepalive测试：	
		telnet HOST PORT
		GET /URL HTTP/1.1
		Host: HOSTNAME or IP

 	DirectoryIndex  index.html index.php  #也可在虚拟主机内部定义，则只对该虚拟主机生效
	Listen 80 
	<VirtualHost *:80>
		ServerName www.c.org
		DocumentRoot "/www/c.org/htdocs"
		<Directory "/www/c.org/htdocs"
			Options	None/Indexes/AllowSymLinks
			AllowOverride None
			Require all granted/denied
		</Directory>
	</VirtualHost>


	*站点访问控制常见机制
	   可基于两种机制指明对哪些资源进行何种访问控制

		  ·文件系统路径:
		   	 <Directory "">
			  ...
			 </Directory>
             --------------------
			 <File "">
			 ...
			 </File "">	
			 --------------------
			 <FileMatch "PATTERN">
			 ...
			 </FileMatch "PATTERN">
		  ·URL路径：
			 <Location "">
			 ...
			 </Location>
			--------------------
			 <LocationMatch "">
			 ...
			 </LocationMatch>	


 	基于来源地址的访问控制机制
		Order allow,deny
		Allow from all
		Deny from all
			来源IP：
				172.16
				172.16.0.0
				172.16.0.0/16
				172.16.0.0/255.255.0.0

		访问日志记录格式：
			2016 马哥Linux运维就业班（第一弹）p70

		路径别名
			Alias /URL/ "PATH/TO/SOMEDIR/"

  https:{2018linux高端运维shell编程(上);p80}

	yum install mod_ssl

	 CA ~]# cd /etc/pki/CA/
	 CA ~]# (umask 077;openssl genrsa -out private/cakey.pem 2048)		//生成密钥
	 CA ~]# openssl req -new -x509 -key private/cakey.pem -out cacert.pem -days 365	//CA自签证书
	 CA ~]# openssl ca -in /the_copy_csr.pem -out certs/httpd_crt.pem

	  ~]# openssl s_client -connect www.abc.com:443 -CAfile /CAfile/TO/cacert.pem  //openssl工具访问https站点

   httpd-manual:本地帮助文档
		yum install httpd-manual
		  重启httpd服务
			访问:http://localhost/manual


		认证质询：
			www-Authenticate:响应码为401，拒绝客户端请求，并说明要求客户端提供账号和密码；
		认证：
			Authorization:客户端用户填入账号和密码后再次发送请求报文；认证通过时，则服务器发送响应的资源；
		  认证方式有两种：
			basic：明文
			digest：消息摘要认证
		basic认证配置示例:
			1)定义安全域

   	httpd认证:basic|digst
		<Directory "">
			Options None
			AllowOverride None
			AuthType Basic
			AuthName "STRING"
			AuthUserFile "/PAHT/TO/HTTPD_USER_PASSWD_FILE"
			AuthGroupFile "/PAHT/TO/HTTPD_GROUP_FILE"
			Require user|group username1|group1 username2|group2 ... | Require valid-user
		</Directory>
			组文件：每一行定义一个组
				GROUP_NAME: user1 user2 ...
	
	 	使用htpasswd命令进行管理
			htpasswd [options] passwordfile username
				-c：自动创建passwordfile,因此，仅应该在添加第一个用户时使用；
				-m：md5加密用户密码
				-s：sha1加密用户密码
				-D：删除指定用户

	*status页面
		LoadModule staus_module modules/mod_status.so

		httpd-2.2
			<Location /server-status>
				SetHandler server-status
				Order allow,deny   [设置默认deny]
				Allow from 172.16
			</Location>

		httpd-2.4
			<Location /server-status>
				SetHandler server-status
				<RequireAll>
					Require ip 172.16
				</RequireAll>
			</Location>
	#========================================================================================		
	基本语法：
		
		<scheme>://[<user>[:<password>]@]<host>:<port>/<path>[;<params>][?<query>]#<frag>
			params:参数
				http://www.a.com/bbs/hello;gender=f
			query:查询条件
				http://www.a.com/bbs/item.php?username=tom&title=abc
			frag:页面位置锚定

	method:请求方法
		GET; HEAD; POST; PUT(DVA); DELETE; TRACE; OPTIONS

	status(状态码）：
		1xx: 100-101，信息提示；
		2xx: 200-206，成功；
		3xx: 300-305，重定向；
		4xx: 400-415，错误类信息，客户端错误；
		5xx: 500-505，错误类信息，服务端错误；

		常用的状态码：
			200：成功，请求的所有数据通过响应报文的entity-body部分发送：OK
			301：请求的URL指向的资源已经被删除：但在响应报文中通过首部Location指明了资源现在所处的新位置：Moved Permanently
			302：与301相似，但在响应报文中通过Location指明资源现在所处临时新位置；Found
			304：客户端发出了条件式请求，但服务器上的资源未曾发生改变，则通过响应此响应状态码通知客户端：Not Modified
			401：需要输入账号和密码认证方能访问资源：Unauthorized
			403：请求被禁止：Forbidden
			404：服务器无法找到客户端请求的资源：Not Found
			500：服务器内部错误：Internal Server Error
			502：代理服务器从后端服务器收到了一条伪响应：Bad Gateway


		
	*mod_deflate模块:压缩页面节约带宽
	  
	   适用场景:
		  1)节约带宽,额外消耗CPU;同时,可能有些较老浏览器不支持
		  2)压缩适于压缩的资源,例如文本文件

		SetOutputFilter DEFLATE		//定义一个名为DEFLATE的过滤器

		#mod_deflate configuration
		#Restrict compression to these MIME types
		AddOutputFilterByType DEFLATE text/plain
		AddOutputFilterByType DEFLATE text/html
		AddOutputFilterByType DEFLATE text/xml
		AddOutputFilterByType DEFLATE text/application/xhtml+xml
		AddOutputFilterByType DEFLATE application/xml
		AddOutputFilterByType DEFLATE application/x-javascript
		AddOutputFilterByType DEFLATE text/javascript
		AddOutputFilterByType DEFLATE text/css

		#Level if compression(Highest 9-Lowest 1)
		DeflateCompressionLevel 9			//默认压缩比level 9

		#Netscape 4.x has some problems
		BrowserMatch ^Mozilla/4 gzip-only-text/html		对不支持压缩的浏览器做匹配设置
	