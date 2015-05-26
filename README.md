# AnnotateThis
Simple, vanilla JS type checking through ES7 decorators

# Type Validation
```js
import {param, returns} from 'AnnotateThis';

class Point {
    constructor() {
        this.x = 0;
        this.y = 0;
    }
    
    // The first param of this method takes a Point, and will throw
    // a type error if a non-Point is passed
    @param(Point)
    // The method returns a Number. If it doesn't, a type error will
    // be thrown before the value is returned.
    @returns(Number)
    distanceTo(point) {
        let squaredDistance = (point.x - this.x) ** 2 + 
            (point.y - this.y) ** 2;
        
        return Math.sqrt(squaredDistance);
    }
}
```
Automatically memoize functions for greater efficiency
# Memoization
```js
import memoize from 'AnnotateThis';
let obj = {
    // Results of the function are stored in a map, which maps arguments
    // to the function's result. This expensive func is only run a single
    // time for a given a/b pair.
    @memoized
    expensiveFunc(a, b) {
        return Math.sin(Math.sqrt(a ** b));
    }
};
```

# Property descriptors
```js
import {enumerable, writable, configurable} from 'AnnotateThis';

class T {
    @configurable(false)
    @enumerable(false)
    @writable(false)
    hiddenMethod() {
        /* ... */
    }
}
```

# Type Validator API
Types can be expressed in a few ways:
 - A native constructor, like `Object` or `Boolean`
 - A constuctor or class, like `Point`
 - An object literal, for duck-typing, like `{name: String, age: Number}`
 - A special helper function from `types`, like `ArrayOf(String)`
 
## Native Constructor Types
The most common types to check for. All other type checks are built around 
these.

```js
@param(Number)
@param(String)
@param(Boolean)
@param(Object)
@param(Array)
@param(Function)
```

Or any function which passes this check for a given set of values:
```js
let passes = Fn(x) === x
```

## Constructor or Class Types
Use your own class constructors to serve as `instanceof` validators:
```js
class T {
    constructor() {

    }
    
    @param(T)
    doThing(t) {

    }
}
```

## Object-literal Duck-types
Object literals are used to do 'duck-type' checks. All keys in a param must
exist in the duck type, and must be of the correct type, for the check to
pass.

```js
class T {
    @param({hello: String, info: {age: Number, color: String}});
    method(options) {
        this.hello = options.hello;
    }
}

// throws error, color is missing
(new T).method({hello: 'hi', info: {age: 5}})

// throws error, info.age is not a number
(new T).method({hello: 'hi', info: {age: '5', color: 'red'}}) 
```

As with all types, these are composable.

## Built-in Types
AnnotateThis provides a small library of helper functions for doing type
validation.

```js 
import {
    AnyOf,
    ObjectOf,
    ArrayOf,
    Optional,
    Any
} from 'AnnotateThis/types';

let util = {
    @param(ArrayOf(Any))
    @param(Optional(Boolean))
    @returns(ArrayOf(Any))
    sortArray(array, reverse=false) {
        /* ... */
    }
}
```

### AnyOf
When a single value is allowed to be one of many types, use `AnyOf`.
```js
@param(AnyOf(Number, String))

let pass = 5;
let pass2 = '5';
let fail = true;
```

### ObjectOf
When an object contains a specific type of values, use `ObjectOf`. If the object
you want to validate contains many types, use duck-typing instead
```js
@param(ObjectOf(Boolean))

let pass = {abc: true, bcd: false};
let fail = {abc: true, bcd: null}
```

### ArrayOf
Like `Array`, but lets you validate the type of each element
```js
@param(ArrayOf(String))

let pass = [];
let pass2 = ['Hello', 'world'];
let fail = 5;
let fail2 = [5];
```

### Optional
If the param is missing, this check passes. If not, it validates the type.
```js
class T = {
    @param(String)
    @param(Optional(Boolean))
    method (name, capitalize=true) {

    }
}

// OK
(new T).method('jake');
(new T).method('jake', false);

// Not OK
(new T).method('jake', 5);
```

### Any
A special check that always passes, unless a value is missing.
```js
class T {
    @param(Any)
    method(i) {

    }
}

// OK
(new T).method('str');
(new T).method(5);
(new T).method(() => {});

// Not OK
(new T).method();
```