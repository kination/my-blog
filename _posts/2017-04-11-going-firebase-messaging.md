---
layout: post
title:  "Going on to Firebase cloud messaging(FCM) in Android"
date:   2017-04-05
tags:
- android
- firebase
permalink: 
description: How-to move legacy GCM to Firebase cloud messaging in Android client
---

During Android being update A to N(and soon O will be updated), messaging library has been updated as `C2DM -> GCM -> GCM in google-play-service -> FCM(Firebase cloud messaging)`. The names are all different, but doing same thing - send push message to target device. Actually there are plenty of updates and functions in Firebase, but I'll focus on messaging implementation here.

### Remove legacy codes

Push messaging of current project I'm working on is implemented with old version of GCM, which is the one before included inside google-play-service module. It has been deprecated more than 4 years, and there are lots of bugs which will never be fixed.

Let's look on legacy stuffs. If your project is based on old GCM, you will find code like below.
{% highlight java %}
{% raw %}
public class MyGcmIntentService extends GCMBaseIntentService {

    @Override
    protected void onRegistered(Context context, String registrationId) {
        // Something needs to be done after gcm registered
    }

    @Override
    protected void onUnregistered(Context context, String registrationId) {
        // Something needs to be done after gcm unregistered
    }

    @Override
    protected void onMessage(Context context, Intent intent) {
        // Something needs to be done when message received
    }
    ...
}
{% endraw %}
{% endhighlight %}
This is the receive handler for push message. If there is something to do when GCM is registered or message is received, it will be handled inside here.

And go on to AndroidManifest.xml, there will be some lines for GCM permission and part for registering `MyGcmIntentService` class.
{% highlight xml %}
{% raw %}
<permission android:name="<your-package-name>.permission.C2D_MESSAGE"
            android:protectionLevel="signature" />
<uses-permission android:name="<your-package-name>.permission.C2D_MESSAGE" />
<uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
...
<service android:name=".MyGcmIntentService" />
{% endraw %}
{% endhighlight %}

These parts are now useless for new Firebase messaging. Permissions for this will be added automatically in FCM by library.
Also, there will be codes for GCM registration when app starts. They don't need anymore either.
{% highlight java %}
{% raw %}
...
GCMRegistrar.checkDevice(context);
GCMRegistrar.checkManifest(context);
GCMRegistrar.register(context, "sender-id");
...
{% endraw %}
{% endhighlight %}

### Setup for FCM implementation

To use Firebase modules, you need to setup Firebase tool in your Android Studio. Go to `Tools > Firebase` and you will see Assistant menu like this.

![Screenshot](/assets/post_img/gcm_to_fcm/tools_firebase.png)

Select `Set up Firebase Cloud Messaging > 1. Connect your app to Firebase`. You can make new project, or call recent project to register. After registration complete, you will see page like this in your browser.

![Screenshot](/assets/post_img/gcm_to_fcm/android_studio_fb.png)

If you clear this part, you will find that file `google-services.json` being added in your project's `/app` folder. This file includes the information which needs for using Firebase service in your app. You don't need for registration process code in your app as before. It will be done automatically by library.

Now open project build.gradle file and add line for google service

{% highlight groovy %}
{% raw %}
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        ...
        classpath 'com.google.gms:google-services:3.0.0'        //Add this line
    }
}
{% endraw %}
{% endhighlight %}

and open app build.gradle file

{% highlight groovy %}
{% raw %}
...
dependencies {
    ...
    compile 'com.google.firebase:firebase-messaging:9.0.0'      //Add this line
}

apply plugin: 'com.google.gms.google-services'      //Add this line
{% endraw %}
{% endhighlight %}

If you did not go on Connecting app process, you will see error for applying `com.google.gms.google-services` plugin while on gradle sync.

You could see `Tools > Firebase > Set up Firebase Cloud Messaging` change like this if you follow up correctly.
![Screenshot](/assets/post_img/gcm_to_fcm/fb_connect_complete.png)

Now setup is all cleared.

### Implements for FCM

In FCM, you need services for handling registeration, and receiver. In old GCM it is handled in single service which extended `GCMBaseIntentService`, but here it is divided with `FirebaseInstanceIdService` and `FirebaseMessagingService`. The first one are being called for handle create/update of registration token, while second one are handling push messages.

`FCMService.java`
{% highlight java %}
{% raw %}
public class FCMService extends FirebaseMessagingService {

    @Override
    public void onMessageReceived(RemoteMessage remoteMessage) {
        // Something needs to be done when message received
    }
}
{% endraw %}
{% endhighlight %}

You can call override method `onMessageReceived` in this service to add code that needs to be done when message received. 
In `GCMBaseIntentService.onMessage` method from old GCM, there were `Intent` param to get push message data. This has been changed to use `RemoteMessage` class. For example...

{% highlight java %}
{% raw %}
// Code in GCMBaseIntentService
@Override
protected void onMessage(Context appCtx, Intent intent) {
    String data = intent.getStringExtra("push_key");
}
{% endraw %}
{% endhighlight %}

you can have same data value here.

{% highlight java %}
{% raw %}
// Code in FirebaseMessagingService
@Override
public void onMessageReceived(RemoteMessage remoteMessage) {
    String data = remoteMessage.getData().get("push_key");
}
{% endraw %}
{% endhighlight %}

And this is the service to get token information. You can call override method `onTokenRefresh` if something needs to be done when token refreshed.

`FBIdService.java`
{% highlight java %}
{% raw %}
public class FBIdService extends FirebaseInstanceIdService {

    @Override
    public void onTokenRefresh() {
        // Something needs to be done when token refreshed
    }
}
{% endraw %}
{% endhighlight %}

Don't forget to register these services in manifest file. Each of them needs intent filter action like below.

{% highlight xml %}
{% raw %}
<service
    android:name=".FCMService">
    <intent-filter>
        <action android:name="com.google.firebase.MESSAGING_EVENT"/>
    </intent-filter>
</service>

<service
    android:name=".FBIdService">
    <intent-filter>
        <action android:name="com.google.firebase.INSTANCE_ID_EVENT"/>
    </intent-filter>
</service>
{% endraw %}
{% endhighlight %}

Now it is done. Because you don't need register process, it will working without kinds of initiating stuffs. There are some more things like sending topic message and upstream message, but this will be done for migrating old one.

While working on this, I found lots of great stuffs in Firebase. Hope to get on more in this later.


