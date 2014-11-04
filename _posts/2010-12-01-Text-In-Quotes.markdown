---
layout: post
title: Text in Quotes
summary: ... or how to quote in Clojure
categories: clojure
tags: Clojure
---

In the last few days I am wandering in the [Land of
Lisp](http://www.amazon.de/Land-Lisp-Learn-Program-Game/dp/1593272812/)
and occasionally wondering if some Common Lisp constructs can be
converted to Clojure.

One of these things is the quoting of _text lists_.

{% highlight clojure %}
  `(there is a ,(caddr edge) going ,(cadr edge) from here.)
{% endhighlight %}

This form will evaluate to something like 
_(THERE IS A DOOR GOING WEST FROM HERE.)_. Now, is there a way to do a
simillar thing in Clojure?

{% highlight clojure %}
  user> '(My first test list)
  (My first test list)
{% endhighlight %}

Simple quotes work as expected but how can we evaluate an expression
inside the quote?

{% highlight clojure %}
  user> '(My ~(inc 1) test list)
  (My (clojure.core/unquote (inc 1)) test list)
  user> `(My ~(inc 2) text list)
  (user/My 3 user/text clojure.core/list)
{% endhighlight %}

The first form unquotes the expression, by the _~_ character, but it didn't get
evaluated. In the second example, the third list, the expression does
get evaluated but the output is kind of ugly. Please notice the first
character, the syntax-quote _&#96;_ (a backquote). This quote resolves the
symbols in the current namespace and prints the fully quallified name
like _namespace/name_ or _fully.qualified.Classname_. You can get more
information on special reader/macro characters at [The Reader](http://clojure.org/reader).

So it seems there is no nice way to translate this Common Lisp
construct to Clojure. If you _do_ know one, please let
me know.

