---
layout: post
title:  "JavaScript: Math.floor versus | 0 Quirks"
date:   2015-02-09 14:09:10
---

It is pretty much common knowledge among JavaScript engineers that this:

~~~ javascript
Math.floor(2.2)
~~~

can be replaced with it's shorter cousin

~~~ javascript
2.2 | 0
~~~

Both yield the same answer ***2***. Let me explain. According to what I have been told, when JavaScript sees the second example, it assumes that the developer is asking the interpreter to Bitwise OR the first operand ***2.2*** with the second operand ***0***. Bitwise OR is only defined for binary bit values, such as ***101*** or ***1011***. In a lot of languages, we have loosened the typing restrictions allowing the bitwise OR operate to take in integer values as well. For example we can do write the following in Ruby:

~~~ ruby
5 | 3
~~~

This is equivalent to:

~~~ ruby
0b101 | 0b11
~~~

Both produce the same answer ***7***. The ruby interpreter sees ***5*** and ***3*** in the first case. It then assumes that the developer is actually asking for ***0b101*** \| ***Ob11***, but is too lazy to write out the ones and zeros. The interpreter does this conversion for the developer.

JavaScript takes this even further than Ruby by allowing same Bitwise OR operator to operate on float values as well. For example, we can write:

~~~ javascript
5.4 | 3.1
~~~

and this would be equivalent to:

~~~ javascript
5 | 3
~~~

Again, both produce the same answer ***7***. Because all numbers in JavaScript are represented internally as floats, ***5*** \| ***3*** is actually ***5.0*** \| ***3.0*** internally. So the question then becomes, how do we handle a bitwise operation that has been mathematically defined for binary values, extended in many languages to operate on their integer equivalents, to take on a totally new, non-isomorphic type floats. JavaScript's answer is remarkably elegant. We treat non-integer floats as their floored equivalents, and then calculate the results from that intermediary. So for example ***5.4*** would be converted to ***5.0*** and ***3.1*** would be converted to ***3.0***, before the result is calculated.

Since ***2.2*** is equivalent to ***2.0*** in the JavaScript bitwise OR domain, and since ORing any number with zero, produces that same number. We can write:

~~~ javascript
2.2 | 0
~~~

and this would be equivalent to

~~~ javascript
2.0 | 0
~~~

or

~~~ javascript
2 | 0
~~~

Any of these should yield the same answer as Math.floor, since 2.2 gets down converted to 2.0. But are these equivalent in JavaScript for any number?

And what may surprise many people here, the answer is no. How can that be you might ask? You have explained the preceding model pretty well. I totally get what is happening under the hood. And now you are shattering this whole beautiful mental conception I have in my mind, by telling me that Math.floor might not be the same as this bitwise OR handiwork.

Well, I can tell you that if this handiwork always worked, then it would not fail the following example.

~~~ javascript
Number.MAX_SAFE_INTEGER | 0
~~~

ORing the maximum safe integer value representable in Javascript with 0 should yield the same number ***Number.MAX_SAFE_INTEGER***.

Not true.

Running this snippet in both Chrome and Firefox produces an unmistakable ***-1***.

WTF is going on, you might ask at this point. Well, because the Bitwise OR operator is defined only for 32-bit values, but ***Number.MAX_SAFE_INTEGER*** goes above those 32 bits. Javascript ignores those extra bits. Since ***Number.MAX_SAFE_INTEGER*** has all its bits set, it produces a 32-bit value with all the bits set.  We then perform the Bitwise OR operation. This produces the same value with all its bits set, or ***-1*** in integer land.

Indeed, we were to run:

~~~javascript
(Math.pow(2, 32) + 1) | 0
~~~

We would get ***1*** and not ***4294967297*** or ***-1*** as expected. And this, folks, is your JavaScript in a snippet. The answer to most questions is a maybe. One should never be surprised.
