+++
title = "JavaScript by reference AKA opening up a box of function side effects"
description = "Objects and Arrays are passed by reference to JavaScript functions. Who would have known."
aliases = ["/blog/2018-03-30-javascript-by-reference-aka-function-side-effects/"]
[taxonomies]
tags = ["javascript"]
+++

I've been using JavaScript for quite a long time. Up until now, I felt like I was some kind of expert - no project was too large or too small for me to complete! I was unstoppable!

That was until I found out that objects and arrays are created by reference, and my world collapsed.

Normally, I'd write something like this:

```js
let arr = [ 0, 1, 2, 3 ]
array = split(arr); // [ 0, 1 ]

function split(array) {
  return array.splice(0, Math.floor(array.length / 2);
}
```

Simple enough. Except it's not working exactly like I think it is. Sure, it's giving the right result, but what about the original array?

```js
console.log(array); // [ 2, 3 ]
```

Whoa! That could cause some serious bugs.

Well apparently I hadn't read the [docs very well](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/splice), because it clearly shows it working how it's supposed to.

Okay, fine, whatever. Let's try something else.

```js
let hey = ["hi"];
let hello = hey;
let hi = hello;

hi.push("there");
```

Yep. Even though I only pushed to `hi`, they all changed.

```js
console.log(hi); // [ 'hi', 'there' ]
console.log(hey); // [ 'hi', 'there' ]
console.log(hello); // [ 'hi', 'there' ]
```

By now it should be obvious how JavaScript really works, and to be honest, it's quite embarrassing that I hadn't figured it out earlier. Tutorials on the internet are riddled with lines like:

```js
let elem = document.getElementById("elem");
elem.innerHTML = "wow";
```

How did I miss it? Maybe adding the DOM into the equation threw me a curveball? Who knows.

## Curing function side effects

You can make a shallow copy of an array by using the `Array.slice()` prototype. This is fine for most arrays, and it's built right into the JavaScript engine so it's snappier than most solutions.

Trying this on a multidimensional array won't go down very well though. It'll make a copy of the first dimension sure, but the rest will still be by reference.

### Enter Lodash

I've never had a reason to use Lodash. I hated the idea of manipulating arrays and objects using JavaScript code because of how slow it is. But I guess I have to do what I have to do.

You can deep copy in lodash by issuing:

```js
_.slice(array);
```

Now that's that! I've only really spoken about arrays, but objects are pretty much the same.

It all makes sense though. I'm sure it's part of the reason why memory leaks aren't a large concern for most JavaScript developers. Also it's the reason why JavaScript works so well with the DOM in the browser.
