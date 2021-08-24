# 

## Test Rust marco syntax

below a snippet of `rust` using marco

```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u8,
}

fn main() {
    // Create struct with field init shorthand
    let name = String::from("Peter");
    let age = 27;
    let peter = Person { name, age };

    // Print debug struct
    println!("{:?}", peter);
}
'''
