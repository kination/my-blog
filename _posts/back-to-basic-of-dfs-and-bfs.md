---
layout: post
title:  Back to basic of DFS and BFS
date:   2021-11-15
description: 
tags:
- java
- dfs
- bfs
permalink: back-to-basic-of-dfs-and-bfs
---

This is just for reminding my knowledge, about tree data structure, which all of us will know:

![tree](https://user-images.githubusercontent.com/1720209/146526771-9c57cb9b-32e2-4a22-b0a1-ff80011a1c9a.png)

yes, something looks like this.


Most of the cases, we don't need to think about the implementation deeply, cause for most of the programming languages there are library package to make tree structure.

```java

```

## Make some tree
When I faced on coding test, mostly it is being held based on coding tool, so it does not need to make things from scratch.

For example if they ask you to find out whether specific value exists in tree, you just need to implement the code inside of given function, and given information of `Tree` object.
```java
public boolean isExists(Tree t, int target) {
  // do something
}
```

But there are cases you should make code in whiteboard, or pure text editor(after COVID, all interviews are being held in online), and in this cases they can start with 'create the tree class'.

Well, it'll be simple for now
```java
class Tree {
  Tree left;
  Tree right;
  int value;
  
  public Tree (int value) {
    this.value = value;
  }
}
```

