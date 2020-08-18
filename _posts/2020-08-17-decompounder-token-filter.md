---
title: Demystifying elasticsearch's Decompounder Token Filter
subtitle: What does the longest match setting do?
date: "2020-08-17"
---
The [Dictionary decompounder token filter](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/analysis-dict-decomp-tokenfilter.html) is yet another tool for text analysis with elasticsearch. It is especially useful for Germanic languages that tend to create awfully long compound words. 

These are some examples of compound words:
* _press conference_: _Pressekonferenz_
* _speed limit_: _Geschwindigkeitsbegrenzung_
* _one-way street_: _Einbahnstrasse_

You can find a nice explanation of this language characteristic along with more examples in this [duolingo post](https://forum.duolingo.com/comment/26620027/Compound-Words-1-Donaudampfschifffahrtsgesellschaft).

This post here is aimed at people already familiar with this concept and does not provide too many technical explanations. Please refer to the official [elasticsearch docs](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/analysis-dict-decomp-tokenfilter.html#analysis-dict-decomp-tokenfilter) for a more thorough description. 
[Here's](https://github.com/aplz/nlp_notebooks/blob/master/elasticsearch-nlp.ipynb) also a notebook giving a high-level demonstration of text analysis with elasticsearch. 

## Decompounder Token Filter
The [Dictionary decompounder token filter](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/analysis-dict-decomp-tokenfilter.html)
 is useful if we want to search for words that are usually contained in other words. Opposed to use cases of the [NGram token filter](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-ngram-tokenfilter.html), we would here _know_ the words or subwords we are looking for and can explicitly provide them. 

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
Having `only_longest_match` enabled, the subword _"dampf"_ will be omitted. That's because we have with _"dampfschiff"_ a longer match starting at the same character. In contrast, the subword _"schiff"_ is included in the result. That's because the `only_longest_match` check uses the character overlap from the first letter (or digit) in the match, and not any other in the sequence.

Lastly: The original word is included in the result so that may also add more boost on words (maybe you do not want that for insignificant words).




