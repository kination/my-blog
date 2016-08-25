---
layout: post
title: Setup python environment
date: 2016-07-25 20:27:30
image: '/assets/img/'
description: Develop python projects with virtualenv & pyenv
main-class: 'python'
color:
tags:
- python
- virtualenv
- pyenv
categories: python
twitter_text:
introduction: Develop python projects with virtualenv & pyenv
---

If you are looking on multiple python projects, you would have experienced a problem with different package version between project. For example, one of the project are using django 1.7.x, while other only supports 1.9.x. In this case, you need to re-setup your packages everytime changing working project.
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

It could be fine with this. But if you want something that could simplify switching python version of your system, there is another thing named `pyenv`.

{% highlight shell %}
{% raw %}
$ brew update
$ brew install pyenv
...
$ echo 'eval "$(pyenv init -)"' >> ~/.bash_profile
$ pyenv --version
pyenv 20160726
{% endraw %}
{% endhighlight %}

Now you can use `pyenv`. It is pretty simple. First look on some commands.
{% highlight shell %}
{% raw %}
$ pyenv versions
  system
  2.7
  2.7.12
* 3.5.1
$ python --version
Python 3.5.1
$ pyenv shell 2.7.12
$ python --version
Python 2.7.12
{% endraw %}
{% endhighlight %}
This shows installed versions of python. I have downloaded 2.7, 2.7.12, 3.5.1, and selected version is marked with \*. If you want to switch the version, just use `pyenv shell (version number)`.

Installing version is simple as this. Check the list of versions, and install it.
{% highlight shell %}
{% raw %}
$ pyenv install -list
Available versions:
  2.1.3
  2.2.3
  2.3.7
...
$ pyenv install 3.6-dev
Cloning https://hg.python.org/cpython...
Installing Python-3.6-dev...
Installed Python-3.6-dev to /Users/kwangin/.pyenv/versions/3.6-dev

$ pyenv versions
  system
  2.7
  2.7.12
* 3.5.1
  3.6-dev
{% endraw %}
{% endhighlight %}

This does not offer seperated environment on same project, but it is more simple to solve python version dependency problem. You don't need disk storage to load python version on each project like virtualenv. There are pros and cons on both of it, so think about good combinations.
