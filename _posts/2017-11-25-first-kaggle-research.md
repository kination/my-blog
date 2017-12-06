---
layout: post
title:  "First data research in kaggle"
date:   2017-11-25
description: Introducing kaggle, and try on first data research with Titanic passenger list
tags:
- python
- data research
- kaggle
permalink: first-kaggle-research
---


## Kaggle?
![Screenshot](/assets/post_img/first_kaggle_research/kaggle-main.png)
[kaggle](https://www.kaggle.com/) is a platform for competiting data analytic and predictive modeling. Lots of companies post their raw data, and researchers compete to find best prediction from here. You can use `Python`, `R`, or `Julia` for a research. 
Each researched records are managed as `kernel`, and you can make your own or look on other user's data.




## Titanic research
This is one of most famous topic in data research, and also `kaggle` suggest this as tutorial of platform, and data research. I'll not explain in detail about Titanic cause it is most infamous disasters and all of you will know about this very well.
In this research, given data is list of passengers and crew, and informations like age, embarked, sex, etc. are in here. Goal of this analysis is to find out what sort of people were likely to survive.

If you are first in data research so don't know how to start on, you can refer other people's kernel. Some of good kernels are commented very well to look and follow up how they work on. Or you can just look through blog posts in Google about 'Titanic data research' cause it is very popular tutorial. Also I'm a starter in this field, I followed research tutorial in `Datacamp`, and public notebooks of other great engineers.


## Read and Visualize data
First thing you have to do when face on dataset, is to read and find out what is in it.

![Screenshot](/assets/post_img/first_kaggle_research/titanic-data-cap.png)

This is the given data, which you need to research on. Each of column means...

- `survival` 	Survival 	0 = No, 1 = Yes
- `pclass` 	Ticket class 	1 = 1st, 2 = 2nd, 3 = 3rd
- `sex` 	Sex 	
- `Age` 	Age in years 	
- `sibsp` 	# of siblings / spouses aboard the Titanic 	
- `parch` 	# of parents / children aboard the Titanic 	
- `ticket` 	Ticket number 	
- `fare` 	Passenger fare 	
- `cabin` 	Cabin number 	
- `embarked` 	Port of Embarkation 	C = Cherbourg, Q = Queenstown, S = Southampton

There are 2 given files. `train.csv` is for training, and you need to create research model based on this. `test.csv` is a data you need to predict survivers with your model, so there is no `survival` column here.



## Reference
* [Kaggle Main page](https://www.kaggle.com/)
* [Titanic Competition kernels](https://www.kaggle.com/c/titanic/kernels)
* [Kaggle tutorial in DataCamp]((https://www.datacamp.com/community/open-courses/kaggle-python-tutorial-on-machine-learning)
