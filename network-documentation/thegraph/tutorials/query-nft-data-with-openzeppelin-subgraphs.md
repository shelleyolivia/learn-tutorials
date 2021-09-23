# Introduction

In this tutorial you will learn how to create a subgraph from an NFT smart contract running on Ethereum mainnet, deploy it to the Subgraph Studio, and then use the Subgraph Studio Playground to query the subgraph.

![Subgraph Studio](../../../.gitbook/assets/graph.png)

Topics covered in this tutorial:

- Retrieving an NFT smart contract address from OpenSea
- Creating a subgraph in the Subgraph Studio
- Creating a subgraph from an NFT smart contract
- Deploying the subgraph to Subgraph Studio
- Querying the subgraph using Subgraph Studio Playground

# Requirements

- You need to have a recent version of Node.js installed. We recommend using v14.17.6 LTS for compatibility.

# Project setup

Run the below command to install the graph cli globally. It's required to deploy your subgraph.

```text
npm i -g @graphprotocol/graph-cli@0.21.1
```

You need to use `v0.21.1` and not the latest version, because the latest version breaks compatibility with OpenZeppelin Subgraphs.

# Getting the NFT smart contract address

Head over to [OpenSea](https://opensea.io/assets), the largest NFT marketplace.

![OpenSea assets](../../../.gitbook/assets/opensea_assets.png)

As seen from the screenshot above, there are over 20 million NFTs on sale!

Click on the **Collections** tab on the left. We shall see many of the popular NFT collections like CryptoPunks, Art Blocks Curated, Galaxy-Eggs, and so on. Let's choose **Galaxy-Eggs** NFT collection for this tutorial.

Click on **Galaxy-Eggs** link, and you shall be taken to <https://opensea.io/collection/galaxyeggs9999>.

![OpenSea Galaxy-Eggs](../../../.gitbook/assets/opensea_galaxyeggs.png)

Click on any of the NFT, and you shall see a similar page like shown below.

![OpenSea Galaxy-Eggs #7836](../../../.gitbook/assets/opensea_galaxyeggs_7836.png)

If you click on the **Details** tab, you shall get its contract address: <https://etherscan.io/address/0xa08126f5e1ed91a635987071e6ff5eb2aeb67c48>.

# Creating the project in Subgraph Studio

First, you will want to head over to the Subgraph Studio at <https://thegraph.com/studio/>.

![Login to Subgraph Studio](../../../.gitbook/assets/graph_connect.png)

Click on the **Connect Wallet** button. Choose a Metamask wallet to login with. Once you are authenticated, you shall see the below screen, where you can create your first subgraph.

![Create your first subgraph](../../../.gitbook/assets/graph_create_subgraph.png)

Next, you need to give your subgraph a name. Give the name as **GalaxyEggNFT**. Once that's done, you can see details about the subgraph like your deploy key, the subgraph slug and status.

![GalaxyEggs NFT subgraph](../../../.gitbook/assets/graph_galaxyeggs_nft.png)

# Creating the subgraph

Create a directory called `galaxyEggsNFT`. Initialize it as a new `npm` project by running:

```text
npm init --yes
```

Next, you need to install the OpenZeppelin Subgraphs package:

```text
npm install --save-dev @openzeppelin/subgraphs
```

To store the subgraph configuration, create a file called `subgraphconfig.json` with the following contents:

```json
{
  "output": "generated/",
  "chain": "mainnet",
  "datasources": [
    { "address": "0xa08126f5e1ed91a635987071e6ff5eb2aeb67c48", "startBlock": 13200000, "module": [ "erc721", "ownable" ] }
  ]
}
```

- **output**: the directory where your subgraph shall be stored
- **datasources**: an array of `datasource`, where each datasource defines the following properties:
  - **address** - smart contract address to create the subgraph from
  - **startBlock** (optional) - block to start the subgraph indexing from
  - **module** - array of modules that we want to index

`startBlock` is optional, but it is recommended to speed up the indexing process.

OpenZeppelin Subgraphs supports the following modules:
- `accesscontrol`
- `erc20`
- `erc721`
- `governor`
- `ownable`
- `pausable`
- `timeblock`

Since NFTs are `erc721` modules, we shall use that for indexing, along with the `ownable` module.

To build the subgraph, run the following command:

```text
npx graph-compiler --config subgraphconfig.json --include node_modules/@openzeppelin/subgraphs/src/datasources --export-schema --export-subgraph
```

You shall get the below output:

```text
- Schema exported to generated/schema.graphql
- Manifest exported to generated/subgraph.yaml
```

It shall create a directory called `generated` with two files:
- `subgraph.yaml`: This stores the [subgraph manifest](https://thegraph.academy/developers/working-with-the-graph/)
- `schema.graphql`: This defines the data to be stored and how it can be queried using GraphQL

Now we are ready to deploy the subgraph.

# Deploying the subgraph

Run the following command to set the deploy key for your subgraph project.

```text
graph auth --studio <DEPLOY_KEY>
```

Replace `<DEPLOY_KEY>` with your own deploy key that you got from <https://thegraph.com/studio/subgraph/galaxyeggnft/>.

You should see the following output:

```text
Deploy key set for https://api.studio.thegraph.com/deploy/
```

Once that is done, we can run the following commands to deploy the subgraph to Subgraph Studio:

```text
cd generated
graph deploy --studio galaxyeggnft
```

You can choose `v1.0.0` when you are prompted for a versibon label. You should see the following output:

```text
✔ Version Label (e.g. v0.0.1) · v1.0.0
  Skip migration: Bump mapping apiVersion from 0.0.1 to 0.0.2 (graph-ts dependency not installed yet)
  Skip migration: Bump mapping apiVersion from 0.0.2 to 0.0.3 (graph-ts dependency not installed yet)
  Skip migration: Bump mapping apiVersion from 0.0.3 to 0.0.4 (graph-ts dependency not installed yet)
  Skip migration: Bump mapping specVersion from 0.0.1 to 0.0.2
✔ Apply migrations
✔ Load subgraph from subgraph.yaml
  Compile data source: erc721 => build/erc721/erc721.wasm
  Compile data source: ownable => build/ownable/ownable.wasm
✔ Compile subgraph
  Copy schema file build/schema.graphql
  Write subgraph file build/node_modules/@openzeppelin/contracts/build/contracts/IERC721Metadata.json
  Write subgraph file build/node_modules/@openzeppelin/contracts/build/contracts/Ownable.json
  Write subgraph manifest build/subgraph.yaml
✔ Write compiled subgraph to build/
  Add file to IPFS build/schema.graphql
                .. QmPDKw3SDB138wUFYYvzAN1GatHRX6NvFwnDcvDfZsjwYp
  Add file to IPFS build/node_modules/@openzeppelin/contracts/build/contracts/IERC721Metadata.json
                .. QmXVNNtLfETmLrarDMwR18Tk2kNUXyczLf7LdES4aFZkLi
  Add file to IPFS build/node_modules/@openzeppelin/contracts/build/contracts/Ownable.json
                .. QmVjhxbJvwq39tGZCNxTDuY6rjW5HYiHzCm9kQnm1bLJ1M
  Add file to IPFS build/erc721/erc721.wasm
                .. QmUKy3JTwyeYNUn5BMa7SAWj3C6PAx47XBJQNfNG1X6Jb4
  Add file to IPFS build/ownable/ownable.wasm
                .. QmQg39Jp9zLUdShCcXoYjrnkhevLLpYerT3RcvYaxgZPph
✔ Upload subgraph to IPFS

Build completed: QmYDdVUPh6br4VqrgQzhM2i4Pedw7ysvDmhovvYVcA1Nu2

Deployed to https://thegraph.com/studio/subgraph/galaxyeggnft

Subgraph endpoints:
Queries (HTTP):     https://api.studio.thegraph.com/query/8676/galaxyeggnft/v1.0.0
Subscriptions (WS): https://api.studio.thegraph.com/query/8676/galaxyeggnft/v1.0.0
```

You have now deployed the subgraph to your Subgraph Studio account!

It will trigger a syncing process, where The Graph nodes will inspect historical blocks of Ethereum mainnet blockchain to retrieve any data for this subgraph. New blocks are inspected as soon as they are mined. Wait for the syncing process to be completed.

You shall also have access to the Playground, where you can run your queries, like shown below.

![GalaxyEggs NFT Subgraph](../../../.gitbook/assets/graph_galaxyeggs_nft_subgraph.png)

# Querying the NFT data

To retrieve the NFTs, use the below query in the Playground:

```text
{
  erc721Tokens {
    identifier
    uri
  }
}
```

You will get an output like:

```text
{
  "data": {
    "erc721Tokens": [
      {
        "identifier": "0",
        "uri": "ipfs://QmfB3KH8GuZA9xoTCNwwiMEMk9hZBUaUZnMMWKra57oUse/0.json"
      },

      ...

    ]
}
```

If you visit the above IPFS link, you shall get the below json details:

```text
{"name": "Galaxy Egg #0", "description": "(art)tificial is an art studio that explores the boundaries of technology and art. Our first project is Galaxy Eggs - a generative collection of 9,999 Eggs of the metaverse that live on the Ethereum Blockchain. Our Art Director, Gal Barkan, has been creating futuristic and sci-fi art for the past 20 years - this collection is the culmination of a lifetime of work on one hand, and the beginning of a new chapter in taking part in the creation of the metaverse. For more info about (art)ificial and Galaxy Eggs, visit - artificial.art", "image": "ipfs://QmUM1uGBE6H8pQ2zQSUj2BTzbpLSvuX6jtpBCB38xSiz2q/"}
```

To retrieve the NFT purchases, run the below query:

```text
{
  transactions {
    id
  }
}
```

You will get an output like:

```text
{
  "data": {
    "transactions": [
      {
        "id": "0x0003b95db41b4bdadddb234d4d932fdc26bf9890267073e488e4b6f9fdb27400"
      },
      {
        "id": "0x000a9c8ddadffcd8832723fd2227c98d5401df2f83f9297205386fc59e8de13b"
      },

      ...

    ]
  }
}
```

Each of those `id`s are transaction hashes in Ethereum mainnet.

As an example, if you visit https://etherscan.io/tx/0x0003b95db41b4bdadddb234d4d932fdc26bf9890267073e488e4b6f9fdb27400, you can see ifs an NFT purchase of **Galaxy Egg #8331**!

You can get the above details from the Playground using the below query:

```text
{
  erc721Transfers (orderBy: timestamp, orderDirection: desc) {
    token {
      identifier
    }
    to {
      id
    }
    transaction {
      id
    }
  }
}
```

You shall get the below output:

```text
{
  "data": {
    "erc721Transfers": [
      {
        "to": {
          "id": "0x98c2f3a23a967ed100b6c51dcab8e354804e05d1"
        },
        "token": {
          "identifier": "2174"
        },
        "transaction": {
          "id": "0x31663947a5619c75217d127caa7c128d868c5b4cc3d8145d9b78e0e9a6e4a155"
        }
      },

      ...

    ]
  }
}
```

If you visit https://etherscan.io/tx/0x31663947a5619c75217d127caa7c128d868c5b4cc3d8145d9b78e0e9a6e4a155, you can see its the sale of **Galaxy Eggs #2174** made by `0x98c2f3a23a967ed100b6c51dcab8e354804e05d1`, which is exactly what is in the output!

# Conclusion

Congratulations on finishing this tutorial! You have learned how to retrieve the smart contract address of an NFT collection from OpenSea. You also learned how to create and deploy a subgraph for an NFT collection on Subgraph Studio, as well as query the subgraph using the Subgraph Studio Playground.

# About the Author

I'm Robin Thomas, a blockchain enthusiast with few years of experience working with various blockchain protocols. Feel free to connect with me on [GitHub](https://github.com/robin-thomas).
