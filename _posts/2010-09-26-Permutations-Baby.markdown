---
layout: post
title: Permutations Baby
summary: ... or how to generate all permutations of a given set af values.
categories: clojure permutations 
tags: Clojure Permutations
---

A few weeks ago a colleague asked if I knew a nice method to generate
permutations of a given set of values. I didn't know one but once
again Clojure came to the rescue.

{% highlight clojure %}
user> (defn perm [counter vals]
            (if (<= counter 1)
                (for [x vals]
                     [x])
                (let [perms (perm (dec counter) vals)]
                     (for [v vals p perms]
                          (conj p v)))))
#'user/perm
user> (perm 2 [0 1])
([0 0] [1 0] [0 1] [1 1])
user> (perm 3 [0 1])
([0 0 0] [1 0 0] [0 1 0] [1 1 0] [0 0 1] [1 0 1] [0 1 1] 
[1 1 1])
user> (perm 3 [\A \B \C])
([\A \A \A] [\B \A \A] [\C \A \A] [\A \B \A] [\B \B \A] 
[\C \B \A] [\A\C \A] [\B \C \A] [\C \C \A] [\A \A \B] 
[\B \A \B] [\C \A \B] [\A \B \B] [\B \B \B] [\C \B \B] 
[\A \C \B] [\B \C \B] [\C \C \B] [\A \A \C] [\B \A \C] 
[\C \A \C] [\A \B \C] [\B \B \C] [\C \B \C] [\A \C \C] 
[\B \C \C] [\C \C \C])
user> (perm 10 [0 1])
([0 0 0 0 0 0 0 0 0 0] [1 0 0 0 0 0 0 0 0 0] 
[0 1 0 0 0 0 0 0 0 0] [1 1 0 0 0 0 0 0 0 0] 
[0 0 1 0 0 0 0 0 0 0] [1 0 1 0 0 0 0 0 0 0] 
[0 1 1 0 0 0 0 0 0 0] [1 1 1 0 0 0 0 0 0 0] 
[0 0 0 1 0 0 0 0 0 0] [1 0 0 1 0 0 0 0 0 0] 
[0 1 0 1 0 0 0 0 0 0] ..... ;shouldn't have done that
{% endhighlight %}


