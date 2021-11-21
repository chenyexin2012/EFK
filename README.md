# EFK
elasticsearch+fluentd+kibana日志系统搭建


# 文件目录详情

	EFK/
	├── docker-compose.yml
	├── elasticsearch
	│   └── config
	│       └── elasticsearch.yml
	├── fluentd
	│   ├── Dockerfile
	│   └── etc
	│       └── fluent.conf
	├── kibana
	│   └── config
	│       └── kibana.yml
	└── README.md


# 安装过程


1. 默认的fluentd没有安装elasticsearch输出插件，因此需要重新制作fluentd镜像

	进入fluentd文件夹，执行命令：
	
		docker build -t fluentd:1.1.1 .
		
2. 创建一个network
	
		docker network create efk_networks
	
		
3. 执行编排文件

		docker-compose up -d


4. fluentd已经配置了http，执行请求测试


		curl -H "Content-Type:application/json" -X POST -d '{"id":10000, "name": "张三", "vsersion": 13123154654, "date": "2021-10-10 12:12:12"}' 'http://localhost:9880/myapp.access'

		myapp.accesss对应fluent.conf的配置内容

	可以查看到elasticsearch索引中新建了一条索引，myapp-XXXXXXXX。

	
	
# elasticsearch设置密码

1. 修改elasticsearch.yml配置文件内容，加入：

		xpack.security.enabled: true
	
2. 重启后，进入elasticsearch容器，执行命令：

		elasticsearch-setup-passwords interactive
	
为各个用户名设置密码。

3. 修改kibana.yml配置文件内容，添加elasticsearch账号与密码：

		elasticsearch.username: 'elastic'
		elasticsearch.password: 'password'
	
4. 修改fluent.conf配置文件，elasticsearch输出配置添加账号密码：

		user elastic
		password elastic@123456

配置参考如下：

	<match *.**>
	  @type copy
	  <store>
		@type elasticsearch
		host elasticsearch
		port 9200
		user elastic
		password elastic@123456
		logstash_format true
		logstash_prefix log
		logstash_dateformat %Y%m%d
		include_tag_key true
		type_name access_log
		tag_key @log_name
		flush_interval 1s
	  </store>
	  <store>
		@type stdout
	  </store>
	</match>


5. 重启kibana、fluentd