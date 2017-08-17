How many times have you written code that looks like this?

```javascript
var elNodeList = document.querySelectorAll("p"),
    els = [].slice.call(elNodeList);
```

or

```javascript
var args = [].slice.call(arguments);
```

Sigh… if I only had a nickel…

Anyway, with `Array.from` we can bypass `slice` (and if you’re like me, the inevitable Google search to remember if it wasn’t `splice` instead) and use the following:

```javascript
const els = Array.from(document.querySelectorAll("p"));
const args = Array.from(arguments);
```

Done, and done!

Well, not quite. As is obvious from the above code, Array.from takes something that quacks like an array and converts it into something that is a real array, but because of this fact, it is capable of quite a lot of things!

For example, Array.from can be used to split a string into its component characters:

```javascript runnable
const chars = Array.from("Hello!");
console.log(chars);
// chars = ["H", "e", "l", "l", "o", "!"]
```

Or, we could do something like this:

```javascript runnable
const items = Array.from({
        "0": "a",
        "1": "b",
        "2": "c",
        "length": 3
    });
console.log(chars);
// items = ["a", "b", "c"]
```

But that’s not all — Array.from doesn’t just take one argument — it can take three. Here’s the signature:
Array.from(arrayLike [, mapFn [, thisArg]]) -> Array
Hmm — interestingly enough, Array.from acts a bit like Array.map, and is mostly functionally equivalent to Array.from(arrayLike).map(mapFn [, thisArg]) so why would we ever do anything else? Simple: it saves some memory and also avoids some unexpected results by avoiding the creation of an intermediate array. This isn’t something you’ll run into with run-of-the-mill array-likes, but can cause problems with certain array subclasses, like TypedArrays.
Side note: TypedArray also has a from method, which converts the array-like into a typed array. It has some subtle differences, though, so be sure to check out Mozilla’s documentation.
So we could take the above and write the following:
const items = Array.from({
        "0": "a",
        "1": "b",
        "2": "c",
        "length": 3
    }, c => c.toUpperCase());
// items = ["A", "B", "C"];
We can also use this feature to create arrays of arbitrary length. An array-like only needs to contain a length property, so we can do this:
const tenIntegers = Array.from({length:10}, (_, idx) => idx);
// tenIntengers = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
We can use this to create arbitrary sequences too, which can be quite useful. For example, we can easily create the first few powers of two:
const firstPowersOfTwo = Array.from({length: 11}, (_, idx) => 
      2 ** idx);
// [1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024]
It’s also worth mentioning that Array.from works on anything that is Iterable, so it works on Maps, Sets, and Generators as well. Those are larger topics in and of themselves, so look for those articles in the future!
