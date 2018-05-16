---
layout: post
title: Multi-variable regression, with tensorflow
date: 2018-04-16
description: 
tags:
- machine-learning
- regression
- python
- tensorflow
permalink: multi-var-regression
---

I posted some note about basics of [linear regression](http://djkooks.github.io/first-kaggle-research) before. In that time, I only treated about regression with single variable. But usually, there is no data with single feature, so I'll try to go on regression with multiple variable.
Most of the note/image is from the lecture in [Andrew Ng's Machine learning course](https://www.coursera.org/learn/machine-learning) in coursera, so go on to this if you want to know more.


## Multi-variable regression
![Screenshot](/assets/post_img/multi-var-regression/multiple-features.png)

This is the variable sets prepared for research. It shows some features of house, with the price. There are 4 features('Size', 'Number of bedrooms', 'Number of floors', 'Age of year'), and as you can see, I'll find out logic to predict the price by this dataset.




