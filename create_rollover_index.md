# Create rollover index

## Create rollover policy
```
data={
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "rollover": {
            "max_primary_shard_size": "50gb",
            "max_age": "10m",
            "max_docs": 1000
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