# How do on-chain upgrades work and how to compile a runtime?

![](./assets/runtime-upgrade.png)

## Explaining the problem

Decentralization of blockchains has advantages and disadvantages, one of the major advantage is that there is no one central unit which has the whole power over a system. From the other side disadvantage is that in the environment without one central unit upgrading is hard due to coordination so many nodes at one time. One of the well known solution of upgrading blockchains is so called `hard fork` but as I have written has many potential vectors of attack and is hard to coordinate.

## Solution to the problem

The unique and newest solution how to make upgrade of decentralized system based on Substrate framework is called `runtime-upgrade`. The solution works smoothly and in the article I will show you how you can upgrade your Substrate based chain on your machine, locally and be honest - it is quite easy. This is how adaptability is implemented in Substrate.

## Prerequisites

You have very basic knowledge about Rust language and Substrate framework.

## Let's start coding

During the tutorial we are going to make following steps:
1. build the simplest Substrate based blockchain
2. run locally the blockchain
3. add a new feature to runtime
4. build WASM file
5. verify of the upgrade - check version

Upgrading runtime state transition function consists of following steps: 

- bump spec_version
- build wasm file
  - `WASM_TARGET_DIRECTORY="$(pwd)" cargo build`
- Use sudo to perform runtime

### Ad. 1: Building the simplest Substrate based blockchain

Checkout https://github.com/TomaszWaszczyk/substrate-runtime-upgrade-tutorial (master branch) then execute `make init` and after around 10 minutes (depends on performance of your machine) you should see something like:

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

### Ad. 2: Run locally blockchain

Execute `make run` command:

```
Running `target/debug/node-template --dev -lruntime=debug`
Sep 29 18:18:09.106  WARN Running in --dev mode, RPC CORS has been disabled.    
Sep 29 18:18:09.106  INFO Substrate Node    
Sep 29 18:18:09.106  INFO ✌️  version 2.0.0-73d7748-x86_64-linux-gnu    
Sep 29 18:18:09.106  INFO ❤️  by Substrate DevHub <https://github.com/substrate-developer-hub>, 2017-2021    
Sep 29 18:18:09.106  INFO 📋 Chain specification: Development    
Sep 29 18:18:09.106  INFO 🏷  Node name: frantic-insect-7496    
Sep 29 18:18:09.106  INFO 👤 Role: AUTHORITY    
Sep 29 18:18:09.107  INFO 💾 Database: RocksDb at /home/tomek/.local/share/node-template/chains/dev/db    
Sep 29 18:18:09.107  INFO ⛓  Native runtime: node-template-1 (node-template-1.tx1.au1)    
Sep 29 18:18:10.070  INFO 🔨 Initializing Genesis block/state (state: 0x0f2a…c2cc, header-hash: 0x1e1d…f017)    
Sep 29 18:18:10.073  INFO 👴 Loading GRANDPA authority set from genesis on what appears to be first startup.    
Sep 29 18:18:10.294  INFO ⏱  Loaded block-time = 2000 milliseconds from genesis on first-launch    
Sep 29 18:18:10.295  WARN Using default protocol ID "sup" because none is configured in the chain specs    
Sep 29 18:18:10.296  INFO 🏷  Local node identity is: 12D3KooWNo1348XGxpiA9uFJn4GJdptp7NcX72pFXsGZXghPKLYa (legacy representation: 12D3KooWNo1348XGxpiA9uFJn4GJdptp7NcX72pFXsGZXghPKLYa)    
Sep 29 18:18:10.618  INFO 📦 Highest known block at #0    
Sep 29 18:18:10.620  INFO 〽️ Prometheus server started at 127.0.0.1:9615    
Sep 29 18:18:10.624  INFO Listening for new connections on 127.0.0.1:9944.    
Sep 29 18:18:12.249  INFO 🙌 Starting consensus session on top of parent 0x1e1dd820c46a22dbd297afd5a62e73511208bafe64c6dc9e6a171562c443f017    
Sep 29 18:18:12.397  INFO 🎁 Prepared block for proposing at 1 [hash: 0x3b7c2d4f453358a70095cc0ad55a5f157a5be3d145c42213ea88c419b48966bf; parent_hash: 0x1e1d…f017; extrinsics (1): [0xff0f…0d06]]    
Sep 29 18:18:12.515  INFO 🔖 Pre-sealed block for proposal at 1. Hash now 0xc61a8f6ea035c0fd2e8acef986a60ec92abbaaa1f30c4781603d4ec83591b216, previously 0x3b7c2d4f453358a70095cc0ad55a5f157a5be3d145c42213ea88c419b48966bf.    
Sep 29 18:18:12.517  INFO ✨ Imported #1 (0xc61a…b216)    
Sep 29 18:18:14.123  INFO 🙌 Starting consensus session on top of parent 0xc61a8f6ea035c0fd2e8acef986a60ec92abbaaa1f30c4781603d4ec83591b216
```

Access frontend of running locally node via: `https://polkadot.js.org/apps/#/extrinsics?rpc=ws://127.0.0.1:9944`

![](./assets/before-upgrade.png)

We can see that kitties pallet has only `create()` function, it is out current state transition - runtime. What we are going now is to show how adaptable Substrate based blockchains are, we add new feature (breed function)  

## Ad 3: Add a new feature to runtime

Before upgrade `spec_version` was equal to `1`, now while making upgrade we have incremented the field into `2` like below:

```rust
pub const VERSION: RuntimeVersion = RuntimeVersion {
	spec_name: create_runtime_str!("node-template"),
	impl_name: create_runtime_str!("node-template"),
	authoring_version: 1,
	spec_version: 2, // incremented version
	impl_version: 1,
	apis: RUNTIME_API_VERSIONS,
	transaction_version: 1,
};
```

Our upgrade will add a new function to `kitties` pallet called `breed`, here is the implementation:

```rust
/// Breed kitties
#[weight = 1000]
pub fn breed(origin, kitty_id_1: u32, kitty_id_2: u32) {
  let sender = ensure_signed(origin)?;
  let kitty1 = Self::kitties(&sender, kitty_id_1).ok_or(Error::<T>::InvalidKittyId)?;
  let kitty2 = Self::kitties(&sender, kitty_id_2).ok_or(Error::<T>::InvalidKittyId)?;

  ensure!(kitty1.gender() != kitty2.gender(), Error::<T>::SameGender);
    
  let kitty_id = Self::get_next_kitty_id()?;

  let kitty1_dna = kitty1.0;
  let kitty2_dna = kitty2.0;

  let selector = Self::random_value(&sender);
  let mut new_dna = [0u8; 16];

  // Combine parents and selector to create new kitty
  for i in 0..kitty1_dna.len() {
    new_dna[i] = combine_dna(kitty1_dna[i], kitty2_dna[i], selector[i]);
  }

  let new_kitty = Kitty(new_dna);

  Kitties::<T>::insert(&sender, kitty_id, &new_kitty);
  Self::deposit_event(RawEvent::KittyBred(sender, kitty_id, new_kitty));
}
```

After addition of `breed` function in `kitties` pallet we need to build WASM file.

Here is a link to the whole new implementation: https://github.com/TomaszWaszczyk/substrate-runtime-upgrade-tutorial/blob/after-runtime-upgrade/pallets/kitties/src/lib.rs

After making upgrade we expect that `kitties` pallet will have `breed` function without stopping running chain. Real adaptability of the blockchain.

### Ad. 4: Build WASM file - runtime

Execute following command in order to build runtime wasm file: `WASM_TARGET_DIRECTORY="$(pwd)" cargo build`

After successfully build we expect to have a new file should appear in current directory:

![](./assets/wasm.png)

## Ad 5: Upgrade runtime and verify version

We have running node with version `1` and upgrade runtime into version `2`, we expect that version should change to version `2`. If it will happen it proves that we have upgraded working chain. Proof of successfully making upgrade runtime will be our freshly added `breed` function. Let's check it.

In order to make upgrade go to `Developer` tab, select `Sudo`. Then make sure that you have selected `system` pallet and `setCode(code)`, check in `file upload` and select freshly created WASM file made in previous step `node_template_runtime.wasm` then check in `with weight override` and in the field `unchecked weight for this call` type example value like `100`. To confirm making upgrade click on `Submit Sudo Unchecked`.

![](./assets/upgrade.png)

After successfuly upgrade you should see updated `spec_version` and have access to our new function:

![](./assets/after-upgrade.png)

On the screenshot we can see successfully upgrade, take into consideration that the version of `spec_version` has been changed to incremented value `2`.

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
### 2. Twitter's thread to the most possible issues: https://twitter.com/tomaszwaszczyk/status/1343512637909458944
