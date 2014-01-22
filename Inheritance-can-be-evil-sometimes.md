Inheritance can be evil sometimes
=================================

In the past I heard statements like "inheritance is evil!", I had some fights
with other programmers on the topic and I read in the net about it, but nobody
was really able to show me what is wrong with it - in practice - not in theory.
Because as evil it may be in theory, the practical solutions are in my opinion
worse.

Finally I understood from my own experience why inheritance can be evil
sometimes. I will use PHP for example. Not long ago PHP received `trait`s, a
beautiful feature, which can solve most of the inheritance problems in a
practical way.

All problems stem from the desire to have reusable classes. One writes a neat
class with clean logic. Then it happens the he or she needs another class that
is slightly different. So naturally one inherits the class and modifies the
behaviour just a little bit in the derived class to escape having to repeat the
logic...

What is evil about inheritance?
-------------------------------

It can lead to unforeseen complications and subtle, hard-to-detect,
hard-to-foresee logic entanglement. Is not really a problem with inheritance but
rather with overriding virtual functions. Since in dynamic languages like PHP
all methods are virtual it is quite realistic scenario. For example:

```php
class Base {
	
	function a () {
		// ... do something within Base
	}
	
	function b () {
		$this->a();
	}
}

class Child extends Base {

	function a () {
		// ... do something within Child
	}

	function b () {
		parent::b();
	}
}
```

In this scenario the class `Child` extends class `Base`. When method `b()` is
called from instance of `Child`, it transfers the flow the parent's class method
`b()`. The latter tries to call method `a()` of the same class, but since it is
overridden in the derived class, `$this` actually refers to the instance of
`Child` and it will call `Child::a()`. But the logic inside `Base` is written
(or at least it should) without prior knowledge of the derived classes. So
`Base::b()` calls `$this->a()` expecting that the flow will stay in the same
class, but the flow is sent back to the derived class `Child`. This can lead to
many problems. It works sometimes because `Child` knows about `Base` and thus it
can be made in such a way that it works. It is usually not a problem at the time
of writing the class, but later when this are forgotten. And even though at the
time of writing of `Child` the logic of the parent class is known, it may be
hard to predict that the flow will return back and the `a()` method of the same
class will be called. Particularly for larger classes with more logic and edge
cases that are not easy to test, are not well documented and unitests are not
available. And if we have even more derived classes after `Child` it could
become a real nightmare because each deeper derivation must take into account
the chain of all base classes. This makes the logic fragile and inflexible. And
changing something in the base classes can break many things on many levels.
Stretch this in time and across multiple developers and it get very bad.


But if used correctly there is nothing wrong with inheritance
-------------------------------------------------------------

If we use inheritance only to extend classes, not to modify them
by overriding functions, then all is fine. For example:

```php
class Base {
	
	function a () {
		// do something within A
	}

	function b () {
		$this->a();
	}
}

class Child extends Base {
	
	function c () {
		parent::b();
	}
}
```

At the time of writing `Base` we have no knowledge of `Child`, so we couldn't
possibly call any function outside of `Base` and consequently the logic remains
healthy. At the time of writing `Child` it is safe to call the `Base` methods
because at this time the behaviour of these methods is known and it is
guaranteed that it will work as documented and that when we call any of the
parent's methods, the flow will remain within the parent and will produce the
intended result. In other words we can extend as much as we want and be safe,
but not modify (override).


Solutions to the problem
------------------------

One solution is dependency injection. It is a nice pattern if used wisely, in
other words when we actually want to parameterize something, but when using it
as workaround of inheritance for reusing code and we are building a class
library it can quickly become too much. Too keep the nice level of abstraction we
must parameterize everything to extreme and the library becomes unpleasant to
use. Yes, it keeps the logic clean and flexible, but the writeability suffers
greatly. It also comes at additional performance cost because of the increased
number of class instances.

The solution I like is called `trait` in PHP, `mixin` in D and maybe has other
names in other languages. It is a method of reusing code where you can "mix in"
blocks of code inside your classes. This way one can build small reusable
blocks, with strong coherent logic, and combine them freely. This with
combination of `interface`s result in beautifully DRY code, without unnecessary
inheritance and without logic entanglement and potential problems for the future.


Authors
-------
Borislav Peev (borislav.asdf at gmail dot com)