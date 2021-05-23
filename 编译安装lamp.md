编译安装lnmp:

　一、NGINX
	直接编译安装即可
	

　二、MYSQL（通用二进制格式）
	tar xf mysql-5.5.62-linux-glibc2.12-x86_64.tar.gz -C /usr/local/mysql
	cd /usr/local/mysql
	chown -R root.mysql ./*    ##
	cp support-file/my-large.cnf /etc/my.cnf
	cp support-file/mysql.server /etc/rc.d/init.d/mysqld
	/scripts/mysql_install_db --user=mysql --datadir=/mydata/data
	chkocnfig --add mysqld
	
  三、编译安装php
	配置好yum源（系统安装源及epel源）后执行如下命令：
	# yum -y groupinstall "Desktop Platform Development"
	#yum -y install bzip2-devel libmcrypt-devel openssl-devel
 	tar xf php-7.1.29.tar
	cd php-7.1.29
	./configure --prefix=/usr/local/php --with-mysql=/usr/local/mysql --with-openssl --with-mysqli=/usr/local/mysql-bin/mysql_config --enable-mbstring --with-freetype-dir --with-jpeg-dir --with-png-dir --with-zlib --with-libxml-dir=/usr --enable-xml --enable-sockets --with-apxs2=/usr/local/apache/bin/apxs --with-mcrypt --with-config-file-path=/etc --with-oxnfig-file-scan-dir=/etc/php.d --with-bz2 --enable-maintainer-zts
	说明：
	# --with-apxs2    		    ##把php编译成httpd的模块，以httpd模块方式运行
	# --enable-maintainer-zts	##为了支持apache的worker或event这两个MPM，编译时使用了--enable-maintainer-zts选项。
	***如果是以fpm方式与httpd服务器交互，则此两则改为--enable-fpm
	以上版本，为了链接MYSQL数据库，可以指定mysqlnd,这样在本机就不需要先安装mysql或mysql开发包了。mysqlnd从php5.3开始可用，可以编译时绑定到它。但是从php5.4开始它就是默认设置了。
	# ./configure --with-mysql=mysqlnd --with-pdo-mysql=mysqlnd --with-mysqli=mysqlnd

	# make
	## make test
	# make install
	为php提供配置文件：
	# cp php.ini-production /etc/php.ini

	编辑apache配置文件httpd.conf,让apache支持php
	vim /etc/httpd/conf/httpd.conf
	添加如下二行:
	 AddType application/x-httpd-php .php
	 AddType application/x-httpd-php-source .phps
	定位至DirectoryIndex index.html
	 修改为：
	 DirectoryIndex index.php index.html

	重新启动httpd,或让其重新载入配置文件测试php是否可用

	测试页面index.php示例如下：
		<?php
			$link =mysql_connect('127.0.0.1','root','xxxx');
			if ($link)
			  echo "Success...";
			else
			  echo "Faliure...";

			mysql_close();
		?>
	
	***配置apach-2.4以fpm方式的php-5.4.
	
	一、apache/MySQL的安装与前一部分相同

	二、编译安装php
	如果想让编译的php支持mcrypt扩展，此处还需要安装：
	libmcrypt libmcrypt-devel mhash mhash-devel

	./configure --prefix=/usr/local/php5 --with-mysql=/usr/local/mysql --with-openssl --with-mysqli=/usr/local/mysql/bin/mysql_config --enable-mbstring --with-freetype-dir --with-jpeg-dir --with-png-dir --with-zlib --with-libxml-dir=/usr --enable-xml --enable-sockets --with-mcrypt --with-config-file-path=/etc --with-config-file-scan-dir=/etc/php.d --with-bz2 --enable-fpm
		
	为php提供配置文件：(在编译目录中)
	# cp php.ini-production /etc/php.ini
	
	配置php-fpm
	为php-fpm提供SysV init脚本，并将其添加至服务列表：
	# cp sapi/fpm/init.d.php-fpm /etc/rc.d/init.d/php-fpm
	# chmod +x /etc/rc.d/init.d/php-fpm
	# chkconfig --add php-fpm
	# chkconfig php-fpm on
 	为php-fpm提供配置文件：
	cd /usr/local/php
	cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
	# vim /usr/local/php/etc/php-fpm.conf
	  pm.max_children = 50
	  pm.start_servers = 5
	  pm.min_spare_servers = 2
	  pm.max_spare_servers = 8
	  pid = /usr/local/php/var/run/php-fpm.pid

	# service php-fpm start
 	# ps aux | grep php-fpm
	
	*在apache httpd2.4以后已经有一个模块针对FastCGI的实现，此模块为mod_proxy_fcgi.so,它其实是作为mod_proxy.so模块的扩充，因此，这两个模块都要加载
	·LoadModule proxy_module modules/mod_proxy.so
	·LoadModule proxy_fcgi_module modules/mod_proxy_fcgi.so

	2、配置虚拟主机支持使用fcgi
	在相应的虚拟主机中添加类似如下两行
	  ProxyRequests OFF
	  ProxyPassMatch ^/(.*\.php)$ fcgi://127.0.0.1:9000/PATH/TO/DOCUMENT_ROOT/$1

	http://www.xxxx.com/admin/index.php

	/web/host1/admin/index.php
	fcgi://127.0.0.1:9000/web/hosts/admin/index.php
	