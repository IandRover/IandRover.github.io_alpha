---
layout: post
title: "Two understanding of the Contrastive Divergence Algorithm."
tags: [ML]
---

##### I am still working on the problem regarding Latex. I provide this post [here](https://hackmd.io/@A8e-o-EGSGq0GvfIVrSFYw/SJW561DQK).

<ul id="toc"></ul>

---
Two understanding of the Contrastive Divergence Algorithm
===

## Introduction

Contrastive Divergence (CD) was proposed by Hinton and has been shown as a powerful algorithm to update the Restricted Boltzmann Machine (RBM).

The objective of RBM is to best reconstruct the input data from a dataset $V$ and at the same time to find a good representation of the input data from $V$.

### The structure of RBM

Basically, a RBM contains two layers: visible layer (with visible nodes) and hidden layer (with hidden nodes). RBM is parameterized with a weight matrix $W$ and two biases $a$ and $b$.
- ![](https://miro.medium.com/max/700/1*LoeBW9Stm6HjK57yBp45sQ.png)

Consider now we have a set of patterns to be remembered, for example 60k MNIST images.
The inference procedure is as follows:
- Given input $v_0$ from MNIST.
- Obtain the 0-1 encoded hidden representation $h_0$: $p(ℎ_0=1│𝑣_0)=𝜎(𝑏+𝑊^𝑇 𝑣_0)$
- Obtain the 0-1 encoded reconstruction $v_1$: $p(v_1=1│h_0)=𝜎(a+𝑊 h_0)$

The objective seems simple, but how can we effortless find a good $\{W, a, b\}$ to achieve so, especially without the gradient descent?

### The objective of RBM
Let us formally define the objective of RBM.
- The energy of RBM given $v$ and $h$: $𝐸(𝑣,ℎ)=−𝑎^𝑇 𝑣−𝑏^𝑇 ℎ−𝑣^𝑇 𝑊ℎ$.
    - Please note that since $h_0$ is sampled from $𝜎(𝑏+𝑊^𝑇 𝑣_1)$, $h_0$ is stochastic.
- The likelihood of $v_0$: $P(v_0) = \frac{1}{𝑍} ∑_ℎ 𝑒xp^{−𝐸(𝑣_0,h)}$ , where $Z=∑_{𝑣,ℎ} 𝑒xp^{−𝐸(𝑣,ℎ)}$
    - Note that the definition of $P(v_0)$ consider all possible 0-1 coded $h$, including $h=[0,...,0]$ or $h=[1,...,1]$, which is exponentially  many.
    - Note that the definition of $Z$ requires considering all possible 0-1 coded $v$ and $h$ pairs, which is also exponentially  many.
- The objective of RBM is to maximize the likelihood of all $v_0$ from a dataset $V$: $J(W,a,b)=argmax_{\{𝑊,a,b\}}⁡∑_{𝑣_0∈𝑉} log⁡(P(𝑣_0))$.
    - Also, we have $J(W,a,b)=argmax_{\{𝑊,a,b\}}⁡∑_{𝑣_0∈𝑉} log⁡(∑_ℎ 𝑒xp^{−𝐸(𝑣_0,h)})-log(Z)$
    - The illustration of the objective of RBM is provided here:
    - ![](https://i.imgur.com/Ib1x2Ep.png)


Clearly, the objective of RBM is not only to best maximize the energy of the seen data but to minimize the energy of the unseen data. For example, we expect that a RBM trained on MNIST to learn to specifically encode the digits, but not be able to encode some white noises.

### The derivation of Contrastive Divergence
To minimize $Z$ is not tractable. Hinton et al. proposed Contrastive Divergence (CD) to solve the problem. And since the birth of CD, there have been many modifications of the original CD method.
With CD-k, we update the parameters $W$ by:
- $\Delta W=v_0\cdot h_0^\top-v_{k}\cdot h_{k}^\top$, where $v_k$ and $h_k$ is generated by running the sampling steps as illustrated below:
- ![](https://www.researchgate.net/profile/Baptiste-Wicht/publication/307908790/figure/fig1/AS:404331623927809@1473411579320/Graphical-representation-of-the-Contrastive-Divergence-Algorithm-The-algorithm-CD-k.png)

CD has its distributional meaning defined using the K-L divergence. Please refer to Section 2.1 of paper [Contrastive Divergence Learning is a Time Reversal Adversarial Game (ICLR 2021)](https://openreview.net/pdf?id=MLSvqIHRidA).

In [On the Convergence Properties of Contrastive Divergence](http://proceedings.mlr.press/v9/sutskever10a/sutskever10a.pdf), Ilya Sutskever et all. showed that CD is not a gradient of any function due to some inconsistency. And some other papers also show that CD has multiple local minimums and therefore can impede learning. There have been some papers working to better understand the mechanism of CD, for example [Contrastive Divergence Learning is a Time Reversal Adversarial Game (ICLR 2021)](https://openreview.net/pdf?id=MLSvqIHRidA). I think there can be other explanations.

In this post, we want to propose two interesting viewpoints, though both of them are immature and incomplete.

## Understanding Contrastive Divergence (1).
Now we provide the more complete idea of the two. Before we start, please allow us to first review the logistic regression.
### A Review of Logistic Regression
Consider input $\{x_1, x_2, ..., x_n\}$ and one-hot target $\{y_1, y_2, ..., y_n\}$, we perform classification using a model parameterized by $\theta$:
$$
\begin{aligned}
f_\theta(x)=\sigma(\theta^T x)
\end{aligned}
$$
where $sigma$ once again is the sigmoid function.
The log likelihood if given by
$$
\begin{aligned}
L(\theta)=\sum_{i=1}^n y_i log(\sigma(\theta^T x_i)) + (1-y_i) log(1-\sigma(\theta^T x_i))
\end{aligned}
$$
The gradient of the log likelihood is the cross product between the error and the input.
$$
\begin{aligned}
\frac{\partial L(\theta)}{\partial \theta}
=\sum_{i=1}^n (y_i-\sigma(\theta^T x_i))x_i^T
\end{aligned}
$$
Below, we show that the components of the CD update of $W$ takes the form of the gradient in the Logistic regression.


### A k-step Contrastive Divergence Contains 2k Logistic Regression Tasks.
Looking at the CD update of $W$, we observe that
$$
\begin{aligned}
\Delta W
& =v_0\cdot h_0^\top-v_{k}\cdot h_{k}^\top \\
& =(v_0-v_1)\cdot h_0^\top + v_1\cdot (h_0-h1) + ... + v_k\cdot (h_{k-1}-h_k)^\top \\
\end{aligned}
$$
For each component $(v_0-v_1)\cdot h_0^\top$, we find that it has the same form of the gradient used in Logistic Regression as we just reviewed. That means for each term $(v_0-v_1)\cdot h_0^\top$, the corresponding loss considers the following task:
- $v_0$: the one-hot target
- $h_0$: the input
- $v_1$: the output
- $L(W)=v_0 log(v_1) + (1-v_0) log(1-h_0)$, where $v_1=\sigma(a+Wh_0)$

Here we provide the illustration:
![](https://i.imgur.com/P2qSwM9.png)

To wrap up, the CD algorithm can be considered as iteratively performing logistic regression tasks. The intuition behind is by decomposing the CD update where we can see that each component actually follows the update used in logistic regression.
By the way, the task itself can also be considered as a reconstruction of the target.


## Understanding Contrastive Divergence (2).

Here we provide another point of view which is more incomplete and immature. Nevertheless, it is still worth proving our intuition.

### The Energy Function in RBM is an Activation Function.

We can rewrite the the energy function of RBM given $v$ and $h$:
$$
\begin{aligned}
 𝐸(𝑣,ℎ)
 & =−𝑎^𝑇 𝑣−𝑏^𝑇 ℎ−𝑣^𝑇 𝑊ℎ \\
 & =−𝑎^𝑇 𝑣−(𝑏 +𝑊^T v)^\top ℎ \\
 & \sim −𝑎^𝑇 𝑣−(𝑏 +𝑊^T v)^\top \sigma(𝑏 +𝑊^T v) \\
 & = −𝑎^𝑇 𝑣−sum(SiLU(𝑏 +𝑊^T v))\\
\end{aligned}
$$
where SiLU stands for Sigmoid Linear Unit and is $SiLU(x)=x\cdot\sigma(x)$

Till here, we have an intuition that the objective of RBM is like maximizing the SiLU activation of $𝑏 +𝑊^T v$ when $v$ comes from dataset, and suppressing the SiLU activation of other inputs.

The derivative of SiLU is not easy to cope with for further analysis. We can here consider an asymptotic case of sigmoid:
$$
\begin{aligned}
 \sigma_\alpha(x) = \frac{1}{1+exp(-\alpha x)}
\end{aligned}
$$
It is clear that when $\alpha$ goes to infinity, sigmoid function becomes a [step function](https://en.wikipedia.org/wiki/Heaviside_step_function)
$$
\begin{aligned}
 step(x) = 1_{\{x>0\}}
\end{aligned}
$$
One property of this step function is that inferencing the hidden representation is no more stochastic: $h_0$: $p(ℎ_0=1│𝑣_0)=step(𝑏+𝑊^𝑇 𝑣_0)$. Also, the step function is the derivative of the ReLU function, meaning that we can get a modified energy function 𝐸(𝑣) (which depends on h):
$$
\begin{aligned}
 𝐸(𝑣)
 & =−𝑎^𝑇 𝑣−(𝑏 +𝑊^T v)^\top step(𝑏 +𝑊^T v) \\
 & =−𝑎^𝑇 𝑣−sum(ReLU(𝑏 +𝑊^T v))\\
\end{aligned}
$$

## The Reconstruction of $v_0$ is the Gradient w.r.t. $v_0$ .
Now something cool happens.
We can calculate the derivative of the energy function $𝐸(𝑣_0)$ w.r.t the input $v_0$.
$$
\begin{aligned}
\frac{\partial E(v_0)}{\partial v_0}
& = - \frac{\partial 𝑎^𝑇 𝑣_0+sum(ReLU(𝑏 +𝑊^T v_0))}{\partial v_0} \\
& = - (a + W step(b+W^\top v_0)) \\
& = - (a + W h_0)
\end{aligned}
$$
Surprisingly, we can discover that it is the preactivation of the reconstruction $v_1$. This means that we can express $v_1$ in two manners:
$$
\begin{aligned}
& v_1 = step(a+W h_0)\\
& v_1 = step(- \frac{\partial E(v_0)}{\partial v_0} )\\
\end{aligned}
$$

What does this tell us?
Recall that we are to minimize the energy (in order to maximize the likelihood), the negative gradient of the energy function (i.e. $- \frac{\partial E(v_0)}{\partial v_0}$) actually entails that direction to lower energy (theoretically). The step function helps normalize the range into 0-1. Also note that, if the value of the gradient is large enough, we can approximately say that:
$$
\begin{aligned}
& v_1 = step(v_0 - \frac{\partial E(v_0)}{\partial v_0} ) \end{aligned}
$$
, when the norm of $v_0$ is much smaller than the norm of $\frac{\partial E(v_0)}{\partial v_0}$.

But now, I cannot combine the concept that "the reconstruction is the gradient" to the CD update of $W$. Because I do not know how to explain the update from this point of view.

## Final Words

Thank you for taking your time.
Generally speaking, I love the mysterious CD algorithm and I still think there can be some other explanation to the CD algorithm. The two points of view provided above are both simply and straight-forward. Hope that this post can give you some inspirations.