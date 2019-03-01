---
layout: post
title:  Reactive programming, and Rx
date:   2019-01-29
description: What is reactive programming, and rx(reactive extension)
tags:
- reactive
- rx
- rxpy
permalink: reactive-programming-rx
---

*This is summary which I presented in Lighting Talk on developer meeting.*

Reactive programming is not a new thing. It is one of programming paradigms, oriented around data flows and the propagation of change. In reactive system, all of events inside applications are going through this stream, including request, data change, input event, and more.


## vs Imperative
We can think imperative programming as the opposite word of reactive programming. 

Imperative programming should the most popular way to code until now. In this, code will be executed in order of code lines and flow/values are being changed by state of program.

Check the difference of basic flow. This describes simple process, to add value 2 to each data inside list:

![Screenshot](/assets/post_img/reactive-programming-rx/imperative-reactive.png)

In imperative way, process will go on order and input data will be blocked from other process until it finished. It is synchronous flow, and we can expect the order of data.

But in reactive way, data in list is going into the stream. During the flow, it will be processed and subscribed by system during the flow.

See some other thing. This is just a pseudo code, but good to understand the difference:
```javascript
// 1. imperative programming
b = 10
c = 20
a = b + c
console.log(a) // 30
c = 40
console.log(a) // 30

// 2. reactive programming
b = 10
c = 20
a = b + c
console.log(a) // 30
c = 40
console.log(a) // 50
```

It is easy to expect the result of example in case 1. No doubt that result of 'a' will not be change because sum of 2 value has been done before 'c' is changed. But in case 2, all data is in the stream, and can be effected during process. Now you can expect, it would make debugging more difficult.


## What is good for?
Though it can make system more complicate, it has several pros which will make you to think about it.

Most of server systems in current days are facing with heavy traffics, as well as web/mobile apps treating variable UI events and interactive actions. By this change, all kinds of programming need to think about asynchronous handling. Reactive programming is making system events running in streams, and subscribe these asynchronously. 

For several years, Netflix has changed their Java-based system into Reactive model. They made ‘observable’ service layer on the system to make it working asynchronous, safely use concurrency methods to cover heavy traffic.


## Rx - Reactive eXtension
Actually, there are already lot of ways to make things do this, like using `Future` in Java, or `Promise` in JavaScript. But you can make it more solid with more simple way, by using `Rx` module.

![Screenshot](/assets/post_img/reactive-programming-rx/reactive-extension.png)

`Rx` is shorten of Reactive eXtension, and it offers tool for reactive coding. It includes library for composing asynchronous and event-based programs using observable sequences and LINQ-style query operators, and make user free from considering low-level stuffs, such as threading, sync/async, concurrency.

Main features are:
> - Observable: Create data stream object
> - Observer: Create object which observes data stream created by Observable.
> - Subscribe: Connector method to data stream

...and the basic flow are:
> 1. Define a method that does something with the return value from the asynchronous call. This method is part of the observer.
> 2. Define the asynchronous data stream as an Observable.
> 3. Attach the observer to Observable object by subscribing it.
> 4. Go on with your business; whenever the call returns, the observer’s method will begin to operate on its return value or values — the items emitted by the Observable.

![Screenshot](/assets/post_img/reactive-programming-rx/rx-flow.png)

This describes the flow of data stream. Data is being attached by method in `Observable`, and by defining transforming method inside, it will return transformed method.

See more detail in below. You could find more in [official site](http://reactivex.io/).


## RxPy example
`RxPy` is extension for python. There are extension for most of popular languages(RxJava, RxJS, RxSwift...), so you can make it fit in your project. 

Sample I created here is the method to make list of string parsed from wikipedia document(search as 'New York City'), and find the sentence which includes string 'expensive'.

![Screenshot](/assets/post_img/reactive-programming-rx/rx-flow.png)

```python
KEY_STRING = 'expensive'

def rx_find_string_has_key(observer):
    str_list = open('new_york.txt', 'r').read().split('.')
    for string in str_list:
        if KEY_STRING in string:
            observer.on_next(string)

    observer.on_completed()
```
This method will split the raw text to list of sentences, and will filter the data by whether having string 'expensive' or not.


```python
class PrintObserver(Observer):
    '''
    An Observable calls this method whenever the Observable emits an item.
    This method takes as a parameter the item emitted by the Observable.
    '''
    def on_next(self, value):
        print('Received: {}\n'.format(value))

    '''
    An Observable calls this method to indicate that it has failed to generate the expected data or has encountered some other error.
    It will not make further calls to onNext or onCompleted.
    The onError method takes as its parameter an indication of what caused the error.
    '''
    def on_completed(self):
        print('Completed')

    '''
    An Observable calls this method after it has called onNext for the final time
    , if it has not encountered any errors.
    '''
    def on_error(self, error):
        print('Error: {}'.format(error))
```
This is observer part. It will define what to do with received value by event type. `Observer` has 3 interfaces(`on_next`, `on_completed`, `on_error`) inside. Other extensions are almost same, so you could refer definitions on comment above.

```python
source = Observable.create(rx_find_string_has_key)
source.subscribe(PrintObserver())
```
This is the activation part. It will create stream and source setter by using `Observable`. Data source will be sent by parameter `rx_find_string_has_key` defined above, and subscribe will be done by `PrintObserver()`. Check the result.

```
...
Received: [369] In 2019, the most expensive home sale ever in the United States achieved completion in Manhattan, at a selling price of US$238 million, for a 24,000 square feet (2,200 m2) penthouse apartment overlooking Central Park

Received: 95 billion, making it the world's most expensive hotel ever sold

Received: Four of the ten most expensive stadiums ever built worldwide (MetLife Stadium, the new Yankee Stadium, Madison Square Garden, and Citi Field) are located in the New York metropolitan area
...
```

If you like more 'lambda' style, you can do same thing like this:
```python
source = Observable.create(rx_find_string_has_key)
source.subscribe(
    on_next=lambda value: print('Received {}\n\n'.format(value)),
    on_completed=lambda: print('Completed'),
    on_error=lambda error: print('Error: {}'.format(error))
    )
```


## Reference
* <http://reactivex.io>
* <https://gist.github.com/staltz/868e7e9bc2a7b8c1f754>
* <https://medium.com/netflix-techblog/optimizing-the-netflix-api-5c9ac715cf19>
* <https://developers.redhat.com/blog/2017/06/30/5-things-to-know-about-reactive-programming/>
* <https://medium.com/@kevalpatel2106/what-is-reactive-programming-da37c1611382>

