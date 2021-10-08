# Crowd Funding Platform
In this tutorial, we will discuss how to create a crowd-funding platform on the Solana blockchain. This tutorial only assumes the reader to know some basic knowledge of [React](https://reactjs.org/) app development. I will be explaining some basics of Rust lang, how to write the Solana program and how to invoke instructions from the front-end.


#### Requirements
- [Git](https://git-scm.com/downloads)
- [Solana CLI](https://docs.solana.com/cli/install-solana-cli-tools#use-solanas-install-tool)

#### Prerequisites
- React
- Javascript


#### Overview:
- Introduction to Rust
    - Basics datatypes
    - Control flow
    - Functions and Macros
    - Enums and match syntax
    - Cargo and borsh
- Solana program
    - What do we want in our program?
    - Where can we store data?
    - How do we create a map with unlimited storage in the Solana ecosystem?
    - Coding the program
    - Deploy solana program
- Front-end with Solana web3.js
    -  Writing functions to invoke instructions
    -  Fetching all campaigns

> If you already have good knowledge of Rust programming language. You can skip the `Introduction to Rust.` I would still recommend having a quick look at all the code blocks.

---

# Introduction to Rust
Rust is a multi-paradigm, high-level, general-purpose programming language designed for performance and safety, especially safe concurrency.

Rust code uses snake case as the conventional style for function and variable names.


###### [Installing](https://doc.Rust-lang.org/book/ch01-01-installation.html) Rust on your system
> Note: If you are using `vs-code` you have to install `matklad.rust-analyzer`. I have found it to work better than `Rust-lang.rust`. For code analysis, auto-completion and snippets.

Before we start the tutorial, we need to understand some basics of Rust. I have added the link to the [Rust book](https://doc.Rust-lang.org/book/ch00-00-introduction.html) pages if you want to read more about any topic.

## Basics [Data types](https://doc.Rust-lang.org/book/ch03-02-data-types.html) in Rust:
Rust has four primary scalar types: integers, floating-point numbers, Booleans, and characters.
Integers are `u8`,`u32`, `i32`, `i64`, `usize`, and the list goes on here basically `u` prefix suggests that we have an unsigned integer and the suffix number tell the number of bits. So `u8` is an unsigned 8-bit number(0 to 255).
|Length| Signed|	Unsigned|
|------- |-------|-------|
|8-bit|	i8|	u8|
|16-bit|	i16|	u16|
|32-bit|	i32	|u32|
|64-bit|	i64|	u64|
|128-bit	|i128|	u128|
|arch|	isize|	usize|

We have `f32` and `f64` for floating-point numbers. `bool` for booleans, and `char` for characters.
Rust has 2 types for strings, `str` and `String`. `String` is a growable, heap-allocated data structure. `str` is an immutable fixed-length string somewhere in memory.


## Creating a variable and [mutability](https://doc.Rust-lang.org/book/ch03-01-variables-and-mutability.html):
We can create a variable with the `let` keyword
```Rust
// The compiler identifies the Rvalue as i32, so it sets the type of variable to i32
let a=0;
```
we can also set the data type for a variable, eg.
```Rust
let a :u8 = 0;
```
In Rust, all the variables are immutable by default. Which means their value can not be changed once set.
And here comes the `mut` keyword. We can initialize the variable with `let mut` to have a mutable variable. eg.
```Rust
// This program will compile
let mut a = 0;
a=a+1;
a=100;
```

## [Control flow](https://doc.Rust-lang.org/book/ch03-05-control-flow.html):
We can use `if` `else` statement in Rust just like we can do in other language, here is a small program for us to understand the syntax.
```Rust
fn main(){
    let a=99;
    if a%2==0{
        println!("Even");
    }
    else{
        println!("Odd");
    }
}
```
We also have loops in Rust. We can create a loop with 3 keywords `loop`, `for`, and `while`. Since `for` is most common. Here is an example of it. You can checkout example for `loop` and `while` [here](https://doc.Rust-lang.org/book/ch03-05-control-flow.html#repeating-code-with-loop).

```Rust
fn main() {
    for i in 0..7 { // 0..7 is range expression including 0 excluding 7.
        println!("variable `i` is : {}", i);
    }
}
```

## [Functions](https://doc.Rust-lang.org/book/ch03-03-how-functions-work.html) and [Macros](https://doc.Rust-lang.org/book/ch19-06-macros.html):

Function definitions in Rust start with fn and have a set of parentheses after the function name. The curly brackets tell the compiler where the function body begins and ends.
```Rust
fn main() {
    another_function(5);
    another_function_with_x_and_y(1,2);
}

fn another_function(x: i32) {// input paramter and type
    println!("The value of x is: {}", x);
}
fn another_function_with_x_and_y(x: i32,y:i32) {
    println!("The value of x is: {} {}", x, y);
}
```
For this tutorial, we can assume macros are also functions. They end with `!`, like `println!` macro, `format!` macro, and `msg!` macro.

## [Enums](https://doc.Rust-lang.org/book/ch06-01-defining-an-enum.html) and the [match syntax](https://doc.Rust-lang.org/book/ch06-02-match.html):
Rust has enums. They are more than simple enums other languages provide. In Rust, we can even store data in the enums.
Here is the example of `Result enum`. 
> We are going to make use of the `Result` enum in our program.

```Rust
// Here the pub keyword means it is public.
// We have used generics T and E for data and error type
pub enum Result<T, E> { 
    Ok(T), 
    Err(E),
}
```

Here is an enum and match example with an explanation of each line.

```Rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Custom(i32),
}
// Creating a coin enums
// notice how some values have no parameter
// and how we can have an i32 value stored in `Custom`

let coin=Coin::Custom(30);
let coin2 =Coin::Nickel;
// We can create instances like this

let a = match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Custom(e) => e,
    };
// We can use the match syntax to know what type of coin we have  
// and set a corresponding value to the variable `a`.

assert_eq!(a,30);
// In this case a will be equal to 30 because coin is Custom with value 30.
```

## [Cargo](https://doc.Rust-lang.org/book/ch01-03-hello-cargo.html) and [Borsh](https://crates.io/crates/borsh)
Cargo is the Rust package manager. We use it to build our program and get dependencies. It also makes adding packages(crates) very easy.

Borsh stands for Binary Object Representation Serializer for Hashing. It is meant to be used in security-critical projects as it prioritizes consistency, safety, speed, and comes with a strict specification. 
We are using it for data serialization and deserialization.
\
\
\
This is all the Rust we would need to get started with our Solana program.

---- 

# Solana program

#### Setup
The first step is to install Rust on your system if you do not have it installed. [Here](https://doc.Rust-lang.org/book/ch01-01-installation.html) is how to do that.

1. We create a react app first. Open your projects directory in the terminal and run
```bash
npx create-react-app crowd-funding
``` 
This creates a react app for us.

2. Now, we will create our program.
In your projects directory.
```bash
cd crowd-funding
cargo new program --lib
```
This will create a `program` folder(An Rust project).

3. We will discuss the front-end side of the project later. Now we can open the `program` folder in vs-code. 
```bash
code program/
```
4. Create  `Xargo.toml` in the program directory.
In your `Xargo.toml`
```toml
[target.bpfel-unknown-unknown.dependencies.std]
features = []
```
5. Update your `Cargo.toml`
```toml
[package]
name = "program"
version = "0.1.0"
edition = "2018"

[dependencies]
solana-program = "1.7.14"
borsh = "0.9.1"
borsh-derive = "0.9.1"

[features]
no-entrypoint = []

[dev-dependencies]
solana-program-test = "1.7.14"
solana-sdk = "1.7.14"

[lib]
crate-type = ["cdylib", "lib"]
```

We have added all the dependencies we are going to need for our program. Run `cargo check` to get all the dependencies. We can now start working in `src/lib.rs` and start coding our program for the Solana blockchain.

## What do we want in our program?

Before we start writing the code, let us discuss what entry points our crowdfunding app should have.
- #### Create Crowd Funding Campaign.
We would need an entry point that anyone can use to create a crowdfunding campaign on our platform. We can have a name, description, and admin fields for it.

- #### Withdraw from the Campaign.
We would need an entry point for campaign administrators only, so they can withdraw funds.

- #### Donate to an Campaign.
We would need an entry point that can be invoked by anyone to donate to a specific Campaign.

These are all the entry points we are going to need for our project. Let us discuss how we will be creating these.

---

## Where can we store data?

Before we start we have to understand Solana programs don't have storage(`contract storage` you might be familiar with). Then how and where should we store our data.

#### program accounts
In Solana, data can only be stored in accounts. So we can create program-owned accounts to save the data.
> Note: The total number of bytes of data an account can have is fixed when we create an account.

One way to deal with this is to create an Account with very large storage. But If we do that the max data limit for an account is 10 mb. If we have so many users that we will run out of storage space. We must think of a way to have an infinite amount of storage.

#### How do we create a map with unlimited storage in the Solana ecosystem?

We can create as many program-owned accounts as we want, so the idea here is that we will have a size limit for every element in our map. And whenever we want to add a new element we will create a new program-owned account. Program-owned accounts are also called PDA(program-derived accounts).

## Coding the program
Now that we have discussed what we want to create. Let start coding.
Go ahead and open up the `program` folder in vs-code or your favorite ide.
File structure in the `program` directory should look  like this.
![file structure](../../../.gitbook/assets/solana-crowdfunding-program-files.png)

Go ahead and open up the `lib.rs` file in vscode editor, and let us add some boilerplate code first.

You can see the completed code on [github](https://github.com/SushantChandla/solana-crowd-funding/blob/main/program/src/lib.rs).


> Note: In Rust, we can pass variables as a reference with `&`.
We can get the value of a reference with `*` Operator.


```Rust 
// First we include what we are going to need in our program. 
// This  is the Rust style of importing things.
// Remember we added the dependencies in cargo.toml
// And from the `solana_program` crate we are including  all the required things.
use solana_program::{
    account_info::{next_account_info, AccountInfo},
    entrypoint,
    entrypoint::ProgramResult,
    msg,
    program_error::ProgramError,
    pubkey::Pubkey,
    rent::Rent,
    sysvar::Sysvar,
};


// Every solana program has one entry point
// And it is a convention to name it `process_instruction`. 
// It should take in program_id, accounts, instruction_data as parameters.
fn process_instruction(
    // program id is nothing but the id of this program on the solana network.
    program_id: &Pubkey,
    // When we invoke our program we can 
    // give meta data of all the account we 
    // want to work with.
    // As you can see it is a array of AccountInfo.
    // We can provide as many as we want.
    accounts: &[AccountInfo],
    // This is the data we want to process our instruction for.
    // It is a list of 8 bitunsigned integers(0..255).
    instruction_data: &[u8],
    
    // Here we specify the return type.
    // If you know a little bit of typescript. 
    // This was of writing types and returns types might we familiar to you.
) -> ProgramResult {
    
    // And then since we can't return null in Rust we pass `Ok(())` to make it compile
    // It means the program executed successfully.
    Ok(())
}


// Then we call the entry point macro to add `process_instruction` as our entry point to our program.
entrypoint!(process_instruction);
```
> Note: In Rust, if the last line doesn't end with `;` means our function is return-ing that.
Here the line `Ok(())` is equivalent to `return Ok(());` 

###### Go through the comments and code.
 
##### In the code, I have mentioned there is only one entry point in the Solana program.
But we want three as we discussed in the `What do we want in our program?` section. Let's fix this issue.
Have you noticed there is no limit to the instruction_data array? We are going to take advantage of that fact. We use the first element of the array to know what entry point we want to call.
Notice we can have 256 entry points like this in a single program (`u8` has a value of 0..255). Realistically we never do that if in case we want that many entry points for a project. It is better to deploy more programs.

Okay, let's do more coding...

```Rust
fn process_instruction(
    program_id: &Pubkey,
    accounts: &[AccountInfo],
    instruction_data: &[u8],
) -> ProgramResult {
    // We check if We have a instruction_data len greater than 0 if it is not we do not want to procced.
    // So we return Error with InvalidInstructionData message.
     if instruction_data.len() == 0 {
        return Err(ProgramError::InvalidInstructionData);
    }
    /// Now we just check and call the function for each of them.
    // I have choosen 0 for create_campaign,
    // 1 for withdraw
    // 2 for donate.
    if instruction_data[0] == 0 {
        return create_campaign(
            program_id,
            accounts,
            /// Notice we pass program_id and accounts as they where 
            // but we pass a reference to slice of [instruction_data]. 
            /// we do not want the first element in any of our functions.
            &instruction_data[1..instruction_data.len()],
        );
    } else if instruction_data[0] == 1 {
        return withdraw(
            program_id,
            accounts,
            &instruction_data[1..instruction_data.len()],
        );
    } else if instruction_data[0] == 2 {
        return donate(
            program_id,
            accounts,
            &instruction_data[1..instruction_data.len()],
        );
    }

    /// If instruction_data doesn't match we give an error.
    // Note I have used msg!() macro and passed a string here. 
    // It is good to do this as this would 
    // also get printed in the console window
    // if a program fails.
    msg!("Didn't found the entrypoint required");
    Err(ProgramError::InvalidInstructionData)
}

entrypoint!(process_instruction);


/// Here, I have created the function for every action we want to do in our program.
/// They take in the same parameters as process_intruction and also have the same return type.

fn create_campaign(
    program_id: &Pubkey,
    accounts: &[AccountInfo],
    instruction_data: &[u8],
) -> ProgramResult {
    Ok(())
}

fn withdraw(
    program_id: &Pubkey,
    accounts: &[AccountInfo],
    instruction_data: &[u8],
) -> ProgramResult {
    Ok(())
}

fn donate(
    program_id: &Pubkey, 
    accounts: &[AccountInfo], 
    _instruction_data: &[u8]
) -> ProgramResult {
    Ok(())
}
```

## Our CampaignDetails struct, Borsh, and code of create_campaign function.
We will create a struct in Rust. We have not discussed structs above so, I will explain them here. In Rust, we do not have class. If we want to store more than 1 variable (group variables) we create a struct.

```Rust
/// For an example, let us create human struct.
#[derive(Debug)]
struct Human {
    /// we can add all the fields in our struct here.
    /// we also have to specify the type of each variable.
    /// Like the [eyes_color] here is a `String` type object.
    pub eyes_color: String,
    pub name: String,
    pub height: i32,
}
```
Now you must be wondering what is the meaning of `#[derive(Debug)]`. It is interesting to note that we can derive some traits for our struct.

[Traits](https://doc.Rust-lang.org/Rust-by-example/trait.html) : A trait in Rust is a group of methods(of the struct) that are defined for a particular type.

Now let's code our `CampaignDetails` struct.
I have added the fields name, admin, description, image_link,amount_donated for our Campaign.

```Rust
#[derive(BorshSerialize, BorshDeserialize, Debug)]
struct CampaignDetails {
    pub admin: Pubkey,
    pub name: String,
    pub description: String,
    pub image_link: String,
    /// we will be using this to know the total amount 
    /// donated to a campaign.
    pub amount_donated: u64,
}
```

Here I have driven BorshSerialize and BorshDeserialize. BorshSerialize is used to convert the struct in an `array of u8`, which is the data we can store in Solana accounts. And is the data we have in `instruction_data` so we can deserialize that to a struct with the help of `BorshDeserialize`.

##### Code: 
At the top of the file import `BorshSerialize` and `BorshDeserialize` from the `Borsh` Crate.
```Rust
use borsh::{BorshDeserialize, BorshSerialize};
```
Now let's add the code of our `create_campaign` function and the `CampaignDetails` Struct.
I have added an explanation to each line in the code.
```Rust 
entrypoint!(process_instruction);

/// We are creating the struct;.
#[derive(BorshSerialize, BorshDeserialize, Debug)]
struct CampaignDetails {
    pub admin: Pubkey,
    pub name: String,
    pub description: String,
    pub image_link: String,
    pub amount_donated: u64,
}
// All the fields are public, which means we can use the `.`  Operator to get/set the values of the fields.

fn create_campaign(
    program_id: &Pubkey,
    accounts: &[AccountInfo],
    instruction_data: &[u8],
) -> ProgramResult {

    /// We create a iterator on accounts
    /// accounts parameter is the array of accounts related to this entrypoint
    let accounts_iter = &mut accounts.iter();
```
 We can use the `next_account_info` function to get an account from the array. This function returns a result enum. We can use `?` Operator on the result enum to get the value. If in case of an error the `?` Operator will chain the error, and our program will return the same error which was returned by next_account_info.


 Solana programs can only write data on a program-owned account.
 > Note: `writing_account` is a program-owned account.
```Rust   
    /// Writing account or we can call it program account.
    /// This is an account we will create in our front-end.
    /// This account should br owned by the solana program.
    let writing_account = next_account_info(accounts_iter)?;

    /// Account of the person creating the campaign.
    let creator_account = next_account_info(accounts_iter)?;

    // Now to allow transactions we want the creator account to sign the transaction.
    if !creator_account.is_signer {
        msg!("creator_account should be signer");
        return Err(ProgramError::IncorrectProgramId);
    }
    /// We want to write in this account so we want its owner by the program.
    if writing_account.owner != program_id {
        msg!("writing_account isn't owned by program");
        return Err(ProgramError::IncorrectProgramId);
    }
```
By deriving the trait `BorshDeserialize` in our `CampaignDetails` struct we have added a method `try_from_slice` which takes in the parameter `array of u8` and creates an object of CampaignDetails with it.
It gives us an enum of type results. We will use the `expect` method on result enums to and pass in the string which we can see in case of error.

```Rust
    let mut input_data = CampaignDetails::try_from_slice(&instruction_data)
        .expect("Instruction data serialization didn't worked");

    // Now I want that for a campaign created the only admin should be the one who created it.
    // You can add additional logical here to check things like
    // The image url should not be null
    // The name shouldn't be smaller than some specific length...
    if input_data.admin != *creator_account.key {
        msg!("Invaild instruction data");
        return Err(ProgramError::InvalidInstructionData);
    }
```
Solana accounts can have data, but size has to be specified when it is created.  We need to have a minimum balance to make it rent exempt. For this project, we create an account that already has a balance equal to the minimum balance.
You can read more about solana account and rent exemption [here](https://docs.solana.com/developing/programming-model/accounts#rent-exemption).
```Rust
    /// get the minimum balance we need in our program account.
    let rent_exemption = Rent::get()?.minimum_balance(writing_account.data_len());
    /// And we make sure our program account (`writing_account`) has that much lamports(balance).
    if **writing_account.lamports.borrow() < rent_exemption {
        msg!("The balance of writing_account should be more then rent_exemption");
        return Err(ProgramError::InsufficientFunds);
    }
    // Then we can set the initial amount donate to be zero.
    input_data.amount_donated=0;
```
If all goes well, we will write the writing_account. Here on our `input_data`(type `CampaignDetails`) variable, we have a method `serialize` this is because of the `BorshSerialize` derivation. We will use this and write the data in a writing account. At the end of the program, we can return `Ok(())`.
```Rust
    input_data.serialize(&mut &mut writing_account.data.borrow_mut()[..])?;

    Ok(())
}
```

Hurry! We are done with the first `create_campaign` function.
Let's continue writing the contract and write the withdraw function next.

## Withdraw function implementation.
For withdraw function also, we create a struct to get the input data. Here Input data is only the amount we want to withdraw.
```Rust
#[derive(BorshSerialize, BorshDeserialize, Debug)]
struct WithdrawRequest {
    pub amount: u64,
}
```
Now let us write the function.
```Rust
fn withdraw(
    program_id: &Pubkey,
    accounts: &[AccountInfo],
    instruction_data: &[u8],
) -> ProgramResult {
```
For the withdraw also we will create iterator and get `writing_account` (which is the program owned account) and `admin_account`.
```Rust
    let accounts_iter = &mut accounts.iter();
    let writing_account = next_account_info(accounts_iter)?;
    let admin_account = next_account_info(accounts_iter)?;
    
    // We check if the writing account is owned by program.
    if writing_account.owner != program_id {
        msg!("writing_account isn't owned by program");
        return Err(ProgramError::IncorrectProgramId);
    }
    // Admin account should be the signer in this trasaction.
    if !admin_account.is_signer {
        msg!("admin should be signer");
        return Err(ProgramError::IncorrectProgramId);
    }
```
Now we will get the data of campaign from the `writing_account`. Note that we stored this when we created the campaign with `create_campaign` function.
> Note: We are currently thinking of a scenario where a campaign is created and we want to withdraw some money from it.
```Rust
    // Just like we used the try_from_slice for 
    // instruction_data we will use it for the 
    // writing_account's data.
    let campaign_data = CampaignDetails::try_from_slice(*writing_account.data.borrow())
        .expect("Error deserialaizing data");

    // Then we check if the admin_account's public key is equal to
    // the public key we have stored in our campaign_data.
    if campaign_data.admin != *admin_account.key {
        msg!("Only the account admin can withdraw");
        return Err(ProgramError::InvalidAccountData);
    }


    // Here we make use of the struct we created.
    // We will get the amount of lamports admin wants to withdraw
    let input_data = WithdrawRequest::try_from_slice(&instruction_data)
        .expect("Instruction data serialization didn't worked");
```
We do not want the campaign to get deleted after a withdrawal. We want it to always have a minimum balance, So we calculate the `rent_exemption` and consider it.
```Rust
    let rent_exemption = Rent::get()?.minimum_balance(writing_account.data_len());

    /// We check if we have enough funds
    if **writing_account.lamports.borrow() - rent_exemption < input_data.amount {
        msg!("Insufficent balance");
        return Err(ProgramError::InsufficientFunds);
    }

    ///Tranfer balance
    /// We will decrease the balance of the program account, and increase the admin_account balance.
    **writing_account.try_borrow_mut_lamports()? -= input_data.amount;
    **admin_account.try_borrow_mut_lamports()? += input_data.amount;
    Ok(())
}
```

> Note: We can only decrease the balance of a program-owned account.

## Donate to a campaign
We want to donate to a campaign, but interestingly we can't decrease the balance of an account (not owned by our program) in our program. This means we can't just transfer the balance as we did in the withdraw function. 
[Solana policies](https://docs.solana.com/developing/programming-model/runtime#policy).
>An account not assigned to the program cannot have its balance decrease.

So for this, we will create a program-owned account in our front-end and then make the sol-token transaction.

```Rust
fn donate(
    program_id: &Pubkey,
    accounts: &[AccountInfo],
    _instruction_data: &[u8],
) -> ProgramResult {
    let accounts_iter = &mut accounts.iter();
    let writing_account = next_account_info(accounts_iter)?;
    let donator_program_account = next_account_info(accounts_iter)?;
    let donator = next_account_info(accounts_iter)?;
```
We get 3 accounts here, first is the program-owned account containing the data of campaign we want to donate to.
Then we have a `donator_program_account` which is also the program-owned account that only has the Lamport we would like to donate.
Then we have the account of the `donator`.
> Note: When we will create it in our front-end, although we do not want to store anything in it we will assign it some size. So that it automatically gets deleted after the sol token the transferred.

Then we want donator account to sign this transaction.
```Rust
  if writing_account.owner != program_id {
        msg!("writing_account isn't owned by program");
        return Err(ProgramError::IncorrectProgramId);
    }
    if donator_program_account.owner != program_id {
        msg!("donator_program_account isn't owned by program");
        return Err(ProgramError::IncorrectProgramId);
    }
    if !donator.is_signer {
        msg!("donator should be signer");
        return Err(ProgramError::IncorrectProgramId);
    }
```
Here we get the campaign_data and we will increment the  `amount_donated`. As the total amount of data donated to this campaign will increased.
```Rust
 let mut campaign_data = CampaignDetails::try_from_slice(*writing_account.data.borrow())
        .expect("Error deserialaizing data");

    campaign_data.amount_donated += **donator_program_account.lamports.borrow();
```
Then we do the actual transaction. Note that the donator_program_account is owned by program so it can decrease it's lamports. no sweat.
```Rust
 **writing_account.try_borrow_mut_lamports()? += **donator_program_account.lamports.borrow();
 **donator_program_account.try_borrow_mut_lamports()? = 0;
```
Then at the end of the program we will write the new updated campaign_data to writing_account's data field. And return `Ok(())`.
```Rust
    campaign_data.serialize(&mut &mut writing_account.data.borrow_mut()[..])?;

    Ok(())
}
``` 

Hurray, We have completed our Solana program, Now we can go ahead and deploy it.

## Deploy solana program.
We are going to deploy on devnet.
Solana Blockchain works on a [BPF system](https://docs.solana.com/developing/on-chain-programs/overview#berkeley-packet-filter-bpf), so we will compile our program for `bpf` machine.
We can use the handly package manager `cargo` to do it.
```bash
cargo build-bpf --manifest-path=Cargo.toml --bpf-out-dir=dist/program
```
We can use this command to create a build. In this command, the manifest-path should be the path of your `Cargo.toml` file.
We will get our output in `dist/program` directory.
> If you will run this command in `program` directory you will see `dist/program` directory added.
![program_complied-bpf](../../../.gitbook/assets/solana-crowdfunding-program-complied-bpf.png)

Now that we have compiled our program we can deploy it. 

You will need the [Solana-cli](https://docs.solana.com/cli/install-solana-cli-tools#use-solanas-install-tool).

We will create a new Solana account to deploy the program. We can do it by 
```bash
solana-keygen new -o keypair.json
```
This command will generate a keypair.json file.
> Important Note: Please do not commit your keypair.json on git. And keep them safe with you.

You will get a public key from this command.
Now we are going to request an airdrop for the sol tokens on `devnet`.
This command will add sol tokens to the account
```bash
solana airdrop 1 <YourPublicKey> --url https://api.devnet.solana.com 
```
Example
```bash
solana airdrop 1 DGqXoguiJnAy8ExJe9NuZpWrnQMCV14SdEdiMEdCfpmB --url https://api.devnet.solana.com 
```
> If you are getting `Error: RPC request error: cluster version query failed: error sending request for url (https://api.devnet.solana.com/): error trying to connect: dns error: failed to lookup address information: nodename nor servname provided, or not known` This error please consider switching your primary server over to one of Google, DNSWatch, OpenDNS, SAFEDNS, Dyn, Yandex, AdGuard, Cloudflare.

If you get `insufficient balance` while deployment, you can re-run this command to airdrop funds in devnet.

Now we will use 
```bash
solana deploy --keypair keypair.json dist/program/program.so --url https://api.devnet.solana.com 
```
command to deploy. Note that the path of `keypair.json` and `dist/program/program.so` might be different in your case.
Please check and then run the command.

Hurray! we have deployed our program.
We will get the program id as output.
```text
Program Id: 286rapsUbvDe1ZgBeNhp37YHvEPwWPTr4Bkce4oMpUKT
```
We can verify this by checking on [solana explorer for devnet](https://explorer.solana.com/?cluster=devnet). 
We can search our program id here.

Hurray! We Have completed everything with Rust and successfully deployed our program. Now, let us move forward and build the react app.

# Front-end with Solana web3.js
We have created a react app, so we can open the `crowd-funding` directory in our vs-code.
This is not a react tutorial, so I will not go into the details of react. But I will be explaining what we are going to do.

Let's first clean our project. We will remove `setupTests.js`, `reportWebVitals.js`, `logo.svg` and `app.test.js`. And remove the usage of `reportWebVitals.js` in index.js.
Then your project should like
![front-end-project](../../../.gitbook/assets/solana-crowdfunding-front-end-files.png)

We will create a basic UI for our app. I have used sementic-ui to do that.
If you want the UI part only and continue coding the Solana(web.js)  library you can use the UI template I created for you from [github ui branch](https://github.com/SushantChandla/solana-crowd-funding/tree/ui).

### @solana/web3.js
Let us add the `@solana/web3.js` package.
```bash
yarn add @solana/web3.js
```
We will also use `@project-serum/sol-wallet-adapter` package. To connect our app with sollet wallet.
```bash
yarn add @project-serum/sol-wallet-adapter
```
And we will also need borsh for serialization and deserialization.
```bash
yarn add borsh
```
Let us create a `solana` folder in our `crowd-funding/src` directory. We will write all the solana related code in this folder so you can refrence it easily.
#### BoilerPlate
In `solana/index.js`
```js
import Wallet from "@project-serum/sol-wallet-adapter";
import {
    Connection,
    SystemProgram,
    Transaction,
    PublicKey,
    TransactionInstruction
} from "@solana/web3.js";
import { deserialize, serialize } from "borsh";
```
In solana we call networks clusters, if you have by any chance deployed the program on testnet/mainnet change cluster.
Also update the programId it should be the one you got after deploying your program on blockchain.
```js
const cluster = "https://api.devnet.solana.com";
const connection = new Connection(cluster, "confirmed");
const wallet = new Wallet("https://www.sollet.io", cluster);
const programId= new PublicKey(
	"286rapsUbvDe1ZgBeNhp37YHvEPwWPTr4Bkce4oMpUKT"
);
```
After this I also like to create 2 helper functions`setPayerAndBlockhashTrasaction` and `signAndSendTrasaction`.
The `setPayerAndBlockhashTransaction` take in `instructions` as parameters. `instructions` will contains all the set of instruction we want to perform on the blockchain.
>#### Note: If any of the instruction fails with a program error all the instructions in the transaction fails.
```js
export async function setPayerAndBlockhashTransaction( instructions) {
    const transaction = new Transaction();
    instructions.forEach(element => {
        transaction.add(element);
    });
    transaction.feePayer = wallet.publicKey;
    let hash = await connection.getRecentBlockhash();
    transaction.recentBlockhash = hash.blockhash;
    return transaction;
}
```
The `setPayerAndBlockhashTransaction` return us the `transaction` and we can pass it to the `signAndSendTransaction` function to make our trasaction.
```js
export async function signAndSendTransaction(transaction) {
    try {
        console.log("start signAndSendTransaction");
        let signedTrans = await wallet.signTransaction(transaction);
        console.log("signed transaction");
        let signature = await connection.sendRawTransaction(
            signedTrans.serialize()
        );
        console.log("end signAndSendTransaction");
        return signature;
    } catch (err) {
        console.log("signAndSendTransaction error", err);
        throw err;
    }
}
```
## Writing a function to invoke `create_campaign` instruction.
Before we start let use create a javascript implementation for the Rust struct `CampaignDetails`
We have created a class and we will call it `CampaignDetails`. For deserialization and serialization we have to create a schema for our class 
We will create map, And match the types of each field.
> Note : PubKey type is nothing but a u8 array with length 32.
```js
class CampaignDetails {
    constructor(properties) {
        Object.keys(properties).forEach((key) => {
            this[key] = properties[key];
        });
    }
    static schema = new Map([[CampaignDetails,
        {
            kind: 'struct',
            fields: [
                ['admin', [32]],
                ['name', 'string'],
                ['description', 'string'],
                ['image_link', 'string'],
                ['amount_donated', 'u64']]
        }]]);
}
```

Now we can write our `createCampaign` function.
This function take in  `name`,`description` and `image_link` as input parameter.

The first thing we will do it to add a call to `checkWallet` function.
```js
export async function createCampaign(
    name, description, image_link
) {
     await checkWallet();
```
###### checkWallet function:
We will check if the wallet is not connected or not. And connect if it isn't.
```js
async function checkWallet() {
    if (!wallet.connected()) {
        await wallet.connect();
    }
}
```

We will create a pubkey for our program account which will contain the data of a campaign, this is `writing_account` we have used in our program. 
```js
    const SEED = "abcdef" + Math.random().toString();
    let newAccount = await PublicKey.createWithSeed(
        wallet.publicKey,
        SEED,
        programId
    );
```
Now we have created a publicKey. Note that we have not given instruction to create an account yet.

Let us setup the campaignDetails we want to send to our program.
```js
    let campaign =new CampaignDetails({
        name: name,
        description: description,
        image_link: image_link,
        admin: wallet.publicKey.toBuffer(),
        amount_donated: 0
    })
```
we will convert the data to `Uint8Array`. Note that all the programs have `instruction_data` datatype an array of u8. 
And before we send this data remember we want the first element in our instruction_data to be (0,1,2) for calling different entry points.
We will set it to 0. As we want to call `create_campaign` instruction. 
```js
    let data = serialize(CampaignDetails.schema, campaign);
    let data_to_send = new Uint8Array([0, ...data]);
```
Now we have the data we want to send to our program.

We will fund it with the minimum number of lamports required to make it rent Exempt.
So we calculate the lamports required like this.
```js
    const lamports =
        (await connection.getMinimumBalanceForRentExemption(data.length));
```
Here we create the instruction to create our account we will pass in the pub key, the size of data it needs to store, initial lamports, and other parameters.
This is the first instruction we will create.
> Note: Only system program can create accounts. We are writing instruction for system program here and not our deployed program.
```js
    const createProgramAccount = SystemProgram.createAccountWithSeed({
        fromPubkey: wallet.publicKey,
        basePubkey: wallet.publicKey,
        seed: SEED,
        newAccountPubkey: newAccount,
        lamports: lamports,
        space: data.length,
        programId: programId,
    });
```
Now we can create the 2nd instruction we invoke our program that would write the data to the program account we just created in the above instruction.
In the keys parameter we will pass all account we want to send.
And in data(`instruction_data` for our program) we want to send,
we will pass programId we want to invoke. Note we have created programId global variable in this file already.
```js
    const instructionTOOurProgram = new TransactionInstruction({
         keys: [
            { pubkey: newAccount, isSigner: false, isWritable: true },
            { pubkey: wallet.publicKey, isSigner: true, isWritable: false },
        ],
        programId: programId,
        data: data_to_send,
    });
```
Now we have created the the instructions we want. We will pass them to our helper function `setPayerAndBlockhashTransaction` which would create a instance `Transaction` for our `instructions` which we can then pass to `signAndSendTransaction`.
```js
    const trans = await setPayerAndBlockhashTransaction(
        [createProgramAccount,
            instructionTOOurProgram]
    );
    const signature = await signAndSendTransaction(trans);
```
Now that is done we can confirm our transaction by passing the `signature` in `confirmTransaction` function.
```js
    const result = await connection.confirmTransaction(signature);
    console.log("end sendMessage", result);
}
```

Now we can go ahead and connect this function to our front-end with our Form.

So in out `form.js`
we can have a on submit, On calling this function we will see sollet dialog. Which would ask us to sign the trasaction.
```js
  const onSubmit = async (e) => {
        e.preventDefault();
        await createCampaign(name, description, image);
        setRoute(0);// we are updated this to change the ui.
    }
```
see final [`form.js`](https://github.com/SushantChandla/solana-crowd-funding/blob/main/src/components/Form.js) file.
Now we can add campaigns.

### Fetch All Campaigns

We have implemented a function to add campaigns, but we do not have any function to fetch all the campaigns. Let's make that.

We know we have added all the data in program accounts, so we can fetch all the program accounts by using `getprogramAccounts` on connection.
This will return us a list of account and their public keys.
Now we can deserialize the data of a account by using the borsh package.
And Then I am converting the data in a desirable list and returning it.
```js
export async function getAllCampaigns() {
    let accounts = await connection.getProgramAccounts(programId);
    let campaigns = []
    accounts.forEach((e) => {
        try {
            let campData = deserialize(CampaignDetails.schema, CampaignDetails, e.account.data);
            campaigns.push({
                pubId: e.pubkey,
                name: campData.name,
                description: campData.description,
                image_link: campData.image_link,
                amount_donated: campData.amount_donated,
                admin: campData.admin,
            });
        } catch (err) {
            console.log(err);
        }
    });
    return campaigns;
}
```

Then we can use this in `app.js` we can render cards with this data.
we will add
```js
useEffect(() => {
    getAllCampaigns().then((val) => {
      setCards(val);
      console.log(val);
    });
  }, []);
```
see final [`app.js`](https://github.com/SushantChandla/solana-crowd-funding/blob/main/src/App.js) file.

![Result](https://i.imgur.com/wjFVAaC.gif)

## we can write the donate function.

For donate, we will again create an program account and we will fund it with the amount we want to donate. As we discussed while writing our program it is not possible to decrease the balance of an account that is not owned by program.

Our function will take in the `campaignPubKey`, and `amount` as parameters.
`amount`: the amount of Solana token we want to donate.
`campaignPubKey`: public key of the account we want to send tokens to. This is the same account where the data related to this campaign is stored.
```js
export async function donateToCampaign(
    campaignPubKey, amount
) {
    await checkWallet();

    const SEED = "abcdef" + Math.random().toString();
    let newAccount = await PublicKey.createWithSeed(
        wallet.publicKey,
        SEED,
        programId
    );

    const createProgramAccount = SystemProgram.createAccountWithSeed({
        fromPubkey: wallet.publicKey,
        basePubkey: wallet.publicKey,
        seed: SEED,
        newAccountPubkey: newAccount,
        lamports: amount,
        space: 1,
        programId: programId,
    });
```
This is similar to `createCampaign` function.
Here I have set the space as `1`, because I want the account to get deleted when its balance becomes zero. And we are setting the initial lamports equal to the number of Solana tokens we want to donate.


Then We will pass 3 keys(Accounts) as our program need 3 accounts(see `donate` function implementation in the program).

And we are sending data an array, `[2]`. Because we want call donate function in program and we have mapped it to `2` value of first element of `instruction_data` array.
```js
   const instructionTOOurProgram = new TransactionInstruction({
        keys: [
            { pubkey: campaignPubKey, isSigner: false, isWritable: true },
            { pubkey: newAccount, isSigner: false, },
            { pubkey: wallet.publicKey, isSigner: true, }
        ],
        programId: programId,
        data:new Uint8Array([2])
    });

    const trans = await setPayerAndBlockhashTransaction(
        [createProgramAccount, instructionTOOurProgram]
    );
    const signature = await signAndSendTransaction(trans);
    const result = await connection.confirmTransaction(signature);
    console.log("end sendMessage", result);
}
```
We have created instructions to our program and then called the `setPayerAndBlockhashTransaction`, `signAndSendTransaction` and `confirmTransaction` to send and confirm the transaction just like we did in the `createCampaign` function.

Let connect this function with the ui 
In `cards.js`
update the onDonate
```js
   const onDonate = async (e) => {
        e.preventDefault();
        await donateToCampaign(data.id, amount);
        let newCards = await getAllCampaigns();
        setCards(newCards);
    }
```
See the final [`card.js`](https://github.com/SushantChandla/solana-crowd-funding/blob/main/src/components/Card.js) file.

\
![Result](https://i.imgur.com/0yahyH7.gif)

## Withdraw function.
Now we will write the withdraw function. For withdraw, we don't have to create a program account, and we will only pass one instruction.

But since we are using a `WithdrawRequest` struct in our program. We will have to create a class and schema for borsh Serialization.
So first we will setup that.
```js
class WithdrawRequest {
    constructor(properties) {
        Object.keys(properties).forEach((key) => {
            this[key] = properties[key];
        });
    }
    static schema = new Map([[WithdrawRequest,
        {
            kind: 'struct',
            fields: [
                ['amount', 'u64'],
            ]
        }]]);

}
```
We have created the schema with 'amount' as `u64` which is the datatype of variable in Rust.
Now let us write the actual function.
Our function will take in the parameter `campaignPubKey` and the `amount` we want to withdraw.
Then we will serialize the data. First, we will create the WithdrawRequest Object, and then with the help of schema, we have as a static member in `WithdrawRequest` class. 
```js
export async function withdraw(
    campaignPubKey, amount
) {
    await checkWallet();

 let withdrawRequest = new WithdrawRequest({ amount: amount });
    let data = serialize(WithdrawRequest.schema, withdrawRequest);
    let data_to_send = new Uint8Array([1, ...data]);
```
Then we will create the instruction and pass the data. Note that we are inserting `1` in our data to before sending.
```js
    const instructionTOOurProgram = new TransactionInstruction({
        keys: [
            { pubkey: campaignPubKey, isSigner: false, isWritable: true },
            { pubkey: wallet.publicKey, isSigner: true, }
        ],
        programId: programId,
        data: data_to_send
    });
```
This part of the code is same as the `donate` and `createCampaign` functions.
```js
    const trans = await setPayerAndBlockhashTransaction(
        [instructionTOOurProgram]
    );
    const signature = await signAndSendTransaction(trans);
    const result = await connection.confirmTransaction(signature);
    console.log("end sendMessage", result);
}
```
Connect the functions with UI
In `card.js`
```js
 const onWithdraw = async (e) => {
        e.preventDefault();
        try {
            await withdraw(data.id, amount);
            alert('Withdraw successful!');
        } catch (e) {
            console.log(e);
            alert("only admin can withdraw");
        }
        let newCards = await getAllCampaigns();
        setCards(newCards);

    }
```
See the final [`card.js`](https://github.com/SushantChandla/solana-crowd-funding/blob/main/src/components/Card.js) file.

And we're done! 

You can see the whole project on [github](https://github.com/SushantChandla/solana-crowd-funding).


# Conclusion

In this tutorial, you learned how to build a Crowd Funding app on Solana. We covered the on-chain program's code using the Rust programming language. We built the User Interface with React.

# About the Author
This tutorial was created by [Sushant Chandla](https://github.com/SushantChandla).