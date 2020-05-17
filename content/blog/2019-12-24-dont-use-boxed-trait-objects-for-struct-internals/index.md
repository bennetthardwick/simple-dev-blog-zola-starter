+++
title = "Don't use boxed trait objects for struct internals"
description = "When writing a struct with the intention of it being reused, it’s important not to use boxed trait objects to represent interior data as they make it very hard to consume."
aliases = ["/blog/2019-12-24-dont-use-boxed-trait-objects-for-struct-internals/"]
[taxonomies]
tags = ["rust","beginner"]
+++

When writing a struct with the intention of it being reused, it's important not to use boxed trait objects to represent interior data.

Namely, this is because turning an object into a `Box<dyn Trait>` loses a lot of type information about the object which is difficult to get back should the developer consuming your struct need it.

### What's a `Trait`?

According to the [rust book](https://doc.rust-lang.org/book/ch10-02-traits.html), traits are Rust's method for defining shared behaviour.
Much like interfaces in other languages, Rust traits are a method of abstraction that allows you to define a schema through which you can
communicate with an object - [and a lot more](https://blog.rust-lang.org/2015/05/11/traits.html).

At it's core, a trait describes a certain behaviour and should only provide methods that achieve that behaviour.

### Boxed trait objects

Since the size of a trait is not known at compile time (anything can implement a trait, no matter what size) it's hard to store an object based on the trait it implements since the compiler doesn't know exactly how much space to make available.

Rust's solution to this is to put a trait inside a `Box`, `Arc` or `Rc` and store that container instead.

This is a very useful feature which allows you to do all kinds of awesome stuff - like creating vectors of traits of unknown sizes.

However, like all things, this typically comes at a cost. Especially when designing a struct that will be consumed by different developers with unknown and evolving use cases.

## Problems with an interior `Box<dyn Trait>`

Say I had a trait that depicted a person.
I might create a method inside that trait that allows the person to `say_hello`.

```rust
trait Person {
  fn say_hello(&self);
}
```

It would be super easy to implement and allow each different person to say hello in their favourite greeting.

Down the track my requirements for people might change - I might need to group them in a `PeopleZoo`.

Typically, I might model this using a vector of boxed trait objects.

```rust
struct PeopleZoo {
  people: Vec<Box<dyn Person>>
}

impl PeopleZoo {
  fn add_person(&mut self, person: Box<dyn Person>) {
    self.people.push(person);
  }

  fn last_person(&self) -> Option<&Person> {
    self.people.last()
  }
}
```

This would work great in most cases.
It allows each `Person` to say their own greeting and be stored in the same vector.

I can even create my own structs and begin adding people into the zoo.

```rust
struct Me {
  name: &'static str
}

impl Person for Me {
  fn say_hello(&self) {
    println!("Hello, it's me.");
  }
}

```

However, this starts to fall apart as soon as I begin to do anything complex.
For example, if I wanted to get the last person added into the zoo and if they're "Me" check what their name is.

```rust
let mut zoo: PeopleZoo = PeopleZoo { people: vec![] };
zoo.add_person(Box::new(Me { name: "Bennett" }));

// How can I figure out that this is "Me"?
let person = zoo.last_person().unwrap();
```

This is where things get a little annoying.
We have a type with not much information that we want to turn back into our original type.

The safe way do this is to cast a reference to `&dyn Any` and use the `downcast` family of functions to get a reference to our original type back.

Unfortunately this means that we have to retrofit our original `Person` trait with a method that allows us to do this.

```rust
trait Person {
    fn say_hello(&self);
    fn as_any(&self) -> &dyn Any;
}
```

Since it's now on the root trait, we would also have to update every implementation of that trait to include this seemingly useless method - as well as force anyone that decides to use our trait to implement it as well.

```rust
impl Person for Me {
    fn say_hello(&self) {
        println!("Hello, it's me.")
    }

    fn as_any(&self) -> &dyn Any {
        self
    }
}
```

Finally, we would need to execute a long chain of functions to get back to our original type.

```rust
let mut zoo: PeopleZoo<Me> = PeopleZoo { people: vec![] };
zoo.add_person(Me { name: "Bennett" });

let me: &Me = zoo
    .last_person()
    .unwrap()
    .as_ref()
    .as_any()
    .downcast_ref::<Me>()
    .unwrap();
```

<p><a target="_blank" href="https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=f4a5360fec6f40b7c68216214a52f060">Full Example</a></p>

It doesn't seem very nice and we'd have to force this upon every user of our struct just in case someone might want to get back to their original type.

Thankfully, we can refactor our struct to make this a little bit easier.

## Generics

When writing a trait, I shouldn't have to worry about providing information that is irrelevant to the function of that trait.

It's better to instead leverage Rust's type system and use traits as a way to describe the form of the data that's provided, whilst letting the user provide whatever data they like (provided it fits within the traits constraints).

In the case of `PeopleZoo`, it's as simple as making it take a generic parameter `P` that is a `Person`.

```rust
struct PeopleZoo<P: Person> {
    people: Vec<P>,
}

impl<P: Person> PeopleZoo<P> {
    fn add_person(&mut self, person: P) {
        self.people.push(person);
    }

    fn last_person(&self) -> Option<&P> {
        self.people.last()
    }
}
```

Next, remove the `as_any` calls from the traits and implementation. This then allows you to simplify the calls to get people from the zoo.

```rust
let mut zoo: PeopleZoo<Me> = PeopleZoo { people: vec![] };
zoo.add_person(Me { name: "Bennett" });

let me: &Me = zoo
    .last_person()
    .unwrap();
```

<p><a target="_blank" href="https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=59c28f6fa71b2bf7888dca26b6f30854">Full Example</a></p>

Our code is a lot simpler now, but unfortunately we've lost a lot of flexibility.
The previous version of `PeopleZoo` allowed any arbitrary number of objects implementing the
`Person` trait.

## Enum wrappers for trait objects

A common pattern that I've noticed which gives you full control of the data that you pass into a struct is creating an enum wrapper for trait objects.

What I mean by this, is creating an enum that implements the same trait that each of it's values implements.

To demonstrate, I'll first create another struct implementing `Person`.

```rust
struct Grandma {
    age: usize
}

impl Person for Grandma {
    fn say_hello(&self) {
        println!("G'day!")
    }
}
```

Since I've now got more than one `Person`, it makes sense to create a wrapper enum to keep track of them and allow easy access through Rust's pattern matching syntax.

Furthermore, since Rust enums are [tagged unions](http://patshaughnessy.net/2018/3/15/how-rust-implements-tagged-unions), they'll only have the memory footprint of the largest entry in the enum (plus a little bit more for type information), and since the size is known at compile time, there will be no heap allocation like there would with a `Box`.

```rust
enum People {
    Grandma(Grandma),
    Me(Me)
}

impl Person for People {
    fn say_hello(&self) {
        match self {
            People::Grandma(grandma) => grandma.say_hello(),
            People::Me(me) => me.say_hello()
        }
    }
}
```

With a few small modifications, it can be used just like the `as_any` example.

```rust
let mut zoo: PeopleZoo<People> = PeopleZoo {
  people: vec![]
};
zoo.add_person(People::Me(Me { name: "Bennett" }));

if let Some(People::Me(me)) = zoo.last_person() {
    println!("My name is {}.", me.name)
}
```

<p><a target="_blank" href="https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=501693ad07bda2f1e83dc31d230f8a8a">Full Example</a></p>

This gives the person who is consuming `PeopleZoo` the most flexibility without having to modify the `Person` trait. It also has the added benefit of not needed to allocate any space on the heap for the `Person`.

## Using `Box<dyn Trait>`

Now I'm not trying to say that using boxed trait objects is bad, there are some cases where they make a lot of sense.
What I am saying is that using `Box<dyn Trait>` gives other developers that may consume your struct a hard time as there's not much flexibility.

In other words, you simply shouldn't force them upon anyone if you don't have to.

That being said, you shouldn't make it hard to use them either.

### A `Box<dyn Trait>` trick

By default a `Box<dyn Trait>` doesn't implement the trait of the object it contains.
This means that trying to construct `PeopleZoo<Box<dyn Person>>` won't work out of the box and will give a type error.

Because of this, it's good practice to give a default implementation of your trait for it's boxed counterpart.
This can be done by calling `as_ref` or `as_mut` on the `Box` and calling the references relevant method.

For just a small bit of effort you can help a bunch of people that may consume your struct.

```rust
trait Person {
    fn say_hello(&self);
}

impl Person for Box<dyn Person> {
    fn say_hello(&self) {
        self.as_ref().say_hello()
    }
}

fn main() {
  let mut zoo: PeopleZoo<Box<dyn Person>> = PeopleZoo {
    people: vec![]
  };
}
```

## Conclusion

- Boxed traits should not be used to represent data inside structs as they sacrifice flexibility.

- Furthermore boxed traits make it really hard to get the original value back again and require you to provide an `as_any` implementation which pollutes the original trait.
  If a developer of a crate doesn't think about this, then you may not be able to safely get the original value back.

- Using generics you can use traits to constrain the data that developers provide, but still let them decide the specifics.
  This method allows developers to use enums to limit heap allocations and make it easier to find the original type, at the expense of greater memory consumption and slightly more code.

- Implementing your trait for it's boxed counterpart makes it easier to use boxed traits in generic parameters.
