			#========================#		
			#        交换部分         #
			#========================#

ROM----->BIOS
NVRAM--->保存startup-config;启动前最后一次保存的配置文件
RAM----->running-config文件；启动时由startup-config拷贝而来
Flash--->存放IOS

破解交换机密码：
	1）重启交换机，按住Mode键3~10秒 进入switch:模式
	2）switch:flash_init		初始化flash
	3) switch:dir flash:	查看flash下的文件
	4）switch:delete flash:config.text	删除/重命名配置文件
	5）switch:boot	重启

SSH配置步骤：
	1）Hostname
	2)domain-name
	3)RSA非对称密钥生成
	4）ssh基本配置

	  e.g:
	   R1(config)#hostname cisco
	   R1(config)#ip domain-name test.com
	   R1(config)#username ccie paaaword ccie
	   R1(config)#crypto key generate rsa modulus 1024
		R1(config)#show crypto key mypubkey rsa
	   R1(config)#line vty 0 4
	   R1(config-line)#login local
	   R1(config-line)#transport input ssh
	   R1(config-line)#exec timeout 1 0	  

无线：
	802.11b--802.11a--802.11g--802.11n--802.11ac--802.11ad

防火墙（ASA）:


Port-Security
	1.有效阻止MAC Flood和MAC Spoof攻击
	2.port security的默认行为
	  a.默认所有的接口port security功能是disable的
	  b.默认每个接口最大的MAC地址容量为一个
	  c.默认violation是shutdown
	3.三种violation方式
	  a.shutdown：使接口处于errordisable状态，并且告警
	  b.restrict：丢弃并告警
	  c.protect：丢弃不产生告警
	4.三种地址学习方式
	  a.自动学习（默认）
	  b.手动指派:switchport port-security mac-address xxxx.xxxx.xxxx
	  c.sticky:switchport port-security mac-address sticky xxxx.xxxx.xxxx
		自动绑定从此端口学习到的第一个MAC地址

#ACL(PACL,VACL,RACL)
  acl动作:permit	 放行、抓取/匹配
		  deny	 拒绝、不抓取/不匹配
	出站调用ACL时只能过滤穿越流量
 
 * 华为（自顶向下匹配。可开启深度匹配:有同类规则时，匹配最精确的语句）
	基本ACL	2000-2999	源IP地址等
	高级ACL	3000-3999	源/目IP地址、源/目端口等		
	二层ACL	4000-4999	源/目MAC地址、以太帧协议类型等
	
  eg: acl 2000
	  rule 5 deny source 192.168.1.0 0.0.0.255
	  rule 10 deny source 172.16.0.0 0.0.0.255
		*结尾默认放行所有
	  
 * 思科：（基于序列号 自顶向下的匹配）
	标准ACL	1-99,1300-1999
	扩展ACL	100-199,2000-2699
	命名ACL	Name(Standard and Extended)

  eg: access-list 10 permit 192.168.1.0 0.0.0.255  #默认seq10
	  access-list 10 deny 192.168.1.0 0.0.0.255	   #seq20
	  access-list 10 deny 0.0.0.0 255.255.255.255  #结尾默认deny any	

 vlan_ACL  
  eg:access-list 1 permit 1.1.1.1  
	vlan access-map VLAN2
	  match ip address 1
	  action drop
 	vlan filter VLAN2 vlan-list 20
 MAC_ACL:
  eg:mac access-list extended mac-vacl deny host 0001.0001.0001 any
    vlan access-map VLAN2
	  match mac address 1
	  action drop
	vlan filter VLAN2 vlan-list 20

	限制用户远程登陆 
	 user-interface=line vty/line console (acl只能在入接口调用）
	流量分类：
	 ·ACL+IPsec—>Crypto ACL(兴趣流量ACL)
	 ·重分发 路由过滤
	 ·流量分类

#IPv6:
	128位：用32位十六数表示（一位十六进制数=4bit）
	XXXX:XXXX:XXXX:XXXX:XXXX:XXXX:XXXX:XXXX/64  前缀（前64位)+接口ID
		每段地址前部分的0可以省略；任意多个全0的段可以用：：表示

	eg:	  ipv4	    	ipv6
		 0.0.0.0 		 ::
		 127.0.0.1		 ::1
		 224.0.0.5		 FF02::5

	Unicast 单播地址		Unicast
	Nulticast组播地址	Multicast/被请求节点组播
	Broadcast广播地址	Anycast

  AGUA：全球可聚合单播地址
  Link-local：链路本地地址  -->以FE80::开头，结合mac地址自动生成；用于本地广播域内通信
   主机可通过ICMPv6所包涵的NDP协议的子协议RA-RS获取地址

网关冗余协议:
	VRRP:(工业标准)
	HSRP:(思科私有)
	GLBP:(思科私有)
	  特点:不但能做冗余还能自动负载(一个AVG,多个AVF负载转发)


#广域网协议：
	HDLC&PPP
	
	 ppp协议认证（cisco设备ppp协议认证：全局用户名密码优于接口配置的认证）
	 R1(config)#username R2 password cisco
	 R1(config)#int s0/0
	 R1(config-if)#encapsulation ppp
	 R1(config-if)#ppp authentication pap
	 |
	 R2(config)#username R2 password cisco
	 R2(config)#int s0/0
	 R2(config-if)#encapsulation ppp
   *>R2(config-if)#ppp pap sent-username R2 password cisco
	>R2(config-if)#ppp chap hostname admin
	>R2(config-if)#ppp chap password 123123


 帧中继&PPPoE
	
   PPPoE服务端配置：	
	 R1(config)#hostname pppoe-server
	 pppoe-server(config)#vpdn enable	
	 pppoe-server(config)#username somebody password cisco	 (用于给对方拔号认证)
	>pppoe-server(config)#ip dhcp pool CISCO
	 pppoe-server(dhcp-config)#network 192.168.1.0 255.255.255.0
	 pppoe-server(dhcp-config)#default-router 192.168.1.1
	 pppoe-server(dhcp-config)#dns-server 8.8.8.8
	 pppoe-server(dhcp-config)#domain-name cisco.com
	>pppoe-server(config)#interface virtual-template 1  (设置给对方拔号的模板)
	 pppoe-server(config-if)#encapsulation ppp
	 pppoe-server(config-if)#ppp authentication chap
	 pppoe-server(config-if)#peer default ip address dhcp-pool CISCO
	 pppoe-server(config-if)#ip mtu 1492
	 pppoe-server(config-if)#ip address 192.168.1.1 255.255.255.0
	 pppoe-server(config-if)#no shutdown
	 pppoe-server(config-if)#exit
	>pppoe-server(config)#bba-group pppoe global  (一般运营商使用独立组关联各种template)
	 pppoe-server(config-bba-group)#virtual-template 1
	 pppoe-server(config-bba-group)#exit
	>pppoe-server(config)#interface f0/0
	 pppoe-server(config-if)#pppoe enable group global （调用global组）
	 pppoe-server(config-if)#end
  PPPoE客户端配置：
	R2(config)#hostname pppoe_client
	pppoe_client(config)#vpdn enable
	pppoe_client(config)#interface dialer 1
	pppoe_client(config-if)#dialer pool 100
	pppoe_client(config-if)#encapsulation ppp 
	pppoe_client(config-if)#ppp chap hostname somebody
	pppoe_client(config-if)#ppp chap password cisco
	pppoe_client(config-if)#ip address negotiated   （自动协商ip地址）
	pppoe_client(config-if)#ppp ipcp route default  （自动指对方为网关：当对方没分配网关时）
	pppoe_client(config-if)#ip mtu 1492
	pppoe_client(config-if)#no shutdown
	pppoe_client(config-if)#exit
	>pppoe_client(config)#interface f0/0
	pppoe_client(config-if)#no shutdown
	pppoe_client(config-if)#pppoe-client dial-pool-number 100
	pppoe_client(config-if)#pppoe-client dial-pool-number 100
	pppoe_client(config-if)#end
	

							#################
							#    路由部分	#
							#################

#OSPF:
	OSPF建邻居条件：
		1,相同的hello&died时间  2，直连接口属于相同的区域  3，相同的认证类型	 
		4，相同的末节区域标识	   5，相同的MTU            6,相同的网络类型

	LSA（link-state Adv）类型：
		Type-1： Router LSA	（网段间LSA）
			·传播范围：只能在一个Area内传递，不能穿越ABR
			 通告者：每台属于一个区域的路由器都会基于该区域通告一条1类LSA
			 包含内容：拓扑信息，描述该路由器所有宣告进该区域链路的前缀，掩码，网络类型及度量值
			 Link-ID：通告该LSA的路由器的RID
			 ADV Router：通告该LSA的路由器的RID			
		
		Type-2:	Network LSA	（MA网络的LSA）
			·传播范围：同1类
			 通告者：MA网段中的DR
			 包含内容：纯拓扑信息，包含了该MA网段直连的所有路由器的RID信息，该MA网段的掩码
			 Link-ID：该MA网段DR接口的IP地址
			 ADV Router：该DR的RID.

		Type-3： Summary Network LSA	 （ABR上区域间的LSA）
			·传递范围：除了该区域外的整个OSPF路由选择域
			 通告者：ABR
			 包含内容：一条3类LSA包含一条OSPF域间路由，O IA
			 Link-ID：3类LSA路由的前缀
			 ADV Router：
			 
			

		Tyep-4： Summary-ASB LSA
			·传递范围：除了ASBR所在区域之外的整个路由选择域
			 通告者：和ASBR在同一个区域的ABR路由器
			 包含内容：纯拓扑信息，描述了ASBR所在位置
			 Link-ID：ASBR的RID
			 ADV Router:通告埏ABR的RID.并且该值每跨越一个ABR都会自动改变，同3类LSA.

		Type-5： External LSA （重分发进OSPF的LSA）
			·传递范围：整个OSPF路由选择区域
			 通告者：ASBR
			 包涵内容：纯路由信息，一条OSPF域外路由对应一条5类LSA.
			 Link-ID：域外路由的路由前缀
			 ADV Router:ASBR的RID.该LSA在OSPF域内传递的时候，ADV Router不会发生改变。

		Type-7

		#一台路由器只要可以产生5类LSA，则该路由器就是ASBR。

		NSSA区域，在NSSA区域内可以拥有ASBR,并且生分发进入OSPF的路由是以7类LSA形式存在，该类型的LSA只能存在于NSSA区域内，该区域所有ABR会通过比较RID选举出一个转换器（最大的RID者），该转换器会将内部传递给外部的NSSA LSA转换成5类LSA并且通告给其他区域，所有该区域内的ABR都会过滤从外部进入该区域的4/5类LSA.但是该区域的任何ABR都不会主动向内部下放缺省路由。为了实现内部路由器的外网可达性，需要在该区域ABR上手工下放缺省路由，O N2 0.0.0.0/0 Seed Metric=1.
	
		Totally NSSA区域，基于NSSA区域的概念基础，ABR会主动阻止3/4/5类LSA进入该区域，并且ABR会主动向区域内下放O IA 0.0.0.0/0 Seed Metric=1的缺省路由

	路由加表优先级：	
		O < O IA < O E1/E2=O N1/N2
		域内路由<域间路由<域外路由

	ospf不规则区域解决方案：
		1.双OSPF进程，重分发
		2.通过gre的Tunnel
		3.Virtual-Link
	
	OSPF认证：
		一、链路级认证：
		 1）R3(config-if)#ip ospf authentication-key CISCO
			R3(config-if)#ip ospf authentication
		   启用链路级明文认证
		 2）R4(config-if)#ip ospf message-digest-key 13 md5 CISCO
			R4(config-if)#ip ospf authentication message-digest
		   启用链路级密文认证

		二、区域级认证：
		 1）启用区域级明文认证
			R2(config-if)#ip ospf authentication-key CISCO
			R2(config-router)#area 0 authentication
		
		 2）启用区域级密文认证
			R2(config-if)#ip ospf message-digest-key 12 md5 CISCO
			R2（config-router)#area 0 authentication message-digest
			
			虚电路间建立认证：
		    R3(config-router)#area 2 virtual-link 91.1.1.1 authentication-key CISCO
			R3(config-router)#area 2 virtual-link 91.1.1.1 authentication


策略路由：
	ACL:access-list:(可抓取路由和数据)
	  R2(config)#access-list 100 permit ip host 12.1.1.1 host 30.1.1.1
	  R2(config)#access-list 101 permit ip host 12.1.1.1 host 40.1.1.1
	
	  R2(config)#route-map TEST permit 1           //#ip policy-list test seq 5 deny 11.1.1.0/24
	  R2(config-route-map)#match ip address 100    //#match policy-list test
	  R2(config-route-map)#set ip next-hop 23.1.1.3
	
	  R2(config)#route-map TEST permit 2
	  R2(config-route-map)#match ip address 101
	  R2(config-route-map)#set ip next-hop 24.1.1.4
	
	  R2(config-if)#ip policy route-map TEST
	
	AS-PATH:
	  R4(config)#ip as-path access-list 10 deny ^$
	  R4(config)#ip as-path access-list 10 permit .*
	  R4(config-router)#neighbor 66.1.1.1 filter-list 10 out
	
	prefix-list:(可以精确控制掩码抓取路由/类似ACL)
	  ip prefix-list NAME permit 172.16.0.0/X ge Y le Z
	  R6(config-router)#do show ip prefix-list detail
	
	  R6(config)#ip prefix-list ABC deny 11.1.0.0/22 ge 24 le 24
	  R6(config)#ip prefix-list ABC permit 0.0.0.0/0 le 32
	  R6(config-router)#neighbor 44.1.1.1 prefix-list ABC in
	
	（两台相互建立邻居的路由器同时开启orf功能，则可以以协商的方式控制对方不发送哪些路由）
	 R4(config-router)#neighbor 66.1.1.1 capability orf prefix-list both
	 R6(config-router)#neighbor 44.1.1.1 capability orf prefix-list both
	 R6(config-router)#do show ip bgp neighbor 44.1.1.1 advertised-routes

	ip policy
	 ip policy route-map PBR
	

BGP:端口号：tcp/179
	1.BGP协议本质？
	2.BGP更新机制？
		GBP和eigrp只有触发更新
	3.BGP三张表:
		邻居表:show ip bgp summary
		转发表:show ip bgp
		路由表:show ip route
	4.BGP几种报文的理解?
		open
		keepalive
		update
		notification
	5.GBP建立邻居的正常状态?不正常状态?

 BGP选路:
	13条选路原则

 BGP联邦和RR
		
 BGP路由过滤:
	1.As-path list
	2.Prefix-list:类似ACL
	3.ORF:与对等体协商不发送某些路由
		neighbor x.x.x.x capability orf prefix-list both  #通告对端不发送prefix-list列表里的路由
	4.Policy list:(一种工具;类似route map)

 BGP优化:
	1.Policy accounting
	2.BGP收敛时间优化
	3.Graceful-restart
	4.Max-prefix
	5.Peer group
	6.BGP-dampening



QOS：
  三种模型：
	 1.尽力而为的：即不做流量区分的转发
	 2.资源预留式（RSVP协议）：弃用
	*3.分级管理

   QOS四组件：
	分类和标记----拥塞管理----拥塞避免----队列--（压缩）
   
   QOS配置方式：
	 1) Legacy CLI
	*2) MQC(module_Qos_CLI)
	 3) Cisco AutoQoS:内嵌的一键配置方法
	 4) Cisco SDM QoS wizard：图形化配置
   
   QOS分类方法：
	1)ACL
	2)NBAR
	3)属性
   QOS标记方法：
	1）PBR（默认优于路由表）
		ip local policy route-map xxx
		  对设备本身产生的流量应用PBR
			--必须应用到数据的in方向
	2）QOS pre-classify：预分类(vpn)
	3) QPPB:(根据BGP的属性分类)
	4) CBMarking

  QOS队列的分类：
   一、QOS队列的分类--WFQ：
      WFQ(B<=E1的默认队列）
     分类（区分过来的流量）：
		基于Flow(FBWFQ)256+8（控制层）
	 加队：
		宏观自动调控，RED丢弃
	 调度：Finish Time=包大小/(IPP+1)
	 Flow 6元组：S/D/P/TOS/SP/DP
   二、QOS队列的分类--CBWFQ：
	 分类（区分过来的流量)：
	  	基于类（CBWFQ)64+1default-class
	 加队：
		宏观自动调控，RED丢弃
	 调度：为每个类保证

	#CBWFQ和WFQ一样，只能应用在出方向；
	#可以在出接口带宽拥塞的情况保证最小带宽；
	#出接口带宽有多余的情况，各个定义的类按照最小保证带宽比例
	#来占用多余的带宽
	
   三、QOS队列的分类--LLQ/CBLLQ

	
   QOS拥塞避免

   QOS拥塞管理--Token Bucket
			BC:桶的大小
		 	CIR:承诺访问速率(CIR)即向令牌桶中填充令牌的速率
			T=BC/CIR
	流量情形分类：
		单速双色：
	  		 每秒世界数据<CIR 为exceed流量
			 每秒世界数据<=CIR 为confrom流量	
		单速三色
			 每秒用户数据<=CIR 为confrom流量
			 每秒用户数据>CIR+BE 为violate流量
			 CIR<每秒用户数据<=CIR+BE 为exceed流量
		双速三色
			 PIR最高承诺速率，用户数据先根据PIR大小转发，假设用户数据为B
			 B>BE 为violate流量，BE和BC都不减少令牌
			 BE>=B>=BC 为exceed流量，BE减少B个令牌
			 BC>B 为confrom流量，BE和BC同时减少B个流量
	 
	 流量监管和流量整形：Policing and Shaping Overview


  #只有分类做在in方向；mark和Policing可做在in&out方向；其他QOS相关策略只能做在出方向
	QOS补充：COPP
	  COPP可控制自身产生的流量，可做在in和出方向；
	   ACL则不能控制自身产生的流量



###VPN
	·站点到站点：
		1.ATM   2.Frame Relay   3.MPLS VPN   4.IPSec
	·点到站点（远程访问VPN）：
		1.IPSec   2.PPTP   3.L2TP+IPSec   4.SSLVPN


MPLS:运营商级VPN方案


 IPSec框架：企业级VPN方案
	1.加密  DES/3DES/AES...
	2.验证  MD5/SHA-1...
	3.封闭封闭协议  ESP/AH...
	4.模式  Transport/Tunnel
	5.密钥有效期  3600/1800
	6. ...


  *IPsec-VPN

   加密算法
 	 1.对称加密算法
	   DES:(56位)		AES:(流媒体)
	   3DES:(3*56位)		RH4:(无线)
     2.非对称加密算法
	   RSA:(ssh)
	   DH:(IPsec)
	   ECC:	
    散列函数(完整性效验)
	  MD5
	  SHA	

   IPSec两种模式
	Transport Mode：GRE
	  从IPSec VPN的角度考虑,这种模式在需要保护的是两台主机之间;(不加密原始IP报头，在原始IP报头后插入ESP报头)
	Tunnel Mode：常用
	  从IPSec VPN的角度考虑,这种模式在需要保护的是多台主机的两个站点之间；(把原始的IP报头一起加密，重新封闭新的IP报头)

		注：通过通讯点和加密点是否一致判断用哪种模式

   控制层面的三种模式二个阶段
	 第一阶段：主模式(6个包)、主动模式(3个包)
		主模式：1-2个包协商加密策略、3-4个包交换加密密钥、双方对身份进行验证
	 第二阶段：快速模式
			   3个包协商加密策略