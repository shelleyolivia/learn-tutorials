# 6. Deploy a program

A *program* is to Solana what a *smart contract* is to other protocols. Once a program has been deployed, any app can interact with it by sending a transaction to a Solana cluster that will pass it to the program.

Solana programs can be written in C or in Rust. You can learn more about Solana's programs [here](https://docs.solana.com/developing/on-chain-programs/overview).

So far we've been using Solana's JS API to interact with the blockchain. In this chapter we're going to deploy a Solana program using another Solana developer tool: their CLI. We'll install it and use it through our Terminal.

## Install Rust and configure the Solana CLI

For simplicity, perform both of these installations inside the project root:

* [Install the latest Rust stable](https://rustup.rs) : 

```bash
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

* [Install Solana CLI](https://docs.solana.com/cli/install-solana-cli-tools) v1.6.6 or later :

```bash
    sh -c "$(curl -sSfL https://release.solana.com/stable/install)"
```

Set the CLI config URL to the devnet cluster by running this command in your Terminal:

```bash
    solana config set --url https://api.devnet.solana.com
```

If this is your first time using the Solana CLI, you will need to generate a new keypair. 
Run the following command in your Terminal :

```bash
solana-keygen new
```

It will be written to `~/.config/solana/id.json` and will be used every time you use the CLI.

## Understanding the hello-world program

There is a `program` folder at the app's root. It contains the Rust program `src/lib.rs` and some configuration files to help us compile and deploy it.

**It's a simple program, all it does is increment a number every time it's called.**

Let’s dissect what each part does.

```rust
use borsh::{BorshDeserialize, BorshSerialize};
use solana_program::{
    account_info::{next_account_info, AccountInfo},
    entrypoint,
    entrypoint::ProgramResult,
    msg,
    program_error::ProgramError,
    pubkey::Pubkey,
};
```

[`use` declarations](https://doc.rust-lang.org/reference/items/use-declarations.html) are convenient shortcuts to other code. In this case, the serialize and de-serialize functions from the [borsh](https://borsh.io/) crate. borsh stands for _**B**inary **O**bject **R**epresentation **S**erializer for **H**ashing_.  
A [crate](https://learning-rust.github.io/docs/a4.cargo,crates_and_basic_project_structure.html#Crate) is a collection of source code which can be distributed and compiled together. Learn more about [Cargo, Crates and basic project structure](https://learning-rust.github.io/docs/a4.cargo,crates_and_basic_project_structure.html).

We also `use` portions of the `solana_program` crate :

* A function to return the next `AccountInfo` as well as the  struct for `AccountInfo` ;
* The `entrypoint` macro and related `entrypoint::ProgramResult` ;
* The `msg` macro, for low-impact logging on the blockchain ;
* `program_error::ProgramError` which allows on-chain programs to implement program-specific error types and see them returned by the Solana runtime. A program-specific error may be any type that is represented as or serialized to a u32 integer ;
* The `pubkey::Pubkey` struct.

Next we will use the `derive` macro to generate all the necessary boilerplate code to wrap our `GreetingAccount` struct. This happens behind the scenes during compile time [with any `#[derive()]` macros](https://doc.rust-lang.org/reference/procedural-macros.html#derive-macros). Rust macros are a rather large topic to take in, but well worth the effort to understand. For now, just know that this is a shortcut for boilerplate code that is inserted at compile time.

The struct declaration itself is simple, we are using the `pub` keyword to declare our struct publicly accessible, meaning other programs and functions can use it. The `struct` keyword is letting the compiler know that we are defining a struct named `GreetingAccount` , which has a single field : `counter` with a type of `u32` , an unsigned 32-bit integer. This means our counter cannot be larger than [`4,294,967,295`](https://en.wikipedia.org/wiki/4,294,967,295).

```rust
#[derive(BorshSerialize, BorshDeserialize, Debug)]
pub struct GreetingAccount {
    pub counter: u32,
}
```

Next, we declare an entry point - the `process_instruction` function :

```rust
entrypoint!(process_instruction);

pub fn process_instruction(
    program_id: &Pubkey, 
    accounts: &[AccountInfo],
    _instruction_data: &[u8],
```

With a quick detour out of the helloworld example and into the Solana CLI source, we can see the `ProcessInstruction` type being used behind the scenes :

![From solana-program-1.6.6/src/entrypoint.rs](../../../.gitbook/assets/processinstruction.png)

`&Pubkey` is a [borrowed reference](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html) to the public key where the contract is stored, this is our program's identifier or programId.

`&[AccountInfo]` , another borrowed reference this time to an array of accounts, however in this example there is only a single account.

Taking another quick detour out of the program code to peek at the `AccountInfo` struct, we see that `accounts.owner` is also going to be a public key :

![From solana-program-1.6.6/src/account\_info.rs](../../../.gitbook/assets/accountinfo_struct.png)

Back to the hello-world code :

```rust
) -> ProgramResult {
    msg!("Hello World Rust program entrypoint");
    let accounts_iter = &mut accounts.iter();
    let account = next_account_info(accounts_iter)?;
```

The return value of the `process_instruction` entrypoint will be a `ProgramResult` .  
[`Result`](https://doc.rust-lang.org/std/result/enum.Result.html) comes from the `std` crate and is used to express the possibility of error.

For [debugging](https://docs.solana.com/developing/on-chain-programs/debugging), we can print messages to the Program Log [with the `msg!()` macro](https://docs.rs/solana-program/1.7.3/solana_program/macro.msg.html), rather than use `println!()` which would be prohibitive in terms of computational cost for the network.

The `let` keyword in Rust binds a value to a variable. By looping through the `accounts` using an [iterator](https://doc.rust-lang.org/book/ch13-02-iterators.html), `accounts_iter` is taking a [mutable reference](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html#mutable-references) of each value in `accounts`. Then `next_account_info(accounts_iter)?`will return the next `AccountInfo` or a `NotEnoughAccountKeys` error. Notice the `?` at the end, this is a [shortcut expression](https://doc.rust-lang.org/std/result/#the-question-mark-operator-) in Rust for [error propagation](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#propagating-errors).

```rust
if account.owner != program_id {
  msg!("Greeted account does not have the correct program id");
  return Err(ProgramError::IncorrectProgramId);
}
```

We will perform a security check to see if the account owner has permission. If the `account.owner` public key does not equal the `program_id` we will return an error.

```rust
let mut greeting_account = GreetingAccount::try_from_slice(&account.data.borrow())?;
greeting_account.counter += 1;
greeting_account.serialize(&mut &mut account.data.borrow_mut()[..])?;

msg!("Greeted {} time(s)!", greeting_account.counter);

Ok(())
```

Finally we get to the good stuff where we "borrow" the existing account data, increase the value of `counter` by one and write it back to storage.

* The `GreetingAccount` struct has only one field - `counter`. To be able to modify it, we need to borrow the reference to `account.data` with the `&`[borrow operator](https://doc.rust-lang.org/reference/expressions/operator-expr.html#borrow-operators). 
* The `try_from_slice()` function from `BorshDeserialize`will mutably reference and deserialize the `account.data`. 
* The `borrow()` function comes from the Rust core library, and exists to immutably borrow the wrapped value. 

Taken together, this is saying that we will borrow the account data and pass it to a function that will deserialize it and return an error if one occurs. Recall that `?` is for error propagation.

Next, incrementing the value of `counter` by `1` is simple, using the addition assignment operator : `+=` .

With the `serialize()` function from `BorshSerialize`, the new `counter` value is sent back to Solana in the correct format. The mechanism by which this occurs is the [Write trait](https://doc.rust-lang.org/std/io/trait.Write.html) from the `std::io` crate.

We can then show in the Program Log how many times the count has been incremented by using the `msg!()` macro.

To finish the program we will call `Ok(())` - which is the Result `Ok()` containing a Rust primitive called a [unit](https://doc.rust-lang.org/std/primitive.unit.html). The unit `()` type has exactly one value :`()`, and is used when there is no other meaningful value that could be returned.

## Testing the program

To ensure that the program code has no obvious bugs and passes any tests defined in the sourcefile, it is good to run the unit tests before building and deploying. 

Run the command `cargo test` inside of the `learn-solana-dapp/program` subdirectory. The first time you do this, Cargo will need to compile a lot of related crates (libc, borsh, the Solana crates, even the program we are testing). This process can take several minutes, although future tests will occur much more rapidly since everything is compiled.

The output from a successful `cargo test` will look like this (timestamps have been removed) :

```bash
running 1 test
test test::test_sanity ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running tests/lib.rs (target/debug/deps/lib-560d97a774ffe546)

running 1 test
[ INFO  solana_program_test] "helloworld" program loaded as native code
[ DEBUG solana_runtime::message_processor] Program log: Hello World Rust program entrypoint
[ DEBUG solana_runtime::message_processor] Program log: Greeted 1 time(s)!
[ DEBUG solana_runtime::message_processor] Program log: Hello World Rust program entrypoint
[ DEBUG solana_runtime::message_processor] Program log: Greeted 2 time(s)!
test test_helloworld ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.54s
```

## Building the program

The first thing we're going to do is compile the Rust program to prepare it for the CLI. To do this we're going to use a custom script that's defined in `package.json`:

```javascript
"scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "build:program-rust": "cargo build-bpf --manifest-path=contracts/solana/program/Cargo.toml --bpf-out-dir=dist/solana/program",
  },
```

`start` , `build` , `test` , `eject` are all default and part of the `create-react-app` template.

The custom build script uses :

* `cargo` : Rust’s build system and package manager ([docs](https://doc.rust-lang.org/book/ch01-03-hello-cargo.html)), like what `npm` is to Javascript.
* `build-bpf` : the cargo command we're going to run to build/compile the program. This is installed along with the Solana CLI. We pass it a manifest file which is the `Cargo.toml` of our program and the desired output directory.

Let's run the script and build the program by running the following command in the terminal (from the project root directory):

```bash
yarn run build:program-rust
```

> This step can take 5 or 10 minutes!

When it's successful you will see a new folder in your app which contains the compiled contract: `hello-world.so`.

> The `.so` extension does not stand for Solana! It stands for "shared object". You can read more about Solana Programs [here](https://docs.solana.com/developing/on-chain-programs/overview) 


## Potential issues building

An error `no such subcommand:build-bpf` indicates that there was an issue with the installation of the Solana CLI or that it is installed, but not in the PATH. So if you see this error and exit code 101 :

```bash
$ cargo build-bpf --manifest-path=program/Cargo.toml --bpf-out-dir=dist/program
error: no such subcommand: `build-bpf`
error Command failed with exit code 101.
info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
```

Be sure to set your PATH according to your Solana release (`active_release`is a symbolic link) :

`PATH="~/.local/share/solana/install/active_release/bin:$PATH"`

## Deploying the program

Next we're going to deploy the program to the devnet cluster. The CLI provides a very simple interface for this :

```bash
solana program deploy -v dist/program/helloworld.so
```

The `-v` Verbose flag is optional, but it will show some related information like the RPC URL and path to the default signer keypair, as well as the expected [Commitment level](https://docs.solana.com/implemented-proposals/commitment). When the process completes, the Program Id will be displayed :

```bash
RPC URL: https://api.devnet.solana.com
Default Signer Path: ~/.config/solana/id.json
Commitment: confirmed
Program Id: 2NsKheB6kXo9rA9eYzdMW978GvbPX6Z4KX4PB7p42Cc
```

## Potential issues deploying

### Insufficient Funds

The first time you run this, the CLI should error out because the account trying to deploy this program **does not have enough funds to spend**. You can fix this by going back a few steps in the React app to the "Fund" step and pasting in the input the public key that you see in the Terminal error. Click "Fund" a few times to get enough devnet tokens to be able to deploy the program

![](../../../.gitbook/assets/screen-shot-2021-06-14-at-8.20.30-pm.png)

### Custom program error 0x1

If you see

> Error: Deploying program failed: Error processing Instruction 1: custom program error: 0x1

simply re-run the deploy command until it succeeds. If you run out of funds, go back to the "Fund" step and get more!

### Program's authority X does not match authority provided Y

This can be due to how your Solana keypair was generated. You can generate a fresh one by running :

```bash
solana-keygen new --force
```

Then go through the tutorial steps again (fund, build, deploy).

### solana deploy --help

For quick reference, you can issue the following command :

```text
solana-deploy --help
Deploy a program

USAGE:
    solana deploy [FLAGS] [OPTIONS] <PROGRAM_FILEPATH> [PROGRAM_ADDRESS_SIGNER]

FLAGS:
        --allow-excessive-deploy-account-balance
...
```

## After successfully deploying the program

On success, the CLI will print the programId of the deployed contract.

```bash
Program Id: DwpsLw56wmAr3FMZiWHHK47vwZx9LYreT9r32Sn9tBZ5
```

If you visit [https://explorer.solana.com](https://explorer.solana.com/), change the cluster to devnet and paste this Program Id string, you should see a page like this:

> Make sure you select Devnet on the Solana Explorer!

![](../../../.gitbook/assets/screen-shot-2021-06-14-at-3.52.29-pm.png)

Notice that the field `Executable` is now set to `Yes` because the address we're looking up is for a program which can be called and executed.

## Save the program and its author secret keys

> Before we move to the next step we need to save two important variables!

1. In your terminal run `cat ~/.config/solana/id.json` and copy its output. In `src/components/Call.jsx` assign it to the constant `PAYER_SECRET_KEY`. This is the Keypair information of the author of the program (you!). We will need to pass it to transactions we make to the program  to authenticate ourselves as the owner of the program. 
2. In the directory, find `dist/program/helloworld-keypair.json` and copy its contents. In `/src/components/Call.jsx` assign it to the constant `PROGRAM_SECRET_KEY`. This is the Keypair information of the program itself. We will need it to generate the program's public key that will be used to call the program.

## Next

So at this point, we've deployed our dummy smart contract to Solana's devnet cluster. We're finally ready for the big moment: Interacting with the program by calling its functionality from the UI!
