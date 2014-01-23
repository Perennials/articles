Object oriented programming with JS
===================================

While in JavaScript everything is an object, there is no concept of classes, so
the classical understanding of OOP may be hard to apply. In JS each object acts
as a bag of properties and each object has a prototype. Prototype is just
another bag of properties. If you try to access a property that is not found in
the object, JS looks for the property in the prototype. Each prototype can have
another prototype so if the property is not found in prototype it is look for in
the chain of prototypes. The prototype is is not unlike the concept of classes.
Chaining prototypes in this way is just like inheriting classes. The only object
that acts somehow differently is the function. It is also a bag of properties,
but it is also a function which can be executed. The odd thing in JavaScript is
how you define prototypes (or types, or classes if you wish). It is done with
functions. For example if I want to define a type called `Earthling`:

```js
function Earthling () {
	this.type = 'earthling';
}

// now we have a type called Earthling

// we can instantiate this type with the new operator
var earthling = new Earthling();

// this condition will pass
if ( earthling instanceof Earthling ) {
	// ...
}
```

We can still call the function as usual, without `new`. The difference is that
if we call it with new, it will be assigned `this` object that is a brand new
object and this object will have the prototype `Earthling`. Additionally we
don't need to return any value from the function. We could, but if we leave it
like this, the `Earthling` object is returned. If we just call the function
normally, `this` will refer to the global object and no new object will be
constructed. So in the example above calling this function without the `new`
will create a global variable called `type`. I like to think of this function
as a class constructor. There is additional way to declare a constructor, but
it includes some unnecessary code and I don't like it so I won't include it here.

At this point we have a new prototype (or type or class). But how to declare
methods in this class? The function the we have declared has a property called
`prototype`. This property is shared among all instances of the type. Whatever
we put in this prototype will be accessible to all instances.

We can illustrate it simply like this:
```js
Earthling.prototype = {
	getType: function () {
		return this.type;
	}
};
```

While this works, it is a bit dirty, because all properties of the object are
enumerable and will show in our for-in loops and other places we probably don't
want to. We can use `Object.defineProperty()` instead. This function is able to
set properties in a more elaborate way and by default they are not enumerable.
MDN is excellent resource to learn all available options.
```js
Object.defineProperty( Earthling.prototype, 'getType', {
	value: function () {
		return this.type;
	},
	writable: true
} );
```

Lets look into inheritance. Inheritance is done by assigning a prototype.
```js
// declare new type
function Cat () {
	this.type = 'cat';
	this.color = 'ginger';
}

// we use Object.create to create new object
// if we assign the Earthling.prototype directly whatever we define in Cat's
// prototype will go in Earthling's protype and vice versa
Cat.prototype = Object.create( Earthling.prototype );

// now we can define a method just for cats, not for all earthlings
Object.defineProperty( Cat.prototype, 'getColor', {
	value: function () {
		return this.color;
	},
	writable: true
} );

// and we can instantiate a Cat
var cat = new Cat();

// this condition will pass
if ( cat instanceof Cat && cat instanceof Earthling ) {
	// ...
}

// this condition will pass
if ( cat.getType() == 'cat' ) {
	// ...
}
```

And lastly we can have sort of static properties by putting them directly in the
type, i.e. the function representing the type.
```js
Cat.staticVar = 5;

// this will evaluate to undefined
(new Cat()).staticVar;

// this will evaluate to 5
Cat.staticVar;
```

While this covers the most important use, there is more to understand to escape
some pitfalls. Good understanding of objects and references is necessary.
Additionally defining types in this way may be tedious, so I've created
[a small library](https://github.com/Perennials/prototype-js) to aid with the task.
Of course one can come up with many different syntaxes, but I wanted to have
something without any overhead and keeping the native JS feel as much as
possible in order to avoid confusion. Rewriting the examples with Prototype will look like this:
```js
function Earthling () {
	this.type = 'earthling';
}

// just a wrapper for Object.defineProperty()
Earthling.define( {
	getType: function () {
		return this.type;
	}
} );

function Cat () {
	this.type = 'cat';
	this.color = 'ginger';
}

// just a wrapper for Object.create() + Object.defineProperty()
Cat.extend( Earthling, {
	getColor: function () {
		return this.color;
	}
} );

// this is not shorter actually, but just for consistency
Cat.defineStatic( {
	staticVar: 5
} );
```

The library has some more OOP sugar like mixins, but this is out of this scope.


Authors
-------
Borislav Peev (borislav.asdf at gmail dot com)