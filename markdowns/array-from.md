Chances are good that, while working with JavaScript, you've encountered data structures that _look_ like arrays, but aren't _really_ arrays. Chances are also good that you then ended up trying to call an `Array` method on that structure, only to have JavaScript throw an error back your way.

Encountering array-like objects is pretty common in JavaScript. If you aren't familiar with array-likes, here's an example:

```javascript runnable
var arrayLike = {
    length: 3,
    0: 4,
    1: 5,
    2: 6
};
console.log(arrayLike.length);
console.log(arrayLike[1]);
```


A commonly encountered example is the `arguments` array-like, which can be used to create functions that take a variable number of arguments like in the following example.

```javascript runnable
function sum() {
    var l = arguments.length;
    var total = 0;
    for (var i = 0; i < l; i++) {
        total += arguments[i];
    }
    return total;
}

console.log(sum(4, 5, 6));
```

So far, `arguments` looks like a normal array, right? You can determine its length via the `length` property, and you can index the object just like any other array with square brackets. 

Now let's imagine that you remember that you could use `Array.reduce` instead of a `for` loop. But what happens when you try that?

```
Error: arguments.reduce is not a function
```

Oh, right -- `arguments` isn't an `Array` and so doesn't inherit any of the methods that work on arrays. How can we fix that?

In ES5 you'd write something like this:

```javascript runnable
function sum() {
    var args = [].slice.call(arguments); // [A]
    return args.reduce(function (acc, num) {
        return acc + num;
    }, 0);
}

console.log(sum(4, 5, 6));
```

In **[A]** we take advantage of `call` to convince `slice` to convert `arguments` into a real `Array`. It works because `slice` can operate on array-like objects. If we do the same thing to the first example, we'd get back a real array of `4, 5, 6`, and the above example works on the same principle.

Another common occurence is when you work with the DOM (Document Object Model) in a web view. You can ask the DOM to return all elements matching a certain criteria, but the return result isn't an array -- it just looks like one. Thankfully the same pattern works here too.

```javascript
var elNodeList = document.querySelectorAll("p"),
    els = [].slice.call(elNodeList);
```

With ES2015, however, there is a simpler way we can convert array-likes into arrays. Do note, however, that ES2015 doesn't change the fact that `arguments` and similar is still _not_ an `Array` -- it just gives us a nicer tool for converting such things.

Say hello to `Array.from`. We can pass anything that looks like an array to it, and we'll get an `Array` back. Let's revisit our `sum` example using `Array.from` instead:

```javascript runnable
function sum() {
    var args = Array.from(arguments); // [A]
    return args.reduce(function (acc, num) {
        return acc + num;
    }, 0);
}

console.log(sum(4, 5, 6));
```

Does this save on a lot of code? No, not particularly -- but it is much easier to read, in my opinion. Plus, for some reason, I can never keep `slice` and `splice` separate in my head, so inevitably I end up having to do a Google search to remind myself which one I should use.

So, you might think that's all there is to `Array.from`, right? Well... not quite. Because it works on array-likes, we can use it to do some interesting things.

For example, `Array.from` can be used to split a string into its component characters:

```javascript runnable
const chars = Array.from("Hello!");
console.log(chars);
```

Or, we could do something like this:

```javascript runnable
const items = Array.from({
        0: "a",
        1: "b",
        2: "c",
        length: 3
    });
console.log(chars);
```

But that’s not all — `Array.from` doesn’t just take one argument — it can take three. Here’s the signature:

> `Array.from(arrayLike [, mapFn [, thisArg]]) -> Array`

Hmm — interestingly enough, `Array.from` acts a bit like `Array.map`, and is mostly functionally equivalent to `Array.from(arrayLike).map(mapFn [, thisArg])` so why would one ever write anything else? Simple: it saves some memory and also avoids some unexpected results by avoiding the creation of an intermediate array. This isn’t something you’ll run into with run-of-the-mill array-likes, but can cause problems with certain array subclasses, like `TypedArrays`.

> **Side note**: `TypedArray` also has a `from` method, which converts the array-like into a typed array. It has some subtle differences, though, so be sure to check out [Mozilla’s documentation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/from).

So we could take the above and write the following:
```javascript runnable
const items = Array.from({
        0: "a",
        1: "b",
        2: "c",
        length: 3
    }, c => c.toUpperCase());
console.log(chars);
```

We can also use this feature to create arrays of arbitrary length. An array-like only needs to contain a length property, so we can do this:

```javascript runnable
const tenIntegers = Array.from({length:10}, (_, idx) => idx);
console.log(tenIntegers);
```

We can use this to create arbitrary sequences too, which can be quite useful. For example, we can easily create the first few powers of two:

```javascript runnable
const firstPowersOfTwo = Array.from({length: 11}, (_, idx) => 
      2 ** idx);
console.log(firstPowersOfTwo);
```

It’s also worth mentioning that `Array.from` works on anything that is Iterable, so it works on Maps, Sets, and Generators as well. Those are larger topics in and of themselves, so look for those articles in the future!
