```text
已完成: 
	Cluster APIs
	Indices APIs
	cat APIs
	Aggregations
	Managing the index lifecycle

了解: 
	Document APIs
	Search APIs
	Query DSL
	Mapping
	Analysis
	Modules
	Index modules	
    Secure a cluster

TOD0:   
    Roll up or transform your data !!! 重要 
    Testing 
    How To !!! 重要。 优化写入速度，查询速度 

目前不重要: 
    Scripting
	SQL access
	Ingest node
	Command line tools
    Monitor a cluster
    Alerting on cluster and index events
    Administering Elasticsearch
	Frozen indices

	X-Pack APIs
```

```text
线程池: https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-threadpool.html

mapper: https://www.elastic.co/guide/en/elasticsearch/reference/7.2/index-modules-mapper.html 

translog : https://www.elastic.co/guide/en/elasticsearch/reference/7.2/index-modules-translog.html 
调整translog的写盘频率，可以加速索引速度。 
```

Questions: 
```text
ES插件是否支持自定义FSDirectory ? 用来自定义hybridfs中使用mmap的部分。
org.elasticsearch.index.store.FsDirectoryService.HybridDirectory


```