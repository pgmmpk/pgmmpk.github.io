---
layout: default
keywords: javascript, angular, angularjs, yeoman
---
### AngularJS Suggested Directory Structure Is Lame
Many [AngularJS](http://angularjs.org) seeds (including [Yeoman](http://yeoman.io)'s AngulaJS plugin)
impose the following structure on project source files:

{% highlight bash %}
js/
   app.js
   controllers.js
   services.js
   filters.js
{% endhighlight %}

or "more sophisticated" version, that may look like this:

{% highlight bash %}
js/
   app.js
   controllers/
      controller1.js
      controller2.js
   services/
      service1.js
      service2.js
      service3.js
   filters/
      filter1.js
{% endhighlight %}

In short, the suggestion is to split source files by the role of object they are defining : controller, service, or filter.

I say: this is plain wrong. Put everything in one file. As project grows, split **by module**, where module defines a unit of
reusable functionality (most often --- a widget).

Why? Just consider the amount of file-switching activity that one has to go thru to make a simple module. Then, once you done,
you may want to reuse this code in another project. You'll have to hunt for all these pieces (again!) and copy them over one by one.
Makes sense? No!

Here is more sane directory structure:
{% highlight bash %}
js/
   app.js
   widget1.js
   widget2/
     service.js
     controllers.js
     filters.js
{% endhighlight %}

Simple module (a widget) fits in a single file, <code>widget1.js</code>. More complex module is split across several JS files (but 
not necessarily by object role as shown). In any case, module is logically encapsulated within a single element of file structure.
Take it and copy - everything is there, in one place.