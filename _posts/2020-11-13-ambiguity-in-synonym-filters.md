---
title: You shall know a word by the company it keeps
subtitle: Ambiguity and synonym filters in elasticsearch
date: "2020-11-13"
---

We've already seen some potential pitfalls when using [synonym token filters](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-synonym-tokenfilter.html) in the post 
about [nGram and synonym filters](https://aplz.github.io/2020-07-13-ngram-synonym-filter). 
Another thing to keep in mind when working with synonym filters is *ambiguity*, the problem that a word or even a phrase can have multiple meanings. 
For instance, the acronym "NL" can refer to the National League in baseball,
the Netherlands, a [complexity class](https://en.wikipedia.org/wiki/NL_(complexity)) or various [other things](https://en.wikipedia.org/wiki/NL) 
and the underlying sense heavily depends on the context of its mention.

Now, when searching for one of these entities, we would like to retrieve their mentions in acronym form as well. 
This is also called *entity based retrieval* and assumes that the sense of a word is resolved. Note that this post scratches only the surface of the underlying problem and aims at demonstrating what can go wrong if ambiguity is neglected. 
If you want to dig deeper, there is a nice and much more detailed blog post by elastic about [synonyms in search](https://www.elastic.co/blog/boosting-the-power-of-elasticsearch-with-synonyms).
And if you want to read more on ambiguity and approaches to resolve it, here's a link to my [thesis on entity disambiguation](https://bonndoc.ulb.uni-bonn.de/xmlui/handle/20.500.11811/6697). 

So, how can we achieve that searching for a word yields also mentions of its acronym? 
One *could* think that we just need to add this information in terms of synsets.

```js
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
            "NL,National League",
            "NL,Netherlands"
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
Add an example document. 

```js
POST my_index/_doc
{
  "text": "We've won the NL this year."
}
```
Note that the mention of "NL" is disambiguated through the context and rather obviously hints at the National League. 

But, searching for "Netherlands"
```
GET my_index/_search
{
  "query": {
    "match": {
      "text": "Netherlands"
    }
  }
}
```
gives you this:
```js
{
  ...
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.43648317,
    "hits" : [
      {
        "_index" : "my_index",
        "_type" : "_doc",
        "_id" : "PrguwXUB_m9BbtwuoOqo",
        "_score" : 0.43648317,
        "_source" : {
          "text" : "We've won the NL this year."
        }
      }
    ]
  }
  ...
}
```
Nope. Not what the user wants, right?

So keep in mind that synonym filters are a powerful tool but can yield undesired results if synsets are not created carefully.
