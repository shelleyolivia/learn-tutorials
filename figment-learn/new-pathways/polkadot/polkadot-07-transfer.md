It’s time to submit your first transactions. In this challenge, we will connect to a Westend node hosted by DataHub and we will transfer our testnet token. As you remember from previous step, we funded our account on the **WESTEND** testnet with 1 **WND**. Now let’s try to transfer some **CELO** tokens to another testnet account.

------------------------

# Challenge

{% hint style="tip" %}
In `pages/api/polkadot/transfer.ts`, complete the code of the function and try to establish your first connection to the celo network. 
{% endhint %}

**Take a few minutes to figure this out**

```typescript
//...
    // Initialize account from the mnemonic
    const keyring = new Keyring({type: 'sr25519'});
    const account = keyring.addFromUri(mnemonic);
  
    // Initialize account from the mnemonic
    const recipient = keyring.addFromUri('//Alice');
    const recipientAddr = recipient.address

    // Transfer tokens
    const transfer = undefined;
    const hash = await transfer.signAndSend(account);
    
    res.status(200).json(hash.toString())
  }
//...
```

**Need some help?** Check out these links
* [**Polkadot{.js} documentation**](https://polkadot.js.org/docs/)  
* [**code examples**](https://polkadot.js.org/docs/api/examples/promise/)  

{% hint style="info" %}
[**You can join us on Discord, if you have questions**](https://discord.gg/fszyM7K)
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

------------------------

# Solution

```typescript
//...
    // Initialize account from the mnemonic
    const keyring = new Keyring({type: 'sr25519'});
    const account = keyring.addFromUri(mnemonic);
  
    // Initialize account from the mnemonic
    const recipient = keyring.addFromUri('//Alice');
    const recipientAddr = recipient.address

    // Transfer tokens
    const transfer = await api.tx.balances.transfer(recipientAddr, txAmount)
    const hash = await transfer.signAndSend(account)
    
    res.status(200).json(hash.toString())
  }
//...
```

**What happened in the code above?**
* First, we create a new instance of the api.
* Next, we call `transfer` method  of the `const.balances` module, passing:
  * The recipient Address
  * The amount
* Finaly, we sign and send the transaction passing our account as an argument of `signAndSend` method.

------------------------

# Make sure it works

Once the code is complete and the file is saved, Next.js will rebuild the API route.
* Fill in the amount of **Planck** you want to send.
* Click on **Submit Transfer**.

![](../../../.gitbook/assets/pathways/polkadot/polkadot-transfer.gif)

-----------------------------

# Conclusion

Congratulations! We have now completed the Polkadot Pathway! 

There are many things that are beyond the scope of this Pathway, but the links provided to expand on some of the concepts contained in the pathway should at least provide ample starting points for further study. Consider refining and playing with the code from the Pathway, to see what can be built out of these foundational blocks of Polkadot.

To get the most out of the collaborative experience at Learn, join the [Community Forum](https://community.figment.io), also join us on [Discord](https://discord.com/invite/fszyM7K) to stay up-to-date with the latest information and educational resources dealing with web3 technologies, across a wide range of protocols.
