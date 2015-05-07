---
layout: post
title:  "Assert.Eventually.Throws"
date:   2015-05-07 16:23:00
---

Asynchronous programming has never been easy. Back in the heyday of multi-threaded programming, asynchronous programming might lead to unexpected race conditions. Code might evaluate to true one moment, only to become false the next. This non-deterministic behavior made bug reproduction very difficult.

Recently, with the introduction of events and callbacks, life as a developer has become a lot easier. But there are still some rough spots. I recently wrote a library called [RFLine][rf]. [RFLine][rf] is a simple file line reader library. Because it relies on asychronous callbacks to handle the customizable, interesting code portions, some aspects of [RFLine][rf] can be hard to test. For example, I might have the following scenario that I want to test:

~~~ javascript
var reader = require('rfline').reader;

reader('this_does_not_exist.txt')
  .error(function(err) { console.log(err.toString()) })
  .finish();

~~~

First, let me explain this code. To start off, we obtain the reader function with a require call. Then, we pass in the file with a call to reader. We specify the error callback with the error method. Finally, we execute the whole chain with a finish call. Pretty straight-foward.

However, the file **this_does_not_exist.txt** does not exist. Its absence should throw an error and our logging callback should get called. We want to make sure this happens. How do we proceed.

You might point out the existence of an [assert.throws][at] function. [Assert.throws][at] takes in a function, which it then executes. If the function throws an error, then the assertion passes. Otherwise, the assertion fails.

Can we add a test to check if the error gets thrown? Can we do something like the following?

~~~ javascript
assert.throws(reader('this_does_not_exist.txt').finish());
~~~


Without the callback, the error should bubble up to the assert.throws statement.

Unfortunately, this does not work. `finish()` returns, before the error gets thrown. And because of the assertion is synchronous, it is not going to stick around till the error shows up. The assertion would always fail as soon as `finish()` returns.

Perhaps then, what we need is something like the following:


~~~ javascript
assert.eventually.throws(reader('this_does_not_exist.txt').finish());

~~~

However, something like this is also problematic, because we would somehow need to hold up on the assertion resolution. We would also need to signal when the `finish()` actually "finished".

At this point, we might give up and consider reaching out for Domenic Denicola's [chai-as-promised][cp]. But using [chai-as-promised][cp] would require adding [chai-as-promised][cp] and [chai][ch]. Plus, we would have to add a promise library and confuse everything in a promise. Ugh!

Instead, the optimal solution is a lot simpler. We just have to remember that not every check needs to be an assertion wrapped function. We can do something very elegant and organic. Something like the following:

~~~ javascript
reader("this_file_does_not_exist.txt")
  .error(function(err) {
    assert.ok(err.message && err.message.length > 0);
    done();
  })
  .finish(function() {
    done(new Error('Error not thrown'));
  });

~~~
This test will not complete before done gets called. And done will only get called if there is an error, or if the reader finishes without error. And by that time, we will have known the result.

This solution meets my standard:

 - It is simple,
 - It is understandable,
 - And it works.



[rf]: https://github.com/dicksont/rfline
[at]: https://nodejs.org/api/assert.html#assert_assert_throws_block_error_message
[cp]: https://github.com/domenic/chai-as-promised
[ch]: https://github.com/chaijs/chai
