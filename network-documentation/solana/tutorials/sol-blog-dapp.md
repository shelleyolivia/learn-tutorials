# Introduction

In this tutorial, we will learn how to create a simple blog post dapp on solana blockchain. while build this dapp we will learn how to write Solana rust program, Test the program and finally integrate the program with React frontend.

# Prerequisites

This tutorial assumes that you have,

- Basic understanding of React.js
- Basic understanding of Rust
- completed [Solana 101 Pathway](https://learn.figment.io/protocols/solana)

## Requirements

This tutorial covers how to build a dapp on Solana, but does not go through installation of individual dependencies(as it assumes you already completed the Solana 101 Pathway on Figment learn).

- **Anchor Framework -**
  Anchor is framework use for solana dapp development, it provides DSL to interact with the solana program. If you are familiar with developing in Solidity, Truffle or Hardhat then consider the DSL is equivalent to ABI.
  Follow the guide to install [Anchor](https://project-serum.github.io/anchor/getting-started/installation.html#install-rust) along with Rust and Solana cli.

- **React.js -**
  To interact with our solana program we will create client side app with React.js.

- **Phantom Wallet -**
  Phantom is a digital wallet which lets you connect your crypto account to any dapp that build on Solana blockchain. We will you Phantom wallet to connect to our Blog Dapp.

- **Vs code -**
  I will recommend using vscode with rust analyzer extension as it has great support for Rust language.

# Solana Programming model

**Program -** Solana is fast and low cost blockchain, to achieve the speed and low cost solana has slight different programming model. Solana uses Rust programing language to create programs, as you notice we are keep saying solana program instead of solana smart contract from choosing programing language to naming concepts solana is different, in solana world smart contracts are known as Solana Programs.

**Account -** Solana program are stateless so if you want to store state you need to use an account for it and the accounts are fixed in size. once the account is initialized with the size, you cannot change the size latter So we have to design our application by keeping this is mind.

**Rent -** on Solana you need to pay rent regularly to store data on blockchain according to the space the data requires,
the account can be made rent exempt (means you wont have to pay rent) if its balance is higher than the some threshold that depends on the space its consuming.

## Application design decision

As we learn we need an account to create our blog dapp that has fixed size, so if we create a single account with X size and start pushing posts inside that account eventually the account exceeds its size limit and we wont be able to create new posts.
if you know solidity, in solidity we create a dynamic array and push as many items to it as we want. but in solana our accounts will be fixed in sized so we have to find a solution this problem.

- **Solution one -** What if we create extremely large size account like in gigabytes? on solana we need to pay rent of account according to its size so if our account grows in size account rent will grow along with it.

- **Solution two -** What if we create multiple accounts and connect them somehow? Yes thats the plan, we will create new account for every single post and create chain of posts linked one after another.

Linked 🤔, yeah you guessed it right we will use LinkedList to connect all the posts.

## Setting up Local development

before we start with actual development we learn some Solana CLI commands [docs](https://docs.solana.com/cli/conventions):

to see your current solana configuration use(I assume you have followed Solana 101 Pathway and you have all the cli installation done):

```
solana config get

# output
Config File: /Users/user/.config/solana/cli/config.yml
RPC URL: https://api.devnet.solana.com
WebSocket URL: wss://api.devnet.solana.com/ (computed)
Keypair Path: /home/user/.config/solana/id.json
Commitment: confirmed
```

your output might have different file paths.

You can check the current wallet address by:

```
solana address
```

You can the check the balance of your wallet.

```
solana balance
```

or You can airdrop tokens to your account.

```
solana airdrop 10
```

Check balance again and now you should have a balance 10 SOL in your wallet.

**Now its time to scaffold our blog app with the help of Anchor CLI:**

```
anchor init blog

cd blog
```

The anchor init command creates the following directories:

```
├── app
├── programs
|   └── blog
|        └── src
|             └── lib.rs
├── test
```

Before writing program code update Anchor.toml

```
wallet = "your Keypair Path from output of solana config get"
```

## Creating blog program

Now we are ready to start with solana rust program, open up the lib.rs file located inside /program/blog/src/ folder.

```rust
use anchor_lang::prelude::*;

declare_id!("Fg6PaFpoGXkYsidMpWTK6W2BeZ7FEfcYkg476zPFsLnS");

#[program]
pub mod blog {
    use super::*;
    pub fn initialize(ctx: Context<Initialize>) -> ProgramResult {
        Ok(())
    }
}

#[derive(Accounts)]
pub struct Initialize {}
```

This is the basic example of an anchor solana program, there is only one function `initialize` which will be invoke by the client.
The initialize function has one argument of type context of `Initialize` struct.

Another noticeable thing is `declare_id!`. declare_id! is a macro it defines the program address and used in internal validation, we dont have to think about it too much this will be handle by the Anchor cli.

**Now its time to start declaring states of our blog app.**

```rust
// pseudo code

blog {
 current_post_key    // latest post id so we can traverse back to other posts
 authority           // who owns the account
}

user {
 name                // store user name
 avatar              // user avatar
 authority           // owner
}

post {
 title              // post title
 content            // post descriptive content
 user               // user id
 pre_post_key       // to create LinkedList
 authority          // owner
}
```

as you have seen the first basic example, we need to create function tha will define out task that we want to perform on program like, init_blog, signup_user, create_post etc.

we will start with create our very first function `init_blog`

```rust
 pub fn init_blog(ctx: Context<InitBlog>) -> ProgramResult {
        Ok(())
 }

 // define ctx type
  #[derive(Accounts)]
  pub struct InitBlog<'info> {
      #[account(init, payer = authority, space = 8 + 32 + 32)]
      pub blog_account: Account<'info, BlogState>,
      #[account(init, payer = authority, space = 8 + 32 + 32 + 32 + 32 + 8)]
      pub genesis_post_account: Account<'info, PostState>,
      pub authority: Signer<'info>,
      pub system_program: Program<'info, System>,
  }

  // from pseudo blog state
  #[account]
  pub struct BlogState {
      pub current_post_key: Pubkey,
      pub authority: Pubkey,
  }
```

as you know every function needs a typed context as first argument, here we have defined `InitBlog` as type of our `init_blog` ctx.
in ctx type we have define the account we needed to execute the function and account will be provided by the client(caller of the function).

in `InitBlog` there are 4 accounts:

- **blog_account**
  - init attribute to create/initialize new account
  - space = 8 + 32 + 32, here we are creating new account thats why we have to specify account size, we will see later how to calculate the account size.
  - payer = authority, authority is an one of account provided by client basically authority is rent payer of blog_account
- **genesis_post_account**
  - we are also creating this account thats why the init, payer and space attributes are there
  - to create LinkedList we initialize the blog account with the very first post, so we can link it to the next post.
- authority
  - program signer, creator of blog.
- system_program
  - required by the runtime for creating the account.

with `init_blog` our plan is to initialize the blog account with current_post_key and authority as blog state so lets write code for that,

```rust
  pub fn init_blog(ctx: Context<InitBlog>) -> ProgramResult {
      // get accounts from ctx
      let blog_account = &mut ctx.accounts.blog_account;
      let genesis_post_account = &mut ctx.accounts.genesis_post_account;
      let authority = &mut ctx.accounts.authority;

      // sets the blog state
      blog_account.authority = authority.key();
      blog_account.current_post_key = genesis_post_account.key();

      Ok(())
  }
```

this how easy to create an account which holds some state data with the anchor framework.

Now we will move to the next function, what we can do next?? user, user signup. Lets define signup function with which user can create his/her profile by providing name and avatar as inputs.

```rust
 pub fn signup_user(ctx: Context<SignupUser>) -> ProgramResult {
    Ok(())
 }
```

Thats the basic skeleton to create new function but here how we get name and avatar from user?? Lets see.

```rust
 pub fn signup_user(ctx: Context<SignupUser>, name: String, avatar: String) -> ProgramResult {
    Ok(())
 }
```

we can accept any number of arguments after ctx like here name and avatar as String (Rust is a statically typed Language, we have to define type while defining variables). Next is `SignupUser` ctx type and `UserState` state.

```rust
#[derive(Accounts)]
pub struct SignupUser<'info> {
    #[account(init, payer = authority, space = 8 + 40 + 120  + 32)]
    pub user_account: Account<'info, UserState>,
    pub authority: Signer<'info>,
    pub system_program: Program<'info, System>,
}

#[account]
pub struct UserState {
    pub name: String,
    pub avatar: String,
    pub authority: Pubkey,
}
```

Here we need three accounts and you already understand in previous function all the attributes(like init, payer, space) so i wont re-explain that here. But i will explain to you how to calculate the account space this time.
to measure account space we need to take look at what state the account is holding, in user_account case **UserState** has 3 values to store name, avatar and authority.

| State Values | Data Types | Size (in bytes) |
| ------------ | ---------- | --------------- |
| authority    | Pubkey     | 32              |
| name         | String     | 40              |
| avatar       | String     | 120             |

**Pubkey:** Pubkey is always 32 bytes and String is variable in size so it is depend on your use case.

**String:** String is an array of chars and each char take 4 bytes in rust.

**Account Discriminator:** All accounts created with Anchor needs 8 bytes

Moving forward, Let's complete the remaining signup function

```rust
    pub fn signup_user(ctx: Context<SignupUser>, name: String, avatar: String) -> ProgramResult {
        let user_account = &mut ctx.accounts.user_account;
        let authority = &mut ctx.accounts.authority;

        user_account.name = name;
        user_account.avatar = avatar;
        user_account.authority = authority.key();

        Ok(())
    }
```

Till now we have created 2 function init_blog and signup user with name and avatar. specifically signup takes two arguments, what if user mistakenly sent wrong name and he/she want to update it??
you guessed it right, we will create next function that allow user to update name and avatar of their account.

```rust
  pub fn update_user(ctx: Context<UpdateUser>, name: String, avatar: String) -> ProgramResult {
      let user_account = &mut ctx.accounts.user_account;

      user_account.name = name;
      user_account.avatar = avatar;

      Ok(())
  }

 #[derive(Accounts)]
  pub struct UpdateUser<'info> {
      #[account(
          mut,
          has_one = authority,
      )]
      pub user_account: Account<'info, UserState>,
      pub authority: Signer<'info>,
  }
```

New attributes mut and has_only

- **mut:** if we want to change/update account state/data we must specify the mut attribute
- **has_one:** has_one checks user_account.authority is equal to authority accounts key ie. owner of user_account is signer(caller) of update_user function

Our blog is initialized user is created now whats remaining?? CRUD of post, in next section we will look into the CRUD of post entity. if you feel overwhelming take break or go through what we have learned/builded so far.

Now lets go little crazy!! CRUD of post!!

```rust
   pub fn create_post(ctx: Context<CreatePost>, title: String, content: String) -> ProgramResult {
        Ok(())
    }

    #[derive(Accounts)]
    pub struct CreatePost<'info> {
        #[account(init, payer = authority, space = 8 + 50 + 500 + 32 + 32 + 32)]
        pub post_account: Account<'info, PostState>,
        #[account(mut, has_one = authority)]
        pub user_account: Account<'info, UserState>,
        #[account(mut)]
        pub blog_account: Account<'info, BlogState>,
        pub authority: Signer<'info>,
        pub system_program: Program<'info, System>,
    }

    #[account]
    pub struct PostState {
        title: String,
        content: String,
        user: Pubkey,
        pub pre_post_key: Pubkey,
        pub authority: Pubkey,
    }
```

What do you think, why we need **blog_account** as mut here? Do you remember **current_post_key** field in **BlogState**. Lets look at the function body.

```rust
    pub fn create_post(ctx: Context<CreatePost>, title: String, content: String) -> ProgramResult {
        let blog_account = &mut ctx.accounts.blog_account;
        let post_account = &mut ctx.accounts.post_account;
        let user_account = &mut ctx.accounts.user_account;
        let authority = &mut ctx.accounts.authority;

        post_account.title = title;
        post_account.content = content;
        post_account.user = user_account.key();
        post_account.authority = authority.key();
        post_account.pre_post_key = blog_account.current_post_key;

        // store created post id as current post id in blog account
        blog_account.current_post_key = post_account.key();

        Ok(())
    }
```

The post is created, Now how we can let the client know that the post is created and the client fetch the post and render it into ui.
Anchor provides a handy feature of emitting an event, Event?? Yup you heard it right. we can emit an event post created post. Before emitting event we need to define it.

```rust

#[event]
pub struct PostEvent {
    pub label: String, // label is like 'CREATE', 'UPDATE', 'DELETE'
    pub post_id: Pubkey, // created post
    pub next_post_id: Option<Pubkey>, // for now ignore this, we will use this when we emit delete event
}
```

Lets emit post create event from post_create function

```rust
    pub fn create_post(ctx: Context<CreatePost>, title: String, content: String) -> ProgramResult {
        ....

        emit!(PostEvent {
            label: "CREATE".to_string(),
            post_id: post_account.key(),
            next_post_id: None // same as null
        });

        Ok(())
    }
```

No questions game, straight to the point "Update Post"!

```rust
    pub fn update_post(ctx: Context<UpdatePost>, title: String, content: String) -> ProgramResult {
        let post_account = &mut ctx.accounts.post_account;

        post_account.title = title;
        post_account.content = content;

        emit!(PostEvent {
            label: "UPDATE".to_string(),
            post_id: post_account.key(),
            next_post_id: None // null
        });

        Ok(())
    }


    #[derive(Accounts)]
    pub struct UpdatePost<'info> {
        #[account(
            mut,
            has_one = authority,
        )]
        pub post_account: Account<'info, PostState>,
        pub authority: Signer<'info>,
    }
```

Updating post is really simple, take title and content from user and update the **mut post_account**

Delete post is little challenging, to store posts we have used LinkedList. if you know LinkedList after delete node from LinkedList we need to link the adjacent node of deleting node. lets understand this through diagram.

 <img src="../../../.gitbook/assets/sblog_delete_post.png" alt="post delete" width="400" >

if we want to delete post 2 we have to link 1 -> 3.

Lets jump to the code,I know you will understand it easily.

```rust
    // Here we need two post account, current_post and next_post account. we get pre_post of current_post from current_post and link it to next_post

    pub fn delete_post(ctx: Context<DeletePost>) -> ProgramResult {
        let post_account = &mut ctx.accounts.post_account;
        let next_post_account = &mut ctx.accounts.next_post_account;

        next_post_account.pre_post_key = post_account.pre_post_key;

        emit!(PostEvent {
            label: "DELETE".to_string(),
            post_id: post_account.key(),
            next_post_id: Some(next_post_account.key())
        });

        Ok(())
    }

    #[derive(Accounts)]
    pub struct DeletePost<'info> {
        #[account(
            mut,
            has_one = authority,
            close = authority,
            constraint = post_account.key() == next_post_account.pre_post_key
        )]
        pub post_account: Account<'info, PostState>,
        #[account(mut)]
        pub next_post_account: Account<'info, PostState>,
        pub authority: Signer<'info>,
    }
```

**constraint** attribute performs simple if check.

So to delete post use needs to send post_account and next_post_account but what is there is no next_post?? what if user want to delete the latest post that has no next post??
to handle this case we need to create another function **delete_latest_post**

```rust
    pub fn delete_latest_post(ctx: Context<DeleteLatestPost>) -> ProgramResult {
        let post_account = &mut ctx.accounts.post_account;
        let blog_account = &mut ctx.accounts.blog_account;

        blog_account.current_post_key = post_account.pre_post_key;

        emit!(PostEvent {
            label: "DELETE".to_string(),
            post_id: post_account.key(),
            next_post_id: None
        });

        Ok(())
    }

    #[derive(Accounts)]
    pub struct DeleteLatestPost<'info> {
        #[account(
            mut,
            has_one = authority,
            close = authority
        )]
        pub post_account: Account<'info, PostState>,
        #[account(mut)]
        pub blog_account: Account<'info, BlogState>,
        pub authority: Signer<'info>,
    }
```

That was last function of our Rust Program.

Next is Testing program. Dont worry, we'll fast forward the next section.

## Writing tests for blog program

before we dive into writing test cases, every test needs a initialized blog, a brand new user and a post so to avoid repetition we will create 3 simple reusable utility functions.

- createBlog - initialize new Blog account
- createUser - create a new User
- createPost - create new Post

**createBlog.js**

```js
const anchor = require("@project-serum/anchor");

const { SystemProgram } = anchor.web3;

// we will discus the parameters when we use it
async function createBlog(program, provider) {
  const blogAccount = anchor.web3.Keypair.generate(); // creates random keypair
  const genesisPostAccount = anchor.web3.Keypair.generate(); // creates random keypair

  await program.rpc.initBlog({
    accounts: {
      authority: provider.wallet.publicKey,
      systemProgram: SystemProgram.programId,
      blogAccount: initBlogAccount.publicKey,
      genesisPostAccount: genesisPostAccount.publicKey,
    },
    signers: [initBlogAccount, genesisPostAccount],
  });

  const blog = await program.account.blogState.fetch(initBlogAccount.publicKey);

  return { blog, blogAccount, genesisPostAccount };
}

module.exports = {
  createBlog,
};
```

**createUser.js**

```js
const anchor = require("@project-serum/anchor");
const { SystemProgram } = anchor.web3;

async function createUser(program, provider) {
  const userAccount = anchor.web3.Keypair.generate();

  const name = "user name";
  const avatar = "https://img.link";

  await program.rpc.signupUser(name, avatar, {
    accounts: {
      authority: provider.wallet.publicKey,
      userAccount: userAccount.publicKey,
      systemProgram: SystemProgram.programId,
    },
    signers: [userAccount],
  });

  const user = await program.account.userState.fetch(userAccount.publicKey);
  return { user, userAccount, name, avatar };
}

module.exports = {
  createUser,
};
```

**createPost.js**

```js
const anchor = require("@project-serum/anchor");
const { SystemProgram } = anchor.web3;

async function createPost(program, provider, blogAccount, userAccount) {
  const postAccount = anchor.web3.Keypair.generate();
  const title = "post title";
  const content = "post content";

  await program.rpc.createPost(title, content, {
    // pass arguments to the program
    accounts: {
      blogAccount: blogAccount.publicKey,
      authority: provider.wallet.publicKey,
      userAccount: userAccount.publicKey,
      postAccount: postAccount.publicKey,
      systemProgram: SystemProgram.programId,
    },
    signers: [postAccount],
  });

  const post = await program.account.postState.fetch(postAccount.publicKey);
  return { post, postAccount, title, content };
}

module.exports = {
  createPost,
};
```

Now we are ready to write our first every test case.

```js
const anchor = require("@project-serum/anchor");
const assert = require("assert");

describe("blog tests", () => {
  const provider = anchor.Provider.env();
  anchor.setProvider(provider);
  const program = anchor.workspace.BlogSol;

  it("initialize blog account", async () => {
    // call the utility function
    const { blog, blogAccount, genesisPostAccount } = await createBlog(
      program,
      provider
    );

    assert.equal(
      blog.currentPostKey.toString(),
      genesisPostAccount.publicKey.toString()
    );

    assert.equal(
      blog.authority.toString(),
      provider.wallet.publicKey.toString()
    );
  });
});
```

Next, run the test:

```
anchor test
```

after running `anchor test` you will see the 1/1 test passing.

Now we complete remaining tests:

```js
const anchor = require("@project-serum/anchor");
const assert = require("assert");

describe("blog tests", () => {
  const provider = anchor.Provider.env();
  anchor.setProvider(provider);
  const program = anchor.workspace.BlogSol;

  it("initialize blog account", async () => {
    const { blog, blogAccount, genesisPostAccount } = await createBlog(
      program,
      provider
    );

    assert.equal(
      blog.currentPostKey.toString(),
      genesisPostAccount.publicKey.toString()
    );

    assert.equal(
      blog.authority.toString(),
      provider.wallet.publicKey.toString()
    );
  });

  it("signup a new user", async () => {
    const { user, name, avatar } = await createUser(program, provider);

    assert.equal(user.name, name);
    assert.equal(user.avatar, avatar);

    assert.equal(
      user.authority.toString(),
      provider.wallet.publicKey.toString()
    );
  });

  it("creates a new post", async () => {
    const { blog, blogAccount } = await createBlog(program, provider);
    const { userAccount } = await createUser(program, provider);

    const { title, post, content } = await createPost(
      program,
      provider,
      blogAccount,
      userAccount
    );

    assert.equal(post.title, title);
    assert.equal(post.content, content);
    assert.equal(post.user.toString(), userAccount.publicKey.toString());
    assert.equal(post.prePostKey.toString(), blog.currentPostKey.toString());
    assert.equal(
      post.authority.toString(),
      provider.wallet.publicKey.toString()
    );
  });

  it("updates the post", async () => {
    const { blog, blogAccount } = await createBlog(program, provider);
    const { userAccount } = await createUser(program, provider);
    const { postAccount } = await createPost(
      program,
      provider,
      blogAccount,
      userAccount
    );

    // now update the created post
    const updateTitle = "Updated Post title";
    const updateContent = "Updated Post content";
    const tx = await program.rpc.updatePost(updateTitle, updateContent, {
      accounts: {
        authority: provider.wallet.publicKey,
        postAccount: postAccount.publicKey,
      },
    });

    const post = await program.account.postState.fetch(postAccount.publicKey);

    assert.equal(post.title, updateTitle);
    assert.equal(post.content, updateContent);
    assert.equal(post.user.toString(), userAccount.publicKey.toString());
    assert.equal(post.prePostKey.toString(), blog.currentPostKey.toString());
    assert.equal(
      post.authority.toString(),
      provider.wallet.publicKey.toString()
    );
  });

  it("deletes the post", async () => {
    const { blogAccount } = await createBlog(program, provider);
    const { userAccount } = await createUser(program, provider);
    const { postAccount: postAcc1 } = await createPost(
      program,
      provider,
      blogAccount,
      userAccount
    );

    const { post: post2, postAccount: postAcc2 } = await createPost(
      program,
      provider,
      blogAccount,
      userAccount
    );

    const {
      post: post3,
      postAccount: postAcc3,
      title,
      content,
    } = await createPost(program, provider, blogAccount, userAccount);

    assert.equal(postAcc2.publicKey.toString(), post3.prePostKey.toString());
    assert.equal(postAcc1.publicKey.toString(), post2.prePostKey.toString());

    await program.rpc.deletePost({
      accounts: {
        authority: provider.wallet.publicKey,
        postAccount: postAcc2.publicKey,
        nextPostAccount: postAcc3.publicKey,
      },
    });

    const upPost3 = await program.account.postState.fetch(postAcc3.publicKey);
    assert.equal(postAcc1.publicKey.toString(), upPost3.prePostKey.toString());

    assert.equal(upPost3.title, title);
    assert.equal(upPost3.content, content);
    assert.equal(upPost3.user.toString(), userAccount.publicKey.toString());
    assert.equal(
      upPost3.authority.toString(),
      provider.wallet.publicKey.toString()
    );
  });
});
```

Run again:

```
anchor test
```

# Building frontend

Now we're ready to build out the front end. we will create react app inside the existing app directory.

```
cd app
npx create-react-app .
```

folder structure of React spp

```
├── public
├── src
|   └── app.js
├── package.json
```

Before we start writing frontend part of tutorial we will create a simple script that will copy program idl file to the React app.
whenever we deploy our rust program with `anchor deploy` the anchor cli generates the idl file that has all the metadata related to our rust program(this metadata helps building client side interface with the rust program).

Create copy_idl.js file root of your project and copy the code given below. The code is just copying the idl file from /target/idl to /app/src directory.

```js
const fs = require("fs");
const blog_idl = require("./target/idl/blog_sol.json");

fs.writeFileSync("./app/src/idl.json", JSON.stringify(blog_idl, null, 2));
```

Next, install dependencie.

```
npm i @solana/wallet-adapter-react @solana/wallet-adapter-wallets @solana/web3.js
```

Next, open app/src/App.js and update it with the following:

```js
import {
  ConnectionProvider,
  WalletProvider,
} from "@solana/wallet-adapter-react";
import { getPhantomWallet } from "@solana/wallet-adapter-wallets";
import { Home } from "./home";

const wallets = [getPhantomWallet()];
const endPoint = "http://127.0.0.1:8899";

const App = () => {
  return (
    <ConnectionProvider endpoint={endPoint}>
      <WalletProvider wallets={wallets} autoConnect>
        <Home />
      </WalletProvider>
    </ConnectionProvider>
  );
};

export default App;
```

in the further tutorial I'm just gonna explain the logical part of our dapp and I'll leave the styling part up to you.

Now let's start with login functionality of our dapp, we will need a button that handle the user login with Phantom browser wallet.

create button inside Home.js component with `onConnect` onClick handler like this,

```JSX
<button onClick={onConnect}>Connect with Phantom</button>
```

then create `onConnect` function that handles the click event of connect button.

```js
import { WalletName } from "@solana/wallet-adapter-wallets";
import { useWallet } from "@solana/wallet-adapter-react";

// inside react component
const { select } = useWallet();
const onConnect = () => {
  select(WalletName.Phantom);
};
```

Lets deploy our Rust program first then copy the idl file with the help of the copy_idl.js script that we wrote before,

before deploy make sure you have `localnet` cluster set in Anchor.toml file. Now
open up new Terminal session and run `solana-test-validator` command this will start local network of solana blockchain.

```
// in new terminal run,
solana-test-validator
```

then deploy the program with,

```
anchor deploy
```

if you run into an error,

- make sure `solana-test-validator` is running
- make sure your solana config is in valid state (I mean the RPC url, KeyPair path etc.)

once you successfully deploy the Rust program with anchor cli then run the copy_idl.js file like,

```
node copy_idl.js
```

it will copy the idl file to /app/src directory, Now you will see the idl.json file inside /app/src directory.

## Initialize Blog

Now we will initialize the blog. Initializing blog is one time process. Lets create `init_blog.js` file inside /app/src folder and copy the code given below.

```js
import { Program } from "@project-serum/anchor";
import { Keypair, PublicKey, SystemProgram } from "@solana/web3.js";
import idl from "./idl.json";

const PROGRAM_KEY = new PublicKey(idl.metadata.address);

export async function initBlog(walletKey, provider) {
  const program = new Program(idl, PROGRAM_KEY, provider);
  const blogAccount = Keypair.generate();
  const genesisPostAccount = Keypair.generate();

  await program.rpc.initBlog({
    accounts: {
      authority: walletKey,
      systemProgram: SystemProgram.programId,
      blogAccount: blogAccount.publicKey,
      genesisPostAccount: genesisPostAccount.publicKey,
    },
    signers: [blogAccount, genesisPostAccount],
  });

  console.log("Blog pubkey: ", blogAccount.publicKey.toString());
}
```

in this `initBlog` function we have imported the program idl then we have generated two Keypair for blog account and initial dummy post account and after that we just call the initBlog function of the program with all the necessary accounts. we are creating two new accounts here `blogAccount` and `genesisPostAccount` thats why we have to pass it as signers.

Now the ui part of this, we will create a temporary button to call the initBlog function. Once the blog is initialize we will remove the button(as it wont be needed anymore).

```JSX
  import {
    useAnchorWallet,
    useConnection
  } from "@solana/wallet-adapter-react";
  import idl from './idl.json'

  const PROGRAM_KEY = new PublicKey(idl.metadata.address);
  const BLOG_KEY = /* new PublicKey(blog key) */;

  // inside react component
  const { connection } = useConnection();
  const wallet = useAnchorWallet();

  const _initBlog = () => {
    const provider = new Provider(connection, wallet, {});
    initBlog(provider.wallet.publicKey, provider);
  };

  <button onClick={_initBlog}>Init blog</button>
```

initBlog function will create a brand new Blog and console log its publicKey. make sure you still have `solana-test-validator` running in another shell and your Phantom wallet is connected to the local network (http://localhost:8899)

if you dont know how to connect Phantom wallet to Localnet here are the steps,

- go to settings tab in Phantom wallet
- scroll down and select `change network`
- finally choose `Localhost`

Next, you need balance in your account on Localnet to get balance you need to run the following command.

```
solana airdrop 10 <your-account-address>
```

then check balance with

```
solana balance <your-account-address>
```

you will see `10 SOL` printed on your terminal.

Once you are connected to the Localhost you are ready to initialize the blog account. Run the app with `npm run start` it will open up your Dapp in your browser there you will see two button one is `connect` and another one is `init blog`. First step is to connect to the Phantom wallet by clicking `connect` button. Once you are connected then click on the `init blog` button it will trigger the Phantom wallet confirmation popup click `Approve`. After 1 to 2 sec the `initBlog` function will console log the publicKey of your blog account, just copy the blog publicKey and store in `BLOG_KEY` variable and thats it we have successfully initialized our blog account.

```js
const BLOG_KEY = new PublicKey(your - blog - key);
```

Your blog is initialized now you can comment out the `init blog` button as well the `_initBlog` function.

## Signup user

Now we will move to the part where user enter his/her name and avatar url and we will initialize his/her account.

let's create two input fields one for user name and another for user avatar.

```JSX
 <input placeholder="user name" />
 <input placeholder="user avatar" />
```

Next attach React state to the input fields and create `signup` button

```JSX
 const [name, setName] = useState("")
 const [avatar, setAvatar] = useState("")

 const _signup = ()=> {

 }

 <input placeholder="user name" value={name} onChange={e => setName(e.target.value)} />
 <input placeholder="user avatar" value={avatar} onChange={e => setAvatar(e.target.value)} />
 <button onClick={_signup}>Signup</button>
```

Now we will write the functionality of \_signup function,
if you remember the initBlog function there we have randomly generated Keypair but in this case we wont generate the Keypair randomly as we need to identify the user for all the subsequent logins.

`Keypair` has `fromSeed` function that takes a `seed` argument and generate unique Keypair from the given seed.

so we will create seed by combining `PROGRAM_KEY` and users `wallet_key` so it will create unique Keypair.

```js
import { Keypair, PublicKey, SystemProgram } from "@solana/web3.js";
import { Program, Provider } from "@project-serum/anchor";

const genUserKey = (PROGRAM_KEY, walletKey) => {
  const userAccount = Keypair.fromSeed(
    new TextEncoder().encode(
      `${PROGRAM_KEY.toString().slice(0, 15)}__${walletKey
        .toString()
        .slice(0, 15)}`
    )
  );

  return userAccount;
};
```

Let's complete the \_signup function,

```js
const _signup = async () => {
  const provider = new Provider(connection, wallet, {});
  const program = new Program(idl, PROGRAM_KEY, provider);
  const userAccount = genUserKey(PROGRAM_KEY, provider.wallet.publicKey);

  await program.rpc.signupUser(name, avatar, {
    accounts: {
      authority: provider.wallet.publicKey,
      userAccount: userAccount.publicKey,
      systemProgram: SystemProgram.programId,
    },
    signers: [userAccount],
  });
};
```

Now we have created user account next we will fetch the user account to see whether user has already signed up.

```jsx
import { useEffect, useState } from "react";

// inside react component
const fetchUser = async () => {
  const provider = new Provider(connection, wallet, {});
  const program = new Program(idl, PROGRAM_KEY, provider);
  const userAccount = genUserKey(PROGRAM_KEY, provider.wallet.publicKey);

  const _user = await program.account.userState.fetch(userAccount.publicKey);

  return _user;
};

const [user, setUser] = useState();
// fetch user when wallet is connected
useEffect(() => {
  if (wallet?.publicKey) {
    fetchUser()
      .then((user) => {
        setUser(user);
      })
      .catch((e) => console.log(e));
  }
}, [wallet]);

// if user is not undefined then show user name and user avatar in ui
// otherwise show signup form to the user

{
  user ? (
    <>
      <h1>user name: {user.name}</h1>
      <h1>user avatar: {user.avatar} </h1>
    </>
  ) : (
    <>
      <input
        placeholder="user name"
        value={name}
        onChange={(e) => setName(e.target.value)}
      />
      <input
        placeholder="user avatar"
        value={avatar}
        onChange={(e) => setAvatar(e.target.value)}
      />
      <button onClick={_signup}>Signup</button>{" "}
    </>
  );
}
```

## create post

first we will create a form to take post title and post content from user.

```jsx
    const [title, setTitle] = useState("");
    const [content, setContent] = useState("");


     const _createPost = async ()=> {

     }

    <input
      placeholder="post title"
      value={title}
      onChange={(e) => setTitle(e.target.value)}
    />
    <input
      placeholder="post content"
      value={content}
      onChange={(e) => setContent(e.target.value)}
    />
    <button onClick={_createPost}>Create post</button>
```

next complete the \_createPost function

```js
const _createPost = async () => {
  const provider = new Provider(connection, wallet, {});
  const program = new Program(idl, PROGRAM_KEY, provider);
  const postAccount = Keypair.generate();
  const userAccount = genUserKey(PROGRAM_KEY, provider.wallet.publicKey);

  await program.rpc.createPost(title, content, {
    accounts: {
      blogAccount: BLOG_KEY,
      authority: provider.wallet.publicKey,
      userAccount: userAccount.publicKey,
      postAccount: postAccount.publicKey,
      systemProgram: SystemProgram.programId,
    },
    signers: [postAccount],
  });
};
```

here we have passed the `BLOG_KEY` the same key that we have obtained by initializing the blog.

Now we will fetch the all the posts created on our Blog. if you remember the Rust program we have attach the previous post id the the current post, here we will use the previous post id to iterate over the list of posts. to do so first we will create a function that finds the post of given post id.

```js
const getPostById = async (postId) => {
  const provider = new Provider(connection, wallet, {});
  const program = new Program(idl, PROGRAM_KEY, provider);

  try {
    const post = await program.account.postState.fetch(new PublicKey(postId));

    const userId = post.user.toString();
    if (userId === SystemProgram.programId.toString()) {
      return;
    }

    return {
      id: postId,
      title: post.title,
      content: post.content,
      userId,
      prePostId: post.prePostKey.toString(),
    };
  } catch (e) {
    console.log(e.message);
  }
};
```

To loop over all the posts we need latest post id and we can find the latest post id from the blog state.
create `fetchAllPosts` function as,

```js
const fetchAllPosts = async () => {
  const provider = new Provider(connection, wallet, {});
  const program = new Program(idl, PROGRAM_KEY, provider);

  // read the blog state
  const blog = await program.account.blogState.fetch(BLOG_KEY);

  const latestPostId = blog.currentPostKey.toString();
  const posts = [];

  let nextPostId = latestPostId;
  while (!!nextPostId) {
    const post = await getPostById(nextPostId, program);
    if (!post) {
      break;
    }

    posts.push(post);
    nextPostId = post.prePostId;
  }

  return posts;
};
```

Now trigger the fetchAllPosts function when user login with Phantom wallet.

```js
const [posts, setPosts] = useState([]);

// fetch all the posts when wallet is connected
useEffect(() => {
  if (wallet?.publicKey) {
    fetchAllPosts()
      .then((posts) => {
        setPosts(posts);
      })
      .catch((e) => console.log(e));
  }
}, [wallet]);
```

Show posts in the UI

```jsx
{
  posts.map(({ title, content }, i) => {
    return (
      <div key={i}>
        <h2>{title}</h2>
        <p>{content}</p>
      </div>
    );
  });
}
```

## Todo

Till now we have integrated initialization of blog, user signup, fetch user, create post, fetch all the posts there still some part is remaining like,

- update user
- update post
- delete post

if your follow the same steps as we have learned you can definitely be able to complete the remaining todo tasks.

# Deploying to Devnet

Deploying to a live network is straightforward:

1. Set solana config to devnet

```
solana config set --url devnet
```

2. Open Anchor.toml and Update the cluster to devnet

```
cluster = "devnet"
```

3. Build program

```
anchor build
```

4. Deploy program

```
anchor deploy
```

## Conclusion

Congratulations on finishing the tutorial. Thank you for taking the time to complete it. There still a room for improvements like,

- UI improvements
- creating frontend app with Typescript
- adding pagination while fetching all the posts
- creating user timeline(posts created user)
- adding rxjs to fetch posts(streaming posts)

## About Author

This tutorial was created by [Kiran Bhalerao](https://github.com/kiran-bhalerao). For any suggestions/questions you can connect with author on [Figment Forum](https://community.figment.io/u/kiranbhalerao/).

## References

[https://github.com/kiran-bhalerao/blog-dapp-solana](https://github.com/kiran-bhalerao/blog-dapp-solana)
