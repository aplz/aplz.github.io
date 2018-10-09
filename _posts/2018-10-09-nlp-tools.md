---
layout: page
title: Favourite NLP tools
---

# Natural Language Processing
There are many great NLP libraries and frameworks out there. But if you work in industry, you will find that they often don't meet your needs. Be it in language support, application domain or simply in the licensing model.  
If you have to dig in and implement your own tool, my favourite go-to libraries are the following. This is by no means a comprehensive list, but my biased selection of tools that I've used on a daily basis during the last 10 plus years.

## mallet: machine learning for language toolkit
[mallet](http://mallet.cs.umass.edu/) is a Java library for many machine learning tasks with a long success story both in academia and industry. 
Originally implemented by [Andrew McCallum et al.](http://mallet.cs.umass.edu/), the code is now maintained by [David Mimno](https://github.com/mimno/Mallet).	

To me, it is the go-to tool mostly for **Topic Modelling** and **Named Entity Recognition** tasks.

### Topic Modelling
In a nutshell, Topic Modelling aims at forming clusters of words that often co-occur in a collection of documents. These clusters form so-called latent topics and each document is represented by a probability distribution over these topics. These distributions can among other things be used for dimensionality reduction (often necessary in classification tasks) or to formulate similarity measures that you need for [information retrieval](https://en.wikipedia.org/wiki/Information_retrieval) or [entity linking](https://en.wikipedia.org/wiki/Entity_linking) problems. 

As such, the algorithm can be applied to any domain where the input can be divided into units: words in documents, pixels in images. Thus, it works on news paper articles, movie reviews, medical documents but also for image analysis tasks.

Next to vanilla topic modelling ([Latent Dirichlet Allocation](http://jmlr.csail.mit.edu/papers/v3/blei03a.html)), mallet also offers [multi-language topic modelling](http://www.aclweb.org/anthology/D09-1092) or [hierarchical topic modelling](https://papers.nips.cc/paper/2466-hierarchical-topic-models-and-the-nested-chinese-restaurant-process.pdf) which is especially useful if you have no idea about the number of topics in your corpus. 

To get started with topic modelling, I highly recommend the [in-browser LDA demo](https://mimno.infosci.cornell.edu/jsLDA/jslda.html) by David Mimno. Just upload your own text collection and see what happens. Or check out this [site](https://ldavis.cpsievert.me/reviews/reviews.html) for an interactive example with IMDB movie reviews. [Here](http://research.cs.rutgers.edu/~ishanic/research/mainpage.html#objdet) is an example using LDA for image analysis.

As said at the beginning, this is a subjective selection. So please also take a look at other prominent topic modelling tools like [gensim](https://radimrehurek.com/gensim/) (Python) or other libraries listed [here](http://www.cs.columbia.edu/~blei/topicmodeling_software.html). 

#### Links
* David Blei's [general introduction to topic modelling](http://www.cs.columbia.edu/~blei/papers/Blei2012.pdf)
* Checkout [Blei Lab](https://github.com/Blei-Lab) for code releases associated with the research group's papers. 

### Named Entity Recognition
Named Entity Recognition (NER) can be considered a sequence analysis task where the goal is to detect subsequences, i.e. tokens in a sentence, that make up the mention of a specifc entity like a person, location or organization. NER has various use cases for instance in semantic search and opinion mining, where your task is to tackle "things not strings".
 
Most available tools (like spaCy, see below) give you feature configurations or even pretrained models for this task. Unfortunately, they usually don't support all languages or entity types. If your entities are for instance gene names (when you try to build a medical search engine), banks (e.g. if you tackle spam detection in email), product names or features (if you aim for opinion mining), you will often find the need to implement and train your own NER model. 

mallet provides with its [Conditional Random Fields](http://dirichlet.net/pdf/wallach04conditional.pdf) (CRF) implementation a sequence segmentation algorithm that is easy to use but also highly configurable. In my opinion, it gives you a deeper understanding of what actually happens under the hood: in contrast to other tools, you can get useful confidence values for predictions and not just "this is it" predictions.  
Additionally, mallet comes with a comprehensive set of preprocessors. From those, you just select the ones most suitable to your needs or maybe add your own customized and fancy ones. You then plug them together into the available pipeline implementation that will also transform the input into the format required by the CRF implementation.

Note that CRFs may be applied not only to NLP tasks but also many other cases like [object recognition](http://www.cs.columbia.edu/~mcollins/papers/NIPS2004_0810.pdf) or [image segmentation](https://ipvs.informatik.uni-stuttgart.de/mlr/marc/publications/09-plath-ICML.pdf). 


## Apache OpenNLP
Apache [OpenNLP](https://opennlp.apache.org/) is Java library that offers along with the application code, pretrained models for word segmentation (a.k.a. tokenization), [Part-of-Speech](https://en.wikipedia.org/wiki/Part-of-speech_tagging) (PoS) tagging, [Shallow Parsing](https://en.wikipedia.org/wiki/Shallow_parsing) (a.k.a. Chunking) and Named Entity Recognition for German, English and other languages. 

You can also use OpenNLP to train your own models. Admittedly, I never did that since I mostly used the PoS tagger. PoS tagging is a cumbersome task if you have to annotate the data on your own (meh!) to train a good model, so pretrained models are great! Also, the PoS tag of a word may be context dependent, but it is never domain dependent. Which means that you can apply the pre-trained model for any domain. But, as a side-node, PoS tagging works best on well formed sentences, so don't be dissappointed if you apply the model on table-like data or user generated content like tweets or short posts in forums. 

### Links
* If you use [elastic](https://www.elastic.co/products/elasticsearch) to store your documents (which I would highly recommend, :heart_eyes:): Alex Reelsen wrote an [OpenNLP ingest processor](https://github.com/spinscale/elasticsearch-ingest-opennlp) that you can use along with your other index ingestion tools.

## Apache lucene
[lucene](https://lucene.apache.org/core/) is a lightning fast search engine used by both [solr](LINK) and [elasticsearch](link) (to name only a few). The basic idea is to store documents in an [inverted index](https://en.wikipedia.org/wiki/Inverted_index) (just like Google does) that maps words to documents in which they occur. Providing your term of interest in a search query, it will give you the most relevant documents.

Since one of the main use cases of lucene is **information retrieval**, i.e. search in documents, it also comes with a bunch of so called _analyzers_. These analyzers split documents into tokens, which are the actual "storage units" in an inverted index. Among many other things, they detect word units, remove stop words or [stem](https://en.wikipedia.org/wiki/Stemming) tokens. These steps are often the start in every NLP processing pipeline. You can feed a text document into an analyzer and it gives you the tokenized variant which you can then feed into the rest of your NLP pipeline. 

Analyzers are available for [many different languages](https://lucene.apache.org/core/6_0_0/analyzers-common/overview-summary.html). I am not aware of a "plug&play" implementation, but the major benefit to me is that this is very fast and  doesn't consume lots of memory to load pretrained models that you might not even need at this point. 

### Links
* [lucidworks blog](https://lucidworks.com/blog/)
* [Baeldung](https://www.baeldung.com/lucene-analyzers)'s overiew of popular lucene analyzers.
 
## Other frameworks

If you just get started with NLP or just want to try out its possibilties, plugging together your own pipelines will probably get you frustrated pretty fast. Luckily, there are several good frameworks on the market. Again, this is a very subjective selection, but my favourites are [spaCy](https://spacy.io/) and [TextBlob](https://textblob.readthedocs.io/en/dev/).  
Both are written in python and come with a bunch of features. 
For a comparision of spaCy and TextBlob on German texts, check out [this notebook](https://github.com/aplz/nlp_notebooks/blob/master/nlp_sandbox.ipynb).

If you work in academia, you should most definitely check out [Stanford NLP](https://nlp.stanford.edu/software/). It has an outstanding success story and implements scientifially sound and proven, state of the art solutions to  many NLP problems. The only drawback to me is that the licensing policy will often not meet your industry budget.

