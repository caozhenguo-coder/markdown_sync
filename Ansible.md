常用自动化运维工具
  *Ansible:python,Agentless,中小型应用环境
  *Saltstack:python,一般需部署agent,执行效率更高
  *Puppet:ruby,功能强大，配置复杂，重型，适合大型环境
  *Fabric:python,agentless
  *Chef:ruby,国内应用少
  *Cfengine
  *func

Ansible:
	模块化
	有Paramiko,pyYAML,jinja2(模板语言)三个关键模块
	支持自定义模块
	部署简单
	安全,基于OpenSSH
	幂等性:同一个任务不会因为重复执行带来意外情况
	无需代理不依赖PKI(无需ssl)
	可使用任何编程语言写模块
	YAML格式,编排任务，支持丰富的数据结构
	较强大的多层解决方案

  安装
	*rpm包安装:EPEL源
	  yum install ansible
	*编译安装
	  yum -y install python-jinja2 PyYAML python-paramiko python-babel python-crypto
	  tar xf ansible-1.5.4.tar.gz
	  cd ansible-1.5.4
	  python setup.py build
	  python setup.py install
	  mkdir /etc/ansible
	  cp -r examples/* /etc/ansible
	*Git方式
	  git clone git://github.com/ansible/ansible.git --rescurive
	  cd ./ansible
	  source ./hacking/env-setup
	*pip安装:pip是安装Python包的管理器,类似yum
	  yum install python-pip python-devel
	  yum install gcc glibc-devel zibl-devel rpm-bulid openssl-devel
	  pip install --upgrade pip
	  pip install ansible --upgrade
	*确认安装: ansible --version

   相关文件
    	/etc/ansible/ansible.cfg 主配置文件,配置ansible工作特性
		/etc/ansible/hosts  主机清单
		/etc/ansible/roles 存放角色的目录
    程序
		/usr/bin/ansible 主程序,临时命令执行工具
		/usr/bin/ansible-doc 查看配置文档,模块功能查看工具
		/usr/bin/ansible-galaxy 下载/上传代码或Roles模块的官网平台
		/usr/bin/ansible-playbook 定制自动化任务,编排剧本工具/usr/bin/ansible-pull远程执行命令的工具
		/usr/bin/ansible-vault 文件加密工具
		/usr/bin/ansible-console 基于console界面与用户交互的执行工具

   添加主机清单
	~]# vi /etc/ansible/hosts
   查看模块帮助
	~]# ansible-doc MODULE
   执行命令
	~]# ansible HOSTS -m MODULE -a 'arguments'	//HOSTS支持正规表达式和逻辑操作;-a后参数用双引号有时候会有问题
	~]# ansible-galaxy install ROLES  //从在线站点下载安装角色;https://galaxy.ansible.com
	~]# ansible-playbook /root/.ansible/hello.yaml  //执行playbook
	~]# ansible-vault [encrypt|decrypt|create|view|edit|rekey] hello.yml  //加密|解密...playbook
	~]# ansible-console  //主机清单交互式控制台
	~]# ansible-playbook --help   //playbook的相对目标为~/.ansible/
   playbook:
  ---
  -	hosts: HOST
	remote_user: USER

	tasks:
	  - name: NAME
	    mod: arg
	  -	name: NAME
		shell:/usr/bin/COMMAND
		when: {{ arg }}='AGR'			  条件判断是否执行
	  - name: NAME
	    group: name={{ item }}   迭代变量item;从with_items中循环取值
        with_items:
		   - group1
		   - group2
		   - group2
	  - name: NAME
	  - user: name={{item.name}} group={{item.group}}  迭代嵌套的了变量(字典)
	    with_items:
		  - { name: 'user1', group: 'group1' }
		  - { name: 'user2', group: 'group2' }
		  - { name: 'user3', group: 'group3' }
	  -	name: NAME
		shell:/usr/bin/COMMAND || /bin/true 忽略命令执行中出现的错误信息
	  - name: NAME							|
		 shell: /usr/bin/COMMAND			|
		 ignore_errors: True				或者使用ignore_errors来忽略错误信息
	  - name: copy conf file
	    copy: src=files/httpd.conf dest=/etc/httpd/conf/ backup=yes
		nodify: A_name  //监控此项
	handlers:		    //监控项发生变化则触发此触发器	
	  -name: A_name
	   service: name=httpd state=restarted	
	
