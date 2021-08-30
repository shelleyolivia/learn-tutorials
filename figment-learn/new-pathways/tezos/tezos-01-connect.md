In this steps we will learn how to connect to a Tezos node via DataHub, using functions from the Taquito JavaScript library. In the previous step, we set up the project directory and installed the Taquito library into our project using the node package manager.

------------------------

# Challenge

{% hint style="tip" %}
In `pages/api/tezos/connect.ts`, complete the code of the function and try to establish your first connection to the tezos network. To verify the connection is ok, try to return the chainId.
{% endhint %}

```typescript
//...
  try {
    const url = getTezosUrl();
    const toolkit = undefined;
    const chainId = undefined;
    if (validateChain(chainId) != 3) {
      throw Error("invalid chain Id");
    }
    res.status(200).json(chainId);
  } 
//...
```

**Need some help?** Check out these links
* [**Class `TezosToolkit`**](https://tezostaquito.io/typedoc/classes/_taquito_taquito.tezostoolkit.html)
* [**Taquito**](https://tezostaquito.io/typedoc/modules.html)  

{% hint style="info" %}
[**You can join us on Discord, if you have questions**](https://discord.gg/fszyM7K)
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

------------------------

# Solution

```typescript
  try {
    const url = getTezosUrl();
    const toolkit = new TezosToolkit(url);
    const chainId = await toolkit.rpc.getChainId();
    if (validateChain(chainId) != 3) {
      throw Error("invalid chain Id");
    }
    res.status(200).json(chainId);
  } 
```

**What happened in the code above?**
* `getTezosUrl` is a helper function to generate a valid endpoint URL.
* The `TezosToolkit` instance manages the connection.
* Tezos does not expose a software version for nodes, so we will instead retrieve the Chain ID with `getChainId`.

------------------------

# Make sure it works

Once the code is complete and the file has been saved, refresh the page to see it update & display the current version.

![](../../../.gitbook/assets/pathways/tezos/tezos-connect.gif)

-----------------------------

# Next

Congratulations! We have connected to the Tezos blockchain and queried an account balance with a few lines of JavaScript code. We can now proceed to creating an account for use on Florence, the Tezos testnet.
