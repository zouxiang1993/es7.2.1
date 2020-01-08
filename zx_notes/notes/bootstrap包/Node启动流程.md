#### 1. new NodeEnvironment()
在/data目录下获取文件锁, 加载以前的metadata或者创建一份新的

#### 2. new PluginsService()
加载modules和plugins，更新配置信息

#### 3. new ThreadPool()
初始化所有的线程池

#### 4. new NodeClient()
用来执行各种命令

#### 5. new ScriptModule()

#### 6. new AnalysisModule()
注册所有的 char filter, token filter, tokenizer, analyzer, normalizer

#### 7. new SettingsModule()
注册所有的settings: node级别的settings, index级别的settings

#### 8. new NetworkService()
publish_host是集群内部节点通信的地址  
bind_host是外部应用可以访问ES集群的地址

#### 9. new ClusterService()
- new MasterService()
- new ClusterApplierService()
- new OperationRouting()

#### 10. new IngestService()

#### 11. newClusterInfoService()

#### 12. new ClusterModule()
new AllocationService() : 负责节点之间分片如何分配 

#### 13. new IndicesModule()
初始化所有的mapper  

#### 14. new SearchModule()
以下组件都支持通过插件自定义实现
- 注册Suggester  
- 注册Highlighter 
- 注册function_score中支持的所有函数 
- 注册QueryParser 
- 注册Rescorer
- 注册Aggregations
- 注册PipelineAggregations
- 注册FetchSubPhase (可以控制召回哪些东西)
- 注册SearchExt (可以在查询语句中新增一个部分)

以下组件不支持插件: 
- 注册Sorts  
- 注册ValueFormats

#### 15. createCircuitBreakerService() 
断路器 !!! 

#### 16. new GatewayModule()

#### 17. createPageCacheRecycler()
????

#### 18. createBigArrays()
????

#### 19. new NamedWriteableRegistry()

#### 20. new NamedXContentRegistry()

#### 21. new MetaStateService()
用来读/写 Manifest, MetaData(集群元数据), IndexMetaData(索引元数据)

#### 22. new IndicesService()

#### 23. new MetaDataCreateIndexService()

#### 24. new ActionModule()
- setupActions : 注册所有Action和TransportAction的映射关系
- new RestController() : http请求分发器

- initRestHandlers : 注册所有的RestHandler

#### 25. new NetworkModule()

#### 26. 构造 Transport 对象 (在transport-netty4 模块中)

#### 27. newTransportService()

#### ... 

#### new DiscoveryModule()

#### new NodeService()

#### newSearchService()
