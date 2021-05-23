iptables:


 *limit:
	[root@localhost ~]# iptables -A INPUT -d 192.168.138.131 -p icmp --icmp-type 8 -m limit --limit-burst 5 --limit 30/minute -j ACCEPT        #每分钟30，峰值为5的速率限制

FORWARD实现网络防火墙流量控制：
	装载相关模块：
	  modprobe nf_conntrack_ftp
	[root@localhost ~]# iptables -P FORWARD DROP  
	[root@localhost ~]# iptables -A INPUT -m state --state ESTABLISHED -j ACCPET
	[root@localhost ~]# iptables -A FORWARD -m state --state ESTABLISHED -j ACCEPT
	[root@localhost ~]# iptabes -A FORWARD -d 192.168.20.2 -p tcp -m multiport --dports 22,80 -m state --state NEW -j ACCEPT

===========================================================================================================

													# IPVS----2016 马哥Linux运维就业班 （第二弹）

LVS:Linux Virtual Server

  lvs集群的类型：
	lvs-nat:多目标IP的DNAT，通过将请求报文中的目标地址和目标端口修改为某个挑出的RS的RIP和PORT实现转发
		1)RIP和DIP必须在同一个网络，且应该使用私网地址；RS的网关要指向DIP
		2)请求报文和响应报文都必须经由Director转发；Director易于成为系统瓶颈
		3)支持端口映射，可修改请求报文的目标PORT
		4）vs必须是Linux系统，rs可以是任意系统
	lvs-dr:操纵封装新的MAC地址
		1)确保前端路由器将目标IP为VIP的请求报文发往Director
			a)在前端网关做静态mac地址绑定
			b)在RS上使用arptables
			c)在RS上修改内核参数以限制arp通告及应答级别
				arp_announce
				arp_ignore
		2)RS的DIP可以使用私网地址，也可以是公网地址；RIP与DIP的网关不能指向DIP，以确保响应报文不会经由Director
		3)RS跟Director要在同一个物理网络
		4)请求铁板鱿鱼要经由Director,但响应不能经由Director,而是由RS直接发往Client
		5)不支持端口映射
	lvs-tun:在原请求IP报文之外新加一个IP首部
		1)VIP,DIP,RIP都应该是公网地址
		2)rs的网关不能也不可能指向DIP
		3)请求报文要经由Director,但响应不能经由Director
		4)不支持端口映射
		5)RS的OS得支持隧道功能
	lvs-fullnat:修改请求报文的源和目标IP,响应时利用连接追踪功能。(非标准类型)

  IPVS scheduler:(要考虑长短连接问题)
	根据其调度时是否考虑各RS当前的负载状态，可分为静态方法和动态方法两种:
	 	静态方法：仅根据算法本身进行调度：
			RR：roundrobin，轮询；
			WRR：Weigthed RR，加权轮询；
			SH：Source Hashing，实现session stick,源IP地址hash;将来自于同一个IP地址的请求始终发往第一次挑中的RS，从而实现会话绑定；
			DH：Destination Hashing；将发往同一个目标地址的请求始终转发至第一次挑中的RS，典型使用场景是正向代理缓存场景中的负载均衡；
		动态方法：主要根据每RS当前的负载状态及调度算法进行调度
			LC：least connections
				Overhead=activeconns*256+inactiveconns
			WLC：Weighted LC
				Overhead=(activeconns*256+inactiveconns)/weight
			SED：Shortest Expection Delay
				Overhead=(activeconns+1)256/weitht
			NQ：Never Queue

			LBLC：Locality-Based LC，动态的DH算法
			LBLCR：LBLC with Replication，带复制功能的LBLC


  lvs管理工具:ipvsadm/ipvs
	*管理集群服务	
		ipvsadm -A|E -t|u|f service_address [-s scheduler]
		ipvsadm -D -t|u|f service_address
	
		service_address:
			tcp: -t ip:port
			udp: -u ip:port
			fwm: -f mark
	
		-s scheculer:
			默认为wlc

	*管理集群服务中的RS
		ipvsadm -a|e -t|u|f service_address -r server_address [-g|i|m] [-w weight]
		ipvsadm -d -t|u|f service_address -r server_address

		server_address:
			ip[:port]

		lvs-type:
			-g: gateway, dr
			-i: ipip, tun
			-m: masquerade, nat

		清空和查看规则：
			ipvsadm -C
			ipvsadm -L|l [options]

		保存和重载规则：
			ipvsadm -R | ipvsadm-restore < /etc/sysconfig/ipvsadm
			ipvsadm -S [n] | ipvsadm-save > /etc/sysconfig/ipvsadm 


		lvs-nat:

		lvs-dr：
			两个内核参数：
				arp_ignore：设置是否响应
					 0.(default)响应所有
					*1.只响应所请求于本网络的
				arp_announce：设置是否通告
					 0.(default)通告物理接口上的所有地址
					 1.尽量避免通告接口上非本网络的地址
					*2.只通告接口上属于本网络的地址

	[root@director ~]# ifconfig ens32:0 192.168.43.43/32 broadcast 192.168.43.43 up	#配置vip为dip的别名接口
	[root@director ~]# route add -host 192.168.43.0 dev ens32:0	#报文送出时必须经由此接口（ens32:0）

	[root@real_serv ~]# echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore 
	[root@real_serv ~]# echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
	||
	[root@real_serv ~]# ifconfig lo:0 192.168.43.43/32 broadcast 192.168.43.43 up
	[root@real_serv ~]# route add -host 192.168.43.0 dev lo:0
	
	[root@director ~]# ipvsadm -A -t 192.168.43.222:80 -s rr
	[root@director ~]# ipvsadm -a -t 192.168.43.222:80 -r 192.168.43.200 -g
	[root@director ~]# ipvsadm -a -t 192.168.43.222:80 -r 192.168.43.201 -g

  WORK:
	dr模型实现http和https两种负载衡集群
		ssl:
			RS：都要提供同一个私钥和同一个证书
	dr模型实现mysql负载均集群

	拓展：规划拓扑实现，VIP与RIP不在同一个网络中的集群；