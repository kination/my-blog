---
layout: post
title:  "Construct front-end project with gulp"
date:   2017-07-04
tags:
- frontend
- javascript
- gulp
permalink: 
description: The way of using gulp for managing front-end project resources.
---

As front-end projects are being complecated, there are many process to think on working such as compressing, linting, transpiling, and more. These are not mandatory stuffs, but it helps managing your project, and upgrade its performance.
Surely, there are several tools for this such as `Grunt` or `Gulp`, and you could set an option in `Webpack` to make these process. I'll look on `Gulp` here.

![Screenshot](/assets/post_img/frontend_with_gulp/gulp_and_grunt.png)


### Why Gulp?

Gulp is a command line task runner based on Node.js. It runs defined tasks and manages processes automatically.

The function `Grunt` and `Gulp` offers are mostly same. It makes process which I mentioned above working during build process. There are lots of blogs which explained about comparison, but the big reason I choose `Gulp` in my project is, its configuration 'seems more easy'.

Settings in `Grunt` is based on JSON, while `Gulp` is using javascript. These are simple usage.

------- Grunt -------
{% highlight javascript %}
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

You can install gulp package through `npm`.
{% highlight shell %}
{% raw %}
$ npm install -g gulp-cli          // install gulp cli tool
$ cd /PATH/TO/YOUR/PROJECT
$ npm install gulp --save-dev      // install gulp in development dependencies
{% endraw %}
{% endhighlight %}

And create a `gulpfile.js` in root directory of your project.
{% highlight javascript %}
{% raw %}
var gulp = require('gulp');

gulp.task('default', function() {
  // place code for your default task here
});
{% endraw %}
{% endhighlight %}

This is a basic form of gulpfile. Now try 
{% highlight shell %}
{% raw %}
$ gulp
[12:15:04] Using gulpfile ~/PATH/TO/YOUR/PROJECT/gulpfile.js
[12:15:04] Starting 'default'...
[12:15:04] Finished 'default' after 123 μs
{% endraw %}
{% endhighlight %}

Command `gulp` will run task `default` in gulp file. If you want to use other task, you could make it run by using `gulp TASK-NAME`. 
Now you are ready. It does nothing because you didn't setup any process in here. You will now add gulp submodules in here by your purpose.


### Compress

Compressing(or minifing) html/js files in your project improves performance, by reducing file load time. Let's make this work first. First you need to add minifier modules.
{% highlight shell %}
{% raw %}
$ npm install gulp-uglify --save-dev
$ npm install gulp-minify-html --save-dev
{% endraw %}
{% endhighlight %}

and add these in your gulp file.
{% highlight javascript %}
{% raw %}
var minify_js = require('gulp-uglify');
var minify_html = require('gulp-minify-html');

gulp.task('minifyJs', function () {
    return gulp.src('JS-PATH/**/*.js')
        .pipe(minify_js())
        .pipe(gulp.dest('COMPRESSED-JS-PATH'));
});

gulp.task('minifyHtml', function () {
    return gulp.src('HTML-PATH/**/*.html')
        .pipe(minify_html())
        .pipe(gulp.dest('COMPRESSED-HTML-PATH'));
});
{% endraw %}
{% endhighlight %}

In gulp, it uses `Stream` logic with [pipe](https://nodejs.org/api/stream.html#stream_readable_pipe_destination_options) method for passing data. It makes multiple tasks being work together, and improve performance of gulp.

By code above, task 'minifyJs' and 'minifyHtml' has been added for compressing .js file and .html files. It sends all files in `gulp.src` to minifing module, and throws result files to locations in `gulp.dest`.

{% highlight shell %}
{% raw %}
$ gulp minifyJs
[12:30:26] Using gulpfile ~/PATH/TO/YOUR/PROJECT/gulpfile.js
[12:30:26] Starting 'minifyJs'...
[12:30:30] Finished 'minifyJs' after 4.25 s
$ gulp minifyHtml
[12:30:49] Using gulpfile ~/PATH/TO/YOUR/PROJECT/gulpfile.js
[12:30:49] Starting 'minifyHtml'...
[12:30:49] Finished 'minifyHtml' after 546 ms
$ 
{% endraw %}
{% endhighlight %}

If you want to make it all running by single command, add new task like this.
{% highlight javascript %}
{% raw %}
gulp.task('release', ['minifyJs', 'minifyHtml']);
{% endraw %}
{% endhighlight %}

Try this.
{% highlight shell %}
{% raw %}
$ gulp release
[20:55:29] Starting 'minifyJs'...
[20:55:29] Starting 'minifyHtml'...
[20:55:33] Finished 'minifyJs' after 4.19 s
[20:55:34] Finished 'minifyHtml' after 4.5 s
[20:55:34] Starting 'release'...
[20:55:34] Finished 'release' after 277 μs
$ 
{% endraw %}
{% endhighlight %}

There are several useful options in `gulp-uglify` module. For example, if you want to get rid of all `console.log` stuff in minify process, add parameter in module like this.
{% highlight javascript %}
{% raw %}
gulp.task('minifyJs', function () {
    return gulp.src('JS-PATH/**/*.js')
        .pipe(minify_js({
            compress: {
                drop_console: true
            }
        }))
        .pipe(gulp.dest('COMPRESSED-JS-PATH'));
});
{% endraw %}
{% endhighlight %}


### Linting

'Linting' in modern programming language usually means keeping your code quality(there are other meaning in wikipedia though...), so make code developer to follow the writing standard and remove potential errors. Especially javascirpt, because it is not strictable, it is hard to keep code clean, and that's why linting is important here.

There are several options for linting for javascript. I'll select [eslint](http://eslint.org/) here. Surely, there are a plugin for gulp. Install it first...

{% highlight shell %}
{% raw %}
$ npm install --save-dev gulp-eslint
$ npm install --save-dev eslint-plugin-angular  # only need in angularjs project
{% endraw %}
{% endhighlight %}

and add task.
{% highlight javascript %}
{% raw %}
var eslint = require('gulp-eslint');
gulp.task('esLint', function () {
    return gulp.src('JS-PATH/**/*.js')
        .pipe(eslint())
        .pipe(eslint.format());
});
{% endraw %}
{% endhighlight %}

`eslint()` module checks lint in target directory, and shows the result by `eslint.format()`. You could see the result like this.
{% highlight shell %}
{% raw %}
$ gulp esLint
[14:17:42] Using gulpfile ~/PATH/TO/YOUR/PROJECT/gulpfile.js
[14:17:42] Starting 'esLint'...
...
/JS/PATH/xxx.js
 9:20  error  Strings must use singlequote                   quotes
 9:61  error  '$filter' is defined but never used            no-unused-vars
68:1   error  Expected indentation of 20 spaces but found 0  indent
79:34  error  Strings must use singlequote                   quotes
79:56  error  'value' is defined but never used              no-unused-vars
86:14  error  Missing semicolon                              semi
...
✖ 167 problems (162 errors, 5 warnings)
  107 errors, 0 warnings potentially fixable with the `--fix` option.
$ 
{% endraw %}
{% endhighlight %}

If you want to add some function which only works when there is no lint error, add pipe for `failAfterError`. This will show 'success!' alert only when lint process is passed for all files in source.

{% highlight javascript %}
{% raw %}
var eslint = require('gulp-eslint');
gulp.task('esLint', function () {
    return gulp.src('JS-PATH/**/*.js')
        .pipe(eslint())
        .pipe(eslint.format())
        .pipe(eslint.failAfterError());

gulp.task('lintResult', ['testLint'], function () {
    alert('success!');
});
{% endraw %}
{% endhighlight %}


### Remove cache

When once static files being load, browser will keep it as a cache data for a pretty long time. So though you update your code into server, your browser could show you the cached templates instead of updated ones. Most of browser will do it like this, for performance purpose. It can make loading quickly, but could be a big problem on your service. 
Add custom parameter in request query like this is the usual way to solve the problem. In this case, parameter 'ver=1.0' is attached in it.
```
http://PATH/OF/FILE/abcd.js?ver=1.0
```
This is one of example. You could attach random code, or timestamp instead of version. But you can't put on this in every path every time, so let's make it done in gulp process.

{% highlight shell %}
{% raw %}
$ npm install --save-dev gulp-cache-bust
{% endraw %}
{% endhighlight %}

I will add this process inside minify task.
{% highlight javascript %}
{% raw %}
var cache_bust = require('gulp-cache-bust');

gulp.task('minifyJs', function () {
    return gulp.src('JS-PATH/**/*.js')
        .pipe(minify_js())
        .pipe(cache_bust({
            type: 'timestamp'
        }))
        .pipe(gulp.dest('COMPRESSED-JS-PATH'));
});

gulp.task('minifyHtml', function () {
    return gulp.src('HTML-PATH/**/*.html')
        .pipe(minify_html())
        .pipe(cache_bust({
            type: 'timestamp'
        }))
        .pipe(gulp.dest('COMPRESSED-HTML-PATH'));
});

{% endraw %}
{% endhighlight %}
Very simple! Just add `gulp-cache-bust` module with parameter. This is the case using timestamp, but you could use other option such as MD5 data, or just constant value like version.


### And...

The function I commented here is really just a part of gulp environment. There are tons of more functions you can do inside gulp process which will help your project. You could find it [here](http://gulpjs.com/plugins/).

### Resources

http://gulpjs.com/
https://www.keycdn.com/blog/gulp-vs-grunt/
https://medium.com/@preslavrachev/gulp-vs-grunt-why-one-why-the-other-f5d3b398edc4
https://ics.media/entry/9199


