---
layout: post
title: Recursive Data Structures
tags: Clojure DataStructures
---

# Recursive Data Structures

Clojure only supports immutable data structures. So, how do you create a
recursive reference? Some weeks ago I asked this question an #Clojure
and got this code as answer:

{% highlight clojure %}
(let [x (atom {}) y (atom {:x x})] (swap! x assoc :y y)) 
{% endhighlight %}

_No further explanation will be given. Just try it at the REPL. ;-)_

