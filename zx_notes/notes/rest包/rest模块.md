## RestController 
RestController类是所有HTTP请求的总入口, 负责请求的转发。
RestController内维护了所有RestHandler构成的请求路径字典树——PathTrie，
在收到请求时，通过PathTrie找到对应的RestHandler来处理请求。

PathTrie结构: 
```text
/
  _cat
    indices
    aliases
    thread_pool 
    ...
	
  {index}
    _setting
    _mget 
    _search
    ...
	
  _bulk
  ...
``` 

## RestHandler 
RestHandler是Rest请求的处理器，它将REST请求转换成ES内部的请求，
然后执行并等待完成，再将结果转换成REST响应返回给客户端。

一个RestHandler一般完成下面3部分功能(以RestBulkAction为例): 
1. 在构造方法中将自己的请求路径注册到RestController去
    ```text
        controller.registerHandler(POST, "/_bulk", this);
        controller.registerHandler(PUT, "/_bulk", this);
        controller.registerHandler(POST, "/{index}/_bulk", this);
        controller.registerHandler(PUT, "/{index}/_bulk", this);
    ```
2. 在prepareRequest方法中将RestRequest转换为ES内部的请求
    ```text
        BulkRequest bulkRequest = Requests.bulkRequest();
        ......
        bulkRequest.timeout(request.paramAsTime("timeout", BulkShardRequest.DEFAULT_TIMEOUT));
        bulkRequest.setRefreshPolicy(request.param("refresh"));
    ```
3. 在prepareRequest方法末位返回一个RestChannelConsumer对象, 用来处理请求。
    ```text
       return channel -> client.bulk(bulkRequest, new RestStatusToXContentListener<>(channel));
    ```