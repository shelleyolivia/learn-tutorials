# 

Like with most Web 3 protocols, transactions on Solana happen between **accounts**.  To create an account a client generates a **keypair**, which has a **public's key** (or **address**, used to identify and lookup an account) and a **secret's key** used to sign transactions.

----------------------------------

## The challenge

{% hint style="warning" %}
In`pages/api/solana/keypair.ts`, implement `keypair` and parse the keypair to extract the address as a string and render it in the webpage
{% endhint %}

**Take a few minutes to figure this out.**

```tsx
//...
  try {
    const keypair = undefined
    const address = undefined
    const secret = JSON.stringify(Array.from(keypair?.secretKey))

    res.status(200).json({
        secret,
        address,
    });
  }
//...
```

**Need some help?** Check out those two links
* [Generate a`Keypair`](https://solana-labs.github.io/solana-web3.js/classes/Keypair.html#constructor)  
* [Convert a`PublicKey`to a string](https://solana-labs.github.io/solana-web3.js/classes/PublicKey.html#tostring)

{% hint style="info" %}
[You can **join us on Discord**, if you have questions](https://discord.gg/fszyM7K)
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

----------------------------------

## The solution

```tsx
//...
  try {
    const keypair = Keypair.generate();
    const address = keypair?.publicKey.toString()
    const secret = JSON.stringify(Array.from(keypair?.secretKey))

    res.status(200).json({
        secret,
        address,
    });
  }
//...
```

**What happened in the code above?**

* We used the JS API's `Keypair` to generate a keypair
* Once React re-renders, we parse the keypair object to extract the public key using `keypair.publicKey`
* But this value is a `Buffer` so we need to convert it to a string using `PublicKey.toString()`

----------------------------------

## Make sure it works

Once you have the code above saved, click on **Generate a Keypair** and you should see:

![](../../../.gitbook/assets/solana-keypair-v3.gif)

**Try and click on "Generate a Keypair" again. And again. And again!** Every time it will generate a new one with virtually no risk that someone else creates the same one as you. That's cause the domain of possible addresses is so vast that the probability of two identical addresses being generated is ridiculously small.

----------------------------------

## Next

Now that we have an account, we can fund it so we can start playing around with tokens!
