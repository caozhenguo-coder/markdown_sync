PXE:
	安装程序：anaconda
	
		pxelinux.0->{boot.cat+isolinux.bin}-->{vmlinuz+initrd.img}
	  类似stag1功能   引导加载光盘上的微系统     微系统(用于运行系统安装程序)

	CenOS的安装过程启动流程：
		MBR:boot.cat
		Stage2:isolinux/isolinux.bin
			配置文件：isolinux/isolinux.cfg

	·CentOS 6 PXE:
		yum -y install syslinux tftp-server
		
		cp /usr/share/syslinux/pxelinux.0 /var/lib/tftpboot/
		cp /media/cdrom/images/pxeboot/{vmlinuz,initrd.img} /var/lib/tftpboot/
		cp /media/cdrom/isolinux/{boot.msg,vesamenu.c32,splash.png} /var/lib/tftpboot/
		mkdir /var/lib/tftpboot/pxelinux.cfg
		cp /media/cdrom/isolinux/isolinux.cfg /var/lib/tftpboot/pxelinux.cfg/default

	·CentOS 7 PXE:
		yum -y install syslinux tftp-server

		cp /usr/share/syslinux/pxelinux.0 /var/lib/tftpboot/
		cp /media/cdrom/images/pxeboot/{vmlinuz,initrd.img} /var/lib/tftpboot
		cp /usr/share/syslinux/{chain.c32,mboot.c32,menu.c32,memdisk} /var/lib/tftpboot
		mkdir /var/lib/tftpboot/pxelibux.cfg
		vim /var/lib/tftpboot/pxelinux.cfg/default
			 内容类似如下：
			default menu.c32
				prompt 5
				timeout 30
				MENU TITLE CentOS 7 PXE Menu

				LABEL Linux
				MENU LABEL Install CentOS 7 x86_64
				KERNEL vmlinuz
				APPEND initrd=initrd.img inst.repo=http://172.16.1.1/centos7 ks=http://172.16.1.1/ks.cfg



	yum install system-config-kickstart  
		

	生成iso光盘镜像文件：	
		]# mkisofs -R -J -T -v --no-emul-boot --boot-load-size 4 --boot-info-table -V "CentOS 7 x86_64 boot" -c isolinux/boot.cat -b isolinux/isolinux.bin -o /root/boot.iso myboot/




ks.cfg参考配置:图形配置工具system-config-kickstart
	[root@PXE ~]# vi /var/www/html/ks.cfg     # 调整配置文件
	# Kickstart file automatically generated by anaconda.
	
	#version=DEVEL
	install                                           # 命令段  ，安装
	url --url=http://192.168.4.150/centos/os/        # 指定网络url安装目录 
	lang en_US.UTF-8                                 # 默认字体
	keyboard us                                      # 键盘类型
	network --onboot yes --device eth0 --bootproto dhcp  --noipv6  
	# 指定开机自启，网络接口eth0 ，dhcp获取网络地址，ipv6 禁用
	
	rootpw  --iscrypted $6$ZOGP2tA0PI/6SI/X$MlC5bJyXfP9TBN5/0vwoc6dqAqIijOQthEbAZUnIXft85Tj9n4sKWB2PfxrsVfkZ2ibqX63apu8ElmdEvBo9o/   
	# root 加密密码，使用grub-crypt 生成的字符串替代
	
	reboot         # 配置完毕后，重启内核
	firewall --disabled    # 防火墙禁用
	authconfig --enableshadow --passalgo=sha512  # 登录身份使用 sha1 的 512bits 加密算法
	selinux --disabled     # selinux 功能禁用
	timezone Asia/Shanghai # 定义上海时区 
	bootloader --location=mbr --driveorder=sda --append="crashkernel=auto rhgb quiet"   
	# 定义bootloader，grub安装mbr ，安装在sda磁盘
	
	# The following is the partition information you requested
	# Note that any partitions you deleted are not expressed
	# here so unless you clear all partitions first, this is
	# not guaranteed to work
	clearpart --all         # 清除磁盘分区表
	text                     # 纯文本格式安装显示
	zerombr                  # 对磁盘进行初始化
	
	part /boot --fstype=ext4 --asprimary --size=2000  # 分区信息 ，定义boot分区 ，格式为ext4 ，大小为2G
	part swap --size=4096          # 分区信息 ，定义swap分区 ，大小为4G
	part pv.008003  --size=80000  # 分区信息 ，定义lv分区 pv.008003，大小为80G
	
	volgroup vg0 --pesize=8192 pv.008003   # 分区信息 ，在lv分区pv.008003定义vg0卷组 ，pe大小为8M
	logvol / --fstype=ext4 --name=root --vgname=vg0 --size=15000
	logvol /usr --fstype=ext4 --name=/usr --vgname=vg0 --size=30000
	logvol /var --fstype=ext4 --name=/var --vgname=vg0 --size=20000
	logvol /home --fstype=ext4 --name=/home --vgname=vg0 --size=12000
	
	repo --name="CentOS-6.6"  --baseurl=http://192.168.4.150/centos/os/  --cost=100  
	# 定义yum仓库 ，类别为bashurl ，名称为CentOS-6.6
	
	%packages             # 包组段，安装包组及程序包
	@group_name
	package
	-package            
	%end
 脚本段
	%pre
	...
	%end

	%post
	...
	%end