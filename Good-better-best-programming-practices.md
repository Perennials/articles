Good, better, best... programming practices
===========================================

I was just checking how GitHub repositories are trending and I found one
saying "JavaScript Style Guide". To be honest first I got a bit annoyed as I
thought "Oh great, the next style guide", but I took a look anyway to update
myself on what are the trendy JavaScript guides. Actually it wasn't what I was
expecting, it was rather good. But some "good" practices caught my attention
as they differ from my experience what is good and I created some tickets and
inspired me to write an article.

#### My background as practitioner
First let me say something about myself so my opinion on this matter can be
related to something. My programming experience is leaning towards a second
decade and I have spent ten of thousands of hours doing it. I read two and
half books on programming and I all know I learned though my hard work and
personal trial and error and always feel skeptical about "style guides" and
similar. I wrote some software in my time in variety of languages
(C/C++/D/PHP/JavaScript/HTML/CSS and a taste of others) and I have several
semi-finished attempts of writing a language/compiler myself. Some of the
software I wrote felt pretty polished and was looking very good and was very
useful so I have some achievements that I'm quite proud of. Yet I consider
__my biggest achievement is my coding style__ and this is the one
that took the longest time and effort to achieve. I have changed my coding
style several times and every time this change took years and hundreds and
even thousands of hours of programming. Particularly in the recent years
because I wasn't directly interested in the results of my work (because I was
working on backends and my passion is for UIs) I found joy in perfecting my
coding style and skills. Plus I have already realized the importance of
"beautiful code" so in these recent years I paid particular attention to my
style, what constitutes a good style and good practices.

Right now my code looks likes this:

```js
function myFunction () {
    if ( something === true || somethingElse < 5 ) {
        call.somethingElse( something, somethingElse );
    }
}
```

It took me a year to reach this from my last level which was very similar:

```js
function myFunction() {
    if(something === true || somethingElse < 5) {
        call.somethingElse(something, somethingElse);
    }
}
```

And this took two years from the previous level:

```js
function myFunction() {
    if(!something || somethingElse < 5) call.somethingElse(something, somethingElse);
}
```

And earlier before that I was using even less spacing and wasn't paying any
attention to code style. I was happy to write very long lines of code and to
put lot of code in small space, I only cared what the software does and it
took many years before I started to even make effort to write consistently
looking code. This has to do with the fact that I always worked alone and if I
worked with others my progress would have been faster, but anyway __all I care
about now in programming is "beautiful code".__ Beautiful code is what
produces beautiful results. So I wrote all this introduction about myself to
point out that I'm someone who is paying lot of attention on coding style. And
having said all of this what is a good programming practice?

#### There is only one good programming practice
and it is to do the right thing for the context. To achieve this "right thing"
is the beauty itself and what makes it beautiful, and since we are talking
about coding this is beautiful code. Beautiful code is beautiful to write,
beautiful to read, it performs beautiful result that is beautiful to make use
of and beautiful to look at.

- But sure there must be a perfect guideline to achieve this?
- No there isn't.
- Why are these fellows in GitHub and elsewhere putting these guidelines if
  this isn't so?
- They certainly are not wrong, they are sharing their valid experience.
- So what guideline to follow?
- Write beautiful code.
- But what is it, this is so vague, what if my boss doesn't like it?
- He will like it if it is beautiful.
...

OK, here is what I mean. For example lets discuss code structure. For example
in PHP. PHP received namespaces not long ago and now everyone is using them
like crazy, imitating Java packages. It seems widely accepted this improves
code structure and organization. Generally speaking I agree, namespaces are
great, classes are great, interface are great, etc. But what if I can solve my
problem with two functions. Why do I need to create a class in a namespace in
two directories? What if my two functions don't fit together very well? Should
I create two classes? What about the performance of this - loading several
files for two functions? Should I apply other popular programming practices,
say something like "dependency injection"? Should I write unitests for these
classes? Should I mock the classes? But maybe there is other best practice out
there that PHP doesn't support. Should I go further and twist PHP to implement
"in software" something that is not in the language. And so on. This is not
beautiful. We write the code to solve a problem or in other words archive
something. The problem is the only context to which the beauty of the result
and our code relates. Maybe the problem is complex application that works on
1000 servers, it may be simple problem that is solved with two functions and
50 lines of code. The problem could be we just want to have some fun with
code. We may just need to scratch/draft/prototype something. Maybe we don't
need to go though all the good practices because we need to just get a
picture, a draft, of what we are doing and will discard this draft. Too good
is not good. In this example creating two files with two classes and two
directories is overkill and is not beautiful, unless we are doing it to test
structure, or learn about namespaces and organisation or something similar.

Similar example. Lets say I have some piece of code and I want to write few
small tests for it. Do I need to install PHPUnit which has million
dependencies and I need to read manual how to use it and write classes for the
tests. Or I just create a 10 line script in 5 minutes which tests few
conditions and outputs "OK" or "NOT OK"? I say the latter. But in other
context going for the long way may be better. For example if I am doing even a
small project, maybe for fun - open source on GitHub, or for profit with
longer term maintenance in mind. In both cases I would use a more elaborate
unitesting solution. In the first case because I'm doing it for fun, to learn
something, because I like it. In the second case because I want it to be
reliable and save me trouble in the future and have stable results and produce
good product for the clients. But because it is right for the context, not
because it is right to write unitests.

__And since there are no good practices I want to include a few examples of
good practices__, particularly the ones that inspired me to write this post.

In regards to the beauty of the code itself, without regards to its workings,
it should be easy to read and write. In most cases it is more important to be
easy to read that to write. Also most popular programming languages are
inherently hard to write because 1) you need special skills and knowledge 2)
and even when you acquire these they are not keyboard friendly and you have to
press shift all the time and write tons of parenthesis that displace your
fingers from the letters and damage your flow. Unfortunately for the same
reasons they are hard to read too, but after getting used to it we can make
some effort for readability. I like my code to be spacious. Just like people
like to have space in their house, since I live in my code I like to have
space in it. I like it to be structured and consistent. Consistency also helps
for navigation because I do a lot (most) of navigation via searching and I'm
not using fancy IDEs.

To be readable the intention of the code should be clear. For example in the
JavaScript guide I was reading there was example of how to convert variables
to boolean, aka truthfulness. They claimed it is a good practice to use double
negation for this purpose. I argued this is bad practice. There are many
reasons. First double negation !! may be confusing, you need to think about
what the result will be. It depends on the context, particularly because JS is
loosely typed language and in a buggy or poorly designed scenario you can't be
sure of the type of the variable and the same expression can lead to different
results. Consider this example:

```js
var age = 0;
var hasAge = !!age;
if ( hasAge ) {
    // do something with age...
}
```

Here because "hasAge" is somehow descriptive we can assume the intention of
this expression is to determine if something has a valid age. But what if the
example looks likes this:

```js
var age = 0;
if ( !!age ) {
    // do something with age...
}
```

And what if we are not using even the name age, but some dummy name like
"m_pC"?

```js
if ( !!m_pC ) {
    // do something with m_pC...
}
```

While this naming may not be recommended it can happen. What does this code
do? Can you tell without thinking about it? I can't. I know what !age does but
!!age gets me confused. But even with a single negation to understand what it
does you need to know the language well because age can be of any type and
value. For example if age is empty string !age will be true, but if it is
non-empty string it will be false. I routinely know this but I don't know what
is !!age. Not only I have to think about each possible scenario for !age, but
then I need to negate this in my mind. And it gets harder if there are bugs
and you are working on someone else's code. But we can rewrite this so it is
more clear. For example lets say that the intended behavior, i.e. our
definition of valid age is age that is greater than zero.

```js
// not clear intention, "not age" means nothing
if ( !age ) {
}

// not clear intention, "not length" means nothing
if ( !string.length ) {
}

// not clear intention; are we interested to check
// if the value is empty string, or if it is zero,
// or if it is defined or something else?
if ( !object.property ) {
}

// clear intention
if ( age > 0 ) {
}

// clear intention
if ( string.length > 0 ) {
}

// clear intention
if ( object.property !== undefined ) {
}
```

Now you can see the intention from the code and you don't have to guess. More
over even in a buggy scenario, except in some case with strange/non-strict
language behavior, the logic will be preserved or at least part of it. Even if
it happens at runtime that age is not integer at all, but maybe an array or
object or something else, the condition will evaluate correctly because the
variable will not be greater than zero. And further in the above example where
I say "not age" means nothing. Yes it means something if we assume that age is
integer and we know how JavaScript applies logical "not" to integers and
converts them to truthfulness. But this is not clean nor expressive nor well
defined logic. This is something specific to how JavaScript treats numbers.
Numbers and any other type have nothing to do with truthfulness. It is
convenient to be able to do such conversation in the language, but in terms of
logic this is not right. No particular number is any more true than any other
number. It is just a value. Conditions can be true or false, not values
themselves. For example if we impose the condition of age being greater than
zero. This can be true or false. Age itself is always age. It is age by
definition so any age is true in being an age. Is an apple true or false?
Apple being red can be true or false.

An example of bad practice is to sacrifice good practices for wrong reasons.
For example it may be necessary to make the code uglier, repeating code,
writing longer code and non-expressive code to conform to the machines better
and achieve better performance. While this sacrifice will not be justified in
long term because what is slow today is fast tomorrow, nevertheless we write
the software for today so this is a good reason. But we may write uglier code
to conform to older software. A very common example is web development and
Internet Explorer. Sometimes developers are forced to conform to older browser
versions but in most cases they shouldn't do it, because users are given the
opportunity to upgrade their browser, for free, with better version, without
additional requirements, and usually automatically and without effort. If
these users are not willing to upgrade this means they want to stick with
older technology and they should be left to use the older technology until
they develop the desire for the newer one and upgrade their browser. Also the
makers of IE should provide fixes and improvements for their older software
and if they disagree to move forward they should be left behind until their
users start to see that their software provider is not providing the best for
them anymore. Otherwise the developers are sacrificing the beauty of their
work and make unpleasant effort for it. So they make unpleasant efforts to
produce uglier results and provide inferior experience to some users in order
that the latter can not change, although this change is beneficial for everyone.
This makes no sense. In this way the developers damage themselves directly and
damage the world indirectly by slowing down progress.


Authors
-------
Borislav Peev (borislav.asdf at gmail dot com)