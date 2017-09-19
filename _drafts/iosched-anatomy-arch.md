---
layout: post
title:  "Google I/O anatomy - overview"
date:   2017-09-05
description: Review of google I/O(2017) android app
tags:
- android
- java
- google
permalink: iosched-overview
---

Google I/O, one of world's most biggest developer conference has been ended few month ago. During several years, Google released source code of google io application everytime after it, and became pretty good reference for application developer because it includes newest development trends for Android. Now Google I/O 2017 has been released, and there are plenty of new things to look on.

You can find it [here](https://github.com/google/iosched). It is still on development, so it can changed.

### Structure

![Screenshot](/assets/post_img/iosched-overview/directory_structure.png)

If you open the project, you could see directories like this. It has been seperated with /apk, /lib, /server . Most of the part for application such as UI, model, services and else is inside `lib` directory. `server` includes the part which need to be access to products in [Google cloud platform](https://cloud.google.com) and [Firebase](https://firebase.google.com/) and you need registration IDs to use full services here. 

I'll try to focus on `lib` part in here.

### MVP

Google I/O app architecture is based on MVP, a Model-View-Presenter pattern.

![Screenshot](/assets/post_img/iosched-overview/mvp_pattern.png)



