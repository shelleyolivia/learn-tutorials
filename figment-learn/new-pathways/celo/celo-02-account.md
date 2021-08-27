Like with most Web 3 protocols, transactions on Celo happen between **accounts**. To create an account, a client generates a **mnemonic** from which it can (re)-create a **public key** and a public address for use with a **wallet**. We're going to learn how to achieve all of this in the next challenge.

------------------------

# Challenge

{% hint style="tip" %}
In `pages/api/celo/account.ts`, complete the code of the function to first create a **mnemonic**, then produce an **address** from the **public key** belonging to the **mnemonic**.
{% endhint %}

**Take a few minutes to figure this out**

```typescript
//...

//...
```

**Need some help?** Check out these links
* [**Documentation for `@iov/crypto`'s BIP39 implementation**](https://iov-one.github.io/iov-core-docs/latest/iov-crypto/classes/bip39.html)
* [**Account example**](https://github.com/enigmampc/SecretJS-Templates/blob/master/2_creating_account/create_account.js)  

{% hint style="info" %}
[**You can join us on Discord, if you have questions**](https://discord.gg/fszyM7K)
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

------------------------

# Solution

```typescript
//...

//...
```

**What happened in the code above?**
* First we create a random **mnemonic** using the `fromMnemonic` method of the `Secp256k1Pen` class.
* Next we deduce the public key from it using the `encodeSecp256k1Pubkey` function.
* Then we deduce the wallet address from it using the `pubkeyToAddress` function.
* Finally we send the mnemonic and address back to the client-side as a JSON object.

{% hint style="tip" %}
Do not forget to fund the newly created wallet using the [celo faucet](https://faucet.secrettestnet.io/) in order to activate it!
{% endhint %}

------------------------

# Make sure it works

Once the code is complete and the file is saved, Next.js will rebuild the API route. Now click on **Generate a Mnemonic** and you should see:

![](../../../.gitbook/assets/pathways/celo/celo-account.png)

-----------------------------

# Next

Before we make our first transfer, let's check that that the account is actually funded by asking the cluster for our balance!
