centos7中yum安装zibbix, 在epel仓库存中
~] yum install epel-release

服务器端需安装:zabbix40.x86_64;    //通用组件 
			 zabbix40-server.noarch;   //服务包
			 zabbix40-server-mysql.x86_64    //zabbix连接mysql的程序包);
		     [zabbix-get];           //获取agent端信息的程序包);
			 zabbix40-web.noarch     //web界面程序包
			 zabbix40-web-mysql.noarch //web程序连接mysql的程序包
	         #注:zabbix3.0可能没有zabbix-server包,被揉和在zabbix-server-mysql组件包中。
被监控端需安装:zabbix40.x86_64(通用组件); 
			 zabbix40-agent.x86_64   //zabbix的agent程序;
			 [zabbix-sender]         //agent端主动发送信息的程序)

###由于按照官方和其它方法都安装成功(可能是由于网络原因,所以手动下载安装包来安装)---->

zabbix3.0安装:

	centos7中yum安装zibbix, 在epel仓库存中
	~]# yum install epel-release

  一:安装zabbix-server  #注:zabbix3.0版本没有zabbix-server包,被揉和在zabbix-server-mysql组件包中。
	   从官方http://repo.zabbix.com/zabbix/3.0/rhel/7/x86_64/下载zabbix-server-mysql zabbix-get  
     1)~]# yum install zabbix-server-mysql zabbix-get
     2)~]# gzip -d /usr/share/doc/zabbix-server-mysql-*/create.sql.gz
	 3)安装mariadb-server并导入数据库
		> create database zabbix charset 'utf8';
		> grant all on zabbix.* to zabbix@'localhost' identified by 'zbxpasswd';
		> flush privileges;
		~]# mysql -uroot -p -Dzabbix < /usr/share/doc/zabbix-server-mysql-*/create.sql
	 4) ~]# vi /etc/zabbix/zabbix_server.conf & systemctl start zabbix-server  #若服务无法启动yum info trousers检查包版本
		~]# grep "^####" /etc/zabbix/zabbix_server.conf 
		############ GENERAL PARAMETERS ######通用配置段
		############ ADVANCED PARAMETERS ######高级配置段
		####### LOADABLE MODULES #######
		####### TLS-RELATED PARAMETERS #######
	 

  二:安装zabbix web
	 1)由于zabbix的web依赖于LAMP环境,安装:
	    ~] yum install httpd php php-mysql php-mbstring php-gd php-bcmath php-ldap php-xml
		~] yum install zabbix-web zabbix-web-mysql  #或直接从官网下载安装包安装
	    针对httpd, zabbix-web包中已经包含了对应zabbix文档的配置文件
		~] vi /etc/httpd/conf.d/zabbix.conf
	   		配置虚拟主机和时区
	  2)访问web
		http://HOST/zabbix
	     zabbix登录页面默认管理员用户为admin,密码为zabbix;
		 完成后生成的配置文件:/etc/zabbix/web/zabbix.conf.php
	  3)web面板菜单
		 Monitoring	     #查看监控
		 Inventory		 #资产清单
		 Reports		 #生成监控报告
		 Configuration   #配置监控功能
		 Administration  #zabbix自己的管理功能配置

  三:zabbix agent安装配置:
	  1)安装
		~]# yum install zabbix-agent-3.0.0-1.el7.x86_64.rpm zabbix-sender-3.0.0-1.el7.x86_64.rpm 

	  2)配置
		 ~]# grep "^####" /etc/zabbix/zabbix_agentd.conf 
			############ GENERAL PARAMETERS #################
			##### Passive checks related  被动监控相关的配置
			##### Active checks related   主动监控相关的配置
			############ ADVANCED PARAMETERS #################
			####### USER-DEFINED MONITORED PARAMETERS #######  用户自定义的监控参数,Userparameter
			####### LOADABLE MODULES #######
			####### TLS-RELATED PARAMETERS #######

   配置监控:
	  快速配置一个监控项:
		host groups-->host-->application-->item-->triggers(events)-->action(condtions,operations)
			operations:remote command,alert

			item-->simple graph
			 items-->graph
			  graphs-->screen
			   screens-->slide show