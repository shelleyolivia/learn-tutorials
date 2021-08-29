Still not sure how to do this? No problem! The solution is below so you don't get stuck.

------------------------

# Challenge

{% hint style="tip" %}
In `pages/api/polkadot/estimate.ts`, complete the code of the function and try to establish your first connection to the celo network. 
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
    const { mnemonic, txAmount } = req.body

    const url = getSafeUrl();
    const provider = new WsProvider(url);
    const api = await ApiPromise.create({ provider: provider })

    // Initialize account from the mnemonic
    const keyring = new Keyring({type: 'sr25519'});
    const account = keyring.addFromUri(mnemonic);

    // Initialize account from the mnemonic
    const recipient = keyring.addFromUri('//Alice');
    const recipientAddr = recipient.address

    // Transfer tokens
    const transfer =  api.tx.balances.transfer(recipientAddr, txAmount)
    const info = await transfer.paymentInfo(account)
    const fees = info.partialFee.toNumber()

    res.status(200).json(fees)
  }
//...
```

**What happened in the code above?**
* First, we create a new `kit` instance.

------------------------

# Make sure it works

Once the code is complete and the file has been saved, refresh the page to see it update & display the current version.

![](../../../.gitbook/assets/pathways/polkadot/polkadot-estimate.gif)

-----------------------------

# Next

Once the code is complete and the file has been saved, refresh the page to see it update & display the current version.
