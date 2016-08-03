---
layout: post
title: Using virtualenv
date: 2016-07-30 20:27:30
image: '/assets/img/'
description: Develop python projects on virtualenv
main-class:
color:
tags:
- python
- virtualenv
categories: python
twitter_text:
introduction: Using virtualenv
---

If you are looking on multiple python projects, you would have experienced a problem with different package version between project. For example, one of the project are using django 1.7.x, while other only supports 1.9.x. In this case, you need to re-install django package everytime changing working project.
Because there are lots of packages which not support backward compatibility, to avoid this issue, it is good to make seperate environment on each project. For this purpose, virtualenv is quite a good answer.

Install first...

{% highlight shell %}
{% raw %}
pip install virtualenv
{% endraw %}
{% endhighlight %}

then go to your python project page, and type:
{% highlight shell %}
{% raw %}
$ cd your-python-project
$ virtualenv venv
$ source venv/bin/activate
{% endraw %}
{% endhighlight %}

Now your project is seperated with your system. You could see '(venv)' on left of terminal. You can use other name than 'venv'. Do as you want.
One thing for sure, your project is only isolated with python pakages installed on system. You can run your project just same as before except that.

You can also create virtual environment working on other version of python. Use -p option:

{% highlight shell %}
{% raw %}
$ cd your-python-project
$ python --version
3.5.1
$ virtualenv -p /usr/bin/python2.7 venv
$ source venv/bin/activate
(venv) $ python --version
2.7.10
{% endraw %}
{% endhighlight %}

Go on project, and you could see folder created namd 'venv'. All of package settings will be setup under this folder, and package setup here only effects on your 'venv' area. Install packages for your project.
{% highlight shell %}
{% raw %}
(venv) $ pip install -r requirements.txt
{% endraw %}
{% endhighlight %}

This is it. Now you could work just same as before.
If you want to get out of 'venv', just type:

{% highlight shell %}
{% raw %}
(venv) $ deactivate
$
{% endraw %}
{% endhighlight %}

In my case, I am working on 2~3 python projects. There is 1 main project, and others are just for study. Because all of them are using different package versions, I installed main project packages to system, and use virtualenv on sub projects. It may be bothering to type these commands everytime for projects you work oftenly.
