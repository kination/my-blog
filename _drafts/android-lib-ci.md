---
layout: post
title:  "Create, build, test your module - Android"
date:   2017-04-05
categories:
- android
- library
- ci
- git
permalink: 
description: How to create, test, and keep stability with continuous integration service 
---

There are lots of modules for Android, and it helps you to develop your own app more easier, and faster. Most of them are open source and it is still being managed/updated by many contributer. You also could be one of them as contributer, or main producer by making your own module.
By develop/manage your own project, you could receive variable feedbacks by lots of great engineers in world. It would improve your skill, learn new things, and it will be good for your career as developer.

??? This is one simple process of making simple module, and update on Github.

### Start on

Let's create new Android project with Android Studio. Because this module is very simple, you don't need to create any view for this. I'll select 'No Activity'.

Now go to `build.gradle` file and change `apply plugin: 'com.android.application'` to `apply plugin: 'com.android.library'` to make your project act as library.

I'll make a simple module which returns list of pair which include word and number. Word are the word included in text, and number is the appearance frequency of word in text. For example,

"My name is John. Last name is Doe. Doe family is living here"
=> 
(My, 1), (name, 2), (is, 3), (John, 1), (Last, 1), (Doe, 2), (family, 1), (living, 1), (here, 1)

There are 2 classes for this.

`Reducer.java` includes logic to count word in text and returns as list of word and number.
{% highlight java %}
{% raw %}
import java.util.ArrayList;

public class Reducer {
    
    public ArrayList<Pair> textToPair(String text, String delimeter) {
        String[] wordList = text.replaceAll("\\.", "").split(delimeter);
        ArrayList<Pair> pairList = new ArrayList<>();
        boolean isIncluded = false;

        for (String word : wordList) {
            isIncluded = false;
            for (int i = 0; i < pairList.size(); i++) {
                if (pairList.get(i).word.equals(word)) {
                    // is already included. Give +1
                    isIncluded = true;
                    int currentNum = pairList.get(i).number;
                    pairList.get(i).number = currentNum + 1;
                    break;
                }
            }

            if (!isIncluded) {
                // Add new pair if word not exists
                pairList.add(new Pair(word, 1));
            }
        }

        return pairList;
    }
}
{% endraw %}
{% endhighlight %}

`Pair.java` is for put in word and count number. There can be some of getter and setter, but this time I'll not think about it.
{% highlight java %}
{% raw %}
public class Pair {

    String word;
    int number;

    public Pair(String word, int number) {
        this.word = word;
        this.number = number;
    }
}
{% endraw %}
{% endhighlight %}

Now you can add this module as a project in your 

### Make test

Test is important to make your project reliable. As your project going bigger, there will be lots of bugs, and it is tough to keep your codes stable while fixing and modifing it.
It is pretty easy to setup your test for Android. Actually, it builds up when you create it.

### Keep testing on

Now your project has been uploaded, and you need to think about CI.
