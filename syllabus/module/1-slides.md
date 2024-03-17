---
title: Module 1 
description: "Rust Basic Concepts"
duration: 30 minutes
---

# Fundamentals

---

## Data Types

<pba-flex center>

* Memory only stores binary data
  * Anything can be represented in binary
* Program determines what the binary represents
* Basic types that are universally useful are provided by the language

</pba-flex>

---

## Basic Data Types

* Boolean
  * true, false
* Integer
  * 1, 2, 50, 99, -2
* Double / Float
  * 1.1, 5.5, 200.0001, 2.0
* Character
  * ‘A’, ‘B’, ‘c’, ‘6’, ‘$’
* String
  * “Hello”, “string”, “this is a string”, “it’s 42”

---

## What is a variable?

* Assign data to a temporary memory location
  * Allows programmer to easily work with memory
* Can be set to any value & type
* Immutable by default, but can be mutable
  * Immutable: cannot be changed
  * Mutable: can be changed

---

## Examples

```rust
let two = 2;
let hello = "hello";
let j = 'j';
let my_half = 0.5;
let mut my_name = "Bill";
let quit_program = false;
let your_half = my_half;
```
