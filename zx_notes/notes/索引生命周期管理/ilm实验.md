#### 1. 修改indices.lifecycle.poll_interval
ILM 默认每隔10分钟检查一次，看有哪些索引满足条件需要处理。
这个时间间隔通过indices.lifecycle.poll_interval来控制。
```text
## 查看当前配置
GET _cluster/settings?include_defaults=true&filter_path=**.lifecycle.*

## 为了方便，将间隔调整成10秒
PUT _cluster/settings
{
  "persistent": {
    "indices.lifecycle.poll_interval": "10s"
  }
}

## 实验完以后要复原
PUT _cluster/settings
{
  "persistent": {
    "indices.lifecycle.poll_interval": null
  }
}
```

#### 2. 创建ilm策略
```text
PUT _ilm/policy/pl_user_log
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_size": "20G",
            "max_docs": 2
          }
        }
      },
      "warm": {
        "min_age": "1m",
        "actions": {
          "shrink": {
            "number_of_shards": 1
          },
          "allocate": {
            "number_of_replicas": 1
          }
        }
      },
      "delete": {
        "min_age": "5m",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```
说明:  
- 在hot阶段，每当1个索引中文档数目超过2, 则滚动到一个新的索引；  
- 滚动完以后，老索引 在1分钟之后 进入 warm 阶段。索引收缩成1个分片，1个副本
- 索引滚动完5分钟以后，进入delete阶段。索引被删除。

#### 3. 创建索引模板 
```text
PUT _template/tp_user_log
{
  "index_patterns": ["user_log-*"],
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 2,
    "index.lifecycle.name": "pl_user_log",
    "index.lifecycle.rollover_alias": "user_log"
  },
  "mappings": {
    "properties": {
      "user_id": {
        "type": "integer"
      },
      "event": {
        "type": "keyword"
      }
    }
  }
}
```
说明:  
- 用index.lifecycle.name指定关联的ilm策略
- 用index.lifecycle.rollover_alias来指定需要滚动的别名

#### 4. 手动创建第一个索引
```text
## 创建索引时，指定一个别名user_log，与模板中的index.lifecycle.rollover_alias保持一致
## 并设置该别名为写别名 is_write_index=true 
PUT user_log-000001
{
  "aliases": {
    "user_log": {
      "is_write_index": true
    }
  }
}

## 查看索引信息
GET user_log-000001 
结果： 
{
  "user_log-000001" : {
    "aliases" : {
      "user_log" : {
        "is_write_index" : true
      }
    },
    "mappings" : {
      "properties" : {
        "event" : {
          "type" : "keyword"
        },
        "user_id" : {
          "type" : "integer"
        }
      }
    },
    "settings" : {
      "index" : {
        "lifecycle" : {
          "name" : "pl_user_log",
          "rollover_alias" : "user_log"
        },
        "number_of_shards" : "5",
        "provided_name" : "user_log-000001",
        "creation_date" : "1575019995348",
        "number_of_replicas" : "2",
        "uuid" : "mDJh73cWT8q_xyj785kfow",
        "version" : {
          "created" : "7020199"
        }
      }
    }
  }
}

```

#### 5. 使用ilm explain API 查看管理情况
```text
GET user_log-*/_ilm/explain 

结果: 
{
  "indices" : {
    "user_log-000001" : {
      "index" : "user_log-000001",
      "managed" : true,
      "policy" : "pl_user_log",
      "lifecycle_date_millis" : 1575019995348,
      "phase" : "hot",
      "phase_time_millis" : 1575019995552,
      "action" : "rollover",
      "action_time_millis" : 1575020003974,
      "step" : "check-rollover-ready",
      "step_time_millis" : 1575020003974,
      "phase_execution" : {
        "policy" : "pl_user_log",
        "phase_definition" : {
          "min_age" : "0ms",
          "actions" : {
            "rollover" : {
              "max_size" : "20gb",
              "max_docs" : 2
            }
          }
        },
        "version" : 5,
        "modified_date_in_millis" : 1575017936298
      }
    }
  }
}

```
说明: 可以看到每个索引当前处于哪个阶段，哪个活动。 

#### 6. 向索引中插入数据 
向别名user_log中插入5条数据， 然后不断使用explain API 来观察进度。 
```text
## 重复5次, 插入5条数据， 自动生成ID 
POST user_log/_doc
{
  "user_id": 111,
  "event": "buy"
}
```

#### 7. 索引自动滚动
等待一段时间后，ES自动创建一个新索引 名称为: user_log-000002。  
此时的别名情况为:  
```text
GET _alias/user_log*
 
结果: 
{
  "user_log-000002" : {
    "aliases" : {
      "user_log" : {
        "is_write_index" : true
      }
    }
  },
  "user_log-000001" : {
    "aliases" : {
      "user_log" : {
        "is_write_index" : false
      }
    }
  }
}
```
可以看到，user_log-000002变成了写入的索引。

```text
GET user_log-*/_ilm/explain  

结果: 
{
  "indices" : {
    "user_log-000002" : {
      "index" : "user_log-000002",
      "managed" : true,
      "policy" : "pl_user_log",
      "lifecycle_date_millis" : 1575020353918,
      "phase" : "hot",
      "phase_time_millis" : 1575020354214,
      "action" : "unfollow",
      "action_time_millis" : 1575020354214,
      "step" : "wait-for-follow-shard-tasks",
      "step_time_millis" : 1575020354265,
      "phase_execution" : {
        "policy" : "pl_user_log",
        "phase_definition" : {
          "min_age" : "0ms",
          "actions" : {
            "rollover" : {
              "max_size" : "20gb",
              "max_docs" : 2
            }
          }
        },
        "version" : 5,
        "modified_date_in_millis" : 1575017936298
      }
    },
    "user_log-000001" : {
      "index" : "user_log-000001",
      "managed" : true,
      "policy" : "pl_user_log",
      "lifecycle_date_millis" : 1575020354100,
      "phase" : "hot",
      "phase_time_millis" : 1575019995552,
      "action" : "complete",
      "action_time_millis" : 1575020354353,
      "step" : "complete",
      "step_time_millis" : 1575020354353,
      "phase_execution" : {
        "policy" : "pl_user_log",
        "phase_definition" : {
          "min_age" : "0ms",
          "actions" : {
            "rollover" : {
              "max_size" : "20gb",
              "max_docs" : 2
            }
          }
        },
        "version" : 5,
        "modified_date_in_millis" : 1575017936298
      }
    }
  }
}

```

#### 8. 老索引进入warm阶段
叒过了一段时间，user_log-000001变成warm阶段， 副本数和分片数都收缩成1 
```text
{
  "indices" : {
    "user_log-000002" : {
      "index" : "user_log-000002",
      "managed" : true,
      "policy" : "pl_user_log",
      "lifecycle_date_millis" : 1575020353918,
      "phase" : "hot",
      "phase_time_millis" : 1575020354214,
      "action" : "rollover",
      "action_time_millis" : 1575020363985,
      "step" : "check-rollover-ready",
      "step_time_millis" : 1575020363985,
      "phase_execution" : {
        "policy" : "pl_user_log",
        "phase_definition" : {
          "min_age" : "0ms",
          "actions" : {
            "rollover" : {
              "max_size" : "20gb",
              "max_docs" : 2
            }
          }
        },
        "version" : 5,
        "modified_date_in_millis" : 1575017936298
      }
    },
    "user_log-000001" : {
      "index" : "user_log-000001",
      "managed" : true,
      "policy" : "pl_user_log",
      "lifecycle_date_millis" : 1575020354100,
      "phase" : "warm",
      "phase_time_millis" : 1575020423893,
      "action" : "shrink",
      "action_time_millis" : 1575020434656,
      "step" : "check-shrink-allocation",
      "step_time_millis" : 1575020443975,
      "step_info" : {
        "message" : "Waiting for node [0V2pWFnETk6toPlNP_W3qA] to contain [5] shards, found [3], remaining [2]",
        "node_id" : "0V2pWFnETk6toPlNP_W3qA",
        "shards_left_to_allocate" : 2,
        "expected_shards" : 5
      },
      "phase_execution" : {
        "policy" : "pl_user_log",
        "phase_definition" : {
          "min_age" : "1m",
          "actions" : {
            "allocate" : {
              "number_of_replicas" : 1,
              "include" : { },
              "exclude" : { },
              "require" : { }
            },
            "shrink" : {
              "number_of_shards" : 1
            }
          }
        },
        "version" : 5,
        "modified_date_in_millis" : 1575017936298
      }
    }
  }
}
```


#### 9. 老索引被删除
叕过了一段时间，user_log-000001进入delete阶段，索引被删除。此时只剩下了user_log-000002
```text
GET user_log-*/_ilm/explain
{
  "indices" : {
    "user_log-000002" : {
      "index" : "user_log-000002",
      "managed" : true,
      "policy" : "pl_user_log",
      "lifecycle_date_millis" : 1575020353918,
      "phase" : "hot",
      "phase_time_millis" : 1575020354214,
      "action" : "rollover",
      "action_time_millis" : 1575020363985,
      "step" : "check-rollover-ready",
      "step_time_millis" : 1575020363985,
      "phase_execution" : {
        "policy" : "pl_user_log",
        "phase_definition" : {
          "min_age" : "0ms",
          "actions" : {
            "rollover" : {
              "max_size" : "20gb",
              "max_docs" : 2
            }
          }
        },
        "version" : 5,
        "modified_date_in_millis" : 1575017936298
      }
    }
  }
}
```
