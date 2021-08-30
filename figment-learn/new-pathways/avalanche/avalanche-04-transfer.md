We already know that Avalanche is not your typical blockchain, with P/X/C chains supporting various operations. Each of these chains also has its own set of transaction types, however, we are looking to create a very simple one - a token transfer, and specifically on X-Chain.

In layman's terms: We will be sending some AVAX tokens from address A to address B to simulate a payment for goods/services. Simple as that.

------------------------

# Challenge

{% hint style="tip" %}
In `pages/api/avalanche/transfer.ts`, complete the code of the function and try to make your first transfer on the Avalanche network. 
{% endhint %}

**Take a few minutes to figure this out**

```typescript
//...
    //  using keychain load the private key to sign transaction
    undefined

    // Fetch UTXO (i.e unspent transaction outputs)
    const { utxos } = undefined;

    // Determine the real asset ID from its symbol/alias
    const binTools = BinTools.getInstance()
    const assetInfo = await chain.getAssetDescription("AVAX")
    const assetID = binTools.cb58Encode(assetInfo.assetID)

    // Create a new transaction
    const transaction = await chain.buildBaseTx(undefined)

    // sign transaction and send
    undefined;
    undefined;

    res.status(200).json(hash)
  }
//...
```

**Need some help?** Check out these links
* [**Transfer example**](https://github.com/ava-labs/avalanchejs/tree/master/examples/avm)  
* [**Manage X-Chain Keys**](https://docs.avax.network/build/tools/avalanchejs/manage-x-chain-keys)
* [**What The Heck is UTXO**](https://medium.com/bitbees/what-the-heck-is-utxo-ca68f2651819)


{% hint style="success" %}
[**You can join us on Discord, if you have questions**](https://discord.gg/fszyM7K)
{% endhint %}


Still not sure how to do this? No problem! The solution is below so you don't get stuck.

------------------------

# Solution

```typescript
//...
  try {
    const { secret, amount, recipeint, address } = req.body
    const client = getAvalancheClient()
    const chain = client.XChain(); 
    const keychain = chain.keyChain()
    keychain.importKey(secret)

    // Fetch UTXO (i.e unspent transaction outputs)
    const { utxos } = await chain.getUTXOs(address)

    // Determine the real asset ID from its symbol/alias
    // We can also get the primary asset ID with chain.getAVAXAssetID() call
    const binTools = BinTools.getInstance()
    const assetInfo = await chain.getAssetDescription("AVAX")
    const assetID = binTools.cb58Encode(assetInfo.assetID)

    // Create a new transaction
    const transaction = await chain.buildBaseTx(
      utxos, // unspent outputs	
      new BN(amount), // transaction amount formatted as a BigNumber
      assetID, // AVAX asset
      [recipeint], // addresses to send the funds
      [address], // addresses being used to send the funds from the UTXOs provided
      [address], // addresses that can spend the change remaining from the spent UTXOs
    )

    // sign transaction and send
    const signedTx = transaction.sign(keychain)
    const hash = await chain.issueTx(signedTx)

    res.status(200).json(hash)
  }
//...
```

**What happened in the code above?**
* First, calling `importKey(secret)` we pass allow keypair to sign transaction
* Next, we fetch the latest unspent transaction outputs with `getUTXOs`.
* Next, we determine the **assetID** using `BinTools.cb58Encode` method.
* Next, we build a base transaction using `buildBasTx` passing:
  * The unspent outputs
  * The transaction amout formatted as BigNumber
  * The assetID
  * The recipent address
  * The sender address
  * The payer address
* Finaly, we sign and send the transaction and return the transaction hash

{% hint style="success" %}
A UTXO is an unspent transaction output. In an accepted transaction in a valid blockchain payment system, only unspent outputs can be used as inputs to a transaction. When a transaction takes place, inputs are deleted and outputs are created as new UTXOs that may then be consumed in future transactions.
{% endhint %}


{% hint style="tip" %}
Remember, **AVAX** asset is using a 9-digit denomination and **AVAX** is the primary asset on the X chain.
{% endhint %}


------------------------

# Make sure it works

Once the code is complete and the file is saved, Next.js will rebuild the API route.
* Fill in the amount of **nAVAX** you want to send.
* Click on **Submit Transfer**.


![](../../../.gitbook/assets/pathways/avalanche/avalanche-transfer.gif)

-----------------------------

# Next

We've learned how to prepare, sign and broadcast a simple transaction on Avalanche. With basically a few lines of code you can transfer funds on the network, though Avalanche.js provides a wide range of examples on how to construct a transaction with a bit more complex properties, for different use cases. Our next topic is the cross-chain transfers.
