# 

A *program* is to Solana what a **smart contract** is to other protocols. Once a program has been deployed, any app can interact with it by sending a transaction to a Solana cluster that will pass it to the program.

{% hint style="info" %}
[You can learn more about Solana's programs here](https://docs.solana.com/developing/on-chain-programs/overview).
{% endhint %}

## Install Rust and configure the Solana CLI

So far we've been using Solana's JS API to interact with the blockchain. In this chapter we're going to deploy a Solana program using another Solana developer tool: their CLI. We'll install it and use it through our Terminal.

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

## Building the program

The first thing we're going to do is compile the Rust program to prepare it for the CLI. To do this we're going to use a custom script that's defined in `package.json`. Let's run the script and build the program by running the following command in the terminal (from the project root directory):

```bash
yarn run build:program-rust
```
{% hint style="warning" %}
This step can take 5 or 10 minutes!
{% endhint %}


When it's successful you will see a new folder in your app which contains the compiled contract: `hello-world.so`.

> The `.so` extension does not stand for Solana! It stands for "shared object". You can read more about Solana Programs [here](https://docs.solana.com/developing/on-chain-programs/overview) 


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
