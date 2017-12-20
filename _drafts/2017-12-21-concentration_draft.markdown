---
layout: post
title: "Concentration Inequalities and GANs"
date:   2017-12-19 20:00:00 -0500 
categories: jekyll update
comments: true
description: concentration inequalities (Markov inequality, McDiarmid's inequality, Azuma-Hoeffding inequality) and proof of poor convergence in Bi-directional (encoder-decoder) generative adversarial networks (GAN).
author: Lei M. Zhang
---

In this post, I'd like to highlight a part of one of my favorite recent papers simply because it seems like a breeze of fresh (read: theoretical) air into my daily arXiv review. In [Theoretical Limitations of Encoder-Decoder GAN Architectures](https://arxiv.org/abs/1711.02651) by Sanjeev Arora, Andrej Ristenski, and Yi Zhang (Nov. 2017), the authors show an interesting negative result regarding [Encoder-Decoder GANs](https://arxiv.org/abs/1605.09782).

My personal interest in this paper was peaked when I realized that the main technique that the paper uses to prove their key theorem is **concentration inequalities**! In particular, the technique involves an expression (e.g., the BiGAN cost function) that contains an expectation  of some random variable, which makes a particular statement (e.g., the cost is very small in a minimizing optimization). It is then proved, using concentration inequalities, that realizations of the random variable is very close to the expectation (as some parameter goes to infinity), essentially making the statement **true for realizations** of the random variable.

Concentration inequalities played an important role in my PhD work on error-correcting codes. In [Spatially-coupled Split-component Codes with Iterative Algebraic Decoding](http://ieeexplore.ieee.org/document/8082562/) the main result on the threshold channel parameter of a class of codes was determined by calculating the evolution of the **expected** bit-error probability throughout the decoding process. How could I justify this expected performance will actually be true in practice with random channel noise (and randomly sampled codes from some ensemble)? Concentration inequalities! In particular, the Azuma-Hoeffding inequality and a very useful variant of them called [Wormald's Theorem](https://projecteuclid.org/euclid.aoap/1177004612), also known as the Differential Equations Method (DEM).

Now, let's see how concentration inequalities are used in Deep Learning. The goal of the following sections is not to describe the result or its proof in every detail. My intention is to illustrate the particular use of concentration inequalities and application of the general technique to an interesting problem.

## Encoder-Decoder (Bi-directional) GANs

Let $$\mathbb{R}^d$$ be the **data space**, from which samples $$x$$ of a given data set is obtained. This could be the pixel space, for example. Let $$\mathbb{R}^{d'}$$ be the **code space**, where elements $$z$$ of a latent code is obtained. 

An encoder-decoder GAN architecture then simply aims to find maps $$E : \mathbb{R}^d \mapsto \mathbb{R}^{d'}$$ and $$G : \mathbb{R}^{d'} \mapsto \mathbb{R}^d$$, referred to as the encoder and decoder (or generator). Of course, there ae many ways of finding them.

Generative adversarial networks (GANs) is one way of finding $$E(x)$$ and $$G(z)$$ that does not involve explicit probability calculations. To use GAN in this case, a discriminator network $$D : \mathbb{R}^{d} \times \mathbb{R}^{d'} \mapsto \mathbb{R}$$ is used to distinguish the distributions of the pairs $$(x, E(x))$$ and $$(G(z), z)$$. If the generative network has learned the joint distribution over data and code spaces $$p(x,z)$$, then $$D$$ should not be able to distinguish between smaples $$(x, E(x))$$ (i.e., data sample $$x$$ and its codeword) and $$(G(z), z)$$ (i.e., generated sample and its codeword).

The objective function is then

$$
\begin{equation*}
\min_{G, E} \max_D \left| \mathbb{E}_{x\sim\mu} \phi( D(x, E(x)) ) - \mathbb{E}_{z\sim\nu} \phi( D(G(z), z)) ) \right|,
\end{equation*}
$$

where ...
