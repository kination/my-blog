---
layout: post
title:  "Test your module continuously - Android"
date:   2017-09-30
tags:
- android
- module
- test
- ci
permalink: android-test-lib
description: How to test your module continuously with github. 
---



### Make tests

Test is important to make your project reliable. As your project going bigger, there will be lots of bugs, and it is tough to keep your codes stable while fixing and modifing it.
It is pretty easy to setup your test for Android. Actually, it builds up when you create it as `ExampleInstrumentedTest` and `ExampleUnitTest`. Instrument test is for testing UI process flow, and it is being tested on virtual/real devices. Unit test is for testing internal data logic of classes or modules and it is running on JVM, so you cannot access Android APIs like `Context` or else. 

This is to just look on process of test, so I'll make a simple unit test.

Check gradle file and see module for test has been included(usually it has by default).
{% highlight shell %}
{% raw %}
dependencies {
    ...
    testCompile 'junit:junit:4.12'
}
{% endraw %}
{% endhighlight %}

Now write test code.
[ExampleUnitTest.java]
{% highlight java %}
{% raw %}
public class ExampleUnitTest {

    private static final String TARGET_TEST_STRING = "My name is John. Last name is Doe. Doe family is living here.";
    private Reducer reducer;

    @Before
    public void setupParam() {
        reducer = Reducer.getInstance();
    }

    @Test
    public void testSpaceDelimeter() throws Exception {
        String delimeter_space = " ";
        ArrayList<WordPair> parsedText = reducer.textToPair(TARGET_TEST_STRING, delimeter_space);
        assertEquals(parsedText.size(), 9);
        assertEquals(parsedText.get(1).word, "name");
        assertEquals(parsedText.get(1).number, 2);
    }

    @Test
    public void moreTest() throws Exception {
        // more test...
    }
}
{% endraw %}
{% endhighlight %}

This is to test `textToPair` logic what we just tried now. 

In test code, there are several decorator being used to define test process. `@Before` is the function which will be running before starting test, and functions with `@Test` decorator will be running in actual test. If you need something to do after test, use `@After`. It once has been defined with fixed function name(runnning function with name `setUp` before test, and test functions which starts with name `test`), but it has been changed from `AndroidJUnit4`.

`assertEquals` is to compare results with expected value. It compares 2 parameter and return exception when it is not equal value. Code in `testSpaceDelimeter` checks whether size of list `parsedText` are 9, second value of list is "name", and number of second value is 2. Because it is all correct as we see in Activity code, test passes here. You can run test simply in Android Studio with `Run` button.

![Screenshot](/assets/post_img/android_module_cli/test-success.png)


