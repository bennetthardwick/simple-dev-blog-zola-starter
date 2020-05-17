+++
title = "How to complete the Advent of Code 2018 challenges with Rust"
description = "An introduction to the Rust programming language through an Advent of Code 2018 challenge."
alises = ["/blog/2018-12-07-introduction-to-rust-through-advent-of-code-2018/"]
[taxonomies]
tags = [ "rust" ]
+++

Over the past month or so I've been learning the Rust programming language. When I heard that Advent of Code was starting up again, I figured it would be a great opportunity to put some of the skills I'd learned to the test.

Since the [first challange](https://adventofcode.com/2018/day/1) was just begging for an easy solution, I decided to start with Rust on the second day.
The challenge was this:

> Given a list of strings, calculate the product of the number of strings with _exactly two of any letter_ and strings with _exactly three of any letter_
> You can read more about the brief [here](https://adventofcode.com/2018/day/2).

Whenever something has to do with counting, you can always assume the fastest solution will involve the use of a hash table. With this in mind, we can create some pseudo-code.

```

```

1. Loop through each string in the file.

2. For each string, loop through the characters.

3. For each character, increment a value in a hash-table using the character as the key, where the value is the frequency of that character

4. Count all the twos and threes in the map

5. Return the product

```
number_of_twos = 0
number_of_threes = 0
for each string in input
  for each character in string
    increment map[character]
  number_of_twos = count where map[entry] = 2
  number_of_threes = count where map[entry] = 2
result = number_of_twos * number_of_threes
```

Now that we've got some pseudo-code down, we can go ahead and implement it. Create a file called `solution.rs` and open it in your favourite text editor.

```

```

Like in most languages, the best place to start is the `main` method - the entry-point to your software.

First, start by declaring a new variable called `file`. I've given this variable the `mut` or mutable attribute, as some of the following methods will need to change it. It's important to note that Rust needs to be explicity told when a variable isn't read-only - else you'll get an error during compilation.

Next, using the `File` struct, call the `open` method to load in the input data. Then, follow it up with a call to the `unwrap` method.

This was one of the first major roadblocks for me when I started learning Rust.
By diving into the docs, you can see that the `open` method on the `File` struct returns a `Result` of type `File`.
The `Result` type has two states, `Ok` and `Err`. If it's in the `Ok` state, it contains the `File`, if it's in the `Err` state, it contains an `Error`.
When the `unwrap` method is called, it returns the value of the `Result` if it's in the `Ok` state, and exits the program / panics if it's in the `Err` state.

Typically you would want to handle the case where Rust cannot find the file, but for the sake of simplicity, I'm going to keep using `unwrap`.

```rust
use std::fs::File;
use std::io::Read;
use std::collections::HashMap;
fn main () {
  let mut file;
  file = File::open("./input.txt").unwrap();
```

Now that we've got our input `File` loaded, we need a place to put it. We can use the `String::new` method to create a new `String` object on the heap where we can store the contents of the file.

Next, we pass a `&mut` (mutable reference) to the string previously created to the `read_to_string` method which populates the string with the file.
If you're unfamiliar with references / pointers, you can read more about them [here](http://www.cplusplus.com/doc/tutorial/pointers/).

```rust
  let input = String::new();
  file.read_to_string(&mut input).unwrap();
```

Now that we've prepared out input string, we can pass read-only references to our part-one and part-two functions. I'm only covering the part-one solution in this article, but you can view the [full code on my Github](https://github.com/bennetthardwick/advent-of-code-2018/blob/master/solutions/2/solution.rs).

```rust
  part_one(&input);
  part_two(&input);
}
```

## part_one

The `part_one` method is quite simple. It loops through all the ids in the input file and determines whether they contain doubles and triples or not.

```

```

First, declare two mutable variables for holding the number of doubles and triples respectively.

Next, loop through each line in the file. This is done by splitting the original string on each new line (`\n`), which creates an array of each line.

Now we can use the `count_doubles_and_triples` method to determine whether the string has a pair or trio of characters in it (or both).

```rust
fn part_one(input: &String) {
    let mut twos = 0;
    let mut threes = 0;
    for id in input.split('\n') {
        let (add_twos, add_threes) = count_doubles_and_triples(id);
```

If it has a exactly two of a character - increment the `twos` count. Also, if it has exactly three of a character - increment the `threes` count.

```rust
        if add_twos {
            twos z+= 1;
        }
        if add_threes {
            threes += 1;
        }
    }
```

With these counts calculated, go a head an multiply them together to find their product.

```rust
    println!("\nPart 1!");
    println!("{}", twos * threes);
}
```

## count_doubles_and_triples

This method is a little bit more complicated than the previous one. Basically, it's purpose is to determine whether the provided string contains exactly two or three of a certain character.

Firstly, initialise three variables. A boolean to depict whether there are exactly two of a character, another to depict whether there are exactly three of a character, and a hash map for storing the character - count information.

```rust
fn count_doubles_and_triples(id: &str) -> (bool, bool) {
    let mut twos = false;
    let mut threes = false;
    let mut character_counts: HashMap<char, u32> = HashMap::new();
```

Now iterate of the characters of the string, and add them to the hash map. We do this by getting a reference to an entry for a character in the hashmap, and incrementing it's value.

With the character frequency counted, loop over the entries in the `HashMap`. If the value of the entry is equal to 2, set `twos` to true. If the value is equal three, set `threes` to true.

For the sake of performance, I've also added in a case that will exit the loop when both have been set to true.

```rust
    for character in id.chars() {
        let entry = character_counts.entry(character).or_insert(0);
        *entry += 1;
    }
    for (_, &value) in character_counts.iter() {
        if value == 2 {
            twos = true;
        } else if value == 3 {
            threes = true;
        }
        if twos && threes { break; }
    }
```

Finally, return the results. In this case, instead of explicitly writing the `return` keyword, we can use Rust's implicit return to save some _precious_ characters.

```rust
    (twos, threes)
}
```

That pretty much wraps up this challenge. You can sign up for the Advent of Code by following [this link](https://adventofcode.com/). Even if you don't get around to completing much (like me), just programming in an unfamiliar language can be a lot of fun.

If you want to learn more about Rust, or didn't understand anything at all, there is [a great free book](https://doc.rust-lang.org/stable/book/) available online called "The Rust Programming Language".
It's pretty much my go-to resource whenever I'm having trouble.

You can find the source code used in this tutorial [right here](https://github.com/bennetthardwick/advent-of-code-2018/blob/master/solutions/2/solution.rs)!
