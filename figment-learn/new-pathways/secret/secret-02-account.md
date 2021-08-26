Like with most Web 3 protocols, transactions on Secret happen between **accounts**. To create an account, a client generates a **mnemonic** from it we can (re)-create a **public key** and an address for our **wallet**. We're going to learn how to achive all of this in the next challenge.

------------------------

# Challenge

{% hint style="tip" %}
In `pages/api/secret/account.ts`,complete the code of the function and first create a **mnemonic** then deduce the wallet's address from it.
{% endhint %}

**Take a few minutes to figure this out**

```typescript
//...
  try {
    const mnemonic = Bip39.encode(Random.getBytes(16)).toString();
    const signingPen = await Secp256k1Pen.fromMnemonic(mnemonic)
    const pubkey = encodeSecp256k1Pubkey(signingPen.pubkey);
    const address = pubkeyToAddress(pubkey, 'secret');
    res.status(200).json({mnemonic, address})
  }
//...
```

**Need some help?** Check out those two links
* [**Account example**](https://github.com/enigmampc/SecretJS-Templates/blob/master/2_creating_account/create_account.js)  
* [**Documentation of `secrectjs`**](https://github.com/enigmampc/SecretNetwork/tree/master/cosmwasm-js/packages/sdk)  

{% hint style="info" %}
[**You can join us on Discord, if you have questions**](https://discord.gg/fszyM7K)
{% endhint %}

Still not sure how to do this ? No problem! The solution is below so you don't get stuck.

------------------------

# Solution

```typescript
  try {
    const mnemonic = Bip39.encode(Random.getBytes(16)).toString();
    const signingPen = await Secp256k1Pen.fromMnemonic(mnemonic)
    const pubkey = encodeSecp256k1Pubkey(signingPen.pubkey);
    const address = pubkeyToAddress(pubkey, 'secret');
    res.status(200).json({mnemonic, address})
  }
```

**What happened in the code above?**
* First, we create a random **mnemonic** using `fromMnemonic` method of `Secp256k1Pen` class.
* Next, we deduce the public key from it using `encodeSecp256k1Pubkey` method.
* Next, we deduce the wallet address from it using `pubkeyToAddress` method.

{% hint style="tip" %}
Do not forget to the fund the newly create wallet using the secret faucet in order to activate it!.
{% endhint %}

------------------------

# Make sure it works

Once the code is complete and the file is saved, Next.js will rebuild the API route. Now click on **Generate a Mnemonic** and you should see:

![](../../../.gitbook/assets/pathways/secret/secret-keypair.gif)

-----------------------------

# Next

Now that we have an account, we're going to associate to an account name something almost only possible with NEAR. But before we need to find an available account name. Let's go!
