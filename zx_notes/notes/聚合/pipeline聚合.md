可以分两类:  
- parent
对父聚合的结果再处理
- sibling
对同级的兄弟聚合的结果再处理

### buckets_path 语法 
大多数管道聚合需要其他聚合的输出来作为它的输入。这些聚合通过buckets_path参数来定义,遵循以下格式: 
```text
AGG_SEPARATOR       =  '>' ;
METRIC_SEPARATOR    =  '.' ;
AGG_NAME            =  <the name of the aggregation> ;
METRIC              =  <the name of the metric (in case of multi-value metrics aggregation)> ;
PATH                =  <AGG_NAME> [ <AGG_SEPARATOR>, <AGG_NAME> ]* [ <METRIC_SEPARATOR>, <METRIC> ] ;
```  

```text
聚合分隔符       = '>'
指标分隔符       = '.'
聚合             = <聚合名称>
指标             = <指标名称>
路径             = <聚合名称>[><聚合名称>]* [.<指标名称>]
```

例如:  my_bucket>my_stats.avg  

这些路径是对管道聚合的相对路径，而不是绝对路径。


### 具体类型
- Avg Bucket Aggregation  
sibling 。 对一个兄弟聚合下的某个指标求平均。 

- Derivative Aggregation  
parent 。 用于计算父直方图聚合中某个指标的导数。 

- Max/Min/Sum Bucket Aggregation
sibling 。 
  
- Stats/Extended Stats  Bucket Aggregation   
sibling 。 

- Percentiles Bucket Aggregation  
sibling 。 

- Moving Average Aggregation  
parent 。 移动平均  

- Moving Function Aggregation
parent 。 对移动平均的扩展， 参见 MovingFunctions类 
  
- Cumulative Sum Aggregation
parent 。 累加和
  
- Bucket Script Aggregation
parent 。 对父聚合中的一个或多个指标做脚本操作。  
  
- Bucket Selector Aggregation
parent 。 选择父聚合中的某个桶是否留存。  
  
- Bucket Sort Aggregation
parent 。 排序 
  
- Serial Differencing Aggregation
parent 。 串行差分， 当前值 减去 固定时间周期之前的值。   




