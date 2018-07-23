---
layout: post
title:  "asyncio and aiohttp"
date:   2018-05-16
description: Writing single-threaded concurrent service with python
categories:
- python
- asyncio
- aiohttp
permalink: python-async
---

`asyncio` is a high-level API for supporting to implement asyncrounous code, which has been added to python default module from version 3.4. There were some workaround for asyncrounous before, but `asyncio` is supporting process to run asyncrounous in language level. It will reduce lots of code comparing with old-style way.


## Why asyncrounous??
Well, developer knows about what asyncronous is...so what is it good for?
Though I have started python programming since 3.5, I didn't need to think about this module for about a year. Service I worked had request that can be handle enough with common syncrounous logic, and some of heavy task is processed with distribution queue like `celery`.

The reason I have been interested is while making some scraper for test...

{% highlight python %}
{% raw %}

{% endraw %}
{% endhighlight %}


{% highlight python %}
{% raw %}
import requests
import time

urls = [
    'https://www.python.org/',
    'https://www.javascript.com/',
    'https://www.amazon.com/',
    'https://www.netflix.com/',
    'https://www.scala-lang.org/'
]

s = time.time()
for url in urls:
    res = requests.get(url)
    print(f'requested : {url} / {res.status_code}')
    
print(time.time() - s)
{% endraw %}
{% endhighlight %}

This is simple syncrounous code to fetch data from urls in the list. Most of delays(in my experience, more than 90%...) in scraping process causes on request/response. You really don't need to worry about speed of parsing, or else. So let's only focus about requesting, and receiving result.

{% highlight shell %}
{% raw %}
requested : https://www.python.org/ / 200
requested : https://www.javascript.com/ / 200
requested : https://www.amazon.com/ / 200
requested : https://www.netflix.com/ / 200
requested : https://www.scala-lang.org/ / 200
6.307806968688965
{% endraw %}
{% endhighlight %}

It takes about 6.3 second to fetch urls.


{% highlight python %}
{% raw %}
import asyncio
import aiohttp
import requests
import time

urls = [
    'https://www.python.org/',
    'https://www.javascript.com/',
    'https://www.amazon.com/',
    'https://www.netflix.com/',
    'https://www.scala-lang.org/'
]


async def fetch(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as res:
            print(url + ' / ' + str(res.status))

s = time.time()
print('start: ' + str(s))

tasks = [fetch(url) for url in urls]
loop = asyncio.get_event_loop()
loop.run_until_complete(asyncio.wait(tasks))

print('end: ' + str(time.time() - s))
{% endraw %}
{% endhighlight %}


{% highlight shell %}
{% raw %}
start: 1532329882.60325
https://www.python.org/ / 200
https://www.javascript.com/ / 200
https://www.amazon.com/ / 200
https://www.scala-lang.org/ / 200
https://www.netflix.com/ / 200
end: 2.1449880599975586
{% endraw %}
{% endhighlight %}

This logic took 2.1 sec for response. What makes the difference?
