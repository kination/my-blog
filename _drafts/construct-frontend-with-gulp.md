---
layout: post
title:  "Construct front-end project with gulp"
date:   2017-06-24
categories:
- frontend
- javascript
- gulp
permalink: 
description: The way of using gulp for managing front-end project resources.
---

As front-end projects are being complecated, there are many process to think on working such as compressing, linting, transpiling, and more. These are not mandatory stuffs, but it helps managing your project, and upgrade its performance.
Surely, there are several tools for this such as `Grunt` or `Gulp`, and you could set an option in `Webpack` to make these process. I'll look on `Gulp` here.

### Why Gulp?

Gulp is a command line task runner based on Node.js. It runs defined tasks and manages processes automatically.

The function `Grunt` and `Gulp` offers are mostly same. It makes process which I mentioned above working during build process. There are lots of blogs which explained about comparison, but the big reason I choose `Gulp` in my project is, its configuration 'seems more easy'.

Settings in `Grunt` is based on JSON, while `Gulp` is using javascript. These are simple usage.

------- Grunt -------
{% highlight json %}
{% raw %}
grunt.initConfig({
  uglify: {
    my_target: {
      files: {
        'dest/output.min.js': ['src/input.js']
      }
    }
  }
});
{% endraw %}
{% endhighlight %}

------- Gulp -------
{% highlight javascript %}
{% raw %}
gulp.task('uglifyJs', function () {
    return gulp.src('src/input.js')
        .pipe(js_minify())
        .pipe(gulp.dest('dest/output.min.js'));
});
{% endraw %}
{% endhighlight %}

It seems okay while it has only few configuration to do, but the more settings being complecated, Grunt will have very unattractive form. Maybe this is subjective suggestion, but maybe this is the reason why I feel Gulp more comportable.

[Grunt focuses on configuration, while Gulp focuses on code](https://medium.com/@preslavrachev/gulp-vs-grunt-why-one-why-the-other-f5d3b398edc4)

Because I'm used on looking code more than config file, it could explain my suggestion.
There are some more comparing point such as performance, supporting 3rd party module, and more. But what I think is, both tool are good enough, so it is developer's choice, and I select `Gulp`.


### Setup




### Compress



### Compile



### Linting




### Remove cache




### Resources

https://www.keycdn.com/blog/gulp-vs-grunt/
https://medium.com/@preslavrachev/gulp-vs-grunt-why-one-why-the-other-f5d3b398edc4


