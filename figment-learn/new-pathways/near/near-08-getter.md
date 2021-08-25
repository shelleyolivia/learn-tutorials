Our Contract is on chain, and we-re going to learn how to fecth the stored message on the contract. 

{% hint style="working" %}
If you want to learn more about **NEAR** Smart Contract take a look [here](https://learn.figment.io/tutorials/write-and-deploy-a-smart-contract-on-near)
{% endhint %}

----------------------------------

# The challenge

{% hint style="tip" %}
In`pages/api/near/getter.ts`, complete the code of the function. 
{% endhint %}

**Take a few minutes to figure this out.**

```tsx
//...
  try {
    const { network, accountId  } = req.body;
    const config = configFromNetwork(network);
    const near = await connect(config);
    const account = await near.account(accountId);
    // Using ViewFunction try to call the contract 
    const response = undefined;
    return res.status(200).json(response)
  }
//...
```

**Need some help?** Check out this link!
* [Learn about `viewFunction`](https://near.github.io/near-api-js/classes/account.account-1.html#viewfunction)  

{% hint style="info" %}
[You can **join us on Discord**, if you have questions](https://discord.gg/fszyM7K)
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

----------------------------------

# The solution

```tsx
//...
  try {
    const { network, accountId  } = req.body;
    const config = configFromNetwork(network);
    const near = await connect(config);
    const account = await near.account(accountId);
    const response = await account.viewFunction(
        accountId, 
        "get_greeting", 
        {account_id: accountId}
    );
    return res.status(200).json(response)
  }
//...
```

**What happened in the code above?**
* We're calling `viewFunction` method of our account, passing to it:
  * The `contractId` same as our account as the contract have been deployed to our account
  * The name of the method we want to call, here `get_gretting`
  * The name of the argument expected by `get_gretting` 

----------------------------------

# Make sure it works

Once you have the code above saved, Click on the button and let's the magic happen.

![](../../../.gitbook/assets/near-getter.gif)

----------------------------------

# Next

Now, time for the last challenge, time to modify the state of the Contract and then the state of the blockchain. Let's go.

