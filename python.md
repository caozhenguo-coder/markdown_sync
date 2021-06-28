python数据结构与算法：
	顺序表：
	  结构：一体式(表头和数据区相连)，分离式(表头和数据区分离)
		1）基本顺序表：存储地址是连续的
		2）外围顺序表：存储地址非连续，存储的是下一个区域的地址
	链表：
		1）单向链表
		2）双向链表

	排序算法：
	  	1）冒泡排序
		2）选择排序
		3）插入算法
		4）希尔排序
		5）快速排序
		6）归并排序
	查找：
		二叉树

================================================================================

PYTHON:

pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/		//设置默认镜像源站点
python -m pip install --upgrade pip		//升级pip
pip --version
pip install 第三方包 [--upgrade]			//安装/升级第三方包
pip uninstall 第三方包
pip list								//查看下载的所有第三方工具包
pip install 第三方包 -i https://镜像源	//临时使用此镜像站下载

安装jupyter notebook 浏览器代码工具：
pip install  [numpy matplotlib pandas] wheel jupyter notebook


###对象###
Python中，一切皆对象(object)。每个对象由：标识(identity)、类型(type)、值(value)组成。
	·identity：用于唯一标识对象，通常对应于对象在计算机内存的地址。使用内置函数id(obj)可返回对象obj的标识。
	·type：用于表示对象存储的“数据”的类型。类型可以限制对象的取值范围以及可执行的操作。可以使用type(obj)获得对象的所属类型。
	·value：值表示对象所存储的数据的信息。使用print(obj)可以直接打印出值。

help()


 Python标识符命名规则：
  开发中，通常约定俗成的遵守如下规则：
	模块和包名：全小写字母，尽量简单。若多个单词之间用下划线
	函数名：全小写字母，多个单词之间用下划线隔开
	类名：首字母大写，采用驼峰原则。
	常量名：全大写字母，多个单词使用下划线隔开

	*数据类型及处理 
		 1）变量: name = "value"
			·变量赋值：
			  1. a=3
			  2. a=b=3		//链式赋值
			  3. a,b,c=4,5,6	//系列解包赋值
			·命名规范：
				字符串、数字、下划线任意组合；不能以数字开头或使用关键字
			·变量删除：del VAR
		 2）·常量:python不支持常量，只是约定俗成的把常量大写，在逻辑上不对其修改
			MAX = 120
		 3）时间：time()
			>>> import time
			>>> time.time()	
		 4）数据类型：整型(数值型)int()、浮点型float()、时间、字符串str()、布尔型、列表list()

	*运算符：算术运算、比较运算、赋值运算、逻辑运算、成员运算、身份运算、位运算
		算术运算：+ - * / % ** //
		比较运算：== != > < >= <=
		赋值运算：= += -= *= /= %= **= //=
		逻辑运算：and, or, not,  | ^ &
		同一运算符：is, is not		//is比较的是变量的id()；==比较的是变量的value
		成员运算：in, not in
		位运算： <<, >>
			注：位运算和算术运算>比较运算符>赋值运算符>逻辑运算符
	  ·进制转换： 
		二进制：0B
		八进制：0O
		十六进制：0X

	*字符串： 支持拼接或乘法拼接  name1+name2； name*3
	   	*字符串编码：python3的字符默认是16位；使用的是Unocode编码
	      >>>ord("a")
	         97
          >>>chr(97)
	         "a"
	   	*字符串提取：
		  >>>x="abcde"
		  >>>x[0]    //x[-5]
		     "a"
	   	*字符切片slice操作
		  >>> a="abcdefghijklmnopqrstuvwxyz"
		  >>> a[0:4:2]        //[起始偏移量start:终止偏移量end:步长step]
		   	  'ac'
		*字符串分割split()和合并jojin()
		  >>> x="hello_world"
		  >>> x.split("_")        //以"_"作为做为分隔符分割字符串
			  ['hello', 'world']
		  >>> time=["2020","06","20"]
		  >>> '-'.join(time)	  //以"_"作为连接符合并字符串
			  '2020-06-20'
		*字符串的驻留机制：仅保存一份相同且不可变字符串的方法（仅对包含“_、字母、数字”符合规则的字符串启用驻留机制）；
		  >>> a = "aa_11"
		  >>> b = "aa_11"
		  >>> a is b        //变量a和b共用一个对象
		      True
		*成员操作符：in ; not in
		  >>> a = "aancdaf789"
		  >>> "aan" in a
		     True
		*字符串处理方法：
		  查找：>>> a = "abdcdf789"
			len(a)：字符串长度；返回值：9
			a.startswith("ab")：是否以指定的字符串开头;返回值：True
			a.endswith("789")：是否以指定的字符串结尾；返回值：True
			a.find("d")：第一次出现指定字符串的位置；返回值：2
			a.rfind("d")：最后一次出现指定字符串的位置；返回值：4
			a.count("d")：指定字符串出现了几次；返回值：2
			a.isalnum()：是否全部为字母字符串；返回值：True。 	isalpha(); isdigit(); isspace(); isupper(); islower()
			>>> "*s x t**".strip('*')      //去除首尾信息；去左：lstrip()；去右：rstrip()
			    's x t'
			a.capitalize()：产生新的字符串，首字母大写
			a.title()：产生新的字符串，每个单词都首字母大写
			a.upper()：产生新的字符串，所有字符全转成大写
			a.lower()：产生新的字符串，所有字符全转成小写
			a.swapcase()：产生新的字符串，所有字母大小写转换
		  格式排版：
			center()、ljust()、rjust()这三个函数用于对字符串实现排版；居中、左对齐、右对齐
				>>> a = "sxt"
				>>> a.center(9,"*")    //共9个字符；居中；左右用"*"填充
					'***sxt***'
			format()：可以通过{索引}/{参数名}，直接映射参数值，实现对字符串的格式化。
			  >>>a="名字是：{0},年龄是：{1}"
			     a.format("高淇",18)
			  >>>a="名字是：{name},年龄是：{age}"	    //或使用参数值直接映射
			     a.format(name="高淇",age=18)
			  填充常跟对齐一起使用：
				^、<、>分别是居中、左对齐、右对齐，后面带宽度
				：号后面带填充的字符，只能是一个字符，不指定的话默认是空格
				>>> a="名字是：{name:*^8},年龄是：{age:*^8}"
				>>> a.format(name="高淇",age=18)
					'名字是：***高淇***,年龄是：***18***'
			 数字格式化：浮点数通过f,整数通过d进行需要的格式化。
				>>> a="我是{0},我的存款有{1:.2f}"
				>>> a.format("高淇",3888.234223)
				    '我是高淇,我的存款有3888.23'
				>>>3.1415926    {:.2f}    3.14      保留小数点后两位
				>  3.1415926    {:+.2f}   3.14      带符号保留小数点后两位
				>  3.1415926    {:.0f}    3         不带小数
				>  5            {:0>2d}   05        用零填充（填充左边，宽度为2）
				>  5            {:x<4d}   5xxx      用x填充（填充右边，宽度为4）
				>  10000        {:,}      1,0000    以逗号分隔的数字格式
				>  0.25         {:.2%}    25.00%    百分比格式
				>  100000       {:.2e}    1.00E+05  指数记法
				>  13           {:10d}          13  右对齐（默认，宽度为10）
				>  13           {:<10d}   13        左对齐，宽度为10
				>  13           {:^10d}       13    中间对齐，宽度为10
		·可变字符串：
		    在Python中，字符串属于不可变对象，不支持原地修改，如果需要修改其中的值，只能创建新的字符串对象。但是，经常我们确实需要原地修改字符串，可以使用io.StringIO对象或array模块。
				>>> import io
				>>> s='hello, sxt'
				>>> sio=io.StringIO(s)
				>>> sio
				    <_io.StringIO at 0x3d2c130438>
				>>> sio.getvalue()
				    'hello, sxt'
				>>> sio.seek(7)    //把指针移动到第7个字符位置，
				    7
				>>> sio.write("g")  //替换修改第7个字符串（不会产生新的对象）
				>>> sio.getvalue()
				    'hello, gxt'
		·浮点型： 3.14=314e-2	//四舍五入：round(4.56)-->5

    *列表：
		列表的创建：
		  1.基本语法[]创建：a=[]		//创建一个空列表对象
		  2.list()创建：a=list()		//使用list()可以将任何可迭代的数据转化成列表
		  3.range()创建整数列表		//range([start,]end[,step]); start默认是0；>>> a=list(range(10))
		  4.推导式生成列表			//用for循环创建多个元素；>>> a=[x*2 for x in range(5)]
														   >>> a=[x*2 for x in range(100) if x%9==0]   #用if过滤元素
		  week = ["monday","tuesday","wednesday"]		//定义列表
		  week.insert(2,"truesday")			//插入元素；获取帮助 help(week.insert)
		  week.append("truesday")		//列表后面附加插入元素
		  del week[3] 			//通过索引(下标)删除元素
		  week.remove("truesday")		//直接删除元素(从左至右,删除第一个此元素)
		  "friday" in week		//查看元素是否存在此列表中
		列表切片slice：
		  a[start:end:step]

		+		 		    列表拼接		列表拼接会产生新的列表对象		//a=[10,20]; a=a+[30]
	 	list.append(x)		增加元素		将元素x增加到列表list尾部
		list.extend(alist)	增加元素		将列表alist所有元素加到列表list尾部
		list.insert(index,x)增加元素		在列表list指定位置index处插入元素x
		list.remove(x)		删除元素		在列表list中删除首次出现的指定元素x
		list.pop(index)		删除元素		删除并返回列表list指定为止index处的元素，默认是最后一个元素
		list.clear()		删除所有元素	删除列表中所有元素，并不是删除列表对象
		list.index(x)		访问元素		返回第一个x的索引位置；index(value[,start[,end]]),可以用start和end指定搜索范围。
		list.count(x)		计数			返回指定元素x在列表list中出现的次数
		len(list)			列表长度		返回列表中包含元素的个数
		list.reverse()		翻转列表		所有元素原地翻转;
		reversed(list)		翻转迭代器	a=[10,20]; c=reversed(a) ;c ;list(c) 
		list.sort()			排序			所有元素原地排序;逆序：list.sort(reverse=True)
		list.copy()			浅拷贝		返回列表对象的浅拷贝
		max(list)			最大值
		min(list)			最小值
		sum(list)			求和			对于非数值型列表运算则会报错

	*元组tuple:列表属于可变序列，可以任意修改列表中的元素。元组属于不可变序列。因此，元组没有增加元素、修改元素、删除元素的方法。
	   元组的创建：
		1.通过()创建元组。小括号可以省略。
			a = (10,20,30) 或者 a = 10,20,30		//如果元组元素只有一个，要加逗号； a=(20,)
		2.通过tuple()创建元组：
			tuple(可迭代对象)：  b=tuple(); b=tuple("abc")		//用法类似list()
		3.生成器推导式创建元组：从形式上看，生成器推导式与列表推导式类似，只是生成器推导式使用小括号。列表推导式直接生成列表对象，生成器推
		  导式生成的不是列表也不是元组，而是一个生成器对象。      
			 我们可以通过生成器对象，转化成列表或者元组。也可以使用生成器对象的__next__()方法进行遍历，或者直接作为迭代器对象来使用。不管
		  什么方式使用，元素访问结束后，如果需要重新访问其中的元素，必须重新创建该生成器对象。
			>>> s=(x*2 for x in range(5))
			>>> s
				<generator object <genexpr> at 0x0000004A83E635C8>
			>>> tuple(s)
				(0, 2, 4, 6, 8)
			>>> list(s)        //只能访问一次元素，第二次就为空了，需要再生成一次
				[]
			也可以通过__next__()方法移动指针
			>>> tuple(s)
			()
			>>> s=(x*2 for x in range(5))
			>>> s.__next__()
			0
			>>> s.__next__()
			2
			>>> s.__next__()
			4
			元组总结：
				1.元组的核心特点是：不可变序列。
				2.元组的访问和处理速度比列表快
				3.与整数和字符串一样，元组可以作为字典的键，列表则永远不能作为字典的键使用

	*字典：字典是“键值对”的无序可变序列，字典中的每个元素都是一个“键值对”
		字典的定义方式：
		  1.通过dict()函数创建
			a = {'name':'gaoqi','age':18,'job':'programmer'}
			b = dict(name='gaoqi',age=18,job='programmer')
			c = dict([("name","gaoqi"),("age",18)])
		  2.通过zip()函数创建：
			k = ['a','b','c']
			v = [100,200,300]
			d = dict(zip(k,v))
		  3.通过fromkeys创建值为空的字典：
			a = dict.fromkeys(['name','age','job'])
			a
			{'name': None, 'age': None, 'job': None}
		字典的访问方式：
			1.a['name']		//如果键不存在报错
			2.通过get()方法：
			  a.get('name')		//如果键不存在则返回None
			  a.get('ddd','不存在')	//也可以自定义返回值
			3.列出所有的键值对：a.items()	
			  列出所有的键：a.keys()	
			  列出所有的值：a.values()		
			  键值对的个数：len(a)	
			  查寻键是否在字典中：
				>>> "name" in a
				    True
		字典元素的添加、修改、删除操作：
			增：a = {'name':'gaoqi','age':18}
				a['gender']='Male'		//如果键已存在则覆盖	
			改：update()：将新字典中的所有键值对全部添加到旧字典对象上。如果重复则覆盖
				a.update(b)
			删：del(a['age'])	//删除某个键
				a.clear()		//删除所有
				a.pop('name')	//弹出指定键并赋值给b: b = a.pop('name')
				a.popitem()		//随机弹出某个键
		字典序列解包(用于元组列表)：
			元组序列解包:[a,b,c]=[10,20,30]
			键解包：	a = {'name':'gaoqi','age':18,'job':'programmer'}
					x,y,z=a
			值解包：	x,y,z=a.values()
			键值对解包：	x,y,z=a.items()

	*集合：集合是无序可变，元素不能重复，实际上，集合底层是字典实现，集合的元素都是字典中的“键对象”，因此是不能重复。
		集合的定义、删除等操作：	
			1.b={10,20,30}
			  b.add(40)		//向集合中添加元素
			2.a=[10,20,30]
			  b=set(a)		//用set()方法可以把列表、元组等可迭代对象换成集合，如果数据有重复，则只保留一个。
			3.remove()删除指定元素；clear()清空整个集合
			4.集合的并集、交集、差集
			  a = {1,3,'sxt'}
			  b = {'he','it','sxt'}
			  并集：a|b 或 a.union(b)
			  交集：a&b 或 a.intersection(b)
    		  差集：a-b 或 a.difference(b)
	 
	*流程控制
		·if..else分支
			print("num小于10" if int(num)<10 else "num大于10")
		·while循环
		·for..in循环

	*推导式：通过推导式可以大量简化代码
		1.列表推导式：通过推导式生成列表对象
			[表达式 for item in 迭代的对象 [if 条件判断]]
				e.g: y = [x*2 for x in range(1,20) if x%5==0]
		2.字典推导式：通过推导式生成字典对象
			{key_expression: value_expression for 表达式 in 可迭代对象}
			  ：类似于列表推导式也可以加if条件判断、多个for循环。
				e.g: text = 'hello world, hello python'		//统计每个字符出现的次数
					 char_count = {c:text.conunt(c) for c in text}
					 print(char_count)
		3.集合推导式：通过推导式生成集合，和列表推导式的语法格式类似	//原理同字典只是没有value部分
			{表达式 for item in 可迭代对象 [if 条件凑数]}
				e.g: {x for x in range(1,100) if x%9==0}
		4.生成器推导式（生成元组）
			q：为什么元组没有推导式？，用小括号是什么推导式？
			>>> (x for x in range(1,100) if x%9==0)
			    <generator object <genexpr> at 0x000000311F1035C8>
			a:提示的是“一个生成器对象”。显然，元组是没有推导式的。
				注：生成器运行一次，迭代完里面就没有数据了。
		
	*函数：
		函数类型：
			1.内置函数
				可以直接使用的函数，类似str()、list()、len()等这些都是内置函数
			2.标准函数
				可以通过import语句导入库，然后使用其中定义的函数
			3.第三方库函数
				Python社区提供的库。下载安装这些库后，也可以通过import语句导入使用
			4.用户自定义函数
	
		函数的定义：
			def FUN_NAME(A,B):
				'''添加函数文档说明'''
				fun_body

			调用help(函数名.__doc__)可以查看函数的文档说明。	
		返回值：
		  return语句		
			1.如果函数体中包含return语句，则结束函数执行并返回值；
			2.如果函数体中不包含return语句，则返回None值；
			3.要返回多个返回值，使用列表、元组、字典、集合将多个值“存起来”即可。	

		#参数类型：
			1.位置参数：def f1(a,b,c):				//实参默认按位置顺序传递，实参需要和形参个数匹配。				
					   	  ...			
			2.默认值参数:def f1(a,b,c=10,d=20):		//带默认值的形参必须放在形参列表后面
					  	 ...	
			3.命名参数：def f1(a,b,c):
					 	...
					f1(c=10,a=20,b=30)			//命名参数传值时可以打乱顺序
			4.可变参数：可变数量的参数。分两种情况：
				1）*param（一个星号），将多个参数收集到一个“元组”对象中。
					def f1(a,b,*c):		//*c作为元组，可被传递任意个参数
						...
					f1(8,9,19,20,...)
				2）**param（两个星号），将多个参数收集到一个“字典”对象中。
					def f2(a,b,**c):	//**c作为字典来接收参数
						...
					f3(8,9,name='tom',age=18)
			5.强制命名参数：在带星号的“可变参数”后面增加新的参数，必须是“强制命名参数”。
					def f1(*a,b,c)
						...
					f1(2,b=3,c=4)		//可变参数后的参数必须强制命名，否则会报错
				
	 常用函数：
		1.zip()：可以对多个序列进行并行迭代，在最短序列“用完”时就会停止。
		2.eval()函数：将字符串当成有效的表达式来求值并返回计算结果
			a = 10;  b = 20
			c = eval("a+b")			//eval()函数内引号中的表达式会被执行运算
		3.divmod()函数：同时得到商和余数并返回
			>>> divmod(13,3)
			(4, 1)
		4.lambda表达式和匿名函数：
			lambda表达式可以用来声明匿名函数。lambda函数是一种简单的、在同一行中定义函数的方法。lambda函数实际生成了一个函数对象。；lambda表达式只允许包含一个表达式，不能包含复杂语句，该表达式的计算结果就是函数的返回值
			lambda表达式的基本语法如下：
				lambda arg1,arg2,arg3,...: <表达式>		//arg1/arg2/arg3为函数的参数，<表达式>相当于函数体，运算结果是返回值。
				e.g:
					f = lambda a,b,c:a+b+c
					print(f)
					print(f(2,3,4))
	
	 递归函数:调用自身的函数
	
	 嵌套函数（内部函数）：内部函数只能在当前定义函数的内部使用
		nonlocal关键字：用于在内层函数中声明外层函数定义的变量，以便可以修改此变量的值
		golbal:	用来声明全局变量
	
	 构造函数：
	  ·实例对象：（实例属性+实例方法）
		class Student:
		    def __init__(self,name,score):
		        self.name = name
		        self.score = score
		    def say_score(self):
		        print("{0}分数是： {1}".format(self.name,self.score))
		              
		s1 = Student("tom",18)
		s1.say_score()

		
	*类对象：（类属性+类方法 & 实例属性+实例方法 ）
		·生成类对象：
			class ClsName:
				pass
		·类属性：
			class Student:
				company = "SXT"	  #类属性
				count = 0		  #类属性
	
		·类方法：类方法是从属于“类对象”的方法。类方法通过装饰器@classmethod来定义，格式如下：
				@classmethod
				def 类方法名(cls [,形参列表]):
					函数体
				注：
				  	1.@classmethod必须位于方法上面一行
					2.第一个cls必须有：cls指的就是“类对象”本身
					3.调用类方法格式："类名.类方法名(参数列表)"。参数列表中，不需要也不能给cls传值。
					4.类方法中访问实例属性和实例方法会导致错误
					5.子类继承父类方法是，传入cls是子类对象，而非父类对象
		·静态方法：Python中允许定义与“类对象”无关的方法，称为“静态方法”。“静态方法”和在模块中定义普通函数没有区别，只不过“静态方法”放到		 了“类的名字空间里面”，需要通过“类调用”。
				静态方法通过 装饰器@staticmethod来定义。格式如下：
					@staticmethod
					def 静态方法名([形参列表]):
						函数体

		  ·要实现对象直接调用可以用__call__方法来定义：
	   			__call__方法和可调用对象：	定义了__call__方法的对象，称为“可调用对象”，即该对象像函数一样被调用。
					函数调用：a()	; -- 通过call方法实现对象调用:obj()		

			#其他：
				1.dir(obj) 可以获得对象 的所有属性、方法
				2.obj__dict__ 对象的属性字典
				3.pass 空语句
				4.isinstance（对象,类型）判断“对象”是不是“指定类型”
	*实例对象：
		实例属性：通过定义构造函数（构造器）def __init__(self)来初始化实例属性
			1)__new__()
			2)__init__()
			第一个参数必须为self
			无返回值

		实例方法：
			def __init__(self,...):		#构造函数固定方法:__init__
				...
			def ins_name(self,...)
				...

		  e.g:
			class Student:
			    company = "SXT"			#类属性
			    count = 0				#类属性
			    
			    def __init__(self,name,score):	#构造函数固定方法:__init__
			        self.name = name			#实例属性
			        self.score = score			#实例属性
			        Student.count = Student.count+1
			        
			    def say_score(self):			#实例方法
			        print("我的公司是：",Student.company)
			        print(self.name,"的分数是：",self.score)
		        
			s1 = Student("张三",80)		#s1是实例对象，自动调用__init__()方法
			s1.say_score()
			print('一共创建(0)个Student对象'.format(Student.count))
		
	   私有属性：self__age = age   调用私有属性： e=mycls  print(e._mycls__age)	
	   私有方法：def __method(self)	调用私有方法：e=mycls  e._mycls__method()

	   @property装饰器：可以将一个方法的调用方式变成“属性调用”

	*面向对象的三大特征：
		·封装（隐藏）：隐藏对象的属性和实现细节，只对外提供必要的方法。
	
		·继承：子类可以继承多个父类的属性；如果在类定义中没有指定父类，则默认父类是object类。object是所有类的父类
			  里面定义了一些所有类共有的默认实现，比如：__new__()
					class 子类类名(父类1[,父类2,...]):
			  定义子类时，需在其构造函数中调用父类的构造函数（但没有严格限定）。调用格式如下：
					父类名.__init__(self,参数列表)	
	
		·多态：多态是指同一个方法的调用由于对象不同会产生不同的行为。
	
	 特殊属性：
		class A:
		    pass
		class B:
		    pass	
		class C(B,A):
		    def __init__(self,nn):
		        self.nn = nn
		    def cc(self):
		        print("cc")
		
		c = C(3)
		
		print(dir(c))
		print(c.__dict__)
		print(c.__class__)
		print(C.__bases__)
		print(C.mro())
		print(A.__subclasses__())

	*设计模式：
		1.工厂模式
		2.单例模式

############################################################################
			
	
	 异常处理：
		try...except...else
		try...except...finally		//无论是否有异常都会执行finally模块里的代码
	
	 traceback:
	   用traceback语句记录错误日志：
		 	import traceback
		
		 	try:
			    print("step111")
			    nummm = 1/0
			except:
			    with open("d:/a.txt","a") as f:		//用with文件管理（用完后会自行关闭文件）
			        traceback.print_exc(file=f)		//将错误写入文件（日志）以便查看
	
	 文件操作：open(),openline[s](),write(),writeline[s]()

		语法：open(name[,mode[,bufsize]])
			mode:r,w,a,+,b

			file = open(r"d:\a.txt","w")
			str = ["aaa\n","bbb\n","ccc\n"]
			file.writelines(str)
			file.close()
	   		===============================
			try:
			    f = open(r"d:\b.txt","a")
			    str = ["aaa\n","dddd\n"]
			    f.writelines(str)			//write:写字符；writelines:写行、列表
			except BaseException as e:
			    print(e)
			finally:
			    f.close()
	
	 二进制文件读写、拷贝：
	
			with open(r"C:\Users\admin\Pictures\splash.jpg","rb") as r:
			    with open(r"D:\mypython\copy.jpg","wb") as w:
			        for line in r.readlines():
			            w.write(line)
			print("拷贝完成.......")
	
	 序列化与反序列化：
		pickle.dump(obj,file)	//obj:要序列化的对象；file:要保存的文件
		pickle.load(file)		//从file读取数据，反序列化成对象
	
	 CSV模块：（comma sparat valume）
	
	 os和os.path模块：可以调用操作系统的可执行文件、命令，直接对操作系统进行操作。
		·os.system()：		//调用系统可执行命令
		·os.startfile():	//调用可执行文件
	   os模块-文件和目录操作：
		*os模块下常用操作文件的方法
		  remove(path)		//删除指定的文件
		  rename(src,dest)	//重命名文件或目录
		  stat(path)			//返回文件的所有属性
		  listdir(path)		//返回path目录下的文件和目录列表
		*os模块下关于目录操作的相关方法
		  mkdir(path)		//创建目录
		  makedirs(path/to/...)	//创建多级目录
		  rmdir(path)		//删除目录
		  removedirs(path/to/...)	//删除多级目录
		  getcwd()			//返回当前工作目录：current work dir
		  chdir(path)		//把path设为当前工作目录
		  walk()			//遍历目录树
		  sep				//当前操作系统所使用的路径分隔符
		*os.path
	
		 *os.walk():递归遍历所有的子目录 各子文件
	
		 *shutil(crs,dest[,ignore=""])模块（拷贝和压缩）文件和目录
		  shutil()和zipfile()模块_压缩和解压缩
	
	 递归：（自己调用自己）
	
	  *递归算法原理——阶乘计算：
			def factorial(n):
			    if n == 1:
			        return n
			    else:
			        return n*factorial(n-1)
			    
			print(factorial(5))
	
	 模块化编程理念_什么是模块_哲学思想：
		语句-(语句封装)-->函数-(函数+变量)-->类&对象-(类&函数)-->模块+=-->包-->程序
		
		
	 模块的导入：
		import:导入模块
		from_import:导入模块中的函数/类
	
	 包(package)的概念和结构：
		当一个项目中有很多个模块时，需要再进行组织。我们将功能类似的模块放到一起，形成了“包”。本质上，“包”就是一个必须有__init__.py的文件夹。
		*新建包：
		*导入包：form PACKAGE.MODULE import FUNTION
	
	 sys.path和模块搜索路径
		1.内置模块	2.当前目录	3.程序的主目录	4.pythonpath目录（如果已经设置了pythonpath环境变量）
		5.标准链接库目录	6.第三方库目录（site-packages目录）	7. .pth文件的内容（如果存在的话）	8. sys.path.append()临时添加的目录
	
			>>>import sys
			>>>print(sys.path)
	
	 GUI编程:		https://docs.python.org/
	  常用的GUI库：
		0.Turtle：是python内置的图形库
		1.Tkinter：是Python的标准GUI库，支持跨平台的GUI程序开发。适合小型的GUI程序编写
		2.Graphics：是基于Tkinter扩展的图形库
		3.wxPython：是比较流行的GUI库，适合大型应用程序开发，功能强于tkinter,整体设计框架类似于MFC(Micorsoft Function Classes微软基础类库）
		4.PyQT：QT是一种开源的GUI库，适合大型GUI程序开发。


第三方库：
	科学计算：
		numpy：数据收集
		matplotlib：数据图形图表展示
		pandas：数据操作

	爬虫：
		Requests：自动爬取HTML页面，自动网络请求提交。

			In [27]: import requests
			In [28]: url = "http://m.ip138.com/ip.asp?ip="
			In [29]: try:
			    ...:     r = requests.get(url+'202.204.80.112')
			    ...:     r.raise_for_status()					//判断访问站点状态码是否为200
			    ...:     r.encoding = r.apparent_encoding		//判断通过http报文header和body猜测的编码格式是否相同
			    ...:     print(r.text[-500:])					//输出url后500字节
			    ...: except:
			    ...:     print("爬取失败")

		Robots：网络爬虫排除标准

		Beautiful Soup：解析HTML页面信息标记与提取方法

		
		








