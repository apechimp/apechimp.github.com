---
layout: post
title: Consider not using mocha
category: software
subcategory: node.js
---

If you use node.js and you are aware of unit testing, you've most likely used
[mocha](https://github.com/visionmedia/mocha) at least once. I've decided to
ditch mocha for [lab](https://github.com/hapijs/lab) because the way mocha
handles exceptions within tests can lead to confusing results. Consider the
following.

{% highlight javascript %}
describe('The issue is', function() {
  it('this threw an error', function(done) {
    setTimeout(function() { throw 'lol'; }, 10);
    done();
  });

  it('but this should pass', function(done) {
    setTimeout(done, 20);
  });
});
{% endhighlight %}

Yields the following output from mocha.

![mocha output][mochaOutput]

Even though the "this threw an error" test threw the error, "but this should
pass" is marked as failing. It's fairly clear what is happening here, but as
you introduce more tests, it becomes less clear why "but this should pass" is
marked as failing. The reason for this confusion is because 
[mocha listens](https://github.com/visionmedia/mocha/blob/c20c3022f063d0fa1d30b9273bdff27a7a6ed952/lib/runner.js#L595) 
to the 
[``uncaughtException`` event](http://nodejs.org/api/process.html#process_event_uncaughtexception).
In listening to ``uncaughtException``, mocha will fail whichever test is
running when it handles the ``uncaughtException``. This is where lab comes in.
Lab follows the documentation of ``uncaughtException`` to use
[domains](http://nodejs.org/api/domain.html) instead. By doing so, lab is able
to identify which test created the ``uncaughtException`` as can be seen in the
following output.

![lab output][labOutput]

This identifies the test which caused the exception instead of the test which
the exception occurred during. This is much more useful information if you want
to figure out what is causing the exception, so consider using lab over mocha.

[mochaOutput]: http://i921.photobucket.com/albums/ad56/apechimp/ScreenShot2014-09-08at85637AM.png
[labOutput]: http://i921.photobucket.com/albums/ad56/apechimp/ScreenShot2014-09-08at91234AM.png
