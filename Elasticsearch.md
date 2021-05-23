ElasticSerach
 	ES是一个基于Lucene实现的开源、分布式、Restful的全文本搜索引擎;此外,它还是一个分布式实时文档存储,其中每个文档的每个field均是被索引的数据,且可被搜索;也是一个带实时分析功能的分布式搜索引擎,能够扩展至数以百计的节点实时处理PB级的数据

   基本组件:
	 索引(index):文档容器
		~= 表
	 类型(type):类型是索引内部的逻辑分区

	 文档(document):文档是Lucene索引和搜索的原子单位,它包含了一个或多个域,是域的容器;基于JSON格式表示。
		~= 记录
	 映射(mapping):类型用于定义格式;映射用于定义数据如何被分析。

	
	Restful API:
		四类API:
			1)检查集群、节点健康状态
			2)管理集群、节点
			3)执行CRUD操作
			4)执行高级操作

		ES访问接口:9200/tcp
		cur -X<VERB> '<PROTOCOL>://HOST:PORT/<PATH>?<QUERY_STRING>' -d '<BODY>'
			VERB: GET, PUT, DELETE等
			PROTOCOL: http, https
			QUERY_STRING:查询参数,例如?pretty表示用易读的JSON格式输出
			BODY:请求的主体

		 _cat API:
		    ~]# curl -XGET 'http://127.0.0.1:9200/_cat/'				
		 	~]# curl -XGET 'http://127.0.0.1:9200/_cat/nodes?v'   显示nodes的详细信息
			~]# curl -XGET 'http://127.0.0.1:9200/_cat/nodes?help'
			~]# curl -XGET 'http://127.0.0.1:9200/_cat/nodes?h=name,ip,port,uptime,heap.current'
		 
		 _cluster API:
			health:
				~]# curl -XGET 'http://127.0.0.1:9200/_cluster/health?pretty'
			state:状态信息
				~]# curl -XGET 'http://127.0.0.1:9200/_cluster/state/<metrics>?pretty'
				~]# curl -XGET 'http://127.0.0.1:9200/_cluster/state/state?pretty'
			stats:统计信息
				~]# curl -XGET 'http://127.0.0.1:9200/_cluster/stats?pretty'

	Plugins:
	   命令路径:
		 ~]# /usr/share/elasticsearch/bin/elasticsearch-plugin -h
		常用站点插件:
		  - marvel; bigdesk; head; kopf
		  站点插件的访问:http://HOST:PORT/_plugin/PLUGIN_NAME

	CRUD操作相关的API:

 		创建文档:PUT
			~]# curl -XPUT 'localhost:9200/students/class1/1?pretty' -d '
			> {
			>   "first_name": "Rong",
			>   "last_name: "Huang",
			>   "gender": "Female",
			>   "age": "23",
			>   "courses": "Luoying_Shenjian"
			> }'
		"error" : "Content-Type header [application/x-www-form-urlencoded] is not supported",
		  "status" : 406     //但是会报错,不确定是不是版本问题

 		获取文档:GET
			~]# curl -XGET 'localhost:9200/students/class1/1?pretty'

 		更新文本:UPDATE
			如果更新部分内容,得使用_update API; PUT方法则会覆盖原有文档
			~]# curl -XPOST 'localhost:9200/students/class1/2/_update?pretty' -d '
			> {
			>   "doc": { "age": 22 }
			> }'

 		删除文档:DELETE
			~]# curl -XDELETE 'localhost:9200/students/class1/2'

 		  删除索引:
			~]# curl -XDELETE 'localhost:9200/students'
			~]# curl -XGET 'localhost:9200/_cat/indices?v'

 	   *查询数据:
		  1.Query API:_search
		    1).通过Restful request API查询:	
		      ~]# curl -XGET 'localhost:9200/students/_search?pretty'
		    2).通过发送REST request body进行:
			  ~]# curl -XGET 'localhost:9200/students/_search?pretty' -d '
			   > {
			   >   "query": { "match_all": {} }
			   > }'
		
		  2.多索引、多类型查询:

			 /_search: 所有索引
			 /INDEX_NAME/_search: 单索引
			 /INDEX1,INDEX2/_search: 多索引
			 /s*,t*/_search:
			 /students/class1/_search: 单类型搜索
			 /students/class1,class2/_search: 多类型搜索

		  3.Mapping和Analysis:
			 ES:对每一个文档,会取得所有域的所有值,生成一个名为“_all”的域;执行查询时,如果在query_string未指定查询的域,则在_all域上执行查询操作:
		
			 	GET /_search?q='Xianglong'		//表示在_all域搜索
				GET /_search?=courses:'Xianglong%20Shiba%20Zhang'	//表示在指定的域上搜索

				数据类型: string, numbers, boolean, dates
			  查看指定类型的mapping示例:
				~]# curl 'localhost:9200/students/_mapping/class1?pretty'



 		ELK stack的另外两个组件:
			