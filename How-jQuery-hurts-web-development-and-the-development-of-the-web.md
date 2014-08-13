How jQuery hurts web development and the development of the web
===============================================================

### My love story with jQuery

Until early 2013 I, like many people at this time, used to be a fan of jQuery.
I've used it heavily in the implementation of at least one of my clients' web
sites, which I built from scratch and everywhere else where I had to deal with
the DOM. I enjoyed using quality third party plugins for jQuery like jQuery UI
and I've had created several jQuery plugins which I published as open source.
In the early days of jQuery I was hesitant to use it. I had trouble grasping
its, at the time, innovative approach and I failed to understand it. I was
born and raised with native development (C, C++, D) and at the dawn of
JavaScript frameworks I appreciated MooTools over jQuery which adopted more
classical class-like API. Over time as I gradually moved more and more from
native development to web development and fully submersed myself in it, I came
to appreciate jQuery and fell in love with it. It had so many benefits - it
was very easy to use, had huge community and a lot of plugins, rich feature
set and most importantly it was solving all cross browser issues. In these
days cross browser issues were a big problem - one had to support early IE
versions and deal with all kinds of browser differences. Not the least CSS was
still maturing and even CSS 2.1 was not well, or at least equally, supported
everywhere. And jQuery was helping the world deal with these problems.

Until early 2013 when I started working in a new company. They were building a
social media site and doing lot of jQuery development. In my standards at the
time their code was rather poor and I had to face some difficulties managing
it. Struggling in this way <em>I suddenly came to the realization that jQuery
was guilty for lot of the difficulties I'm experiencing with the poor code
base and is also to blame for the slow development of some technologies</em>.
I'm not saying this was intended by the jQuery developers, but it nevertheless
caused these problems over time.

### And it was over

I soon quit this company (well not because of jQuery but it surely played a
role in my decision). I dropped all my jQuery development immediately and
deleted all jQuery plugins from my GitHub account - I didn't want to support
this development anymore.

### It hurts web development

It encourages lazy coding and in turn poor programming practices. It offers a
lot of convenience by providing an error-free layer and it successfully hides
the internals of browsers' JavaScript. But using this layer comes at a price.
It has a big performance cost and because jQuery code is rather expressive,
one is tempted to abuse it. In the code I was dealing with, it was very common
to see a jQuery selector like `$('.some > .thing.else')` used multiple times
over few lines of code and even in the same statement, for example multiple
times in the same if statement. The proper thing to do would be to call the
selector only once and store it in a variable and reuse this variable. But
start doing it right and it quickly looses its expressiveness and becomes
cumbersome. And as hard as I try to write clean and performant code,
expressiveness is important too and I was tempted myself to employ similar
practices at times.

And even worse. jQuery is not a framework, it is a glue, a collection of
helper functions that solve cross browser issues and simplify some tasks. Yet
because everything is so easy with it people are abusing and use it as a
framework. Normally one evolves some kind of structure, own helper functions
or own mini-framework, based on the low level APIs that the platform provides,
such as vanilla JS. But with jQ this is not the case. Everything is based on
jQuery and custom improvements come as jQuery extensions. You see event
handlers everywhere, disconnected from one another. The code becomes
completely entangled. It becomes almost impossible to trace where each event
is registered. Similar to this different jQ plugins are used everywhere, DOM
elements are created everywhere and in short everything becomes a flat mess of
selectors and functions that steam from the jQuery object.

Performance suffers. Nowadays one does not need to support ancient browsers
anymore. Most importantly Document.querySelectorAll() is supported everywhere,
even in very old browsers like IE8, and many other APIs too. But the jQ
function does not do one thing. It does million things, and even when one
needs to do the most trivial thing like getting a reference to a DOM element,
the jQ function does not merely return this reference, but it constructs a new
object and copy a lot of stuff in it. Event handlers as everything else are
wrapped in jQuery and this eats the CPU cycle by cycle. Not to mention the
jQuery code itself can often suffer from performance issues or employ
less-than-the-best practices. I used to keep a whole article on my site about
one of these practices employed by jQuery for long time, which could easily be
replaced by native language constructs which are faster and more expressive.

And last but not least code structure becomes a mess. Selectors are hard coded
everywhere, HTML code is rendered to DOM with the $ function and so HTML is
hard coded everywhere. Animations are performed using jQuery and presentation
related properties are manipulated using jQuery. The logic and presentation
become mixed everywhere and the structure suffers. Everything becomes an ugly
picture.

### It hurts development of the web

When jQuery appeared it did tremendous job to evolve the web forward. It
taught innovative and expressive techniques to system programmers like myself.
It contributed to better understanding of JavaScript in the world overall and
made the web technologies accessible to wider audience by simplifying the
otherwise not very friendly HTML APIs, and of course by having all the other
merits I list above. Importantly it greatly contributed to the development of
beautiful, interactive and simple (as in simple to use) sites and apps, in
other words to the development of Web 2.0. Now this Web 2.0 is the current
stable version so to speak, but back in the days it was not the case and it
was not so well understood. Web sites were heavy and ugly and although the
technology was there to change it, it seems people still lacked vision how to
put the tech to best use. And I think jQuery did amazing job in teaching
people how to overcome this and achieve great results with the existing
technology.

But now things are changed. It is well understood among people of today how to
make good web sites and web apps. It is time to drop the training wheels, that
is jQuery and similar libraries. They've done their job and it is time to move
forward. It is time to learn and perfect new concepts like separation of code
and presentation, code structure, components, etc. Also it is time many
programmers learn that they are actually doing programming and how it is done
today. I've seen questions in StackOverflow similar to "How to iterate over
elements with jQuery?". Guess what - you don't need jQuery for this, there is
a language construct for looping over things. But jQuery is so good at making
things expressive and simple, some people don't even understand the basics of
the language they are using and don't realize they are not writing in jQuery,
they are writing in JavaScript. And this, in my opinion, hurts the development
of technology. Now that people understand what they need to achieve, i.e. good
sites and apps, they can start to learn the tools to achieve it - in this
context the HTMl5 APIs. Browsers of today are very similar in their
capabilities and while some are still better than others it is relatively easy
to achieve cross-browser compatibility with very little effort. Even thou
browser APIs are not yet perfect and jQuery can still hide some problems,
these problems should not remain hidden anymore. In the earlier web days if
you wanted to have a cross-browser Web 2.0 experience you would face so many
browser related problems that you had to spend all your time dealing with them
and you would forget your goals. But these times are gone. While some of the
old problems are still here, they are generally very easy to overcome. And
what is more important - the way to clear these problems is by realizing them.
The more people realize them the quicker browser vendors and standard bodies
will solve them. But by sticking to jQuery these problems can not surface, or
at least they surface slower and get solved even slower because the overall
awareness of the problems is very low. This prevents the web technologies of
evolving at the speed they could evolve. There are areas which are far from
perfect and anyone who tried to use technologies like animations, components
or tried to achieve better code structure and separation of logic and
presentation is painfully aware of the shortcomings in the current state of
the web technologies. Yet there is not enough critical mass so these matters
can evolve.

These realizations inspired me to start a new frontend framework called
[docviewjs](https://github.com/Perennials/docviewjs), having the major feature
of being written in pure JavaScript. It successfully powered two projects -
[Jsdocgen](https://github.com/Perennials/jsdocgen) and later
[Phptestr](https://github.com/Perennials/phptestr) and proved that it is
perfectly possible to have a nice, modern, fast, web app experience, using
only vanilla JavaScript and with almost no effort to make this experience
cross-browser. By doing this I further improved my skills, learned a great
deal how to use the browser APIs directly and learned what are some of the
actual problems of web development today - and compatibility issues are not
among these problems.

### And I'm glad I'm not alone in this

Major browser vendors like Google and Mozilla produce great efforts like the
Polymer Project and the X-Tags library. People and companies here and there
produce great libraries with pure JavaScript and few days ago I found this
jokingly accurate site about the [Vanilla-JS](http://vanilla-js.com)
"framework".

I hope this article help more people realize it is time to ditch jQuery,
Bootstrap and similar once we've learned what we could from it and in this way
the understating and development of open web technologies will flourish.


Authors
-------
Borislav Peev (borislav.asdf at gmail dot com)