OpenStack:

	IaaS云栈（OpenStack,CloudStack)、PaaS(Docker,Openshift)、SaaS
	

  部署环境：
	·各节点可基于hostname访问
	·各节点时间同步
	·关闭防火墙，NetworkManager等服务
	·database

  安装配置：官方指南文档 https://docs.openstack.org/liberty/zh_CN/install-guide-rdo/

	一、Identity：主要功能有两个
		1）用户管理：认证和授权
			认证方式有两种：
				token
				账号和密码
		2）服务编录：所有可用服务的信息库，包含其API endpoint路径
		   
		   几个核心术语：
			 User, Role, Tanent, Service, Endpoint, Token

	二、computer:Nova		能过虚拟化技术提供资源池
	
	三、Block Storage:Cinder	提供块存储

	四、Image Service     	提供虚拟镜像的注册和存储管理

	五、Telemetry:Ceilometer	提供监控和数据采集、计量服务