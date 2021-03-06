# [A Deep Reinforced Model for Abstractive Summarization](http://notesite.paperweekly.site/https://arxiv.org/pdf/1705.04304.pdf)

![](https://c1.sfdcstatic.com/content/dam/web/en_us/www/images/einstein/publications/tldr-image-3.png)

The input (reading) and output (generating) RNNs can be combined in a joint model where the the final hidden state of the input RNN is used as the initial hidden state of the output RNN. Combined in this way, the joint model is able to read any text and generate a different text from it. This framework is called an encoder-decoder RNN (or Seq2Seq) and is the basis of our summarization model.

![](https://c2.sfdcstatic.com/content/dam/web/en_us/www/images/einstein/publications/tldr-image-4.png)

To make our model outputs more coherent, we allow the decoder to look back at parts of the input document when generating a new word with a technique called temporal attention. Instead of relying entirely on its own hidden state, the decoder can incorporate contextual information about different parts of the input with an attention function. This attention is then modulated to ensure that the model uses different parts of the input when generating the output text, hence increasing information coverage of the summary.

![](https://c1.sfdcstatic.com/content/dam/web/en_us/www/images/einstein/publications/tldr-fig-6.png)

To train this model on real-world data like news articles, a common way is to use the <font style="color:red">teacher forcing algorithm</font>: a model generates a summary while using a reference summary, and the model is assigned a word-by-word error (or “local supervision”.)

![](https://c1.sfdcstatic.com/content/dam/web/en_us/www/images/einstein/publications/tldr-image-7.png)

What exactly is this scorer, and how does it tell if summaries are "good"? Since asking a human to manually evaluate millions of summaries is long and impractical at scale, we rely on an automated evaluation metric called ROUGE (Recall-Oriented Understudy for Gisting Evaluation).

To bring the best of both worlds, our model is trained with teacher forcing and reinforcement learning at the same time, being able to make use of both word-level and whole-summary-level supervision to make it more coherent and readable. In particular, we find that ROUGE-optimized RL helps improve recall (i.e all important information that needs to be summarized is indeed summarized) and word level learning supervision ensures good language flow, making the summary more coherent and readable.

## Improvements

1. Teacher forcing loss
  $$
  \mathcal{L}_{MLE} = -\sum_{t=0}^{T-1}\log p(y_t^*|y_1^*,\cdots, y_{T-1}^*)
  $$
2. Self-critical reinforcement learning (Borrow from "[Self-critical Sequence Training for Image Captioning](https://arxiv.org/abs/1612.00563)")
  $$
  \mathcal{L}_{RL}=-(r(\mathbf{y}^s))-r(\hat{\mathbf{y}}_{baseline})\sum_{t=0}^{T-1}\log p(y_t^s|y_1^s,\cdots, y_{T-1}^s)
  $$
  here baseline $\hat{\mathbf{y}}_{baseline}$ is obtained by maximizing the output probability distribution at each time step, essentially performing a greedy search.
## Weakness
1. Why they just use ROUGE-L?
2. Their final loss function is a combination of RL loss and MLE loss : $L_{mixed}=\gamma L_{RL}+l_{MLE}$, they did not clarify how to set the $\gamma$, learned or tried?
