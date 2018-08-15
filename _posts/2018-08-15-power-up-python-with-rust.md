---
layout: post
title:  Could rust make your python better?
date:   2018-08-15
description: 
tags:
- rust
- python
permalink: power-up-python-with-rust
---


It was one IT news that how I know about, and starting to look on [Rust language](https://www.rust-lang.org/). The most attractive phrase in that time was `guaranteed memory safety`, but actually there are more pros than that.

In document, you can find like:

- zero-cost abstractions
- move semantics
- guaranteed memory safety
- threads without data races
- efficient C bindings
- and more...

Still, it is not widely-used major language. It is complecate to learn(for me...comparing with `python`, `javascript`), and there are substitutions like `golang`. Though, by strengths above, it is [being used widely](https://www.rust-lang.org/en-US/friends.html) as sub-language.


## Python performance
This topic is still being discussed in lots of community. Because it is script language, or because of GIL(Global Interpreter Lock), it has limitation in performance. Though it is still one of the most popular language because it has enough performance to do something with it, proved by lots of major companies/services(`Google`, `Instagram`...) you've heard of.

Actually, the most strength is, you can learn and develop quickly with way more short coding.

[Java]
{% highlight java %}
{% raw %}
public class HelloWorld
{
    public static void main (String[] args)
    {
        System.out.println("Hello, world!");
    }
}
{% endraw %}
{% endhighlight %}

[Python]
{% highlight python %}
{% raw %}
print('Hello, world!')
{% endraw %}
{% endhighlight %}

Maybe the reason of this discussion is because it is being used widely, and anyway, it has performance issue though it is enough to use or not. So some of modules wrap-up compile language and handle process which could take time in there. For example, [Numpy](https://github.com/numpy/numpy) seems using `C` for the part that requires performance.


## Attach with Rust
![Screenshot](/assets/post_img/power-up-python-with-rust/rust-python.png)

I found good comment about merit of binding rust module:

> Rust is a language that can usually detect, during compilation, the worst parallelism and memory management errors (such as accessing data on different threads without synchronization, or using data after they have been deallocated), but gives you a hatch escape in the case you really know what you're doing.

Using `Rust` also can make performance better, and moreover, it can avoid memory management problem, which offtenly caused in `C`. And as written in document, Rust library can expose a C ABI(application binary interface) to Python with zero overhead.

You can work with only using native Rust API, but there are already good extension modules which will save your pain and code.
- [rust-cpython](https://github.com/dgrunwald/rust-cpython)
- [pyo3](https://github.com/PyO3/pyo3)

Actually, root of both are same because `pyo3` are the fork project of `rust-cpython`. It requires bit less coding than `rust-cpython`, but it only supports nightly version of rust for now, so need to aware of using it.

The motivation for this work was [post in RedHat developer blog](https://developers.redhat.com/blog/2017/11/16/speed-python-using-rust/). My work refered lot of parts from here. 


## Basic wrapper
I'll use `rust-cpython` here, with newest rust stable version `1.28.0`.

I create empty rust project with `cargo`:
{% highlight shell %}
{% raw %}
$ cargo new pyrust --lib
     Created library `pyrust` project
{% endraw %}
{% endhighlight %}

Fix `Cargo.toml` file to make sample uses cpython:
{% highlight shell %}
{% raw %}
[lib]
name = "pyrust"
crate-type = ["cdylib"]

[dependencies.cpython]
version = "0.2"
features = ["extension-module"]
{% endraw %}
{% endhighlight %}

Before testing performance, I created basic rust method, to check calling in python works well. This is simple function, which print out project version:
{% highlight rust %}
{% raw %}
#[macro_use]
extern crate cpython;

use cpython::{Python, PyResult};


fn print_from_rs(_: Python) -> PyResult<u64> {
    const VERSION: &'static str = env!("CARGO_PKG_VERSION");
    println!("PyRust version : {}", VERSION);
    Ok(1)
}
{% endraw %}
{% endhighlight %}

This method has only single `Python` instance, imported from `cpython`. This is zero-size marker struct that is required for most Python operations. This is used to indicate that the operation accesses/modifies the Python interpreter state. It is default parameter for wrapper method, but not being used in this method, so set as underbar. Return value is being given as `PyResult`.
{% highlight rust %}
{% raw %}
...
py_module_initializer!(libpyrust, initlibpyrust, PyInit_libpyrust, |py, m| {
    try!(m.add(py, "print_from_rs", py_fn!(py, print_from_rs())));
    Ok(())
});
...
{% endraw %}
{% endhighlight %}

This is extension to an `extern "C"` function to make python loading the rust code. First param `libpyrust` is the module name, which will be used in python. Second and third value is necessary for python initiation, but don't need to think about it for now. Last `|py, m|` is a lambda of `Fn(Python, &PyModule) -> PyResult<()>` and it makes importing the module inside initializer.

Now you need to build library to use. If you are MacOS user like me, make sure that you need to change `*.dylib` file to `*.so` to import. Put in library file in same directory with your python file.
```
$ cargo build --release
    Compiling pyrust v0.1.0...
    ...
    Finished release [optimized] target(s) in 2.14s
$ mv target/release/libpyrust.dylib <path/to/pythonfile/libpyrust.so>
```

Import `libpyrust` and call the method. It will show `PyRust version : 0.1.0` if correct.
{% highlight python %}
{% raw %}
import libpyrust

libpyrust.print_from_rs()
{% endraw %}
{% endhighlight %}


## Performance test #1: Search text
I make 2 method for test. One is for searching word in text and return the number of appearance in text, and other one is sum calculation in array.
This is Rust code:
{% highlight rust %}
{% raw %}
// search target word from text
fn search_text(_: Python, target: &str, text: &str) -> PyResult<u64> {
    let mut total = 0u64;
    // split word by whitespace
    let iter = text.split_whitespace();

    // compare word with target, and total += 1 if it is same
    for word in iter {
        match word == target {
            true => total += 1,
            false => {}
        }
    }
    
    Ok(total)
}

// add all value in list
fn sum_list(_: Python, list: Vec<u64>) -> PyResult<u64> {
    let mut total = 0u64;

    for num in list {
        total += num
    }

    Ok(total)
}
...
py_module_initializer!(libpyrust, initlibpyrust, PyInit_libpyrust, |py, m| {
    ...
    try!(m.add(py, "search_text", py_fn!(py, search_text(target: &str, val: &str))));
    try!(m.add(py, "sum_list", py_fn!(py, sum_list(list: Vec<u64>))));
    ...
});
{% endraw %}
{% endhighlight %}

...and Python part. I tried to make code form similar as possible:
{% highlight python %}
{% raw %}
...
def search_word(target, val):
    word_list = val.split(' ')
    total = 0
    for word in word_list:
        if word == target:
            total += 1

    return total

def sum_list(val):
    num_list = random_list
    sum = 0
    for num in num_list:
        sum += num
    
    return sum
...
{% endraw %}
{% endhighlight %}

It is good to use python benchmark module for test. It has attached module with `pytest`, to show performance result by test. Install [pytest-benchmark](https://github.com/ionelmc/pytest-benchmark) in your project, and create test module like this. Text I used for test is part of text from [Wikipedia-New York State Route 22](https://en.wikipedia.org/wiki/New_York_State_Route_22). I copy/paste itself to make file more bigger(size of 'nystreet.txt' is about 2Mb). This test will return how often word 'NY' appeared in text file.
{% highlight python %}
{% raw %}
import librust2py

...
ny_data = ''
with open('nystreet.txt', 'r') as myfile:
    ny_data = myfile.read().replace('\n', '')

def test_python(benchmark):
    benchmark(search_word, 'NY', ny_data)

def test_rust(benchmark):
    benchmark(librust2py.search_text, 'NY', ny_data)
{% endraw %}
{% endhighlight %}

![Screenshot](/assets/post_img/power-up-python-with-rust/search-text-result.png)

This is the result. Though I just used very rough code, you could find positive side of using this.


## Performance test #2: Add number in array
Now I'll use another method `sum_list` to test simple calculation. In this test, there are array of size 10, including random value 1~99999.
{% highlight python %}
{% raw %}
import librust2py

...
random_list = random.sample(range(1, 99999), 10)

def test_python(benchmark):
    benchmark(sum_list, random_list)

def test_rust(benchmark):   #  <-- Benchmark the Rust version
    benchmark(libpyrust.sum_list, random_list)
{% endraw %}
{% endhighlight %}

![Screenshot](/assets/post_img/power-up-python-with-rust/add-num-result-1.png)

To the contrary, pure python is more fast than before. Than how about making array more bigger? I changed array size as 10000.
{% highlight python %}
{% raw %}
...
random_list = random.sample(range(1, 99999), 10000)
...
{% endraw %}
{% endhighlight %}

![Screenshot](/assets/post_img/power-up-python-with-rust/add-num-result-2.png)

Okay, it shows expected result. Maybe loading to rust could make little delay, and that makes performance worse in small calculation.


## Performance test #2-2: Compare calculation with NumPy
For the last, I'll try to compare with NumPy array. It uses C language for performance upgrade, so we can expect that it would be faster than at least of pure python version.

{% highlight python %}
{% raw %}
import librust2py
import numpy as np
...
def sum_np_list(val):
    sum = val.sum(0)
    
    return sum
...

def test_python(benchmark):
    benchmark(sum_list, random_list)

def test_rust(benchmark):   #  <-- Benchmark the Rust version
    benchmark(libpyrust.sum_list, random_list)

def test_np(benchmark):
    benchmark(sum_np_list, random_list)
{% endraw %}
{% endhighlight %}

![Screenshot](/assets/post_img/power-up-python-with-rust/add-num-result-3.png)

Okay, something seems wrong. Using NumPy is showing the worst result.
After researching stackoverflow, I found loading array to NumPy array is causing delay. So I created NumPy array outside.

{% highlight python %}
{% raw %}
...
// put array in numpy array, and calculate sum value
def sum_np_list(val):
    np_random_list = np.array(val)
    sum = np_random_list.sum(0)
    
    return sum
...
random_list = random.sample(range(1, 99999), 10000)
np_random_list = np.array(random_list)
...

def test_python(benchmark):
    benchmark(sum_list, random_list)

def test_rust(benchmark):   #  <-- Benchmark the Rust version
    benchmark(libpyrust.sum_list, random_list)

def test_np(benchmark):
    benchmark(sum_np_list, np_random_list)
{% endraw %}
{% endhighlight %}

![Screenshot](/assets/post_img/power-up-python-with-rust/add-num-result-4.png)

Now you could find out why NumPy is being used in lots of data research modules. It shows overwhelming performance than competitors.


## Pros
For now, you could find out wrapping Rust module for Python could be worth in particular case. Actually, there are some modules like NumPy which already has its own wrapped `C/C++` module for performance, but if there is not where you need, this can be one alternative. Moreover, if you can make better performanced Rust code, you can make more better result, without worrying memory crash.


## Reference
* https://www.rust-lang.org/
* https://developers.redhat.com/blog/2017/11/16/speed-python-using-rust/
* https://github.com/dgrunwald/rust-cpython
* https://hackernoon.com/yes-python-is-slow-and-i-dont-care-13763980b5a1
* https://blog.sentry.io/2016/10/19/fixing-python-performance-with-rust.html
