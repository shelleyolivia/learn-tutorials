# Introduction

Solana is a decentralized blockchain built to enable scalable, user-friendly apps for the world at $0.00025 average cost per transaction. In this tutorial, we are going to build a decentralized mail app to showcase the potential of Solana, we will build the on-chain program with [Rust](https://www.rust-lang.org/) and the web UI with [React](https://reactjs.org/) and [MaterialUI](https://material-ui.com/). We'll go through the code together, building the program and web UI step by step.

You can find the final code to the on-chain program [here](#todo) and the UI [here](#todo)

# Prerequisites

A good understanding of the Rust programming language and React is very important to grasping the contents of this tutorial. I suggest a primer on the both following the links provided in the introduction section.

# Requirements

- Install rust from [HERE](https://rustup.rs/)
- [Git](https://git-scm.com/downloads) and
- Solana CLI [HERE](https://docs.solana.com/cli/install-solana-cli-tools#use-solanas-install-tool)

# Building the mail program

# Project setup

Head over to [this template](https://github.com/mvines/solana-bpf-program-template) on Github and click on **Use this template** button, which will allow you to create a new repository. Give your repo a new name and check the 'Include all branches' box, then click on **Create repository from template** button. Now clone the repository to your local machine with your preferred method, or simply by using `git clone https://github.com/<your Github username>/<what you named the repo>`.

Open `Cargo.toml` in your favourite editor and remove the crates under `[dev-dependencies]`. It should now look like this:

```toml
[package]
name = "solana-mail"
version = "0.1.0"
edition = "2018"
license = "WTFPL"
publish = false

[dependencies]
solana-program = "=1.7.10"

[features]
test-bpf = []

[lib]
crate-type = ["cdylib", "lib"]
```

# entrypoint, programs, and accounts

Open `lib.rs` in the `src` folder. It currently contains:

```rust
use solana_program::{
  account_info::AccountInfo, entrypoint, entrypoint::ProgramResult, msg, pubkey::Pubkey,
};

entrypoint!(process_instruction);
fn process_instruction(
  program_id: &Pubkey,
  accounts: &[AccountInfo],
  instruction_data: &[u8],
) -> ProgramResult {
  msg!(
    "process_instruction: {}: {} accounts, data={:?}",
    program_id,
    accounts.len(),
    instruction_data
  );
  Ok(())
}

#[cfg(test)]
mod test {
  use {
    super::*,
    assert_matches::*,
    solana_program::instruction::{AccountMeta, Instruction},
    solana_program_test::*,
    solana_sdk::{signature::Signer, transaction::Transaction},
  };

  #[tokio::test]
  async fn test_transaction() {
    let program_id = Pubkey::new_unique();

    let (mut banks_client, payer, recent_blockhash) = ProgramTest::new(
      "bpf_program_template",
      program_id,
      processor!(process_instruction),
    )
    .start()
    .await;

    let mut transaction = Transaction::new_with_payer(
      &[Instruction {
        program_id,
        accounts: vec![AccountMeta::new(payer.pubkey(), false)],
        data: vec![1, 2, 3],
      }],
      Some(&payer.pubkey()),
    );
    transaction.sign(&[&payer], recent_blockhash);

    assert_matches!(banks_client.process_transaction(transaction).await, Ok(()));
  }
}
```

The crates used by this file are brought into scope with the [use](https://doc.rust-lang.org/stable/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html) keyword. Then the `entrypoint!` [macro](https://doc.rust-lang.org/stable/book/ch19-06-macros.html) is used to declare `process_instruction` as the [entrypoint](https://docs.solana.com/developing/on-chain-programs/developing-rust#program-entrypoint) to the program. As the name suggests, entrypoint is our program's point of entry, all calls to the program goes through the `process_instruction` function.

The function takes three arguments:

- `program_id` - This is the [public key](https://docs.solana.com/terminology#public-key-pubkey) of our program, the program's identifier
- `accounts` - Solana programs are stateless, the programs themselves don't store data between transactions. To store state, we use [accounts](https://docs.solana.com/developing/programming-model/accounts)
- `instruction_data` - This is the data passed to the program by the calling code

# Code structure

We are going to structure our code using the following format:

```text
├─ src
│  ├─ lib.rs -> registering modules
│  ├─ entrypoint.rs -> entrypoint to the program
│  ├─ instruction.rs -> program API, (de)serializing instruction data
│  ├─ processor.rs -> program logic
│  ├─ state.rs -> program objects, (de)serializing state
│  ├─ error.rs -> program specific errors
├─ .gitignore
├─ Cargo.lock
├─ Cargo.toml
├─ Xargo.toml
```

Using this structure, our program's flow will look like this:

1. `entrypoint` forwards incoming calls to the `processor`
2. The `processor` decodes the `instruction_data` using functions from `instruction.rs`
3. After decoding the data, the `processor` will then use one of the prepared functions to handle the request
4. `state.rs` contains models of the data used in the project.

For ecample, say Alice creates an account on our program. What that means for us is to generate a [Program Derived Address (PDA)](https://docs.solana.com/developing/programming-model/calling-between-programs#program-derived-addresses) from her wallet's public key, a seed phrase, and our program's public key. The PDA will act as Alice's account id which she will use to send and receive mails. The PDA doesn't only act as her account id but also as a store for her data, this is where all the mail she has sent and received will be stored.

{% hint style="info" %}
Accounts in Solana are owned by programs. A program can only manipulate the data in an account that it owns, hence the need for generating a PDA account owned by our program as our user's account id.
{% endhint %}

If Alice wants to send a mail to Bob, she sends the mail data as a request to our program, the program's `entrypoint` function fowards the request and all its data to the `processor`. Then the `processor` decodes the data with the help of `instruction.rs` and the decoding logic in `state.rs`. Finally the `processor` calls a function to save the mail data sent in the request to the inbox of the receiver's account (which was sent along with the request).

Create the following files in the src folder:

- entrypoint.rs
- error.rs
- instruction.rs
- processor.rs
- state.rs

# State, part 1

Open the file `lib.rs`, remove the existing content then add the following code, which defines the modules we will use:

```rust
pub mod entrypoint;
pub mod error;
pub mod instruction;
pub mod processor;
pub mod state;
```

The next thing we need to do here is open `state.rs` and declare our models. In the file `state.rs`, add the following struct:

```rust
pub struct Mail {
  pub id: String,
  pub from_address: String,
  pub to_address: String,
  pub subject: String,
  pub body: String,
  pub sent_date: String,
}
```

What we just did here is declare a struct that will represent our mail objects. Each mail object will have a `from_address` and `to_address`, fields of type String that hold a base58 representations of the sender's and receiver's public key. A `subject` field representing the subject of the mail, a `body` field that is the content of the mail and finally `sent_Date`, the date the mail was sent. We are using the `String` data-type for the `sent_date` field because the date we will really be saving is going to be the string value of the specified date.

One other thing to note here is that data in accounts are stored in `Uint8Array` format. What this means is, if we want to store the string "Solana is awesome" in an account's data, we will need to serialize the string. We do this by converting each character in the string to decimals following the [UTF-8](https://en.wikipedia.org/wiki/UTF-8) format. In this case, the string "Solana is awesome" will be serialized into a `Uint8Array` as "[83, 111, 108, 97, 110, 97, 32, 105, 115, 32, 97, 119, 101, 115, 111, 109, 101]"

Keeping this in mind, we now know we need to serialize our mail data before we can store it on the network. For this we will need to use the [Borsh](https://borsh.io/) crate. Open `Cargo.toml` and add the following dependencies:

```toml
...
[dependencies]
solana-program = "=1.7.10"
borsh = "0.9.1"
borsh-derive = "0.9.1"
...
```

{% hint style="info" %}
Three dots ... on a line by themselves indicate the presence of other code which we have trimmed for display purposes.
{% endhint %}

Then add the following code to `state.rs`

```rust
use borsh::{BorshDeserialize, BorshSerialize};

#[derive(BorshDeserialize, BorshSerialize, Debug)]
pub struct Mail{
...
```

What we just did here is provide basic implementations for `BorshDeserialize`, `BorshSerialize`, and the `Debug` traits using the `derive` macro. The `Debug` trait allows us to print the content of a mail object to the console using the `"{:?}" debug `formatter. `BorshDeserialize`adds an associated method named`try_from_slice()`that we can use to construct a Mail object from a reference to a slice of `u8`. Finally, `BorshSerialize` provides a basic implementation of a method named `serialize()`, this allows us to serialize the content of a Mail object into a slice of `u8`.

Now that we have a `struct` that we can build mail objects from, lets add another `struct` that will serve as the state of our user's Account data. This `struct` will have two fields, `inbox` and `sent`, the two fields will be vectors of `Mail` representing the lists of mails the user has in their Account.

Still in `state.rs`, add the following after the Mail struct declaration:

```rust
#[derive(BorshDeserialize, BorshSerialize, Debug)]
pub struct MailAccount {
  pub inbox: Vec<Mail>,
  pub sent: Vec<Mail>,
}
```

With this, we are done with `state.rs` for now.

# Entrypoint

Open the file `entrypoint.rs` in the editor and paste the following:

```rust
use crate::processor::Processor;
use solana_program::{
  account_info::AccountInfo, entrypoint, entrypoint::ProgramResult, pubkey::Pubkey,
};

entrypoint!(process_instruction);
fn process_instruction(
  program_id: &Pubkey,
  accounts: &[AccountInfo],
  instruction_data: &[u8],
) -> ProgramResult {
  Processor::process(program_id, accounts, instruction_data)
}
```

The first two lines here import the `Processor` struct from the processor module which we will declare soon and `AccountInfo`, `entrypoint`, `ProgramResult`, and `Pubkey` from the `solana_progam` crate.

The `process_instruction()` serves as the entrypoint of our program, every request sent to the program will be handled by this function. The function takes as arguments:

- `program_id` - As mentioned before, this is a reference to a public key of the deployed program which acts as a way to identify the program
- `accounts` - This a reference to a slice of `AccountInfo`
- `instruction_data` - This is a reference to a slice of `u8` containing the data passed to the program to be processed, this will contain the mail object among other things.

In the body of the `process_instruction()` function, we have just one line of code, a call to a yet to be declared function, `process()`, of the `Processor` struct from the processor module. This function is going to be handling all the requests that come to our program.

# Instruction, part 1

The `instruction.rs` module will contain the API definition of our program. The first endpoint we are going to define here is the `InitAccount` endpoint.

```rust
#[derive(Debug)]
pub enum MailInstruction {
  /// Initialize a new account
  ///
  /// Accounts expected
  ///
  /// 1. `[writable]` The AccountInfo of the account to be initialized
  InitAccount,
}
```

The `instruction` module defines an enum `MailInstruction`. This enum currently declares one endpoint, `InitAccount`. The comment above it indicates that it takes a writable `account` which is the PDA generated for the user. We'll see how this works when we start building the UI.

```rust
use crate::error::MailError::InvalidInstruction;
use solana_program::program_error::ProgramError;
...
impl MailInstruction {

  pub fn unpack(input: &[u8]) -> Result<Self, ProgramError> {
    let (tag, rest) = input.split_first().ok_or(InvalidInstruction)?;

    Ok(match tag {
      0 => Self::InitAccount,
      _ => return Err(InvalidInstruction.into()),
    })
  }
}
```

Still in the instruction module, we add an associated method to the MailInstruction enum named `unpack()`. The method expects a reference to a slice of `u8` - this will be the `instruction_data` argument passed to our entrypoint function.

In the first line of the `unpack()` method, we call the [split_first()](https://doc.rust-lang.org/std/primitive.slice.html#method.split_first) method on the slice of `u8`. The `split_first()` method returns an [Option()](https://doc.rust-lang.org/std/option/enum.Option.html) enum with the first and all the rest of the elements of the slice. The first element serves as a tag that determines how we decode the rest of the instruction. Note the [ok_or()](https://doc.rust-lang.org/std/option/enum.Option.html#method.ok_or) call on the returned `Option()` from the `split_first()` call? You should look up the methods yourself to get a deeper understanding of what's going on with the calls. What's most important here is that you understand that we provide a custom error `InvalidInstruction` to `ok_or()` method in case the call to `split_first()` fails.

This code won't compile because we have yet to define the custom error.

# Error

In the `error.rs` module:

```rust
use thiserror::Error;

#[derive(Error, Debug, Copy, Clone)]
pub enum MailError {
  /// Invalid Instruction
  #[error("Invalid Instruction")]
  InvalidInstruction,
}
```

Update the dependencies in `Cargo.toml` with:

```rust
thiserror = "1.0.24"
```

We [defined an error type](https://doc.rust-lang.org/rust-by-example/error/multiple_error_types/define_error_type.html) in `error.rs` and then use the `Error` trait from the [thiserror](https://docs.rs/thiserror/latest/thiserror/) library to implement `fmt::Display` for our custom error.

Still one more thing left to do here. We are not completely done with our error module just yet. If you look at our `unpack()` method in `instruction.rs`, it's supposed to return a `ProgramError` in case of an error but the error we just declared is not implemented for `ProgramError`. Let's take care of that with:

```rust
use solana_program::program_error::ProgramError;

impl From<MailError> for ProgramError {
  fn from(e: MailError) -> Self {
    ProgramError::Custom(e as u32)
  }
}
```

Here, we implement the [From](https://doc.rust-lang.org/std/convert/trait.From.html) trait which the `?` operator uses to return an error.

Now that we have a basic setup for our error, entrypoint and instruction, we are now ready to code `processor.rs`

# Processor, part 1

In the `processor.rs` module, paste:

```rust
use crate::instruction::MailInstruction;
use solana_program::{
  account_info::AccountInfo,
  entrypoint::ProgramResult,
  msg,
  pubkey::Pubkey,
};

pub struct Processor;
impl Processor {
    pub fn process(
      program_id: &Pubkey;
      accounts: &[AccountInfo],
      instruction_data: &[u8],
    ) -> ProgramResult {
    let instruction = MailInstruction::unpack(instruction_data)?;

    match instruction {
      MailInstruction::InitAccount => {
        msg!("Instruction: InitAccount");
        Self::process_init_account(accounts, program_id)
      }
    }
  }
}
```

Back in our `entrypoint.rs` module, we said the `process()` method was going to handle every request sent to our program. This is how we accomplish it. We pass the reference to the slice holding the `instruction_data` from the `process_instruction()` function in the `entrypoint.rs` module to the `unpack()` method of `MailInstruction`. We then use `match` to decide which request we are responding to, and the right method to call using the enum returned from the call to `unpack()`. In the case of `InitAccount` we call the `process_init_account()` function:

```rust
/// inside the Processor implementation, right after the process function
...

fn process_init_account(
    account: &AccountInfo,
    program_id: &Pubkey
  ) -> ProgramResult {

  Ok(())
}
```

This is the declaration of the `process_init_account` method, it takes a reference to an `AccountInfo` - the account info of the PDA address of the user, and a reference to a `Pubkey` - the program id, the reason we need the program id will soon become clear.

To initialize an account, we are only going to send a welcome mail to the account. This serves to show that our setup works. Before we go aheaad to actually compose a mail object and then add the instance to the user's inbox, we need to validate the account to make sure we can write to it and that it belongs to this program.

```rust
use crate::error::MailError::NotWritable;
use crate::instruction::MailInstruction;
use solana_program::{
  account_info::AccountInfo,
  entrypoint::ProgramResult,
  msg,
  program_error::ProgramError,
  pubkey::Pubkey,
};

...

fn process_init_account(
    account: &AccountInfo,
    program_id: &Pubkey
  ) -> ProgramResult {
  if !account.is_writable {
    return Err(NotWritable.into());
  }

  if account.owner != program_id {
    return Err(ProgramError::IncorrectProgramId);
  }

  Ok(())
}
```

The first `if statement` checks wether the account is writable and returns an error if it's not. `NotWritable` is a custom error and should be added to the `MailError` struct in the `error.rs` module. You should do that before moving on.

The second `if statement` confirms that the account owner is the same as program id. If they are not the same, it will return an error.

Update the import section in `proessor.rs`:

```rust
use crate::error::MailError::NotWritable;
use crate::instruction::MailInstruction;
use crate::state::{Mail, MailAccount};
use borsh::BorshSerialize;
use solana_program::{
  account_info::AccountInfo,
  entrypoint::ProgramResult,
  msg,
  program_error::ProgramError,
  pubkey::Pubkey,
};
```

Next, we compose a welcome mail and add it to the accounts data. Inside the `process_init_account` method, right after the two if statements, add:

```rust
...
let welcome_mail = Mail {
  id: String::from("00000000-0000-0000-0000-000000000000"),
  from_address: program_id.to_string(),
  to_address: account.key.to_string(),
  subject: String::from("Welcome to SolMail"),
  body: String::from("This is the start of your private messages on SolMail
  Lorem, ipsum dolor sit amet consectetur adipisicing elit. Quos ut labore, debitis assumenda, dolorem nulla facere soluta exercitationem excepturi provident ipsam reprehenderit repellat quisquam corrupti commodi fugiat iusto quae voluptates!"),
  sent_date: "9/29/2021, 3:58:02 PM"
};

let mail_account = MailAccount {
  inbox: vec![welcome_mail],
  sent: Vec::new(),
};

mail_account.serialize(&mut &mut account.data.borrow_mut()[..])?;
...
```

We instantiated an instance of the `Mail` struct, and then filled in data for its fields. We also instantiated a `MailAccount` object, added the welcome mail instance to the inbox field using the `vec!` macro and then assigned an empty vector object to the sent field. In the last line, we serialized the mail_account instance and then write the serialized data to the user's account data by calling the `serialize()` method of the `mail_account` instance with a mutably borrowed data from the user's account data.

You might notice the tricky `&mut &mut account.data.borrow_mut()[..]` expression. The `serialize()` method takes a reference to a mutable slice of `u8` as an argument, the `borrow_mut()` method returns a `RefMut`. We can't pass `RefMut` to a method that expects a slice, so we take a mutable slice of `RefMut` which returns a mutable slice of `u8`. The [repo](todo) on Github has some sanity test for the methods here.

We are done with our account initialization, we can now move on to adding the logic to handle requests for sending mails between two accounts.

# Instruction, part 2

In the `instruction.rs` module, we stopped at:

```rust
impl MailInstruction {
  use crate::error::MailError::InvalidInstruction;
  use solana_program::program_error::ProgramError;

#[derive(Debug)]
pub enum MailInstruction {
  /// Initialize a new account
  ///
  /// Accounts expected
  ///
  /// 1. `[writable]` The account to be initialized
  InitAccount,
}

impl MailInstruction {
  pub fn unpack(input: &[u8]) -> Result<Self, ProgramError> {
    let (tag, rest) = input.split_first().ok_or(InvalidInstruction)?;

    Ok(match tag {
      0 => Self::InitAccount,
      _ => return Err(InvalidInstruction.into()),
    })
  }
}
```

We had one endpoint, the InitAccount endpoint. We now need to add another endpoint, the `SendMail` endpoint. This endpoint will be responsible for deserializing a mail instance and returning it to the `process` method.

Update the `MailInstruction` enum declaration to:

```rust
#[derive(Debug)]
pub enum MailInstruction {
  /// Initialize a new account
  ///
  /// Accounts expected
  ///
  /// 1. `[writable]` The AccountInfo of the account to be initialized
  InitAccount,
  /// Send a mail to an account.
  ///
  /// Accounts expected:
  ///
  /// 1. `[writable]` The AccountInfo of the sender
  /// 2. `[writable]` The AccountInfo of the receiver
  SendMail { mail: Mail },
}
```

We added a new enum `SendMail` - a struct like enum that has a mail field. The endpoint takes two writable PDA accounts, the sender and and receiver's accounts respectively.

Update the code in the `match` expression of the `unpack()` method to:

```rust
Ok(match tag {
  0 => Self::InitAccount,
  1 => Self::SendMail {
    mail: Mail::try_from_slice(&rest)?,
  },
  _ => return Err(InvalidInstruction.into()),
})
```

This case matches instances where the value of the first element of the instruction data is `1`. If the value is `1`, the function returns a `SendMail` enum with the mail instance that was deserialized from the rest of the `data`.

# Processor, part 2

In the `process()` method, add another case to the match expression:

```rust
MailInstruction::SendMail { mail } => {
  msg!("Instruction: SendMail");
  Self::process_send_mail(accounts, mail, program_id)
}
```

The case gets the mail instance from the `MailInstruction` enum then call the associated `process_send_mail()` method. The method is responsible for writing the mail instance to the inbox array of mails already in the `inbox` field of the receiver's account data, do the same for the sender's account data but to the `sent` field instead.

Inside the `process()` method, right after the `process_init_account` method, declare:

```rust
fn process_send_mail(accounts: &[AccountInfo], mail: &Mail, program_id: &Pubkey) -> ProgramResult {
  Ok(())
}
```

The method takes a reference to a slice of `AccountInfo`, a reference to a `Mail` instance and the program_id as arguments.

Paste this into the body of the method right before `Ok(())`:

```rust
...
let sender_account = &accounts[0];

if !sender_account.is_writable {
  return Err(NotWritable.into());
}

if sender_account.owner != program_id {
  return Err(ProgramError::IncorrectProgramId);
}

let receiver_account = &accounts[1];

if !receiver_account.is_writable {
  return Err(NotWritable.into());
}

if receiver_account.owner != program_id {
  return Err(ProgramError::IncorrectProgramId);
}
...
```

As declared by the endpoint in `MailInstruction`, the endpoint takes two `AccountInfo` objects as arguments, the `sender` and `receiver` respectively.

To start this method, we validate the accounts. Same as we did at the start of the `process_init_account()` method. Next, update the import section:

```rust
use crate::error::MailError::NotWritable;
use crate::instruction::MailInstruction;
use crate::state::{Mail, MailAccount};
use borsh::{BorshDeserialize, BorshSerialize};
use solana_program::{
  account_info::AccountInfo,
  entrypoint::ProgramResult,
  msg,
  program_error::ProgramError,
  pubkey::Pubkey,
}
`
```

Then, right after the last code in the `process_send_mail()` method just before `Ok(())`, add:

```rust
...
let sender_data = MailAccount::try_from_slice(&sender_account.data.borrow()[..]);
sender_data.sent.push(mail.clone());
sender_data.serialize(&mut &mut sender_account.data.borrow_mut()[..])?;

receiver_data = MailAccount::try_from_slice(&receiver_account.data.borrow()[..])?;
receiver_data.inbox.push(mail.clone());
receiver_data.serialize(&mut &mut receiver_account.data.borrow_mut()[..])?;
...
```

Here, we used the `try_from_slice()` associated method we got from implementing the `BorshDeSerialize` trait on the `MaiAccount` struct. `try_from_slice()` constructs and returns an instance of the object it was called on from the serialized slice it was passed.

We then add the mail instance that was deserialized from the request data to the `sent` field of the `MailAccount` instance of the sender, serialized and write the sender's `MailAccount` instance back to their accounts data.

In the last part of the code, we used the same pattern as we did above to add the `mail` instance to the receiver's `inbox` field then write the data back to their account.

Notice the `clone()` call on the mail reference?, `mail` is a reference to a `mail` instance, not an instance itself. The `push()` method takes a mail instance argument and not a refernce to one, and that is why we called `clone()` on the reference. The `clone()` method returns a copy of the mail instance that mail references. There's still a problem though, we need to implement the `Clone` trait for the `Mail` struct for us to be able to make this call. You should figure out how to do this yourself.

With this, users will be able to send and receive mails to/from other users.

The `process_send_mail()` method will fail if we were to leave it in its current state and here's why: The `serialize()` method that we called on the `mail_account` instance is derived from implementing `BorshSerialize` in the `MailAccount's` struct declaration. The method takes a reference to a slice of `u8` argument, serializes the data of the instance it's called on (`mail_account` in our case), then writes the serialized data to the slice it was called with. The method will try to read all the bytes in the slice it was passed. The method will fail to construct an object from the slice if the slice has more bytes in total than the actual bytes of the object.

To better understand this, let's shift our focus from this project to the following example:

```rust
/// Declare a de/serializable struct with just one field
#[derive(BorshDeserialize, BorshSerialize, Debug)]
struct DataLength {
  pub length: u32,
}

/// Instantiate a DataLength instance
let data_length = DataLength {
  length: 5
};

/// Assigns a length 8 array filled with 0 to temp_slice
let temp_slice = [0; 8];
/// Serialize data_length and write it to temp_slice
data_length.serialize(&mut &mut temp_slice[..]);
/// Will print [5, 0, 0, 0, 0, 0, 0, 0]
msg!("{:?}", temp_slice);

/// The piece of code above should execute without any error.
/// On the next line, we try to deserialize a DataLength instance from temp_slice
let data_length = DataLength::try_from_slice(&temp_slice);
msg!("{:?}", data_length);
/// The above line will fail with: Err(Custom { kind: InvalidData, error: "Not all bytes read" })
/// The try_from_slice() method tried to read all 8 bytes from the slice when it only needed to read 4 bytes to construct a valid DataLength instance.
/// To fix the above bug, we need to pass a slice of the bytes try_from_slice() needs to construct a valid object as its argument.
/// Change the last line to
let data_length = DataLength::try_from_slice(&temp_slice[..4]);
msg!("{:?}", data_length);
/// What we just did is pass a reference of the first four bytes of temp_slice to try_from_slice()
/// The exact bytes that holds the serialized data for a DataLength instance.
/// With this edit, the code should print Ok(DataLength { length: 5 })
```

With the above example in mind, lets focus back on our program. We now know the reason the `process_send_mail()` method will fail, we need to pass a slice that holds only the bytes of a serialized `MailAccount` struct instance to the `try_from_slice()`. To do this, we need to store the length of the `MailAccount` instance in a user's account.

# State, part 2

We need to add another struct to the state module:

```rust
#[derive(BorshDeserialize, BorshSerialize, Debug)]
pub struct DataLength {
  pub length: u32,
}
```

This struct has one `u32` field, `length`. We will use an instance of this struct to store the length of the serialized `MailAccount` instance stored in the user's account data every time we update the data

# Processor, part 3

By adding the `DataLength` struct, we now have what we need to fix the bug in `process_send_mail()`. Update the `process_send_mail()` method from the **processor part 2** section to:

```rust
/// Update the import
use crate::error::MailError::NotWritable;
use crate::instruction::MailInstruction;
use crate::state::{DataLength, Mail, MailAccount};
use borsh::{BorshDeserialize, BorshSerialize};
use solana_program::{
  account_info::AccountInfo, borsh::get_instance_packed_len, entrypoint::ProgramResult, msg,
  program_error::ProgramError, pubkey::Pubkey,
};
use std::convert::TryFrom;
...
fn proess_send_mail(
  accounts: &[AccountInfo],
  mail: &Mail,
  program_id: &Pubkey,
) -> ProgramResult {
  ...
  /// right after the if statements:
  let offset: usize = 4;

  let data_length = DataLength::try_from_slice(&sender_account.data.borrow()[..offset])?;

  let mut sender_data;
  if data_length.length > 0 {
    let length = usize::try_from(data_length.length + u32::try_from(offset).unwrap()).unwrap();
    sender_data = MailAccount::try_from_slice(&sender_account.data.borrow()[offset..length])?;
  } else {
    sender_data = MailAccount {
      inbox: Vec::new(),
      sent: Vec::new(),
    };
  }

  sender_data.sent.push(mail.clone());
  let data_length = DataLength {
    length: u32::try_from(get_instance_packed_len(&sender_data)?).unwrap(),
  };
  data_length.serialize(&mut &mut sender_account.data.borrow_mut()[..offset])?;
  sender_data.serialize(&mut &mut sender_account.data.borrow_mut()[offset..])?;

  let data_length = DataLength::try_from_slice(&receiver_account.data.borrow()[..offset])?;

  let mut receiver_data;
  if data_length.length > 0 {
    let length = usize::try_from(data_length.length + u32::try_from(offset).unwrap()).unwrap();
    receiver_data = MailAccount::try_from_slice(&receiver_account.data.borrow()[offset..length])?;
  } else {
    receiver_data = MailAccount {
      inbox: Vec::new(),
      sent: Vec::new(),
    }
  }
  receiver_data.inbox.push(mail.clone());

  let data_length = DataLength {
    length: u32::try_from(get_instance_packed_len(&receiver_data)?).unwrap(),
  };
  data_length.serialize(&mut &mut receiver_account.data.borrow_mut()[..offset])?;
  receiver_data.serialize(&mut &mut receiver_account.data.borrow_mut()[offset..])?;
}
```

If you compare this edit to the one in the **process part 2** section, you should see the changes we made. We are storing two struct instances in the user's account data now, the user's `MailAccount` instance and the `DataLength` of the `MailAccount` instance.

The state of a user account data would now look like:

```rust
[data_length, mail_account, 0, 0, 0, 0, 0, ..];
```

A `data_length` instance is 4 bytes in length so the `MailAccount` instance will start from the third index of the array. We initialized the `offset` variable to `4` with `let offset = 4`, we made the type of the variable `usize` because that's the type we need to index into a slice.

We grabbed the `data_length` instance from the account's data making sure to pass only the required slice with `let data_length = DataLength::try_from_slice(&sender_account.data.borrow()[..offset])?;`. With an if statement, we checked the length of the data instance is more than `0`. If true, we get the length of the `MailAccount` instance and then deserialize it from the data:

```rust
let length = usize::try_from(data_length.length + u32::try_from(offset).unwrap()).unwrap();
sender_data = MailAccount::try_from_slice(&sender_account.data.borrow()[offset..length])?;
```

On the other hand, if length is not greater than `0`, we only instantiate a `MailAccount` object and assigned it to `sender_data`:

```rust
sender_data = MailAccount {
  inbox: Vec::new(),
  sent: Vec::new(),
};
```

Next, we added the `mail` instance to the sender account's `MailAccount` instance, got the length of the serialized form of the instance with the [get_instance_packed_len()](https://docs.rs/solana-program/1.7.3/solana_program/borsh/fn.get_instance_packed_len.html) method, used that to instantiate a new `DataLength` object. We then proceeded to serialize and write both instances to the user's data:

```rust
sender_data.sent.push(mail.clone());
let data_length = DataLength {
    length: u32::try_from(get_instance_packed_len(&sender_data)?).unwrap(),
  };
data_length.serialize(&mut &mut sender_account.data.borrow_mut()[..offset])?;
sender_data.serialize(&mut &mut sender_account.data.borrow_mut()[offset..])?;
```

We did the same for the logic that handles the addition of the mail instance to the receiver's account.

With this change in how our code works, we need to edit our `process_init_account()` method, specificaly the part that reads and write data. In the `process_init_account()` method, edit:

```rust
let mail_account = MailAccount {
  inbox: vec![welcome_mail],
  sent: Vec::new(),
};

mail_account.serialize(&mut &mut account.data.borrow_mut()[..])?;
```

to:

```rust
let mail_account = MailAccount {
  inbox: vec![welcome_mail],
  sent: Vec::new(),
};

let data_length = DataLength {
  length: u32::try_from(get_instance_packed_len(&mail_account)?).unwrap(),
};

let offset: usize = 4;
data_length.serialize(&mut &mut account.data.borrow_mut()[..offset])?;
mail_account.serialize(&mut &mut account.data.borrow_mut()[offset..])?;
```

With this we've come to the end of the program chapter. In the next chapter, we will write the code for the User Interface, deploy the program to a local Solana cluster (test validator) as well as the Devnet, then connect them to complete the project.

Note: The code in the Github repo won't be an exact match of the code in the tutorial as some edits have been made to clean up the program. Please have a look at the Github repo for the latest code!

# Building the web UI

[todo](#todo)
