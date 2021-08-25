Our Contract is on chain, and we-re going to learn how to modify stored message of the contract. 

{% hint style="working" %}
If you want to learn more about **NEAR** Smart Contract take a look [here](https://learn.figment.io/tutorials/write-and-deploy-a-smart-contract-on-near)
{% endhint %}

----------------------------------

# The challenge

{% hint style="tip" %}
In`pages/api/near/setter.ts`, complete the code of the function. 
{% endhint %}

**Take a few minutes to figure this out.**

```tsx
//...
  try {
    const { network, accountId, secret, newMessage } = req.body;
    const config = configFromNetwork(network);
    const keypair = KeyPair.fromString(secret);
    config.keyStore?.setKey(network, accountId, keypair);        

    const near = await connect(config);
    const account = await near.account(accountId);
    // Look at functionCall and pass the rexpected agrs
    // ... fill here
    return res.status(200).json(response.transaction.hash)
  }
//...
```

**Need some help?** Check out this link!
* [Learn about `functionCall`](https://near.github.io/near-api-js/classes/account.account-1.html#functioncall)  

{% hint style="info" %}
[You can **join us on Discord**, if you have questions](https://discord.gg/fszyM7K)
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

----------------------------------

# The solution

```tsx
//...
  try {
    const { network, accountId, secret, newMessage } = req.body;
    const config = configFromNetwork(network);
    const keypair = KeyPair.fromString(secret);
    config.keyStore?.setKey(network, accountId, keypair);        

    const near = await connect(config);
    const account = await near.account(accountId);

    const optionsCall = {
        contractId: accountId,
        methodName: 'set_greeting',
        args: { message: newMessage }
    }
    const response = await account.functionCall(optionsCall);
    console.log(response)
    return res.status(200).json(response.transaction.hash)
  }
//...
```

**What happened in the code above?**
* We're calling `functionCall` method of our account, passing to it:
  * The `contractId` same as our account as the contract have been deployed to our account
  * The name of the method we want to call, here `set_greeting`
  * The name of the argument expected by `get_gretting` 
----------------------------------

# Make sure it works

Once you have the code above saved:
* Fill the amount in **NEAR** 
* Click on **Submit Transfer**

![](../../../.gitbook/assets/near-setter.gif)

----------------------------------

# Conclusion

Congratulations! You have successfully created, deployed, and interacted with your smart contract on the NEAR Testnet using DataHub.

While we have only covered a very small area of contract development you are more than welcome to continue exploration and experiments on your own, feel free to check out [**NEAR Developer**](https://examples.near.org/) site for more examples and tutorials.

The complete code for this tutorial can be found on [**Github**](https://github.com/figment-networks/tutorials/tree/main/near/5_contracts).

If you had any difficulties following this tutorial or simply want to discuss NEAR and DataHub tech with us you can join [our community](https://discord.gg/fszyM7K) today!
