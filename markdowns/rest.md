The “rest” operator (`...`) collects the remaining items from an iterable into an array. It’s used hand-in-hand with destructuring and with argument lists, and is really pretty awesome. 

Note: In this section I’m only going to cover Rest’s association with iterables. There is a proposal at stage 3 that allows it to work with objects, and you’ve probably already seen it in production code when using Babel as a transpiler alongside React or other similar frameworks. Even so, it’s not standard yet, so I’m not covering it in this post.

Let’s imagine that we’re writing a logging function that needs to take a format string and an arbitrary number of additional arguments. In ES5 we’d have written the following (expand to see the full code snippet if you want):

```javascript runnable
function fmt(formatStr) {
    var args = [].slice.call(arguments, 1),
// { autofold
        regex = /(\%[s|i|f])/g,
        pieces = (formatStr || "").split(regex),
        argIdx = 0,
        str = pieces.reduce(function(acc, piece) {
            if (piece[0] === "%") {
                switch(piece[1]) {
                    case "s":
                        acc += args[argIdx++].toString();
                        break;
                    case "f":
                        acc += parseFloat(args[argIdx++]);
                        break;
                    case "i":
                        acc += parseInt(args[argIdx++], 10);
                        break;
                    default:
                        acc += piece;
                }
            } else acc += piece;
            return acc;
        }, "");
// }
}

console.log(fmt("Hello, %s! The answer to the ultimate question is %i", "World", 42));
```

I have several problems with a function like this:

* It requires conversion of `arguments` to an `array` using `[].slice.call`.
* The code has to avoid the first argument, since the function shouldn't treat the format string as an additional argument.
* It’s not at all self-documenting — the arity (number of arguments taken) looks like 1, but in reality, the function can take any number of arguments. Without additional documentation, it’s not immediately obvious that this function can take any number of arguments, and is likely to confuse any IDE that attempts to provide some degree of code insight.

But with ES2015, things become clearer:

```javascript runnable
function fmt(formatStr = "", ...args) {
// { autofold
    let argIdx = 0;
    const regex = /(\%[s|i|f])/g,
          pieces = formatStr.split(regex),
          str = pieces.reduce((acc, piece) => {
            if (piece[0] === "%") {
                switch(piece[1]) {
                    case "s":
                        acc += args[argIdx++].toString();
                        break;
                    case "f":
                        acc += parseFloat(args[argIdx++]);
                        break;
                    case "i":
                        acc += parseInt(args[argIdx++], 10);
                        break;
                    default:
                        acc += piece;
                }
            } else acc += piece;
            return acc;
        }, "");
    return str;
    // }
};

console.log(fmt("Hello, %s! The answer to everything is %i", "World", 42));
```

So, what do I like about this? Several things:
* args is an actual array containing all the arguments after the format string — no fiddling with slice and no need to specify an offset.
* Self-documenting. We know that if nothing is provided, formatStr will be an empty string, and it’s obvious from the signature that the function can accept any number of additional arguments.
* Extra bonus: args is always guaranteed to be an Array. If no additional arguments are passed, it just happens to be an empty array with length zero.

The rest operator works similarly with destructuring as well:

const [favoriteColor, ...otherColorsILike] = "purple blue pink red black".split(/\s+/);

Here, favoriteColor will receive the value “purple”, and otherColorsILike will be an array of ["blue", "pink", "red", "black"] .

There is a catch to the operator, and it’s implied in the name: it can only collect the rest of the items in a list. That is, it has to be at the end of a list — it can’t be in the middle somewhere. Which… is kind of unfortunate. I mean, how cool would be to write something like this:

const [favoriteColor, ...otherColorsILike, leastFavoriteColor] = "purple blue pink red black".split(/\s+/);

But don’t do it — it won’t work. If you don’t believe me, try it. You’ll get an error that reads like “Unexpected token ‘,’. Expected a closing ‘]’ following a rest element destructuring pattern.”
