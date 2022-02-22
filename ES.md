# Elasticsearch commands

This file list some elasticsearch commands.

## Index template

```
PUT _index_template/mangamix-template
{
  "template": {
    "settings": {},
    "mappings": {
      "dynamic": "true",
      "dynamic_date_formats": [
        "strict_date_optional_time",
        "yyyy/MM/dd HH:mm:ss Z||yyyy/MM/dd Z"
      ],
      "dynamic_templates": [],
      "date_detection": true,
      "numeric_detection": false,
      "properties": {
        "audio": {
          "type": "text"
        },
        "metadata": {
          "properties": {
            "genre": {
              "type": "text"
            },
            "image": {
              "type": "text"
            },
            "theme": {
              "type": "text"
            }
          }
        },
        "name": {
          "type": "search_as_you_type",
          "doc_values": false,
          "max_shingle_size": 3
        },
        "video": {
          "type": "text"
        }
      }
    },
    "aliases": {
      "mangamix": {}
    }
  }
}
```

## Index

- `PUT /mangamix-00001`
- `DELETE /mangamix-00001`
- `GET /mangamix-00001/_mapping`
```
POST _reindex
{
  "source": {
    "index": "mangamix-000001"
  },
  "dest": {
    "index": "mangamix-000002"
  },
  "script": {
    "source": "ctx._source.remove('theme'); ctx._source.remove('audio'); ctx._source.remove('video')"
  }
}
```

### Document

`GET /mangamix/_doc/ec64a1b9b7376919cbce4c0090b7ddcc265948da504089c96fc067841578a0f6`

```
POST mangamix/_update/ec64a1b9b7376919cbce4c0090b7ddcc265948da504089c96fc067841578a0f6
{
  "script": {
    "source": "if (!ctx._source.containsKey('audio')) ctx._source.audio = []; if (!ctx._source.audio.contains(params.value)) ctx._source.audio.add(params.value)",
    "lang": "painless",
    "params": {
      "value": "url1"
    }
  }
}
```

```
POST mangamix/_update/ec64a1b9b7376919cbce4c0090b7ddcc265948da504089c96fc067841578a0f6
{
  "script": {
    "source": "if (ctx._source.audio.contains(params.tag)) { ctx._source.audio.remove(ctx._source.audio.indexOf(params.tag)) }",
    "lang": "painless",
    "params": {
      "tag": "url1"
    }
  }
}
```

### Search

```
GET /mangamix/_search
{
  "query": {
    "query_string": {
      "query": "jojo Kym"
    }
  }
}
```

```
GET /mangamix/_search
{
  "query": {
    "multi_match": {
      "query": "jojo Kym",
      "type": "bool_prefix",
      "fields": [
        "name",
        "name._2gram",
        "name._3gram"
      ]
    }
  }
}
```

