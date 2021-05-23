ip link set multicast on dev ens33

Keepalived:
	vrrp协议的软件实现，原生设计的目的为了高可用ipvs服务(高可用ipvs或http)
	

环境：各节点时间同步
	 各节点能互相解析生机名
	 关闭iptables、selinux	
yum install keepalived
 
  vim /etc/keepalived/keepalived.conf
      1 ! Configuration File for keepalived
      2 
	  3 global_defs {				//全局配置段	
      4    notification_email {		//服务器变更邮件通知
      5         root@localhost		//mail to someone
      6    }
      7    notification_email_from keepalived@localhost		//mail from
      8    smtp_server 127.0.0.1		//mail server
      9    smtp_connect_timeout 30		//mail timeout
     10    router_id node1				
     11    vrrp_skip_check_adv_addr
     12    vrrp_strict
     13    vrrp_garp_interval 0			
     14    vrrp_gna_interval 0
     15 }
     16 
     17 vrrp_instance VI_1 {		//vrrp配置段，每个vrrp_instance对应一个虚拟路由器
     18     state MASTER			//默认抢占模式
     19     priority 100			//优先级
     20     interface ens38			
     21     virtual_router_id 51	//虚拟路由ID
     22     advert_int 1			//每隔1秒通告一次
			vrrp_mcast_net 224.0.0.18	//每个instance需要单独的组播地址
     23     authentication {
     24         auth_type PASS		//集群认证方式
     25         auth_pass mypasswd	//认证密码
     26     }
     27     virtual_ipaddress {
		#	   <IPADDR>/<MASK> brd <IPADDR> dev <STRING> scope <SCOPE> label <LABEL>
     28        192.168.43.141/24 dev ens38 label ens38:0
     29     }
     30 }

===========================================

  资源组配置示例：

-------------定义资源组（nat模型下的LVS集群）：		//内外网卡做资源组绑定一起切换
	vrrp_sync_group VG_1 {		
		group {
		 VI_1
		 VI_2
		}
	}

	vrrp_instance VI_1 {
		eth0
		vip
	}
	vrrp_instance VI_2 {
		eth1
		dip
	}

