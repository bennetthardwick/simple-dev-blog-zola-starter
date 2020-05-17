+++
title = "Making Typescript packages JavaScript friendly"
description = "Everyone assumes that Typescript packages are backwards compatible with JavaScript by default, but importing won't always work exactly how you think."
aliases = ["/blog/2018-02-06-making-typescript-packages-javascript-friendly/"]
[taxonomies]
tags = ["javascript", "typescript", "beginner"]
+++

I don't know about you, but when I first started using Typescript it was an absolute game changer for me. Even for simple projects, where I'd be writing less than 100 lines of JavaScript, I'd always opt-in to using Typescript instead.

Now, this wasn't because I'm a massive fan of Microsoft, or even because of static typing, it was solely because Sublime Text would give me hints as I typed, but besides, why would it matter what I used? The compiled JavaScript would be usable by everyone... Right?

If you've used Typescript, you've definitely had to deal with importing a JavaScript package before, it goes something like this:

```typescript
import * as express from "express";
```

Isn't that just so ugly?

Okay, so we've imported a JavaScript module with Typescript, let's create a Typescript module to require with JavaScript.

```typescript
// Greeting.ts
export class Greeting {
  private greeting;

  constructor(name: string, greeting: string) {
    this.greeting = greeting;
  }

  greet() {
    console.log(`${name} says ${this.greeting}`);
  }
}
```

Let's try importing that with Typescript

```typescript
import { Greeting } from "Greeting";
let hello = new Greeting("Me", "Hi!");
hello.greet();
// Me says Hi!
```

And with JavaScript

```javascript
const Greeting = require("Greeting").Greeting;
let hello = new Greeting("Me", "Boo!");
hello.greet();
// Me says Boo!
```

That Javascript is a little bit tedious, but it's understandable since we're choosing one of the exports of the Greeting module anyway. Let's try and make that a little bit better.

```typescript
// Greeting.ts
export default class Greeting {
  ...
}
```

Now we shouldn't have to choose a module when we import with Typescript

```typescript
import Greeting from "Greeting";
let hello = new Greeting("Me", "Hey!");
hello.greet();
// Me says Hey!
```

Awesome! Since we're not choosing an export like last time, requiring with Javascript should work perfectly.

```javascript
const Greeting = require("Greeting");
let hello = new Greeting("Me", "Hello!");
hello.greet();
// undefined
```

Wow, that's a little bit annoying. What's going on? Well when Typescript tries to compile a default export into Javascript, it can't just make the whole module the export. Instead, it does it like this

```javascript
...
module.exports.default = Greeting;
...
```

So now the part that you've been waiting for. The obvious answer to a timeless question that no-one is going to ask.

> How do I write Typescript modules that work exactly the same as Javascript modules would?

Well, you'd write them like you would in Javascript of course!

There is a little bit of a catch though, you can't write it like this.

```typescript
export default class Greeting {
  ...
}
module.exports = Greeting;
```

Why? Because the Javascript would look something like this

```javascript
module.exports.default = Greeting;
module.exports = Greeting;
```

and since Typescript looks for that `.default` when importing, you're gonna have a whole bunch of problems. Infact, the solution likes in the `index.ts` of your module.

```typescript
import Greeting from "./Greeting";
module.exports = Greeting;
export default Greeting;
```

Voilà! We've created a module that's intuitive to import with both Typescript and JavaScript.

There are a few caveats though, most notably that if the main export class has any methods that share the same name as other exports there's going to be issues. This won't be a problem 50% because of naming conventions (classes having capital letters or whatever), but if you're exporting a similarly named function or variable you're going to run into issues.

Just on a final note, [kinda like my first article](/blog/2017-10-31-params-in-c-go-backwards) this whole making Typescript packages intuitive to use with JavaScript is really a non-issue, since no one guesses how to use a package without first reading the documentation. Oh well.
