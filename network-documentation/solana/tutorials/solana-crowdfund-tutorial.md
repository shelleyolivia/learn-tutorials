# Crowd Funding Platform
In this tutorial we will discuss how to create a crowd funding platform on solana blockchain. This tutorial only assumes the reader to know some basic knowledge of react app development. I will be explaning everthing about solana, some basics of rust and how to call entry points from front end.


#### Overview:
- Introduction to rust
    - Basics types in rust
    - Control flow
    - Functions and Macros
    - Enums and match syntax
    - Cargo and borsh
- Solana Program
    - What we want in our program?
    - Where can we store data?
    - How do we create a map with unlimited storage in solana eco system?
    - Writing create an Crowd fund campaign Instruction
    - Writing create an withdraw fund campaign Instruction
    - Writing donate to a Crowd fund campaign Instruction
    - Deploy solana program.
- Frontend with solana web3.js
    -  Connecting wallet and Encoding and decoding with Borsh.
    -  Calling entrypoint
- Tips on scaling

## Introduction to Rust
Rust is a multi-paradigm, high-level, general-purpose programming language designed for performance and safety, especially safe concurrency.

Rust code uses snake case as the conventional style for function and variable names.

> Note: If you already have an idea of rust lang you can feel free to skip this and move on to the Solana program. Though I would recommend having a quick look on all the code blocks.


###### [Installing](https://doc.rust-lang.org/book/ch01-01-installation.html) rust on your system
> Note: If you are using vs-code you have to install matklad.rust-analyzer . I have found it to work better then rust-lang.rust.

For this tutorial, we will try to understand some basic of rust. I have added the link to the [rust book](https://doc.rust-lang.org/book/ch00-00-introduction.html) page if you want to read more about these topics.

##### Basics [Data types](https://doc.rust-lang.org/book/ch03-02-data-types.html) in rust:
Rust has four primary scalar types: integers, floating-point numbers, Booleans, and characters.
Integers can be represented with `u8`,`u32`,`i32`,`i64` , `usize` and the list goes on here basically `u` prefix suggest that we have a unsigned integer and the suffix number tell the no of bits. So `u8` is unsigned 8 bit number(0 to 255).
|Length| Signed|	Unsigned|
|------- |-------|-------|
|8-bit|	i8|	u8|
|16-bit|	i16|	u16|
|32-bit|	i32	|u32|
|64-bit|	i64|	u64|
|128-bit	|i128|	u128|
|arch|	isize|	usize|

We have `f32` and `f64` for floating point number. `bool` for boolens and `char` for characters.
Rust does have 2 types of Strings `str` and `String`. `String` is a growable, heap-allocated data structure whereas `str` is an immutable fixed-length string somewhere in memory.


##### Creating a variable and [mutability](https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html):
We can create a variable with `let` keyword
```rust
// the compiler identifies the left parameter to be
// an i32, so it sets the type of variable to i32
let a=0;
```
we can also set the data type for variable, eg.
```rust
let a :u8 = 0;
```
In rust all the variables are immutable my default, which means there value can't be changed once set.
And here comes the `mut` keyword to rescue.we can intitalize variable with `let mut` to have mutable variable. eg.
```rust
let mut a =0;
a=a+1;
a=100;
```

##### [Control flow](https://doc.rust-lang.org/book/ch03-05-control-flow.html):
We can use if else statement in rust just like we can do in any other language, here is a small program for you to understand the syntax.
```rust
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
We also have loops in rust, we can create loop with 3 keywords `loop` , `for` and `while`. Since `for` is most common here is an example for it. You can checkout example for `loop` and `while` [here](https://doc.rust-lang.org/book/ch03-05-control-flow.html#repeating-code-with-loop).

```rust
fn main() {
    for i in 0..7 { // 0..7 is range expression including 0 excluding 7.
        println!("variable `i` is : {}", i);
    }
}
```

##### [Functions](https://doc.rust-lang.org/book/ch03-03-how-functions-work.html) and [Macros](https://doc.rust-lang.org/book/ch19-06-macros.html):

Function definitions in Rust start with fn and have a set of parentheses after the function name. The curly brackets tell the compiler where the function body begins and ends.
```rust
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
For this tutotials we will assume macros are also function they just end with !, like `println!` macro, `format!` macro and `msg!` macro.

##### [Enums](https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html) and the [match syntax](https://doc.rust-lang.org/book/ch06-02-match.html):
This rust has enums but they are quite more then simple enums other languages provide. In rust we can even store data in the enums.
Here is the example of `Result enums`. Note this will be widely used the our solana program too.

```rust
// Here the pub keyword means it is public.
// We have used generics T and E for data and error type
pub enum Result<T, E> { 
    Ok(T), 
    Err(E),
}
```

Here is a enum and match example with explanation of each line
```rust
// Creating a coin enums
// notice how enums can have no parameter
// and how we have a i32 value stored in for [Quarter]
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(i32),
}
// We can create instances like this
let coin=Coin::Quarter(30);
let coin2 =Coin::Nickel;

// we can use the match syntax to see, what type of coin we 
// have and set a corresponding value to the variable a.
let a = match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(a) => a,
    };
// In this case we because coin is Quarter with value 30
// a will be equal to 30.
assert_eq!(a,30);
```

##### [Cargo](https://doc.rust-lang.org/book/ch01-03-hello-cargo.html) and [Borsh](https://crates.io/crates/borsh)
Cargo is the Rust package manager. We will be using it to test and build our program. It also makes adding packages(in rust ecosystem we call them crates) very easy.

Borsh stands for Binary Object Representation Serializer for Hashing. It is meant to be used in security-critical projects as it prioritizes consistency, safety, speed, and comes with a strict specification. 
We using it for data serialization and deserilization.
\
\
\
This is all the Rust we would need to get started with our solana smart program.

---- 

# Solana Program

##### Setup
First step is to install rust on your system. If you don't have it installed [here](https://doc.rust-lang.org/book/ch01-01-installation.html) is how to do that.

1. We will create a react app first. Open your projects directory in terminal and run
```bash
npx create-react-app crowd-funding
```
This will create a react app for us.

2. We will create our program for this.
In you projects directory.
```bash
cd crowd-funding
cargo new program --lib
```
This will create a `program` folder in `crowd-funding` directory.

3. We will discuss the frontend side of the project in 3rd section. Now we can open the program in vs-code.
```bash
code program/
```
4. Create  `Xargo.toml` in program directory.
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
And run `cargo check` to get all the dependencies.
We have added all the dependencies we are going to need for our program. We can now start working in `src/lib.rs` and start writing our smart program for solana blockchain.

### What we want in our program?

Before we start writing the code lets discuss what entrypoints our crowd funding app should have.
- #### Create Crowd Funding Campaign.
We would need a entry point with which anyone can create a crowd funding campaign on our platform. We can have a name, small description and an admin for it. And anyone can invoked this entrypoint.

- #### Withdraw from the Campaign.
We want a entry point for campaign admin's only so they can withdraw funds.

- #### Donate to an Campaign.
We want a entry point which can be invoked by anyone to donate to a specific Campaign.

These are all the entrypoints we are going to need for our project. Lets discuss how we will be creating these.

---

### Where can we store data?

Before we start we have to understand solana programs don't have storage(contract storage you might be familiar with). Then how and where should be store our data.

##### Program accounts
In solana data can only we stored in accounts. So we can create Program owned accounts to save the data.
> Note: The total no of bytes of data a account can store is fixed When we create an account.

One way to deal with this is to create a Account with very large storage. But If we do that we would still be limited at a point.
What if we have so many users that we run out of store. We must think a way to have infinite amount of storage.

#### How do we create a map with unlimited storage in solana eco system?

We can create as many program owned account as we want, so the idea here is that we will have a will have a size limit for every element in our map. And when ever we want to add a new Element we will create a new Program account. Sounds cool right.

### Writing create an Crowd fund campaign Instruction
Now that we have discussed what we want to create let just into coding.
Go ahead and open up the `program` folder in vs-code or your favorite ide.
File structure in the `program` directory should look  like this.
![file structure](../../../.gitbook/assets/solana-crowdfunding-program-files.png)

Go ahead and open up the `lib.rs` file in vscode editor and lets add some boilerplate code first.
> Note: In rust we will pass variables as refrences with `&`.
We can get the value of a refrence with `*` Operator.
```rust 
// First we include what we are going to need in our program. 
// This  is the rust style of importing things.
//Remember we added the dependance(we should call it crate though) in cargo.toml
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
// And it should take in program_id, accounts, instruction_data as paramters.
fn process_instruction(
    // Program id is nothing but the id of this program on the solana network.
    program_id: &Pubkey,
    // When we invoke our program we can 
    // give meta data of all the account we 
    // want to work with.
    // As you can see it is a list of AccountInfo.
    // We can provide as many as we want.
    accounts: &[AccountInfo],
    // This is the data we want to process our instruction for.
    // It is a list of (0..255) unsigned integers.
    instruction_data: &[u8],
    

    // Here we specify the return type.
    // If you know a little bit of typescript. This was of writing types and returns types might we familiar to you.
) -> ProgramResult {
    
    
    // And then since we can't return null in rust we pass `Ok(())` to make it compile
    // It basically means the program executed successfully.
    Ok(())
}


// Then we call the entrypoint macro to add `process_instruction` as our entrypoint to program.
entrypoint!(process_instruction);
```
> Note: In rust if the last line doesn't end with `;` this means we are returning that.
Here the line `Ok(())` is equivalent to `return Ok(());` 

###### Please go through the comments and code.
 
##### In the code I have mentioned there is only one entrypoint in solana program.
But we want three as, I discussed in `What we want in our program?` section.
Lets overcome this issue.
Have you noticed there is no limit to the instruction_data array. We are going to take advantage of that fact.We are going to use the first element of the array to know what entrypoint we will call.
Notice we can have 256 entrypoints like this in a single program (u8 has a value of 0..255). Realisticaly we never do that if we want that many entrypoints for a project it is better to deploy more number of programs.

Okay, now that we have a solution for this program lets do more coding coding...

```rust
fn process_instruction(
    program_id: &Pubkey,
    accounts: &[AccountInfo],
    instruction_data: &[u8],
) -> ProgramResult {
    // As we have discussed we will use first element to get the paramters.
    // we check if We have a instruction_data len greater then 0 if it is not we do not want to procced.
    // So we return Error with InvalidInstructionData message.
     if instruction_data.len() == 0 {
        return Err(ProgramError::InvalidInstructionData);
    }
    /// now we just check and call the function for each of them.
    // I have choosen 0 for create_campaign,
    // 1 for withdraw
    // 2 for donate.
    if instruction_data[0] == 0 {
        return create_campaign(
            program_id,
            accounts,
            /// Notice we pass program_id and accounts as they where but we pass a reference to slice of 
            /// [instruction_data]. 
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


/// Here I have create the the function for every action we want to do in out program.
/// They take in the same paramters  as process_intruction and also have the same return type.

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

fn donate(program_id: &Pubkey, accounts: &[AccountInfo], _instruction_data: &[u8]) -> ProgramResult {
    Ok(())
}
```

##### Our CampaignDetails struct, Borsh and code of create_campaign function.
We will create a struct in rust, we have not discussed struct above so I will explain them here. In rust we do not have class. If we want to store more then 1 variable (group variable) we create a struct.
```rust
/// For an example, let use create human struct.
#[derive(Debug)]
struct Human {
    /// we add all the fields in our struct here.
    /// we also have to specify the type of each variable.
    /// Like the [eyes_color] here is a `String` type object.
    pub eyes_color: String,
    pub name: String,
    pub height: i32,
}
```
Now you must be wondering what is the the meaning of `#[derive(Debug)]`. It is intersting to note that we can derive some traits for our struct.
[Traits](https://doc.rust-lang.org/rust-by-example/trait.html) : A trait in Rust is a group of methods(of struct) that are defined for a particular type.

Okay, Now lets see what is `CampaignDetails` struct.
I have added the fields name, admin, description, image_link,amount_donated for a Campaign.

```rust
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

Here I have drived BorshSerialize and BorshDeserialize. BorshSerialize if used to convert the struct in a array of u8.
Which is the type of data we can store in a solana account. And is the data we get a `instruction_data` so we can deserialize that to a struct with the help of `BorshDeserialize`.

###### Code: 
At the top of the file import `BorshSerialize` and `BorshDeserialize` from the `Borsh` Crate.
```rust
use borsh::{BorshDeserialize, BorshSerialize};
```
Now lets add the code of our `create_campaign` function and the `CampaignDetails` Struct.
I have added explanation to each line in the code.
```rust 
entrypoint!(process_instruction);

/// We are creating the struct;.
#[derive(BorshSerialize, BorshDeserialize, Debug)]
struct CampaignDetails {
    // We have a admin with data type pub key.
    pub admin: Pubkey,
    pub name: String,
    pub description: String,
    pub image_link: String,
    pub amount_donated: u64,
}
/// All the fields are public, which means we can use the `.` Operator to change/get the values of the fields.

fn create_campaign(
    program_id: &Pubkey,
    accounts: &[AccountInfo],
    instruction_data: &[u8],
) -> ProgramResult {

    /// We create a iterator on accounts
    /// Accounts is the list of account related to this entrypoint
    let accounts_iter = &mut accounts.iter();
```
 We can use the `next_account_info` function to get a account from the array. This function returns a result enum. We can use `?` Operator on result enum to get the value. If in case of error the `?` Operator will chain the error and our program will throw the same error which was thrown by next_account_info.


 Solana programs can only write data on a program owned account.
 > Note : `writing_account` is a program owned account.
```rust   
    /// Writing account or we can call it program account.
    /// This is account we will create in our frontend.
    /// This account should owned by the solana program.
    let writing_account = next_account_info(accounts_iter)?;

    /// Account of the person creating the campaign.
    let creator_account = next_account_info(accounts_iter)?;

    // Now to allow transactions we want the creator account to sign the transaction.
    if !creator_account.is_signer {
        msg!("creator_account should be signer");
        return Err(ProgramError::IncorrectProgramId);
    }
    /// We want to write in this account so we want its owner to be program.
    if writing_account.owner != program_id {
        msg!("Writter account isn't owned by program");
        return Err(ProgramError::IncorrectProgramId);
    }
```
By importing the trait `BorshDeserialize` in our `CampaignDetails` struct we have added a method to `try_from_slice` which take in the parameter of a array of u8. And creates a object of CampaignDetails with it.
It gives us a enum of type result. We will use the `expect` method on result enums to and pass in the string which we can see in case of error.
```rust
    let mut input_data = CampaignDetails::try_from_slice(&instruction_data)
        .expect("Instruction data serialization didn't worked");

    // Now I want that for a campaign created the only admin should be the one who created it.
    // You can add additional logical like this to check things like
    // The image url should not be null
    // The name shouldn't be small then some specific length..
    if input_data.admin != *creator_account.key {
        msg!("Invaild instruction data");
        return Err(ProgramError::InvalidInstructionData);
    }
```
In solana accounts can have a data, but it has to be specified when it is created and also we need to have a minimum balance to make it rent exempt. For this project we create account which already has a balance equal to minimum balance.
You can read more about solana account and rent exemption [here](https://docs.solana.com/developing/programming-model/accounts#rent-exemption).
```rust
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
If all goes will we will write the writing_account. Here on our `input_data`(type `CampaignDetails`) has a method `serialize` this is because of the `BorshSerialize` derivation. We Will use this and write the data in writing account. At the end of the program we can return `Ok(())`.
```rust
    input_data.serialize(&mut &mut writing_account.data.borrow_mut()[..])?;

    Ok(())
}
```

Hurry! We are done with the first `create_campaign` function.
Lets continue writing the contract and write the withdraw function next.

##### Withdraw function implementation.
For withdraw function also, we will create a struct to get the input data. Here Input data is only the amount
```rust
#[derive(BorshSerialize, BorshDeserialize, Debug)]
struct WithdrawRequest {
    pub amount: u64,
}
```
```rust
fn withdraw(
    program_id: &Pubkey,
    accounts: &[AccountInfo],
    instruction_data: &[u8],
) -> ProgramResult {
```
For the withdraw also we will create iterator and get writing_account (which is the program owned account) and admin_account.
```rust
    let accounts_iter = &mut accounts.iter();
    let writing_account = next_account_info(accounts_iter)?;
    let admin_account = next_account_info(accounts_iter)?;
    
    // We check if the writing account is owned by program.
    if writing_account.owner != program_id {
        msg!("Writter account isn't owned by program");
        return Err(ProgramError::IncorrectProgramId);
    }
    // Admin account should be the signer in this trasaction.
    if !admin_account.is_signer {
        msg!("admin should be signer");
        return Err(ProgramError::IncorrectProgramId);
    }
```
Now we will get the data of campaign from the `writing_account`. Note that we stored this when we created the campaign with create_campaign function.
> Note: We are currently thinking of a scenario where a campaign is created and we want to withdraw some money from it.
```rust
    // Just like we used the try_from_slice for 
    // instruction_data we will use it for the 
    // writing_account's data.
    let campaign_data = CampaignDetails::try_from_slice(*writing_account.data.borrow())
        .expect("Error deserialaizing data");

    // Then we check if the admin_account's public key is equal to
    // the public key we have stored in our campaign_data.
    if campaign_data.admin == *admin_account.key {
        msg!("Only the account admin can withdraw");
        return Err(ProgramError::InvalidAccountData);
    }


    // Here we make use of the struct we created.
    // We will get the amount of lamports admin wants to withdraw
    let input_data = WithdrawRequest::try_from_slice(&instruction_data)
        .expect("Instruction data serialization didn't worked");
```
We do not want the campaign to get deleted after a withdrawal. We want it to always have a minimum balance So we calculate the `rent_exemption` and take it into consideration.
```rust
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

> Note: We can only decrease the balance of a program owned account.

##### Donate to a campaign.
We want to donate to a campaign but intrestingly we can't decrease the balance of a program in our program. This means we can't just tranfer balance like we did in the withdraw account. 
As you can see the [policies here](https://docs.solana.com/developing/programming-model/runtime#policy).
>An account not assigned to the program cannot have its balance decrease.

So for this account we will create program owned account in our frontend and then make the donate transaction.

```rust
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
We get 3 account here first is the program owned account containing the data of campaign we want to donate to.
Then we have a donator_program_account which is also the program owned account which only has the lamport we would like to donate.
> Note: When we will create it in our frontend, although we do not want to store anything in it we will assign it some size. So that it automatically gets deleted from the solana after the donation is done.
```rust
  if writing_account.owner != program_id {
        msg!("Writter account isn't owned by program");
        return Err(ProgramError::IncorrectProgramId);
    }
    if donator_program_account.owner != program_id {
        msg!("Writter account isn't owned by program");
        return Err(ProgramError::IncorrectProgramId);
    }
    if !donator.is_signer {
        msg!("donator should be signer");
        return Err(ProgramError::IncorrectProgramId);
    }
```
Then we have our regular checks. We want the signer to be the donator for this transaction.

Here we get the campaign_data and we will increment the amount donated. As the total amount of data donated to this campaign will increased.
```rust
 let mut campaign_data = CampaignDetails::try_from_slice(*writing_account.data.borrow())
        .expect("Error deserialaizing data");

    campaign_data.amount_donated += **donator_program_account.lamports.borrow();
```
Then we do the actuall transaction. Note that the donator_program_account is owned by program so it can decrease it lamports. no sweat.
```rust
 **writing_account.try_borrow_mut_lamports()? -= **donator_program_account.lamports.borrow();
 **donator_program_account.try_borrow_mut_lamports()? += 0;
```
Then at the end of the program we will write the new updated campaign_data to writing_account's data field. And return `Ok(())`.
```rust
    campaign_data.serialize(&mut &mut writing_account.data.borrow_mut()[..])?;

    Ok(())
}
``` 

Hurray, We have completed our solana program Now we can go ahead and deploy it.

### Deploy solana program.
We are going to deploy on devnet.
Solana Blockchain works on a [BPF system](https://docs.solana.com/developing/on-chain-programs/overview#berkeley-packet-filter-bpf) so we will compile our program for bpf machine.
We can use the handly package manager `cargo` to do it.
```bash
cargo build-bpf --manifest-path=Cargo.toml --bpf-out-dir=dist/program
```
We can use this command to create a build, In this command the manifest-path should be the path your to `Cargo.toml`.
We will get our output in `dist/program` directory.
> If you will wun this command in `program` directory you will see dist/program directory added.
![program_complied-bpf](../../../.gitbook/assets/solana-crowdfunding-program-complied-bpf.png)


Now that we have complied our program we can deploy it. 

We will create a new solana account to deploy the program. We can do it by 
```bash
solana-keygen new -o keypair.json
```
This command will generate a keypair.json file.
> Important Note: it is not required to generate a new solana account for deployment. I am doing it for the purpose of this tutorial. Please do not commit your keypair.json on git. And keep them safe with you.

You will get a public key from this command.
Now we are going to request an airdrop for solana on `devnet`.
This command will add dev tokens to account
```bash
solana airdrop 1 <YourPublicKey> --url https://api.devnet.solana.com 
```
Example
```bash
solana airdrop 1 DGqXoguiJnAy8ExJe9NuZpWrnQMCV14SdEdiMEdCfpmB --url https://api.devnet.solana.com 
```
> If you are getting `Error: RPC request error: cluster version query failed: error sending request for url (https://api.devnet.solana.com/): error trying to connect: dns error: failed to lookup address information: nodename nor servname provided, or not known` This error please consider switching your primary server over to one of Google, DNSWatch, OpenDNS, SAFEDNS, Dyn, Yandex, AdGuard, Cloudflare.

If you get insufficent balance while deployment you can rerun this command to airdrop funds in devnet.

Now we will use 
```bash
solana deploy --keypair keypair.json dist/program/program.so --url https://api.devnet.solana.com 
```
this command to deploy. Note that the path of `keypair.json` and `dist/program/program.so` might be different.
Please check and then run the command.
And Hurray we have deployed our program
we will get the program id
```
Program Id: GzNH211q1cybWkGnCaUSyPNBf4UmRHHpC6GqrvEM1sud
```
We can verify this by checking on [solana explorer for devnet](https://explorer.solana.com/?cluster=devnet). 
We can search our program id here.

Hurray! We Have completed everything with rust and successfully deployed our program. Now lets move forward and build the react app.

# Frontend with solana web3.js
We will have create a react app, so we can open the `crowd-funding` in our vs code.
This is not a react tutorial, so I will not go into the details of react creating an app.
But I will be explaning what we are going to do.

Lets first clean our project. We will remove `setupTests.js`, `reportWebVitals.js`, `logo.svg` and `app.test.js`. And remove the usage of `reportWebVitals.js` in index.js.
Then you project should like
![frontend-project](../../../.gitbook/assets/solana-crowdfunding-frontend-files.png)

We will create a basic UI for our app. I have used sementic ui to do that.
If you wan the ui part only and continue coding the solana(web.js)  libaray you can use the ui template I created you getit From [github ui branch](https://github.com/SushantChandla/solana-crowd-funding/tree/ui).

### @solana/web3.js
Let us add the `@solana/web3.js` package.
```bash
yarn add @solana/web3.js
```
We will also use `@project-serum/sol-wallet-adapter` package. To connect our app with sollet wallet.
```bash
yarn add @project-serum/sol-wallet-adapter
```
And we will also need borsh for seralization and deserialization.
```bash
yarn add borsh
```
Let us create a `solana` folder in our `crowd-funding/src` directory. We will write all the solana realated code in this folder.
#### BoilerPlate
In `solana/index.js`
```js
import Wallet from "@project-serum/sol-wallet-adapter";
import {
    Connection,
    SystemProgram,
    Transaction,
    PublicKey,
} from "@solana/web3.js";
```
In solana we call networks clusters, if you have by any chance deployed the program on testnet/mainnet change cluster.
Also update the programId it should be the one we got after deploying our program on blockchain.
```js
const cluster = "https://api.devnet.solana.com";
const connection = new Connection(cluster, "confirmed");
const wallet = new Wallet("https://www.sollet.io", cluster);
const programId= new PublicKey(
	"GzNH211q1cybWkGnCaUSyPNBf4UmRHHpC6GqrvEM1sud"
);
```
After this I also like to create 2 helper functions`setPayerAndBlockhashTrasaction` and `signAndSendTrasaction`.
We take the instructions a input. In the `setPayerAndBlockhashTransaction` we take in instructions as paramters, `instructions` Will contains all the set of instruction we want to perform in the blockchain.
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
### Writing a function to invoke `create_campaign` instruction.
Before we start let use create a javascript implementation for the rust struct `CampaignDetails`
We have created a class to hold all our campaignDetails struct fields and we will call it `CampaignDetails`. For deserialization and serialization we have to create a schema for our class 
We will create map, And match the types of each element.
> Note : PubKey type is nothing but a u8 array of 32 bytes.
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

Now we can write our create_campaign function.
This function take in  name,description and image_link as input parameter.

The first we will do it to add a call to checkWallet, which would connect the wallet if it is not connect.
```js
export async function createCampaign(
    name, description, image_link
) {
     await checkWallet();
```
###### checkWallet function:
We will check if the wallet if not connected we will connect it
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
Now we have created a publicKey. Note that we have not given instruction to create account yet.

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
And before we send this data remember we want the first element in our instruction_data to be (0,1,2) for calling different entrypoints.
We will set it to 0. As we want to call `create_campaign` instruction. 
```js
    let data = serialize(CampaignDetails.schema, campaign);
    let data_to_send = new Uint8Array([0, ...data]);
```
Now we have the data we want to send to our program.

We want our program account To have some initial lamports because it does contain some amount of data. And we do not want it to get deleted. So will fund it with the minimum number of lamports required to make it rent Exempt.
So we calculate the lamports required like this.
```js
    const lamports =
        (await connection.getMinimumBalanceForRentExemption(data.length));
```
Here we create the instruciton to create our account we will pass in the pub key, the size of data it need to store, initial lamports and other paramters.
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
Now we can create the 2nd instruction we our program which would write the data to program account we just created in the above instruction.
In the keys paramater we will pass all account we want to send.
And in data we want to send `instruction_data` variable,
We will pass programId we want to invoke. Note we have created programId global variable in this file already.
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
Now we have create the the instruction we want. We will pass them to our helper function `setPayerAndBlockhashTransaction` which would create a instance `Transaction` for our which we can then pass to  `signAndSendTransaction`.
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

Now we can go ahead and connect this function to our frontend with our Form.

So in out `form.js`
we can have a on submit, On calling this function we will see sollet dialog. Which would ask us to sign the trasaction.
```js
    const onSubmit = async (e) => {
        e.preventDefault();
        await createCampaign(name, description, image);
    }
```
Now we can add first campaign.

### Fetch All Campaigns

We have a implemented a function to add campaigns but we do not have any function to fetch all the campaigns. Lets make that.

We know we have added all the data in program accounts so we can fetch all the program accounts by using `getProgramAccounts` on connection.
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

### Now we can write the donate function.

For donate we will again create an program account and we will fund it with the amount we want to donate. As we discussed while writing our program it is not possible to decrease the balance of an account not owned by program.

Our function will take in the campaignPubKey, and amount
amount: the amount of solana token we want to donate.
campaignPubKey: public key of the account we want to send tokens to. This is the same account where the data related to this campaign is stored.
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
This is similar to the last entry point call.
Here I have set the space as 1, because I want the account to get deleted when it balance becomes zero. And we are setting the initial lamports equal to the amount of solana tokens we want to donate.


Here We will pass 3 keys(Accounts) as our program need 3 accounts(see donate function implementation).

And we are sending data an array of `[2]` Because we want call donate function in program and we have mapped it to `2` value of first element of `instruction_data` array.
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
We have created instruction to our program here and then called the `setPayerAndBlockhashTransaction`, `signAndSendTransaction` and `confirmTransaction` to send and confirm the transaction just like we did for the `createCampaign` function.

### Withdraw function.
Now we will write the withdraw function. For withdraw we don't have to create an program account and we will only pass one instruction.

But since we are using a `WithdrawRequest` struct in our program and we are constructing it with `instruction_data` we will have to create a class and schema for borsh Serialization.
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
We have created the schema with 'amount' as `u64` which is the datatype of variable in rust.
Now Let us write the actual function.
Our function will take in the parameter `campaignPubKey` and the amount we want to withdraw.
Then we will serialize the data. First we will create the WithdrawRequest Object and then with the help of schema we have as a static member in `WithdrawRequest` class. 
```js
export async function withdraw(
    campaignPubKey, amount
) {
    await checkWallet();

 let withdrawRequest = new WithdrawRequest({ amount: amount });
    let data = serialize(WithdrawRequest.schema, withdrawRequest);
    let data_to_send = new Uint8Array([1, ...data]);

```
Then we will create the instruction and pass the data. Note that we are inserting `1` in our data to send array.
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