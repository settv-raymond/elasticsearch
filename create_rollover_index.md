# Create rollover index
>如何新建一個依日期滾動的index
## Create rollover policy
>建立rollover policy，定義rollover的條件(筆數/時間等)
```
data={
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "rollover": {
            "max_primary_shard_size": "50gb",
            "max_age": "1M"
          },
          "set_priority": {
            "priority": 100
          }
        }
      }
    }
  }
}
ret = requests.put('http://192.168.31.128:9200/_ilm/policy/settv_logs_rollover-policy',json=data)
print(ret.text)
```

## Create index template
建立index範本，只要符合這個名稱的index，會自動套用這個範本
- index_patterns: 符合通配字符的index，會套用這個條件
- lifecycle: 定義要套用那個rollover policy,自動rollover後的index要使用那一個alias
- mapping: 
- aliases: 符合上述通配字符的index，都使用這個aliases。也就是說
	- 新增資料，使用settv_logs_write這個alias
	- 查詢資料，使用settv_logs這個alias，這個alias會查詢所有符合通配字符的index
```
data = {
  'index_patterns':['settv_logs-*'],
  "template": {
    "settings": {
      "index": {
        "lifecycle": {
          "name": "settv_logs_rollover-policy",
          "rollover_alias": "settv_logs_write"
        },
        "number_of_shards": "2"
      }
    },
    "mappings": {
      "dynamic": "strict",
      "properties": {
        "message": {
          "type": "text"
        },
        "messageType": {
          "type": "keyword"
        },
        "visitTime": {
          "type": "date",
          "format": "epoch_second"
        }
      }
    },
    "aliases": {
      "settv_logs": {}
    }
  }
}
ret = requests.put('http://192.168.31.128:9200/_index_template/template_settv_logs',json=data)
print(ret.text)
```
## Create index
- index 會以 settv_logs-2022.03.02-000001 命名
- 符合policy條件會自動增加
```
data = {
    'aliases': {
        'settv_logs_write':{}
    }
}
ret = requests.put('http://192.168.31.128:9200/<settv_logs-{now-d}-000001>', json=data)
print(ret.text)
```

## Add test data to index
```
es = Elasticsearch(host='192.168.31.128', port=9200)
i = 1000
for i in range(1000):
    doc = {
        'visitTime' : int(time.time()),
        'messageType' : 'T',
        'message' : str(i)
    }
    res = es.index(index='settv_logs_write', body=doc)
print(res)
```
