# [The Concrete Distribution: a Continuous Relaxation of Discrete Random Variables](https://arxiv.org/abs/1611.00712)

This paper presents a way to differentiate through discrete random variables by replacing them with continuous random variables. Say you have a discrete [categorical variable][cat] and you're sampling it with the [Gumbel trick][gumbel] like this ($G_k$ is a Gumbel distributed variable and $\mathbf{\alpha}/\sum_k \alpha_k$ are our categorical probabilities):

$$
z = \text{onehot} ({\text{argmax}}_k[G_k+\log \alpha_k])
$$

This paper replaces the one hot and argmax with a softmax, and they add a $\lambda$ variable to control the "temperature". As $\lambda$ tends to zero it will equal the above equation.

$$
z = \text{softmax} \left( \frac{  G_k + \log \alpha_k }{\lambda} \right)
$$

I made [some notes][nb] on how this process works, if you'd like more intuition.

Comparison to [Gumbel-softmax][gs]
--------------------------------------------

These papers are proposed precisely the same distribution with notation changes ([noted there][gs]). Both papers also reference each other and the differences. Although, they mention differences in the variatonal objectives to the Gumbel-softmax. This paper also compares to [VIMCO][], which is probably a harder benchmark to compare against (multi-sample versus single sample).

The results in both papers compare to SOTA score function based estimators and both report high scoring results (often the best). There are some details about implementations to consider though, such as scheduling and exactly how to define the variational objective.

[cat]: https://en.wikipedia.org/wiki/Categorical_distribution
[gumbel]: https://hips.seas.harvard.edu/blog/2013/04/06/the-gumbel-max-trick-for-discrete-distributions/
[gs]: http://www.shortscience.org/paper?bibtexKey=journals/corr/JangGP16
[nb]: https://gist.github.com/gujiuxiang/acf30fe639364aa97d9148cd50777b71
[vimco]: https://arxiv.org/abs/1602.06725

The Concrete Distribution: A Continuous Relaxation of Discrete Random Variables
by Chris J. Maddison, Andriy Mnih, Yee Whye Teh
ArXiv, 2016

Categorical Reparameterization with Gumbel-Softmax
by Eric Jang, Shixiang Gu, Ben Poole
ArXiv, 2016

The two papers present a method for the estimation of gradients in a computation graph with discrete stochastic nodes. The method is based on a combination of two established tricks: <font style="color:red">the reparametrization trick (in which stochastic variables are rewritten as a deterministic function of some (learned) parameters and a known fixed noise distribution)</font> and <font style="color:red">the Gumbel-Max trick (in which a sample from a discrete distribution is produced by adding Gumbel-distributed noise to a logit observation and taking the argmax)</font>.

In particular, using the Gumbel-Max trick, we can refactor the sampling of a discrete random variable into a deterministic function of some parameters and the Gumbel distribution. By doing so, we get around the fact that simple back propagation cannot be applied in the case of a stochastic computation graph with discrete random variables, as the function defined by the graph is not differentiable. The trick allows a low-variance but biased estimator of the gradient to be obtained via Monte Carlo sampling.


The estimator performs on par with previously established methods in tasks of density estimation. However, neither paper presents results for datasets more complex than MNIST. The reason for this is not made clear. An immediate extension of these works therefore is to consider more complex data that deal with discrete random variables. For example, the Gumbel-Softmax trick could be applied in the case of sequences such as language data.

s for the difference between the papers, those are slight: Jang et al. use a vector of Gumbels passed through a softmax activation, instead of the Gumbel density, in defining a variational lower bound, which results in a looser bound. Maddison et al. compared with a stronger baseline and found the Concrete / Gumbel-Softmax estimator to perform on par with other estimators on average (whereas Jang et al. find it performs slightly better, in terms of both accuracy and efficiency, than a weaker baseline). Lastly, Jang et al. describe the Straight-Through (ST) Gumbel Estimator which allows discrete samples (useful for robotics tasks) with a continuous backward pass.

A final point about both pieces of work is that there is a bias-variance tradeoff in the choice of the temperature parameter: with a small temperature, samples are close to a Categorical variable but the variance of the estimates of the gradients are large, and with large temperatures, samples are smooth (non-Categorical) but the variance of the gradients is small. The papers keep the temperature fixed and thus do not fully investigate how performance varies as a function of this parameter. However, they do suggest an annealing schedule such that the Gumbel-Softmax nodes behave more and more like Categorical random variables further on in training.
