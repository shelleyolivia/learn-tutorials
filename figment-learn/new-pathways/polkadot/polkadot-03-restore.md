Still not sure how to do this? No problem! The solution is below so you don't get stuck.

------------------------

# Challenge

{% hint style="tip" %}
In `pages/api/polkadot/restore.ts`, complete the code of the function and try to establish your first connection to the celo network. 
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
    const { mnemonic } = req.body
    
    const keyring = new Keyring({type: 'sr25519'});
    const account = keyring.addFromUri(mnemonic);
    const address = account.address;
    const jsonWallet = JSON.stringify(
      keyring.toJson(account.address), 
      null, 
      2
    )
  
    res.status(200).json({
      address,
      mnemonic,
      jsonWallet,
    });
  }
//...
```

**What happened in the code above?**
* First, we create a new `kit` instance.

------------------------

# Make sure it works

Once the code is complete and the file has been saved, refresh the page to see it update & display the current version.

![](../../../.gitbook/assets/pathways/polkadot/polkadot-restore.gif)

-----------------------------

# Next

Once the code is complete and the file has been saved, refresh the page to see it update & display the current version.
