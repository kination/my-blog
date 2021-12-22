---
layout: post
title:  Reminding tree structure
date:   2021-10-15
description: 
tags:
- programming
- java
- datastructure
- dfs
- bfs
permalink: reminding-tree-structure
---

This is just for reminding my knowledge, about tree data structure, which most of developers will know:

![tree](https://user-images.githubusercontent.com/1720209/146526771-9c57cb9b-32e2-4a22-b0a1-ff80011a1c9a.png)

yes, something looks like this.

Most of the cases we don't need to think about the implementation deeply, cause most of the programming languages have library packages to make tree structure. But if you need to do your own, how would we start?


## Start to make some tree
Tree is one of favorite question in coding test, and I think because it's pretty good to know interviewee's knowledge bit deeply, with single question.

If test goes on coding tool, usually it does not need to make things from scratch. For example if they ask you to find out whether specific value exists in tree, you just need to implement the code inside of given function, with given information of `Node`(branch of tree) object.
```java
public boolean isExists(Node t, int target) {
  // do something
}
```

But there are cases you should make code in whiteboard or pure text editor, and they can start asking you too 'create some tree class'.

Well, it'll be simple, but it's good to be friendly in this situation.
```java
class Node {
  Node left;
  Node right;
  int value;
  
  public Node (int value) {
    this.value = value;
  }
}
```

## DFS & BFS
With this tree, if we need to implement `isExists` function above, we can think of doing it in recursive way.

```java
public boolean isExists(Node t, int target) {
  if (t == null) return false;
  
  return isExists(t.left) || isExists(t.right);
}
```

Using recursion is one common way to go through tree structure by calling same function until it finds leaf.

But this is the way using DFS, cause it will go through left child first, until it faces `null`.
If you need to make this in BFS way, it needs some additional features.

```java
public boolean isExists(Node t, int target) {
  Queue<Node> q = new LinkedList<Node>();
  q.add(t);
  
  while (!q.isEmpty()) {
    Node tree = q.poll();
    
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

Cause it should read all data in same level before going on to next level, recursive way is not adaptable.

Using queue is not mandatory, but the logic required collection to push/poll value in FIFO order, so it is most optimized way.


## Binary tree
Now you can maybe face on additional request, to make `binary tree` using `Node` above.
It requires one more condition that every node has a value that is...
- greater than or equal to the node values in the left sub-tree
- less than or equal to the node values in the right sub-tree`

In the previous tree it should define the location of every node manually, like:
```java
Node t = new Node(3);
t.left = new Node(4);
t.right = new Node(5);
```

But for binary tree, it only needs to add, and location should be defined by itself.
```java
BinaryTree t = new BinaryTree(3);
t.add(4);
t.add(5);

/*
should be performed like this...
   3
    \
     4
      \ 
       5
*/
```

Using `Node` class above, we could start with:
```java
class BinaryTree {
  Node root;
  
  public BinaryTree(int val) {
    root = new Node(val);
  }
}
```

and for `add`, it needs logic to find the location to be placed.

Doing recursively will be good, to track the tree by depth:
```java
...
public Node findRecursively(Node node, int val) {
  if (node == null) {
    return new Node(val);
  }

  if (val < node.val) {
    // value is lower than value in node, so go left
    node.left = findRecursively(node.left, val);
  }
  else if (val > node.val) {
    // value is bigger than value in node, so go right
    node.right = findRecursively(node.right, val);
  }
  
  return current;
}
```

and with this method, make `add`:
```java
public void add(int val) {
  // make a tree with new value, and setup at root node
  root = findRecursively(root, val);
}
```


## Reference
* <https://www.baeldung.com/>
* <https://www.geeksforgeeks.org/>
