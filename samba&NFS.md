samba

	[root@linuxprobe ~ ]# yum install samba

   表12-1                                      Samba服务程序中的参数以及作用
	[global]		#全局参数。
	 workgroup = MYGROUP	#工作组名称
	 server string = Samba Server Version %v	#服务器介绍信息，参数%v为显示SMB版本号
	 log file = /var/log/samba/log.%m	#定义日志文件的存放位置与名称，参数%m为来访的主机名
	 max log size = 50	#定义日志文件的最大容量为50KB
	 security = user	#安全验证的方式，总共有4种
	  #share：来访主机无需验证口令；比较方便，但安全性很差
	  #user：需验证来访主机提供的口令后才可以访问；提升了安全性
	  #server：使用独立的远程主机验证来访主机提供的口令（集中管理账户）
	  #domain：使用域控制器进行身份验证
	 passdb backend = tdbsam	#定义用户后台的类型，共有3种
	  #smbpasswd：使用smbpasswd命令为系统用户设置Samba服务程序的密码
	  #tdbsam：创建数据库文件并使用pdbedit命令建立Samba服务程序的用户
	  #ldapsam：基于LDAP服务进行账户验证
	 load printers = yes	#设置在Samba服务启动时是否共享打印机设备
	 cups options = raw	#打印机的选项
	[homes]		#共享参数
	 comment = Home Directories	#描述信息
	 browseable = no	#指定共享信息是否在“网上邻居”中可见
	 writable = yes	#定义是否可以执行写入操作，与“read only”相反
	[printers]		#打印机共享参数
	 comment = All Printers	
	 path = /var/spool/samba	#共享文件的实际路径(重要)。
	 browseable = no	
	 guest ok = no	#是否所有人可见，等同于"public"参数。
	 writable = no	
	 printable = yes

	[root@linuxprobe ~]# vi /etc/samba/smb.conf	
	
	 [database]	共享名称为database
	 comment = Do not arbitrarily modify the database file	警告用户不要随意修改数据库
	 path = /home/database	共享目录为/home/database
	 public = no	关闭“所有人可见”
	 writable = yes	允许写入操作

  *创建用于访问共享资源的账户信息。在RHEL 7系统中，Samba服务程序默认使用的是用户口令认证模式（user）。这种认证模式可以确保仅让有密码且受信任的用户访问共享资源，而且验证过程也十分简单。不过，只有建立账户信息数据库之后，才能使用用户口令认证模式。另外，Samba服务程序的数据库要求账户必须在当前系统中已经存在，否则日后创建文件时将导致文件的权限属性混乱不堪，由此引发错误。

  pdbedit命令用于管理SMB服务程序的账户信息数据库，格式为“pdbedit [选项] 账户”。在第一次把账户信息写入到数据库时需要使用-a参数，以后在执行修改密码、删除账户等操作时就不再需要该参数了。 
	pdbedit:
		参数			作用
		-a 用户名	建立Samba用户
		-x 用户名	删除Samba用户
		-L			列出用户列表
		-Lv			列出用户详细信息的列表

12.1.3 Linux挂载共享
  Samba服务程序还可以实现Linux系统之间的文件共享。;客户端需安装支持文件共享服务的软件包（cifs-utils）。

	[root@客户端 ~]# yum install cifs-utils

	在Linux客户端，按照Samba服务的用户名、密码、共享域的顺序将相关信息写入到一个认证文件中。为了保证不被其他人随意看到，最后把这个认证文件的权限修改为仅root管理员才能够读写：
		[root@客户端 ~]# vim auth.smb
		username=linuxprobe
		password=redhat
		domain=MYGROUP
		[root@客户端 ~]# chmod -Rf 600 auth.smb

	现在，在Linux客户端上创建一个用于挂载Samba服务共享资源的目录，并把挂载信息写入到/etc/fstab文件中，以确保共享挂载信息在服务器重启后依然生效：
		[root@客户端 ~]# mkdir /database
		[root@客户端 ~]# vim /etc/fstab
		 //192.168.10.10/database /database cifs credentials=/root/auth.smb 0 0
		[root@客户端 ~]# mount -a


12.2 NFS网络文件系统

	[root@linuxprobe ~]# yum install nfs-utils
	[root@linuxprobe ~]# vim /etc/exports
	 /nfsfile 192.168.10.*(rw,sync,root_squash)
	
	表12-7             用于配置NFS服务程序配置文件的参数
	
	参数				作用
	ro				只读
	rw				读写
	root_squash		当NFS客户端以root管理员访问时，映射为NFS服务器的匿名用户
	no_root_squash	当NFS客户端以root管理员访问时，映射为NFS服务器的root管理员
	all_squash		无论NFS客户端使用什么账户访问，均映射为NFS服务器的匿名用户
	sync			同时将数据写入到内存与硬盘中，保证不丢失数据
	async			优先将数据保存到内存，然后再写入硬盘；这样效率更高，但可能会丢失数据

	第4步:启动和启用NFS服务程序。由于在使用NFS服务进行文件共享之前，需要使用RPC（Remote Procedure Call，远程过程调用）服务将NFS服务器的IP地址和端口号等信息发送给客户端。因此，在启动NFS服务之前，还需要顺带重启并启用rpcbind服务程序，并将这两个服务一并加入开机启动项中
	 [root@linuxprobe ~]# systemctl restart rpcbind
     [root@linuxprobe ~]# systemctl start nfs-serve

  NFS客户端:

	表12-8      showmount命令中可用的参数以及作用
	参数	 作用
	-e	 显示NFS服务器的共享列表
	-a	 显示本机挂载的文件资源的情况NFS资源的情况
	-v	 显示版本号

	[root@客户端 ~]# showmount -e 192.168.10.10
	[root@linuxprobe ~]# mkdir /nfsfile
	[root@linuxprobe ~]# mount -t nfs 192.168.10.10:/nfsfile /nfsfile


12.3 AutoFs自动挂载服务