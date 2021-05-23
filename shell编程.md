bash脚本编程:
	顺序执行
	选择分支
	循环执行
		for,while,until

  while语句:	
	  循环控制语句:
		while CONDITION1; do
			CMD1
			...
			if CONDITION2; then
				continue
			fi
			CMDn
			...
		done

	  while循环的特殊用法(遍历文件的行):
		while read VARIABLE; do
			循环体;
		done < /path/to/somefile

		注:依次读取/path/to/somefile文件中的每一行,且将其赋值给VARIABLE变量;
  
  for语句:
	  for循环的特殊用法:
		for ((控制变量初始化;条件判断表达式;控制变量的修正语句)); do
			循环体
		done

	   e.g:
		for ((j=1j<=9;j++)); do
			for ((i=1;i<=j;i++)); do
				echo -e -n "${i}X${j}=$[${i}*${j}]\t"
			done
			echo
		done

  if语句:
	单分支语句:
	  if CONDITION; then
	 	分支

    双分支语句:
	  if CONDITION; then
		分支1
	  else
		分支2
	  fi

	多分支语句:
	  if CONDITION1; then
		分支1
	  elif CONDITION2; then
		分支2
	  ...
	  else CONDITION; then
		分支n
	  fi

  case语句:
	  case $VARIABLE in
	  PAT1)
	 	   分支1
		   ;;
	  PAT2)
		   分支2
		   ;;
	     *)
		   分支n
		   ;;
	   esac

	  #PATTERN仅支持GLOB风格通配符;
		*:任意长度的任意字符
		?:任意单个字符
		[]:范围内任意单个字符
		a|b:a或b

  函数:结构化编程,代码重复使用
	两种方法定义函数:  
	1)	function f_name {
			函数体
		}

	2)	f_name() {
			函数体
		}

	函数调用:给定函数名;
	函数的重修周期:每次被 调用时创建,返回时终止;
		返回结果为函数体中运行的最后一条命令的状态结果;
		自定义状态返回值,需要使用:return
			return[0-255]
				0:成功
				1-255:失败

  变量
	局部变量:作用域是函数的生命周期;在函数结束时自动销毁;
    	定义局部变量:local VARIABLE
	本地变量:作用域是运行脚本的shell进程的生命周期;因此,其作用范围为当前shell脚本程序文件
	环境变量:

	算术运算:
		let VAR=expression
		VAR=$[expression]
		VAR=$((expression))
		VAR=$(expr arg1 arg2 arg3)	//(单括号)表示命令引用;把命令的结果当作要处理的字符串

		注:增强型赋值:+= -= *= /= %= 在bash中使用let来描述

		自增(自减):
			VAR=$[$i+1]
			let VAR+=1
			let VAR++
 	变量赋值
	~]# echo ${A:-30}	//若变量A有值则输出A的值；否则输出A的默认值30
	~]# echo ${A:+30}	//若变量B不空则赋值为30
	~]# echo ${A:=30}	//
	~]# echo ${A:2:3}	//2：offset偏移量(可省)； 3：lengh所要取出的长度

	#!/bin/bash
	#
	. /root/parameter	//在脚本中定义此行；给此脚本提供配置文件
	echo ${para:-silent}

	~]# mktemp /tmp/file.XXX	//XXX是mktemp命令防文件重名自动给的随机字符串(给定的X要大于等于3个)；加-d选项为创建临时目录
	~]# FILE=`mktemp /tmp/file.XXX`	//放置于变量中以引用临时文件

	trap命令：bash内建命令；用来在脚本中捕捉信号(9&15信号不可捕捉)
	  ~]# trap 'COMMAND' 信号列表
						]# kill -l  //各种信号列表
						1: HUP
						2: INT
						9: KILL
						15: TERM
	 e.g:
		 #!/bin/bash
      	 #
       	 trap 'echo "You go...."' INT
       	 while :; do
      	        date
      	        sleep 2
       	 done
					

  数组:
	变量:存储单个元素的内存空间;
	数组:存储多个元素的连续的内存空间;
		数组名:整个数组只有一个名字;
		数组索引:编号从0开始
			数组名[索引]
			${ARRAY_NAME[index]}   //中括号会被识别为特殊字符,所以大括号不能省

		注:bash-4及之后的版本支持自定义索引格式,而不仅仅是0,1,2,...数字格式;
		   此类数组称之为"关联数组";

	声明数组:
		declare -a NAME 声明索引数组;
		declare -A NAME 声明关联数组;
	数组中元素的赋值方式:
		索引数组的赋值:
		1):~]# array[0]=pig
		   ~]# array[1]=dog
		2):~]# weekdays=("Mon" "Tue" "wed")
		3):~]# read -a animals
		   pig dog cat
		   ~]# echo ${animals[2]}
		关联数组的赋值:
		4):~]# animals=([0]="dog" [3]="pig") 	//支持稀疏格式的序列/ 关联数组赋值
		   ~]# echo ${animals[3]}
		5):~]# animals[us]="america"
		   ~]# animals[uk]="United Kingdom"
		   ~]# echo ${animals[uk]}
	
		引用数组中的所有元素:
		  [root@node2 ~]# echo ${animals[*]} 或
		  [root@node2 ~]# echo ${animals[@]}		    
		    	[root@node2 ~]# echo ${#animals[*]}  //数组的长度(数组中元素的个数)	

		数组元素切片: ${ARRAY_NAME[*]:offset:num}
			offset:偏移量;  num:要取出的元素个数

		向非稀疏格式数组中追加元素:
		   ARRAY_NAME[${#ARRAY_NAME[*]}]=

		删除数组中的某元素:
			unset ARRAY[INDEX]
			unset ARRAY	  //删除数组

		关联数组:
			declare -A ARRAY_NAME
			  ARRAY_NAME=([index_name1]="value1" [index_name2]="value2" ...)


  	练习:生成10个随机数;排序
		求100内偶数之和
		各种打印九九乘法表;及倒序排列
		


