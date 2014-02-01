Node.js introduction for synchronous guys
=========================================

Node.js is a library and runtime for JavaScript. It works on top of the V8
engine. Its primary highlight is asynchronous io.

* [What's the big deal about nodejs?](#whats-the-big-deal-about-nodejs)
  * [Nodejs against PHP/Apache](#nodejs-against-phpapache)
  * [Asynchronous workflow](#asynchronous-workflow)
  * [Promises](#promises)
  * [Error handling](#error-handling)
* [Other nodejs specifics](#other-nodejs-specifics)
  * [Modules / npm](#modules--npm)
  * [On next tick](#on-next-tick)
  * [Signal handling / exiting](#signal-handling--exiting)


What's the big deal about nodejs?
---------------------------------

### Nodejs against PHP/Apache
Nodejs works differently from PHP. With PHP often Apache (the web server) runs
all the time and starts PHP whenever there is request. Nodejs serves itself.
I.e. the script itself acts as HTTP server so the same script does the job of
Apache and of PHP, in the same process. If the script stops we don't have a web
server anymore. So one needs to think about nodejs in web applications a bit
differently. In production we need to do with it how we do with Apache - start
it on machine boot time. And after we update the script we need to restart the
process, because the original script is already loaded in memory and if we only
replace the script it will not affect anything. This may look unfamiliar at first
but actually this is the right way, because Apache is just a bloated file server
and HTTP has nothing to do with serving files.

### Asynchronous workflow
Nodejs achieves good performance by accessing OS resource (files, sockets)
asynchronously, this means you can start opening another file _while_ opening
the first file. So node makes heavy use of callbacks and the program
flow/logic maybe quite different in some cases. Node also provides synchronous
version of most functions, but it is still important to understand how the
async logic works. Here is the basic idea.

```js
// sync logic

var Fs = require( 'fs' );

var fileDescriptor1 = Fs.openSync( 'file1.txt', 'r' );
// at this point we are guaranteed that file1 is opened
// now we can use the file descriptor (in absence of errors)

var fileDescriptor2 = Fs.openSync( 'file2.txt', 'r' );
// at this point we are guaranteed that file2 is opened
// now we can use the file descriptor (in absence of errors)
```

When we use synchronous call, our `open()` function will not return before it
either open the file or fail to open it. When we use asynchronous calls the
function will return immediately and the file will be opened in the background
and we will receive notification when it is open or if there were some error.

```js
// async logic

var Fs = require( 'fs' );

Fs.open( 'file1.txt', 'r' );
// at this point we are *not* guaranteed that file1 is opened
// and at this point we don't know if there is error or not

Fs.open( 'file2.txt', 'r' );
// at this point we are *not* guaranteed that either file1 or file2 is opened
// and at this point we don't know if there is error or not

// so we need to use callbacks to be notified when an IO operation finishes
fs.open( 'file1.txt', 'r', function ( error, fileDescriptor ) {
	if ( error ) {
		// do some error handling if we need
		return;
	}

	// now we can use the file descriptor
} );
```

This is seemingly easy but it can lead to many complications. For example if we
are working with a file and we want to delete it once we are finished, normally
we would close the file and delete it on the next line.

```js
// sync logic

Fs.closeSync( fileDescriptor1 );
Fs.unlinkSync( 'file1.txt' );
```

```js
// async logic

Fs.close( fileDescriptor1, function ( error ) {
	if ( !error ) {
		// only at this point the file is closed and we can delete it
		
		Fs.unlink( 'file1.txt', function ( error ) {
			if ( !error ) {
				// only at this point the file is deleted
			}
		} );
		// at this point the file may still exists,
		// not only that but our program can not terminate
		// only after the unlink() callback is finished, the program can close
	}
} );
```

It may be obvious at this point that this way we can reach quite a deep nesting
level. The code will become unreadable and further complication may arise. For
example lets assume we have many open files and we want to perform another
operation only after _all_ the files are closed. But first why do we want to use
async calls in this case? Because of performance. If we use sync calls, the time
it takes to close all files will be the sum of the time to close each individual
file. If we use async mode, the time it takes to close all files will be the
time it takes to close the slowest it takes to close one individual file.

```js
// async logic

// count how many files we need to close so we know when the last one is closed
var filesLeftToClose = files.length;

// we declare a function that will be notified for each closed file
function _oneFileClosed ( error ) {
	if ( error ) {
		// do something about the error
		return;
	}
	if ( --filesLeftToClose == 0 ) {
		// at this point all files are closed so we can do something
	}
}

// here we have a catch, read after the example
for ( var i = 0; i < filesLeftToClose; ++i ) {
	
	// we use reference to the callback here
	// if we placed the code of the callback withing this loop,
	// we would create a new function instance on each iteration -
	// it wouldn't affect the logic (in this case) but is wasteful and dirty
	Fs.close( file[i], _oneFileClosed );

}

// at this point we are not guaranteed that even a single file is closed
// but we are (almost) guaranteed to get at this point fast!
```

But sometimes it gets really tricky. Particularly because nodejs is not so well
documented. In the last example there is catch that could be very hard to
detect, even harder without detailed documentation. Can you find it?

Here is the catch - we have a loop that uses `filesLeftToClose` for counting.
Inside this loop we call some code and we don't know when this code executes. It
may execute before the loop is finished, so it will modify our counter and our
loop will end prematurely. While working with nodejs many similar things can
happen that can drive you crazy for hours until you find them. The documentation
of the `close()` function only says "Asynchronous close(2)." and it assumes
that this means something to you, or that you need to know what man pages are
and you want to read about the standard C library. Even if these assumptions were
right, which they are not, there is no telling what "asynchronous close" means,
since `close()` is synchronous, even less what happens on other systems like Win32.

Well it happens that nodejs's behavior in regards to callbacks is consistent,
and callbacks are called "on next tick" (read more below), so this edge case
will not actually occur, but this is something that is not documented and you
can learn by experience and by looking at the sources, and the latter is not a
something that anyone in their right mind should expect from their users. I
wanted to point nevertheless that async logic can be very tricky.

### Promises
Promises are method of event handling, where the event listener is guaranteed to
be notified even if it is registered after the event has fired. Nodejs is not
directly concerned with promises - it does not support promises out of the box.
But it is rather important concept for asynchronous programming and recently
I spent many hours debugging a subtle problem that was hard to reproduce. It turned
out to be nodejs internal bug with the same problem that promises solve.

__This is a fictitious example, because the problem it illustrates is worked around in the W3C specs.__  
__Not to be confused that this problem is present with the WebSocket implementations.__

```js
var socket = new WebSocket( 'ws://server.com/socketserver' );
socket.onopen = function () { /* do something on connection */ };
```

Since the `WebSocket` constructor on the first line starts connecting
asynchronously and returns immediately, there is no telling on the second line
if the socket is connected or not. So at the time we have the socket object
initialized and ready for use on the second line, it may be already connected.
This means that we won't receive notification in our `onopen` callback because
it is registered after the event has finished. This sort of problems may be
hard to detect and reproduce because they depend on timing - on the speed
of your computer, how busy it is at the time, how fast your Internet connection
is, if the remote server is busy or not, etc. So it often happens that something
seemingly run but later problems start to popup inconsistently.

If the object returned by the constructor was a "promise", it would keep track
that the open event has occurred and once the event listener is registered it
would fire immediately and we can have a nice logic flow which is guaranteed to
proceed as we expect.

### Error handling
Error handling in asynchronous environment may be more challenging than usual,
because the code is not run where it is written, for example:

```js
try {
	// setTimeout is called, but it won't execute the callback immediately
	// the program flow will exit the try block and at the time of the callback
	// there will be no try block because it will run inside some internal loop
	setTimeout( function () {
		// this will result in uncaught exception
		throw new Error( 'ERROR' );
	}, 1000 );
}
catch ( e ) {
	// this will not trigger
}

// node's way of handling uncaught exceptions
process.on( 'uncaughtException', function ( error ) {
  // our exception will get here, but obviously it can come from any place
} );
```

This again can lead to hard to detect bug, particularly with native extensions,
where we don't have insight of the code. It can happen that such extension
accepts a callback defined in JavaScript, and when the native code calls the
function and some error occurs in the function, if the native code doesn't take
great care in properly handling all errors and re-throwing them, the error may
get silently discarded and debugging may become hell.


Other nodejs specifics
----------------------

### Modules / npm
Another highlight of nodejs are modules. Normally, in a browser environment, all
scripts that one loads are loaded in the global scope so everything they define
becomes a global. Nodejs changes this by introducing modules. Each script that
you load in nodejs has it own scope, so even if you declare variables at file
level they are local to the module and the module need to export variables and
methods explicitly.

Deeply integrated in the nodejs ecosystem is npm - node package manager. npm is
an application that allows you to connect to a central repository and will
simple command like `npm install websocket` gives access to thousands and
thousands packages uploaded by the users. It also can be used so can upload his
own package (and other things). All packages are listed on npmjs.org and they
are usually hosted on GitHub. In this way although node's built-in capabilities
contain only the essentials, in the end node has more libraries than, for
example, PHP. Additionally node is relatively easy to extend with native C++
code and this opens a lot of possibilities.

### On next tick
`process.nextTick()` is a function we can use when we need to schedule a function
to be called outside of the current context, but without any delay. It is a substitute
for `setTimeout( cb, 0 )` that is (supposed to be) more efficient.

### Signal handling / exiting
One often needs to apply some logic when the program terminates. This can be
tricky with nodejs because there are several possible events which will result
in program termination, or is often desirable to result in program termination,
but node handles them differently (and in some cases they can not be handled).
The following OS signals will normally result in termination: `SIGINT`,
`SIGHUP`, `SIGTERM`. Additionally there are uncaught exceptions. Additionally
there is `SIGKILL` which we can do nothing about (for more info on signals refer
to nodejs' docs). Node's default behaviour is to exit on this events, e.g. when
we press `Ctrl+C`, so we want to apply a clean up logic when `SIGINT` occurs,
but if we handle the event, this will replace node's default behaviour and our
program will not terminate. The following example takes some extra care in
handling cleanup logic on exit.

```js
function cleanUpAndExit ( code ) {
	// do some async cleanup
	setTimeout( function () {
		// after all cleanup is ready we can force exit()
		// if we don't, pressing Ctrl+C will not terminate the program
		process.exit( code );
	}, 1000 );
}

function _cleanUp_NoError () {
	cleanUpAndExit( 0 );
}

process.on( 'uncaughtException', function ( e ) {
	console.error( e.stack.toString() );
	cleanUpAndExit( 1 );
} );

process.on( 'SIGINT', _cleanUp_NoError );
process.on( 'SIGHUP', _cleanUp_NoError );
process.on( 'SIGTERM', _cleanUp_NoError );
```

We do not handle the `exit` event and we do not call `process.exit()` anywhere
in our code. Instead we call `cleanUpAndExit()` because if we do otherwise, the
`exit` event will fire after the main event loop is finished and we couldn't use
any asynchronous code in our cleanup procedure. And we most likely will need to
use such code because we will probably need to wait for files and sockets to
flush and close, wait for all writes to finish, close any servers, etc.


Authors
-------
Borislav Peev (borislav.asdf at gmail dot com)