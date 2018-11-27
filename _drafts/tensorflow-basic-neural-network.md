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
This note, is the challenge for going with this from basic.


## Tensorflow, and deep learning libraries
Though you are not a deep learning engineer, you would heard about tensorflow. Since google released as open source on 2015, it has been most popular deep learning libraries.
Somebody could just think it is the power of google brand, but it is really a thing. It has well-designed APIs, good documents translated with various languages, and most important, it is being used in lots of real world service including google photo, google translate, gmail, android, and more.

Now other major IT companies(Facebook, Microsoft, Amazon...) are releasing their deep learning libraries as open source, and competiting with tensorflow. It's good news for engineers having lot of options. But I think it is meaningless thinking which option you will select, if you are not used to on logic of how deep learning are working on and how to use it. So I'll only deal with tensorflow here. As I said, I'm really a beginner in deep learning yet.


## Artificial Neural network
If you heard of deep learning, you would also heard about artificial neural network(ANN). As you could expect in word 'artificial', it is something imitate neural network in human brain to replicate how human learns from data.

![Screenshot](/assets/post_img/tensorflow-basic-neural-network/ann-basic-graph.jpg)

This is basic form of ANN. There are input layer, hidden layer(can be more than 1), and output layer. These layers are made of nodes, which are workin as neuron in human. Each node receives multiple data, calculate based on some function, and send result to connected nodes.

![Screenshot](/assets/post_img/tensorflow-basic-neural-network/node-neuron.jpeg)

Each neuron uses weight, and bias(or offset) to calculate data. If you look about regression in mathematics, you would be familiar with `y = W * x + b`. 
...
...


## Setup for research
First, import tensorflow library, and image data for research. I'll use fashion-mnist data, 60,000 imageset of clothing/shoes. Each of them are 28x28 grayscale image, classified to 10 classes. Data includes label list of class, as below.

| Label | Description |
| --- | --- |
| 0 | T-shirt/top |
| 1 | Trouser |
| 2 | Pullover |
| 3 | Dress |
| 4 | Coat |
| 5 | Sandal |
| 6 | Shirt |
| 7 | Sneaker |
| 8 | Bag |
| 9 | Ankle boot |

You can download dataset [here](https://github.com/zalandoresearch/fashion-mnist).

```python
import tensorflow as tf
from tensorflow.examples.tutorials.mnist import input_data
fashion_mnist = input_data.read_data_sets("path/of/fashion-mnist", one_hot=True)
```

Import tensorflow module, and datasets to your project. By using `tensorflow.examples.tutorials.mnist.input_data`, you can receive data as tensorflow `DataSet` object from formatted zip files. This `DataSet` type offers APIs to make training code more simple.

Now check received data. It has 2 `DataSet` as train and test. Each of it includes 2-dimension numpy array, including information of dataset.

```python
fashion_mnist.train.images.shape    #(55000, 784)
fashion_mnist.train.labels.shape    #(55000, 10)
fashion_mnist.test.images.shape     #(10000, 784)
fashion_mnist.test.labels.shape     #(10000, 10)
```

Each image data are composed as an array with 784 formula. It is data of each pixel in 28x28(=784) image. Labels are more simple, it is just array with 10 formula, which defines what kind of image it is.

Than let's look on few data.

```python
import matplotlib.pyplot as plt

class_names = [
    'T-shirt/top',
    'Trouser',
    'Pullover',
    'Dress',
    'Coat', 
    'Sandal',
    'Shirt',
    'Sneaker',
    'Bag',
    'Ankle boot'
]

fig = plt.figure(figsize=(8, 8))
for i in range(4):
    target_train_num = i + 5
    fig.add_subplot(2, 2, i+1)
    plt.imshow(fashion_mnist.train.images[target_train_num].reshape(28, 28), cmap='Greys')
    label_num = np.where(fashion_mnist.train.labels[target_train_num] == 1)[0][0]
    print(class_names[label_num])
    
plt.show()
```
This picks 5th to 8th data, and show label name and image. Because image is defined as 1d array, I changed to 28x28 2d array.

![Screenshot](/assets/post_img/tensorflow-basic-neural-network/fashion-mnist-plt.png)

Okay, data seems good for now.


## Entropy, and Cross-entropy
Surprisal analysis is one technique of Information theory, which calculates the degree/amount of 'surprised' by getting certain information. In formula, it can be show as

![Screenshot](/assets/post_img/tensorflow-basic-neural-network/surprisal-formula.png)

For example, let's say I have 2% possibility of winning the lottery. In this case:

- Winning the lottery => log(1/0.02) = 1.69897000434
- Not winning the lottery => log(1/0.98) = 0.0087739243

In information theory, it assume 'degree of uncertain event' is larger than certain ones. By this theory, it shows the surprisal degree of getting information 'winning lottery' is almost 200 times larger than the other case.

Okay, now go with `Entropy`(a.k.a 'Shannon Entropy'). This is mean value of this 'surprisal' in one event. It is defined as:

![Screenshot](/assets/post_img/tensorflow-basic-neural-network/shannon-entropy-formula.png)

In case above, it can described as
=> `(0.98 x 0.0087739243) + (0.02 x 1.69897000434)`

So the `Entropy` of my lottery game is 0.04256.

But there will be the difference bet



## Making layers in TF

When you are trying to research on image, it will... In MNIST example, there are 55000 images, and size of each image is 28*28 = 784. So input will be [55000, 784] vector.






## References
* https://www.tensorflow.org/tutorials/
* https://hackernoon.com/artificial-neural-network-a843ff870338
* https://www.digitalocean.com/community/tutorials/how-to-build-a-neural-network-to-recognize-handwritten-digits-with-tensorflow
* https://github.com/zalandoresearch/fashion-mnist
* https://en.wikipedia.org/wiki/Information_content
* https://medium.com/@vijendra1125/understanding-entropy-cross-entropy-and-softmax-3b79d9b23c8a

