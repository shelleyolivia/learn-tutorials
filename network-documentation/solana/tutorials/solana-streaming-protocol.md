# Introduction
Real-time streaming can be loosely translated as a constant flow of assets from one wallet to another every second. It makes transactions much faster and also enables a trust-less environment. Subscription services or freelancers to maintain a trustless environment with their customers can use this type of transaction. There are numerous other applications you can check out the [SuperFluid](https://www.superfluid.finance/home) protocol build on the Ethereum network.

# Prerequisites

A good understanding of the [Rust](https://www.rust-lang.org/) programming language and [React](https://reactjs.org/) and [Redux](https://redux.js.org/) is required to grasp the contents of this tutorial.

# Requirements
The following software is required to complete this tutorial:
- Git, install it from [HERE](https://git-scm.com/downloads).
- Solana CLI, install it from [HERE](https://docs.solana.com/cli/install-solana-cli-tools#use-solanas-install-tool).
- Solana wallet 
- The Rust toolchain, install it from [HERE](https://www.rust-lang.org/tools/install).
- Node.js (v14.18.1+), install it from [HERE](https://nodejs.org/en/download/).
- Postgress, install it from [HERE](https://www.postgresql.org/)

# Building Sol Streaming Protocol
We will write the tutorial in 3 parts. First, we will write our Solana program then we will create a backend for our protocol, and in the end, we will connect it all with our front-end.

# Solana Program
Before coding, let us discuss what a streaming platform means and what instructions we need in our Solana program.
A Streaming protocol creates an escrow account that will keep track of the balance of both parties with the help of time. Initially, all the funds in the escrow account will be owned by the sender. With time, ownership of some funds will be transferred to the receiver. That means the receiver can withdraw those funds. 

We will need 3 instructions in our Solana program
- **To Create a Stream**: This can be used the the sender and they will be funding our escrow account, give information about the receiver and define start and end time for the Stream.
- **Withdraw funds**: This can be used by the receiver to withdraw the funds they have earned.
- **Cancel Stream**: This may be used by the sender to cancel a stream. This instruction will distribute the owned funds in the escrow account to senders and receiver accounts.

Let's now create our program open terminal in your projects folder and run the following command to create a rust project using the library template.

```text
cargo new --lib sol-stream-program 
```

We will be now able to see the `sol-stream-program` folder in our projects directory so we can open it open our code editor, I am using VS-Code for this tutorial you can use code editor's of our choice and in `Cargo.toml`, we can add the required dependencies.

```toml
[package]
name = "sol-stream-program"
version = "0.1.0"
edition = "2018"

[dependencies]
solana-program = "=1.8.1"
borsh = "0.9.1"
thiserror = "1.0.24"

[dev-dependencies]
solana-program-test = "=1.8.1"
solana-sdk = "=1.8.1"

[lib]
crate-type = ["cdylib", "lib"]
``` 

Now to get all the crates we can save this file and in the terminal run:

```text
cargo check
```

we will create our program in the following code structure:

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
```

The flow of a program using this structure looks like this:

1. Someone calls the entrypoint
2. The entrypoint forwards the arguments to the processor
3. The processor asks instruction.rs to decode the instruction_data argument from the entrypoint function.
4. Using the decoded data, the processor will now decide which processing function to use to process the request.
5. The processor may use state.rs to encode state into or decode the state of an account which has been passed into the entrypoint.

> `instruction.rs` defines the "API" of a program.

Now let us create the `entrypoint.rs`, `instruction.rs`, `processor.rs`, `state.rs`, and `error.rs` files in the `sol-stream-program/src` directory. We can register the newly created files in lib.rs by updating lib.rs to:

```rs
pub mod error;
pub mod instruction;
pub mod native_mint;
pub mod processor;
pub mod state;
pub mod entrypoint;
```

Now in `entrypoint.rs` let us add code for our entry point function and register it with `entrypoint!` macro:

```rs
//! Program entrypoint

use solana_program::{
    account_info::AccountInfo, entrypoint, entrypoint::ProgramResult,
    program_error::PrintProgramError, pubkey::Pubkey,
};

fn process_instruction(
    program_id: &Pubkey,
    accounts: &[AccountInfo],
    instruction_data: &[u8],
) -> ProgramResult {
    Ok(())
}
entrypoint!(process_instruction);
```

We are importing required structs and functions and macros from the `solana_program` crate. Then we create  a function that will take accounts, instruction_data, and program_id as input parameters and return ProgramResult and register this function with `entrypoint!` in the last line.

Now let us open `instruction.rs` and add the following code:

```rs
use borsh::BorshDeserialize;
use solana_program::program_error::ProgramError;

/// Instructions supported by the sol-streaming program.
#[repr(C)]
#[derive(Clone, Debug, PartialEq)]
pub enum StreamInstruction {
    /// Create a stream with a escrow account created and funded by sender
    /// account should have a total_lamport=admin_cut+program_rent_account+amount_to_send.
    ///
    /// Accounts expected:
    ///
    /// `[writable]` escrow account, it will hold all necessary info about the trade.
    /// `[signer]` sender account
    /// `[]` receiver account
    /// `[]` The rent sysvar
    /// `[]` Admin account
    CreateStream,

    /// Withdraw from a stream for receiver
    ///
    /// Accounts expected:
    ///
    /// `[writable]` escrow account, it will hold all necessary info about the trade.
    /// `[signer]` receiver account
    WithdrawFromStream,

    /// Close a stream and transfer tokens between sender and receiver.
    ///
    /// Accounts expected:
    ///
    /// `[writable]` escrow account, it will hold all necessary info about the trade.
    /// `[signer]` sender account
    /// `[]` receiver account
    CloseStream,
}
```

Note that `CreateStream` and `WithdrawFromStream` would require the , let us create structs for them in `state.rs`.
In `state.rs` add the following code:

```rs
use borsh::{BorshDeserialize, BorshSerialize};
use solana_program::{clock::UnixTimestamp, pubkey::Pubkey};

#[derive(Clone, Debug, PartialEq, BorshDeserialize, BorshSerialize)]
pub struct CreateStreamInput {
    pub start_time: UnixTimestamp,
    pub end_time: UnixTimestamp,
    pub receiver: Pubkey,
    pub lamports_withdrawn: u64,
    pub amount_second: u64,
}

#[derive(Clone, Debug, PartialEq, BorshDeserialize, BorshSerialize)]
pub struct WithdrawInput {
    pub amount: u64,
}
```

For `CreateStream` we will want `CreateStreamInput` struct:
- `start_time`: The [Unix timestamp](https://en.wikipedia.org/wiki/Unix_time) at which the stream will start.
- `end_time`: The [Unix timestamp](https://en.wikipedia.org/wiki/Unix_time) at which the stream will end.
- `receiver`: Public key of the receiver.
- `lamports_withdrawn`: We allow the receiver to withdraw Lamports when they have its ownership, we would also want to keep track of the Lamport withdrawn for calculation purposes.
- `amount_speed`: The num
ber of  Lamport transferred to receiver every second.

For `WithdrawFromStream` we will want the `WithdrawInput` struct: 
- `amount`: The amount of Lamport receiver wants to withdraw.

Now let us import these in `instruction.rs` and use them:

add this line at the very top of the file to import our structs:

```rs
use crate::state::{CreateStreamInput, WithdrawInput};
```

Update `CreateStream` and `WithdrawFromStream` to :

```rs
...
// CreateStream,
CreateStream(CreateStreamInput),
...
```

```rs
...
//WithdrawFromStream
WithdrawFromStream(WithdrawInput)
...
```

For reference, you can check the file [HERE](https://github.com/SushantChandla/sol-stream-program/blob/main/src/instruction.rs).

Now that our enums are complete, let us add a function to unpack the instruction given to our program. At the end of `instruction.rs`:

```rs
impl StreamInstruction {
    pub fn unpack(instruction_data: &[u8]) -> Result<Self, ProgramError> {
        let (tag, data) = instruction_data
            .split_first()
            .ok_or(ProgramError::InvalidInstructionData)?;
        match tag {
            1 => Ok(StreamInstruction::CreateStream(
                CreateStreamInput::try_from_slice(data)?,
            )),
            2 => Ok(StreamInstruction::WithdrawFromStream(
                WithdrawInput::try_from_slice(data)?,
            )),
            3 => Ok(StreamInstruction::CloseStream),
            _ => Err(ProgramError::InvalidInstructionData),
        }
    }
}
```
We have added a function to unpack data since there is only one entry point. We have used the first element of `insturction_data` as a tag. Then we have used the `BorshDerialization` Derivation to which provides us with the `try_from_slice` function to unpack data. We are using:
- Tag 1 -> Create Stream Instruction.
- Tag 2 -> Withdraw Instruction.
- Tag 3 -> Close Stream Instruction.

Returning an error otherwise. Now we can open the `processor.rs` file and add the logic for instructions in it.

```rs
use std::str::FromStr;

use crate::{
    instruction::StreamInstruction,
    state::{CreateStreamInput, StreamData, WithdrawInput},
};
use borsh::{BorshDeserialize, BorshSerialize};
use solana_program::{
    account_info::{next_account_info, AccountInfo},
    clock::Clock,
    entrypoint::ProgramResult,
    program_error::ProgramError,
    pubkey::Pubkey,
    sysvar::{rent::Rent, Sysvar},
};
pub struct Processor;

impl Processor {
    pub fn process(
        program_id: &Pubkey,
        accounts: &[AccountInfo],
        instruction_data: &[u8],
    ) -> ProgramResult {
        let instruction = StreamInstruction::unpack(instruction_data)?;

        match instruction {
            StreamInstruction::CreateStream(data) => todo!(),
            StreamInstruction::WithdrawFromStream(data) =>todo!(),
            StreamInstruction::CloseStream => todo!(),
        }
    }
}
```
We have imported the required struct and function from the `solana_program` crate. Then we will create a struct Processor and implement on Processor and create a function named `process` which would take 
- `program_id`: Program ID of program.
- `accounts`: Accounts passed in the transaction.
- `instruction_data`: Instruction data passed for instruction.

This function will return `ProgramResult`. In the function, we have unpacked the `instruction_data` using the unpack function we created on `StreamInstruction` enum. We have used `?`  operator to unwrap valid values or return the erroneous value, propagating them to the calling function. Then we have used a match statement on instruction and added `todo!()` macro for them.

Before we do that, let us update the `entrypoint.rs` file.

```rs
use crate::processor::Processor;
fn process_instruction(
    program_id: &Pubkey,
    accounts: &[AccountInfo],
    instruction_data: &[u8],
) -> ProgramResult {
    Processor::process(program_id, accounts, instruction_data)
}
```

We have imported the `Processor` and updated the function to call the `process` function passing the same arguments.
For reference, you can check the file [HERE](https://github.com/SushantChandla/sol-stream-program/blob/main/src/processor.rs).

Now let us go to `processor.rs` and remove the todos. For each instruction, we will create a function and write the logic in there.

we can update the code to:

```rs
impl Processor {
    pub fn process(
        program_id: &Pubkey,
        accounts: &[AccountInfo],
        instruction_data: &[u8],
    ) -> ProgramResult {
        let instruction = StreamInstruction::unpack(instruction_data)?;

        match instruction {
            StreamInstruction::CreateStream(data) => {
                Self::process_create_stream(program_id, accounts, data)
            }
            StreamInstruction::WithdrawFromStream(data) => {
                Self::process_withdraw(program_id, accounts, data)
            }
            StreamInstruction::CloseStream => Self::process_close(program_id, accounts),
        }
    }

    fn process_create_stream(
        _program_id: &Pubkey,
        accounts: &[AccountInfo],
        data: CreateStreamInput,
    ) -> ProgramResult {
        Ok(())
    }

    fn process_withdraw(
        _program_id: &Pubkey,
        accounts: &[AccountInfo],
        data: WithdrawInput,
    ) -> ProgramResult {
        Ok(())
    }

    fn process_close(_program_id: &Pubkey, accounts: &[AccountInfo]) -> ProgramResult {
        Ok(())
    }
}
```

We have created empty functions process_create_stream, process_withdraw, and process_close. The `process_create_stream` and `process_withdraw` have a parameter `data` which would be the struct `CreateStreamInput` and `WithdrawInput` respectively.


Now let us go open `errors.rs` and write some error that our program might return in some case.
In `errors.rs`:

```rs
use thiserror::Error;

use solana_program::program_error::ProgramError;

#[derive(Error, Debug, Copy, Clone)]
pub enum StreamError {
    #[error("Failed to parse the pubkey")]
    PubKeyParseError,
    #[error("Admin account invalid")]
    AdminAccountInvalid,
    #[error("Not enough lamports in account")]
    NotEnoughLamports,
    #[error("Start time or end time for the stream is invalid")]
    InvalidStartOrEndTime,
    #[error("Receiver does not own enough tokens")]
    WithdrawError,

}

impl From<StreamError> for ProgramError {
    fn from(e: StreamError) -> Self {
        ProgramError::Custom(e as u32)
    }
}
```

For reference, you can check the file [HERE](https://github.com/SushantChandla/sol-stream-program/blob/main/src/error.rs).

## Writing Instruction logic
Now we can open the `processor.rs` file again and, complete `process_create_stream`, `process_withdraw` and, `process_close` function.

**process_create_stream**

We will parse pub key for admin. I have used my pubkey here if you can use your public key here. We are making using `from_str` method. In case of error we return `PubKeyParseError` which we created in `errors.rs`.

Then we will get all the accounts. First we create a iterator and then we can use the `next_account_info` function to get all the account. We will store all the accounts with a corresponding variable name.

Now we can check if the admin account provide was correct or incorrect by comparing it's key with the pubkey we produce in our function. If it is incorrect we return a error.

```rs
// Updated at top of file.
use crate::{
    error::StreamError,
    instruction::StreamInstruction,
    state::{CreateStreamInput, StreamData, WithdrawInput},
};
    ...
    fn process_create_stream(
        _program_id: &Pubkey,
        accounts: &[AccountInfo],
        data: CreateStreamInput,
    ) -> ProgramResult {
        let admin_pub_key = match Pubkey::from_str("DGqXoguiJnAy8ExJe9NuZpWrnQMCV14SdEdiMEdCfpmB") {
            Ok(key) => key,
            Err(_) => return Err(StreamError::PubKeyParseError.into()),
        };

        let account_info_iter = &mut accounts.iter();
        let escrow_account = next_account_info(account_info_iter)?;
        let sender_account = next_account_info(account_info_iter)?;
        let receiver_account = next_account_info(account_info_iter)?;
        let admin_account = next_account_info(account_info_iter)?;

        if *admin_account.key != admin_pub_key {
            return Err(StreamError::AdminAccountInvalid.into());
        }
```

Now, we can make a transaction of 0.03 lamports which I have chosen arbitrarily and send those to admin account.

> Note: If the program fails, this transaction will not be refected.

Now, we can check the given instruction data.
We will check the start time shouldn't be smaller then end time and the start time shouldn't be less then the current time. We can get current unix_timestamp by using `Clock::get()?.unix_timestamp`. We will return `InvalidStartOrEndTime` error in case of failure.

Then we can check that the total Lamport deposited in the account should be equal to the amount we want to send + the minimum number of Rent required to create the account on the chain. We can get the minimum amount of balance required by using the `Rent::get()?.minimum_balance(len)` method. If this failed we can return the `NotEnoughLamports` error.

```rs
        // 0.03 sol token admin account fee
        // 300000000 Lamports = 0.03 sol
        **escrow_account.try_borrow_mut_lamports()? -= 300000000;
        **admin_account.try_borrow_mut_lamports()? += 300000000;

        if data.end_time <= data.start_time || data.start_time < Clock::get()?.unix_timestamp {
            return Err(StreamError::InvalidStartOrEndTime.into());
        }

        if data.amount_second * ((data.end_time - data.start_time) as u64)
            != **escrow_account.lamports.borrow()
                - Rent::get()?.minimum_balance(escrow_account.data_len())
        {
            return Err(StreamError::NotEnoughLamports.into());
        }
```

Then we will check who signed this transaction. And the pub key of the receiver is data is equal to the account provided to us.

```rs
        if !sender_account.is_signer {
            return Err(ProgramError::MissingRequiredSignature);
        }

        if *receiver_account.key != data.receiver {
            return Err(ProgramError::InvalidAccountData);
        }
```

Now we are all set to write the stream data to our program account. We will create a `StreamData` struct and store that in our escrow account.
In `state.rs` at the end add a new struct:

```rs
#[derive(Clone, Debug, PartialEq, BorshDeserialize, BorshSerialize)]
pub struct StreamData {
    pub start_time: UnixTimestamp,
    pub end_time: UnixTimestamp,
    pub receiver: Pubkey,
    pub lamports_withdrawn: u64,
    pub amount_second: u64,
    pub sender: Pubkey,
}

impl StreamData {
    pub fn new(data: CreateStreamInput, sender: Pubkey) -> Self {
        StreamData {
            start_time: data.start_time,
            end_time: data.end_time,
            receiver: data.receiver,
            lamports_withdrawn: 0,
            amount_second: data.amount_second,
            sender,
        }
    }
}
```

For reference, you can check the file [HERE](https://github.com/SushantChandla/sol-stream-program/blob/main/src/state.rs).

We have added a method new to create an instance of the struct with the help of `CreateStreamInput` and Sender's pub key.

Now let's jump back to our function in the `processor.rs` file and complete the function.

```rs
       let escrow_data = StreamData::new(data, *sender_account.key);

        escrow_data.serialize(&mut &mut escrow_account.data.borrow_mut()[..])?;
        Ok(())
    }
```

We will first create the `escrow_data` and then with the help of borsh, we have the `serialize` method we can use to write data to `escrow_account`. We complete the function by returning `Ok(())` at the end of the function.

**process_withdraw**

We have stored accounts into variables just like we did in the `process_create_stream` function. Then we have deserialized data in the `escrow_account` not that this data is what we saved in the `process_create_stream` function. Then we perform a check that the receiver of this account is the singer and this `escrow_account` belongs to him. 

```rs
 fn process_withdraw(
        _program_id: &Pubkey,
        accounts: &[AccountInfo],
        data: WithdrawInput,
    ) -> ProgramResult {
        let account_info_iter = &mut accounts.iter();
        let escrow_account = next_account_info(account_info_iter)?;
        let receiver_account = next_account_info(account_info_iter)?;

        let mut escrow_data = StreamData::try_from_slice(&escrow_account.data.borrow())
            .expect("failed to serialize escrow data");

        if *receiver_account.key != escrow_data.receiver {
            return Err(ProgramError::IllegalOwner);
        }

        if !receiver_account.is_signer {
            return Err(ProgramError::MissingRequiredSignature);
        }
```

Then we can check if the user can withdraw the required Lamport or not. We will get the current time and calculate the total number of Lamport owned by them. By subtraction `lamports_withdrawn` we can keep track of the Lamport that are already withdrawn by the receiver.

```rs
        let time = Clock::get()?.unix_timestamp;

        let total_token_owned = escrow_data.amount_second
            * ((std::cmp::min(time, escrow_data.end_time) - escrow_data.start_time) as u64)
            - escrow_data.lamports_withdrawn;

        if data.amount > total_token_owned {
            return Err(StreamError::WithdrawError.into());
        }
```

Now we can proceed with the transaction and send the token to `receiver_account`. We will also make an increment in `lamported_withdrawn`. We finish the function by writing the new `escrow_data` to `escrow_account` and then returning `Ok(())`.

```rs
        **escrow_account.try_borrow_mut_lamports()? -= data.amount;
        **receiver_account.try_borrow_mut_lamports()? += data.amount;
        escrow_data.lamports_withdrawn += data.amount;

        escrow_data.serialize(&mut &mut escrow_account.data.borrow_mut()[..])?;
        Ok(())
    }
```

**process_close**

In this function also we will get the accounts and store them in variables. Then we get the `escrow_data` just like we did in the `process_withdraw` function. We then check if the `sender_account` is the owner of `escrow_account` and if `sender` has signed the transaction or not.

```rs
fn process_close(_program_id: &Pubkey, accounts: &[AccountInfo]) -> ProgramResult {
        let account_info_iter = &mut accounts.iter();
        let escrow_account = next_account_info(account_info_iter)?;
        let sender_account = next_account_info(account_info_iter)?;
        let receiver_account = next_account_info(account_info_iter)?;

        let mut escrow_data = StreamData::try_from_slice(&escrow_account.data.borrow())
            .expect("failed to serialize escrow data");

        if escrow_data.sender != *sender_account.key {
            return Err(ProgramError::IllegalOwner);
        }
        if !sender_account.is_signer {
            return Err(ProgramError::MissingRequiredSignature);
        }
```

We are closing the escrow account, so we want to transfer the funds to the receiver and sender which they own. So we can calculate the total tokens owned by the receiver.

```rs
        let time: i64 = Clock::get()?.unix_timestamp;
        let mut lamport_streamed_to_receiver: u64 = 0;

        if time > escrow_data.start_time {
            lamport_streamed_to_receiver = escrow_data.amount_second
                * ((std::cmp::min(time, escrow_data.end_time) - escrow_data.start_time) as u64)
                - escrow_data.lamports_withdrawn;
        }
```

Now we have the total Lamports that are owned by the receiver. We can send the remaining Lamports to the sender. At last, we set the escrow_account balance to 0. And then we can return `Ok(())`.

```rs
        **receiver_account.try_borrow_mut_lamports()? += lamport_streamed_to_receiver;
        escrow_data.lamports_withdrawn += lamport_streamed_to_receiver;
        **sender_account.try_borrow_mut_lamports()? += **escrow_account.lamports.borrow();

        **escrow_account.try_borrow_mut_lamports()? = 0;

        Ok(())
    }
```

For reference, you can check the file [HERE](https://github.com/SushantChandla/sol-stream-program/blob/main/src/processor.rs).

You can check the complete code of the Solana program on [GitHub](https://github.com/SushantChandla/sol-stream-program).

## Deploy the program.

Let us deploy the program. We can use the following command to create a build. In this command, the manifest-path should be the path of your `Cargo.toml` file. This will output the compiled program in Shared Object format (.so) in the `dist/program` directory

```text
cargo build-bpf --manifest-path=Cargo.toml --bpf-out-dir=dist/program
```

We will create a new Solana account to deploy the program. Run the following command:

```text
solana-keygen new -o keypair.json
```

The command will prompt you for a passphrase to secure the recovery seed phrase:

```text
Generating a new keypair

For added security, enter a BIP39 passphrase

NOTE! This passphrase improves security of the recovery seed phrase NOT the
keypair file itself, which is stored as insecure plain text

BIP39 Passphrase (empty for none):
```

You can choose a passphrase or leave it empty. Continuing will provide both the public key of the account and the seed phrase used to create the private key:

```text
Wrote new keypair to keypair.json
=====================================================================
pubkey: 7WQDnydTTtyb2DsTuuFpeu2bDxQdpZMRc4R6qja1UzP
=====================================================================
Save this seed phrase and your BIP39 passphrase to recover your new keypair:
lemon avoid all erase chair acid fire govern glue outside wheel clock
=====================================================================
```

Once complete you will have the `keypair.json` file, containing the private and public keys of a new Solana account. It is important to keep your keypair safe. Do not commit this file to a remote code repository. It is best to add this file to a `.gitignore` to prevent this from happening.

Now we are going to request an airdrop of SOL tokens on the Solana Devnet. This command will add 1 SOL token to the account:

```text
solana airdrop 1 <YourPublicKey> --url https://api.devnet.solana.com 
```

Example

```text
solana airdrop 1 7WQDnydTTtyb2DsTuuFpeu2bDxQdpZMRc4R6qja1UzP --url https://api.devnet.solana.com 
```

> If you are getting an error: "**RPC request error**: cluster version query failed: error sending request for url (https://api.devnet.solana.com/): error trying to connect: dns error: failed to lookup address information: nodename nor servname provided, or not known." - Please consider switching your primary DNS server over to one of Google, DNSWatch, OpenDNS, SAFEDNS, Dyn, Yandex, AdGuard, or Cloudflare.

If you get **insufficient balance** while deploying, you can re-run the command to airdrop funds on Devnet.

Now we will use the following command to deploy. Note that the path of `keypair.json` and `dist/program/program.so` might be different in your case. Please check and then run the command.

```text
solana deploy --keypair keypair.json dist/program/sol_stream_program.so --url https://api.devnet.solana.com
```

We will get the program id as output.

```text
Program Id: GoKSo1QVBx1jqeA15xSx6vJm3tYBM1586qp58VxXJayZ
```

We can verify this by checking on the [Solana Explorer for Devnet.](https://explorer.solana.com/?cluster=devnet)
We can search our program id here.

# Backend
We will be creating our backend with help of the [rocket web framework](https://rocket.rs/), but before we start writing the code for the backend it would be a great time to understand why we need a backend.

Let's think of a scenario where we do not have a backend for our protocol. We have stored all the data in the Program driven account(PDA) so for fetching all streams we can `getProgramAccounts` function provided in the `@solana/web3.js` package and then with the help of the `borsh` package we will deserialize the byte data. Then we can check among all the PDA's data which one of those belongs to a user i.e either they are sending or receiving. 

Now let us suppose we have made our app live and it has about 1000 users and all users have created 2 streams. So this means we will have 2000 Program driven accounts and then fetching all the accounts just to display the 2 streams will make our protocol slow and with the increase in users unuseable. 

We will we using our backend to index our PDA's and fix the scalability issue. Let us create a rust project again but this time with default template i.e binary (application) template.
Open console in your projects folder and run the following command:

```text
cargo new sol-stream-backend
```

This will generate `sol-stream-backend` directory. We can go ahead and open it on VS-Code now. For now, our project will look like:

```text
├─ src
│  ├─ main.rs -> contain
├─ .gitignore
├─ Cargo.lock
├─ Cargo.toml
```

Now open the `Cargo.toml` file and update it to add the required crates:

```toml
[package]
name = "sol-stream-backend"
version = "0.1.0"
edition = "2018"


[dependencies]
borsh = "0.9.1"
rocket = {version = "0.5.0-rc.1", features = ["tls", "json"]}
rocket_cors = {git = "https://github.com/lawliet89/rocket_cors", branch = "master"}
solana-client = "1.7.10"
serde = "1.0.130"
solana-sdk = "1.8.2"
```

- borsh: To serialize and derserialize data
- rocket_cors: To enable cross origin resource sharing
- solana-client: To  fetch all program accounts on the solana blockchain
- serde: For Json Serialization and Deserialization.

We are going to need all of these crate, to get them we can save this file and in the terminal run:

```text
cargo check
```

Now in the `main.rs` let us add code to create a "Hello world!" route with rocket.rs.

```rs
use rocket::{get, routes};

#[rocket::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let cors = rocket_cors::CorsOptions::default().to_cors()?;

    rocket::build()
        .mount("/", routes![index,])
        .attach(cors)
        .launch()
        .await?;

    Ok(())
}

#[get("/")]
fn index() -> &'static str {
    "Hello, world!"
}
```

Before I explain this code run the following command in the terminal:

```text
cargo run
```

It will compile and run the program then we can open `http://127.0.0.1:8000/` on our browser to see:
![browser image](../../../.gitbook/assets/solana-streaming-protocol-1.png)

Now let's see what code made this happen. In the first line, we have imported `routes` macro and `get` from rocket. We will use
`#[rocket::main]` on our function which will transfrom our function into a regular main function that internally initializes a Rocket-specific tokio runtime and runs the attributed async fn inside of it.

Then inside the function we will get default cors in a variable `cors`. The main line of code which attachs our `cors` and will start a server with a base `"/"` and it will have routes which will be passed in the `routes!` macro. As you can see we have a route and the function for it is called `index` so I have passed it in the macro. Then we await this call and return `Ok(())` so it compiles.

Now let's see how to create a route with `rocket.rs` framework. We will first create a function and then using `route_attribute` macro rules we can generate a `get` route at `"/"` path.

> NOTE: You can stop the running server by using `cmd+c` on mac and `control+c` on windows.

Now let's see how to make a post request:
Add the top of the files change the `use` statement to:

```rs
use rocket::serde::json::Json;
use rocket::{get, post, routes};
use serde::Deserialize;
```
we can add this code at the bottom of the file:

```rs
#[derive(Deserialize)]
struct User {
    name: String,
    age: i32,
}

#[post("/user", data = "<user>")]
fn user_function(user: Json<User>) -> String {
    format!("Hello {}, age: {}", user.name, user.age)
}
```

We have first created a struct which contains all the variables which we are expecting when the post request will be made. Then in the function we can get the data in `user` input variable. and our function will return a formatted string with user's name and age.

Now our route is created but we are not using it. So let us add it in `routes!` macro. Update main function:

```rs
    rocket::build()
        // add `user_function` here in the list.
        .mount("/", routes![index,user_function])
        .attach(cors)
        .launch()
        .await?;
```

Now we can run the program again by using in a terminal:

```text
cargo run
```

We can test the route in tools like `Postman` or in the another terminal by running the following command (you should have curl installed for this):

```text
curl --data '{"name":"sushant","age":20}' --header "Content-Type: application/json"  http://127.0.0.1:8000/user
```

Output:

```text
Hello sushant, age: 20
```

If you want to learn more about `rocket.rs` you can check the docs [HERE.](https://rocket.rs/v0.4/guide/)

Lets now add some required routes to our app. We will structure our app in the following format:

![Rocket app File Structure](../../../.gitbook/assets/solana-streaming-protocol-2.png)

Create the missing files and add them in main.rs by adding at the top of the file:

```rs
mod solana;
mod models;
mod routes;
```

- solana.rs: We are going to add the solana related function in this file.
- models.rs: We will be using this to add `struct` we will need in the web app.
- routes.rs: We will use this file to all the route functions.

Let's start writing the code.

Firsly in `model.rs` lets add the `StreamData`. Add the following code:

```rs
use borsh::{BorshDeserialize, BorshSerialize};
use serde::ser::SerializeStruct;
use serde::Serialize;
use solana_sdk::clock::UnixTimestamp;
use solana_sdk::pubkey::Pubkey;

#[derive(Clone, Debug, PartialEq, BorshDeserialize, BorshSerialize)]
pub struct StreamData {
    pub start_time: UnixTimestamp,
    pub end_time: UnixTimestamp,
    pub receiver: Pubkey,
    pub lamports_withdrawn: u64,
    pub amount_second: u64,
    pub sender: Pubkey,
}

impl Serialize for StreamData {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: serde::Serializer,
    {
        let mut s = serializer.serialize_struct("PaymentStream", 7)?;
        s.serialize_field("end_time", &self.end_time)?;
        s.serialize_field("start_time", &self.start_time)?;
        s.serialize_field("amount_second", &self.amount_second)?;
        s.serialize_field(
            "lamports_withdrawn",
            &((self.end_time - self.start_time) * self.amount_second as i64),
        )?;
        s.serialize_field("lamports_withdrawn", &self.lamports_withdrawn)?;
        s.serialize_field("sender", &self.sender.to_string())?;
        s.serialize_field("from", &self.receiver.to_string())?;
        s.end()
    }
}
```

We have added the same model we had in our `program` with one more derivations. We have derived `Serialize` from `serde` to serialize data to json format so we can use that in our React application. We have choose to go for manual implemnation because we wanted to add one additional key-value pair `lamports_withdrawn` and use `to_string` function on the `PubKey` type. 

**Desiel Orm**
