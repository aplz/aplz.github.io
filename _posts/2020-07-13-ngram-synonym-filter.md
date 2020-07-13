---
title: Combining Synonym and Ngram Filters in elasticsearch
subtitle: What could possibly go wrong?
date: "2020-07-13"
---

## Combine with caution
The [Synonym token filter](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-synonym-tokenfilter.html) allows us to 
incorporate known synonyms in order to increase retrieval. However, when combined with an [NGram token filter](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-ngram-tokenfilter.html) you might get some strange results.

Now, what happens, if the ngram token filter is applied after the synonym filter?

### Create an example index
```js
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "text_analyzer": {
          "tokenizer": "standard",
          "filter": [
            "synonym_filter",
            "ngram_filter"
          ]
        }
      },
      "filter": {
        "ngram_filter": {
          "max_gram": 3,
          "min_gram": 2,
          "type": "ngram"
        },
        "synonym_filter": {
          "type": "synonym",
          "synonyms": [
            "word => longword"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "ctext": {
        "type": "text",
        "analyzer": "text_analyzer",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```
### Use the analyze API to check the output
```js
GET my_index/_analyze
{
  "text": "long",
  "field": "ctext" 
}
```
Result:
```js
{
  "tokens" : [
    {
      "token" : "lo",
      "start_offset" : 0,
      "end_offset" : 4,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "lon",
      "start_offset" : 0,
      "end_offset" : 4,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "on",
      "start_offset" : 0,
      "end_offset" : 4,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "ong",
      "start_offset" : 0,
      "end_offset" : 4,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "ng",
      "start_offset" : 0,
      "end_offset" : 4,
      "type" : "<ALPHANUM>",
      "position" : 0
    }
  ]
}
```
### Add a document
```js
POST my_index/_doc
{
  "ctext": "long"
}
```
### Search for "word"
```js
GET my_index/_search
{
  "query": {
    "match": {
      "ctext": "word"
    }
  }
}
```
### Surprised?
```js
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.57746404,
    "hits" : [
      {
        "_index" : "my_index",
        "_type" : "_doc",
        "_id" : "ftZkSHMBSWP7ubXHVybb",
        "_score" : 0.57746404,
        "_source" : {
          "ctext" : "long"
        }
      }
    ]
  }
}
```


