安全：文件加解密
	非对称密钥： ~]# ssh-keygen
				 ]# gpg --gen-key	

	加密：gpg -c filename
	解密：gpg -o newfile d filename.gpg

	文件校验：md5sum filename
		     sha1sum filename
vimrc:
	set nu
	"激活鼠标的使用"
	set mouse=a
	set selection=exclusive
	set selectmode=mouse,key
	"显示下划线;高亮显示当前行"
	set cursorline
	hi cursorline guibg=#00ff00
	hi CursorColumn guibg=#00ff00
	"显示状态栏(默认值为1,表示无法显示状态栏)"
	set laststatus=2
	"设置背景颜色"
	set background=dark

===================================================================
systemd
	查看服务当前状态激活与否：systemctl is-active NAME.service
	查看所有已激活的服务：systemctl list-units --type service
	查看所有服务（已激活与未激活）：systemctl list-units -t service --all

	禁止某服务设定为开机自启：systemctl mask NAME.service
	取消此禁止：systemctl unmask NAME.service

	管理target units:
		运行级别：
			0==>runlevel0.target, poweroff.target
			1==>runlevel1.target, rescue.target
			2==>runlevel2.target, multi-user.target
			3==>runlevel3.target, multi-user.target
			4==>runlevel4.target, multi-user.target
			5==>runlevel5.target, graphical.target
			6==>runlevel6.target, reboot.target

		级别切换：init N ==> systemctl isolate NAME.target
			查看所有级别：systemctl list-units -t target -a

			获取默认运行级别：systemctl get-default
			修改默认运行级别：systemctl set-default NAME.target

		其它常用命令：
			关机：systemctl halt, systemctl poweroff
			重启：systemctl reboot
			挂起：systemctl suspend
			快照：systemctl hibernate
			快照并挂起：systemctl hybrid-sleep


	service unit file:
	 ·/usr/lib/systemd/system/*
	 ·/run/systemd/system/*
	 ·/etc/systemd/system/*	
	
		文件通常由三部分组成:
			[Unit]:定义与Unit类型无关的通用选项;用于提供unit的描述信息、unit行为及依赖关系等;
			[Service]:与特定类型相关的专用选项;此处为Service类型；
			[Insall]:定义由"systemctl enable"以及"systemctl disable"命令在实现服务启用或禁用时用到的一些选项；

		Unit段的常用选项：
			Description:描述信息；意义性描述；
			After：定义unit的启动次序；表示当前unit应该晚于哪些unit启动；其功能与Before相反；
			Requies:依赖到的其它units；强依赖；
			Wants：依赖到的其它units;弱依赖；
			Confllcts:定义units间的冲突关系；

		Service段的常用选项：
			Type:用于定义影响ExecStart及相关参数的功能的unit进程启动类型；
				类型：
					simple;	forking; oneshot; dbus; notify:类似于simple
			EnvironmentFile：环境配置文件；
			ExecStart：指明启动unit要运行的命令或脚本；ExecStartPre,ExecStartPost
			Execstop：指明停止unit要运行的命令或脚本；
			Restart:

		Install段的常用选项：
			Alias:
			RequiredBy:	被哪些units所依赖
			WantedBy：哪些units所依赖；

		注意：对于新创建的unit文件或修改了的unit文件，要通知systemd重载此配置文件
			#systemctl daemon-reload			
===================================================================================

	route add -net|host 192.168.20.0/24 gw 172.16.100.9
	ip route add 192.168.20.0/24 via 172.16.100.9 dev eth1 [src 10.10.10.10]
ip 命令：
	ip route add|delete|show|flush|get|change|replace
		ip route add TYPE PREFIX via GW [dev IFACE] [src SOURCE_IP]
	添加默认网关
	 ip route add default via 172.16.1.1 dev eth0

   用到非默认网关路由：/etc/sysconfig/network-scripts/route-IFACE
	支持两种配置方式，但不混用；
	  1）每行一个路由条目：
		TARGET via GW
	  2)每三行一个路由条目：
		ADDRESS#TARGET
		NETMASK#=MASK
		GATEWAY#=NEXTHOP



OPENSSH
	交互生成密钥对:
	指定密钥对生成位置与名称：
	  ssh-keygen -f /PATH/TO/id_rsa
    用户家目录下.ssh/authorized_keys文件用于存放远程主机公钥
		

LVM:
	扩展ext文件系统命令:resize2fs /dev/to/lv
	扩展xfs文件系统命令:xfs_growfs /dev/to/lv
   对逻辑卷创建快照:
	]# lvcreate -L 1G [-p r] -s -n testsnapshot_mylv /dev/mapper/myvg-mylv
===============================================

网卡绑定-
  工作模式：多网卡同时工作需对端交换机ether_channel支持
	Mode0(balance-rr):轮转模式(Rond-robin）
	Mode1(active-backup):主备模式
	Mode3(broadcast)：广播模式；所有接口同时工作

   在centos6上实现：
	vi /etc/sysconfig/network-scripts/ifcfg-bond0
		DEVICE=bond0
		BONDING_OPTS="MIIMON=100 MODE=0"
		IPADDR=1.1.1.1
		PREFIX=24
	vi 	/etc/sysconfig/network-scripts/ifcfg-eth0
		DEVICE=eth0
		MASTER=bond0
		SLAVE=yes
		USERCTL=no
	查看bond0状态：/proc/net/bonding/bond0

	网卡名称：
	 ·基于BIOS支持启用biosdevname软件
		内置网卡：em1,em2
		pci卡：pYpX	 Y:slot,X:port
	   名称组成格式
		 en:Ethernet 有线局域网
		 wl:wlan 无线局域网
		 ww:wwan无线广域网
	   名称类型
		 o<index>:集成设备的设备索引号
		 s<slot>:扩展槽的索引号
		 x<MAC>:基于MAC地址的命名
	 	 p<bus>s<slot>:enp2s1
	centos7网卡名回归传统命名方式：
	  1）vi /etc/default/grup
			GRUB_CMDLINE_LINUX="rhgb quiet net.ifnames=0"
	  2）为grub2生成其配置文件
			grub2-mkconfig -o /etc/grub.cfg
	  3)重启系统

	nmcli命令为同一块网卡提供多个配置：													   auto
		nmcli connection add con-name eth1-test ifname eth1 type ethernet ipv4.method manual ipv4.addresses 1.1.1.1/24
		切换：nmcli connection up eth1-test
		网卡自启动]# nmcli c modify ens33 connection.autoconnect yes
			nmcli connection modify IFNAME [+|-]setting.property VALUE
				setting.property:
				ipv4.addresses	ipv4.gateway
				ipv4.dns1	ipv4.method manual|auto
   在centos7上用nmcli实现bonding
	·添加bonding接口
	  nmcli con add type bond con-name mybond0 ifname bond0 mode active-backup
	·添加从属接口
	  nmcli con add type bond-slave ifname ens7 master bond0
	  nmcli con add con-name mybond0-ens3 type bond-slave ifname ens3 master bond0
	·要启动绑定，必须先启动从属接口
	  nmcli con up bond-slave-eth0
	  nmcli con up bond-slave-eth1
	   nmcli con up mybond0

   在centos7上网络组示例：
	1）nmcli con add type team con-name myteam0 ifname team0 config '{ "runner" :{ "name" : "loadbalance" }}'ipv4.addresses 1.1.1.1/24 ipv4.method namual
	2）nmcli con add con-name team0-eth1 type team-slave ifname eth1 master team0
	  nmcli con add con-name team0-eth2 tyep team-slave ifname eth2 master team0
	3)nmcli con up myteam0
	  nmcli con up tean0-eth1
	  nmcli con up team0-eth2
	  teamdctl team0 state
	  nmcli dev dis eth1

==========================================================================================
								流处理

 grep (文本搜索过滤工具)		 	更适合单纯的查找或匹配文本
 sed  (stream editor文本编辑工具)	更适合编辑匹配到的文本
 awk  (文本报告生成器)				更适合格式化文本,对文本进行较复杂的格式化处理
 seq  命令用于产生从某个数到另外一个数之间的所有整数。
	seq [选项]... 尾数
	seq [选项]... 首数 尾数
	seq [选项]... 首数 增量 尾数


  报告生成器AWK:
    语法:awk [option] 'pattern'{action}' file

	  options:
		-f --file: 从文件中读取AWK命令
		-F --field-separator: 定义分隔符
		-v var=val: 定义变量

	内置变量:
		NR:(number-of-record)行号
		NF:(number-of-field)当前行的字段数量
		FS:(field-separator)字段分隔符(默认是任何空格)
		OFS:(out-field-separator)输出字段分隔符(默认是一个空格)
		RS:(record-separator)输入记录分隔符(默认换行符)
		ORS:(out-record-separator)输出换行符,输出时用指定符号代替换行符
		FNR:(file-number-record)各文件分别计数的行号
		FILENAME:当前 文件名
		ARGC:命令行参数 的个数
		ARGV:数组,保存的是命令行所给定的各参数

	if语句语法:  if(condition){statement;...}[else statement]
				if(condition){statement1}else if(condition2){statement2}
					else{statement3}

	while循环:

=========================================================================

DNS:named

	资源记录定义的格式：
	  语法： name [TTL]IN  rr_type  value
		注：
		 1）TTL：从其它DNS查询到的记录在本地缓存的时长；可从全局继承
		 2）IN：internet记录；可从全局继承
		 3）rr_type：(resource recode)资源记录类型包括：A、AAAA、NS、MX、PTR、CNAME、SOA
		 2）@可用于引用当前区域的名字
		 3）同一个名字可以通过多条记录定义多个不同的值；此时DNS服务器会以轮询方式响应
		 4）同一个值也可能有多个不同的定义名字；通过多个不同的名字指向同一个值进行定义；此仅表示通过多个不同的名字可以找到同一个主机

	e.g:
		$TTL 3H		//从其它DNS查询到的记录在本地缓存的时长
		@       IN SOA  @ rname.invalid. (
	                                        0       ; serial	//序列号、版本号，其它通过此判断此数据库有没有更新
	                                        1D      ; refresh	//主备DNS服务器之间同步间隔
	                                        1H      ; retry		//同步失败重试时长
	                                        1W      ; expire	//数据库有效期
	                                        3H )    ; minimum	//否定答案的TTL值
	---------------------------------------------------------------------------------      
	|			$TTL 3H																|	
	|			@       IN SOA  dns1.zeyond.com.  360918310.qq.com. (				|
	|			                                        20200419        ; serial	|
	|			                                        1H      ; refresh			|
	|			                                        10M     ; retry				|
	|			                                        1W      ; expire			|
	|			                                        3H )    ; minimum			|
	|			        NS      dns1												|
	|			        NS      dns2												|
	|			dns1    A       183.56.167.80										|
	|			dns2    A       183.56.167.81										|
	|			webser	A		192.168.2.222										|
	|			www		CNAME	webser		//可以短格式webser或全名webser.zeyond.com.|
	---------------------------------------------------------------------------------

  SWAP：
	dd if=/dev/zero of=/tmp/swap bz=100 count=10
	mkswap /dev/swap
	swapon

	


				