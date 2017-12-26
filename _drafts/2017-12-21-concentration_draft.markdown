---
layout: post
title: "Concentration Inequalities and GAN Theory"
date:   2017-12-26 10:00:00 -0500 
categories: jekyll update
comments: true
description: concentration inequalities (Markov inequality, McDiarmid's inequality, Azuma-Hoeffding inequality) and proof of poor convergence in Bi-directional (encoder-decoder) generative adversarial networks (GAN).
author: Lei M. Zhang
---

In this post, I'd like to highlight one of my favorite new papers. In [Theoretical Limitations of Encoder-Decoder GAN Architectures](https://arxiv.org/abs/1711.02651) by Sanjeev Arora, Andrej Ristenski, and Yi Zhang (Nov 2017), the authors show an interesting **negative** result regarding [Encoder-Decoder GANs](https://arxiv.org/abs/1605.09782). 

The key result of the paper is that an Encoder-Decoder (or Bi-directional) generative adversarial network (BiGAN) can have arbitrarily low training cost (i.e., appear arbitrarily good during training) but also be essentially useless (see [Theorem 1](https://arxiv.org/abs/1711.02651)). Useless here means that the encoder merely extracts noise from an input sample, while the generator has learned a distribution with small support (i.e., mode collapse). 

It is worth to mention that in the proof of the main theorem, the authors used the techniques of defining probabilistic ensembles and applying concentration inequalities to their expectations and realizations.

## Ensembles and Concentration 

These techniques are often used when a statement is difficult to prove for particular instances of a problem, but easy in aggregate. For this case, there are many possible different encoder and generator instances and their combinations.

The key idea is to consider all encoder/generator pairs together, i.e., to define an ensemble of encoders and generators using probability distributions. Their aggregate properties, such as an expectation with respect to an input (data) distribution, can then be found.

Once the expectation is found, it would be useful to know how far from the expected value a **sample** of the ensemble will be. A **concentration inequality** essentially provides a guarantee for the closeness of a sample of the ensemble to the expected value of the ensemble.

The Law of Large Numbers is a well-known expression of the idea of concentration. It is guaranteed, in the weak sense, by Tchebychev's inequality.

## Concentration Inequalities in Coding Theory

These techniques are well-known and plays important roles in information theory and the theory of error-correcting codes. For example, they were used in [Spatially-coupled Split-component Codes with Iterative Algebraic Decoding](http://ieeexplore.ieee.org/document/8082562/), to derive a method of analyzing the performance of the class of Spatially-coupled Split-component (SCSC) codes. They also played an instrumental part in the theory of [Low-Density Parity-Check](http://ieeexplore.ieee.org/abstract/document/910575/) [(LDPC) Codes](http://ieeexplore.ieee.org/abstract/document/910577/).

In the SCSC work, such codes are too numerous and too large (in terms of number of bits per codeword) to study individually. In order to determine the channel noise-level at which decoding success or fails, called the **threshold** noise level, the following steps were required:

1. Define a graphical model for the class of SCSC codes
2. Define an ensemble of graphs which models all possible SCSC codes with given parameters
3. Calculate the threshold noise-level for the **expected** decoder output bit-error probability (BER)
4. Use concentration inequalities to prove that, as the length of codewords tends to infinity, any sample from the graph would have decoder output BER similar to the expected BER calculated in step 3.

Note that the expectation is taken over the random graph ensemble and channel noise samples. Calculating the probability of error for all codes and channel noise samples would be impractical, as in the case of BiGANS. 

We used the concentration inequality known as [Wormald's Theorem](https://projecteuclid.org/euclid.aoap/1177004612) was used. It is quite interesting as the method first finds a system of differential equations describing the evolution of the expected graph throughout a random edge-deletion process. The concentration guarantee is given for the solution of the differential equations. Hence, Wormald's theorem is also known as the [Differential Equations Method](https://www.cs.upc.edu/~diaz/papersd/DEM.pdf). For more details of the proof and technique, please refer to Chapter 4 of [Analysis and Design of Staircase Codes for High Bit-Rate
Fibre-Optic Communication](http://hdl.handle.net/1807/79549).

Let's now see a glimpse of the application of concentration inequalities for BiGANs.

## Encoder-Decoder (Bi-directional) GANs

Let $$\mathbb{R}^d$$ be the **data space**, from which samples $$x$$ of a given data set is obtained. This could be the pixel space, for example. Let $$\mathbb{R}^{d'}$$ be the **code space**, where elements $$z$$ of a latent code is obtained. 

An encoder-decoder GAN architecture then simply aims to find maps $$E : \mathbb{R}^d \mapsto \mathbb{R}^{d'}$$ and $$G : \mathbb{R}^{d'} \mapsto \mathbb{R}^d$$, referred to as the encoder and decoder (or generator). Of course, there are many ways of finding them.

Generative adversarial networks (GANs) is one way of finding $$E(x)$$ and $$G(z)$$ that does not involve explicit probability calculations. To use GAN in this case, a discriminator network $$D : \mathbb{R}^{d} \times \mathbb{R}^{d'} \mapsto \mathbb{R}$$ is used to distinguish the distributions of the pairs $$(x, E(x))$$ and $$(G(z), z)$$. If the generative network has learned the joint distribution over data and code spaces $$p(x,z)$$, then $$D$$ should not be able to distinguish between samples $$(x, E(x))$$ (i.e., data sample $$x$$ and its codeword) and $$(G(z), z)$$ (i.e., generated sample and its codeword).

The objective function is then

$$
\begin{equation*}
\min_{G, E} \max_D \left| \mathbb{E}_{x\sim\mu} \phi( D(x, E(x)) ) - \mathbb{E}_{z\sim\nu} \phi( D(G(z), z)) ) \right|,
\end{equation*}
$$

where $$\mu$$ and $$\nu$$ are data and code distributions, $$\phi$$ is some concave function (such as log), assumed to have outputs in the range $$[-\Delta,\Delta]$$ for some $$\Delta \geq 1$$ and is $$L_{\phi}$$-[Lipschitz](https://en.wikipedia.org/wiki/Lipschitz_continuity) ($$L_{\phi}$$ is a constant. This is a continuity requirement from the concentration inequality). 

### The main theorem

For $$G$$ of support $$m \triangleq p\Delta^2\log(p\Delta L L_{\phi} / \epsilon)^2 / \epsilon^2 $$, $$E$$ with at most $$d'$$ non-zero weights, and all discriminators $$D$$ that are $$L$$-Lipschitz ($$L$$, like $$L_{\phi}$$, is a constant) and has $$p$$ parameters, with probability $$1 - \exp(-\Omega(p\log(\Delta/\epsilon)))$$:

$$
\begin{equation}\label{eqn:theorem1}
\left| \mathbb{E}_{x\sim\mu}\phi(D(x,E(x))) - \mathbb{E}_{z\sim\nu}\phi(D(G(z),z)) \right| \leq \epsilon.
\end{equation}
$$ 

In the following, we only focus on the concentration proof for the generator. The authors also construct the encoder ensemble in an interesting way that is and well-worth taking a look.

## Concentration of Good Generators

### Definitions and Notation

The technical theorem requires a brief introduction of new notation and concepts. Most of the notation is from the original paper, with minor simplifications.

The idea here is to first build a probabilistic model for $$G$$, i.e., to consider every $$G$$ of small support to be sampled from an ensemble according to some probability distribution. 

To do so, first partition $$\mathbb{R}^{d'}$$ into $$m$$ blocks with equal probability as given by the code distribution $$\nu$$. Then, sample $$m$$ samples $$x_1^*, x_2^*,\dots,x_m^*$$ from the data distribution. The randomness in choosing these samples from the data space is what allows $$G$$ to be a random variable.

Given $$z$$ from the code space, let $$i(z) \in \{1,\dots, m\}$$ be the index of the partition in which $$z$$ lies. The generator is defined as $$G(z) \triangleq x_{i(z)}^* \otimes z$$ where $$\otimes$$ is some masking function. In other words, the generator simply selects the sampled image associated with the partition for $$z$$ and masks it. 

Finally, we call $$z_i$$ and $$z_j$$ non-colliding if $$i(z_i) \neq i(z_j)$$. A set $$T = \{z_1,\dots,z_m\}$$ is non-colliding if all pairs in the set are non-colliding. Let $$\mathcal{T}$$ denote the distribution over non-colliding sets, such that each $$z_i$$ is sampled independently from the conditional distribution of $$\nu$$ inside the $$i$$-th partition.

### Many Expectations

Recall that property that we want to show is for ($$\ref{eqn:theorem1}$$) to be arbitrarily small. For the generator, we are then interested in evaluating 

$$
\begin{equation}\label{eqn:expectation}
\mathbb{E}_{z\sim\nu}\phi(D(G(z),z)).
\end{equation}
$$

Note that since $$G$$ is now a random variable, this expectation is also a random variable.

Now, the key expectation here is 

$$
\begin{equation*}
\mathbb{E}_G \mathbb{E}_{z\sim\nu} \phi( D( G(z), z) ),
\end{equation*}
$$

which is shown in the paper to be equal to ($$\ref{eqn:expectation}$$). The task remaining is to show the concentration of ($$\ref{eqn:expectation}$$) to its expectation under the distribution of $$G$$.

Note that there are two sources of randomness, namely $$\nu$$, the distribtuion over $$z$$, and some distribution over $$G$$. This is similar to the case analyzed in my PhD work, where the two sources of random are randomly graphical models of SCSC codes and channel noise samples.

### Concentration

Using the idea of non-colliding sets and their distribution $$\mathcal{T}$$, the paper shows that ($$\ref{eqn:expectation}$$) is equivalent to

$$
\begin{equation*}
\mathbb{E}_{T \sim \mathcal{T}} \mathbb{E}_{z \sim T} \phi ( D(G(z), z)),
\end{equation*}
$$

where $$\mathbb{E}_{z \sim T}$$ is sample mean over the elements in $$T$$. 


This trick essentially provides an easier way to handle the randomness in $$z$$, since fixing $$T$$ means samples $$z_i$$ are non-colliding, independent, and has equal probability under $$\nu$$.

The authors then consider $$D$$ to be fixed and note that the random variables in the function 

$$
\begin{equation*}
\mathbb{E}_{z \sim T} \phi( D( G(z), z) ) \triangleq f(x_1^*, \dots, x_m^*, z_1, \dots, z_m)
\end{equation*}
$$

are all independent. Hence, the concentration inequality called [McDiarmid's Inequality](https://people.eecs.berkeley.edu/~bartlett/courses/281b-sp08/13.pdf) (MI) can be used to guarantee concentration.

McDiarmid's Inequality has the following two conditions:

1. Independence: the random variable is dependent only on mutually independent random variables 
2. Bounded differences, given by

$$
\begin{equation*}
\sup_{x_1,\dots, x_n, \hat{x}_i} | f(x_1, \dots, x_n) - f(x_1, \dots, x_{i-1}, \hat{x}_i, x_{i+1}, \dots, x_n) | \leq c_i,
\end{equation*}
$$

for $$1 \leq i \leq n$$.

The first condition is satisfied by the setup of $$G$$ and $$T$$. The second condition is also verified by the author, with the bound given by $$1 / m$$. Note that this is part of the reason for the complicated expression of $$m$$.

Applying **McDiarmid's Inequality**, we have the result:

$$
\begin{equation*}
\textrm{Pr}_{T,G}( |f(x_i^*, z_i) - \mathbb{E}_{T,G} f(x_i^*, z_i) | ) \leq \exp(-\Omega(m\epsilon^2)),
\end{equation*}
$$

where we abbreviated the notation of $$f$$.

From here, there are just a couple of more steps to the final generator concentration result using an epsilon-net / union bound argument and applications of another simple concentration inequality called Markov's inequality. 