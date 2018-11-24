---
layout: post
title:  Basic neural network with tensorflow
date:   2018-11-21
description: 
tags:
- tensorflow
- neuralnetwork
- deeplearning
- python
permalink: tensorflow-basic-neural-network
---

It has been several years since deeplearning became common in our life. Lots of services are based on it, and you can find guide books/documents easily in bookstore or google. But it is hard to go on deeply by understanding theories and logics, for most of engineer who is not majored this. Though I'm working on deep learning company(as a web engineer), still suffering with its principle and mathematical formulas. 
This note, is the challenge for going with this.


## Tensorflow, and deep learning libraries
Though you are not a deep learning engineer, you would heard about tensorflow. Since google released as open source on 2015, it has been most popular deep learning libraries.
Somebody could just think it is the power of google brand, but it is really a thing. It has well-designed APIs, good documents translated with various languages, and most important, it is being used in lots of real world service including google photo, google translate, gmail, android, and more.

Now other major IT companies(Facebook, Microsoft, Amazon...) are releasing their deep learning libraries as open source, and competiting with tensorflow. It's good news for engineers having lot of options. But I think it is meaningless thinking which option you will select, if you are not used to on logic of how deep learning are working on and how to use it. So I'll just think with tensorflow here. As I said, I'm really a beginner in deep learning yet.


## Artificial Neural network
If you heard of deep learning, you would also heard about artificial neural network(ANN). As you could expect in word 'artificial', it is something imitate neural network in human brain to replicate how human learns from data.

![Screenshot](/assets/post_img/tensorflow-basic-neural-network/ann-basic-graph.jpg)

This is basic form of ANN. There are input layer, hidden layer(can be more than 1), and output layer. These layers are made of nodes, which are workin as neuron in human. Each node receives multiple data, calculate based on some function, and send result to connected nodes.

![Screenshot](/assets/post_img/tensorflow-basic-neural-network/node-neuron.jpeg)

Each neuron uses weight, and bias(or offset) to calculate data. If you look about regression in mathematics, you would be familiar with `y = W * x + b`. 


## Making layers in TF

When you are trying to research on image, it will... In MNIST example, there are 55000 images, and size of each image is 28*28 = 784. So input will be [55000, 784] vector.



## References
* https://www.tensorflow.org/tutorials/
* https://hackernoon.com/artificial-neural-network-a843ff870338
* https://www.digitalocean.com/community/tutorials/how-to-build-a-neural-network-to-recognize-handwritten-digits-with-tensorflow
* https://skymind.ai/wiki/neural-network

