# How do on-chain upgrades work and how to compile a runtime?

![](./assets/runtime-upgrade.png)

## Explaining the problem

Decentralization of blockchains has advantages and disadvantages, one of the major advantage is that there is no one central unit which has the whole power of over the system. From the other side disadvantage is that in the environment without one central unit upgrading is hard due to coordination so many nodes at one time. One of the well known solution of upgrading blockchains is so called `hard fork` but as I have written is hard and has many potential vectors of attack. 

## Solution to the problem

The unique and newest solution how to make upgrade of decentralized system based on Substrate framework is called `runtime-upgrade`. The solution works smoothly and in the article I will show you how you can upgrade your Substrate based chain on your machine, locally and be honest - it is quite easy

## Prerequisites

You have very basic knowledge about Rust language and Substrate framework.

## Let's start coding

During the tutorial we are going to make following steps:
1. build the simplest Substrate based blockchain
2. build runtime part where is implement logic (ewasm file)
3. run locally the blockchain
4. check version already installed runtime
5. upgrade runtime
6. verify of the upgrade

Upgrading runtime blockchains consists of following steps: 

- bump spec_version
- build wasm file
  - WASM_TARGET_DIRECTORY="$(pwd)" cargo build
- Use sudo to perform runtime

### Ad. 1: Building the simplest Substrate based blockchain

Execute `make init` and after around 10 minutes (depends on performance of your machine) you should see something like:

```
Compiling sc-finality-grandpa v0.8.0
Compiling rocksdb v0.15.0
Compiling kvdb-rocksdb v0.9.1
Compiling sc-client-db v0.8.0
Compiling sc-service v0.8.0
Compiling sc-cli v0.8.0
Compiling frame-benchmarking-cli v2.0.0
Finished dev [unoptimized + debuginfo] target(s) in 13m 01s
```

Hint: If you want build optimized version of the project execute `cargo build --release`

### Ad. 2: Build ewasm - runtime

Execute following command in order to build runtime wasm file: `WASM_TARGET_DIRECTORY="$(pwd)" cargo build`

After successfully build a new file should appear in current directory:

![](./assets/ewasm.png)

## Troubleshooting

### 1. Problem: First build could be not easy

The possible issue you can face with while building is error shown below, it means that you do not have installed proper version of Rust language.

```
   Compiling prost-derive v0.6.1
error[E0034]: multiple applicable items in scope
   --> /home/tomek/.cargo/registry/src/github.com-1ecc6299db9ec823/prost-derive-0.6.1/src/lib.rs:111:14
    |
111 |             .intersperse(quote!(|));
    |              ^^^^^^^^^^^ multiple `intersperse` found
    |
    = note: candidate #1 is defined in an impl of the trait `Iterator` for the type `Map<I, F>`
    = note: candidate #2 is defined in an impl of the trait `Itertools` for the type `T`
help: disambiguate the associated function for candidate #1
    |
107 ~         let tags = Iterator::intersperse(field
108 +             .tags()
109 +             .into_iter()
110 +             .map(|tag| quote!(#tag)), {
111 +         let mut _s = $crate::__private::TokenStream::new();
112 +         $crate::quote_each_token!(_s $($tt)*);
  ...
help: disambiguate the associated function for candidate #2
    |
107 ~         let tags = Itertools::intersperse(field
108 +             .tags()
109 +             .into_iter()
110 +             .map(|tag| quote!(#tag)), {
111 +         let mut _s = $crate::__private::TokenStream::new();
112 +         $crate::quote_each_token!(_s $($tt)*);
  ...

For more information about this error, try `rustc --explain E0034`.
   Compiling asn1_der_derive v0.1.2
error: could not compile `prost-derive` due to previous error
warning: build failed, waiting for other jobs to finish...
error: build failed
make: *** [Makefile:11: build-full] Error 101
```

### Solution - configure and make default proper version of Rust language on your machine

You need to install and make default `rustup update nightly-2020-08-23` Rust language

1. Installation

`rustup update nightly-2020-08-23`

2. Make default version

`rustup default nightly-2020-08-23-x86_64-unknown-linux-gnu`

3. Checking of default version

`rustup show` and in the output you should see following output:

``
active toolchain
----------------

nightly-2020-08-23-x86_64-unknown-linux-gnu (default)
rustc 1.47.0-nightly (663d2f5cd 2020-08-22)
``

Twitter's thread to the most possible issues: https://twitter.com/tomaszwaszczyk/status/1343512637909458944

<!-- https://www.youtube.com/watch?v=MQgDV37JrIY

https://substrate.dev/docs/en/tutorials/forkless-upgrade/

https://www.notion.so/Sample-Tutorial-Structure-667ac2aad36f4f94a3ebaca053180b2d -->
