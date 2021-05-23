Docker(LXC):lxc-> libcontainer-> runC
	用户空间隔离(namespaces)：
	| namesppace	  |  系统调用参数		| 隔离内容	  		   	 |  内核版本
	 ·UTS(用户名&域名) |	 CLONE_NEWUTS	| 主机名和域名  			 |  2.6.19
	 ·IPC(进程间通信)  |	 CLONE_NEWIPC	| 信号量、消息队列和共享内存|  2.6.19
	 ·PID(进程树)	  |	 CLONE_NEWPID	| 进程编号				 |  2.6.24
	 ·Network		  |  CLONE_NEWNET	| 网络设备、网络栈、端口等	 |  2.6.29
	 ·Mount(挂载树)	  |	 CLONE_NEWNS	| 挂载点（文件系统）	     |  2.4.19
	 ·User			  |	 CLINE_NEWUSER  | 用户和用户组			 |  3.10
 
	iproute命令：
	  管理名称空间：
		ip netns help
	  在名称空间里添加网卡：
		ip link help	
		ip link add veth1.1 type veth peer name veth1.2
		ip link set veth1.2 netns NS_NAME
		ip netns exec R1 ip link set dev veth1.2 name eth0

	[root@master ~]# docker run --name nginx -it --rm nginx:latest /bin/sh
	[root@master ~]# docker run --name web1 --rm -p 80 nginx:mainline-alpine
	[root@master ~]# docker run --name web1 --rm -p 192.168.43.88:80:80 nginx:mainline-alpine
	[root@master ~]# docker port web1
	[root@master ~]# docker run --name web1|host --network container:web2 -it --rm nginx:mainline-alpine
		--network container:web2 #共享web2|host的名称空间
	  wget -O - -q 127.0.0.1

	自定义docker0桥的属性信息：/etc/docker/daemon.json文件
		{
			"bip": "192.168.1.5/24",
			"fixed-cidr": "10.20.0.0/16",
			"fixed-cidr-v6": "2001:db8::/64",
			"mtu": 1500,
			"default-gateway": "10.20.1.1",
			"default-gateway-v6": "2001:db8:abcd::89",
			"dns": ["10.20.1.2","10.20.1.3"]
		}
	
	dockerd守护进程的C/S，其默认仅监听Unix Socket格式的地址，/var/run/docker,sock;如果使用TCP套接字
		/etc/docker/daemon.json:
			"hosts": ["tcp://0.0.0.0:2375", "unix:///var/run/docker.sock"]

		则客户端可用"-H|--host"选项来连接查看已监听IP的dockerd服务端；
			docker -H 172.20.0.67:3275 COMMADN

	可为docker创建其它网桥：
		docker info 查看支持网格类型
		docker network --help
		docker network create -d bridge --subnet "172.26.0.0/16" --gateway "172.26.0.1" mybr0
		docker network ls
	容器加入mybr0网络：
		docker run --name t1 -it --net mybr0 busybox:latest

  存储:
   在容器中使用Volumes
	·为docker run 命令使用-v选项即可使用Volume
		·Docker-managed volume
		  ~]# docker run -it -name bbox1 -v /data busybox
		  ~]# docker inspect -f {{.Mounts}} bbox1
			·查看bbox1容器的卷、卷标识符及挂载的主机目录
		·Bind-mount Volume
		  ~]# docker run -it -v HOSTDIR:VOLUMEDIR --name bbox2 busybox
		  ~]# docker inspect -f {{.Mounts}} bbox2
		·复制使用其它容器的卷，使用--volumes-from选项
		  ~]#docker run -it --name bbox2 --volumes-from bbox1 busybox


 Dockerfile:
	


   ·RUN
	 ·用于指定docker build过程中运行的程序，其可以是任何命令
	 ·Syntax
		·RUN <command>或
		·RUN ["<executable>","<param1>","<param2>"]
	 ·第一种格式中，<command>通常是一个shell命令，且以"/bin/sh -c"来运行它，这意味着此进程在容器中的PID不为1,不能接收Unix信号，因此，当使用docker stop <container>命令停止容器时，此进程接收不到SIGTERM信号；
	 ·第二种语法格式中的参数是一个JSON格式的数组，其中<executable>为要运行的命令，后面的<paramN>为传递给命令的选项或参数；然而此种格式的命令不会以"/bin/sh -c"来发起，因此常见的shell操作如变量换以及通配符(?,*)替换将不会运行；不过，如果要运行的命令依赖于此shell特性的话，可以将其替换为类似下面的格式；
		·RUN ["/bin/bash","-c","<executable>","<param1>"]

   ·CMD
	  ·类似于RUN指令，CMD指令也可用于运行任何命令或应用程序，不过，二者的运行时间点不同
		·RUN指令运行于映像文件构建过程中，而CMD指令运行于基于Dockerfile构建出的新映像文件启动一个容器时
		·CMD指令的首要目的在于为启动的容器指定默认要运行的程序，且其运行结束后，容器也将终止；不过，CMD指定的命令其可以被docker run的命令行选项所覆盖
		·在Dockerfile中可以存在多个CMD指令，但仅最后一个会生效
	·Syntax
		·CMD <command>或
		·CMD ["<executable>","<param1>","<param2>"]或
		·CMD ["<param1>","<param2>"]
	  ·前两种语法格式的意义同RUN
	  ·第三种则用于为ENTRYPOINT指令提供默认参数