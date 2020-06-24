---
layout: default
---

# Attention Study

## Tree Decoding

[Htut, et al. (2019)](https://arxiv.org/pdf/1911.12246.pdf) find that Masked Language Models' (BERT, ROBERTA) attention heads do not reliably encode UD dependency trees in English. For a given layer and attention head, Htut, et al. (2019) experiment with two strategies for extracting trees: *MAX* and *MST*. With *MAX*, they take the maximum incoming attention score for every word. For an $ N \times N $ attention matrix, where $ N $ is the sentence length, this amounts to taking the $ \arg \max $ over every row. With *MST*, they run the Chu-Liu-Edmonds algorithm over the matrix in order to extract a maximum spanning tree. With both approaces, Htut, et al. (2019) report Unlabeled Undirected Attachment Scores (UUAS) of approximately 40%, which is slightly better than the left-to-right positional baseline. Might we expect better accuracy if we examine other languages? Here, we replicate the experiments of Htut, et al. (2019) over the 19 languages represented in the Parallel UD treebank. In addition to their *MAX* and *MST* methods, we also consider *JS*, where we calculate the Jensen-Shannon similarity (1 - JS divergence) across all pairs of incoming attention vectors and take the argmax per word. We overlay the best performing layer-head combination on each bar in the figure below. Out of the tree approaches, *MST* works best. Layer 2, head 3 tends to provide the best decoding accuracy using *MST* for the majority of languages.

![tree_results](images/uuas_total.png)

## Accuracy per Relation

Perhaps the most interesting finding of Htut, et al. (2019) is that some heads *specialize* in tracking specific dependency relations. Included among those relations are *nsubj*, *obj*, *advmod*, and *amod*, which, unlike the full trees, are decoded with higher accuracy than the position baselines. We investigate the extent to which this persists across the remaining 18 PUD languages below. 

![rel_mst](images/attention_mst.max.png)
![rel_max](images/attention_max.max.png)
![rel_js](images/attention_js.max.png)

Again, the best-performing method is *MST* and the 2-3 and 9-5 layer-head combinations are observed across many languages and relations. The easiest-to-decode relations appear to be *amod*, *aux*, *case*, and *det*. These, however, are typically easy relations where dependents often appear within one word of their heads in linear order. Below, we plot the maximum UUAS (as returned by any layer/head via *MST*) as a function of the average linear distance between words involved in a set of dependencies. 

![linear_distance_score](images/linear_distance_max_corr.png)

Here, our suspicions are confirmed, as words in the easily-decodable relations appear to be, on average, very close to each other. Other relations appear to follow a predictable trend in decreasing in decoding accuracy as the distance in words becomes longer. 

## Confidence

To characterize the behavior of particular attention heads, we might also consider their *confidence*. [Voita, et al. (2019)](https://www.aclweb.org/anthology/P19-1580.pdf) define the *confidence* of a head as "*the average of its maximum attention weight excluding the end of sentence symbol, where the average is taken over tokens in a set of sentences. A confident head is one that usually assigns a high proportion of its attention to a single token.*" Below, we plot the *confidence* every layer and head per language. 

![confidence](images/confidence.png)

Here, we tend to see the same patterns repeated across languages. Also, layer-head 2-3 appears to be consistently confident, while 9-5 does not. To visualize how decoding accuary might interact with confident, we plot the UUAS of each layer-head combination as a function of its confidence below. 

![confidence_score](images/confidence_corr.png)

There appears to be a clear relationship between decoding accuracy and head confidence. It is possible to visualize layer-head 2-3 clustered in the upper-right corner of the *MST* facet. 

## Tree Visualization

It might be an interesting qualitative experiment to visualize a dependency tree decoded by our methods. Below, we illustrate several trees returned by our *MST* method. 

### Head 2, Layer 3

![2_3_dog](images/2-3_dog.png)
![2_3_ball](images/2-3_ball.png)

For both English and Russian sentences (the latter meaning *I threw him the ball*), this head appears to be a simple adjacency tracker. 

### Head 9, Layer 5

![9_5_dog](images/9-5_dog.png)
![9_5_ball](images/9-5_ball.png)

Similarly to 2-3, this head largely tracks adjacency, save for the arc drawn between 'dog' and 'the'.

### Head 12, Layer 9

![12_9_dog](images/12-9_dog.png)
![12_9_ball](images/12-9_ball.png)

This head appears to encode the syntactic structure somewhat accurately, if we ignore directionality (we aren't measuring UAS) and some idiosyncracies. For example, in the English sentence, arcs from "the", "angry", "chased" are correctly connected to "dog". Strangely, "from" is also connected to "the" and the third "the" to the second "the". The Russian structure is captured correctly (again, ignoring directionality**. 

## XLM-RoBERTa

**mBERT** is not the only multilingual model on the market. **XLM-RoBERTa** is a similar, transformer-based model, with *base* (12 layers, 12 heads) and *large* (24 layers, 16 heads) versions. How do these models stack up to mBERT when decoding dependency trees from attention matrices? Might we expect better UUAS for the *large* model? Below, we plot each model's accuracy per language, using the *MST* method. 

![uuas_per_model](images/uuas_total_per_model.png)

There doesn't appear to be a huge difference here, though the *base* XLM model tends to decode trees slightly better than mBERT, with 9-2, 2-9, and 11-6 as the top-scoring combinations. Surprisingly, the *large* model performs worse than this in most cases. 

Since we observed strong relationship between head confidence and UUAS, it might be interesting to see if this persists across models. 

![confidence_corr_model](images/confidence_corr_model.png)

Here, we observe similar distributions across models, though the mBERT facet still appears to depict the tighest fit. How do the confidence matrices of both XLM models look like? 

### XLM-R-B

![confidence_xlm_b](images/confidence_xlm_b.png)

### XLM-R-L

![confidence_xlm_l](images/confidence_xlm_l.png)

Interestingly, both XLM models feature less confident heads than mBERT does. Another view of the relationship between head probability distributions and UUAS might be offered by looking at the average entropy of a head, as opposed to simply taking the average max probability. Here, we might expect realy confident heads to likewise have a low entropy, in that high probabilities are assigned to single words. Conversely, a head that assigns low probabilities to all words will likely yield a high entropy, which, per information theory, dictates that it is "uncertain". Below, we plot confidence as a function of mean entropy. 

![entropy_conf](entropy_conf_corr_model.png)

The best fit again appears to be for mBERT, which signifies that entropy increases when a head is less confident. Interestingly, the *large* XLM model does not depict a strong relationship between confidence and entropy, and tends to place the vast majority of its probability density below 0.25. 

Now we might ask, what does the relationship between entropy and UUAS look like?

![entropy_corr](entropy_corr_model.png)

Expectedly, we see a strong negative relationship between UUAS and entropy. This trend is almost identical for mBERT and XLM *base*, with similar distributions of layers. Again, we do not see a strong trend for the *large* model. 

To dive even deeper, we can look at the exact entropy returned by the layer/head combinations of each model. 

### mBERT

![entropy_mbert](images/entropy_mbert.png)

### XLM-R-B

![entropy_xlm_b](images/entropy_xlm_b.png)

### XLM-R-L

![entropy_xlm_l](images/entropy_xlm_l.png)
