---
layout: post
title: "Enter the dojo: Deferreds and promises"
date: 2012-09-18 17:23
comments: true
sharing: true
categories:  [javascript, dojo]
author: Gonz Saavedra
---

If you've been working with javascript long enough, you probably have
faced a very common asynchronous programming challenge many times:
*Having to execute code after an asynchronous task finishes*.

A common solution is to use *callbacks*, just passing one or more
functions to the asynchronous task so it can call them when the result
is available. Unfortunately as the problems get more complex, like
having different callbacks for handling success/error results, nested
async tasks (and callbacks), tracking progress, communication between
callbacks themselves or all of them together things start to get ugly,
code usually becomes much harder to maintain, errors are harder to
handle, etc...

In fact this problem is so common in the javascript world that
[many implemented solutions][1] exist and a lot  those use promises
or other similar concepts.

**Promises are a simple solution found in many concurrent languages
that provide an effective and efficient way for handling and compose
concurrent tasks**.

The idea is that instead of having the main task passing the callback
to the async task so it can then call it, responsibilities are
flipped: the async task creates and returns a promise (a proxy object)
that allows the main task to bind one or many callbacks. These
callbacks will be automatically executed when the result of the async
task is available.

This approach has many advantages over good ol' callbacks, I'll try to
elaborate on this here while introducing dojo promises implementation
(v1.8). Note that most of what's discussed here applies to other
implementations, specially the ones that follow
[Promises/A CommonJS proposal][0].

<!-- more -->

Dojo Deferreds basics
---------------------

In the dojo toolkit the *dojo/Deferred* module defines the *Deferred*
class which along other modules in *dojo/promises* allow communication
between async tasks using the promises concept. Many dojo modules rely
on deferreds for their async communication, probably the best example
is the *dojo/request* module it can be used by your own modules as
well.

Code first, a simple example to demonstrate how you can create a
Deferred object in a async process and return a promise to the
consumer, who will be able to add a callback which will be executed
when the promise is resolved (and the result becomes available).

<iframe style="width: 100%; height: 300px"
        src="http://jsfiddle.net/gonz/2R3Af/embedded/js,html,result/"
        allowfullscreen="allowfullscreen" frameborder="0"></iframe>

A Deferred object can be in one of three states at any given time:
*in progress* (or unfulfilled), *resolved* or *rejected*. It can only
go from *in progress* to *resolved* or from *in progress* to
*rejected*. This method allow us to make these status transitions:

* ***resolve**(value, strict)*: This method will be called to inform
  the deferred that the async task finished successfully, this will
  fire all the success callbacks added to it's promises through the
  *then()* method.
  * *value* usually holds a result from the async tasks which will be
    passed to the first callback as it param.
  * *strict* is a boolean, when set to true makes the call raise
    an exception if the deferred is already fulfilled. Defaults to
    *false*.
* ***reject**(value, strict)*: Will be called when the async process
  had some kind of error, all error callbacks functions will be
  called. *value* and *strict* params work like the ones described for
  *resolve()* above.

You can check the current status of a deferred (or it promises) by
calling isResolved(), isRejected(), isFulfilled().

As can see in the example above *runAsync()* function returns the
*promise* attribute of the *Deferred* object, this is a
*dojo/promises/Promise* object that shares the *then()* method (and
it's context) with the deferred, so when adding new callbacks through
the promise they will be available for the main deferred object
to call.

As you can guess *then()* is how we add callbacks with the code we
want to be run once the deferred is fulfilled.

* ***then**(callback, errback, progback)*: Add callbacks to
  * *callback* is a function that will be called when the deferred is
  resolved (finishes successfully).
  * *errback* is an optional function that will be called when the
  deferred is rejected.
  * *progback* is an optional function will be called when the deferred
  receives a progress update.


Chaining *then()* calls
-----------------------

*then()* returns a new *Promise*, so you can easily chain many *then()*g
calls or pass the promise to other contexts which may add their own
callbacks (and this detail is key for understanding the power of
promises).

An important detail here is that the callback added first (whether is
a sucess, error or progress callback) will get the result sent by the
*resolve()*/*reject()*/*progress()* call as its first param, the
return value of a callbacks is what the next callback in the *then()*
chain will recieve as first param, allowing callbacks modify or extend
it, as a simple uni-directional simple communication system between them.

<iframe style="width: 100%; height: 300px"
        src="http://jsfiddle.net/gonz/7JdjB/embedded/js,html,result/"
        allowfullscreen="allowfullscreen" frameborder="0"></iframe>


Progress updates
----------------

A great feature of dojo's deferreds/promises are progress callbacks,
They allow the consumer code to get progress updates from the async
process.

*Deferred* defines the progress() method, which will call all
progback functions added using then():

* ***progress**(value, strict)*
  * *update* is passed along to the progress callbacks added through
    *then()*.
  * *strict* works like *resolve()* and *reject()* methods strict param.

Take a look at this simple but cool example that make use of this feature:

<iframe style="width: 100%; height: 300px"
        src="http://jsfiddle.net/gonz/g7ELP/embedded/js,html,result/"
        allowfullscreen="allowfullscreen" frameborder="0"></iframe>


Cancelling async processes
--------------------------

You can also cancel a deferred from the consumer side by calling the
*cancel()* method on a *Promise* object. Note that cancelling
behaviour has to be implemented on the deferred since it doesn't
always makes sense to support that.

<iframe style="width: 100%; height: 300px"
        src="http://jsfiddle.net/gonz/TNZHd/embedded/js,html,result/"
        allowfullscreen="allowfullscreen" frameborder="0"></iframe>

As you can see in this example the cancel behaviour for the deferred
is implemented passing a function to the *Deferred* constructor
function, and later called after the *cancel()* call in the consumer
side *Promise* object.


Composing promises
------------------

Dojo also provide two very useful modules for composing promises:
*dojo/promises/all* and *dojo/promises/first*.

*dojo/promises/all* receives multiple promises and returns a new
promise that will become fullfilled once all of the given promises are
fullfilled.

<iframe style="width: 100%; height: 300px"
        src="http://jsfiddle.net/gonz/65hdt/embedded/js,html,result/"
        allowfullscreen="allowfullscreen" frameborder="0"></iframe>

As you can see in the example the promise created by *all()* recieves
as first param an array of the results ordered in the same order we
passed the original promises. Alternatively you could pass it an
object using some keys to identify each value promise, the *then()*
callback would also recieve an object with the results in
corresponding key.

Similarly *dojo/promises/first* takes many promises and returns a new
promise that will become resolved as soon as any of the orginal promises
gets resolved.

<iframe style="width: 100%; height: 300px"
        src="http://jsfiddle.net/gonz/VhESK/embedded/js,html,result/"
        allowfullscreen="allowfullscreen" frameborder="0"></iframe>


Why promises are better?
------------------------

*Promises* are an awesome solution for many reasons, for me the main
ones are:

### Simple APIs ###

Not having to pollute function signatures with callback params allows
to have cleaner and intuitive APIs, just saying this method returns a
promise is enough.

If you have used libraries like jQuery I bet you had to go back to the
docs more than once to check async functions signatures, is the success
callback is the first or second param? What key does the param object
param expects? was it "complete", "onComplete", "success"?.

### Modular ###

From the consumer perspective this approach makes things much flexible,
you can create as many callbacks as you want and compose them as you
see fit.

You can pass promises around your modules where each can add callbacks
or query the current status of the promise in a modular way, without
affecting the original deferred.

From the "async" side you don't have to think about how to call the
callback, it's just a matter of create the promise and manage it's
states, what happens then is not it's business.


### Composable ###

As you saw above you can compose promises in useful ways and the
resulting code is very clean and intuitive. *then()* chaining,
*all()* and *first()* are great examples, and you can also easily
define your own logic to get a new promise from other(s).

With *callbacks* solution defining several serial callbacks it's a
mess, I have seen those awfully nested functions too many times, of
course you can arrange the code to avoid nesting things too much but
the complexity to follow the code is the same: too high and the worst
thing: each function is responsible to call the next. With promises
you just would chain *then()* calls each function doesn't know
anything about the others.


Resources
---------

 * [dojo/promise/Promise reference][3]
 * [dojo/Deferred reference][4]
 * [dojo/promise/all reference][5]
 * [dojo/promise/all reference][6]
 * [CommonJS Promises/A][0]
 * [Control flow/async modules][1]
 * [Callbacks, Promises, and Coroutines (oh my!): Asynchronous Programming Patterns in JavaScript][2]


[0]: http://wiki.commonjs.org/wiki/Promises/A
     "CommonJS Promises/A"
[1]: https://github.com/joyent/node/wiki/modules#wiki-async-flow
     "Control flow/async modules"
[2]: http://www.slideshare.net/domenicdenicola/callbacks-promises-and-coroutines-oh-my-the-evolution-of-asynchronicity-in-javascript
     "Callbacks, Promises, and Coroutines (oh my!): Asynchronous Programming Patterns in JavaScript"
[3]: http://dojotoolkit.org/reference-guide/1.8/dojo/promise/Promise.html
     "dojo/promise/Promise reference"
[4]: http://dojotoolkit.org/reference-guide/1.8/dojo/Deferred.html
     "dojo/Deferred reference"
[5]: http://dojotoolkit.org/reference-guide/1.8/dojo/promise/all.html
     "dojo/promise/all reference"
[6]: http://dojotoolkit.org/reference-guide/1.8/dojo/promise/first.html
     "dojo/promise/all reference"
