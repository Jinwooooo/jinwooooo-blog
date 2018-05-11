---
layout: post
title:  "Introduction to Neural Network"
image: ''
date:   2018-05-16 00:12:00
tags:
- English
- Deep Learning
- Neural Network
- Python
description: 'One of the popular Deep Learning Algorithms for Image Classification'
categories:
- Neural Network
---

## Abstract

I've been interested in Neural Networks recently and I've started to study about it. Here is how I understood the basics of
Neural Network. (The sources I've referenced the most when learning was
[MIT OpenCourse Lecture](https://www.youtube.com/watch?v=uXt8qF2Zzfo) and
[O'Reilly recent Machine Learning Book](http://shop.oreilly.com/product/0636920052289.do))

-----

## Table of Contents

* Basic Structure of Neuron
* Basic Concept of Neuron Applied for Computers Visualized
* Elaboration on Neural Network Optimization
* Applying Concepts into Python (Tensorflow)

-----

## Basic Structure of Neuron

<img src="../uploads/ml-textbook-NN-neuron.jpg">

Basically, there are multiple neurons that communicate with each other by receiving information with dendritic tree and sending
information with axon. As you can see in the close up part, they don't send just 1 signals, but multiple signals. Those multiple
signals either makes the other neuron stimulated or not (basically it's either 0 or 1). Note that each signals are also either
0 or 1, but each signal have different **weights**. The 'learning' part is applied to these weights. If signal n is giving wrong answer, then you reduce the weight of signal n.

-----

## Basic Concept of Neuron Applied for Computers Visualized

For simplicity, I'm going to ignore about weights, and just stick with signals as 0 and 1. In the O'Reilly ML Book, it has this
tree like diagram showing simple logical computations.

<img src="../uploads/ml-textbook-NN-logical-comp.png">

For 2 signals (in this case A and B), you can see with diagram how signals can lead up to stimulate the other neuron. In the
textbook, it also leaves a question at the end : *How would A XOR B look like?*

<img src="../uploads/challenge-accepted.jpg">

It didn't really say if there could be intermediate nodes to get the inputs, so for the sake of a better looking graph,
I'm going to assume there can be intermediate nodes to save a computed value. Here is my diagram.

<img src="../uploads/ml-textbook-NN-q2.png">

------

## Elaboration on Neural Network Optimization

Time to get our hands dirty with a bit of mathematical concepts! I'm going to leave out some maths out (you have to do
chain rules and stuff) However, you do need to understand what taking a derivate of a certain function does.

Here is the first step

<img src="../uploads/ml-textbook-NN-convert.jpg">

* Let there be **n** number of signals.
* **x** is when will either be 0 or 1 (it fires or doesn't fire signals)
* **w** is weight

Note there is an **-T** input with the signal inputs. This is done for mathematical convenience (translating the T to origin).
The figure below shows the translation.

<img src="../uploads/ml-textbook-NN-translation.jpg">

Then there is 3 popular activation functions. Before I elaborate on those functions, a good question is, **why do we need to transform a stepwise function into the another function?** Later on when we try to find the maximum/minimum (depending on how you look at it), we'd have to use a optimization concept called **gradient ascent/descent**. This can be only applied to a continuous function. Thus, the stepwise function was transformed into a continuous function.

* **sigmoid** : this one is intuitive and is done quite a lot in the field of data analysis (i.e. if you have a question that has answers yes/no, then normally a sigmoid is done on the data). However, this usually needs more computation than other function and usually gives mediocre results compared to other activation function
* **tanh (hyperbolic tangent function)** : it's pretty similar to sigmoid, but it helps speed up convergence due to stronger gradient casued by the larger range of the function.
* **ReLU (Rectified Linear Unit)** : there is a slight problem at origin due to abrupt change, it has mathematical problems when differentiation. However, due to it's shape, it is fast to compute it does reduce some issues with gradient ascent/descent.

Now it's time to move on the how this turns into a gradient ascent/descent problem. First we have an output **z** that can be compared to the desired (actual) result to measure performance, so that math goes like this.

<img src="../uploads/ml-textbook-NN-transform-minmax.jpg">

*Just incase the reader is not familiar why the values are in vector... since the desired and output have multiple values, it's possible to think of them as a 1xN dimension matrix (where N = number of values), which is basically a vector.*

If just subtracted the two vectors, it's not very convienent for math, so the the whole performance is squared. The **-** is done so that we can do gradient ascent (usually climbing is the norm). Of course there is no problem just using the positive value and do a gradient descent. (for this blog I will keep using maximum / gradient ascent)

Now all we have to do is find the maximum. Now the problem can be taken on as shown below.

<img src="../uploads/ml-textbook-NN-transform-gradient-ascent.jpg">

It's explained above, but gradient ascent done naively is exponential (intractable), so differentiation is used. When both in vertical and horizontal direction changes the most positively means that that is the optimal step to take.

Now with this we keep the weights adjusting so the neural network is modified to be closer and closer to the desired output!

Since overall theory of neural network is covered let's get our hands dirty with Python codes... '^'

<img src="../uploads/coding.gif">

------

## Applying Concepts into Python (Tensorflow)

Soon to be added
