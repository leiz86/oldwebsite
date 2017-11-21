---
layout: post
title: "Capsule Network"
date:   2017-11-19 16:35:00 -0500 
categories: jekyll update
---

In this post, I discuss the paper [Dynamic Routing Between Capsules](https://arxiv.org/abs/1710.09829) (CapsNet17) by S. Sabour, N. Frosst, and G. Hinton. Let's start with the two main ideas from the paper: **capsules** and **dynamic routing**.

## What are Capsules?

A capsule is a generalization of a scalar neuron to a vector. 

Let $$N$$ be the number of inputs to a standard neuron. Let the set $$\mathcal{I} \triangleq \{0, \dots, N-1\}$$ be an index set of the inputs $$x_i \in \mathbb{R}$$ and input weights $$w_i \in \mathbb{R}$$ for $$i \in \mathcal{I}$$, then the output $$y \in \mathbb{R}$$ of a standard neuron is given by

$$
\begin{equation}\label{eqn:neuron}
y = f\left( \sum_{i \in \mathcal{I}} w_i x_i \right) 
\end{equation}
$$

where $$f: \mathbb{R} \mapsto \mathbb{R}$$ is a non-linear function such as sigmoid or ReLU.

We use the notation $$\mathbf{x} \triangleq (x_0, \dots, x_{d-1})$$ to denote a $$d$$-tuple. If the $$d$$-tuple is an element of a $$d$$-dimensional vector space, then $$\mathbf{x}$$ is referred to as a (row) vector. We denote the 2-norm of $$\mathbf{x}$$ by $$ \|\mathbf{x}\| $$.

There are two types of capsules in the capsule network for the MNIST dataset given in CapsNet17. **Primary capsules** are intended to represent local features. **Digit capsules** are intended to represent the digit. Let us first focus on digit capsules.

Here, assume that an arbitrary digit capsule, $$\mathbf{v}$$, has the same number of inputs, $$N$$, as the above neuron. Let $$\mathbf{u}_i$$ for $$i \in \mathcal{I}$$ be all of the primary capsules that are possible inputs to $$\mathbf{v}$$. We assume that $$\mathbf{u}_i$$ and $$\mathbf{v}$$ are elements in vector spaces $$\mathbb{R}^{d_u}$$ and $$\mathbb{R}^{d_v}$$, where $$d_u \neq d_v$$, in general.

Assume that we know the set of $$ c_i $$ for $$i \in \mathcal{I}$$. These are routing coefficients that is determined by the dynamic routing algorithm (see next section). To define a similar expression as ($$\ref{eqn:neuron}$$) for capsules, we need to redefine $$w_i$$ and $$f$$. 

One simple solution is to replace each $$w_i$$ by a matrix of dimension $$d_u \times d_v$$, $$W_i$$. Then, replace $$f$$ by a non-linear, vector-input scalar-ouput function. In CapsNet17, this function is given by

$$
\begin{equation*}
g(\mathbf{x}) = \left(1 - \frac{1}{1 + \|\mathbf{x}\|^2}\right) \frac{\mathbf{x}}{\|\mathbf{x}\|}.
\end{equation*}
$$

Finally, we arrive at the expression for an arbitrary digit capsule

$$
\begin{equation}\label{eqn:capsule}
\mathbf{v} = g\left(\sum_{i \in \mathcal{I}} c_i \mathbf{u}_i W_i\right).
\end{equation}
$$

It is no surprise that ($$\ref{eqn:capsule}$$) is similar to ($$\ref{eqn:neuron}$$). Since, as we mentioned, a capsule is really a generalization of a neuron to a vector.

### Discussion

From this perspective, the idea of capsules appears to be a straightforward generalization of the scalar neuron. Are there any good reasons for doing so?

From CapsNet17 and an [earlier work](http://www.cs.toronto.edu/~fritz/absps/transauto6.pdf) by Prof Hinton, there appear to be two reasons for thinking about vectors rather than scalars.

First, the pooling operation in convolutional neural networks, as a way of obtaining higher-level representations from filter outputs, is sub-optimal: 

The max operation could remove information (i.e., filter outputs that are less than the maximum). Primary capsules attempt to capture a similar higher-level representation **without loss of information** from convolution filter outputs. By simply aggregating groups of filter outputs into a capsule (a vector), all of the information is captured by different characteristics of the vector. For example, given a capsule $$\mathbf{x}$$, $$\| \mathbf{x} \|$$ is considered to be the probability that the feature represented by $$\mathbf{x}$$ is present in the input image. 

Second, capsules can retain more information about affine transformations of a feature than CNNs with pooling:

CNNs with max pooling is well-known to be translation . Translation is a type of affine transformation. In general, however, an affine transformation is simply as linear map between two vector spaces. In the case where the vector spaces are the same, for example, $$\mathbb{R}^d$$ and a vector is a value of a capsule $$\mathbf{x} \in \mathbb{R}^d$$, then any affine transformation $$T : \mathbb{R}^d \mapsto \mathbb{R}^d$$ with

$$ T(\mathbf{x}) = A\mathbf{x} + \mathbf{b} $$

simply maps between different values of the capsule. 

## What is dynamic routing?

The idea of dynamic routing introduced in CapsNet17 is, in my opinion, the most interest part of the work. It provides a way of connecting capsules between the primary and digit layers **without using back-propagation**. 

What it uses is essentially the correlation between different capsule outputs. Recall that each primary capsule $$\mathbf{u}_i$$ is first multiplied (on the right) by the matrix $$W_i$$. Denote this $$d_v$$-dimensional vector by $$\hat{\mathbf{u}}_i$$. The algorithm then calculates

$$
\hat{\mathbf{u}}_i \mathbf{v}^T
$$

to update the routing coefficients. Since the calculation of $$c_i$$ is now recursive, the dynamic routing algorithm is performed for a number of iterations.

Since $$g$$ essentially limits the norm of both vectors to less than or equal to 1, the above project is a measure of how closely matched the two vectors are in **orientation**.

As shown, the algorithm does not explicitly use back-propagation to find the routing coefficients. However, back-propagation is used to optimize the  parameters involved in the routing calculations. In particular, $$W_i$$.

## My Implementation



