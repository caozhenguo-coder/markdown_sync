


docker-ce
	·镜像管理：（默认先从本地再从https://hub.docker.com来获取镜像）
	docker inspect NAME	#查看镜像详细信息
	docker tag NAME [REPOSITORY[:TAG]] 给镜像打标签
	docker commit  CONTAINER [REPOSITORY[:TAG]]	#基于已有镜像来制作镜像
	[root@master ~]#docker commit -c 'CMD ["/bin/httpd""-f","-h","/data/html"]' -p Which-IMAGE NEW-NAME	#更改镜像启动时运行的命令
	docker save NAME 镜像打包
	docker load NAME 加载打包镜像	

 自制镜像： docker build 命令
   Dockerfile:在一个的工作目录中，基于一个已存在的基础镜像制作镜像
	*工作目录中的Dockerfile文件名必须大写开头
	1）FROM：第一个非注释行必须以FROM指令开头
	2）MAINTAINER：作者信息；已被LABEL命令代替
	3）COPY：将工作目录中的文件制作包含进镜像
		1.COPY <src> <des>	
			如果目标是目录必须以/结尾,若目标目录或父目录不存在会被自动创建；如果源是目录，拷贝时源目录不会被一同复制 ；可以在工作目录下建.dockerignore文件排除源目录中的文件
		2.COPY "<src>" ... "<dest>"  
	4）ADD：类似于COPY指令，ADD支持使用TAR文件和URL路径
		1.用法同COPY指令
		2.如果<src>为URL且<dest>不以/结尾，则<src>指定的文件将被下载并直接被创建为<dest>；如果<dest>以/结尾，则文件名URL指定的文件将将被直接下载并保存为<dest>/<filename>
		3.如果<src>是一个本地系统上的压缩格式的tar文件，它将被展开为一个目录，其行为类似于"tar -x"命令；然而，通过URL获取到的tar文件将不会自动展开。
		4.如果<src>在多个，或其间接或直接使用了通配符，则<dest>必须是一个以/结尾的目录路径；如果<dest>不以/结尾，则其被视作一个普通文件，<src>的内容将被直接写入到<dest>
	5）WORKDIR：指定一个默认目录
	6）VOLUME：定义镜像使用存储卷
	7）EXPOSE：用于为容器打开指定要监听的端口以实现与外部通信
		EXPOST <port>[/<protoclt>] ...
	8）ENV：用于为镜像定义所需的环境变量，并可被Dockerfile文件中位于其后的其它（如ENV、ADD、COPY等）所调用
		ENV <key> <value> 或
		ENV <key>=<value> ...

=========================================================================================

kuberneters安装：（kubeadm方式）
	参考文档：https://github.com/kubernetes/kubeadm/blob/master/docs/design/design_v1.10.md
	
* 一、主机环境预设
	1.node节点时间同步
	2.通过DNS完成各节点的主机名称解析，测试环境可使用hosts文件进行；
	3.各节点禁用SELinux;
	4.关闭各节点iptables/firewalld服务；
	5.各节点禁用Swap设备；
        1)swapoff -a 或 2)注释fstab中swap的挂载
	6.若要使用ipvs模型的proxy，各节点还需载入ipvs相关的各模块；
		创建内核模块载入相关的脚本文件/etc/sysconfig/modules/*ipvs.modules*，设定自动载入内核模块的脚本如下：
			#!/bin/bash
			ipvs_mods_dir="/usr/lib/modules/$(name -r)/kernel/net/netfilter/ipvs"
			  ..
			  ..

* 二、安装程序包
	master节点：docker-ce;kubeadm;kubelet;kubectl
	  master节点：kubeadm init
	node节点：kubeadm;kebelet;docker-ce
		yum install docker-ce
		若通过默认的k8s.gcr.io镜像仓库获取kubernetes系统组件的相关镜像，需要配置docker Unit File(/usr/lib/systemd/system/docker.service文件)中的Environment变量，为其定义实用的HTTPS_PROXY
			
			Environment="HTTPS_PROXY=PROTOCOL:/HOST:PORT"
			Environment="NO_PROXY=172.20.0.0/16,127.0.0.0/8"
        另外docker自1.13版起自动设置Iptables的FORWARD默认策略为DROP,这可能会影响kubernetes集群依赖的报文转发功能。因此，需要在docker服务启动后设置FORWARD链的默认策略为ACCEPT:
			Environment="/usr/sbin/iptables -P FORWARD ACCEPT" 

* 三、启动服务
	~]#systemctl daemon-reload
	~}#systemctl start docker.service
	~]#systemctl enable docker kubelet 

* 四、初始化master节点
	1.vi /etc/sysconfig/kuelet #设置忽略Swap启用的状态错误
	 KUBELET_EXTRA_ARGS="--fail-swap-on=flase"
	
    [root@master ~]# kubeadm init --kubernetes-version="v1.16.0" --pod-network-cidr="10.244.0.0/16" --dry-run --image-repository="http://xxxxx " --ignore-preflight-errors=Swap





Service:
	模型：userspace, ipatbles, ipvs


配置清单：
	创建资源的方法：
		apiserver仅接收JSON格式的资源定义；
		yaml格式提供配置清单，apiserver可自动将其转为json格式，而后再提交；

	大部分资源的配置清单：
		*apiVersion:group/version
			$ kubectl api-versions

		*kind：资源类别

		*metadata：元数据
			name
			namespace
			labels
			annotations

			每个资源的引用PATH
				/api/GROUP/VERSION/namespaces/NAMESPACE/TYPE/NAME

		*spec：期望的状态，disired state

		*status：当前状态，current state,本字段由kubernetes集群维护；

	Object URL:
		/apis/<GROUP>/<VERSION>/namespaces/<NAMESPACE_NAME>/<KIND>[/OBJECT_ID]/



网络插件(cni)：
	flannel
	calico
	canel
	kube-router
	...

	解决方案：
		虚拟网桥
		多路复用：MacVLAN
		硬件交换：SR-IOV

	kubelet, /etc/cni/net.d/

	flannel:
		支持多种后端：
			VxLAN
			host-gw:Host Gateway
			UDP:
	