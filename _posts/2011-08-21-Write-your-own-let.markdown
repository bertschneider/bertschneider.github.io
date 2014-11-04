---
layout: post
title: Write your own let
summary: ... or how to build your own stuff
categories: clojure macro
tags: Clojure Macro
---

I am currently reading [Structure and Interpretation of Computer
Programs](http://www.amazon.de/gp/product/0262011530/ref=as_li_ss_tl?ie=UTF8&tag=herrnorbertde-21&linkCode=as2&camp=1638&creative=19454&creativeASIN=0262011530)
and stumbled upon the mention that closures could be used to implement
the `let`-function. Next thing I knew, the Clojure REPL was running ...

## The First Try

{% highlight clojure %}
user>(defmacro my-let [bindings & body]
  `((fn [~@(flatten (partition 1 2 bindings))]
      ~@body)
    ~@(flatten (partition 1 2 (rest bindings)))))

user> (macroexpand '(my-let [a 6 b 7] (* a b)))
((clojure.core/fn [a b] (* a b)) 6 7)

user> (my-let [a 6 b 7] (* a b))
42
{% endhighlight %}

The basic idea is to generate a closure via the `fn`-function, set the
binding names as parameter names and invoke the function with the
binding values.

As you can see the basic binding functionality is working. But there
are still two problems. First you can only bind primitive values
otherwise the `flatten`-function would _destroy_ the passed in nested
lists. Second you can not access a preceding binding in a succeeding
one.

{% highlight clojure %}
;; Problem 1
user> (macroexpand '(my-let [a 6 b (+ 3 4)] (* a b)))
((clojure.core/fn [a b] (* a b)) 6 + 3 4)

;; Problem 2
user> (macroexpand '(my-let [a 6 b a] (* a b)))
((clojure.core/fn [a b] (* a b)) 6 a)
{% endhighlight %}

## The Second Try

To sort out the problems I switched to nested closures:

{% highlight clojure %}
user> (defmacro my-let2 [bindings & body]
  (if (seq bindings)
    `((fn [~(first bindings)]
        (my-let2 ~(rest (rest bindings)) ~@body))
      ~(second bindings))
    `(eval ~@body)))

user> (my-let2 [a 6 b (+ 3 4)] (* a b))
42

user> (my-let2 [a 6 b a] (* a b))
36
{% endhighlight %}

The idea behind this solution is to generate nested closures so that
preceding bindings can be accessed via the parameter of the outer
closures. As it turns out both problems get fixed by this approach.

The main challenge in writing the second macro was the `(eval ~@body)`
part. In the example the form `(* a b)` will be wrapped in another list
because `body` is an optional parameter. To get rid of the extra
parentheses unquote-splicing `~@` could be used. Sadly, well actually it
makes sense, `~@` can only be used in already escaped lists. Just
wrapping another list around wouldn't make any sense, `~@` just got rid
of it. After a long time of try and error I finally found the `eval`
solution.

## Conclusion

This little example demonstrates once again how powerful Lisp /
Clojure and macros can be. And don't forget to read
[SICP](http://www.amazon.de/gp/product/0262011530/ref=as_li_ss_tl?ie=UTF8&tag=herrnorbertde-21&linkCode=as2&camp=1638&creative=19454&creativeASIN=0262011530).


[![SICP](http://ecx.images-amazon.com/images/I/41CPGEDXMDL._SL160_.jpg)](http://www.amazon.com/Structure-Interpretation-Computer-Programs-Engineering/dp/0262011530?SubscriptionId=AKIAIOCEJJCIN2OCGMCQ&tag=herrnorbertde-21&linkCode=xm2&camp=2025&creative=165953&creativeASIN=0262011530)

I just described the basic idea behind the solution. If you have any
further questions feel free to contact me or write a comment.
