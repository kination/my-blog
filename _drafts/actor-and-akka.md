---
layout: post
title:  "Actor model, and implementation with Akka"
date:   2018-03-28
description: 
tags:
- actor
- akka
- java
- scala
permalink: actor-model-and-akka
---

First time I heard of Akka, was the blog post written about case of Twitter, that they implemented their service based on this to handle massive twit data in real-time. I've had some experiences developing web server, and currently having interests of concurrency issue, and it makes me looking on this.

## Actor model
As you could see in other documents, theory of Actor has been more than 40 years. But before early 21 century, we usually did not need to think about concurrency or multi-threading deeply, because there were no smartphones which causes heavy requests and no big data to handle in real-time. Now, massive internet services like Facebook and Twitter needs to handle millions of requests per minute, and so they are finding/developing platform to satisfy this.

Actor is an object, composed with behavior, state, mailbox. Behavior is the job which Actor need to do, state is the current state of Actor, and mailbox is to send/receive the message from other Actors. 

![Screenshot](/assets/post_img/actor-model-and-akka/actor-diagram.png)

Yes, this has some similarity with `Thread`. In programming we could use these to run process asyncrounously. The point which makes it different is, each of them cannot access to other actor's memory or state. Each of them are only doing their job without sharing other's area and every communication is being done by mailbox.
As there is no sharing resources, there is no state like `lock` in `Thread` because it does not need to wait for waiting memory from others. It just sends the message without blocking so you can get benefits of multi-thread programming without worring about the problems you can face with 'sharing' issue.


## Akka
`Akka` is a toolkit + runtime for building highly concurrent, distributed, and fault-tolerant applications which is based on actor model. It is OSS(Open Source Software), and currently managed by `Lightbend`. You could find this in lots of module for handling concurrency and distribution web service. Core Akka is implemented in Scala, but they also offer Java SDK for development.

Before creating `Hello world` stuff, let’s check the relationships between actors you create in your code and what happens below the modules.

An actor in Akka always belongs to a parent. If you generate new actor in your code, it will be created as child of `/user` actor, which are being created by system.
When you setup the Akka library and start the code, it will create three actors including `/user`. The names of these built-in actors contain guardian because they supervise every child actor in their path.

![Screenshot](/assets/post_img/actor-model-and-akka/actor-top-tree.png)

- The one in top is root guardian. It is the parent of the all actors in system, and will be removed after system is destroyed.
- `/user` is parent actor for all user created actors. Every actor you create will have the path '/user/' prepended to it.
- `/system` is the system guardian.


## Akka implementations
The 'quickstart' code used here is from official page of Lightbend. You can download full project from [here](https://developer.lightbend.com/start/?group=akka&project=akka-quickstart-java).

Now let's see the main file. It starts with creating actors.

{% highlight java %}
{% raw %}
final ActorSystem system = ActorSystem.create("helloakka");

final ActorRef printerActor = system.actorOf(Printer.props(), "printerActor");
final ActorRef howdyGreeter = system.actorOf(Greeter.props("Howdy", printerActor), "howdyGreeter");
final ActorRef helloGreeter = system.actorOf(Greeter.props("Hello", printerActor), "helloGreeter");
final ActorRef goodDayGreeter = system.actorOf(Greeter.props("Good day", printerActor), "goodDayGreeter");
...
{% endraw %}
{% endhighlight %}

First line is to create container which actors will be placed. Now actor instance create by `system.actorOf` will be child of `user` node which we saw in diagram above. In this code, it creates three 'Greeter' actor and one 'Printer' actor. Now we need to see what 'Printer' and 'Greeter' is for.

{% highlight java %}
{% raw %}
public class Printer extends AbstractActor {
  static public Props props() {
    return Props.create(Printer.class, () -> new Printer());
  }

  static public class Greeting {
    public final String message;

    public Greeting(String message) {
      this.message = message;
    }
  }

  private LoggingAdapter log = Logging.getLogger(getContext().getSystem(), this);

  public Printer() {
  }

  @Override
  public Receive createReceive() {
    return receiveBuilder()
        .match(Greeting.class, greeting -> {
            log.info(greeting.message);
        })
        .build();
  }
}
{% endraw %}
{% endhighlight %}

'Printer' extends `akka.actor.AbstractActor` class to implement actor instance. Implemented method `createReceive` will be called when message from other instance is received. 
The role of this class is to create log instance and show the message received from 'Greeter' class.

{% highlight java %}
{% raw %}
public class Greeter extends AbstractActor {
  static public Props props(String message, ActorRef printerActor) {
    return Props.create(Greeter.class, () -> new Greeter(message, printerActor));
  }

  static public class WhoToGreet {
    public final String who;

    public WhoToGreet(String who) {
        this.who = who;
    }
  }

  static public class Greet {
    public Greet() {
    }
  }

  private final String message;
  private final ActorRef printerActor;
  private String greeting = "";

  public Greeter(String message, ActorRef printerActor) {
    this.message = message;
    this.printerActor = printerActor;
  }

  @Override
  public Receive createReceive() {
    return receiveBuilder()
        .match(WhoToGreet.class, wtg -> {
          this.greeting = message + ", " + wtg.who;
        })
        .match(Greet.class, x -> {
          printerActor.tell(new Greeting(greeting), getSelf());
        })
        .build();
  }
}
{% endraw %}
{% endhighlight %}

'Greeter' class also extends `akka.actor.AbstractActor` class. You can see constructor in here requests two parameter, 'message(String)' and 'printerActor(ActorRef)'. 
As you can see in `createReceive` method, it returns `receiveBuilder` to define the behavior depends on the message received. It expects two types of messages, 'WhoToGreet' and 'Greet'. The former will update the greeting state of the Actor with received message string while the latter one will trigger a sender of updated greeting message to the Printer Actor.

Now, let's see the remain codes in main class.

{% highlight java %}
{% raw %}
...
howdyGreeter.tell(new WhoToGreet("Akka"), ActorRef.noSender());
howdyGreeter.tell(new Greet(), ActorRef.noSender());

howdyGreeter.tell(new WhoToGreet("Lightbend"), ActorRef.noSender());
howdyGreeter.tell(new Greet(), ActorRef.noSender());

helloGreeter.tell(new WhoToGreet("Java"), ActorRef.noSender());
helloGreeter.tell(new Greet(), ActorRef.noSender());

goodDayGreeter.tell(new WhoToGreet("Play"), ActorRef.noSender());
goodDayGreeter.tell(new Greet(), ActorRef.noSender());
...
{% endraw %}
{% endhighlight %}

`ActorRef.tell` method is to putting the message into Actor's mailbox. By this codes, it calls `createReceive` implemented in each ActorRef class. As we see above, first line will add "Akka" string in greeting message, while second will send greeting message to 'Printer' class to display this via `Log` method, and so on.


## Message flow

![Screenshot](/assets/post_img/actor-model-and-akka/hello-akka-messages.png)

This is the process flow which happens in code. Each of 'Greeting' instance generate, and sends message to 'Printer' instance to show this.

One thing to make sure is, all of actor is independent and work asyncrounously. This means it does not guarantee the order of process. Though we call the classes in order of 'howdyGreeter' -> 'helloGreeter' -> 'goodDayGreeter', the printed result can be changed by process handling duration. 

This is the first term...
{% highlight shell %}
{% raw %}
$ gradle run

> Task :run
>>> Press ENTER to exit <<<
[INFO] [04/11/2018 19:43:02.692] [helloakka-akka.actor.default-dispatcher-4] [akka://helloakka/user/printerActor] Hello, Java
[INFO] [04/11/2018 19:43:02.693] [helloakka-akka.actor.default-dispatcher-4] [akka://helloakka/user/printerActor] Good day, Play
[INFO] [04/11/2018 19:43:02.693] [helloakka-akka.actor.default-dispatcher-4] [akka://helloakka/user/printerActor] Howdy, Akka
[INFO] [04/11/2018 19:43:02.693] [helloakka-akka.actor.default-dispatcher-4] [akka://helloakka/user/printerActor] Howdy, Lightbend
<=========----> 75% EXECUTING [19s]
...
{% endraw %}
{% endhighlight %}

and this is second term...
{% highlight shell %}
{% raw %}
$ gradle run

> Task :run
>>> Press ENTER to exit <<<
[INFO] [04/11/2018 19:43:25.298] [helloakka-akka.actor.default-dispatcher-3] [akka://helloakka/user/printerActor] Howdy, Akka
[INFO] [04/11/2018 19:43:25.299] [helloakka-akka.actor.default-dispatcher-3] [akka://helloakka/user/printerActor] Howdy, Lightbend
[INFO] [04/11/2018 19:43:25.299] [helloakka-akka.actor.default-dispatcher-3] [akka://helloakka/user/printerActor] Hello, Java
[INFO] [04/11/2018 19:43:25.299] [helloakka-akka.actor.default-dispatcher-3] [akka://helloakka/user/printerActor] Good day, Play
<=========----> 75% EXECUTING [33m 14s]
...
{% endraw %}
{% endhighlight %}


Okay, this is the first access with Akka.

## Reference

* https://doc.akka.io
* https://developer.lightbend.com/guides/
