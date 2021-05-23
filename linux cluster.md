Cluster：LB, HA, HP, hadoop
		LB:
			四层：lvs、nginx(upstream)、haproxy(mode:tcp)
			应用层：nginx(http), haproxy(mode:http), httpd(balance_mod), perlbal, ats, varnish
		HA:
			vrrp：keepalived
			AIS：heartbeat, OpenAIS, corosync/pacemaker, cman/rgmanager(conga)=RHCS


  Linux HA Cluster

	*Messaging Layer:
		heartbeat
			v1
			v2	  //独立出CRM:pacemaker
			v3
		corosync
			v1
			v2
		cman

	*Cluster Resource Manager(CRM):
		heartbeat v1: haresources (配置接口：配置文件haresources)
		heartbeat v2: crm (在每个节点运行一个crmd(5560/tcp)守护进程，在命令行接口crmsh ; GUI:hb_gui)
		heartbeat v3: pacemaker (配置接口：crmsh; pcs; GUI:hawk(suse),LCMC, pacemaker-gui)
		rgmanager (配置接口：cluster.conf, system-config-cluster, conga(webgui), cman_tool, clustat)

		组合方式：
			heartbeat v1 (haresources)
			heartbeat v2 (crm)
			heartbeat v3 + pacemaker
			corosync + pacemaker
				corosync v1 + pacemaker (plugin)
				corosync v2 + pacemaker (standalone service)
			cman + rgmanager
			corosync v1 + cman + pacemaker
			keepalived

		  RHCS：Red Hat Cluster Suite
			RHEL5: cman + rgmanager + conga (ricci/luci)
			RHEL6: cman + rgmanager + conga (ricci/luci)
				   corosync + pacemaker
				   corosync + cman + pacemaker
			RHEL7: corosync + pacemaker

	*Local Resource Manager(LRM)

	  Resource Agent:
		service: /etc/ha.d/haresources.d/目录下的脚本
		LSB： /etc/rc.d/init.d/目录下的脚本			//LSB:Linux Standards Base
		OCF： Open Cluster Framework
			provider:
		STONITH：
		Systemd:

	 资源类型：
		primitive：优先级
		target-role：started, stopped, master
		is-managed：是否允许集群管理此资源
		resource-stickiness：资源粘性
		allow-migrate：是否允许迁移

	 约束：score
		位置约束：资源对节点的倾向性
			（-∞, +∞)
		排列约束：资源彼此间是否能运行于同一节点的倾向性
			（-∞, +∞)
		顺序约束：多个资源启动顺序依赖关系
			（-∞, +∞)	

	
	安装配置：
		Centos7： corosync v2 + pacemaker
			corosync v2: vote system
			pacemaker: 独立服务

		集群的全生命周期管理工具：	//centos6上只支持集群配置不支持集群的安装
			pcs：agent(pcsd)
			crmsh：agentless(依赖于pssh)	//依赖安装：crmsh.rpm; python-pssh.rpm; pssh.rpm

		配置集群的前提：
			1）时间同步
			2）基于当前正在使用的主机名互相访问
			3）是否会用到仲裁设备
		

















 https://www.haproxy.com/documentation
   HAProxy是一种免费，非常快速且可靠的解决方案，可为基于TCP和HTTP的应用程序提供 高可用性， 负载平衡和代理。它特别适合于流量非常高的网站，并为世界上许多访问量最大的网站提供支持。多年来，它已成为事实上的标准开源负载平衡器，现已随大多数主流Linux发行版一起提供，并且通常默认情况下部署在云平台中



Haproxy:
	TCP/HTTP Load Balancer  #(tcp和http的负载均衡器)

yum install haproxy
vim /etc/haproxy/haproxy.cfg
#---------------------------------------------------------------------
# main frontend which proxys to the backends
#---------------------------------------------------------------------
frontend  main *:80  	##定义前端
    default_backend             websrvs

#---------------------------------------------------------------------
# static backend for serving up images, stylesheets and such
#---------------------------------------------------------------------
backend websrvs			##定义后端
    balance     roundrobin
    server      node1 192.168.137.10:80 check
    server      node2 192.168.137.20:80 check

#---------------------------------------------------------------------
# round robin balancing between the various backends
#---------------------------------------------------------------------
listen statistics		##定义haproxy的状态页
        bind *:9000
        stats enable
        stats hide-version
        stats scope .
        stats uri /haproxyadmin?stats
        stats realm "HAProxy Statistics"
        stats auth admin:adminn
#		stats admin if TRUE

--------------------------------
/***其它配置参数请参考官方文档***//
----------------------------------------------------------------------------------------

###定义haproxy的日志:
vi /etc/rsyslog.conf

  # Provides UDP syslog reception  #启用rsyslog的UDP/514端口接收日志
  $ModLoad imudp
  $UDPServerRun 514

  local2.*                               /var/log/haproxy.log

===================================================================

   nginx做反向代理:(可异步请求)
		proxy-set-header:x-real-ip		//-->做反向代理时接收请求向后端代理时添加的请求报文首部
						 x-forward-for
		add-header:x-via					//<--做反向代理时响应客户端请求时向响应报文首部添加的信息
				  x-cache	