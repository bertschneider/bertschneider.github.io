---
layout: post
title: Assoc vs. Assoc-in vs. Update-in
summary: ... or which one to user
categories: clojure performance
tags: Clojure Performance
---

Currently I am fighting some performance issues in Clojure. While I
was looking out for some tips I came across the somewhat famous blog post [Clojure
performance
tips](http://gnuvince.wordpress.com/2009/05/11/clojure-performance-tips). Now
I want to add one of my own.

### Assoc vs. Assoc-in vs. Update-in

`Assoc`, `assoc-in` and `update-in` can be used to update data structures. While
`assoc` only allows updates on the first level of the structure,
`assoc-in` and `update-in` expect some kind of path expression
pointing at the value to update and a function to produce a new
value. Some examples can be found on [ClojureDocs](http://clojuredocs.org/clojure_core/clojure.core/assoc).

So, how is the performance of this functions? 
*I only did some microbenchmarks, so don't think to much of them*.

First lets have a look at `update-in`. To compare the function we increment the value of `:value` inside a nested map.

{% highlight clojure %}
user> (dotimes [_ 5]
        (time 
         (dotimes [_ 1000000]
           (let [x {:stats {:value 0}}] 
             (update-in x [:stats :value] inc)))))
"Elapsed time: 1915.204366 msecs"
"Elapsed time: 1905.402848 msecs"
"Elapsed time: 1919.48168 msecs"
"Elapsed time: 1919.289938 msecs"
"Elapsed time: 1954.808177 msecs"
nil
{% endhighlight %}

As you can see `update-in` is a very concise function and I would use
it in most cases.

{% highlight clojure %}
user> (dotimes [_ 5]
        (time 
          (dotimes [_ 1000000] 
            (let [x {:stats {:value 0}}]
              (assoc-in x [:stats :value] 
                (inc (get (get x :stats) :value)))))))
"Elapsed time: 1207.653239 msecs"
"Elapsed time: 1192.098058 msecs"
"Elapsed time: 1188.202821 msecs"
"Elapsed time: 1191.817293 msecs"
"Elapsed time: 1183.57983 msecs"
nil
{% endhighlight %}

`Assoc-in` isn't that concise, you manually have to read and update
the value, but it's almost double as fast as `update-in`.

{% highlight clojure %}
user> (dotimes [_ 5]
        (time 
         (dotimes [_ 1000000] 
           (let [x {:stats {:value 0}}] 
             (assoc x :stats (assoc (get x :stats) :value 
               (inc (get (get x :stats) :value))))))))
"Elapsed time: 576.143596 msecs"
"Elapsed time: 571.017181 msecs"
"Elapsed time: 553.624249 msecs"
"Elapsed time: 551.965917 msecs"
"Elapsed time: 549.777064 msecs"
nil
{% endhighlight %}

The `assoc` version looks kind of messy but it's the fastest one so far
and doubles the performance of `assoc-in`. You can clean it up a
little and _make it even faster_ by extracting intermediate results. I
don't know exactly why local variables should make the it faster
(perhapes some kind of JIT-magic?), so test it yourself.

{% highlight clojure %}
user> (dotimes [_ 5]
        (time 
         (dotimes [_ 1000000] 
           (let [x {:stats {:value 0}}
                 stats (get x :stats)
                 value (get stats :value)
                 inc-value (inc value)
                 inc-stats (assoc stats :value inc-value)]
             (assoc x :stats inc-stats)))))
"Elapsed time: 553.294597 msecs"
"Elapsed time: 539.322251 msecs"
"Elapsed time: 529.916189 msecs"
"Elapsed time: 531.741025 msecs"
"Elapsed time: 532.512921 msecs"
nil
{% endhighlight %}

### Summary 

If you _really need_ performance you could think about favoring
`assoc` over the other two functions. But beware the messier code!
