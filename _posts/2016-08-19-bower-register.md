---
layout: post
title: "Publish package to Bower"
date: 2016-08-23 17:53:14
image: '/assets/img/'
description: "How to register own package to bower repository"
main-class: 'dev'
color:
tags:
- bower
- package manager
- front-end
categories:
twitter_text:
introduction: "How to register own package to bower repository"
---

Now, most of projects are implemented with group of open sources, and package managers are helping us to add these easily. If you are working on project using javascript, you will be familiar with `bower` and `npm`. I am working on AngularJS project currently, and most of library are being managed with `bower`.

As you know, not all of open sources are being developed as you want. Some of are not being updated with critical issues, though it needs to be fixed urgently. One way to handle this situation is download the file, fix the source, and add to your local project. But your library packages will be seperated(one in bower component, one in local folder), and it is not good to maintain source codes.

This is my current situation, so I tried to add my forked project on bower.
It is simple. First, I made fork project from original one.

![Screenshot](/assets/post_img/bower_register/fork_project.png)

I will register this on bower.

{% highlight bash %}
{% raw %}
$ bower register ion-range-slider-angularjs https://github.com/djKooks/ion-range-slider-angularjs.git
Registering a package will make it visible and installable via the registry.
Proceed (y/n)? y
{% endraw %}
{% endhighlight %}

Make sure to do not use name already registered. There are plenty of packages, so first check your name with `bower search`.

{% highlight bash %}
{% raw %}
$ bower search ion-range-slider
Search results:

    ion-range-slider https://github.com/IonDen/ion.rangeSlider.git
    ion-range-slider-angularjs https://github.com/djKooks/ion-range-slider-angularjs.git
    angular-foundation-range-slider https://github.com/pxdunn/RangeSlider.git
    foundation-range-slider-angular https://github.com/csaftoiu/foundation-range-slider-angular.git
{% endraw %}
{% endhighlight %}

Now your project has been enrolled. Before install this project, first look on your package.

{% highlight bash %}
{% raw %}
$ bower info ion-range-slider-angularjs
bower cached        https://github.com/djKooks/ion-range-slider-angularjs.git#1.0.4
bower validate      1.0.4 against https://github.com/djKooks/ion-range-slider-angularjs.git#*
...
{% endraw %}
{% endhighlight %}

When you look on line after `bower cached`, sometimes original package(here, it is  `ion.rangeslider-angularjs`) are being indicated. In this case, `bower install ion-range-slider-angularjs` will install package on cache instead of mine. So let's clean the cache first.

{% highlight bash %}
{% raw %}
$ bower cache clean
...
$ bower info ion-range-slider-angularjs
bower not-cached    https://github.com/djKooks/ion-range-slider-angularjs.git#*
bower resolve       https://github.com/djKooks/ion-range-slider-angularjs.git#*
bower checkout      ion-range-slider-angularjs#1.0.4
bower resolved      https://github.com/djKooks/ion-range-slider-angularjs.git#*

{
  name: 'ion-range-slider-angularjs',
  version: '1.0.4',
  homepage: 'https://github.com/djKooks/ion-range-slider-angularjs.git',
  'original authors': [
    'Geoffrey Bauduin <bauduin.geo@gmail.com>'
  ],
  description: 'fork project of ion.rangeslider-angularjs',
  main: 'ionic-range-slider.js',
  keywords: [
    'angularjs',
    'angular',
    'ionic',
    'slider',
    'rangeslider',
    'ion.rangeslider'
  ],
  license: 'MIT',
  ignore: [
    'node_modules',
    'bower_components',
    'test',
    'tests'
  ],
  dependencies: {
    'ion.rangeslider': '~2.1.2'
  }
}

Available versions:
  - 1.0.4
  - 1.0.3
  - 1.0.2
  - 1.0.1
  - 1.0.0
{% endraw %}
{% endhighlight %}

Your package has been setup well, so now fix issue on fork project.

After clearing issue, see bower.json on your project and update version number. You can fix package information like description, publisher, etc. here. What you see in `bower info` comes from here. Also, you need to setup tag after update, or bower package will still keep looking on old version.

```
- Your package should use semver Git tags. The v prefix is allowed.
- Your package must be publically available at a Git endpoint (e.g., GitHub). Remember to push your Git tags!
```
This is mentioned on official bower web page.

I fixed the issue, updated version to 1.0.5. Now updating tag:

{% highlight bash %}
{% raw %}
$ git tag 1.0.5
$ git push origin 1.0.5
Username for 'https://github.com': djkooks
Password for 'https://djkooks@github.com':
Total 0 (delta 0), reused 0 (delta 0)
To https://github.com/djKooks/ion-range-slider-angularjs.git
 * [new tag]         1.0.5 -> 1.0.5
{% endraw %}
{% endhighlight %}

Clear the cache, and check package info to see version has been changed successfully.

{% highlight bash %}
{% raw %}
$ bower cache clean
$ bower info ion-range-slider-angularjs
bower not-cached    https://github.com/djKooks/ion-range-slider-angularjs.git#*
bower resolve       https://github.com/djKooks/ion-range-slider-angularjs.git#*
bower checkout      ion-range-slider-angularjs#1.0.5
bower resolved      https://github.com/djKooks/ion-range-slider-angularjs.git#1.0.5

{
  name: 'ion-range-slider-angularjs',
  version: '1.0.5',
  homepage: 'https://github.com/djKooks/ion-range-slider-angularjs.git',
  'original authors': [
...
{% endraw %}
{% endhighlight %}

Now bower package with issue cleared has been ready to launch.

I registered to handle known issue quickly, but some of people could use this to customize open source package to make it fit on their working project. Just always think to clear cache, and setup tag when updating. I have wasted so much time because of this. Hope nobody being stuck like me.
