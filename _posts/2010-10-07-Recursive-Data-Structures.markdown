---
layout: post
title: Recursive Data Structures
summary: ... and how to build them in Clojure
categories: clojure recursive datastructures
tags: Clojure DataStructures
---

Clojure only supports immutable data structures. So, how do you create a
recursive reference? Some weeks ago I asked this question an #clojure
and got this code as answer:

{% highlight clojure %}
(let [x (atom {}) y (atom {:x x})] (swap! x assoc :y y)) 
{% endhighlight %}

_No further explanation will be given. Just try it at the REPL. ;-)_

