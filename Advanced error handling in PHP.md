Advanced error handling in PHP
==============================

Once I had to write some PHP code and I wanted to ensure the code will execute
as "transaction", i.e. it will either succeed or fail and if it fails I will
be able to do clean up and not leave the system in messy state. But it turns
out this is not easy and actually almost impossible with PHP. It is possible
but to have really reliable solution it becomes so complex and with such a
high performance cost that it becomes unusable.

### Error handling with PHP has some issues

it is __inconsistent__ - there are several methods of handling errors and PHP
makes distinction between error handling and exception handling. This means
some APIs produce "errors" and some APIs produce "exceptions" and you need
special code to handle each of them.

It is __unreliable__. Apparently, from PHP's view point some errors are so
bad, "fatal errors", that they shouldn't go though the standard error handling
mechanism and they should crash the script without a chance to do something
about it. But they are not that bad that they shouldn't trigger the shutdown
callbacks and they still get recorded somewhere and can be queried via
`error_get_last()`. But it gets even worse. If a piece of code includes
another script which has syntax error in it, this syntax error will crash PHP
unconditionally - no error handling will be triggered, no exception will be
raised and the script will die without calling any shutdown callback.

The case with exceptions is not so bad. They behave as expected, the only
problem with them is that most of PHP's APIs are not using them. Not only
that, but errors in the code like calling undefined functions, division by
zero, etc. will not raise an exception and can not be handled with a
`try/catch` block. As of PHP 5.5 there is support for `finally` blocks but
they are almost useless because PHP errors will not trigger the `finally`
code. I wonder what's so "finally" about it in this case?

So all of this quickly becomes __very inconvenient__. Normally in PHP if we
want to handle all of these cases we need a `try/catch` block, an error
handler callback and a shutdown callback, we need to clean these up after we
finish and we need to mind that we don't break the behavior of existing error
handlers and shutdown callbacks. And writing these handlers in PHP looks awful
and is terribly inconvenient to repeat. And if we want to nest error handlers
the mess becomes too much to write even once.

### The first solution

combines all approaches outlined above in a reusable function that can be
nested. It is not 100% reliable but it will work in most cases at a relatively
small performance cost. It will handle all types of errors besides syntax
errors but one needs to be careful when using it because it is not
unbreakable. For example it will catch a fatal error, but will not be able to
catch a second fatal error - one inside the the function that is handling the
first one. The code bellow is PHP 5.3, it will work with PHP 5.2 but may need
some tweaks and you can't use closures there which destroys the point.

In short all we need to do is this - a few lines of code.

```php
_try(
    // some piece of code that will be our try block
    function () {
    },

    // some (optional) piece of code that will be our catch block
    function ( $exception ) {
    },

    // some (optional) piece of code that will be our finally block
    function () {
    }
);
```

Lets see how to really use it. The code is available as part of a library that
I'm working on and is open sourced [on GitHub](https://github.com/Perennials/travelsdk-core-php).
This library is very much work in progress but the code in question is
documented and has many test cases. At the time of writing I recommend
downloading the most up-to-date
[branch](https://github.com/Perennials/travelsdk-core-php). You will need two
files (see below) which are found in the `src/sys` directory. There is
additional documentation for the `_try()` function itself, it can be found in
the `docs` directory and once the docs are opened in the browser
(`docs/index.html`) look inside the `travelsdk.sys` package, or alternatively
it can be found in `TryCatch.php` in markdown format.

```php
// you can merge these two if you don't want to include two files
require_once 'ShutdownCallback.php';
require_once 'TryCatch.php';

// we are displaying text in our example
header( 'Content-Type: text/plain' );

// disable php's error display
ini_set( 'display_errors', 'Off' );

// just some custom exception
class MyException extends Exception {
}

// catching custom exceptions
// the catch and finally blocks are optional, we can use _try just to silince the error
$ret = _try(

    // try code
    function () {
        echo 'trying some code that throws', "\n";
        throw new MyException( 'My very own exception' );
    },

    // catch code. will catch all exceptions and php errors, except for syntax errors
    function ( $exception ) {
        echo 'caught error: ', $exception->getMessage(), "\n";
        if ( $exception instanceof MyException ) {
            // if we want to handle specific type of exception
            // we need to check the class of the exception
        }
        else {
            // we can re-throw the exception if we want
            // our finally block will execute before
            // the exception leaves our _try()
            throw $exception;
        }
    },

    // finally code
    function () {
        //do clean up here
        echo 'finally 1...', "\n";
        return 'returnvalue';
    }
);

echo 'continuing script execution after the exception...', "\n";
echo 'we can use the return value of the last executed block - in this case the finally block - ', $ret, "\n";

// catching non fatal errors
_try(

    //try code
    function () {
        // something that may produce an error
        // but shouldn't crash the script
        echo 'trying division by zero', "\n";
        $a = 5 / 0;
    },

    // catch code. will catch all exceptions and php errors, except for syntax errors
    function ( $exception ) {
        echo 'caught error: ', $exception->getMessage(), "\n";
        if ( $exception instanceof ErrorException ) {
            // a PHP error
            // if we encounter one our script
            // will terminate after the finally block
            echo 'oops, a PHP error, check your code. the script will terminate after the finally block', "\n";

            //we can even nest _try blocks. lets cause a fatal error on purpose
            echo 'causing a fatal error...', "\n";
            _try( function () {
                $a = new stdClass();
                $a->method();
            }, function ( $exception ) {
                if ( $exception instanceof ErrorException && $exception->getSeverity() == E_ERROR ) {
                    echo 'great, we are catching fatal errors', "\n";
                }
            }, function () {
                echo 'finally 2...', "\n";
            } );
        }
    },

    // finally code
    function () {
        // do clean up here
        echo 'finally 3...', "\n";
    }
);

echo 'this code won\'t execute because of the php errors, but the finally blocks will execute';
```

The above example will output this.
```
trying some code that throws
caught error: My very own exception
finally 1...
continuing script exectuion after the exception...
we can use the return value of the last executed block - in this case the finally block - returnvalue
trying division by zero
caught error: Division by zero
oops, a PHP error, check your code. the script will terminate after the finally block
causing a fatal error...
great, we are catching fatal errors
finally 2...
finally 3...
```

### The second solution

is really reliable. It can detect any kind of error and take actions if
needed, but is extremely complicated and slow. The approach is to start the
code that needs to be "unbreakable" in a separate process, i.e. sandbox, so it
will not be able to crash the main process. This requires a lot of extra work
and the performance cost is huge. Obviously it is not applicable as
`try/catch` replacement, but nevertheless has its uses. This is the approach I
use in __phptestr__, my unitesting framework for PHP and I will dedicate a
separate blog post on phptestr where I will elaborate how I am able to
guarantee that all errors will be traced and how I am able to write unitests
for complicated scenarios like the `_try()` function itself.


Authors
-------
Borislav Peev (borislav.asdf at gmail dot com)