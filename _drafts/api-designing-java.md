---
layout: post
title:  API designing for Java module
date: 2020-02-11
tags:
- java
- api
- design
permalink: 
---



## 
API is a gate, entrance of the service for the user. Developer who is willing to use the service will research API design carefully, and make implementation plan based on this. It means good API design will make user's code more clear and simple, and let them keep use our service consistently.

But we couldn't sure how the user will integrate this, so cannot satisfy everyone. And once you've release officially, it is far more difficult to remove it. That would be the reason every document about API design saids 'make it simple'. In other words, it should be 'easy to understand, use, extend'.

- Should be intuitive enough
- Can be used without strict condition(if possible)
- Function of API can be extend easily




## Designing Java API
Based on these suggestions, let's think about how to make Java API.


### Make it simple
About ways to make it simple, we can think of 'naming' first. Name should be self-explanatory as possible. It should describe what is doing, and avoid abbreviations.

```java
Car myCar = new Car();

/**
assume it needs method to return current mileage of car
*/

myCar.currentMileage(); // good naming

myCar.currMil();  // bad

myCar.abc();  // horrible
```

No doubt that documentation is important, but most developers will get information directly from code before reading long document. Intuitive naming will help developer reduce time searching document for understanding the function.


### About performance
Making return type as `Immutable` can give plus for this. As you know, making 'mutable' object will require memory alloc/free which can harm performance. Though making object immutable will take more memory, I think you don't needs to worry in most of cases.

It's a bit old story, but for example, common Java function `Component.getSize()` is returning mutable `Dimension` object. It means JVM needs to allocate memory for this every time when function called. Though user only need to know the size of component, it causes massive useless memory alloc/free process.

Java has several APIs which makes things immutable. 

```java
class Cart {
  List<String> currentItems;
  // ...
  setItem(String item) {
    currentItems.add(item);
  }

  List<String> getCurrentItems() {
    
    return Collections.unmodifiableList(currentItems);  // preferred
    return currentItems;  // bad
  }

}
```


## From Java 8
There are features only in Java 8 or later which can help API stable:

- Lambda
- Method Reference
- Stream
- Optional

### Lambda
Lambda provides clear way to represent method interface using expression. It makes function to be created without name, and no need to be belong to any classes.
  







## Reference
* <http://www.cs.bc.edu/~muller/teaching/cs102/s06/lib/pdf/api-design>
* <https://jarthorn.github.io/EclipseCon2014/java8/>
* <http://wiki.netbeans.org/API_Design>
* <https://medium.com/better-practices/design-apis-like-you-design-user-experience-a7adeb2ee90f>
* <https://www.slideshare.net/MiroCupak/the-good-the-bad-and-the-ugly-of-java-api-design-179537151>
* <https://jonathangiles.net/presentations/java-api-design-best-practices/>
