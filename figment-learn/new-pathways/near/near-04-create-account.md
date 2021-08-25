Unlike with most Web 3 protocols, NEAR uses human readable account IDs instead of a public key hash. You can link to a NEAR account as much `keypair` as you want. Here, we're going to Learn how to create an **NEAR** account. 

------------------------

# Challenge

{% hint style="tip" %}
In `pages/api/near/create-account.ts`, complete the code of the function.
{% endhint %}

**Take a few minutes to figure this out**

```typescript
try {
    const { freeAccountId, publicKey, network }: AccountReq = req.body
    const config = configFromNetwork(network);
    const near = await connect(config);
    undefined;
    return res.status(200).json(freeAccountId)
}
```

**Need some help?** Check out those two links
* [`createAccount` method](https://near.github.io/near-api-js/classes/near.near-1.html#createaccount)  
* [`NEAR Account`](https://docs.near.org/docs/concepts/account)

{% hint style="info" %}
[You can **join us on Discord**, if you have questions](https://discord.gg/fszyM7K)
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

------------------------

# Solution

```typescript
try {
    const { freeAccountId, publicKey, network }: AccountReq = req.body
    const config = configFromNetwork(network);
    const near = await connect(config);
    await near.createAccount(freeAccountId, publicKey);
    return res.status(200).json(freeAccountId)
}
```

**What happened in the code above?**
* First, we create an `near` object.
* Next, we call the `createAccount`, method passing the `name` and the `public key` 

------------------------

# Make sure it works

Once the code is complete and the file is saved, Next.js will rebuild the API route: 
* Click on **Create Account** 
You should see:

![](../../../.gitbook/assets/pathways/near/near-create-account.gif)

-----------------------------

# Next

When a new account is created on the `testnet` you'll end up with a free **airdrop** of 200 **NEAR** let's check this on the next step.
