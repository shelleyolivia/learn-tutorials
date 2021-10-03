# Manifest, so what?

To capture and process information from a blockchain, we must:

- Define which information we're looking for
- Know what this information is made of
- Define the shape of the processed information when we are finished with it

The right place to do so is within the `subgraph.yaml` file, the manifest of our subgraph.

# Defining the manifest

## The scaffolded subgraph.yaml

```yaml
specVersion: 0.0.2
schema:
  file: ./schema.graphql
dataSources:
  - kind: ethereum/contract
    name: punks
    network: mainnet
    source:
      address: "0xb47e3cd837dDF8e4c57F05d70Ab865de6e193BBB"
      abi: punks
    mapping:
      kind: ethereum/events
      apiVersion: 0.0.4
      language: wasm/assemblyscript
      entities:
        - Assign
        - Transfer
        - PunkTransfer
        - PunkOffered
        - PunkBidEntered
        - PunkBidWithdrawn
        - PunkBought
        - PunkNoLongerForSale
      abis:
        - name: punks
          file: ./abis/punks.json
      eventHandlers:
        - event: Assign(indexed address,uint256)
          handler: handleAssign
        - event: Transfer(indexed address,indexed address,uint256)
          handler: handleTransfer
        - event: PunkTransfer(indexed address,indexed address,uint256)
          handler: handlePunkTransfer
        - event: PunkOffered(indexed uint256,uint256,indexed address)
          handler: handlePunkOffered
        - event: PunkBidEntered(indexed uint256,uint256,indexed address)
          handler: handlePunkBidEntered
        - event: PunkBidWithdrawn(indexed uint256,uint256,indexed address)
          handler: handlePunkBidWithdrawn
        - event: PunkBought(indexed uint256,uint256,indexed address,indexed address)
          handler: handlePunkBought
        - event: PunkNoLongerForSale(indexed uint256)
          handler: handlePunkNoLongerForSale
      file: ./src/mapping.ts
```

Let's have a look at what this all means. Firstly, `.yaml` is the file extension for YAML ([**Y**AML **A**in't **M**arkup **L**anguage](https://www.cloudbees.com/blog/yaml-tutorial-everything-you-need-get-started)) which is an easy to read file format commonly used for configuration files. 

In order to save time when indexing the datasource, we must specify a starting block (or else the node would start indexing from the genesis block, which would take a very long time and not be useful to us). To specify a sensible starting block, insert the line `startBlock: 3914495` into the `subgraph.yaml` file for the datasource of the "punks" contract. Block 3914495 occurred on June 22, 2017 - and contained the deployment of the CryptoPunksMarket contract.

Next, we need to make sure the datasource mapping includes the entities "Punk" and "Account".

Finally, we will want to include an event handler for the **Assign( indexed address, uint256)** event.

## The changed subgraph.yaml

```yaml
specVersion: 0.0.2
schema:
  file: ./schema.graphql
dataSources:
  - kind: ethereum/contract
    name: punks
    network: mainnet
    source:
      address: '0xb47e3cd837dDF8e4c57F05d70Ab865de6e193BBB'
      abi: punks
      startBlock: 3914495
    mapping:
      kind: ethereum/events
      apiVersion: 0.0.5
      language: wasm/assemblyscript
      entities:
        - Account
        - Punk
      abis:
        - name: punks
          file: ./abis/punks.json
      eventHandlers:
        - event: Assign(indexed address,uint256)
          handler: handleAssign
      file: ./src/mapping.ts
```

# Time to verify your work

Now, it's time for you to verify if you have followed the instructions carefully, click on the button **Test manifest** to check the manifest file. 

{% hint style="info" %}
You can [**join us on Discord**](https://discord.gg/fszyM7K), if you have questions or want help completing the tutorial.
{% endhint %}

# Conclusion

Nice, you made it! You were able to add the correct information to the manifest to get the subgraph working from the [CryptoPunk](https://www.larvalabs.com/cryptopunks) smart contract.