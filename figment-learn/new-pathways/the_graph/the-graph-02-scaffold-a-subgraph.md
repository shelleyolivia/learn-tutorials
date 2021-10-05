# What's a subgraph?

If a Graph node is a Christmas tree then subgraphs are like garlands. They represent structured datasets, tied up to a Graph node, and easily accessible using the GraphQL query language for APIs. The same way you can add garlands to your Christmas tree you can also add subgraphs to your local Graph node. Any subgraphs you add can be customized:

- By their shape: The way you structure your data.
- By adding decorations: We call them Entities (more about this later).
- The way you connect your decorations reflects how they are related, your entities upkeep.

As the garlands are made by _staking_ fil, subgraphs are built up by _staking_ events triggered from smart contracts residing on a blockchain. This means we need at least one smart contract to build up a subgraph for.

Here, we'll use the [CryptoPunk](https://www.larvalabs.com/cryptopunks) smart contract, as CryptoPunks are a well known example of NFTs.

# Initialize an empty subgraph

## Install the Graph client

Fortunately, we won't have to start from scratch, there are tools to scaffold a new subgraph. To install the Graph CLI, run:

```text
yarn global add @graphprotocol/graph-cli
```

After the install process is complete, check for the presence of the Graph CLI binary in your PATH by running: 

```text
graph --version
```

This should output the current version of the graph-cli, `0.22.1` at the time of writing this tutorial.

## Scaffold your subgraph

For learning purposes, we're going to use more options than strictly necessary to explain them all. To scaffold your subgraph from the existing smart contract, run this command in your terminal:

```text
graph init --allow-simple-name --from-contract 0xb47e3cd837dDF8e4c57F05d70Ab865de6e193BBB \
  --index-events --contract-name punks --network mainnet --node http://localhost:8020/ punks
```

> _NOTE_: Linux and macOS use the backslash character \ for multi-line input. Windows uses the ^ character. If you paste this command into a Windows terminal (PowerShell, cmd.exe or Windows Terminal), replace the \ with ^ so it will work properly.

Once the command is run, it will prompt you to confirm the information, you can just accept the suggested input by pressing enter five (5) times. The output will be similar:

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

Quick overview:

- `graph init` will initialize an empty subgraph.
- `--allow-simple-name` simplifies the naming convention of our local graph.
- `--from-contract` uses an already deployed contract at the specified address.
- `--index-event` creates entities from events (not a good idea).
- `--contract-name punks` sets the contract name to the supplied string, for example: punks
- `--network mainnet` tells the Graph CLI to look on Mainnet Ethereum to find the contract ABI
- `--node http://localhost:8020/` will prepare our script to deploy to our local graph node
- `punks` is the name of the folder under which the files are created

Curious readers will have noticed the argument passed to the `--from-contract` option. The argument here is an Ethereum address. You guessed it! It's the address of the CryptoPunk smart contract.

Now we can go into the newly created folder, install the dependencies and then start hacking our subgraph manifest.

```bash
cd punks
yarn
```

# Time to verify your work

Now, it's time for you to verify if you have followed the instructions carefully, click on the button **Test subgraph scaffold** to check for the presence of a scaffolded subgraph.

{% hint style="info" %}
You can [**join us on Discord**](https://discord.gg/fszyM7K), if you have questions or want help completing the tutorial.
{% endhint %}

# Conclusion

Nice, you made it! You scaffolded a subgraph directory from the [CryptoPunk](https://www.larvalabs.com/cryptopunks) smart contract. As is, it leaves a lot to be desired. Let's see what we can do to make it more interesting by hacking the manifest file in the next step.