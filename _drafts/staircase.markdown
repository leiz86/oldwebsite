---
layout: post
title: "Staircase Code Simulator in C++"
date:   2017-12-12 21:55:00 -0500 
categories: jekyll update
comments: true
description: introduce staircase codes, a class of high bit-rate, low decoding-complexity error-correcting codes for fiber-optic communication systems, along with a C++ simulator implementation
author: Lei M. Zhang
---

In today's post, I'd like to describe a class of error-correcting codes that have been the [focus](http://ieeexplore.ieee.org/document/6787025/) of my [research](http://ieeexplore.ieee.org/document/8082562/) [career](http://ieeexplore.ieee.org/document/7950925/) for the better part of the last 6 years. They are called [Staircase Codes](http://ieeexplore.ieee.org/document/6074908/) and were originally invented Professor Kschisching's research group. There has been some recent interest in using staircase codes as efficient error-correcting codes for next-generation, long-haul fiber-optic communication networks.

I have implemented a staircase code simulator in C++, available at my [GitHub](https://github.com/leiz86/staircase-simulator).

## What is an error-correcting code?

First, let's quickly review error-correcting codes. Given a particular channel model, e.g., the binary-input additive white Gaussian noise (AWGN) channel, we would like to transmit information (measured in bits) over the channel with arbitrarily low probability of error at the receiver.

According to information theory, such reliable communication is only achievable while operating below the **capacity** (or **Shannon limit**) of the channel, given as the rate bits/channel-use. The purpose of an error-correcting code, then, is to achieve the maximum rate of reliable communication, while operating below the channel capacity.

Operationally, the channel capacity gives us a convienent upper-bound on the performance of an error-correcting code. In simple terms, a good error-correcting code operates very close to the channel capacity.

## Staircase codes

# Channel Model

Staircase codes are error-correcting codes designed for the binary-symmetric channel (BSC). The BSC is a simple bit-flipping channel where each transmitted is flipped, independently, with probability $$p$$. The channel is usually denoted by $$\textrm{BSC}(p)$$. The capacity of this simple channel is given by

$$
\begin{equation*}
C(p) = 1 + p\log_2(p) + (1-p)\log_2(1-p).
\end{equation*}
$$

Though simple, the BSC is surprisingly effective at modelling modern, long-haul fiber-optic communication systems. Of course, the error-correcting code is usually the very last block in a very long chain of digital, analog, and optical processing modules in which the chanel is not very BSC-like. Nevertheless, at the input of the error-correcting code, the BSC is a good model for the channel.

# Motivating Factors

Good error-correcting codes for the BSC has been known for a few decades. First to come to mind are low-density parity-check (LDPC) codes and Turbo codes. The key motivation for the invention of the new class of staircase codes for long-haul fiber-optic communications is the fact that these systems have extremely high bit-rates: on the order of 1, 10, even 100 Gbits/s. Modern error-correcting codes such as LDPC or Turbo codes have significant decoding complexity and data-flow which makes supporting the high bit-rates difficult.

Staircase codes are designed to minimize decoding complexity and data-flow. Emphasizing these requirements means their performance (in terms of coding gain) is slightly below that of other modern error-corrrecting codes. However, the decrease in complexity is well-worth the trade-off.

For a detailed (and quantitative) discussion of the motivating factors behind staircase codes, please refer to the paper [Staircase Codes: FEC for 100 Gb/s OTN](http://ieeexplore.ieee.org/abstract/document/6074908/). 

# Code Structure and Encoding

Here is a block diagram of the staircase code structure. The origin of its name is now obvious!

![staircase diagram](/files/fig/staircase/staircase.jpg "Staircase Code Diagram")

To understand this diagram, we first define a **component code**, which is any systematic binary block code $$\mathcal{C}$$ of length $$n$$, information bits $$k$$, and decoding radius $$t$$, denoted by $$\mathcal{C}(n,k,t)$$. We also assume that $$\mathcal{C}$$ has a low-complexity decoding algorithm, for example, a syndrome decoder.

First, we select $$w=n$$ and arrange $$w$$ copies of $$\mathcal{C}$$, row-by-row, within each consecutive pair of staircase blocks: $$B_i$$, $$B_{i+1}$$. Naturally, this requires that $$n$$ is even.

Initially, block $$B_0$$ is filled with all-zero bits. Since this block carries no information, it is actually not transmitted. 

To encode the next block, arrange information (input) bits row-by-row into the white part of block $$B_1$$. Arrange blocks $$B_0$$ and $$B_1$$ into a $$w \times 2w$$ matrix, encode each component code $$\mathcal{C}$$ using its own systematic encoder. The resulting parity (or check) bits are used to populate the red part of block $$B_1$$. For subsequent blocks, given $$B_{i-1}$$, the matrix $$B_{i-1}^T$$ and $$B_i$$ is formed ($$()^T$$ denotes matrix transpose) and the component code encoding proceeds as above.

Since the red blocks do not carry information, the code rate of a staircase code is given by

$$
\begin{equation*}
R = \frac{w - (n-k)}{w} = 1 - 2(1  - k/n).
\end{equation*}
$$

# Decoding