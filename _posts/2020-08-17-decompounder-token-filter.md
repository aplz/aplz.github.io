---
title: Demystifying elasticsearch Decompounder Token Filter
subtitle: Why does longest match not work?
date: "2020-08-17"
---
The [Dictionary decompounder token filter](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/analysis-dict-decomp-tokenfilter.html) is yet another tool for text analysis with elasticsearch that is especially useful for languages like German that tend to create awefully long composita.

Again, this post is aimed at people already familiar with this concepts and does not provide too many technical explanations. Please refer to the official [elasticsearch docs](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/analysis-dict-decomp-tokenfilter.html#analysis-dict-decomp-tokenfilter) for a more thorough description. 
[Here's](https://github.com/aplz/nlp_notebooks/blob/master/elasticsearch-nlp.ipynb) also a notebook giving a high-level demonstration of text analysis with elasticsearch. 

## Decompounder Token Filter
The [Dictionary decompounder token filter](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/analysis-dict-decomp-tokenfilter.html)
 is useful if we want to search for words that are usually contained in other words. Opposed to use cases of the [NGram token filter](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-ngram-tokenfilter.html), we would here _know_ the words or subwords we ware looking for and can explicitely provide them. 

Here's an example.   

```js
GET _analyze
{
  "tokenizer": "standard",
  "filter": [
    {
      "type": "dictionary_decompounder",
      "word_list": [
        "dampf",
        "dampfschiff",
        "schiff",
        "kapitän"
      ],
      "only_longest_match": "true"
    }
  ],
  "text": "dampfschifffahrtskapitän"
}
```

This gives us

```js
{
  "tokens" : [
    {
      "token" : "dampfschifffahrtskapitän",
      "start_offset" : 0,
      "end_offset" : 24,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "dampfschiff",
      "start_offset" : 0,
      "end_offset" : 24,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "schiff",
      "start_offset" : 0,
      "end_offset" : 24,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "kapitän",
      "start_offset" : 0,
      "end_offset" : 24,
      "type" : "<ALPHANUM>",
      "position" : 0
    }
  ]
}
```
The subword "dampf" will be omitted as "only_longest_match" is enabled. However, "schiff" is included in the result since for this check, the overlap from the first letter in the match is used. This only counts from the first character of the match!

Lastly: The original word is included in the result so that may also add more boost on words (maybe you do not want that for insignificant words).




