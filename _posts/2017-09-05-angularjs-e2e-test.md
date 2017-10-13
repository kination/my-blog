---
layout: post
title:  "e2e test for AngularJS"
date:   2017-09-05
tags:
- angularjs
- test
- protractor
permalink: angularjs-e2e-test
description: end-to-end testing AngularJS client project
---


### e2e test

e2e, or end-to-end testing is a testing method that goes on to actual application usage flow. It means, it tests your applicaion in user's point of view. It is to check whether client showing correct information what system requests. You can register test scenario like components shows/hides correctly, action(button click, drag&drop) works well, and more things related with user interfaces/moves.

As you can expect, there are cons comparing with unit test. It takes long time, it is not reliable, and cannot isolate faliures. Though it is still important because unit test can find problems in data, but it could miss the bugs on user's usage flow. 

![Screenshot](/assets/post_img/angular_e2e_test/unit-vs-e2e.jpg)

Like this chart, it has different purpose with unit test. It is focusing on user interaction and UI instead of checking code method and result value. So if you are running web service and client/server project has been seperated, you could think about applying e2e test logic for client. Unit test will be suitable for your server project, cause it needs to focus on API data format and result value.


### Setup Protractor

`Angular Scenario Runner` was a default componenet for AngulsrJS e2e testing called, but now has been deprecated. Official AngularJS guide suggests to use [Protractor](http://www.protractortest.org/) for test. It supports AngularJS, and Angular project.

Install dependencies first.

{% highlight shell %}
{% raw %}
$ npm install -g protractor
...
$ protractor --version
Version 5.1.2
$ webdriver-manager update
[14:41:41] I/update - chromedriver: file exists /usr/local/lib/node_modules/protractor/node_modules/webdriver-manager/selenium/chromedriver_2.33.zip
[14:41:41] I/update - chromedriver: unzipping chromedriver_2.33.zip
[14:41:41] I/update - chromedriver: setting permissions to 0755 for /usr/local/lib/node_modules/protractor/node_modules/webdriver-manager/selenium/chromedriver_2.33
[14:41:41] I/update - chromedriver: chromedriver_2.33 up to date
[14:41:41] I/update - selenium standalone: file exists /usr/local/lib/node_modules/protractor/node_modules/webdriver-manager/selenium/selenium-server-standalone-3.6.0.jar
[14:41:41] I/update - selenium standalone: selenium-server-standalone-3.6.0.jar up to date
[14:41:43] I/update - geckodriver: file exists /usr/local/lib/node_modules/protractor/node_modules/webdriver-manager/selenium/geckodriver-v0.19.0.tar.gz
[14:41:43] I/update - geckodriver: unzipping geckodriver-v0.19.0.tar.gz
[14:41:43] I/update - geckodriver: setting permissions to 0755 for /usr/local/lib/node_modules/protractor/node_modules/webdriver-manager/selenium/geckodriver-v0.19.0
[14:41:43] I/update - geckodriver: geckodriver-v0.19.0 up to date
{% endraw %}
{% endhighlight %}

 Because it is e2e test, it needs target browser to run tests and Selenium will help this to do. After install `protractor`, you need to update `webdriver-manager` to get instance for running Selenium server. Now start Selenium server with `webdriver-manager start`. Your test spec will be tested above here.

{% highlight shell %}
{% raw %}
$ webdriver-manager start
[15:19:55] I/start - java -Dwebdriver.chrome.driver=/usr/local/lib/node_modules/protractor/node_modules/webdriver-manager/selenium/chromedriver_2.33 -Dwebdriver.gecko.driver=/usr/local/lib/node_modules/protractor/node_modules/webdriver-manager/selenium/geckodriver-v0.19.0 -jar /usr/local/lib/node_modules/protractor/node_modules/webdriver-manager/selenium/selenium-server-standalone-3.6.0.jar -port 4444
...
15:19:56.818:INFO:osjs.AbstractConnector:main: Started ServerConnector@5e853265{HTTP/1.1,[http/1.1]}{0.0.0.0:4444}
15:19:56.819:INFO:osjs.Server:main: Started @775ms
15:19:56.819 INFO - Selenium Server is up and running
{% endraw %}
{% endhighlight %}


### Setup configuration

You need to write script for configuration and test specification. First create `protractor.conf.js` file.

{% highlight javascript %}
{% raw %}
exports.config = {
    baseUrl: 'http://localhost:8000/',
    capabilities: {
        'browserName': 'chrome'
    },
    framework: 'jasmine',
    seleniumAddress: 'http://localhost:4444/wd/hub',
    specs: ['protractor.spec.js'],
    allScriptsTimeout: 11000
};
{% endraw %}
{% endhighlight %}

Param `baseUrl` is to put in target url for test. I'm testing with local web server, so setup with `localhost`. You can define your test framework with `framework`, and selenium address would be `http://localhost:4444/wd/hub`. Define spec script file in `specs`, and you can include glob pattern like 'spec/*_spec.js'. You can also setup running time limitation for each script with `allScriptsTimeout`. If you want to find more option, check [here](https://github.com/angular/protractor/blob/5.1.2/lib/config.ts).


### Write test spec

{% highlight html %}
{% raw %}
<div class="login-container bg-white">
    <div class="p-l-50 m-l-20 p-r-50 m-r-20 p-t-50 m-t-30 sm-p-l-15 sm-p-r-15 sm-p-t-40">
        <form id="form-login" name="login" class="p-t-15" role="form" ng-submit="vm.login()" novalidate>
                <label>Email</label>
                <div class="controls">
                    <input id="email" type="email" name="email" ng-model="vm.email" class="form-control">
                </div>
            </div>
            <div class="form-group form-group-default">
                <label>Password</label>
                <div class="controls">
                    <input type="password" class="form-control" name="password" ng-model="vm.password">
                </div>
            </div>
            <div class="row">
                <div class="col-md-9">
                    <label ng-class="{'error-msg-area' : (vm.login_alert_msg.show==false)}">
                        {{ vm.login_alert_msg.message }}
                    </label>
                </div>
            </div>
            <button class="btn btn-danger btn-cons m-t-10" type="submit">LOG IN</button>
        </form>
    </div>
</div>
{% endraw %}
{% endhighlight %}

This is part of the template I will test on. This has simple login form, which has input for email/password, login button.

Now write test spec file 'protractor.spec.js'.
{% highlight javascript %}
{% raw %}
describe('Protractor Demo Test', function() {
    var userId;
    var password;
    var login;

    beforeEach (function () {
        browser.get(browser.baseUrl)
        userId = element(by.model('vm.email'));
        password = element(by.model('vm.password'));
        login = element(by.buttonText('LOG IN'));
    });


    it ('should have a title', function () {
        expect(browser.getTitle()).toEqual('Protractor Test');
    });

    ...
});
{% endraw %}
{% endhighlight %}

This is basic form of spec script, based on jasmine. `beforeEach` is same as `setUp` in usual test syntax which runs before tests in script begins, and `it` is for defining each test module.

In this script, browser loads target url from configuration file above(here it is localhost:8000) and setup elements(input, button, etc) for test. There are some ways to get these. If you want to get element by angular model, call with `by.model`. It will find elements defined with `ng-model`. You could also find using text in button with `by.buttonText`(I'm not sure this is a good way). This `by` is called `locator` and you could find more info [here](http://www.protractortest.org/#/api?view=ProtractorBy).

Now in first test 'should have a title' is checking web page title. `expect` is something like `assertTrue`, which returns error when result is not true. You can get title with `browser.getTitle` and compare it with `toEqual` method.

{% highlight javascript %}
{% raw %}
describe('Protractor Demo Test', function() {
    ...

    it ('should show error', function () {
        userId.sendKeys(''); // put in email
        password.sendKeys(''); // ...and wrong password
        login.click();
        expect(element(by.css('.error-msg-area')).isPresent()).toBe(true);
    });

    it ('should success login', function () {
        var EC = protractor.ExpectedConditions;
        userId.sendKeys('koni50@naver.com'); // put in email
        password.sendKeys('12341234'); // ...and correct password
        login.click();
        browser.wait(EC.urlContains('/home'), 5000);
    })
});
{% endraw %}
{% endhighlight %}

These part, checks bit more actions.

Test case 'should show error' tests that error message is being shown when trying to log in with wrong password. Define action to put in email and password by using `sendKeys` at input element and to click login button. Error message has class `error-msg-area` so you can get this element with `by.css`, and check it has been visible with `isPresent` method. 
If you want to check detail result with `expect`, put `toBe` after and give expected value.

Last one is for checking login succedded with correct email/password. There are several ways to check this, and I'll define success with comparing url after login. In this test, url after login should include `/home`.
After click event happens, make test to wait browser result for 5 seconds cause it needs time to complete login.
`protractor.ExpectedConditions` is a set of built-in conditions you can get from browser, which related with Selenium server. Here `urlContains` has been used for comparing url. You could find more APIs in [document](http://www.protractortest.org/#/api?view=ProtractorExpectedConditions).


### Reference
* [http://www.protractortest.org/](http://www.protractortest.org/)
* [https://rocketeer.be/articles/testing-with-angular-js/](https://rocketeer.be/articles/testing-with-angular-js/)



