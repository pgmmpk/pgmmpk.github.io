---
layout: default
keywords: javascript, jQuery, AngularJS, Angular
---
### Current events
What is the best user-facing API for working with events emitted by DOM elements? 
The task is to be able to subscribe to an event, and to unsubscribe. The latter is 
crucial, if often overlooked, requirement. 

Lets start with the begining.

##W3 DOM specs
Official W3 DOM specification suggests this:

{% highlight javascript %}
function listener(evt) {
	alert(evt);
}

element.addEventListener('keypress', listener);
element.removeEventListener('keypress', listener);
{% endhighlight %}

Problem? Many:

1. too verbose

2. does not allow chaining

3. you can not inline listener parameter as an anonymous function, because you need a reference
to it so that you can pass it to the `removeEventListener`.

### jQuery: on() and off()
jQuery's answer to this is `on`/`off` metaphor:

{% highlight javascript %}

function listener(evt) {
	alert(evt);
}

$element.on('keypress', listener);
$element.off('keypress', listener);

{% endhighlight %}

Much better! It is chainable, succint. However you still need a name for the listener to be able to unsubscribe. Alternatively you can use
somewhat clunky [IMHO] "namespace" mechanism:

{% highlight javascript %}

$element.on('keypress.mine', function(evt) {
	alert(evt);
});
$element.off('keypress.mine');

{% endhighlight %}

Here we effectively give a name to anonymous function, but attach this name to the event as a "namespace". Using this "namespaced" event 
name one can cleanly detach from event source.

### Attach returns detacher (AngularJS)

AngularJS has `on`, but not `off`. This is because call to `on` now returns a *function that will detach listener from the event source*. 

{% highlight javascript %}

var detach  = $scope.on('keypress', function() {
	alert(evt);
});
...
detach();

{% endhighlight %}

This nicely removes redundancy and avoids messing with event names. Also it allows anonymous handlers to be registered and detached. Is it perfect? No! I want chaining
of `on()` calls (like jQuery).

### Subscription
We can improve on Angular's approach by introducing an intermediate `subscription` object. Subscription keeps track of attached listeners and allows
to detach them, but only all together. Here is how it look: 

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

All said, each method has its merits. Personally I would go with the subscription style, just because it "reminds" user 