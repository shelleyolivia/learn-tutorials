We won't go through the process of reviewing the smart contract code base, compiling it or testing it. We will focus instead on how one can deploy a smart contract using the `secretjs` library. To do this, we're going to use a pre-compiled smart contract, you can find it under `./contract/secret/contract.wasm`.

Our contract implements a simple counter. The contract is created with a parameter for the initial count and allows subsequent incrementing:
* The `get_count` function returns the value of the counter stored on the contract.
* The `increment` function increases the value of the counter stored on the contract by 1.

{% hint style="working" %}
If you want to learn more about Secret smart contracts, follow the [**Developing your first secret contract**](https://learn.figment.io/tutorials/creating-a-secret-contract-from-scratch) tutorial.
{% endhint %}


----------------------------------

# The challenge

{% hint style="tip" %}
In `pages/api/celo/deploy.ts`, complete the code of the default function. Upload your first smart contract on the **Secret** network.
{% endhint %}

**Take a few minutes to figure this out.**

```tsx
//...

//...
```

**Need some help?**
* [**Contract example**](https://github.com/enigmampc/SecretJS-Templates/tree/master/5_contracts)  

{% hint style="info" %}
[You can **join us on Discord**, if you have questions](https://discord.gg/fszyM7K)
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

----------------------------------

# The solution

```tsx
//...

//...
```

**What happened in the code above?**
* First, we upload the contract using `upload` method of the `SigningCosmWasmClient`.

----------------------------------

# Make sure it works

Once you have the code above saved, click on **Deploy Contract**

![](../../../.gitbook/assets/pathways/celo/celo-swap.png)

----------------------------------

# Next

Now that we have deployed a smart contract, let's interact with it! In the following tutorials, we will look at how to use both view and change functions.
