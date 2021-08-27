Please make sure that you completed the previous step.
It’s time to create your first Celo account on the Alfajores testnet. Without it, you won’t be able to fully take advantage of Celo's features.

------------------------

# Challenge

{% hint style="tip" %}
In `pages/api/celo/account.ts`, complete the code of the function to first create a **mnemonic**, then produce an **address** from the **public key** belonging to the **mnemonic**.
{% endhint %}

**Take a few minutes to figure this out**

```typescript
//...
try {
    const url = getSafeUrl();
    const kit = newKit(url);
    const account = undefined;
    const address = undefined;
    const secret = undefined;

    res.status(200).json({
        address,
        secret
    })
} 
//...
```

**Need some help?** Check out these links
* [**To start working with contractkit you need a kit instance**](https://docs.celo.org/developer-guide/sdk-code-reference/summary-2/modules/_kit_#functions)  
* [**Account documentation**](https://web3js.readthedocs.io/en/v1.4.0/web3-eth-accounts.html)  

{% hint style="info" %}
[**You can join us on Discord, if you have questions**](https://discord.gg/fszyM7K)
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

------------------------

# Solution

```typescript
//...
try {
    const url = getSafeUrl();
    const kit = newKit(url);
    const account = kit.web3.eth.accounts.create();
    const address = account.address;
    const secret = account.privateKey;

    res.status(200).json({
        address,
        secret
    })
} 
//...
```

**What happened in the code above?**
* First, we create a new `kit` instance.
* Next, using `web3.eth` we can acces a proxy of the [**web3.js - Ethereum Javascript API**](https://web3js.readthedocs.io/en/v1.4.0/)
* Next, calling `create` of ``account` module we can create a new account.
* Finaly to access: 
    * The address use the `address` property.
    * The private key use the `privateKey` property.

{% hint style="tip" %}
Do not forget to fund the newly created wallet using the [celo faucet](https://celo.org/developers/faucet) in order to activate it!
{% endhint %}

------------------------

# Make sure it works

Once the code is complete and the file is saved, Next.js will rebuild the API route. Now click on **Generate a Keypair** and you should see:

![](../../../.gitbook/assets/pathways/celo/celo-account.png)

-----------------------------

# Next

Now that we have a Celo account created and funded with testnet tokens, let’s move on to querying a Celo node to get the current balance!
