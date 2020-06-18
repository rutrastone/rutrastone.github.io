---
layout: default
---

# Attention Study

[Htut, et al. (2019)](https://arxiv.org/pdf/1911.12246.pdf) find that Masked Language Models' (BERT, ROBERTA) attention heads do not reliably decode UD dependency trees in English. For a given layer and attention head, Htut, et al. (2019) experiment with two strategies for extracting trees: *MAX* and *MST*. With *MAX*, they take the maximum incoming attention score for every word. For an $ N \times N $ matrix, where $ N $ is the sentence length, this amounts to taking the $ \arg \max $ over every row. With *MST*, they run the Chu-Liu-Edmonds algorithm over the matrix in order to extract a maximum spanning tree. 

![tree_results](images/uuas_total.png)
