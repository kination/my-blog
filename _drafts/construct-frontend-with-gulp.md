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

![Screenshot](/assets/post_img/frontend_with_gulp/gulp_and_grunt.png)

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

You can install gulp package through `npm`.
```
$ npm install -g gulp-cli          // install gulp cli tool
$ cd /PATH/TO/YOUR/PROJECT
$ npm install gulp --save-dev      // install gulp in development dependencies
```

And create a `gulpfile.js` in root directory of your project.
```
var gulp = require('gulp');

gulp.task('default', function() {
  // place code for your default task here
});
```

This is a basic form of gulpfile. Now try 
```
$ gulp
[12:15:04] Using gulpfile ~/PATH/TO/YOUR/PROJECT/gulpfile.js
[12:15:04] Starting 'default'...
[12:15:04] Finished 'default' after 123 μs
```

Command `gulp` will run task `default` in gulp file. If you want to use other task, you could make it run by using `gulp TASK-NAME`. 
Now you are ready. It does nothing because you didn't setup any process in here. You will now add gulp submodules in here by your purpose.

### Compress

Compressing(or minifing) html/js files in your project improves performance, by reducing file load time. Let's make this work first. First you need to add minifier modules.
```
$ npm install gulp-uglify --save-dev
$ npm install gulp-minify-html --save-dev
```

and add these in your gulp file.
```
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
```

In gulp, it uses `Stream` logic with [pipe](https://nodejs.org/api/stream.html#stream_readable_pipe_destination_options) method for passing data. It makes multiple tasks being work together, and improve performance of gulp.

By code above, task 'minifyJs' and 'minifyHtml' has been added for compressing .js file and .html files. It sends all files in `gulp.src` to minifing module, and throws result files to locations in `gulp.dest`.

```
$ gulp minifyJs
[12:30:26] Using gulpfile ~/PATH/TO/YOUR/PROJECT/gulpfile.js
[12:30:26] Starting 'minifyJs'...
[12:30:30] Finished 'minifyJs' after 4.25 s
$ gulp minifyHtml
[12:30:49] Using gulpfile ~/PATH/TO/YOUR/PROJECT/gulpfile.js
[12:30:49] Starting 'minifyHtml'...
[12:30:49] Finished 'minifyHtml' after 546 ms
$ 
```

If you want to make it all running by single command, add new task like this.
```
gulp.task('release', ['minifyJs', 'minifyHtml']);
```

Try this.
```
$ gulp release
[20:55:29] Starting 'minifyJs'...
[20:55:29] Starting 'minifyHtml'...
[20:55:33] Finished 'minifyJs' after 4.19 s
[20:55:34] Finished 'minifyHtml' after 4.5 s
[20:55:34] Starting 'release'...
[20:55:34] Finished 'release' after 277 μs
$ 
```

There are several useful options in `gulp-uglify` module. For example, if you want to get rid of all `console.log` stuff in minify process, add parameter in module like this.
```
gulp.task('minifyJs', function () {
    return gulp.src('JS-PATH/**/*.js')
        .pipe(minify_js({
            compress: {
                drop_console: true
            }
        }))
        .pipe(gulp.dest('COMPRESSED-JS-PATH'));
});
```

### Linting



### Remove cache




### Resources

http://gulpjs.com/
https://www.keycdn.com/blog/gulp-vs-grunt/
https://medium.com/@preslavrachev/gulp-vs-grunt-why-one-why-the-other-f5d3b398edc4
https://ics.media/entry/9199


