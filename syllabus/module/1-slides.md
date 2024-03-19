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

---


## What are functions?

* A way to encapsulate program functionality
* Optionally accept data
* Optionally return data
* Utilized for code organization
  * Also makes code easier to read

---

## Anatomy of a function

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

---

## Using a function

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

let x = add(1, 1);
let y = add(3, 0);
let z = add(x, 1);
```

---

## The println macro

* Macros expand into additional code
* println “Prints” (displays) information to the terminal
* Useful for debugging

```rust
let life = 42;

println!("hello");
println!("{:?}", life);
println!("{:?}{:?}", life, life);
println!("the meaning is {:?}", life);

println!("{life:?}");
println!("{life}");
```

---

* Macros use an exclamation point to call/invoke
* Generate additional Rust code
* Data can be printed using println!:
  * {:?}
  * {varname:?}

---

## Execution Flow

* Code executed line-by-line
* Actions are performed & control flow may change
  * Specific conditions can change control flow
    * “if”
    * “else”
    * “else if”

---

## Example – if..else

```rust
let a = 99;
if a > 99 {
    println!("Big number");
} else {
    println!("Small number");
}
```

---

## Example – Nested if..else

```rust
let a = 99;
if a > 99 {
    if a > 200 {
        println!("Huge number");
    } else {
        println!("Big number");
    }
} else {
    println!("Small number");
}
```

---

## Example – if..else if..else

```rust
let a = 99;
if a > 200 {
    println!("Huge number");
} else if a > 99 {
    println!("Big number");
} else {
    println!("Small number");
}

// This will not work
if a > 99 {
    println!("Big number");
} else if a > 200 {
    println!("Huge number");
} else {
    println!("Small number");
}
```

---

## Repetition

* Called “looping” or “iteration”
* Multiple types of loops
  * “loop” - infinite loop
  * “while” – conditional loop

---

## Loop

```rust
let mut a = 0;
loop {
    if a == 5 {
        break;
    }
    println!("{:?}", a);
    a = a + 1;
}

```

---

## While loop

```rust
let mut a = 0;
while a != 5 {
    println!("{:?}", a);
    a = a + 1;
}
```

---

## Match

* Add logic to program
* Similar to if..else
* Exhaustive
  * All options must be accounted for

---

## Example with boolean

```rust
fn main() {
    let some_bool = true;
    match some_bool {
        true => println!("it's true"),
        false => println!("it's false"),
    }
}

```

---

## Example with int

```rust
fn main() {
    let some_int = 3;
    match some_int {
        1 => println!("it's 1"),
        2 => println!("it's 2"),
        3 => println!("it's 3"),
        _ => println!("it's something else"),
    }
}

```

---

## match vs else..if

* match will be checked by the compiler
  * If a new possibility is added, you will be notified when this occurs
* else..if is not checked by the compiler
  * If a new possibility is added, your code may contain a bug

---

## Enumeration

* Data that can be one of multiple different possibilities
  * Each possibility is called a “variant”
* Provides information about your program to the compiler
  * More robust programs

---

## Example

```rust
enum Direction {
    Up,
    Down,
    Left,
    Right,
}

fn which_way(go: Direction) -> &'static str {
    match go {
        Direction::Up => "up",
        Direction::Down => "down",
        Direction::Left => "left",
        Direction::Right => "right",
    }
}
```

---

## Structure

* A type that contains multiple pieces of data
  * All or nothing – cannot have some pieces of data and not others
* Each piece of data is called a “field”
* Makes working with data easier
  * Similar data can be grouped together

---

## Example

```rust
struct ShippingBox {
    depth: i32,
    width: i32,
    height: i32,
}

fn main() {
    let my_box = ShippingBox {
        depth: 3,
        width: 2,
        height: 5,
    };
    
    let tall = my_box.height;
    println!("the box is {:?} units tall", tall);
}
```

---

## Tuples

* A type of “record”
* Store data anonymously
  * No need to name fields
* Useful to return pairs of data from functions
* Can be “destructured” easily into variables

---

## Example

```rust
enum Access {
    Full,
}

fn one_two_three() -> (i32, i32, i32) {
    (1, 2, 3)
}

fn main() {
    let numbers = one_two_three();
    let (x, y, z) = one_two_three();
    println!("{:?}, {:?}", x, numbers.0); // 1
    println!("{:?}, {:?}", y, numbers.1); // 2
    println!("{:?}, {:?}", z, numbers.2); // 3

    let (employee, access) = ("Jake", Access::Full);
}
```

---

## Expressions

* Rust is an expression-based language
  * Most things are evaluated and return some value
* Expression values coalesce to a single point
  * Can be used for nesting logic

---

## Example

```rust
let my_num = 3;
let is_lt_5 = if my_num < 5 {
    true
} else {
    false
};

let is_lt_5 = my_num < 5;
```

---

## Example

```rust
let my_num = 3;
let message = match my_num {
    1 => "hello",
    _ => "goodbye"
};
```

---

## Example

```rust
enum Menu {
    Burger,
    Fries,
    Drink,
}

let paid = true;
let item = Menu::Drink;
let drink_type = "water";
let order_placed = match item {
    Menu::Drink => {
        if drink_type == "water" {
            true
        } else {
            false
        }
    },
    _ => true,
};

```

---

## Basic memory refresh

* Memory is stored using binary
  * Bits: 0 or 1
* Computer optimized for bytes
  * 1 byte == 8 contiguous bits
* Fully contiguous

---

## Addresses

* All data in memory has an “address”
  * Used to locate data
  * Always the same – only data changes
* Usually don’t utilize addresses directly
  * Variables handle most of the work

---

## Offsets

* Items can be located at an address using an “offset”
* Offsets begin at 0
* Represent the number of bytes away from the original address
  * Normally deal with indexes instead

---

# Ownership

---

## Managing memory

* Programs must track memory
  * If they fail to do so, a “leak” occurs
* Rust utilizes an “ownership” model to manage memory
  * The “owner” of memory is responsible for cleaning up the memory
* Memory can either be “moved” or “borrowed”

---

## Example - Move

```rust
enum Light {
    Bright,
    Dull,
}

fn display_light(light: Light) {
    match light {
        Light::Bright => println!("bright"),
        Light::Dull => println!("dull"),
    }
}

fn main() {
    let dull = Light::Dull;
    display_light(dull);
    display_light(dull);
}

```

---

## Example - Borrow

```rust
enum Light {
    Bright,
    Dull,
}

fn display_light(light: &Light) {
    match light {
        Light::Bright => println!("bright"),
        Light::Dull => println!("dull"),
    }
}

fn main() {
    let dull = Light::Dull;
    display_light(&dull);
    display_light(&dull);
}

```

---

## Vector

* Multiple pieces of data
  * Must be the same type
* Used for lists of information
* Can add, remove, and traverse the entries

---

## Example

```rust
let my_numbers = vec![1, 2, 3];

let mut my_numbers = Vec::new();
my_numbers.push(1);
my_numbers.push(2);
my_numbers.push(3);
my_numbers.pop();
let length = my_numbers.len(); // this is 2

let two = my_numbers[1];
```

---

## Example

```rust
let my_numbers = vec![1, 2, 3];

for num in my_numbers {
    println!("{:?}", num);
}
```

---

## String and &str

* Two commonly used types of strings
  * String - owned
  * &str – borrowed String slice
* Must use an owned String to store in a struct
* Use &str when passing to a function

---

## Example – Pass to function

```rust
fn print_it(data: &str) {
    println!("{:?}", data);
}

fn main() {
    print_it("a string slice");
    let owned_string = "owned string".to_owned();
    let another_owned = String::from("another");
    print_it(&owned_string);
    print_it(&another_owned);
}
```

---

## Example – Will not work

```rust
struct Employee {
    name: &'static str,
}

fn main() {
    let emp_name = "Jayson";
    let emp = Employee {
        name: emp_name,
    };
}
```

---

## Example – Works!

```rust
struct Employee {
    name: String,
}

fn main() {
    let emp_name = "Jayson".to_owned();
    let emp = Employee {
        name: emp_name
    };
}

```

---

## Type Annotations

* Required for function signatures
* Types are usually inferred
* Can also be specified in code
  * Explicit type annotations

---

## Example - Basic

```rust
fn print_many(msg: &str, count: i32) { 
    // Presumably the body of the function would print the message many times
    // but since it's not shown in the image, it's left empty here.
}

enum Mouse {
    LeftClick,
    RightClick,
    MiddleClick,
}

fn main() {
    let num: i32 = 15;
    let a: char = 'a';
    let left_click: Mouse = Mouse::LeftClick;
}
```

---

## Example - Generics

```rust
enum Mouse {
    LeftClick,
    RightClick,
    MiddleClick,
}

let numbers: Vec<i32> = vec![1, 2, 3];
let letters: Vec<char> = vec!['a', 'b'];
let clicks: Vec<Mouse> = vec![
    Mouse::LeftClick,
    Mouse::LeftClick,
    Mouse::RightClick,
];
```

---

## Enums

* enum is a type that can represent one item at a time
  * Each item is called a variant
* enum is not limited to just plain variants
  * Each variant can optionally contain additional data

---

## Example

```rust
enum Mouse {
    LeftClick,
    RightClick,
    MiddleClick,
    Scroll(i32),
    Move(i32, i32),
}
```

---

## Example

```rust
enum PromoDiscount {
    NewUser,
    Holiday(String),
}

enum Discount {
    Percent(f64),
    Flat(i32),
    Promo(PromoDiscount),
    Custom(String),
}
```

---

## Option

* A type that may be one of two things
  * Some data of a specified type
  * Nothing
* Used in scenarios where data may not be required or is unavailable
  * Unable to find something
  * Ran out of items in a list
  * Form field not filled out

---

## Definition

```rust
enum Option<T> {
    Some(T),
    None,
}

```

## Example

```rust
struct Customer {
    age: Option<i32>,
    email: String,
}

let mark = Customer {
    age: Some(22),
    email: "mark@example.com".to_owned(),
};

let becky = Customer {
    age: None,
    email: "becky@example.com".to_owned(),
};

match becky.age {
    Some(age) => println!("customer is {} years old", age),
    None => println!("customer age not provided"),
}
```

---

## Example

```rust
struct GroceryItem {
    name: String,
    qty: i32,
}

fn find_quantity(name: &str) -> Option<i32> {
    let groceries = vec![
        GroceryItem { name: "bananas".to_owned(), qty: 4 },
        GroceryItem { name: "eggs".to_owned(), qty: 12 },
        GroceryItem { name: "bread".to_owned(), qty: 1 },
    ];

    for item in groceries {
        if item.name == name {
            return Some(item.qty);
        }
    }

    None
}
```

---

## Result

* A data type that contains one of two types of data:
  * “Successful” data
  * “Error” data
* Used in scenarios where an action needs to be taken, but has the possibility of failure
  * Copying a file
  * Connecting to a website

---

## Definition

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

---

## Example

```rust
struct SoundData {
    // assuming SoundData has a field called `name`
    name: String,
}

impl SoundData {
    fn new(name: &str) -> SoundData {
        SoundData {
            name: name.to_owned(),
        }
    }
}

fn get_sound(name: &str) -> Result<SoundData, String> {
    if name == "alert" {
        Ok(SoundData::new("alert"))
    } else {
        Err("unable to find sound data".to_owned())
    }
}

fn main() {
    let sound = get_sound("alert");
    match sound {
        Ok(_) => println!("sound data located"),
        Err(e) => println!("error: {}", e),
    }
}
```

---

## Hashmap

* Collection that stores data as key-value pairs
  * Data is located using the “key”
  * The data is the “value”
* Similar to definitions in a dictionary
* Very fast to retrieve data using the key

---

## Example: find data

```rust
use std::collections::HashMap;

fn main() {
    let mut people = HashMap::new();
    people.insert("Susan", 21);
    people.insert("Ed", 13);
    people.insert("Will", 14);
    people.insert("Cathy", 22);
    people.remove("Susan");

    match people.get("Ed") {
        Some(age) => println!("age = {:?}", age),
        None => println!("not found"),
    }
}

```

---

## Example: iterate

```rust
for (person, age) in people.iter() {
    println!("person = {:?}, age = {:?}", person, age);
}

for person in people.keys() {
    println!("person = {:?}", person);
}

for age in people.values() {
    println!("age = {:?}", age);
}

```
