---
layout: post
title:  Reloading React project, Javascript to Typescript
date:   2018-08-15
description: 
tags:
- react
- redux
- javascript
- typescript
permalink: react-js-to-ts
---


Typescript is a superset of Javascript that compiles to pure Javascript, like CoffeeScript or Dart. When I first heard of this, thought as 'Why do we need to learn a new thing, which will create javascript file? Is it better to make javascript directly?'. But now it has been standard development language of `Angular` framework, become official programming language in Google, and it is being spread more widely in various projects. Yeah, there should be a reason.


## Why Typescript?
Javascript has been started as script language, and usage was limited for handling DOM activities, supporting html in web page. Now, client-side application has been very complicated by rising of client-side rendering, and it handles not only rendering, but also flow of data received from server. More than that, NodeJS allowed javascript language to be used in system level. 
Now, there are lots of large-scale application which created by javascript only, and they found it is missing several features necessary to be able to productively write and maintain these system.

TypeScript...
...
...

Also, specification of javascript is being changed, such as ES6, ES7, and it is quite a pain to followup every year. Thanks to Typescript developers, its transpiler is being update to support newest features of javascript, so you don't need to think about this deeply.


## React-redux sample
https://redux.js.org/advanced/exampleredditapi

This is one of famouse asyncrounous web application example. This includes lot of important features for developing actual React web apps such as `action` and `reducer` from `Redux` pattern, middleware usage, and how to handle asyncrounous process like http connection. 

![Screenshot](/assets/post_img/react-js-to-ts/reddit-api-sample.png)

Process is simple, it has dropdown which has 'reactjs' and 'frontend', and it fetches the headlines which includes this word from Reddit when selected.

{% highlight javascript %}
{% raw %}

{% endraw %}
{% endhighlight %}



## Go as Typescript

![Screenshot](/assets/post_img/react-js-to-ts/js-to-ts.png)


## Reference
- https://www.typescriptlang.org/
- https://github.com/Microsoft/TypeScript-React-Conversion-Guide
- https://blogs.msdn.microsoft.com/somasegar/2012/10/01/typescript-javascript-development-at-application-scale/

