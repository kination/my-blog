Now, most of engineers around me are talking about Docker, whether they are using it right now or just heard about it. All I know about it is that Docker is a virtualized container that could include complete file system so it could run as like independent system. Really great point is, it would be totally simple to make same build environment in lots of other computers.
So to get sure, I worked to set a Django build docker system.

You can install with package tool

{% highlight html %}
$ sudo apt-get update
$ sudo apt-get install docker-engine
{% endhighlight %}

Or could get newest version with CURL
{% highlight html %}
$ curl -fsSL https://get.docker.com/ | sh
{% endhighlight %}
