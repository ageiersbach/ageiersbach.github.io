---
layout: post
title:  "Playing with strings in Rust"
tags: viola rust
date: 2018-07-08
---

A few days ago, I set aside some time for learning [Rust](https://www.rust-lang.org/en-US/).
I do not work with compiled languages often, but I was not totally unfamiliar with them,
having used Java in a lot of my early CS courses.
I chose to tackle String manipulations, because I figured this would be a useful mechanism
for seeing how Rust works. The String type came up multiple times in Rust's [tutorial book](https://doc.rust-lang.org/stable/book/second-edition/ch00-00-introduction.html).
Rust treats strings differently than other types like chars, numbers, and booleans,
which can be added to the stack because their size is known at compile-time.
Strings can be variable-length, however, so they are added to the heap, and the
program can refer to them by reference. There's a [complicated system of
_ownership_](https://doc.rust-lang.org/stable/book/second-edition/ch04-00-understanding-ownership.html) that applies to anything that is kept on the heap, which means that
you can't do the following:

```
let string1 = String::from("what's in here is a _str_, but I turn it into String");
let string2 = string1;
```

Because of how _ownership_ works in Rust, string1 passes to string2. Any
future reference to string1 will result in a compiler error. Even sending
the string to a function will result in its ownership being handed over!

So the first problem I worked on in [viola](https://github.com/ageiersbach/viola)
was breaking a string into a vector (`Vec<String>`) of _words_.

```
let v: Vec<String> = ["one", "two", "three"].to_vec();
let instrument = Instrument::new(s);
assert_eq!(instrument.words(), v);
```

This worked well as long as there was no punctuation. If I parsed a string like
"one, two, three", it would include the commas in the words: `["one,", "two,", "three"]`. Ick! So I turned to
the matcher to parse out punctuation, and created an _enum_ to create categories
for characters.

```
pub enum CharType {
    Space,
    Punctuation(PunctuationType),
    Alphanumeric,
    Other,
}

impl CharType {
    pub fn from_char(c: char) -> CharType {
        let c_int = c as u32;
        match c_int {
            0..=32 => CharType::Space, // includes space, tab, carriage returns but not DEL (127)
            33..=47 | 58..=64 | 91..=96 | 123..=126 => {
                CharType::Punctuation(PunctuationType::from_int(c_int))
            },
            48..=57 => CharType::Alphanumeric, // 0..9
            65..=90 => CharType::Alphanumeric, // 'A' .. 'Z'
            97..=122 => CharType::Alphanumeric, // 'a' .. 'z'
            _ => CharType::Other // emojis and other non-latin chars
        }
    }
}
pub fn words(&self) -> Vec<String> {
   let mut words: Vec<String> = Vec::new();
   let mut s: String = String::new();
   for c in self.chars() {
       match CharType::from_char(c) {
           CharType::Space => {
               words.push(s.clone());
               s.clear();
           },
           CharType::Alphanumeric |
           CharType::Punctuation(PunctuationType::Apostrophe) |
           CharType::Punctuation(PunctuationType::Symbol) |
           CharType::Other => {
               s.push(c);
           }
           CharType::Punctuation(_) => {},
       }
   }
   words.push(s);
   words
}
```

With the above code, I was able to parse a string such that chars were added to
the mutable type string (the word in progress), punctuation was ignored and
spaces meant the word was ready to be added to the vector.

All of this was made easier by the built-in testing features in Rust, which
let me write unit tests right away without any further setup. At first I was
a bit put off by having the unit tests in the same file as the working code,
but after a while it seemed like a natural choice. What better way to find out
how something is supposed to work than by reading through the unit tests?

For the full diff of the initial project, see: **[viola/compare/3cd9a9...479e13](https://github.com/ageiersbach/viola/compare/3cd9a97573a26f761ef50f5339c76fb343b3af1a...479e13668b6c8608869122976a45d8e89872e694)**
