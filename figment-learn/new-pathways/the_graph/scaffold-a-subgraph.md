## What's a subgraph?

A subgraph defines which data The Graph will index from Ethereum, and how it will store it.

It's made up of 3 main pieces: a manifest, a schema of entities and mappings.

<add image>



## Picking a smart contract

A subgraph is basically a piece of software that indexes events emitted by a smart contract. So the first thing we need to do is pick the smart contract our subgraph will be listening to.

For the purpose of this tutorial we have decided to pick a fun and popular smart contract: the Crypto Punk ETH-20 contract. You can view it on Etherscan [here](https://etherscan.io/address/0xb47e3cd837dDF8e4c57F05d70Ab865de6e193BBB) and if you click on the "Contract" tab you can also have a look at [its Solidity code](https://etherscan.io/address/0xb47e3cd837dDF8e4c57F05d70Ab865de6e193BBB).

> Looking at the code, can you find its Events? What functions are they calling? What arguments are they passing? Browse around the codebase, we will come back to those events very soon.

## Install the Graph CLI 

Fortunately, we won't have to build a subgraph from scratch, The Graph provides a CLI to do this. Install the CLI by running:

```text
yarn global add @graphprotocol/graph-cli
```

Verify the installation was successful by running:

```text
graph --version
```

This should output the current version of the graph-cli, `0.22.1` at the time of writing this tutorial.

## Generate a subgraph scaffold

Scaffolding a subgraph will create a subgraph template. It will have the right shape but will be incomplete.

Let's first cd into the folder that will hold out subgraphs

```sh
cd subgraphs
```

Then run this command in your terminal to generate a subgraph scaffold:

```sh
graph init \
  --allow-simple-name \
  --from-contract 0xb47e3cd837dDF8e4c57F05d70Ab865de6e193BBB \
  --index-events \
  --contract-name punks \
  --network mainnet \
  --node http://localhost:8020/ punks
```

That's a mouthful! What does this code do?

- `graph init` is the CLI command that will initialize an empty subgraph.
- `--allow-simple-name` simplifies the naming convention of our local graph.
- `--from-contract` uses an already deployed contract at the specified address.
- `--index-event` creates entities from events (not a good idea).
- `--contract-name punks` sets the contract name to the supplied string, for example: punks
- `--network mainnet` tells the Graph CLI to look on Mainnet Ethereum to find the contract ABI
- `--node http://localhost:8020/` will prepare our script to deploy to our local graph node
- `punks` is the name of the folder under which the files are created

> _NOTE_: Linux and macOS use the backslash character \ for multi-line input. Windows uses the ^ character. If you paste this command into a Windows terminal (PowerShell, cmd.exe or Windows Terminal), replace the \ with ^

Once you type Enter, you will be prompted to confirm the information: you can just accept the five suggested inputs The output should look like:

```text
✔ Subgraph name · punks
✔ Directory to create the subgraph in · punks
✔ Ethereum network · mainnet
✔ Contract address · 0xb47e3cd837dDF8e4c57F05d70Ab865de6e193BBB
✔ Fetching ABI from Etherscan
✔ Contract Name · punks
———
  Generate subgraph from ABI
  Write subgraph to directory
✔ Create subgraph scaffold
✔ Initialize subgraph repository
✔ Install dependencies with yarn
✔ Generate ABI and schema types with yarn codegen

Subgraph punks created in punks

Next steps:

  1. Run `graph auth` to authenticate with your deploy key.

  2. Type `cd punks` to enter the subgraph.

  3. Run `yarn deploy` to deploy the subgraph.

Make sure to visit the documentation on https://thegraph.com/docs/ for further information.
```

## Make sure it works

Now, it's time for you to verify if you have followed the instructions carefully, click on the button **Test subgraph scaffold** to check for the presence of a scaffolded subgraph.
