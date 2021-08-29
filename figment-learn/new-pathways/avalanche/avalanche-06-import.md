Still not sure how to do this? No problem! The solution is below so you don't get stuck.

------------------------

# Challenge

{% hint style="tip" %}
In `pages/api/avalanche/import.ts`, complete the code of the function and try to establish your first connection to the celo network. 
{% endhint %}

**Take a few minutes to figure this out**

```typescript
//...

//...
```

**Need some help?** Check out these links
* [**ContractKit usage**](https://docs.celo.org/developer-guide/contractkit/usage)  

{% hint style="info" %}
[**You can join us on Discord, if you have questions**](https://discord.gg/fszyM7K)
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

------------------------

# Solution

```typescript
//...
  try {
    const { secret } = req.body
    const client = getAvalancheClient()
    const binTools = BinTools.getInstance()

    // Initialize chain components
    const xChain = client.XChain()
    const xKeychain = xChain.keyChain()
    xKeychain.importKey(secret)

    const cChain = client.CChain()
    const cKeychain = cChain.keyChain()
    cKeychain.importKey(secret)
    
    // Derive Eth-like address from the private key
    const keyBuff = binTools.cb58Decode(secret.split('-')[1])
    // @ts-ignore
    const ethAddr = Address.fromPrivateKey(Buffer.from(keyBuff, "hex")).toString("hex")
    console.log("Derived Eth address:", ethAddr)
    
    const sourceChain = await client.Info().getBlockchainID("X")

    // Fetch UTXOs (i.e unspent transaction outputs)
    const { utxos } = await cChain.getUTXOs(cKeychain.getAddressStrings(), sourceChain)

    // Generate an unsigned import transaction
    const importTx = await cChain.buildImportTx(
        utxos,
        ethAddr,
        cKeychain.getAddressStrings(),
        sourceChain,
        cKeychain.getAddressStrings()
    )

    // Sign and send import transaction
    const hash = await cChain.issueTx(importTx.sign(cKeychain))
    console.log("C-Chain import TX:", hash)
    console.log(` - https://explorer.avax-test.network/tx/${hash}`)

    res.status(200).json(hash)
  } 
//...
```

**What happened in the code above?**
* First, we create a new `kit` instance.

------------------------

# Make sure it works

Once the code is complete and the file has been saved, refresh the page to see it update & display the current version.

![](../../../.gitbook/assets/pathways/avalanche/avalanche-import.gif)

-----------------------------

# Next

Once the code is complete and the file has been saved, refresh the page to see it update & display the current version.
