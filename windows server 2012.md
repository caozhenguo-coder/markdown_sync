windows2012-datacenter

	系统安装时用diskpart工具来分区：
		在分区界面按Shift+F10调出cmd>diskpart
							DISPPART>?
							DISPPART>create partition primary size=60000	#创建大小为60G的主分区；默认单位是M	
							DISPPART>create partition extended	#创建扩展分区，size指定大小或直接回车：分配所有剩余空间
	核心安装下配置命令CMD>sconfig