Unlike with most Web 3 protocols, NEAR uses human readable account IDs instead of a public key hash. You can link to a NEAR account as much `keypair` as you want. Here, we're going to Learn how tho check the availability of a near account. As you can expected, **figment.testnet** is already taken.

------------------------

# Challenge

{% hint style="tip" %}
In `pages/api/near/check-account.ts`, complete the code of the function.
{% endhint %}

**Take a few minutes to figure this out**

```typescript
  try {
    const { freeAccountId, network } = req.body
    const config = configFromNetwork(network);
    const near = await connect(config);
    // try to query the account info of the 
    const accountInfo = undefined
    try {
        undefined;
        return res.status(200).json(false)
    } catch (error) {
        return res.status(200).json(true)
    }
  }
```

**Need some help?** Check out those two links
* [The `Account` class](https://near.github.io/near-api-js/classes/account.account-1.html)  
* [`NEAR Account`](https://docs.near.org/docs/concepts/account)

{% hint style="info" %}
[You can **join us on Discord**, if you have questions](https://discord.gg/fszyM7K)
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

------------------------

# Solution

```typescript
  try {
    const { freeAccountId, network } = req.body
    const config = configFromNetwork(network);
    const near = await connect(config);
    const accountInfo = await near.account(freeAccountId);
    try {
        await accountInfo.state();
        return res.status(200).json(false)
    } catch (error) {
        return res.status(200).json(true)
    }
  }
```

**What happened in the code above?**
* First, we create an `Account` object.
* Next, we query the `state` of this object if it's failed then the account doesn't exist.

------------------------

# Make sure it works

Once the code is complete and the file is saved, Next.js will rebuild the API route: 
* Choose an account Id.
* Click on **Check it!** 
You should see:


![](../../../.gitbook/assets/pathways/near/near-check-account.gif)

-----------------------------

# Next

Great you have test the avaibility of the choosen account name, now it's time to create it. Let's do this in the next step.
