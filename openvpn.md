准备工作：

Centos7.7 作为服务端
Windows 10 作为客户端
Easy-RSA 3.0.6
服务端openvpn版本 2.4.8
客户端openvpn版本2.4.8 :下载地址： https://swupdate.openvpn.org/community/releases/openvpn-2.4.8.tar.gz

关闭selinux：
	[root@localhost ~]# sed -i '/^SELINUX/s/enforcing/disabled/g' /etc/selinux/config
	[root@localhost ~]# setenforce 0

安装epel仓库和openvpn, Easy-RSA
	[root@localhost ~]# yum -y install epel-release && yum -y install openvpn easy-rsa

配置EASY-RSA 3.0
在/etc/openvpn文件夹下面创建easy-rsa文件夹，并把相关文件复制进去
	[root@localhost ~]# cp -r /usr/share/easy-rsa/3/* /etc/openvpn/easy-rsa/
	[root@localhost ~]# cp -p /usr/share/doc/easy-rsa-3.0.6/vars.example /etc/openvpn/easy-rsa/vars

创建OpenVPN相关的密钥
我们将创建CA密钥，server端、client端密钥，DH和CRL PEM, TLS认证钥匙ta.key。
	[root@localhost easy-rsa]# cd /etc/openvpn/easy-rsa/

初始化并建立CA证书
创建服务端和客户端密钥之前，需要初始化PKI目录
	[root@localhost easy-rsa]# ./easyrsa init-pki
	[root@localhost easy-rsa]# ./easyrsa build-ca nopass

创建服务器密钥
创建服务器密钥名称为 server1.key
	[root@localhost easy-rsa]# ./easyrsa gen-req server1 nopass

用CA证书签署server1密钥
	[root@localhost easy-rsa]# ./easyrsa sign-req server server1

创建客户端密钥
创建客户端密钥名称为 client1.key
	[root@localhost easy-rsa]# ./easyrsa gen-req client1 nopass

用CA证书签署client1密钥
	[root@localhost easy-rsa]# ./easyrsa sign-req client client1

创建DH密钥
根据在顶部创建的vars配置文件生成2048位的密钥
	[root@localhost easy-rsa]# ./easyrsa gen-dh

创建TLS认证密钥
	[root@localhost easy-rsa]# openvpn --genkey --secret /etc/openvpn/easy-rsa/ta.key

生成 证书撤销列表(CRL)密钥
CRL(证书撤销列表)密钥用于撤销客户端密钥。如果服务器上有多个客户端证书，希望删除某个密钥，那么只需使用./easyrsa revoke NAME这个命令撤销即可。
生成CRL密钥：
	[root@localhost easy-rsa]# ./easyrsa  gen-crl

复制证书文件：
复制ca证书，ta.key和server端证书及密钥到/etc/openvpn/server文件夹里
	[root@localhost easy-rsa]# cp -p pki/ca.crt /etc/openvpn/server/
	[root@localhost easy-rsa]# cp -p pki/issued/server1.crt /etc/openvpn/server/
	[root@localhost easy-rsa]# cp -p pki/private/server1.key /etc/openvpn/server/
	[root@localhost easy-rsa]# cp -p ta.key /etc/openvpn/server/
	复制ca证书，ta.key和client端证书及密钥到/etc/openvpn/client文件夹里
	[root@localhost easy-rsa]# cp -p pki/ca.crt /etc/openvpn/client/
	[root@localhost easy-rsa]# cp -p pki/issued/client1.crt /etc/openvpn/client/
	[root@localhost easy-rsa]# cp -p pki/private/client1.key /etc/openvpn/client/
	[root@localhost easy-rsa]# cp -p ta.key /etc/openvpn/client/
	复制dh.pem , crl.pem到/etc/openvpn/client文件夹里
	[root@localhost easy-rsa]# cp pki/dh.pem /etc/openvpn/server/
	[root@localhost easy-rsa]# cp pki/crl.pem /etc/openvpn/server/

修改OpenVPN配置文件:
  复制模板到主配置文件夹里面
	[root@localhost server]# cp -p /usr/share/doc/openvpn-2.4.8/sample/sample-config-files/server.conf /etc/openvpn/server/
	# 修改后的内容如下
	[root@localhost server]# cat server.conf |grep '^[^#|^;]'
	port 1194
	proto udp
	dev tun
	ca ca.crt
	cert server1.crt
	key server1.key  # This file should be kept secret
	dh dh.pem
	crl-verify crl.pem
	server 10.8.0.0 255.255.255.0
	ifconfig-pool-persist ipp.txt
	push "redirect-gateway def1 bypass-dhcp"
	push "dhcp-option DNS 114.114.114.114"
	duplicate-cn
	keepalive 10 120
	tls-auth ta.key 0 # This file is secret
	cipher AES-256-CBC
	compress lz4-v2
	push "compress lz4-v2"
	max-clients 100
	user nobody
	group nobody
	persist-key
	persist-tun
	status openvpn-status.log
	log-append  openvpn.log
	verb 3
	explicit-exit-notify 1

开启转发
修改内核模块
	[root@localhost server]# echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
	[root@localhost server]# sysctl -p
	net.ipv4.ip_forward = 1
修改防火墙
	[root@localhost server]# firewall-cmd --permanent --add-service=openvpn
	success
	[root@localhost server]# firewall-cmd --permanent --add-interface=tun0
	success
	[root@localhost server]# firewall-cmd --permanent --add-masquerade
	success
	[root@localhost server]# firewall-cmd --permanent --direct --passthrough ipv4 -t nat -A POSTROUTING -s  10.8.0.0/24 -o ens33 -j MASQUERADE
	success
	[root@localhost server]# firewall-cmd --reload
	success

启动服务并开机启动
	[root@localhost server]# systemctl enable openvpn-server@server
	[root@localhost server]# systemctl start openvpn-server@server
检查一下服务是否启动
	[root@localhost server]# netstat -tlunp
	[root@localhost server]# systemctl status openvpn-server@server


### OpenVPN 客户端配置

在openvpn服务器端操作，复制一个client.conf模板到/etc/openvpn/client文件夹下面。然后编辑该文件/etc/openvpn/client/client.conf
	[root@localhost openvpn]# cp -p /usr/share/doc/openvpn-2.4.8/sample/sample-config-files/client.conf /etc/openvpn/client/
	# 修改后的内容如下
	[root@localhost client]# cat client.conf |grep '^[^#|^;]'
	client
	dev tun
	proto udp
	remote 192.168.43.138 1194
	resolv-retry infinite
	nobind
	persist-key
	persist-tun
	ca ca.crt
	cert client1.crt
	key client1.key
	remote-cert-tls server
	tls-auth ta.key 1
	cipher AES-256-CBC
	verb 3

更改client.conf文件名为client.ovpn，然后把/etc/openvpn/client文件夹打包压缩：
	# 更改client.conf文件名为client.ovpn
	[root@localhost openvpn]# mv client/client.conf client/client.ovpn
	# 安装lrzsz工具，通过sz命令把 client.tar.gz传到客户机上面
	[root@localhost openvpn]# yum -y install lrzsz
	# 打包client文件夹
	[root@localhost openvpn]# tar -zcvf client.tar.gz client/
	# 使用sz命令，把client.tar.gz传到客户机上面
	[root@localhost openvpn]# sz client.tar.gz


在windows 10客户机连接测试
1.客户机安装openvpn-install-2.4.8-I602-Win10该软件包，安装完成之后解压刚才的client.tar.gz压缩包，把里面的文件复制到C:\Program Files\OpenVPN\config

2.需要把client.conf的名字改成client.ovpn，然后点击桌面上的OpenVPN GUI运行

>>>电脑右下角有一个小电脑,右键点击连接，连接成功之后小电脑变成绿色。


###注意事项：
做实验时在测试环境做的。如果在真实环境操作，请在出口防火墙添加端口映射，开放openvpn的端口1194 tcp和udp协议的