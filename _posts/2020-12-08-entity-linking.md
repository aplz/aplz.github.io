---
layout: post
mathjax: true
title:  "Entity Linking as a Ranking Problem"
date:   "2020-12-08"
---

In their seminal paper on entity linking, [Bunescu and Pasca](#BunescuPasca2006) proposed a model based on a ranking algorithm, more specifically a Ranking SVM. This algorithm was first introduced by [Thorsten Joachims](#Joachims2002) in the context of search engine analysis and was also used in later linking approaches, for instance [here](https://www.aclweb.org/anthology/C10-1032/), [here](https://www.aclweb.org/anthology/N10-1072.pdf) and [here](https://www.aclweb.org/anthology/C12-1137/).
Since providing full details on SVMs and Ranking SVMs is out of the scope of this post, we here provide the basic ideas and briefly point out how Ranking SVMs differ from standard SVMs. We assume background knowledge on SVMs and hint the reader at [Vapnik's Statistical Learning Theory](#Vapnik2000) for in-depth details.


### References
* <a name="Joachims2002">Thorsten Joachims, Optimizing search engines using clickthrough data, KDD 2002</a>
* <a name="BunescuPasca2006"> Razvan Bunescu, Marius Paşca, Using Encyclopedic Knowledge for Named entity Disambiguation, EACL 2006</a>
* <a name="Kendall1955">Maurice Kendall, Rank Correlation Methods, 1955</a>
* <a name="Vapnik2000">Vladimir N. Vapnik, The Nature of Statistical Learning Theory, Springer 2000</a>
* <a name="Dredze2010">Dredze et al., Entity Disambiguation for Knowledge Base Population, ACL 2010</a>
