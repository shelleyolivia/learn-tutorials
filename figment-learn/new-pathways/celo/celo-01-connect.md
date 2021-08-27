Celo is an open platform that makes financial tools accessible to anyone with a mobile phone. There are one tools that is especially useful when it comes to developing DApps on the Celo network. ContractKit - This is a javascript package that makes it easy to interact with the Celo network and the core Celo Smart Contracts as well as custom ones built by the community.

We are now all set up with our application and we can start writing some Javascript code. The first step here is to connect to the Celo **Alfajores** network using ContractKit.

------------------------

# Challenge

{% hint style="tip" %}
In `pages/api/celo/connect.ts`, complete the code of the function and try to establish your first connection to the celo network. 
{% endhint %}

**Take a few minutes to figure this out**

```typescript
//...
  try {
    const url = getSafeUrl();
    const kit = undefined;
    const version = undefined;
    res.status(200).json(version.slice(5, 11));
  } 
//...
```

**Need some help?** Check out these links
* [**To start working with contractkit you need a kit instance**](https://docs.celo.org/developer-guide/sdk-code-reference/summary-2/modules/_kit_#functions)  
* [**To access web3 using the kit**](https://docs.celo.org/developer-guide/contractkit/usage)  

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
    const version = await kit.web3.eth.getNodeInfo();
    res.status(200).json(version.slice(5, 11));
  } 
//...
```

**What happened in the code above?**
* First, we create a new `kit` instance.
* Next, using `web3.eth` we can acces a proxy of the [**web3.js - Ethereum Javascript API**](https://web3js.readthedocs.io/en/v1.4.0/)
* Finally, calling `getNodeInfo` method we can query the node to return the protocol version.

------------------------

# Make sure it works

Once the code is complete and the file has been saved, refresh the page to see it update & display the current version.

![](../../../.gitbook/assets/pathways/celo/celo-connect.png)


Congratulations, you have successfully made function that can connect to the Celo node!

-----------------------------

# Next

In this tutorial you’ve learned how to use the ContractKit package and DataHub to connect to the Celo node. You also had a chance to run one simple query to test that connection.