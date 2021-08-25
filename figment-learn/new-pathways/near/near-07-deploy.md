We won't go throught the process of reviewing the Smart Contract code base, compiling it and/or testing it. We will be more focus on how one can deploy a Smart Contract using the `near-js-api`. Then to fill this requirement, we're going to use a pre-compiled Smart Contract, you can find it under `./contract/nearout/main.wasm`.

Our contract will be pretty basic. It's store a string and implement two function:
* `get_gretting` function which take an **accountId** and return the string stored for a specific account id
* `set_gretting` function which take an **accountId** and return the string stored for a specific account id

{% hint style="working" %}
If you want to learn more about **NEAR** Smart Contract take a look [here](https://learn.figment.io/tutorials/write-and-deploy-a-smart-contract-on-near)
{% endhint %}

----------------------------------

# The challenge

{% hint style="tip" %}
In`pages/api/near/deploy.ts`, complete the code of the function. And deploy your first Smart Contract on **NEAR** network.
{% endhint %}

**Take a few minutes to figure this out.**

```tsx
//...
  try {
    const { network, accountId, secret } = req.body;
    const config = configFromNetwork(network);
    const keypair = KeyPair.fromString(secret);

    // Again you will need to set your keystore
    config.keyStore?.undefined;

    const near = await connect(config);
    const account = await near.account(accountId);

    // Time to deploy the Smart Contract
    const response = undefined;
    return res.status(200).json(response.transaction.hash)
  }
//...
```

**Need some help?** Check out those two links
* [Learn more about `deployContract`](https://near.github.io/near-api-js/classes/account.account-1.html#deploycontract)  

{% hint style="info" %}
[You can **join us on Discord**, if you have questions](https://discord.gg/fszyM7K)
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

----------------------------------

# The solution

```tsx
//...
  try {
    const { network, accountId, secret } = req.body;
    const config = configFromNetwork(network);
    const keypair = KeyPair.fromString(secret);
    config.keyStore?.setKey(network, accountId, keypair);
    const near = await connect(config);
    const account = await near.account(accountId);
    const response = await account.deployContract(fs.readFileSync(WASM_PATH));
    return res.status(200).json(response.transaction.hash)
  }
//...
```

**What happened in the code above?**
* First, we need to *rehydrate* our `KeyPair` using our secret
* Next, we're calling the `deployContract` method will the `WASM_PATH` pointing to the the location of the Smart Contract.

----------------------------------

# Make sure it works

Once you have the code above saved:
* Fill the amount in **NEAR** 
* Click on **Submit Transfer**

![](../../../.gitbook/assets/near-transfer.gif)

----------------------------------

# Next

Now that we have an account, we can fund it so we can start playing around with tokens!
