Like with most Web 3 protocols, transactions on NEAR happen between **accounts**. To create an account, one first need to generate a **keypair** which has a **public key** (or **address**, used to identify and lookup an account) and a **secret key** used to sign transactions.

------------------------

# Challenge

{% hint style="tip" %}
In `pages/api/near/keypair.ts`, implement `keypair` and retrieve the string formatted representation of the keypair into the variable `secret`.
{% endhint %}

**Take a few minutes to figure this out**

```typescript
  try {
    const keypair = undefined;
    const secret = undefined;
    return res.status(200).json(secret);
  } 
```

**Need some help?** Check out those two links
* [The `KeyPair` class](https://near.github.io/near-api-js/modules/utils_key_pair.html)  
* [Retrieve the `secret`](https://near.github.io/near-api-js/classes/utils_key_pair.keypaired25519.html#tostring)

{% hint style="info" %}
[You can **join us on Discord**, if you have questions](https://discord.gg/fszyM7K)
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

------------------------

# Solution

```typescript
  try {
    const keypair = KeyPair.fromRandom('ed25519');
    const secret = keypair.toString();
    return res.status(200).json(secret);
  } 
```

**What happened in the code above?**
* First, we create a random `keypair` using the `ed25519` cryptographic curve.
* Next, we retrieve the string formatted representation of the `secret` key.

------------------------

# Make sure it works

Once the code is complete and the file is saved, Next.js will rebuild the API route. Now click on **Generate a Keypair** and you should see:

![](../../../.gitbook/assets/pathways/near/near-keypair-v2.gif)

-----------------------------

# Next

Now that we have an account, we're going to associate to an account name something almost only possible with NEAR. But before we need to find an available account name. Let's go!
