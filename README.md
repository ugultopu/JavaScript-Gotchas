Accessing a non-existent property of an object returns `undefined`. It does not raise an exception. JavaScript treats `undefined` just like another value. It allows performing comparisons with it. For example, `undefined > 5` does not raise an exception. It simply returns `false`.

The problem with this is, when you mistype/misspell an object property, there is no indication about this in JavaScript. Since a non-existent property simply returns `undefined`, and since JavaScript simply treats `undefined` like a regular value, your program will just break and you won't have any idea about the reason. An example:

    const a = [1,2,3];
    if (a.len !== 3) console.log("a's length is not 3!");

The code above will print "a's length is not 3!". Because of a simple mistyping, our program is broken without any indications. We have typed `a.len` instead of `a.length`. Since returning `undefined` when accessing a non-existent property (and not throwing an exception, printing a warning, etc.) is the default behavior in JavaScript, we didn't get any indication/warning about this.

The solution to this is to [use TypeScript][My answer about non-existent property access]. Had we used TypeScript, we would have gotten an error right on the "compile time" (that is, on our editor before we even run the code) saying `Property 'len' does not exist on type 'number[]'` and underlining the erroneous part of the code with red squiggles.

[My answer about non-existent property access]: https://stackoverflow.com/a/63548095/3395831

----

You cannot create an array of arrays using [`Array.prototype.fill()`]. Specifically, you cannot create an array of objects using `Array.prototype.fill()`, because every element in the array will be the _exact same object_. Hence, modifying _any_ element of the array modifies _every_ element of the array, since every element is the _exact same element_ (same reference). Example:

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

Instead, you can use `Array.from(Array(5), () => Array(3).fill(null))` to get the expected behavior.

[`Array.prototype.fill()`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/fill
