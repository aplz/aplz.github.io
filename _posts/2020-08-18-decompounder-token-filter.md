---
title: Demystifying elasticsearch's Decompounder Token Filter
subtitle: What does the longest match setting do?
date: "2020-08-18"
---
The [Dictionary decompounder token filter](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/analysis-dict-decomp-tokenfilter.html) is yet another tool for text analysis with elasticsearch. It is especially useful when creating search engines that need to handle languages like German. These languages tend to create awfully long compound words. If you are not familiar with compound words, you can find a nice explanation of this language characteristic along with more examples in this [duolingo post](https://forum.duolingo.com/comment/26620027/Compound-Words-1-Donaudampfschifffahrtsgesellschaft).

Now, why is this an issue?

Suppose you have medical text data and want to find mentions of diseases (_"Erkrankung"_ or _"Krankheit"_ in German). To achieve a high recall, you would have to search for _"Nierenerkrankung"_ (kidney disease) and _"Atemwegserkrankung"_ (respiratory disease) and many, many more words denoting individual diseases. And this does not even include synonyms like _"Nierenerkrankheit"_ (this post on [handling synonyms with elasticsearch](https://aplz.github.io/2020-07-13-ngram-synonym-filter/) may help you with that)! 

This problem can be approached with the decompounder token filter.  

## Decompounder Token Filter
The decompounder token filter is useful if we want to search for words that are usually contained in other words. Opposed to use cases of the [NGram token filter](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-ngram-tokenfilter.html), we here _know_ the words or subwords we are looking for and can explicitly provide them. Just like the word parts _"erkrankung"_ or _"krankheit"_ in the example above.

You'll find a detailed description and how this token filter works in the [official docs](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/analysis-dict-decomp-tokenfilter.html). This post aims to explain a potentially unexpected behavior that we can observe with the `only_longest_match` setting enabled.

Here it goes.   
Let's try out the Analyze API with a custom dictionary decompounder filter and the compound word _"dampfschifffahrtskapitän"_ as input.

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
