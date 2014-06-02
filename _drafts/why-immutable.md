---
layout: post
title: "Why Objects Should Be Immutable"
date: 2014-06-09
tags: quality immutability
description:
  The article gives arguments about why classes/objects
  in object-oriented programming have to be immutable,
  i.e. never change their encapsulated state
keywords:
  - object design
  - why immutable
  - immutability of objects
  - agruments for immutable objects
  - why immutable objects are better
  - immutability in oop
  - oop immutable objects
  - immutable classes
  - immutable class
  - immutable object
  - best practices immutable
---

In object-oriented programming, an object is
[immutable](http://en.wikipedia.org/wiki/Immutable_object) if its state can't be
modified after it is created. In Java, a good example of an immutable
object is [`String`](http://docs.oracle.com/javase/7/docs/api/java/lang/String.html).
Once it is created we can't modify its state. We
can request it to create new strings, but its own state will never change.

However, there are not so many immutable classes in JDK. Take, for example,
class [`Date`](http://docs.oracle.com/javase/7/docs/api/java/util/Date.html).
It is possible to modifiy its state using `setTime()`.

I don't know why JDK designers decided to make these two very similar classes
differently. However, I believe that the design of a mutable `Date` has
a lot of flaws, while the immutable `String` is much more in spirit
of object-oriented paradigm.

Moverover, I think that **all classes should be immutable in a perfect
object-oriented world**. Unfortunately, sometimes, it is technically not
possible due to limitations of, say, JVM. However, we should aim for the
best. Hopefully, in the future, there will be an object-oriented language without mutable
objects at all.

This is an incomplete list of arguments in favor of immutability:

 * immutable objects are simpler to construct, test, and use
 * truely immutable objects are always thread-safe
 * they help to avoid temporal coupling
 * their usage is side-effects free
 * identity mutability problem is avoided with them
 * they always have failure atomicity

Let's discuss most important of them.

## Thread Safety

The first and the most obvious argument is that immutable objects are
thread-safe. That means that multiple threads can access the same
object at the same time, without being afraid of clashing with
another thread.

If no object methods can modify its state, no matter how many of them and
how often are being called parallel &mdash; they will work in their
own memory space, in stack.

Goetz et al. explained this advantage of immutable objects in more
details in their very famous book
[Java Concurrency in Practice](http://www.amazon.com/Java-Concurrency-Practice-Brian-Goetz/dp/0321349601)
(highly recommend to read).

## Avoiding Temporal Coupling

Here is an example of temporal coupling (the code makes
two consecutive HTTP POST requests, where the second one contains
HTTP body):

{% highlight java linenos=table %}
Request request = new Request("http://example.com");
request.method("POST");
String first = request.fetch();
request.body("text=hello");
String second = request.fetch();
{% endhighlight %}

This code works. However, everybody has to remember that
the first request should be configured before the second one may
happen. If we decide to remove the first request from the script, we will
remove the second and the third line, and won't get any errors
from the compiler:

{% highlight java %}
Request request = new Request("http://example.com");
//request.method("POST");
//String first = request.fetch();
request.body("text=hello");
String second = request.fetch();
{% endhighlight %}

Now the script is broken, although compiled without errors. This is
what temporal coupling is about &mdash; there is always some hidden
information in the code that a programmer has to remember. In this example,
we have to remember that the configuration done for the first request
is used for the second one. We have to remember that the second request
should always stay together and be executed after the first one.

If `Request` class would be immutable, the first snippet won't work in the
first place, and would have been rewritten like that:

{% highlight java %}
final Request request = new Request("");
String first = request.method("POST").fetch();
String second = request.method("POST").body("text=hello").fetch();
{% endhighlight %}

Now, these two requests are not coupled. We can safely remove the first
one, and the second one will remain working correctly. You may point out
that there is a code duplication. Yes, we should get rid of it and re-write
the code:

{% highlight java %}
final Request request = new Request("");
final Request post = request.method("POST");
String first = post.fetch();
String second = post.body("text=hello").fetch();
{% endhighlight %}

See, refactoring didn't break anything and we still
don't have temporal coupling. The first request
can be safely removed from the code without affecting the second one.

I hope this example demonstrates that the code manipulating
immutable objects is more readable and maintainable, because
it doesn't have temporal coupling.

## Avoiding Side Effects

Let's try to use our `Request` class in a new method (now it is mutable):

{% highlight java %}
public String post(Request request) {
  request.method("POST");
  return request.fetch();
}
{% endhighlight %}

Let's try to make two requests &mdash; the first with GET method and the second
with POST:

{% highlight java %}
Request request = new Request("http://example.com");
request.method("GET");
String first = this.post(request);
String second = request.fetch();
{% endhighlight %}

Method `post()` has a "side effect" &mdash; it makes changes to the mutable
object `request`. These changes are not really expected in this case. We expect
it to make a POST request and return its body. We don't want to read its
documentation just to find out that behind the scene it also modifies
the request we're passing to it as an argument.

Needless to say that such side effects lead to bugs and maintainability
issues. Would be much better to work with an immutable `Request`:

{% highlight java %}
public String post(Request request) {
  return request.method("POST").fetch();
}
{% endhighlight %}

In this case we may not have any side effects. Nobody can modify our `request`
object, no matter where it is used and how deep through the call stack
it is passed by method calls:

{% highlight java %}
Request request = new Request("http://example.com").method("GET");
String first = this.post(request);
String second = request.fetch();
{% endhighlight %}

This code is perfectly safe and side effect free.

## Avoiding Identity Mutability

In every object-oriented language

## Failure Atomicity

## Arguments Against Immutability

There are a number of arguments against immutability.

1. **Immutability is not for enteprise systems**.
Very often I'm hearing people saying that immutability is a fancy
feature, while absolutely impractical in real enterprise systems.
As a counter-argument I can only show some examples of
real-life applications that contain only immutable Java objects.
[jcabi-http](http://http.jcabi.com),
[jcabi-xml](http://xml.jcabi.com),
[jcabi-github](http://github.jcabi.com),
[jcabi-s3](http://s3.jcabi.com),
[jcabi-dynamo](http://dynamo.jcabi.com),
[jcabi-simpledb](http://simpledb.jcabi.com)
are all Java libraries that work solely with immutable classes/objects.
[netbout.com](https://github.com/netbout/netbout) and
[stateful.co](https://github.com/sttc/stateful) are web applications
that work solely with immutable objects.

2. **It's cheaper to update an existing object than create a new one**.
Oracle [thinks](http://docs.oracle.com/javase/tutorial/essential/concurrency/immutable.html) that
&ldquo;The impact of object creation is often
overestimated, and can be offset by some of the efficiencies
associated with immutable objects. These include decreased overhead
due to garbage collection, and the elimination of code needed
to protect mutable objects from corruption.&rdquo;

## References

More on this subject:

http://www.javapractices.com/topic/TopicAction.do?Id=29
http://stackoverflow.com/questions/214714/mutable-vs-immutable-objects
http://programmers.stackexchange.com/questions/151733/if-immutable-objects-are-good-why-do-people-keep-creating-mutable-objects
http://www.javaranch.com/journal/2003/04/immutable.htm