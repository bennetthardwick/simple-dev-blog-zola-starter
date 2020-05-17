+++
title = "Writing safe, efficient and parallel Node.js extensions with Rust, Neon and Rayon"
slug = "writing-safe-efficient-parallel-native-node-extensions-in-rust-and-neon"
description = "Rayon and Neon allow you to take full advantage of your system's hardware by writing super fast parallel native JavaScript modules with Rust."
side-by-side = true
aliases = ["/blog/2019-11-26-writing-native-node-extensions-in-rust/"]
[taxonomies]
tags = [ "rust", "javascript", "nodejs" ]
+++

When writing server applications with Node.js, sometimes JavaScript just doesn't cut it and you need to employ native code.
This could be because of a few different reasons:

1. You need to complete a CPU-bound task
  Because JavaScript runs inside a VM, every operation will take longer than the native equivalent.
  At a high level, this is because a VM needs to interpret the code and translate it into machine code to run on the system as the program is executing.
  This process is greatly improved by techniques such as Just-In-Time (JIT) compilation but it's still nowhere close to native speeds.

1. You need to quickly and often create and destroy memory
  As JavaScript is a garbage-collected language, creating and destroying lots of object and arrays can be very expensive and slow.
  This is compounded by the fact that no-one really knows when the JavaScript garbage compiler runs, resulting in random and unpredictable stalls.

1. You need to multi-thread your code
  Node.js recently added the Worker API which allow developers to multi-thread their JavaScript code.
  Unfortunately, not only is the overhead of creating workers and message passing is quite high, but the interface is tedious.
  Libraries like microjob have made efforts to combat both points, but projects like Rayon in Rust are more performant and easier to work with.

Typically when you needed a native module you'd be stuck writing add-ons in C or C++ - but thanks to Neon, this is no longer the case.

## Installing Rust

In order to start developing Rust code to work with JavaScript - you first need to install the Rust development tool-chain.
This is first done by installing the Rust tool-chain manager `rustup`. [You can lean how to do it here.](https://www.rust-lang.org/tools/install).
After installing `rustup` and configuring your `PATH`, it's time to install Neon.

## Getting started with Neon

```

```

Before starting a Neon project, you have to install the CLI. The Neon CLI can easily be installed by using the NPM package manager.

```
npm i -g neon-cli
```

After the installation is complete, you can bootstrap a new project by running the `new` command.

```
neon new <project-name>
```

With the new project created, navigate to the new `<project-name>` directory and run `npm install` to install dependencies.
Neon also builds your rust code through the `install` command, so you'll see it compile as well.
If there are any errors at this point, ensure that you have the Rust tool-chain properly installed.

## Creating your first project

Let's make a simple project that finds the factorial of `n`.

```

```

Before getting started, you'll need to add the `rayon` crate to your `native/Cargo.toml` file.

Rayon is an awesome library with a simple interface that allows you to take full advantage of your system by multi-threading many of the common tasks done by Rust iterators.

```toml
[dependencies]
neon = "0.3.3"
rayon = "1.2.1"
```

After adding it to your `native/Cargo.toml` file, run `cd native && cargo build` to check that everything builds correctly.

Now that everything is ready to go, start by opening `native/src/lib.rs` in your favourite editor.

```

```

In order to use `neon` and `rayon` they need to be imported.
`rayon` and `neon` both expose a prelude of common stable APIs.
For convenience, just include the entire prelude.

```rust
extern crate neon;
extern crate rayon;

use neon::prelude::*;
use rayon::prelude::*;
```

As this task is going to be multi-threaded, it makes sense to make it not block the main JavaScript thread.
In order to write async code with Neon, we have to create a "Task" struct.
The contents of this struct is extra information / params that are needed by the async task.
We'll add `n` to the struct, to depict the number that we will sum up to.

```rust
struct FactorialTask {
    n: usize
}
```

In order to allow Neon to run our task we need to implement the `Task` trait.
The task trait requires you to define three types `Output`, `Error` and `JsEvent`.

- The `Output` type is the output of your task - as a rust type.
- The `Error` type is the type of error your task would throw - as a rust type.
- The `JsEvent` type is the output of your task - as a JavaScript type.

Neon exposes every type that is supported by JavaScript. For a full list, [visit the neon docs page](https://neon-bindings.com/api/neon/types/).

```rust
impl Task for FactorialTask {
    type Output = u128;
    type Error = String;
    type JsEvent = JsNumber;
```

The next part of the Task trait that needs to be implemented is the `perform` function.
This is the function is what does all the work.

```

```

The factorial of a number if all the numbers from `1` to `n` multiplied together.
When the task is run in parallel `1 * 2` may be added together in one thread while `3 * 4` is added together in another thread, and so on.
In theory, for large values of `n` this should be very fast - although at that point we might exceed the limit of `u128`.

`into_par_iter()` is what does the `rayon` magic here. Another variant on this is the common `par_iter()` which you can use on `Vec` and slices.

```rust
    fn perform(&self) -> Result<Self::Output, Self::Error> {
        let result = (1..self.n + 1)
            .into_par_iter()
            .map(|x| x as u128)
            .reduce_with(std::ops::Mul::mul);

```

As there are some cases when `reduce_with` can fail, `result` will be set to an `Option` object.
Using Rust pattern matching, you can check for a value, else return an error.

```rust
        if let Some(value) = result {
            Ok(value)
        } else {
            Err(String::from("Something went wrong"))
        }
    }
```

After that is complete, the final part of the `Task` trait to implement is the `complete` function.
This function is called after the async task is completed, and it's main role is to convert the output value of the task into something that is readable by Neon and JavaScript.

In this example, we convert our result to a number using `cx.number()`, although the respective methods exist for each primitive.

```rust
    fn complete(
        self,
        mut cx: TaskContext,
        result: Result<Self::Output, Self::Error>,
    ) -> JsResult<Self::JsEvent> {
        Ok(cx.number(result.unwrap() as f64))
    }
}
```

Next you need to create the JavaScript handler method.
Inside this method you need to parse the arguments from the JavaScript call and register the task.
Arguments are parsed using the `cx.argument::<type>()` method.
If the coercion to a particular type fails, it will return an error from the JavaScript function.

```rust
fn factorial(mut cx: FunctionContext) -> JsResult<JsUndefined> {
    let n = cx.argument::<JsNumber>(0)?.value() as usize;

    let task = FactorialTask { n };

    let callback = cx.argument::<JsFunction>(1)?;
    task.schedule(callback);

    Ok(cx.undefined())
}
```

With all the code complete, you can use the `register_module!` macro from the `neon` crate to register
your JavaScript function.

```rust
register_module!(mut cx, { cx.export_function("factorial", factorial) });
```

## Build and Run

Now that the native component of the library is complete, it's time to build and run the extension.
To do so, just run `npm install` from your extension.
You should see messages from `cargo` stating that's it's compiling.

```

```

To execute you extension, update `lib/index.js` to call the `factorial` function.

```javascript
var addon = require("../native");

addon.factorial(10, (error, value) => console.log(value));
```

And just like that, you've created your own hyper-parallel, native extension for Node.js.
If you'd like to see more of the project, check out the [Github repo](https://github.com/bennetthardwick/factorial-rayon-neon).

## Notes

- By default the `package.json` file runs `neon build`. This only runs a debug build, to enable to optimized production build change it to `neon build --release`
- This factorial is only an example and doesn't work past numbers after 25 or so. If doing this for real it's probably a good idea to use a BigNum implementation of some sort, such as [this rayon example](https://github.com/rayon-rs/rayon/blob/master/rayon-demo/src/factorial/mod.rs)
- Parsing JavaScript objects in Neon is very slow, it's best to parse them all at once and only work with Rust structs
- There's currently no good way of distributing compiled Neon binaries. Anyone that consumes your library will need to have the Rust tool-chain installed
