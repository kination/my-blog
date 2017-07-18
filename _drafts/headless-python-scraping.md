---
layout: post
title:  "Web scraping inside headless browser"
date:   2017-07-24
tags:
- python
- web
- scraper
permalink: 
description: Web scraping with python, via headless browser
---

'Web scraping' is a logic to get web page data as HTML format. With this information, not only could get text/image data inside of target page, we could also find out which tag has been used and which link is been included in. When you need lots of data for research in your system, this is one of the common way to get data.

### Use parser

But not like CSV or Excel sheet, raw HTML is pretty rough and disordered data. 

{% highlight xml %}
{% raw %}

{% endraw %}
{% endhighlight %}


{% highlight python %}
{% raw %}
import requests

url = 'http://bleacherreport.com/'
TIMEOUT_SECONDS = 30

response = requests.get(url, timeout=30)
{% endraw %}
{% endhighlight %}

This is the case of getting raw HTML data from [Bleacher Report](http://bleacherreport.com/). It will return data like this.

{% highlight html %}
{% raw %}
<!DOCTYPE html><html class="no-js" lang="en"><head><meta charset="utf-8"/><meta http-equiv="Accept-CH" content="DPR,Width,Viewport-Width"/><meta name="aol-te-auth" property="aol-te-auth" content="1c424580-0f86-4d9b-88b2-bc8c0d029d4c"/><meta name="blitz" property="blitz" content="mu-6e4ce5cd-57f20d11-7c0ecee9-d55c79e2"/><meta name="msvalidate.01" property="msvalidate.01" content="7A63840181953B2A5A1FEA25FB45A991"/><meta name="robots" property="robots" content="NOODP,NOYDIR"/><meta name="verify-v1" property="verify-v1" content="+Ntj422Jc4V03qgBqLYbF3LMvrursV0X2btn2Zoqn9w="/><meta name="description" property="description" content="Sports journalists and bloggers covering NFL, MLB, NBA, NHL, MMA, college football and basketball, NASCAR, fantasy sports and more. News, photos, mock drafts, game scores, player profiles and more!"/><meta name="keywords" property="keywords" content="Bleacher Report, bleacherreport.com, sports, writers, articles, comments, blogs, Formula 1, Premier League, Hockey, Soccer, Recaps, Boxscores, Stats, Schedules, Tennis, Golf, WWE, fantasy football, Lebron James"/><meta name="viewport" property="viewport" content="width=device-width, initial-scale=1"/>
...
{% endraw %}
{% endhighlight %}

It is inconvinent for workers to find target data from here. You need a process of organizing before going further. Maybe you could make a parser yourself, but that's not an effective way. Python have some great modules for this, and I will use one of this named [BeautifulSoap](https://www.crummy.com/software/BeautifulSoup/).

Before going on, install it via python package manager with `pip install beautifulsoup4`.

{% highlight python %}
{% raw %}
import requests
from bs4 import BeautifulSoup

url = 'http://bleacherreport.com/'
TIMEOUT_SECONDS = 30

response = requests.get(url, timeout=30)
bs_object = BeautifulSoup(response.text, "html.parser")
{% endraw %}
{% endhighlight %}

With this code, raw HTML data has been converted to beautifulsoap object. Now you can get data with text or tag info.

{% highlight python %}
{% raw %}
# 'find' method returns the first searched data, while 'findAll' method returns all finded data in HTML as list.
# this will find data which has 'p' tag
p_tag_data = bs_object.find('p')
p_tag_data = bs_object.findAll('p')

# find exact text 'NBA' in HTML
nba_text = bs_object.find(text='NBA')
nba_text = bs_object.findAll(text='NBA')

# show all data which includes text 'NBA'
import re
nba_text = bs_object.findAll(text=re.compile('NBA'))

# return all p tag data which includes word 'fans'
fans_data = bs_object.findAll('p', text=re.compile('fans'))

# find link tag which includes 'rel="canonical"' element
bs_object.findAll("link", {"rel": "canonical"})
{% endraw %}
{% endhighlight %}


### Why do we need browser for scraping?

There are a problem on process above




