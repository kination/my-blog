---
layout: post
title:  "Web scraping inside headless browser"
date:   2017-07-23
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

{% highlight python %}
{% raw %}
import requests

url = 'http://bleacherreport.com/'
TIMEOUT_SECONDS = 30

response = requests.get(url, timeout=30)
{% endraw %}
{% endhighlight %}

This is the case of getting raw HTML data from [Bleacher Report](http://bleacherreport.com/). It will return data like this.

{% highlight HTML %}
{% raw %}
<html class="no-js" lang="en"><head><meta charset="utf-8"/><meta http-equiv="Accept-CH" content="DPR,Width,Viewport-Width"/><meta name="aol-te-auth" property="aol-te-auth" content="1c424580-0f86-4d9b-88b2-bc8c0d029d4c"/><meta name="blitz" property="blitz" content="mu-6e4ce5cd-57f20d11-7c0ecee9-d55c79e2"/><meta name="msvalidate.01" property="msvalidate.01" content="7A63840181953B2A5A1FEA25FB45A991"/><meta name="robots" property="robots" content="NOODP,NOYDIR"/><meta name="verify-v1" property="verify-v1" content="+Ntj422Jc4V03qgBqLYbF3LMvrursV0X2btn2Zoqn9w="/><meta name="description" property="description" content="Sports journalists and bloggers covering NFL, MLB, NBA, NHL, MMA, college football and basketball, NASCAR, fantasy sports and more. News, photos, mock drafts, game scores, player profiles and more!"/><meta name="keywords" property="keywords" content="Bleacher Report, bleacherreport.com, sports, writers, articles, comments, blogs, Formula 1, Premier League, Hockey, Soccer, Recaps, Boxscores, Stats, Schedules, Tennis, Golf, WWE, fantasy football, Lebron James"/><meta name="viewport" property="viewport" content="width=device-width, initial-scale=1"/>
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


### Why it needs to use browser

There are a problem on process above, not in parser, but in HTML request process. 
If we just request data by http request method, it cannot get <b>dynamically rendered</b> part because they are not in HTML file before loading process. For example, let's see inside.

![Screenshot](/assets/post_img/python_scraping_headless/js_enable.png)

This is the ordinary form of main page. Now let's check how this will be look like after disabling javascript.

![Screenshot](/assets/post_img/python_scraping_headless/js_disable.png)

Menu in top has been disappeared, because they are being rendered dynamically from javascript controller. And also, menu texts will not being scraped when you get this HTML page with http request process. So, to get all of these data, you need to get fully rendered result data. For getting it, it has to be rendered in some kind of 'fake browser', and that's why we will use 'headless browser'. 

This is description of headless browser in wikipedia.
{% highlight html %}
{% raw %}
A headless browser is a web browser without a graphical user interface.

Headless browsers provide automated control of a web page in an environment similar to popular web browsers, but are executed via a command-line interface or using network communication. 
{% endraw %}
{% endhighlight %}


### Scraping via headless browser

To make scraper via headless browser, we need headless browser, and module to make this run in virtual. 
I will use [PhantomJS](http://phantomjs.org/) here for headless browser, and will make it run with [Selenium](http://www.seleniumhq.org/). You need to install these first.

PhantomJS can be installed by downloading from main page, but can be installed with `brew` or `npm`. You can use one of 2 command below.
{% highlight shell %}
{% raw %}
$ brew phantomjs
$ npm -g install phantomjs-prebuilt
{% endraw %}
{% endhighlight %}

Try make a class for scraper.

{% highlight python %}
{% raw %}
from selenium import webdriver
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities

class HBScraper():

    def __init__(self):
        self.web_driver = webdriver.PhantomJS(desired_capabilities=dict(DesiredCapabilities.PHANTOMJS))
        self.web_driver.set_window_size(960, 540)
        self.web_driver.set_page_load_timeout(60)
        self.web_driver.set_script_timeout(60)
        self.web_driver.delete_all_cookies()

    def scrape_page(self, page_url):
        try:
            self.web_driver.get(page_url)
        except:
            return None

        return self.web_driver.page_source

{% endraw %}
{% endhighlight %}

`HBScraper` class is the class for getting 'loaded' page data. It initiates browser setting like max loading time, window size, etc. on `__init__` method. You can do scraping and get scraped result from browser with `scrape_page`.

Use it like below.
{% highlight python %}
{% raw %}
url = 'http://bleacherreport.com/'

hb_scraper = HBScraper()
bs_object = BeautifulSoup(hb_scraper.scrape_page(url), "html.parser")
{% endraw %}
{% endhighlight %}

Because it needs time for loading, scraping with headless browser needs more time to get result than just getting data with HTTP request. But to get exact data of target page, you will need to consider of using it.


### Reference

* [https://en.wikipedia.org/wiki/](https://en.wikipedia.org/wiki/)
* [http://phantomjs.org/](http://phantomjs.org/)
* [http://www.seleniumhq.org/](http://www.seleniumhq.org/)
* [https://stackoverflow.com/](https://stackoverflow.com/)

