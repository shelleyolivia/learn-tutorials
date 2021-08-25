In order to transfer some value to another account, we need to signed a transaction to the **network**. **NEAR** blockchain provide an abstract class to help us, **KeyStore**.

Here you are going to learn how to set your keystore in order to sign your transfer of token.

----------------------------------

# The challenge

{% hint style="warning" %}
In`pages/api/near/transfer.ts`, complete the code of the function. They're a lot to fill here.
{% endhint %}

**Take a few minutes to figure this out.**

```tsx
  const {
    txSender,
    txAmount,
    txReceiver,
    network,
    secret,
  } = req.body

  try {
    const config = configFromNetwork(network)

    // recreate the keypair from secret
    const keypair = undefined
  
    // Set the keystore with the expected method and args
    config.keyStore?.undefined

    // Here we convert the NEAR into yoctoNEAR using utilities from NEAR lib
    const yoctoAmount = parseNearAmount(txAmount) as string
    const amount = new BN(yoctoAmount) 

    // Fill the Gap: connect, create an Account Object and send some money
    const near = undefined
    const account = undefined
    const transaction = undefined

    return res.status(200).json(transaction.transaction.hash)
  } 
//...
```

**Need some help?** Check out those two links
* [Manage `KeyStore`](https://near.github.io/near-api-js/classes/key_stores_in_memory_key_store.inmemorykeystore.html)  
* [Check `sendMoney`](https://near.github.io/near-api-js/classes/account.account-1.html#sendmoney)

{% hint style="info" %}
[You can **join us on Discord**, if you have questions](https://discord.gg/fszyM7K)
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

----------------------------------

# The solution

```tsx
//...
  const {
    txSender,
    txAmount,
    txReceiver,
    network,
    secret,
  } = req.body

  try {
    const config = configFromNetwork(network)
    const keypair = KeyPair.fromString(secret)
    config.keyStore?.setKey(network, txSender, keypair) 

    const yoctoAmount = parseNearAmount(txAmount) as string
    const amount = new BN(yoctoAmount) 

    const near = await connect(config)
    const account = await near.account(txSender)
    const transaction = await account.sendMoney(txReceiver, amount)
    return res.status(200).json(transaction.transaction.hash)
  } 
//...
```

**What happened in the code above?**
* First, we need to *rehydrate* our `KeyPair` using our secret
* Next, we convert the **NEAR** into **yoctoNEAR** the smallest unit of money in **NEAR** blockchain
* Next, we create using `sendMoney` method we submit the signed transaction to the network 

----------------------------------

# Make sure it works

Once you have the code above saved:
* Fill the amount in **NEAR** 
* Click on **Submit Transfer**

![](../../../.gitbook/assets/near-transfer.gif)

----------------------------------

# Next

Now that we have an account, we can fund it so we can start playing around with tokens!
