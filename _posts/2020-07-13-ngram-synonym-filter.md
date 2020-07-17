---
title: Combining Synonym and Ngram Filters in elasticsearch
subtitle: What could possibly go wrong?
date: "2020-07-13"
---
The [Synonym token filter](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-synonym-tokenfilter.html) and the [NGram token filter](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-ngram-tokenfilter.html) are two frequently used tools for text analysis with elasticsearch. 

This post is aimed at people already familiar with these concepts and does not provide too many technical explanations. Please refer to the official [elasticsearch docs](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-synonym-tokenfilter.html) for a more thorough description. 
[Here's](https://github.com/aplz/nlp_notebooks/blob/master/elasticsearch-nlp.ipynb) also a notebook giving a high-level demonstration of text analysis with elasticsearch. 

## Synonym token filter
The [Synonym token filter](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-synonym-tokenfilter.html) allows us to incorporate known synonyms in order to increase retrieval.

For instance, we can assume that a user searching for *"new york"* will also want to see results for *"big apple"*.
So let's assume that a *"new york"* and *"big apple"* can be treated as synonyms.

To demonstrate how the synonym token filter works, let's create a tiny index with just one synonym mapping saying that *"big apple"* is the same as *"new york"*.

```yaml
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "synonym_analyzer": {
          "tokenizer": "standard",
          "filter": [
            "synonym_filter"
          ]
        }
      },
      "filter": {
        "synonym_filter": {
          "type": "synonym",
          "synonyms": [
            "big apple => new york"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "text": {
        "type": "text",
        "analyzer": "synonym_analyzer"
      }
    }
  }
}
```

We can now use the [analyze API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-analyze.html) to see what this does.
```js
GET my_index/_analyze
{
  "text": "big apple",
  "analyzer": "synonym_analyzer"
}
```
This gives
```yaml
{
  "tokens" : [
    {
      "token" : "new",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "SYNONYM",
      "position" : 0
    },
    {
      "token" : "york",
      "start_offset" : 4,
      "end_offset" : 9,
      "type" : "SYNONYM",
      "position" : 1
    }
  ]
}
```
So, if we now add a document with text *big apple* and search for *new york* we will get that document as result:
```yaml
POST my_index/_doc
{
  "text": "big apple"
}

GET my_index/_search
{
  "query": {
    "match": {
      "text": "new york"
    }
  }
}
```
```yaml
...
"hits" : [
  {
    "_index" : "my_index",
    "_type" : "_doc",
    "_id" : "5u4RXXMBEQvh_5H7Ia1K",
    "_score" : 0.5753642,
    "_source" : {
      "text" : "big apple"
    }
  }
]
...
```

## Combine with Ngram Filter
If we combine the synonym filter now with an [NGram token filter](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-ngram-tokenfilter.html) you might get some strange results.

Especially, if the ngram token filter is applied **after** the synonym filter.  
Note: the other way is not allowed, you'll get an `Token filter [ngram_filter] cannot be used to parse synonyms` error.

So let's create our tiny index again, this time with an ngram filter included.
```yaml
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "synonym_analyzer": {
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
            "big apple => new york"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "text": {
        "type": "text",
        "analyzer": "synonym_analyzer",
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
Again, use the analyze API to check the output.
```yaml
GET my_index/_analyze
{
  "text": "big apple",
  "analyzer": "synonym_analyzer"
}
```
yields
```yaml
{
  "tokens" : [
    {
      "token" : "ne",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "SYNONYM",
      "position" : 0
    },
    {
      "token" : "new",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "SYNONYM",
      "position" : 0
    },
    {
      "token" : "ew",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "SYNONYM",
      "position" : 0
    },
    {
      "token" : "yo",
      "start_offset" : 4,
      "end_offset" : 9,
      "type" : "SYNONYM",
      "position" : 1
    },
    {
      "token" : "yor",
      "start_offset" : 4,
      "end_offset" : 9,
      "type" : "SYNONYM",
      "position" : 1
    },
    {
      "token" : "or",
      "start_offset" : 4,
      "end_offset" : 9,
      "type" : "SYNONYM",
      "position" : 1
    },
    {
      "token" : "ork",
      "start_offset" : 4,
      "end_offset" : 9,
      "type" : "SYNONYM",
      "position" : 1
    },
    {
      "token" : "rk",
      "start_offset" : 4,
      "end_offset" : 9,
      "type" : "SYNONYM",
      "position" : 1
    }
  ]
}
```
This will be different if you search only for *"apple"* as then the synonym filter does not apply. It works only for the phrase *"big apple"*:
```yaml
GET my_index/_analyze
{
  "text": "apple",
  "analyzer": "synonym_analyzer"
}
```
yields
```yaml
{
  "tokens" : [
    {
      "token" : "ap",
      "start_offset" : 0,
      "end_offset" : 5,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "app",
      "start_offset" : 0,
      "end_offset" : 5,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "pp",
      "start_offset" : 0,
      "end_offset" : 5,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "ppl",
      "start_offset" : 0,
      "end_offset" : 5,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "pl",
      "start_offset" : 0,
      "end_offset" : 5,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "ple",
      "start_offset" : 0,
      "end_offset" : 5,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "le",
      "start_offset" : 0,
      "end_offset" : 5,
      "type" : "<ALPHANUM>",
      "position" : 0
    }
  ]
}
```
### Here's where the trouble starts
Lets add a document
```yaml
POST my_index/_doc
{
  "text": "yorkshire"
}
```
and search for "big apple"
```yaml
GET my_index/_search
{
  "query": {
    "match": {
      "text": "big apple"
    }
  }
}
```
Would you want a match here?
```yaml
{
  ...
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.59039235,
    "hits" : [
      {
        "_index" : "my_index",
        "_type" : "_doc",
        "_id" : "5-4fXXMBEQvh_5H7a62y",
        "_score" : 0.59039235,
        "_source" : {
          "text" : "yorkshire"
        }
      }
    ]
  }
  ...
}
```
To summarize: synonyms are applied on each ngram produced by the ngram filter. So make sure to combine them with caution.



