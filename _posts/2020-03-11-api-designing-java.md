---
layout: post
title:  API design for Java module
date: 2020-02-11
tags:
- java
- api
- design
permalink: api-design-java-module
---


![Screenshot](/assets/post_img/api-design-java-module/java-logo-icon.png)

## Designing Java API
API is a gate, entrance of the service for the user. Developer who is willing to use the service will research API design carefully, and make implementation plan based on this. It means good API design will make user's code more clear and simple, and let them keep use our service consistently.

But we couldn't sure how the user will integrate this, so cannot satisfy everyone. And once you've release officially, it is far more difficult to remove it. That would be the reason every document about API design saids 'make it simple'. In other words, it should be 'easy to understand, use, extend'.

> - Should be intuitive enough
> - Can be used without strict condition(if possible)
> - Function of API can be extend easily

 And, if possible, we should think of performance too, to not harm user's service.

Based on these suggestions, let's think about how to make Java API. As you know there are tons of guide about  good API design, and most of them are defined more than 10 years. These are the points which I'm bit more focusing on when creating Java APIs.


## Make it easy
About ways to make it easy, we can think of 'naming' first. Name should be self-explanatory as possible. It should describe what is doing, and avoid abbreviations.

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

Also, names in same package should follow consistent naming and convention. This seems easy, but hard to keep while your service is growing up. You can even find on several standard Java APIs.

```java
Stream.of(2, 3, 4, 5).skip(2)  // 3, 4, 5
```

`skip(long n)` in `Stream` API will skip n item of the stream data, and

```java
Stream.of(1, 2, 3, 4, 5).dropWhile(n -> n % 2 == 1) // 2, 3, 4, 5
```

`dropWhile(Predicate<? super T> predicate)` is returning remaining element after dropping a subset of elements that match the given predicate.
Though both offers similar operation, it has different naming. Of course, from functional point, they have proper name. But in developer's side, if they only knows `skip`, they will think name `skipWhile` first when they need to search this kind of API.


## Make things immutable
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


## Static factory method
This is to call static method inside class for getting class instance, instead of calling `new`. It can be find very easily, such as:

```java
Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5);
String value = String.valueOf(1);
```

It is one of popular design pattern, and it offers several advantages:

> - You can give meaningful name, more than just 'new'
> - You can make returning type more flexible, same type that implements the method(s), a subtype, and also primitives
> - Static factory methods can be controlled-instanced methods. You can make class as Singleton more easily.


## Null handling
Returning null is a bad option. If user didn't setup check logics, they will see friendly `NullPointerException`. Though they prepared for that case, code will not look good. Let's think of case that `MyClass.getMyHobbies` will return the list of string.

```java
MyClass myClass = MyClass.create(...);
List<String> hobbies = myClass.getMyHobbies();
if (hobbies != null) {

}
```

For safety, user should check null every time when calling this.

One way is to return empty collection for this case. Java offers several API for this:
```java
return Collections.emptyList();

return List.of(); // for Java 9(or above) user
```

If you are targeting Java 8, use `Optional` can be another option. It is a container object which may or may not contain a non-null value. These kinds of object are being applied on many other modern programming language(such as `Option` in Rust, or `Maybe` in Haskell), for avoiding null exceptions.

Brian Goetz, Java Language Architect at Oracle, explained about this:

> ...Our intention was to provide a limited mechanism for library method return types where there needed to be a > clear way to represent “no result”, and using null for such was overwhelmingly likely to cause errors.

We can return `Optional` instead of 

```java
class MyClass {
  public static MyClass create(...) {
    return new MyClass(...);
  }

  Optional<List<String>> getMyHobbies() {
    // generate the list...
    if (emptyList) {
      return Optional.empty();
    }

    return Optional.of(listObject);
  }
}
```

This will throw `Optional` container of list. User can receive this by:
```java
MyClass myClass = MyClass.create(...);
List<String> hobbies = myClass.stream()
               .flatMap(Optional::stream)
               .collect(Collectors.toList());
```

Or instead of using `empty` and `of` by status, you can use `ofNullable`. It will check the status of list and return, whether it is null or not.

```java
Optional.ofNullable(listObject);
```


## Reference
* <http://www.cs.bc.edu/~muller/teaching/cs102/s06/lib/pdf/api-design>
* <http://tutorials.jenkov.com/api-design/index.html>
* <https://www.artima.com/interfacedesign/contents.html>
* <https://medium.com/97-things/lets-make-a-contract-the-art-of-designing-a-java-api-854441cd42f5>
* <https://www.codebyamir.com/blog/stop-returning-null-in-java>
* <https://www.slideshare.net/MiroCupak/the-good-the-bad-and-the-ugly-of-java-api-design-179537151>
* <https://jonathangiles.net/presentations/java-api-design-best-practices/>
