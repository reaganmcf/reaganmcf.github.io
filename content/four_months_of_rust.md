+++
title="4 months of Rust - what I've learned"
description="I started learning and using Rust just under 4 months ago. After reading a few books, watching hundreds of videos, and many projects later, here is what I have learned."
date=2019-07-16

[taxonomies]
tags = ["rust", "review"]
categories = ["programming"]
+++

I started learning and using Rust just under 4 months ago. After reading a few books, watching hundreds of videos, and many projects later, here is what I have learned.

## `cargo` is 10/10

The utility a powerful package manager and build tool gives you is hard to put into words. Not only is it better than any other package manager that exists for low level programming languages, it is even better than `pip` and `npm` in my opinion.
- I've never once had a unknown error while running any `cargo` command.
- The crate ecosystem is very robust with high quality projects. And since docstrings get automatically hosted on [docs.rs](https://docs.rs), they aren't just packages that are thrown together. Maintainers take pride in their crates and **very rarely** is a crate undocumented.
- `cargo`'s toolchain management system with `rustup` is about as good as you could ever ask for. You can easily compile for dozens of systems interchangeably, and even specify your own toolchain in `json` format.
- `cargo`'s built in test suite is a game changer for promoting quality software.

## I've been writing unsafe code my whole life without realizing

After learning about the `unsafe` keyword, it really puts in perspective just how unsafe all my non Rust projects have been in the past, whether they were in C, Java, C++, etc.

For example, when I took OS I remember getting so frustrated with C when working on my [partner](https://github.com/evanwire) and I's [`ext2` file system implementation](https://github.com/reaganmcf/tiny-file-system).
We spent probably 50 hours on that project, 20 of which were debugging memory issues. Large scale C projects can fall apart quick when UB appears in sections you thought was correct. 
You can easily become stuck in a large section of code that is very difficult to debug.
There is so much that can go wrong, and when all you see is `Segmentation fault` in the output you want to tear your hair out.

However after using Rust, C was not _really_ to blame - it was us! Pointers **are just inherently dangerous**, especially in file system implementations where you are casting pointers many times to various types and reading / writing directly into memory from files. All of these operations can go south very quickly, and Rust forces you to acknowledge that.

Now when I go back to C, I find my self being much more careful with how I deal with pointers, explicitly adding comments describing why this operation is safe, just like I would do in Rust.

## Helpful error messages make programming fun

Anyone who has used Rust for more than 5 minutes has learned first hand just how great the error messages are. Instead of constantly fighting the compiler just to end up with a program that has plenty of runtime issues, `Rustc` holds your hand,
giving you very helpful tips when things go wrong. These tips usually include
  1. What exactly the error is
  2. A very descriptive breakdown of the error, using the actual contents of your file and highlighting the lines in question
  3. A suggested fix that is usually correct about 90% of the time

No more are the days of `Segmentation fault` with no real guidance on how to fix it besides a line number. I once heard someone describe the compiler as a pair programmer always by your side, and I don't think I can put it in better words.

As an example, let's take a look at a dangling pointer in C and the error message we get.
```c
int main()
{
  int* my_ptr;
  some_function(my_ptr);
  printf("my pointer is: %d", *my_ptr);
}

void some_function(int* pointer) {
  int* local = 5;
  pointer = &local;
}
```

Running with `gcc -fsanitize=address -O3` we get
```
ASAN:DEADLYSIGNAL
=================================================================
==46979==ERROR: AddressSanitizer: SEGV on unknown address 0x000000000000 (pc 0x564cc67208a4 bp 0x564cc6720a20 sp 0x7ffdd2317750 T0)
==46979==The signal is caused by a READ memory access.
==46979==Hint: address points to the zero page.
    #0 0x564cc67208a3 in main (/common/home/rpm141/a.out+0x8a3)
    #1 0x7fd05438bbf6 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x21bf6)
    #2 0x564cc6720929 in _start (/common/home/rpm141/a.out+0x929)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: SEGV (/common/home/rpm141/a.out+0x8a3) in main
==46979==ABORTING
```

Now I don't know about you, but this is _kind of gibberish_. Can you imagine being a beginner again and trying to debug this? Can you imagine having a 50k line C project and trying to debug this?

Let's try doing the same thing in Rust.

```rust
fn main() {
  let pointer: &i32;
  some_function(pointer);
  print!("my pointer is: {}", pointer);
}

fn some_function(mut pointer: i32) {
  let local: i32 = 5;
  pointer = &local;
}
```

Running `cargo check` we get
```rust
error[E0597]: `local` does not live long enough
  --> src/main.rs:9:15
   |
7  | fn some_function(mut pointer: &i32) {
   |                               - let's call the lifetime of this reference `'1`
8  |     let local: i32 = 5;
9  |     pointer = &local;
   |     ----------^^^^^^
   |     |         |
   |     |         borrowed value does not live long enough
   |     assignment requires that `local` is borrowed for `'1`
10 | }
   | - `local` dropped here while still borrowed
```

Look at that error message! It's not just address dumps, it's in _english_. It's telling us exactly what the issue is, and what the fix is. It's makes programming so much more fun when you aren't stuck in these hard to debug issues with no context.

## The dev tools are top notch
I've already talked about how great `cargo` is, but the fantastic devtool suite that comes with Rust doesn't stop with just `cargo`.

Rust has enterprise quality plugins for all IDEs, vim, emacs, etc.

Rust analyzer is faster and more accurate than any other Intellisense engine I've ever used (including typescript's, which is arguably the best on the market).

Clippy is a cli tool that gives you refactoring suggestions for your code, and is much smarter compared to the other refactoring tools I've used before (like ReSharper). 
It gives new programmers confidence in the fact that their code is not only correct, but also easy for others to read. 
This lowers the barrier of entry for contributing to open source projects significantly.

Rustfmt is the code formatting tool used by practically every single Rust project. It makes every codebase more readable because you are not having to adjust mentally to a new formatting style when you switch projects.

## The community is amazing

For my last point, I have to point out the Rust community.

The Rust community is like no other as cliché as it sounds. There are so many fantastic resources, both free and paid, for all levels of Rust programmers. Here is a list of great resources I have used:

#### Beginner

- [The Rust Programming Language](https://doc.rust-lang.org/book/) is the official book for Rust, and is a much lighter read than you would imagine. Definitely worth it.
- [A Gentle Introduction To Rust](https://stevedonovan.github.io/rust-gentle-intro/readme.html) is another great beginner tutorial to get your feet wet.
- [Rust Discord](https://discord.com/invite/rust-lang) has channels where beginners can ask any questions, or learn more on how to contribute to the project.

#### Intermediate

- [Jon Gjengset's YouTube channel](https://www.youtube.com/c/jongjengset) has been my favorite resource. He has great tutorials on the harder parts of Rust to wrap your head around like Interior Mutability, Iterators, Macros, etc.
- [JT's Creating a line editor in Rust series](https://www.youtube.com/user/giard321) is great to see how to build a project from the ground up.
- [Rust for Rustaceans](https://nostarch.com/rust-rustaceans) is fantastic as well.
- [Arzg's Make a Language blog posts](https://arzg.github.io/lang/) is great for learning advanced Rust and programming languages.
- [Phil Oppermann's "Writing an OS in Rust" blog posts](https://os.phil-opp.com) is another great resource for learning advanced Rust and operating systems.
