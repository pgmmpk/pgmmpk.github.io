---
layout: default
keywords: javascript, jQuery, AngularJS, Angular
---
# Current events
This is not about politics or sports (luckily, and sadly), but about styles of event-dispatching APIs in JavaScript.

The variety of styles can make one (me) easily confused. And confusion inexplicably creates an urge for blogging, so here it goes.
 
Event dispatching is straightforward. Managing event subscriptions is not -- this is where all of the aforementioned variety happens (and what is discussed herewithin).
Idea is that one should be able to subscribe to an event, and to unsubscribe. The latter is crucial, if often overlooked, requirement. 

Lets start with the beginning.

###W3 DOM specs
Official W3 DOM specification suggests this:

{% highlight javascript %}
function listener(evt) {
	alert(evt);
}

element.addEventListener('keypress', listener);
...
element.removeEventListener('keypress', listener);
{% endhighlight %}

Problem? Many:

1. too verbose
2. does not allow call chaining
3. you can not inline the listener as an anonymous function, because you need a reference
   to it so that you can pass it later to the `removeEventListener`.

### jQuery: on() and off()
jQuery's answer to this is `on`/`off` metaphor:

{% highlight javascript %}

function listener(evt) {
	alert(evt);
}

$element.on('keypress', listener);
...
$element.off('keypress', listener);

{% endhighlight %}

Much better! It is chainable, succint. However you still need a name for the listener
to be able to unsubscribe. Alternatively you can use somewhat clunky (IMHO) "namespace" mechanism:

{% highlight javascript %}

$element.on('keypress.mine', function(evt) {
	alert(evt);
});
...
$element.off('keypress.mine');

{% endhighlight %}

Here we effectively give a name to anonymous function, but attach this name to the event as a "namespace". Using this "namespaced" event 
name one can cleanly detach from event source.

### Attach returns detacher (AngularJS)

AngularJS scope events[^*] have `on`, but not `off`. This is because call to `on` now returns a *function that will detach listener from the event source*. 

{% highlight javascript %}

var detach  = $scope.on('keypress', function() {
	alert(evt);
});
...
detach();

{% endhighlight %}

This nicely removes redundancy and avoids messing with event names. Also it allows anonymous handlers to be registered
and detached. Is it perfect? No! I want chaining of `on()` calls (like jQuery).

### Subscription
We can improve on Angular's approach by introducing an intermediate `subscription` object. Subscription keeps track of attached listeners and allows
to detach them, but only all together. Here is how it works: 

{% highlight javascript %}

var subscription = $element.subscription();

subscription.on('keydown', function() {
	alert('keydown');
}).on('keyup', function() {
	alert('keyup');
});
...
subscription.release(); // releases all attached handlers

{% endhighlight %}

And we can make it even shorter by slightly abusing the chain notation:

{% highlight javascript %}

var subscription = $element.subscription().on('keydown', function() {
	alert('keydown');
}).on('keyup', function() {
	alert('keyup');
});
...
subscription.release(); // releases all attached handlers

{% endhighlight %}

All said, each method has its merits. Personally I would go with the subscription style. Its a little more verbose than I'd like
but it is simple to understand and the very existence of `subscription` object in the API reminds the user (me) to close it
when it is no longer needed.

[^*]: Do not confuse scope events with with DOM events in AngularJS, latter are handled by jqLite -- Angular lightweight version 
of jQuery -- and therefore DOM events API is pretty much the same as that of jQuery.