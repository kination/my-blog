---
layout: post
title:  Back to basic of Tree
date:   2021-11-15
description: 
tags:
- java
- dfs
- bfs
permalink: back-to-basic-of-tree-structure
---

This is just for reminding my knowledge, about tree data structure, which most of developers will know:

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

## DFS & BFS
With this tree, if we need to implement `isExists` function above, we can think of doing it in recursive way.

```java
public boolean isExists(Tree t, int target) {
  if (t == null) return false;
  
  return isExists(t.left) || isExists(t.right);
}
```

Using recursion is one common way to go through tree structure by calling same function until it finds leaf.

But this is the way using DFS, cause it will go through left child first, until it faces `null`.
If you need to make this in BFS way, it needs some additional features.

```
public boolean isExists(Tree t, int target) {
  Queue<Tree> q = new LinkedList<Tree>();
  q.add(t);
  
  while (!q.isEmpty()) {
    Tree tree = q.poll();
    
    if (tree.val == target) return true;
    
    if (tree.left != null) {
      q.add(tree.left);
    }
    
    if (tree.right != null) {
      q.add(tree.right);
    }
  }
  
  return false;
}
```

Cause it should read all data in same level before going on to next, recursive way is not adaptable.


## Binary tree


