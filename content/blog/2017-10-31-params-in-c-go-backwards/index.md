+++
title = "Parameters go backwards in C"
description = "C parameters don’t work like other programming languages… except when they do."
aliases = [ "/blog/2017-10-31-params-in-c-go-backwards/" ]
[taxonomies]
tags = ["c", "beginner"]
+++

The funny thing about C, is that it gets harder the longer you spend trying to learn it.

In the beginning, C seems like an incredibly simple language. Hell, [there’s only 32 keywords!][32-keywords] But as soon as you nail the basics, it’s difficulty increases exponentially.

Coming from a background in JavaScript, I went through a complete paradigm shift when I began to learn C. [Concepts like pointers](https://www.guru99.com/c-pointers.html) just didn’t stick at first, and I had some serious JavaScript baggage.

A lot of JavaScript logic is present in C. Functions are executed sequentially, multiplying two numbers together returns their product, and up until recently, I would have assumed that function arguments were executed from left to right. Take a look at this:

```c
#include <stdio.h>

int a = 0;

int increment(){
    return ++a;
}

int main(){
    printf("%d and %d\n", 1, 2);
    printf("%d and %d\n", increment(), increment());
    return 0;
}
```

I’ve written a function which increments a global variable `a`, and returns the result. The first line returns `1 and 2`, but what’s surprising is that the second line returns `2 and 1`.

### Why does this happen?

Well, according to [stack overflow][passing-args-reverse], it comes down to C functions' way of handling a variable number of arguments. Basically, parameters are called and pushed in reverse so that the left-most parameter is always on the top of the stack.

### Does it matter?

Yes and no. I think it's always best to write code that works no matter what order the parameters are executed. Calling convention can change depending on architecture and compiler, and I think it's always better to be sure that your code will compile the way you want it to. Remember the first segment? This is the output when compiled with `clang`:

```
1 and 2
1 and 2
```

Now you could just always use `clang`, but I think structuring your code like this by habit is the best idea:

```c
#include <stdio.h>

int a = 0;

int increment(){
    return ++a;
}

int main(){
    int i = increment(), j = increment();
    printf("%d and %d\n", i, j);
    return 0;
}
```

[passing-args-reverse]: https://stackoverflow.com/questions/18690322/what-is-the-point-of-passing-arguments-in-the-reverse-order-in-c
[32-keywords]: https://www.programiz.com/c-programming/list-all-keywords-c-language
