#### 创建一个policy
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
        "min_age": "10m",
        "actions": {
          "forcemerge": {
            "max_num_segments": 1
          }
        }
      },
      "cold": {
        "min_age": "30m",
        "actions": {
          "allocate": {
            "require": {
              "type": "cold"
            }
          }
        }
      },
      "delete": {
        "min_age": "1h",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

#### 将ilm策略应用到索引模板中 
```text
PUT _template/my_template
{
  "index_patterns": ["test-*"], 
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1,
    "index.lifecycle.name": "my_policy", 
    "index.lifecycle.rollover_alias": "test-alias"
  }
}
```

#### 查看/修改 当前检查时间
```text
## 查看。 (iml默认10分钟检查1次)
GET _cluster/settings?include_defaults=true&filter_path=**.lifecycle.*

## 修改 
PUT _cluster/settings
{
  "persistent": {
    "indices.lifecycle.poll_interval": "1m"
  }
}
```

#### 解释ILM行为 
```text
GET user_log-*/_ilm/explain  
```