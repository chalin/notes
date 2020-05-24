Word embeddings in 2017: Trends and future directions

 21 Oct 2017  in [word embeddings](http://ruder.io/tag/word-embeddings/index.html)  [natural language processing](http://ruder.io/tag/natural-language-processing/index.html)  [nlp](http://ruder.io/tag/nlp/index.html)   ~ 16 min read.

 [![Word embeddings in 2017: Trends and future directions](../_resources/50df57a943ba6602593ec589fa975a00.png)   # Word embeddings in 2017: Trends and future directions](https://twitter.com/intent/tweet?text=Word%20embeddings%20in%202017%3A%20Trends%20and%20future%20directions%20%C2%BB&hashtags=word%20embeddings,natural%20language%20processing,nlp&url=http://ruder.io/word-embeddings-2017/)

Table of contents:

- [Subword-level embeddings](http://ruder.io/word-embeddings-2017/index.html#subwordlevelembeddings)
- [OOV handling](http://ruder.io/word-embeddings-2017/index.html#oovhandling)
- [Evaluation](http://ruder.io/word-embeddings-2017/index.html#evaluation)
- [Multi-sense embeddings](http://ruder.io/word-embeddings-2017/index.html#multisenseembeddings)
- [Beyond words as points](http://ruder.io/word-embeddings-2017/index.html#beyondwordsaspoints)
- [Phrases and multi-word expressions](http://ruder.io/word-embeddings-2017/index.html#phrasesandmultiwordexpressions)
- [Bias](http://ruder.io/word-embeddings-2017/index.html#bias)
- [Temporal dimension](http://ruder.io/word-embeddings-2017/index.html#temporaldimension)
- [Lack of theoretical understanding](http://ruder.io/word-embeddings-2017/index.html#lackoftheoreticalunderstanding)
- [Task and domain-specific embeddings](http://ruder.io/word-embeddings-2017/index.html#taskanddomainspecificembeddings)
- [Embeddings for multiple languages](http://ruder.io/word-embeddings-2017/index.html#embeddingsformultiplelanguages)
- [Embeddings based on other contexts](http://ruder.io/word-embeddings-2017/index.html#embeddingsbasedonothercontexts)

The word2vec method based on skip-gram with negative sampling (Mikolov et al., 2013) [[49](http://ruder.io/word-embeddings-2017/index.html#fn:49)] was published in 2013 and had a large impact on the field, mainly through its accompanying software package, which enabled efficient training of dense word representations and a straightforward integration into downstream models. In some respects, we have come far since then: Word embeddings have established themselves as an integral part of Natural Language Processing (NLP) models. In other aspects, we might as well be in 2013 as we have not found ways to pre-train word embeddings that have managed to supersede the original word2vec.

This post will focus on the deficiencies of word embeddings and how recent approaches have tried to resolve them. If not otherwise stated, this post discusses *pre-trained* word embeddings, i.e. word representations that have been learned on a large corpus using word2vec and its variants. Pre-trained word embeddings are most effective if not millions of training examples are available (and thus transferring knowledge from a large unlabelled corpus is useful), which is true for most tasks in NLP. For an introduction to word embeddings, refer to [this blog post](http://ruder.io/word-embeddings-1/).

## Subword-level embeddings

Word embeddings have been augmented with subword-level information for many applications such as named entity recognition (Lample et al., 2016) [[8](http://ruder.io/word-embeddings-2017/index.html#fn:8)], part-of-speech tagging (Plank et al., 2016) [[9](http://ruder.io/word-embeddings-2017/index.html#fn:9)], dependency parsing (Ballesteros et al., 2015; Yu & Vu, 2017) [[17](http://ruder.io/word-embeddings-2017/index.html#fn:17), [10](http://ruder.io/word-embeddings-2017/index.html#fn:10)], and language modelling (Kim et al., 2016) [[11](http://ruder.io/word-embeddings-2017/index.html#fn:11)]. Most of these models employ a CNN or a BiLSTM that takes as input the characters of a word and outputs a *character-based* word representation.

For incorporating character information into pre-trained embeddings, however, character n-grams features have been shown to be more powerful than composition functions over individual characters (Wieting et al., 2016; Bojanowski et al., 2017) [[2](http://ruder.io/word-embeddings-2017/index.html#fn:2), [3](http://ruder.io/word-embeddings-2017/index.html#fn:3)]. Character n-grams -- by far not a novel feature for text categorization (Cavnar et al., 1994) [[1](http://ruder.io/word-embeddings-2017/index.html#fn:1)] -- are particularly efficient and also form the basis of Facebook's fastText classifier (Joulin et al., 2016) [[4](http://ruder.io/word-embeddings-2017/index.html#fn:4)]. Embeddings learned using fastText are [available in 294 languages](https://github.com/facebookresearch/fastText/blob/master/pretrained-vectors.md).

Subword units based on byte-pair encoding have been found to be particularly useful for machine translation (Sennrich et al., 2016) [[12](http://ruder.io/word-embeddings-2017/index.html#fn:12)] where they have replaced words as the standard input units. They are also useful for tasks with many unknown words such as entity typing (Heinzerling & Strube, 2017) [[13](http://ruder.io/word-embeddings-2017/index.html#fn:13)], but have not been shown to be helpful yet for standard NLP tasks, where this is not a major concern. While they can be learned easily, it is difficult to see their advantage over character-based representations for most tasks (Vania & Lopez, 2017) [[50](http://ruder.io/word-embeddings-2017/index.html#fn:50)].

Another choice for using pre-trained embeddings that integrate character information is to leverage a state-of-the-art language model (Jozefowicz et al., 2016) [[7](http://ruder.io/word-embeddings-2017/index.html#fn:7)] trained on a large in-domain corpus, e.g. the 1 Billion Word Benchmark (a pre-trained Tensorflow model can be found [here](https://github.com/tensorflow/models/tree/master/research/lm_1b)). While language modelling has been found to be useful for different tasks as auxiliary objective (Rei, 2017) [[5](http://ruder.io/word-embeddings-2017/index.html#fn:5)], pre-trained language model embeddings have also been used to augment word embeddings (Peters et al., 2017) [[6](http://ruder.io/word-embeddings-2017/index.html#fn:6)]. As we start to better understand how to pre-train and initialize our models, pre-trained language model embeddings are poised to become more effective. They might even supersede word2vec as the go-to choice for initializing word embeddings by virtue of having become more expressive and easier to train due to better frameworks and more computational resources over the last years.

## OOV handling

One of the main problems of using pre-trained word embeddings is that they are unable to deal with out-of-vocabulary (OOV) words, i.e. words that have not been seen during training. Typically, such words are set to the UNK token and are assigned the same vector, which is an ineffective choice if the number of OOV words is large. Subword-level embeddings as discussed in the last section are one way to mitigate this issue. Another way, which is effective for reading comprehension (Dhingra et al., 2017) [[14](http://ruder.io/word-embeddings-2017/index.html#fn:14)] is to assign OOV words their pre-trained word embedding, if one is available.

Recently, different approaches have been proposed for generating embeddings for OOV words on-the-fly. Herbelot and Baroni (2017) [[15](http://ruder.io/word-embeddings-2017/index.html#fn:15)] initialize the embedding of OOV words as the sum of their context words and then rapidly refine only the OOV embedding with a high learning rate. Their approach is successful for a dataset that explicitly requires to model nonce words, but it is unclear if it can be scaled up to work reliably for more typical NLP tasks. Another interesting approach for generating OOV word embeddings is to train a character-based model to explicitly re-create pre-trained embeddings (Pinter et al., 2017) [[16](http://ruder.io/word-embeddings-2017/index.html#fn:16)]. This is particularly useful in low-resource scenarios, where a large corpus is inaccessible and only pre-trained embeddings are available.

##  Evaluation

Evaluation of pre-trained embeddings has been a contentious issue since their inception as the commonly used evaluation via word similarity or analogy datasets has been shown to only correlate weakly with downstream performance (Tsvetkov et al., 2015) [[21](http://ruder.io/word-embeddings-2017/index.html#fn:21)]. The [RepEval Workshop at ACL 2016](https://sites.google.com/site/repevalacl16/) exclusively focused on better ways to evaluate pre-trained embeddings. As it stands, the consensus seems to be that -- while pre-trained embeddings can be evaluated on intrinsic tasks such as word similarity for comparison against previous approaches -- the best way to evaluate them is extrinsic evaluation on downstream tasks.

## Multi-sense embeddings

A commonly cited criticism of word embeddings is that they are unable to capture polysemy. [A tutorial at ACL 2016](http://wwwusers.di.uniroma1.it/~collados/Slides_ACL16Tutorial_SemanticRepresentation.pdf) outlined the work in recent years that focused on learning separate embeddings for multiple senses of a word (Neelakantan et al., 2014; Iacobacci et al., 2015; Pilehvar & Collier, 2016) [[18](http://ruder.io/word-embeddings-2017/index.html#fn:18), [19](http://ruder.io/word-embeddings-2017/index.html#fn:19), [20](http://ruder.io/word-embeddings-2017/index.html#fn:20)]. However, most existing approaches for learning multi-sense embeddings solely evaluate on word similarity. Pilehvar et al. (2017) [[22](http://ruder.io/word-embeddings-2017/index.html#fn:22)] are one of the first to show results on topic categorization as a downstream task; while multi-sense embeddings outperform randomly initialized word embeddings in their experiments, they are outperformed by pre-trained word embeddings.

Given the stellar results Neural Machine Translation systems using word embeddings have achieved in recent years (Johnson et al., 2016) [[23](http://ruder.io/word-embeddings-2017/index.html#fn:23)], it seems that the current generation of models is expressive enough to contextualize and disambiguate words in context without having to rely on a dedicated disambiguation pipeline or multi-sense embeddings. However, we still need better ways to understand whether our models are actually able to sufficiently disambiguate words and how to improve this disambiguation behaviour if necessary.

## Beyond words as points

While we might not need separate embeddings for every sense of each word for good downstream performance, reducing each word to a point in a vector space is unarguably overly simplistic and causes us to miss out on nuances that might be useful for downstream tasks. An interesting direction is thus to employ other representations that are better able to capture these facets. Vilnis & McCallum (2015) [[24](http://ruder.io/word-embeddings-2017/index.html#fn:24)] propose to model each word as a probability distribution rather than a point vector, which allows us to represent probability mass and uncertainty across certain dimensions. Athiwaratkun & Wilson (2017) [[25](http://ruder.io/word-embeddings-2017/index.html#fn:25)] extend this approach to a multimodal distribution that allows to deal with polysemy, entailment, uncertainty, and enhances interpretability.

Rather than altering the representation, the embedding space can also be changed to better represent certain features. Nickel and Kiela (2017) [[52](http://ruder.io/word-embeddings-2017/index.html#fn:52)], for instance, embed words in a hyperbolic space, to learn hierarchical representations. Finding other ways to represent words that incorporate linguistic assumptions or better deal with the characteristics of downstream tasks is a compelling research direction.

## Phrases and multi-word expressions

In addition to not being able to capture multiple senses of words, word embeddings also fail to capture the meanings of phrases and multi-word expressions, which can be a function of the meaning of their constituent words, or have an entirely new meaning. Phrase embeddings have been proposed already in the original word2vec paper (Mikolov et al., 2013) [[37](http://ruder.io/word-embeddings-2017/index.html#fn:37)] and there has been consistent work on learning better compositional and non-compositional phrase embeddings (Yu & Dredze, 2015; Hashimoto & Tsuruoka, 2016) [[38](http://ruder.io/word-embeddings-2017/index.html#fn:38), [39](http://ruder.io/word-embeddings-2017/index.html#fn:39)]. However, similar to multi-sense embeddings, explicitly modelling phrases has so far not shown significant improvements on downstream tasks that would justify the additional complexity. Analogously, a better understanding of how phrases are modelled in neural networks would pave the way to methods that augment the capabilities of our models to capture compositionality and non-compositionality of expressions.

## Bias

Bias in our models is becoming a larger issue and we are only starting to understand its implications for training and evaluating our models. Even word embeddings trained on Google News articles exhibit female/male gender stereotypes to a disturbing extent (Bolukbasi et al., 2016) [[26](http://ruder.io/word-embeddings-2017/index.html#fn:26)]. Understanding what other biases word embeddings capture and finding better ways to remove theses biases will be key to developing fair algorithms for natural language processing.

## Temporal dimension

Words are a mirror of the zeitgeist and their meanings are subject to continuous change; current representations of words might differ substantially from the way these words where used in the past and will be used in the future. An interesting direction is thus to take into account the temporal dimension and the diachronic nature of words. This can allows us to reveal laws of semantic change (Hamilton et al., 2016; Bamler & Mandt, 2017; Dubossarsky et al., 2017) [[27](http://ruder.io/word-embeddings-2017/index.html#fn:27), [28](http://ruder.io/word-embeddings-2017/index.html#fn:28), [29](http://ruder.io/word-embeddings-2017/index.html#fn:29)], to model temporal word analogy or relatedness (Szymanski, 2017; Rosin et al., 2017) [[30](http://ruder.io/word-embeddings-2017/index.html#fn:30), [31](http://ruder.io/word-embeddings-2017/index.html#fn:31)], or to capture the dynamics of semantic relations (Kutuzov et al., 2017) [[31](http://ruder.io/word-embeddings-2017/index.html#fn:31)].

##  Lack of theoretical understanding

Besides the insight that word2vec with skip-gram negative sampling implicitly factorizes a PMI matrix (Levy & Goldberg, 2014) [[33](http://ruder.io/word-embeddings-2017/index.html#fn:33)], there has been comparatively little work on gaining a better theoretical understanding of the word embedding space and its properties, e.g. that summation captures analogy relations. Arora et al. (2016) [[34](http://ruder.io/word-embeddings-2017/index.html#fn:34)] propose a new generative model for word embeddings, which treats corpus generation as a random walk of a discourse vector and establishes some theoretical motivations regarding the analogy behaviour. Gittens et al. (2017) [[35](http://ruder.io/word-embeddings-2017/index.html#fn:35)] provide a more thorough theoretical justification of additive compositionality and show that skip-gram word vectors are optimal in an information-theoretic sense. Mimno & Thompson (2017) [[36](http://ruder.io/word-embeddings-2017/index.html#fn:36)] furthermore reveal an interesting relation between word embeddings and the embeddings of context words, i.e. that they are not evenly dispersed across the vector space, but occupy a narrow cone that is diametrically opposite to the context word embeddings. Despite these additional insights, our understanding regarding the location and properties of word embeddings is still lacking and more theoretical work is necessary.

##  Task and domain-specific embeddings

One of the major downsides of using pre-trained embeddings is that the news data used for training them is often very different from the data on which we would like to use them. In most cases, however, we do not have access to millions of unlabelled documents in our target domain that would allow for pre-training good embeddings from scratch. We would thus like to be able to adapt embeddings pre-trained on large news corpora, so that they capture the characteristics of our target domain, but still retain all relevant existing knowledge. Lu & Zheng (2017) [[40](http://ruder.io/word-embeddings-2017/index.html#fn:40)] proposed a regularized skip-gram model for learning such cross-domain embeddings. In the future, we will need even better ways to adapt pre-trained embeddings to new domains or to incorporate the knowledge from multiple relevant domains.

Rather than adapting to a new domain, we can also use existing knowledge encoded in semantic lexicons to augment pre-trained embeddings with information that is relevant for our task. An effective way to inject such relations into the embedding space is retro-fitting (Faruqui et al., 2015) [[41](http://ruder.io/word-embeddings-2017/index.html#fn:41)], which has been expanded to other resources such as ConceptNet (Speer et al., 2017) [[55](http://ruder.io/word-embeddings-2017/index.html#fn:55)] and extended with an intelligent selection of positive and negative examples (Mrkšić et al., 2017) [[42](http://ruder.io/word-embeddings-2017/index.html#fn:42)]. Injecting additional prior knowledge into word embeddings such as monotonicity (You et al., 2017) [[51](http://ruder.io/word-embeddings-2017/index.html#fn:51)], word similarity (Niebler et al., 2017) [[53](http://ruder.io/word-embeddings-2017/index.html#fn:53)], task-related grading or intensity, or logical relations is an important research direction that will allow to make our models more robust.

Word embeddings are useful for a wide variety of applications beyond NLP such as information retrieval, recommendation, and link prediction in knowledge bases, which all have their own task-specific approaches. Wu et al. (2017) [[54](http://ruder.io/word-embeddings-2017/index.html#fn:54)] propose a general-purpose model that is compatible with many of these applications and can serve as a strong baseline.

## Embeddings for multiple languages

As NLP models are being increasingly employed and evaluated on multiple languages, creating multilingual word embeddings is becoming a more important issue and has received increased interest over recent years. A promising direction is to develop methods that learn cross-lingual representations with as few parallel data as possible, so that they can be easily applied to learn representations even for low-resource languages. For a recent survey in this area, refer to Ruder et al. (2017) [[43](http://ruder.io/word-embeddings-2017/index.html#fn:43)].

## Embeddings based on other contexts

Word embeddings are typically learned only based on the window of surrounding context words. Levy & Goldberg (2014) [[44](http://ruder.io/word-embeddings-2017/index.html#fn:44)] have shown that dependency structures can be used as context to capture more syntactic word relations; Köhn (2015) [[45](http://ruder.io/word-embeddings-2017/index.html#fn:45)] finds that such dependency-based embeddings perform best for a particular multilingual evaluation method that clusters embeddings along different syntactic features.

Melamud et al. (2016) [[46](http://ruder.io/word-embeddings-2017/index.html#fn:46)] observe that different context types work well for different downstream tasks and that simple concatenation of word embeddings learned with different context types can yield further performance gains. Given the recent success of incorporating graph structures into neural models for different tasks as -- for instance -- exhibited by graph-convolutional neural networks (Bastings et al., 2017; Marcheggiani & Titov, 2017) [[47](http://ruder.io/word-embeddings-2017/index.html#fn:47), [48](http://ruder.io/word-embeddings-2017/index.html#fn:48)], we can conjecture that incorporating such structures for learning embeddings for downstream tasks may also be beneficial.

Besides selecting context words differently, additional context may also be used in other ways: Tissier et al. (2017) [[56](http://ruder.io/word-embeddings-2017/index.html#fn:56)] incorporate co-occurrence information from dictionary definitions into the negative sampling process to move related works closer together and prevent them from being used as negative samples. We can think of topical or relatedness information derived from other contexts such as article headlines or Wikipedia intro paragraphs that could similarly be used to make the representations more applicable to a particular downstream task.

## Conclusion

It is nice to see that as a community we are progressing from applying word embeddings to every possible problem to gaining a more principled, nuanced, and practical understanding of them. This post was meant to highlight some of the current trends and future directions for learning word embeddings that I found most compelling. I've undoubtedly failed to mention many other areas that are equally important and noteworthy. Please let me know in the comments below what I missed, where I made a mistake or misrepresented a method, or just which aspect of word embeddings you find particularly exciting or unexplored.

# Hacker News

Refer to the [discussion on Hacker News](https://news.ycombinator.com/item?id=15521957) for some more insights on word embeddings.

## Other blog posts on word embeddings

If you want to learn more about word embeddings, these other blog posts on word embeddings are also available:

- [On word embeddings - Part 1](http://sebastianruder.com/word-embeddings-1/index.html)
- [On word embeddings - Part 2: Approximating the softmax](http://sebastianruder.com/word-embeddings-softmax/index.html)
- [On word embeddings - Part 3: The secret ingredients of word2vec](http://sebastianruder.com/secret-word2vec/index.html)
- [Unofficial Part 4: A survey of cross-lingual embedding models](http://sebastianruder.com/cross-lingual-embeddings/index.html)

## References

1. Cavnar, W. B., Trenkle, J. M., & Mi, A. A. (1994). N-Gram-Based Text Categorization. Ann Arbor MI 48113.2, 161–175. https://doi.org/10.1.1.53.9367  [[↩](../_resources/13d6b80d558020fd3bc65cb2de284224.bin)](http://ruder.io/word-embeddings-2017/index.html#fnref:1)

2. Wieting, J., Bansal, M., Gimpel, K., & Livescu, K. (2016). Charagram: Embedding Words and Sentences via Character n-grams. Retrieved from http://arxiv.org/abs/1607.02789  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:2)

3. Bojanowski, P., Grave, E., Joulin, A., & Mikolov, T. (2017). Enriching Word Vectors with Subword Information. Transactions of the Association for Computational Linguistics. Retrieved from http://arxiv.org/abs/1607.04606  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:3)

4. Joulin, A., Grave, E., Bojanowski, P., & Mikolov, T. (2016). Bag of Tricks for Efficient Text Classification. arXiv Preprint arXiv:1607.01759. Retrieved from http://arxiv.org/abs/1607.01759  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:4)

5. Rei, M. (2017). Semi-supervised Multitask Learning for Sequence Labeling. In Proceedings of ACL 2017. [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:5)

6. Peters, M. E., Ammar, W., Bhagavatula, C., & Power, R. (2017). Semi-supervised sequence tagging with bidirectional language models. In Proceedings of the 55th Annual Meeting of the Association for Computational Linguistics (pp. 1756–1765). [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:6)

7. Jozefowicz, R., Vinyals, O., Schuster, M., Shazeer, N., & Wu, Y. (2016). Exploring the Limits of Language Modeling. arXiv Preprint arXiv:1602.02410. Retrieved from http://arxiv.org/abs/1602.02410  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:7)

8. Lample, G., Ballesteros, M., Subramanian, S., Kawakami, K., & Dyer, C. (2016). Neural Architectures for Named Entity Recognition. In NAACL-HLT 2016. [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:8)

9. Plank, B., Søgaard, A., & Goldberg, Y. (2016). Multilingual Part-of-Speech Tagging with Bidirectional Long Short-Term Memory Models and Auxiliary Loss. In Proceedings of the 54th Annual Meeting of the Association for Computational Linguistics. [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:9)

10. Yu, X., & Vu, N. T. (2017). Character Composition Model with Convolutional Neural Networks for Dependency Parsing on Morphologically Rich Languages. In Proceedings of the 55th Annual Meeting of the Association for Computational Linguistics (pp. 672–678). [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:10)

11. Kim, Y., Jernite, Y., Sontag, D., & Rush, A. M. (2016). Character-Aware Neural Language Models. AAAI. Retrieved from http://arxiv.org/abs/1508.06615  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:11)

12. Sennrich, R., Haddow, B., & Birch, A. (2016). Neural Machine Translation of Rare Words with Subword Units. In Proceedings of the 54th Annual Meeting of the Association for Computational Linguistics (ACL 2016). Retrieved from http://arxiv.org/abs/1508.07909  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:12)

13. Heinzerling, B., & Strube, M. (2017). BPEmb: Tokenization-free Pre-trained Subword Embeddings in 275 Languages. Retrieved from http://arxiv.org/abs/1710.02187  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:13)

14. Dhingra, B., Liu, H., Salakhutdinov, R., & Cohen, W. W. (2017). A Comparative Study of Word Embeddings for Reading Comprehension. arXiv preprint arXiv:1703.00993. [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:14)

15. Herbelot, A., & Baroni, M. (2017). High-risk learning: acquiring new word vectors from tiny data. In Proceedings of the 2017 Conference on Empirical Methods in Natural Language Processing. [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:15)

16. Pinter, Y., Guthrie, R., & Eisenstein, J. (2017). Mimicking Word Embeddings using Subword RNNs. In Proceedings of the 2017 Conference on Empirical Methods in Natural Language Processing. Retrieved from http://arxiv.org/abs/1707.06961  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:16)

17. Ballesteros, M., Dyer, C., & Smith, N. A. (2015). Improved Transition-Based Parsing by Modeling Characters instead of Words with LSTMs. In Proceedings of EMNLP 2015. https://doi.org/10.18653/v1/D15-1041  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:17)

18. Neelakantan, A., Shankar, J., Passos, A., & Mccallum, A. (2014). Efficient Non-parametric Estimation of Multiple Embeddings per Word in Vector Space. In Proceedings fo (pp. 1059–1069). [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:18)

19. Iacobacci, I., Pilehvar, M. T., & Navigli, R. (2015). SensEmbed: Learning Sense Embeddings for Word and Relational Similarity. In Proceedings of the 53rd Annual Meeting of the Association for Computational Linguistics and the 7th International Joint Conference on Natural Language Processing (pp. 95–105). [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:19)

20. Pilehvar, M. T., & Collier, N. (2016). De-Conflated Semantic Representations. In Proceedings of EMNLP. [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:20)

21. Tsvetkov, Y., Faruqui, M., Ling, W., Lample, G., & Dyer, C. (2015). Evaluation of Word Vector Representations by Subspace Alignment. Proceedings of the 2015 Conference on Empirical Methods in Natural Language Processing, Lisbon, Portugal, 17-21 September 2015, 2049–2054. [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:21)

22. Pilehvar, M. T., Camacho-Collados, J., Navigli, R., & Collier, N. (2017). Towards a Seamless Integration of Word Senses into Downstream NLP Applications. In Proceedings of the 55th Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers) (pp. 1857–1869). https://doi.org/10.18653/v1/P17-1170  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:22)

23. Johnson, M., Schuster, M., Le, Q. V, Krikun, M., Wu, Y., Chen, Z., … Dean, J. (2016). Google’s Multilingual Neural Machine Translation System: Enabling Zero-Shot Translation. arXiv Preprint arXiv:1611.0455. [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:23)

24. Vilnis, L., & McCallum, A. (2015). Word Representations via Gaussian Embedding. ICLR. Retrieved from http://arxiv.org/abs/1412.6623  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:24)

25. Athiwaratkun, B., & Wilson, A. G. (2017). Multimodal Word Distributions. In Proceedings of the 55th Annual Meeting of the Association for Computational Linguistics (ACL 2017). [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:25)

26. Bolukbasi, T., Chang, K.-W., Zou, J., Saligrama, V., & Kalai, A. (2016). Man is to Computer Programmer as Woman is to Homemaker? Debiasing Word Embeddings. In 30th Conference on Neural Information Processing Systems (NIPS 2016). Retrieved from http://arxiv.org/abs/1607.06520  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:26)

27. Hamilton, W. L., Leskovec, J., & Jurafsky, D. (2016). Diachronic Word Embeddings Reveal Statistical Laws of Semantic Change. In Proceedings of the 54th Annual Meeting of the Association for Computational Linguistics (pp. 1489–1501). [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:27)

28. Bamler, R., & Mandt, S. (2017). Dynamic Word Embeddings via Skip-Gram Filtering. In Proceedings of ICML 2017. Retrieved from http://arxiv.org/abs/1702.08359  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:28)

29. Dubossarsky, H., Grossman, E., & Weinshall, D. (2017). Outta Control: Laws of Semantic Change and Inherent Biases in Word Representation Models. In Conference on Empirical Methods in Natural Language Processing (pp. 1147–1156). Retrieved from http://aclweb.org/anthology/D17-1119  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:29)

30. Szymanski, T. (2017). Temporal Word Analogies : Identifying Lexical Replacement with Diachronic Word Embeddings. In Proceedings of the 55th Annual Meeting of the Association for Computational Linguistics (pp. 448–453). [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:30)

31. Rosin, G., Radinsky, K., & Adar, E. (2017). Learning Word Relatedness over Time. In Proceedings of the 2017 Conference on Empirical Methods in Natural Language Processing. Retrieved from https://arxiv.org/pdf/1707.08081.pdf  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:31)

32. Kutuzov, A., Velldal, E., & Øvrelid, L. (2017). Temporal dynamics of semantic relations in word embeddings: an application to predicting armed conflict participants. In Proceedings of the 2017 Conference on Empirical Methods in Natural Language Processing. Retrieved from http://arxiv.org/abs/1707.08660  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:32)

33. Levy, O., & Goldberg, Y. (2014). Neural Word Embedding as Implicit Matrix Factorization. Advances in Neural Information Processing Systems (NIPS), 2177–2185. Retrieved from http://papers.nips.cc/paper/5477-neural-word-embedding-as-implicit-matrix-factorization  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:33)

34. Arora, S., Li, Y., Liang, Y., Ma, T., & Risteski, A. (2016). A Latent Variable Model Approach to PMI-based Word Embeddings. TACL, 4, 385–399. Retrieved from https://transacl.org/ojs/index.php/tacl/article/viewFile/742/204  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:34)

35. Gittens, A., Achlioptas, D., & Mahoney, M. W. (2017). Skip-Gram – Zipf + Uniform = Vector Additivity. In Proceedings of the 55th Annual Meeting of the Association for Computational Linguistics (pp. 69–76). https://doi.org/10.18653/v1/P17-1007  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:35)

36. Mimno, D., & Thompson, L. (2017). The strange geometry of skip-gram with negative sampling. In Proceedings of the 2017 Conference on Empirical Methods in Natural Language Processing (pp. 2863–2868). [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:36)

37. Mikolov, T., Chen, K., Corrado, G., & Dean, J. (2013). Distributed Representations of Words and Phrases and their Compositionality. NIPS. [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:37)

38. Yu, M., & Dredze, M. (2015). Learning Composition Models for Phrase Embeddings. Transactions of the ACL, 3, 227–242. [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:38)

39. Hashimoto, K., & Tsuruoka, Y. (2016). Adaptive Joint Learning of Compositional and Non-Compositional Phrase Embeddings. ACL, 205–215. Retrieved from http://arxiv.org/abs/1603.06067  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:39)

40. Lu, W., & Zheng, V. W. (2017). A Simple Regularization-based Algorithm for Learning Cross-Domain Word Embeddings. In Proceedings of the 2017 Conference on Empirical Methods in Natural Language Processing (pp. 2888–2894). [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:40)

41. Faruqui, M., Dodge, J., Jauhar, S. K., Dyer, C., Hovy, E., & Smith, N. A. (2015). Retrofitting Word Vectors to Semantic Lexicons. In NAACL 2015. [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:41)

42. Mrkšić, N., Vulić, I., Séaghdha, D. Ó., Leviant, I., Reichart, R., Gašić, M., … Young, S. (2017). Semantic Specialisation of Distributional Word Vector Spaces using Monolingual and Cross-Lingual Constraints. TACL. Retrieved from http://arxiv.org/abs/1706.00374  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:42)

43. Ruder, S., Vulić, I., & Søgaard, A. (2017). A Survey of Cross-lingual Word Embedding Models Sebastian. arXiv preprint arXiv:1706.04902. Retrieved from http://arxiv.org/abs/1706.04902  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:43)

44. Levy, O., & Goldberg, Y. (2014). Dependency-Based Word Embeddings. Proceedings of the 52nd Annual Meeting of the Association for Computational Linguistics (Short Papers), 302–308. https://doi.org/10.3115/v1/P14-2050  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:44)

45. Köhn, A. (2015). What’s in an Embedding? Analyzing Word Embeddings through Multilingual Evaluation. Proceedings of the 2015 Conference on Empirical Methods in Natural Language Processing, Lisbon, Portugal, 17-21 September 2015, (2014), 2067–2073. [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:45)

46. Melamud, O., McClosky, D., Patwardhan, S., & Bansal, M. (2016). The Role of Context Types and Dimensionality in Learning Word Embeddings. In Proceedings of NAACL-HLT 2016 (pp. 1030–1040). Retrieved from http://arxiv.org/abs/1601.00893  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:46)

47. Bastings, J., Titov, I., Aziz, W., Marcheggiani, D., & Sima’an, K. (2017). Graph Convolutional Encoders for Syntax-aware Neural Machine Translation. In Proceedings of the 2017 Conference on Empirical Methods in Natural Language Processing. [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:47)

48. Marcheggiani, D., & Titov, I. (2017). Encoding Sentences with Graph Convolutional Networks for Semantic Role Labeling. In Proceedings of the 2017 Conference on Empirical Methods in Natural Language Processing. [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:48)

49. Mikolov, T., Corrado, G., Chen, K., & Dean, J. (2013). Efficient Estimation of Word Representations in Vector Space. Proceedings of the International Conference on Learning Representations (ICLR 2013), 1–12. [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:49)

50. Vania, C., & Lopez, A. (2017). From Characters to Words to in Between: Do We Capture Morphology? In Proceedings of the 55th Annual Meeting of the Association for Computational Linguistics (pp. 2016–2027). [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:50)

51. You, S., Ding, D., Canini, K., Pfeifer, J., & Gupta, M. (2017). Deep Lattice Networks and Partial Monotonic Functions. In 31st Conference on Neural Information Processing Systems (NIPS 2017). Retrieved from http://arxiv.org/abs/1709.06680  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:51)

52. Nickel, M., & Kiela, D. (2017). Poincaré Embeddings for Learning Hierarchical Representations. arXiv Preprint arXiv:1705.08039. Retrieved from http://arxiv.org/abs/1705.08039  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:52)

53. Niebler, T., Becker, M., Pölitz, C., & Hotho, A. (2017). Learning Semantic Relatedness From Human Feedback Using Metric Learning. In Proceedings of ISWC 2017. Retrieved from http://arxiv.org/abs/1705.07425  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:53)

54. Wu, L., Fisch, A., Chopra, S., Adams, K., Bordes, A., & Weston, J. (2017). StarSpace: Embed All The Things! arXiv preprint arXiv:1709.03856. [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:54)

55. Speer, R., Chin, J., & Havasi, C. (2017). ConceptNet 5.5: An Open Multilingual Graph of General Knowledge. In AAAI 31 (pp. 4444–4451). Retrieved from http://arxiv.org/abs/1612.03975  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:55)

56. Tissier, J., Gravier, C., & Habrard, A. (2017). Dict2Vec : Learning Word Embeddings using Lexical Dictionaries. In Proceedings of the 2017 Conference on Empirical Methods in Natural Language Processing. Retrieved from http://aclweb.org/anthology/D17-1024  [(L)](http://ruder.io/word-embeddings-2017/index.html#fnref:56)

Cover image credit: Hamilton et al. (2016)

 [← Multi-Task Learning Objectives for Natural Language Processing](http://ruder.io/multi-task-learning-nlp/index.html)

- [0 comments]()
- [**Blog**](https://disqus.com/home/forums/sebastianruder/)
- [(L)](https://disqus.com/embed/comments/?base=default&f=sebastianruder&t_u=http%3A%2F%2Fruder.io%2Fword-embeddings-2017%2F%3Futm_campaign%3DRevue%2520newsletter%26utm_medium%3DNewsletter%26utm_source%3DThe%2520Wild%2520Week%2520in%2520AI&t_d=Word%20embeddings%20in%202017%3A%20Trends%20and%20future%20directions&t_t=Word%20embeddings%20in%202017%3A%20Trends%20and%20future%20directions&s_o=default#)
- [](https://disqus.com/home/inbox/)
- [ Recommend  5](https://disqus.com/embed/comments/?base=default&f=sebastianruder&t_u=http%3A%2F%2Fruder.io%2Fword-embeddings-2017%2F%3Futm_campaign%3DRevue%2520newsletter%26utm_medium%3DNewsletter%26utm_source%3DThe%2520Wild%2520Week%2520in%2520AI&t_d=Word%20embeddings%20in%202017%3A%20Trends%20and%20future%20directions&t_t=Word%20embeddings%20in%202017%3A%20Trends%20and%20future%20directions&s_o=default#)
- [⤤  Share](https://disqus.com/embed/comments/?base=default&f=sebastianruder&t_u=http%3A%2F%2Fruder.io%2Fword-embeddings-2017%2F%3Futm_campaign%3DRevue%2520newsletter%26utm_medium%3DNewsletter%26utm_source%3DThe%2520Wild%2520Week%2520in%2520AI&t_d=Word%20embeddings%20in%202017%3A%20Trends%20and%20future%20directions&t_t=Word%20embeddings%20in%202017%3A%20Trends%20and%20future%20directions&s_o=default#)
- [Sort by Best](https://disqus.com/embed/comments/?base=default&f=sebastianruder&t_u=http%3A%2F%2Fruder.io%2Fword-embeddings-2017%2F%3Futm_campaign%3DRevue%2520newsletter%26utm_medium%3DNewsletter%26utm_source%3DThe%2520Wild%2520Week%2520in%2520AI&t_d=Word%20embeddings%20in%202017%3A%20Trends%20and%20future%20directions&t_t=Word%20embeddings%20in%202017%3A%20Trends%20and%20future%20directions&s_o=default#)

[![Avatar](../_resources/675fb4b91ca717db030507f2d84bcfdf.png)](https://disqus.com/by/disqus_Q5LZbDr3pW/)

Start the discussion…

- [Attach](https://disqus.com/embed/comments/?base=default&f=sebastianruder&t_u=http%3A%2F%2Fruder.io%2Fword-embeddings-2017%2F%3Futm_campaign%3DRevue%2520newsletter%26utm_medium%3DNewsletter%26utm_source%3DThe%2520Wild%2520Week%2520in%2520AI&t_d=Word%20embeddings%20in%202017%3A%20Trends%20and%20future%20directions&t_t=Word%20embeddings%20in%202017%3A%20Trends%20and%20future%20directions&s_o=default#)

Be the first to comment.

## Also on **Blog**

- [

### On word embeddings - Part 1

    - 28 comments •

    - 2 years ago

[Sebastian Ruder— No, word embeddings cannot directly replace languages in MT or other tasks, as …](http://disq.us/?url=http%3A%2F%2Fruder.io%2Fword-embeddings-1%2F&key=D_06nrmXbJ57HHR4mIQUTA)](http://disq.us/?url=http%3A%2F%2Fruder.io%2Fword-embeddings-1%2F&key=D_06nrmXbJ57HHR4mIQUTA)

- [

### Highlights of EMNLP 2016: Dialogue, deep learning, and more

    - 2 comments •

    - a year ago

[Sebastian Ruder—Hey Domyoung, glad you found it helpful. :)](http://disq.us/?url=http%3A%2F%2Fruder.io%2Femnlp-2016-highlights%2Findex.html&key=b22s94ITZlkCrrRBftptRQ)](http://disq.us/?url=http%3A%2F%2Fruder.io%2Femnlp-2016-highlights%2Findex.html&key=b22s94ITZlkCrrRBftptRQ)

- [

### About

    - 19 comments •

    - 2 years ago

[Sebastian Ruder—Wow, thanks a lot! :)](http://disq.us/?url=http%3A%2F%2Fruder.io%2Fabout%2F&key=9UoxqAvzY-z_cYI98wPMGQ)](http://disq.us/?url=http%3A%2F%2Fruder.io%2Fabout%2F&key=9UoxqAvzY-z_cYI98wPMGQ)

- [

### Transfer Learning - Machine Learning's Next Frontier

    - 11 comments •

    - 7 months ago

[Tim Martin— Very informative post! I would just offer one criticism on where you touch on psychology: "Toddlers …](http://disq.us/?url=http%3A%2F%2Fruder.io%2Ftransfer-learning%2F&key=Paj64VaYaekn7T5ORTA9jA)](http://disq.us/?url=http%3A%2F%2Fruder.io%2Ftransfer-learning%2F&key=Paj64VaYaekn7T5ORTA9jA)

- [Powered by Disqus](https://disqus.com/)
- [*✉*Subscribe*✔*](https://disqus.com/embed/comments/?base=default&f=sebastianruder&t_u=http%3A%2F%2Fruder.io%2Fword-embeddings-2017%2F%3Futm_campaign%3DRevue%2520newsletter%26utm_medium%3DNewsletter%26utm_source%3DThe%2520Wild%2520Week%2520in%2520AI&t_d=Word%20embeddings%20in%202017%3A%20Trends%20and%20future%20directions&t_t=Word%20embeddings%20in%202017%3A%20Trends%20and%20future%20directions&s_o=default#)
- [*d*Add Disqus to your site](https://publishers.disqus.com/engage?utm_source=sebastianruder&utm_medium=Disqus-Footer)
- [*🔒*Privacy](https://help.disqus.com/customer/portal/articles/466259-privacy-policy)