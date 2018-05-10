---
layout: post
title:  "Pre-process data, for research"
date:   2018-01-15
description: Why, and simple how-to of data pre-processing
categories:
- python
- data
- pre-process
permalink: data-pre-processing
---


## What is pre-processing of data?
As I wrote in previous post, most dataset in real world are not clean. They are usually messed up, incompatible(ex:'-1' is included in data which defines people's age), and some of are missing. These kind of dataset will mostly cause error, or return wrong result when you just put on your elaborated logic. That's why you need to concern pre-process before researching it.

![Screenshot](/assets/post_img/data-pre-processing/data-pre-process-enjoyable.png)
![Screenshot](/assets/post_img/data-pre-processing/data-pre-process-spend-time.png)

This is the survey from data scientists. You can find out this work will be unpleasure, takes lot of time, but also shows the importance of this work.

I'll add on some pre-processing logic in data which I worked on [previous post](http://djkooks.github.io/first-kaggle-research), to see how this process makes change.


## Data transform
Look values in `Name` column, and you could find there are additional title such as 'Mr', 'Mrs', 'Sir', 'Dr', or else. By this data, you could assume target's gender or position. Because this is story of early 20 century, which was hierarchical society, so we could estimate this could have relation with survival rate. Let's add new column `Title` to use this.

{% highlight python %}
{% raw %}

{% endraw %}
{% endhighlight %}

{% highlight python %}
{% raw %}
train_df['Title'] = train_df['Name'].str.extract('([A-Za-z]+)\.')
pd.crosstab(train_df['Title'], train_df['Sex'])
{% endraw %}
{% endhighlight %}

![Screenshot](/assets/post_img/data-pre-processing/titanic-title-before.png)

Now column `Title` keeps parsed value from name, but there are bit varied so need to wrap similar values. Purpose for this column is to find out relation between 'title' and 'survival' so I'll wrap values with new name by its rank. 
Also, unify value of different spelling but has similar meaning, such as 'Mlle' and 'Ms' which all can be equal with 'Miss'.

- ‘Capt’, ‘Col’, ‘Don’, ‘Dr’, ‘Major’, ‘Rev’, ‘Jonkheer’, ‘Dona’ => Rare(others)
- ‘Countess’, ‘Lady’, ‘Sir’ => Royal(people of high standing)
- ‘Mlle’, ‘Ms’ => Miss
- ‘Mme’ => Mrs

Now redefine value of `Title`, and compare survival rates by this.

{% highlight python %}
{% raw %}
train_df['Title'] = train_df['Title'].replace(['Capt', 'Col', 'Don', 'Dr', 'Major', 'Rev', 'Jonkheer', 'Dona'], 'Rare')
train_df['Title'] = train_df['Title'].replace(['Countess', 'Lady', 'Sir'], 'Royal')
train_df['Title'] = train_df['Title'].replace(['Mlle', 'Ms'], 'Miss')
train_df['Title'] = train_df['Title'].replace('Mme', 'Mrs')
train_df[['Title', 'Survived']].groupby(['Title'], as_index=False).mean()
{% endraw %}
{% endhighlight %}

![Screenshot](/assets/post_img/data-pre-processing/titanic-title-survival.png)

Survival rate of people with title 'Miss'/'Mrs', who are female, appears high, as we look before. And also, who has title ‘Royal’/‘Master’, that we could assume high-positioned people, also survived more than others. So we could define as 'people's rank is related with survival rate'.


## Missing value treatment - Age
There are some ways for treating missing values

1. Delete
2. Replace with mean or mode value
3. Replace with estimated value using regression or else
4. etc.

In this post, I'll go on with 2, but not just fill mean value of all data in all missing data. 

I'll work with assumption:
> People who has same `Title` will be in same age group.

So first look on mean value of age by `Title`.
{% highlight python %}
{% raw %}
train_df.groupby('Title')['Age'].mean()
{% endraw %}
{% endhighlight %}

...it will be:
{% highlight shell %}
{% raw %}
Title
Master     4.574167
Miss      21.845638
Mr        32.368090
Mrs       35.788991
Rare      45.894737
Royal     43.333333
Name: Age, dtype: float64
{% endraw %}
{% endhighlight %}

Now fill in value to missing data.
{% highlight python %}
{% raw %}
train_df.loc[(train_df['Age'].isnull())&(train_df['Title']=='Master'),'Age'] = 5
train_df.loc[(train_df['Age'].isnull())&(train_df['Title']=='Miss'),'Age'] = 22
train_df.loc[(train_df['Age'].isnull())&(train_df['Title']=='Mr'),'Age'] = 33
train_df.loc[(train_df['Age'].isnull())&(train_df['Title']=='Mrs'),'Age'] = 36
train_df.loc[(train_df['Age'].isnull())&(train_df['Title']=='Rare'),'Age'] = 45
train_df.loc[(train_df['Age'].isnull())&(train_df['Title']=='Royal'),'Age'] = 43
{% endraw %}
{% endhighlight %}
 

## Data binning - Age
The case we are looking is binary classification, as there are only two label for result(live or die). In this case, continuous data needs to be grouped as category to deduct classifed result. 

Data binning is a process wraping continuous data with group by specific criteria.
My rule is:
- Less than 7 : Baby & Kids (0)
- 8~20: Students (1)
- 21~30: Young Adults (2)
- 31~40: Adults (3)
- 41~60: Seniors (4)
- More than 60: Elders (5)

I'll make a column `AgeGroup`, and update these value.
{% highlight python %}
{% raw %}
train_df['AgeGroup'] = 0
train_df.loc[ train_df['Age'] <= 7, 'AgeGroup'] = 0
train_df.loc[(train_df['Age'] > 7) & (train_df['Age'] <= 18), 'AgeGroup'] = 1
train_df.loc[(train_df['Age'] > 18) & (train_df['Age'] <= 30), 'AgeGroup'] = 2
train_df.loc[(train_df['Age'] > 30) & (train_df['Age'] <= 40), 'AgeGroup'] = 3
train_df.loc[(train_df['Age'] > 40) & (train_df['Age'] <= 60), 'AgeGroup'] = 4
train_df.loc[ train_df['Age'] > 60, 'AgeGroup'] = 5
f,ax=plt.subplots(1,1,figsize=(10,10))
sns.countplot('AgeGroup',hue='Survived',data=train_df, ax=ax)
plt.show()
{% endraw %}
{% endhighlight %}

![Screenshot](/assets/post_img/data-pre-processing/titanic-agegroup-survival.png)

People in group 0(Baby & Kids) has high survival rate, while other groups looks same. We can think that elder people make way to live to yongsters.


## Data binning - family member
There are also data which defines family members.

