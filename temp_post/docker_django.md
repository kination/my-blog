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

If install success, run and verify it.

{% highlight html %}
$ sudo service docker start
$ sudo docker run hello-world
{% endhighlight %}

//image

If you can see this, you have completed first step. But there are few more options to do.

Docker daemon binds to Unix socket(not TCP port), and this is owned by user 'root'. So docker daemon is working with 'sudo' command. If you want to make docker run without using it, you need to setup a group called 'docker' and add users in it.

{% highlight html %}
$ sudo usermod -aG docker 'yourname'
{% endhighlight %}

Now try like this to see it works.

{% highlight html %}
$ docker run hello-world
{% endhighlight %}

...
