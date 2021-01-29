Accessing a non-existent property of an object returns `undefined`. It does not raise an exception. JavaScript treats `undefined` just like another value. It allows performing comparisons with it. For example, `undefined > 5` does not raise an exception. It simply returns `false`.

The problem with this is, when you mistype/misspell an object property, there is no indication about this in JavaScript. Since a non-existent property simply returns `undefined`, and since JavaScript simply treats `undefined` like a regular value, your program will just break and you won't have any idea about the reason. An example:

```javascript
const a = [1,2,3];
if (a.len !== 3) console.log("a's length is not 3!");
```

The code above will print "a's length is not 3!". Because of a simple mistyping, our program is broken without any indications. We have typed `a.len` instead of `a.length`. Since returning `undefined` when accessing a non-existent property (and not throwing an exception, printing a warning, etc.) is the default behavior in JavaScript, we didn't get any indication/warning about this.

The solution to this is to [use TypeScript][My answer about non-existent property access]. Had we used TypeScript, we would have gotten an error right on the "compile time" (that is, on our editor before we even run the code) saying `Property 'len' does not exist on type 'number[]'` and underlining the erroneous part of the code with red squiggles.

[My answer about non-existent property access]: https://stackoverflow.com/a/63548095/3395831

----

You cannot create an array of arrays using [`Array.prototype.fill()`]. Specifically, you cannot create an array of objects using `Array.prototype.fill()`, because every element in the array will be the _exact same object_. Hence, modifying _any_ element of the array modifies _every_ element of the array, since every element is the _exact same element_ (same reference). Example:

```javascript
const a = Array(5).fill(Array(3).fill(null));
/*
 * a is:
 *
 * [[null, null, null]
 *  [null, null, null]
 *  [null, null, null]
 *  [null, null, null]
 *  [null, null, null]]
 */
a[2][1] = 'A';
/*
 * a is:
 *
 * [[null, "A", null]
 *  [null, "A", null]
 *  [null, "A", null]
 *  [null, "A", null]
 *  [null, "A", null]]
 *
 * instead of
 *
 * [[null, null, null]
 *  [null, null, null]
 *  [null, "A", null]
 *  [null, null, null]
 *  [null, null, null]]
 */
```

Instead, you can use `Array.from(Array(5), () => Array(3).fill(null))` to get the expected behavior.

[`Array.prototype.fill()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/fill

----

You should never modify in-place an object in JavaScript. If you modify an object in-place that's passed into a function, that modification affects the object where the function is called from as well. Example:

```javascript
const a = [1,2,3];
const b = [4,5,6];

function x(p) {
  p.unshift(0);
  // Or, similarly:
  // for (let i = 0; i < p.length; i++) p[i]++;
}

function y(p) {
  [0, ...p];
}

// Shows [1, 2, 3]
a
x(a)
// Shows [0, 1, 2, 3]
a


// Shows [4, 5, 6]
b
y(b)
// Shows [4, 5, 6]
b
```

This is, in fact, the same behavior with Java. Example Java code:

```java
import java.util.Arrays;

public class HelloWorld {

    static void increment(int[] arr) {
        for (int i = 0; i < arr.length; i++) arr[i]++;
    }

    public static void main(String []args) {
        int[] a = {1,2,3};
        // Prints [1, 2, 3]
        System.out.println(Arrays.toString(a));
        HelloWorld.increment(a);
        // Prints [2, 3, 4]
        System.out.println(Arrays.toString(a));
    }
}
```

In short, in most (all?) object-oriented programming languages, if you modify an object in-place, the modifications are reflected in the calling context. The reason is that the objects are actually passed-by-reference. That is, instead of passing a copy of the object to the function, a _copy of the reference_ is passed to the function. The copy of the reference simply points to the same original object. Hence, any modifications within the function actually modify the object.

----

Optional chaining operator (`?.`) should be used AFTER the element that will be potentially `null`/`undefined`, NOT before. Example:

WRONG:

```javascript
const desiredProp = someVariable?.somePotentiallyNonExistentProperty.desiredProp;
```

RIGHT:

```javascript
const desiredProp = someVariable.somePotentiallyNonExistentProperty?.desiredProp;
```

The reason is that optional chaining operator protects against property accesses on non-existing things (objects, properties, etc.) That is, you place the optional chaining operator NOT BEFORE the thing that has the potential to be non-existent, but AFTER it. The right version of the code above means that "we are sure that `someVariable` will never be non-existent, even if it just an object with no properties in it (that is, an empty object). However, there is a possibility that `somePotentiallyNonExistentProperty` prop of `someVariable` (and in turn, everything that descends from `somePotentiallyNonExistentProperty`) has the potential to be non-existent".

Note that in order to go deeper in the tree of an object, you DO NOT have to use the optional chaining repeatedly. Just using once after the property that has the potential to be non-existent is enough. Example:

```javascript
const a = {p1: {p2: {p3: 'v3'}}};
const b = {};
// 'c' is a variable that can potentially be 'a' or 'b'.
let c;
// Following works perfectly. Essentially it means "c might not have the
// 'p1' prop but if it does, then 'p1' prop will, for sure, have the
// 'p2' prop and 'p2' prop will, in turn, for sure have the 'p3' prop".
c = a;
// Works. Prints "v3".
console.log(c.p1?.p2.p3);
c = b;
// Works. Prints "undefined". That is, it does not throw "TypeError:
// Cannot read property ... of ... at ...".
console.log(c.p1?.p2.p3);
```

The reason that optional chaining works this way is because of "short-circuiting". That is, if the expression *before* the optional chaining operator is non-existent (aka "nullish", that is, `undefined` or `null`), the expression immediately returns `undefined` without even continuing to evaluate the rest of the expression (the rest of the expression is everything that's on the right or bottom side of the same expression). This behavior is named "short-circuiting", and it is the reason that when we set `c = b;` in the example above, `c.p1?.p2.p3` simply returns `undefined` instead of causing a `TypeError` to be thrown.

----

On a (web) browser, DON'T USE `console.log` to inspect something. It is broken. When you log something using `console.log` and then expand it using the little arrow on the output of `console.log`, the values shown will be the values of the logged object _at the time that you have expanded the `console.log` output_. They won't be the values at the time that the object was logged.

That is, say that you logged something using `console.log`. Later, the program changed (mutated) that variable. When you go to the place where the log is and click on the little arrow to expand it, the values shown there are not the values at the time of logging. They are the values at the time that you expand it (by clicking on that little arrow near the log). That is, if you expand it after the program finished execution, it means that the values shown will be the final version of the variable, not the version at the time of logging. Hence, `console.log` cannot be used to inspect a variable.

Even though the little blue information box when we expand a console log says "Value below was evaluated just now", the truth is, value shown when we expand a console log is evaluated at "the first time when we expand a console log". If we contract a log and then later expand it again, the value will not be re-evaluated. The value obtained the first time when we expanded a log will be shown. Below is a demonstration of this:

![console.log demonstration](assets/console-log.gif)

Apparently, this has been the behavior the whole time. Here are some Stack Overflow questions from 2010s:

[Can't access object property, even though it shows up in a console log](https://stackoverflow.com/questions/17546953)

[console.log() shows the changed value of a variable before the value actually changes](https://stackoverflow.com/questions/11284663)

[Is Chrome's JavaScript console lazy about evaluating arrays?](https://stackoverflow.com/questions/4057440)

[console.log() async or sync?](https://stackoverflow.com/questions/23392111)

[Bizarre console.log behaviour in Chrome Developer Tools](https://stackoverflow.com/questions/4198912)

[javascript constructor with google chrome bugged?](https://stackoverflow.com/questions/8007397)

Other articles talking about this issue:

[Logging JavaScript Objects](https://code-maven.com/logging-javascript-objects)

[Please stop using console.log(), it’s broken](https://hackernoon.com/please-stop-using-console-log-its-broken-b5d7d396cf15)

A quick and "good enough for most cases" workaround is to serialize and deserialize the object and then log it:

```javascript
function log(object) {
  console.log(JSON.parse(JSON.stringify(object)));
}
```

A more robust solution is to deep clone an object and then simply log the clone. This example is using the [lodash library] to deep clone an object:

```javascript
function log(object) {
  console.log(require('lodash.clonedeep')(object));
}
```

Note that deep cloning still does not work to take a snapshot of object prototype (the `__proto__` property). As far as I can tell, for now, there is no way to take a snapshot of the `__proto__` property:

```html
<!DOCTYPE html>
<html>
<head>
  <script type="module">
    import cloneDeep from 'https://cdn.jsdelivr.net/npm/lodash-es@4.17.15/cloneDeep.js';

    function log(object) {
      console.log(cloneDeep(object));
    }

    function A() {}

    const a = new A();
    log(a);
  </script>
</head>
</html>
```

When we expand the log output of the code above, we observe the following output:

<pre>
A {}
  __proto__:
    constructor: ƒ A()
      arguments: (...)
      caller: (...)
      length: 0
      name: "A"
      <b>prototype: {constructor: ƒ}:</b>
        ...
      ...
    __proto__: Object:
      ...
</pre>

Note how `a.__proto__.constructor.prototype` has a value (that is, it is not null).

Now, we add the line `A.prototype = null;` after the last line, save the HTML file and refresh the web page. When we expand the console output, we observe:

<pre>
A {}
  __proto__:
    constructor: ƒ A()
      arguments: (...)
      caller: (...)
      length: 0
      name: "A"
      <b>prototype: null</b>
      ...
    __proto__: Object:
      ...
</pre>

Again, the same issue with `console.log` without deep cloning. I can tell that this is the same (or very similar) issue with that, because instead of entering the JavaScript code on an HTML file and opening the HTML file on a browser, when we just open a browser console without any HTML file and run:

```javascript
function A() {}

const a = new A();

console.log(a);
```

and then expand the log value, we observe that `a.__proto__.constructor.prototype` is not null. Then, when we collapse the log output, execute the line `A.prototype = null;` and re-expand the log output, we observe that `a.__proto__.constructor.prototype` is still not null.

On the other hand, if we just execute:

```javascript
function A() {}

const a = new A();

console.log(a);

A.prototype = null;
```

and then check out the log output by expanding it, we observe that `a.__proto__.constructor.prototype` is null. Hence, this is again the same issue. That is, the log output shown in expanded view are evaluated _at the time that we expand it_. Not at the time of logging. Here is a GIF that demonstrates this:

![console.log proto demonstration](assets/console-log_proto_demonstration.gif)

On another note, even the [MDN notes] suggest not to use `console.log` directly. They suggest to use at least `JSON.stringify`. The same page also says:

> Please be warned that if you log objects in the latest versions of Chrome and Firefox what you get logged on the console is a reference to the object, which is not necessarily the 'value' of the object at the moment in time you call console.log(), but it is the value of the object at the moment you open the console.

[This is the issue][Chromium issue] that I have created on Chromium bug tracker to change the on hover message on the little blue information box that appears when we expand a `console.log` output from "Value below was evaluated just now" to "Value below was evaluated the first time that you have expanded the log output".

One more note: You might think that using `Object.entries` will be a solution to this, but `Object.entries` is essentially the same thing as making a shallow copy. Example:

```javascript
const arr = {a: [0,1,2,3]};
arr.a[0] = -1;
console.log(Object.entries(arr));
arr.a[1] = -1;
console.log(Object.entries(arr));
arr.a[2] = -1;
console.log(Object.entries(arr));
arr.a[3] = -1;
console.log(Object.entries(arr));
```

The problem persists here as well. When you expand the log outputs, you observe all elements of the array are -1, instead of the values at the time that they were logged.

[lodash library]: https://lodash.com/
[MDN notes]: https://developer.mozilla.org/en-US/docs/Web/API/Console/log#Logging_objects
[Chromium issue]: https://bugs.chromium.org/p/chromium/issues/detail?id=1141675

----

You don't need to wrap an invocation of a "constructor function" within parentheses to access a property on the returned (created / initialized) object. It is redundant. That is, it will work just fine without wrapping it:

```javascript
'use strict';

function a() {
  this.x = 1;
  this.y = 2;
}

a();
// Gives:
// Uncaught TypeError: Cannot set property 'x' of undefined

// CORRECT:
new a().x
// Evaluates to 1. Hence, this shows that we don't need to wrap an
// initialization statement within parentheses in order to access a
// property on the returned (created / initialized) object.

// REDUNDANT / NEEDLESS:
(new a()).x
// Again, evaluates to 1, but it is pointless / redundant.
```

----

You cannot add properties to primitive types
============================================

```javascript
let a = 1;
a.someProp;
// undefined
a.someProp = 5;
// 5
a.someProp;
// undefined

// Another example
let b = 'hello';
b.anotherProp;
// undefined
b.anotherProp = 10;
// 10
b.anotherProp;
// undefined
```

The [primitive types] in JavaScript are:
- string
- number
- boolean
- symbol
- bigint
- undefined

Although `null` seems like a primitive, it is a special case of the Object type, though it behaves like a primitive for most purposes.

Everything else are instances of the Object type or a descendant of the Object type.

[primitive types]: https://developer.mozilla.org/en-US/docs/Glossary/Primitive

----

How does the array index operator (the square bracket operator) work in JavaScript?
===================================================================================
Well, it doesn't work because there is no such thing in JavaScript. That is, there is no special square bracket operator for accessing array indices in JavaScript. But how? Aren't we able to access the arrays using syntax like `someArray[someInteger]`? Well, yes we do, but that is not a special syntax defined to access the indices of an array. It is nothing but simply the good old "bracket notation" for accessing _object properties_.

Bracket notation is an alternative to dot notation to access object properties. In JavaScript, an array is nothing but an object. When we use bracket notation, if there is an identifier (a variable name, etc.) inside the brackets, then the value of the identifier is resolved. If its value is not a string or a symbol, then its value is _[coerced]_ into a string and then JavaScript tries to find a property whose name is equal to that string.

That is, when we write `someObject[5]`, 5 is not an identifier, hence its value not resolved. It is not a string or symbol either, so its value must be coerced into a string. It is a number. Coercing it into a string yields '5'. Hence, JavaScript searches for a property named '5' in `someObject` and if it finds that property, it returns its value. If not, it returns `undefined`.

To prove that this is indeed the case, let's define an array:

```javascript
let a = ['x', 'y', 'z'];
Object.getOwnPropertyDescriptors(a);
// Returns:
// {
//   '0': { value: 'x', writable: true, enumerable: true, configurable: true },
//   '1': { value: 'y', writable: true, enumerable: true, configurable: true },
//   '2': { value: 'z', writable: true, enumerable: true, configurable: true },
//   length: { value: 3, writable: true, enumerable: false, configurable: false }
// }
```

As you can observe, an "array" in JavaScript is actually nothing but an object which has properties named 0, 1, 2, etc. That is, property names are strings which represent integers. These property names "masquerade" as if they are array indices, but they are not. What they really are is that they are simply regular object properties which can be "parsed into" an integer.

Another feature in JavaScript is that it is possible to access a string's characters by index. That is:

```javascript
let a = 'hello';
a[2];
// Returns:
// 'l'
```

This is again not a "special syntax" or something for strings (or "array-like elements" or something, since there is no such thing as "array-like elements" in JavaScript) but it is simply the good old object property access using bracket notation. Examining the properties of a string clarifies things:

```javascript
let a = 'hello';
Object.getOwnPropertyDescriptors(a);
// Returns:
// {
//   '0': { value: 'h', writable: false, enumerable: true, configurable: false },
//   '1': { value: 'e', writable: false, enumerable: true, configurable: false },
//   '2': { value: 'l', writable: false, enumerable: true, configurable: false },
//   '3': { value: 'l', writable: false, enumerable: true, configurable: false },
//   '4': { value: 'o', writable: false, enumerable: true, configurable: false },
//   length: { value: 5, writable: false, enumerable: false, configurable: false }
// }
```

Again, as you can observe, a string has properties named '0', '1', '2', etc. and the value of the each of these properties is the character in the string that is at the respective index. But wait! Isn't a string in JavaScript a primitive? How come it has properties? Well, true, a string in JavaScript is a primitive and hence, it cannot have any properties. However, if a primitive is passed to a place where an object is needed, JavaScript does something called [autoboxing]. That is, it creates the object version (the wrapper) of that primitive.

Let's give one more example from [MDN] to the type coercion on object properties. As we said, object properties are either strings or symbols. Hence, any other thing given as the object property must be coerced into a string. The following is another example to this:

```javascript
let foo = {a: 1},
    bar = {b: 2},
    baz = {};

baz[foo] = 'hello';
baz
// Returns:
// { '[object Object]': 'hello' }
baz[bar]
// Returns:
// 'hello'
```

What's happening above is, when we write `baz[foo] = 'hello';`, JavaScript detects that `foo` is an identifier. Hence, JavaScript tries to resolve its value. It resolves it's value and detects that the value is an object. As we have indicated, object properties can only be strings or symbols, anything else must be _coerced into_ a string. Hence JavaScript coerces `foo` (`foo`'s value, which is an object) into a string. Coercing an object (any object) into a string _always_ gives the string `[object Object]` for any object, regardless of the complexity of the object or the property names present in the object. Hence, the line `baz[foo] = 'hello';` results in a new property named `[object Object]` is being created in `baz` and having the value `'hello'` assigned to it. Essentially, `baz[foo] = 'hello';` is equivalent to:

```javascript
baz['[object Object]'] = 'hello';
```

Then, we have the line `baz[bar]`. Here, JavaScript detects that `bar` is and identifier and hence resolves its value. It realizes that `bar` refers to an object. It then realizes that it is being used as an object property. Hence, it _coerces_ it into a string. As we have indicated, coercing _any_ object (regardless of the complexity, etc. of the object) into a string results in the string `[object Object]`. Hence, the line `baz[bar]` is equivalent to:

```javascript
baz['[object Object]']
```

[coerced]: https://developer.mozilla.org/en-US/docs/Glossary/Type_coercion
[autoboxing]: https://egghead.io/lessons/javascript-autoboxing-primitive-types-in-javascript
[MDN]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Property_accessors

----

`this` does not work the same for arrow functions as it does for conventional functions
=======================================================================================
`this` works as expected in "conventional" functions. "Conventional" functions are all "non-arrow functions". Example:

```javascript
const date = {
  year: 1990,
  month: 8,
  day: 16,
};
console.log(`date is ${date}`);
// date is [object Object]
date.toString = function() {
  return `${this.year}.${this.month}.${this.day}`;
};
console.log(`date is ${date}`);
// date is 1990.8.16
```

Another example:

```javascript
const date = {
  year: 1990,
  month: 8,
  day: 16,
  toString() { return `${this.year}.${this.month}.${this.day}` },
};
console.log(`date is ${date}`);
// date is 1990.8.16
```

However, when we use an arrow function:

```javascript
const date = {
  year: 1990,
  month: 8,
  day: 16,
};
console.log(`date is ${date}`);
// date is [object Object]
date.toString = () => `${this.year}.${this.month}.${this.day}`
console.log(`date is ${date}`);
// date is undefined.undefined.undefined
```

Similarly:

```javascript
const date = {
  year: 1990,
  month: 8,
  day: 16,
  toString: () => `${this.year}.${this.month}.${this.day}`,
};
console.log(`date is ${date}`);
// date is undefined.undefined.undefined
```

Examining what exactly happens when we use an arrow function gives us a better idea:

```javascript
const date = {
  year: 1990,
  month: 8,
  day: 16,
  toString: () => { console.log('this is', this) },
};
date.toString();
// this is <the global object. That is, `this` is the object named "global" on Node.JS, and "window" on a web browser>
```

The same example when we use a conventional function:

```javascript
const date = {
  year: 1990,
  month: 8,
  day: 16,
  toString() { console.log('this is', this) },
};
date.toString();
// this is { year: 1990, month: 8, day: 16, toString: [Function: toString] }
```

So, we observe that `this` value is set as we expect we we use a "conventional function", and when we use an arrow function, `this` value is just the global object.

`this` is not set to the caller in arrow functions. In arrow functions, `this` is searched in the lexical scope, as if it was another, regular "free" variable in a "closure".

There are more differences in arrow functions compared to regular functions. To learn more about them, you can [visit the MDN page](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions).

----

`const` does not provide anything for "immutability"
====================================================
```javascript
const date = {
  year: 1990,
  month: 8,
  day: 16,
  toString() { return `${this.year}.${this.month}.${this.day}` },
};

function person(name, DATE_OF_BIRTH) {
  const dateOfBirth = DATE_OF_BIRTH;
  return {
    identify() {
      console.log(`My name is ${name} and I was born at ${dateOfBirth}`);
    },
  }
}

const p = person('harry', date);
p.identify();
// My name is harry and I was born at 1990.8.16
date.year = 1960;
p.identify();
// My name is harry and I was born at 1960.8.16
```

As you can observe, even though we did not change anything in the person object at all, and even though we have declared the `dateOfBirth` as `const` in `person`, the value of the `dateOfBirth` still changed when we _mutated_ the `date` object. This is simply because when we pass the `date` to the person constructor, a _reference_ of the date is passed, since date is an object (well, a _copy of the reference_ is passed but where that reference _points to_ in the memory is the same location). Since the location pointed by both the `date` variable (the `date` `const`) outside the constructor and the `dateOfBirth` variable (the `dateOfBirth` `const`) in the person object are the same location, mutating what is pointed to by either of them mutates "both". Well, it does not mutate "both". It mutates a single object and since both references point _to the same location_, after mutation, they report the new, mutated version of the object. So, all `const` does in `person` is that it prevents _re-assignment_ to the `dateOfBirth` "field". That is, for example we cannot have a `reassignDateOfBirth` method in person like:

```javascript
function person(name, DATE_OF_BIRTH) {
  const dateOfBirth = DATE_OF_BIRTH;
  return {
    identify() {
      console.log(`My name is ${name} and I was born at ${dateOfBirth}`);
    },
    reassignDateOfBirth(newDateOfBirth) {
      // WON'T WORK. Will get "TypeError: Assignment to constant variable." exception
      dateOfBirth = newDateOfBirth;
    }
    mutateDateOfBirth(newDateOfBirth) {
      // Will work perfectly fine
      dateOfBirth.year = newDateOfBirth.year;
      dateOfBirth.month = newDateOfBirth.month;
      dateOfBirth.day = newDateOfBirth.day;
    }
  }
}
```

Hence, using `const` does not have much of a point. A better idea might be to deep clone each argument in the constructor:

```javascript
function person(name, DATE_OF_BIRTH) {
  var fields = {
    // Let's assume `cloneDeep` is a function available somehow (either
    // defined by us, or acquired from a library).
    name: cloneDeep(name),
    dateOfBirth: cloneDeep(DATE_OF_BIRTH),
  }
  return {
    identify() {
      console.log(`My name is ${fields.name} and I was born at ${fields.dateOfBirth}`);
    },
  }
}
```

Or, even a better idea might be to not clone anything at all, but always be aware of the mutability of the objects and hence, take extra care while programming not to mutate any objects except within a method. That is, perform the mutations only while the execution is within a method, and not perform any mutations when the code is outside them. However, this is not a complete solution either. Example:

```javascript
function person(name, DATE_OF_BIRTH) {
  const dateOfBirth = DATE_OF_BIRTH;
  return {
    identify() {
      console.log(`My name is ${name} and I was born at ${dateOfBirth}`);
    },
    mutateDateOfBirth(newDateOfBirth) {
      dateOfBirth.year = newDateOfBirth.year;
      dateOfBirth.month = newDateOfBirth.month;
      dateOfBirth.day = newDateOfBirth.day;
    },
  }
}

const date = {
  year: 1990,
  month: 8,
  day: 16,
  toString() { return `${this.year}.${this.month}.${this.day}` },
};

const p1 = person('harry', date);
p1.identify();
// My name is harry and I was born at 1990.8.16
const p2 = person('ron', date);
p2.identify();
// My name is ron and I was born at 1990.8.16
p1.mutateDateOfBirth({
  year: 1960,
  month: 10,
  day: 8,
});
p1.identify();
// My name is harry and I was born at 1960.10.8
p2.identify();
// My name is ron and I was born at 1960.10.8
/**
 * Assuming we don't have access to the source code of the person
 * "class" (that is, assuming we don't have any idea about the
 * implementation of the person class), we would expect `p2.identify();`
 * to print:
 *
 * My name is ron and I was born at 1990.8.16
 *
 * Since we think `p1.mutateDateOfBirth` would have mutated only `p1`'s
 * dateOfBirth. However, the reality is (the actual output of
 * `p2.identify();` is):
 *
 * My name is ron and I was born at 1960.10.8
 *
 * The reason is simply _mutability_. As you can observe, mutability is
 * (shared mutability I guess) is, in almost all cases, a bad thing.
```

Note that the same thing happens in Java as well. The reason is, the cause of this is _mutability of the objects_. It is not a peculiarity of JavaScript or something. In any programming language with mutable objects, this behavior happens. Java example:

```java
import java.util.Date;

class Person {
    private Date dateOfBirth;

    public Person(Date dateOfBirth) {
	    this.dateOfBirth = dateOfBirth;
    }

    public void sayDateOfBirth() {
        System.out.println("I was born at "
                            + (this.dateOfBirth.getYear() + 1900)
                            + "."
                            + this.dateOfBirth.getMonth()
                            + "."
                            + this.dateOfBirth.getDay()
        );
    }

    public void changeDateOfBirth() {
        this.dateOfBirth.setDate(this.dateOfBirth.getDate() + 10000);
    }
}

public class Main {
    public static void main(String[] args) {
        Date dob = new Date();
        Person p1 = new Person(dob);
        Person p2 = new Person(dob);
        p1.sayDateOfBirth();
        // Prints
        // I was born at 2020.10.1
        p2.sayDateOfBirth();
        // Prints
        // I was born at 2020.10.1
        p1.changeDateOfBirth();
        p2.sayDateOfBirth();
        // Prints
        // I was born at 2048.2.5
    }
}
/**
 * Again, even though we think that we have only changed `p1`'s date of
 * birth, we observe that `p2`'s date of birth has changed as well.
 */
```

----

# V8 (JavaScript Engine) "Caches" Serialized (Logged) Errors

Not sure if it is just V8, or this is something in the ECMAScript standard, but I observed this on Node.js v15.5.0 and Chromium 88.0.4324.96, both of which use V8.

The phenomenon is, when you log an `Error` instance, change the `message` property of it and log it again, the second log output is identical to the first log output, whereas I expected it to reflect the error with the new message.

On the other hand, if you don't log an `Error` instance before, change the `message` property of it and then log it, the log will have the updated error message.

This happens when the error is thrown as well. That is, if you have logged an error using `console.log`, changed the `message` property of the error and `throw`n the error, the terminal output as a result of the uncaught error will NOT include the updated `message` property.

Following is an example with logging error two times vs. one time. You can create an example that logs the error one time and throws it, vs. throws the error without logging at all. The results will be the same:

```javascript
logErrorTestHelper('TWO TIMES');
logErrorTestHelper('ONE TIME');


function logErrorTestHelper(logAmount) {
  const header = `TESTING LOGGING ERROR ${logAmount}`;
  console.log(`BEGIN: ${header}`);

  const error = Error('Hello');

  logStringRepresentation(error);
  // First log. Skip it if we will log only one time.
  if (logAmount === 'TWO TIMES') logVariable(error);

  const message = '. How are you?';
  error.message += message;
  console.log(`\nAppended "${message}" to error's 'message' property.`);

  logStringRepresentation(error);
  // Second log. The output will be identical to the first log's output
  // if the error was logged previously. Check the output to verify
  // this.
  logVariable(error);

  console.log(`END: ${header}`);
}

function logStringRepresentation(error) {
  console.log('\nLogging error\'s string representation:');
  console.log(error.toString());
}

function logVariable(error) {
  console.log('\nLogging error:');
  console.log(error);
}
```

Output:

```
$ node error-log-test.js
BEGIN: TESTING LOGGING ERROR TWO TIMES

Logging error's string representation:
Error: Hello

Logging error:
Error: Hello
    at logErrorTestHelper (/path/to/error-log-test.js:9:17)
    at Object.<anonymous> (/path/to/error-log-test.js:1:1)
    <Rest of the call stack>

Appended ". How are you?" to error's 'message' property.

Logging error's string representation:
Error: Hello. How are you?

Logging error:
Error: Hello
    at logErrorTestHelper (/path/to/error-log-test.js:9:17)
    at Object.<anonymous> (/path/to/error-log-test.js:1:1)
    <Rest of the call stack>
END: TESTING LOGGING ERROR TWO TIMES
BEGIN: TESTING LOGGING ERROR ONE TIME

Logging error's string representation:
Error: Hello

Appended ". How are you?" to error's 'message' property.

Logging error's string representation:
Error: Hello. How are you?

Logging error:
Error: Hello. How are you?
    at logErrorTestHelper (/path/to/error-log-test.js:9:17)
    at Object.<anonymous> (/path/to/error-log-test.js:2:1)
    <Rest of the call stack>
END: TESTING LOGGING ERROR ONE TIME
$
```
