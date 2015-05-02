---
layout: post
title: "Walking the DOM With Vanilla JavaScript"
categories: [programming]
tags: [javascript, DOM]
date: 2015-05-02T15:14:12-04:00
---

The other day I was presented with the following interview question:

>  For a given element on a given webpage, "reverse-print" all of the text in the element  (i.e. for each subtree, print the leaf node text first, then parent node, etc etc).

Looks like a straightforward [post-order tree traversal](http://en.wikipedia.org/wiki/Tree_traversal#Post-order) (visit left subtree, visit right subtree, print root data).

To actually make this work, you need to know a few things about the DOM API.

-  The DOM is built out of [Node](https://developer.mozilla.org/en-US/docs/Web/API/Node) objects.  Well, actually objects that inherit the Node interface.

-  Not all Nodes are the same.  In fact, there are 12 different kinds of Nodes!  For this problem, we'll only be concerned with text nodes and element nodes:

    -  `Node.nodeType === 1`:  an element node (i.e. what we normally think of as a DOM node, like a div or paragraph or body)

    -  `Node.nodeType === 3`:  a text node

{% highlight javascript %}

var reversePrint = function (node){
    var i, j, child;
    if(node.firstChild){
        for (i=0; i<node.children.length; i++){
            child = node.children[i];
            reversePrint(child);
        }
        for (j=0; j<node.childNodes.length; j++){
            child = node.childNodes[j];
            if (child.nodeType === 3){ console.log(child.nodeValue); }
        }
    }
};

node = document.getElementsByClassName('container')[0];
reversePrint(node);

{% endhighlight %}

Since we're traversing a tree, a recursive approach is the easiest to implement.  The second line, `if(node.firstChild)`, establishes that `node` has at least one child (our base case).  (On a side note, this strikes me as poor design on JavaScript's part...wouldn't it be better to have a method `.hasChildren()`?)

Next we have two `for` loops (has to be the traditional `for` loops, not `forEach`, because both `node.children` and `node.childNodes` return a [`NodeList`](https://developer.mozilla.org/en-US/docs/Web/API/NodeList), which is not arrays, and which generally sucks.)  

Notice how the first `for` loops over `node.children`, and the second `for` loops over `node.childNodes`.  What's the difference?

`node.children` is a NodeList of only the child _element_ nodes, whereas `node.childNodes` is a NodeList of _all_ the children, to include text nodes.  This ensures we print the lowest text leaf node first.

