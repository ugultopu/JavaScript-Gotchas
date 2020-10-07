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
