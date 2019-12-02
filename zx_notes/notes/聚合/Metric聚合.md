- Avg & Weighted Avg  平均,加权平均

- Cardinality 基数 discinct count  
    注意: cardinality的结果是近似值。基于HyperLogLog概率算法实现  
    TODO: 可以将要去重的字段作为文档id或routing, 使得相同的字段都落到同一个分片上,从而简单的实现精确的基数聚合。  
    TODO: 如果确定数量种类很少，也可以暴力去重聚合   
    
- Stats & Extended Stats  
    Stats : min / max / avg / sum / count   
    Extends Stats : 以上几种再加 平方和sum_of_squares / 方差variance, 标准差std_deviation, std_deviation_bounds
       
- Max / Min / Sum / Value Count   
 
- Percentiles / Percentile Ranks 

- Top Hits 

- Median Absolute Deviation 