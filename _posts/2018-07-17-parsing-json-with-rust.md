---
layout: post
title: 'Parsing JSON with Rust'
tags: rust cmdr
date: 2018-07-17

---

For [CMDRLog](https://github.com/ageiersbach/cmdr-log), a project I
started a while back to integrate with the game Elite Dangerous, I wanted to parse the "Player Journal," a file that contains a line of JSON
for just about every event that happens during the course of the game.
Since I knew how I would approach this in Node or Ruby, I thought
I would tackle it in Rust.

The main library for working with json in rust appears to be [serde](https://serde.rs/), which aims to prevent the kind of
errors that could arise from typos in code--something the compiler
might not catch. So, I added serde to a new rust project:

Cargo.toml:
```
[dependencies]
"serde" = "1.0"
"serde_derive" = "1.0"
"serde_json" = "1.0"

```

main.rs:
```
#[macro_use]
pub extern crate serde_derive;

pub extern crate serde;
pub extern crate serde_json;
```

And then I used its macros with an enum to parse out the data, since I wanted to be able to differentiate between the types of events. The result looked something like:

```
#[derive(Serialize, Deserialize, Debug)]
#[serde(tag = "event")]
enum PlayerJournalEvent {
  BuyDrones {
      timestamp: String,

      #[serde(rename(deserialize = "Count"))]
      count: u32
  },
  ...
}
```

Two cool things to note above: the first is the `#[serde(tag = "event")]` macro, which meant that when I deserialized a json string, it would
automatically know which variant it was. So an { event: "BuyDrones", ... }
would become an instance of PlayerJournalEvent::BuyDrones. Helpful, since all of the lines from the player journal had a different structure.
I couldn't have fields that were in some but not all the journal events.
That caused the program to panic.The second cool thing was the macro `#[serde(rename(deserialize = "Count"))]`, which allowed me to clean up the parsed json, which was in a strange mix of lowercase, PascalCase, and a few capitalized Snake_Case thrown in for good measure.


Still, there was a problem to the approach above: the player journal had _many_ distinct events in the game. I didn't know them all and only wanted to track a handful of them. It also meant that I had to repeat
the `timestamp` property for each event type.
So I created a struct with the only two fields that were common to all the events, and a third field called `raw_json`, where I stored a clone of the unparsed json--for later use.


```
#[derive(Serialize, Deserialize, Debug)]
struct PlayerJournalEvent {
    event: String,
    timestamp: String,

    #[serde(skip)]
    raw_json: String,
}

impl PlayerJournalEvent {
    fn new(s: String) -> PlayerJournalEvent {
        let mut pje: PlayerJournalEvent = from_str(&s).unwrap();
        pje.raw_json = s.clone();
        pje
    }
}
```
