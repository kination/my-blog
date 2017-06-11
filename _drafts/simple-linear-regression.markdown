---
layout: post
title:  "Linear regression"
date:   2017-06-17 23:56:45
categories:
- python
- statistic
- machine learning
permalink: sample-post-with-images
description: Simple instruction, and implementation about linear regression
---

One of the trend on developer is AI and machine learing, and I think most of engineer, like me, are trying to going into this field.
To start on machine learning, we need a knowledge about data analysis, and for that we have to remind statistics, which we suffered when we are a student. 


### Linear Regression?

Linear Regression is one way to find relation between variable, with given dataset. This theory has been more than a hundread years, but it is still broadly used and printed in most of machine learning text books.

Linear Regression tries to make a linear line like `y = Ax + B` which closely match values in data. It is the way to figure out `A` and `B` value from predictor value `x` and target value `y`. There also can be more than one variable in predict value, it is called as 'Multiple Linear Regression', and then it will have form like `y = A1x + A2x + A3x + A4x + ... + B`. Similarly, first case with single depandant value are being called as 'Single Linear Regression'


### Prepare

I'll try not to use python modules for machine learning this time, such as `pandas` or `scikit-learn`, or else, to look on detail process of this.
I prepared simple dataset for test. Here are information about 'weight', 'age', 'blood fat' of 25 anonymous people.
You can find this, and some other dataset in `http://people.sc.fsu.edu/~jburkardt/datasets/regression/regression.html`


### Implement for Single Linear Regression

Let's think about the first case, with single dependant variable. I'll look on how age is related with blood fat.

{% highlight python %}
{% raw %}
import csv
import matplotlib.pyplot as plt

csv_file = open('data.csv', 'r')
reader = csv.reader(csv_file)
x = []  # age
y = []  # blood fat

for value in reader:
    x.append(value[2])
    y.append(value[3])
    
plt.scatter(x, y, color = "m", marker = "o", s = 30)
plt.show()
{% endraw %}
{% endhighlight %}

You could find result like below.
![Screenshot](/assets/post_img/simple_linear_regression/single-plot.png)

