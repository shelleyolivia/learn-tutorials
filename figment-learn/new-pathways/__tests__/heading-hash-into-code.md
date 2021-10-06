# The test

here we test this usage of `#` inside code-block
This should work

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
```

# The raw file

{% hint style="info" %}
[click here](https://raw.githubusercontent.com/figment-networks/datahub-learn/master/figment-learn/new-pathways/__tests__/heading-hash-into-code.md)
{% endhint %}
