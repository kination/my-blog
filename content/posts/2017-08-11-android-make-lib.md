---
layout: post
title:  "Create own module project - Android"
date:   2017-08-11
tags:
- android
- library
permalink: android-make-lib
description: 
---

There are lots of modules for Android, and it helps you to develop your own app more easier, and faster. Most of them are open source and it is still being managed/updated by many contributer. You also could be one of them as contributor, or main producer by making your own module.
By develop/manage your own project, you could receive variable feedbacks by lots of great engineers in world. It would improve your skill, learn new things, and it will be good for your career as developer. If you have good idea, go on!


### Start on

Let's create new Android project with Android Studio. Though we are focusing on module, it is better to create both app and module for making sample and tests. First create new project that has Empty Activity with Android Studio. Project will be look like this.

![Screenshot](/assets/post_img/android_module_cli/empty-activity.png)

Now go to `File > New > New Module` and select `Android Library` to create new module. If you done it right, project branch will has 2 package, for app and module.

![Screenshot](/assets/post_img/android_module_cli/empty-activity-module.png)

Now look on gradle script to import module `mylibrary` project to `app`. Find `settings.gradle` and see packages has included well, and add it if not.
{% highlight shell %}
{% raw %}
include ':app', ':mylibrary'
{% endraw %}
{% endhighlight %}

Now go to `build.gradle` file in app module file and add dependency for `mylibrary` module.
{% highlight shell %}
{% raw %}
...
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:26.+'
    compile 'com.android.support.constraint:constraint-layout:1.0.2'
    testCompile 'junit:junit:4.12'
    
    compile project(path: ':mylibrary') // <= Add this code
}
...
{% endraw %}
{% endhighlight %}

Now your `app` module can import classes in `mylibrary` package.


### Create simple module

I'll make a simple module which returns list of pair which include word and number. Word are the word included in text, and number is the appearance frequency of word in text. For example,

"My name is John. Last name is Doe. Doe family is living here"
=> 
(My, 1), (name, 2), (is, 3), (John, 1), (Last, 1), (Doe, 2), (family, 1), (living, 1), (here, 1)

Now I'll put on 2 file into `mylibrary` package, `Reducer.java` and `WordPair.java`.
`Reducer.java` includes logic to count word in text and returns as list of word and number.
{% highlight java %}
{% raw %}
public class Reducer {

    private static Reducer reducerInstance;

    private Reducer() {
    }

    public static Reducer getInstance() {
        if (reducerInstance == null) {
            reducerInstance = new Reducer();
        }

        return reducerInstance;
    }

    public ArrayList<WordPair> textToPair(String text, String delimeter) {
        String[] wordList = text.replaceAll("\\.", "").split(delimeter);
        ArrayList<WordPair> pairList = new ArrayList<>();
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
                pairList.add(new WordPair(word, 1));
            }
        }

        return pairList;
    }
}
{% endraw %}
{% endhighlight %}

`WordPair.java` is for put in word and count number. There can be some of getter and setter, but this time I'll not think about it.
{% highlight java %}
{% raw %}
public class WordPair {
    public String word;
    public int number;

    public WordPair(String word, int number) {
        this.word = word;
        this.number = number;
    }
}
{% endraw %}
{% endhighlight %}

Now try to use this module in app. I'll make a simple Activity that shows the result of example above.
[MainActivity.java]
{% highlight java %}
{% raw %}
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Reducer reducer = Reducer.getInstance();

        ArrayList<WordPair> parsedText = reducer.textToPair("My name is John. Last name is Doe. Doe family is living here", " ");

        StringBuilder sBuilder = new StringBuilder();
        TextView parsed = (TextView) findViewById(R.id.text_pair);
        for (WordPair wordPair : parsedText) {
            sBuilder.append("(" + wordPair.word + ":" + wordPair.number + ")\n");
        }

        parsed.setText(sBuilder);
    }
}
{% endraw %}
{% endhighlight %}

[activity_main.xml]
{% highlight xml %}
{% raw %}
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="io.kination.myapplication.MainActivity">

    <TextView
        android:id="@+id/text_pair"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</android.support.constraint.ConstraintLayout>
{% endraw %}
{% endhighlight %}

It will show parsed result in empty page.
![Screenshot](/assets/post_img/android_module_cli/show-parsed.png)

Now we saw module is working fine on main project. I'll go on some more with this next time.


### Reference
* https://developer.android.com
