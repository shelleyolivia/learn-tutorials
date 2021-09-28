# What's a subgraph?

If a graph node is a Christmas tree then subgraph are garland. There represents structured dataset, tied up to a graph node, and easily accessible using GraphQL. The same way you can add garland to your Christmas tree you can also add subgraphs to your local graph node. Any subgraphs you add, can be customized:

- By their shape: The way your structure your data
- By adding Christmas' balls: We call them entities (more about this later)
- The way you connect your Christmas' balls reflect the relation your entities upkeep.

As the garland are made by _staking_ fil, subgraph are build up by _staking_ events' triggered from smart-contract residing on-chain. This mean we need at least one smart contract to build up a subgraph.

Here, we'll use the [CryptoPunk](https://www.larvalabs.com/cryptopunks) smart-contract.

# Initialize an empty subgraph

## Install the graph client

Fortunately, we won't have to start from scratch, there is a nice tool to scaffold a new subgraph. To install it, run:

```text
npm install -g @graphprotocol/graph-cli
```

To check if everything is ok:

```text
graph --version
```

It should output the version of the graph-cli, here for me it was `0.22.1`

## Scaffold your subgraph

For learning purpose we're going to use more options than needed, but all of them will be explained. To scaffold your subgraph, run :

```text
$ graph init --allow-simple-name --from-contract 0xb47e3cd837dDF8e4c57F05d70Ab865de6e193BBB \
  --index-events --contract-name punks --network mainnet --node http://localhost:8020/ punks
```

Quick overview:

- `graph init` will initialize an empty subgraph
- `--allow-simple-name` option will simply the naming convention of our local graph
- `--from-contract` option allow us to select an already deployed contract
- `--index-event` option will create entity from event (not a good idea)
- `--contract-name punk` rename the contract punks
- `--network mainnet` tell to look at mainnet ethereum to find the contract abi
- `--node http://localhost:8020/` will prepare our script to deploy to our local graph node
- `punk` is the name of the folder under which the files are created

Curious reader have notice the argument passed to `--from-contract` option. The argument here is an Ethereum address. You guessed right! It's the address of the [CryptoPunk](https://www.larvalabs.com/cryptopunks) smart-contract.

Now we can go into the newly created folder, install dependencies and start hacking our subgraph.

```bash
cd punks
yarn
```

# Time to verify your work

Now, it's time for you to verify if you have follow carefully our instruction, click on the button `Test subgraph scaffold` and see if magic happen.

{% hint style="info" %}
You can [**join us on Discord**](https://discord.gg/fszyM7K), if you have questions or want help completing the tutorial.
{% endhint %}

# Conclusion

Nice you made it, you scaffolded a subgraph directory from the [CryptoPunk](https://www.larvalabs.com/cryptopunks) smart-contract. As is, it far to be enough,
