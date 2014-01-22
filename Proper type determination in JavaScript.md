Proper type determination in JavaScript
=======================================

This is an article I wrote two years ago and was published on my own website
(which is now retired). The discussion uses details that were relevant with
the up-to-date software then (jQuery 1.7.1 and Chrome 22) and I thought I
publish the article here just as information on JavaScript, but nevertheless I
checked the most recent jQuery's source (version 1.10.2 and 2.0.3) and sadly
although I couldn't find the same exact code (see below), they still use the
same technique, just slightly refactored. Actually it is my opinion that
jQuery's (and some similar I don't care to include by name) code is rather
poor. It may be expressive, but when you write a library that the whole world
depends on, it is of higher importance to make it screaming fast, not to make
every bit of its *internals* expressive or not even expressive but cleverly
written.

For example here is an excerpt from jQuery 2.0.3:

```js
// Populate the class2type map
jQuery.each("Boolean Number String Function Array Date RegExp Object Error".split(" "), function(i, name) {
    class2type[ "[object " + name + "]" ] = name.toLowerCase();
});
```

Why you "ninjas" need to write the types as a string and split it to convert
it to array and call jQuery.each() which will make a function call for each
item and do this on every page load for every page in the world? Couldn't you
just write the array manually instead of splitting it and for-loop over it
instead of calling function for each item (not to mention jQuery.each() is not
as simple as Array.forEach() and do much more stuff so it is slower and needs
additional checks)? Or you could skip all of this because you know all the
values in advance and don't need to process them in any way and spend any CPU
on it and just write it like this:

```js
var class2type = {
    '[object Boolean]': 'boolean',
    '[object Number]': 'number',
    '[object String]': 'string',
    '[object Function]': 'function',
    '[object Array]': 'array',
    '[object Date]': 'date',
    '[object RegExp]': 'regexp',
    '[object Object]': 'object',
    '[object Error]': 'error'
};
```

### Anyway... proper type determination in JavaScript

Many people these days (2011-11-25), including reputable libraries like
jQuery, use the technique of converting their variables to strings to
determine their type. I'm presenting the benefits of using the built in
JavaScript operator "instanceof" rather than calling ".toString()". For
example here is a random excerpt from jQuery 1.7.1:

```js
if ( toString.call(array) === "[object Array]" ) {
    //...
}
```

What this does is, and this is the absolute minimum it needs to do and there
is much more in the JavaScript engine internals that is going on when calling
functions and constructing objects:

1. Find the toString variable and check its collection of properties for
   'call'
2. Check if call is a function
3. Construct function execution environment like stack, etc
4. Since .call() is a function used to call other functions perform the
   previous step at least once more
5. In the actual toString function of the array construct a string object
   (memory allocation, etc) and populate it with '[object Array]'
6. Return this string object for the === operator
7. Now the === operator is an identity operator so it will check the type of
   the vars on the both sides and check for identity of the objects
8. Although obviously these are two different string objects and they are not
   identical the === operator will still act as a == operator and check their
   contents
9. The actual function for comparing the contents of the strings need to
   iterate the strings and compare each byte - the longer the string the
   longer the cycle
10. Return true to the if and continue...

If we do it properly and use the instanceof operator which JavaScript has, the
code will look like this, not only it looks better as it is more expressive
and readable and short and easy to type, it is also much faster:

```js
if ( array instanceof Array ) {
    //...
}
```

What this does is:

1. Iterate over the chain of prototypes of the right variable and see if some
   prototype matches the prototype of the the left variable (in our case this
   cycle is repeated only once)
2. Return true to the if and continue...

You can <a href="http://jsfiddle.net/bobef/DQ5E6/" target="_blank">run</a> the following snippet:

```js
var toString = Object.prototype.toString; //excerpt from jQuery
var array = [ 1, 2, 3 ];

$( '#div0' ).text( 'Testing 1 million loops...' );
    var t1 = (new Date()).valueOf();
    var n = 0;
    for ( var i = 0; i < 1000000; ++i ) {
        if ( toString.call( array ) === '[object Array]' ) {
            ++n;
        }
    }
    t1 = ( (new Date()).valueOf() - t1 ) / 1000;
$( '#div1' ).text( 'The first snippet ate ' + t1 + ' seconds of your users\' experience' );

$( '#div2' ).text( 'Testing 1 million loops...' );
    var t2 = (new Date()).valueOf();
    n = 0;
    for ( var i = 0; i < 1000000; ++i ) {
        if ( array instanceof Array ) {
            ++n;
        }
    }
    t2 = ( (new Date()).valueOf() - t2 ) / 1000;
$( '#div3' ).text( 'The second snippet ate ' + t2 + ' seconds of your users\' experience' );
$( '#div4' ).text( 'Second snippet is ~' + parseInt( t1 / t2 ) + 'x faster' );​
```

On my machine (Chrome 22) the results are:

```
Testing 1 million loops...
The first snippet ate 0.803 seconds of your users' experience
Testing 1 million loops...
The second snippet ate 0.022 seconds of your users' experience
Second snippet is ~36x faster
```

Now on Chrome 30 and my newer laptop the results are ~20x faster, not ~36x,
but it is still a huge difference and the point is still as valid.

Yes, a program is not likely to run such call one million times, and yes the
results may vary on different browsers. If not else the string method is ugly
programming practice. And also on the server side such small difference can be
significant if you have a lot of visitors. Not for the single visitor, but for
your server where you pay for CPU time and also want to save CPU time so it
can be distributed to more visitors.

Enough said for the speed. Now lets look at the lame aspects of instanceof.
There are few catches when you use instanceof. First arrays for example are
both array and object so it will be instanceof both Array and Object. Strings
and numbers and booleans have some kind of special treatment in JavaScript.
For example "asd" is not an object but still you can call String methods on it
like "asd".substr(1). Same goes for numbers. And finally you can use
instanceof to check if a variable is of some type, but not to determine its
type. For the latter you need to use other methods like typeof or
Object.getPrototype(), but if you need the exact name of the type as string
there is no clean way and you have to resort to hacks for not-built-in types.
You can run the following in the browser console to see the results if you
want.

```js
// strings
typeof "asd"; // string
"asd" instanceof Object || String( "asd" ) instanceof Object; // false
"asd" instanceof String || String( "asd" ) instanceof String; // also false, naturally

typeof new String( "asd" ); // object
new String( "asd" ) instanceof Object; // true
new String( "asd" ) instanceof String; // also true, naturally
"asd".substr( 1 ) == ( new String( "asd" ) ).substr( 1 ); // true

// numbers
typeof 5; // number
5 instanceof Object || Number( 5 ) instanceof Object; // false
5 instanceof Number || Number( 5 ) instanceof Number; // also false, naturally

typeof new Number( 5 ); // object
new Number( 5 ) instanceof Object; // true
new Number( 5 ) instanceof Number; // also true, naturally

// booleans
typeof true; // boolean
true instanceof Object || Boolean( true ) instanceof Object; // false
true instanceof Boolean || Boolean( true ) instanceof Boolean; // also false, naturally

typeof new Boolean( true ); // object
new Boolean( true ) instanceof Object; // true
new Boolean( true ) instanceof Boolean; // also true, naturally

// arrays are ok
[] instanceof Object; // true
[] instanceof Array; // true

// a proper string test to cover the both cases
if ( typeof a == 'string' || a instanceof String ) {
    // ...
}

// a proper number test to cover the both cases
if ( typeof a == 'number' || a instanceof Number ) {
    // ...
}

// a proper boolean test to cover the both cases
if ( typeof a == 'boolean' || a instanceof Boolean ) {
    // ...
}

// a proper test for array
if ( a instanceof Array ) {
    // ...
}

// a proper test for function
if ( a instanceof Function ) {
    // ...
}

// if you are testing for objects you must beware because anything can be object too
if ( o instanceof Object ) {
    // we are expecting a {} here, but not [] or String or Number,
    // but they are still objects, not to mention functions and custom types
    // depending on your needs you may need to run second test besides instanceof
    if ( Object.getPrototypeOf( o ) === Object.prototype ) {
        // ...
    }
}

// you can use instanceof to test your custom objects
function A () {}
A instanceof Function; // true
new A instanceof Object; // true
new A instanceof A; // true
new A instanceof Function; // false
```

You can find the functions
[Object.isObject()](https://github.com/Perennials/prototype-js/blob/master/Object.js),
[String.isString()](https://github.com/Perennials/prototype-js/blob/master/String.js),
[Number.isNumber()](https://github.com/Perennials/prototype-js/blob/master/Number.js)
and
[Boolean.isBoolean()](https://github.com/Perennials/prototype-js/blob/master/Boolean.js)
in my [Prototype](https://github.com/Perennials/prototype-js) library for
JavaScript.


Authors
-------
Borislav Peev (borislav.asdf at gmail dot com)