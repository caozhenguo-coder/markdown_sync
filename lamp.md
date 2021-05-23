###LAMP###
  **amp两种结合方式：
	 Centos 7:
	   #一:Modules:httpd,php,php-mysql,mariadb-server	#php做为httpd模块方式运行
		    client-->httpd+php_module-->php-mysql-->mysqld
 	   #二:FastCGI:httpd,php-fpm,php-mysql,mariadb-server	#各服务独立的运行方式
			client-->httpd-->fastcgi_protocol-->fpm(php application server)-->php-mysql-->mysqld
	 CentOS 6:
	   *一：Modules:程序包，httpd,php,php-mysql,mariadb-server	#php做为httpd模块方式运行

========================================================================================================

  一：yum install httpd mariadb-server php php-mysql		##测试动态站点wordpress|discuz/phpwind|phpMyAdmin

  二：
   MYSQL:
	yum install mariadb-server

   HTTPD:
	yum install httpd 
	 
	[root@bogon ~]# vim /etc/httpd/conf.d/vhost.conf 
	  NameVirtualHost IP:PORT  ##HTTP2.2基于FQDN的虚拟主机需要加此项##
	  	#也可在虚拟主机内部定义，则只对该虚拟主机生效
	  Listen 80 
	<VirtualHost *:80>
          DocumentRoot "/data/www/html"		#(下载解压phpMyAdmin至该目录下即可）
          ServerName www.caomou.com
          ProxyRequests Off			#做反向代理服务器时要显式声明关闭正向代理
          ProxypassMatch  ^/(.*\.php)$    fcgi://127.0.0.1:9000/data/www/html/$1	#定义将动态页面代理至php-fpm
          ProxyPassMatch  ^/(ping|pmstatus)$ fcgi://127.0.0.1:9000/$1	#定义php-fpm状态页
          <Directory "/data/www/html">
                Options FollowSymLinks|None
                AllowOverride None
                Require all granted/denied
          </Directory>
		  CustomLog	"logs/access_log" combined
	  </VirtualHost>
	
	[1]配置httpd,添加/etc/httpd/conf.d/fcgi.conf配置文件，内容类似:
			DirectoryIndex  index.php
         	ProxyRequests Off
          	ProxypassMatch  ^/(.*\.php)$    fcgi://127.0.0.1:9000/data/www/html/$1
		
  PHP-FPM:(FastCGI Process Manager)
	CentOS6:
		php-5.3.2：默认不支持fpm机制；需要自行打补丁并编译安装；
		httpd-2.2：默认不支持fcgi协议，需要自行编译此模块；
		解决方案：编译安装httpd-2.4，php-5.3.3+；

	CentOS7:
		httpd-2.4：rpm包默认编译支持了fcgi模块；
		php-fpm包：专用于将php运行于fpm模式；

	#配置文件：
		*服务配置文件(php-fpm)：/etc/php-fpm.conf, /etc/php-fpm.d/*.conf
		*php环境配置文件(php-common)：/etc/php.ini, /etc/php.d/*.ini           ##配置指令严格区分大小写
			·php.ini的核心配置选项文档：http://php.net/manual/zh/ini.core.php
			 	php.ini配置选项列表：http://php.net/manual/zh/ini.list.php

	yum install php-fpm php-mysql php-mbstring php-mcrypt [php-xcache]
	
	[root@localhost ~]# vi /etc/php-fpm.d/www.conf

	创建session目录，并确保运行php-fpm进程的用户对此目录有读写权限；
	(php-fpm的/etc/php-fpm.d/www.conf配置文件中定义了http-session会话存放目录)	
		# mkdir /var/lib/php/session
		# chown apache.apache /var/lib/php/session

	#php连接mysql测试：
	#	<?php
	#		$conn = mysql_connect('127.0.0.1','root','');
	#		if ($conn)
	#			echo “OK";
	#		else
	#			echo “Failure";
	#	?>


###LNMP###
   NGINX:
	配置nginx向后端代理：
		server {
				listen 80;
				server_name www.linux.com;
				index index.php index.html;

				location / {
						root /data/nginx/html;
						[proxy_pass http://10.10.10.30:80;]	#动静分离，把静态内容代理至后端
				}
		
				location ~* \.php$ {
						fastcgi_pass 10.10.10.20:9000;
						fastcgi_index index.php;
						fastcgi_param  SCRIPT_FILENAME	/data/apps$fastcgi_script_name;
						include fastcgi_params;
						#向后端传递/etc/nginx/fastcgi_params中定义的参数；
				}
			}


####upstream && proxy_pass && fastcgi_pass的区别是？？？？？？
#ngx_http_proxy_模块允许将请求传递到另一个服务器。	#作为http客户端;只能反代至后端http_server主机	
#ngx_http_fastcgi_模块允许将请求传递到fastcgi服务器		#作为fastcgi客户端，反代fastcgi请求
#ngx_http_upstream_模块用于定义可由proxy_pass、fastcgi_pass、uwsgi_pass、scgi_pass、memcached_pass和grpc_pass指令引用的服务器    					组。 	#前端做负载均衡直接转发报文