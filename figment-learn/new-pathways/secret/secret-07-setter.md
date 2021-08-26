Our contract is on-chain, and we're going to learn how to modify the message stored in the state of the contract. 

{% hint style="working" %}
If you want to learn more about Secret smart contracts, look at [**Developing your first secret contract**](https://learn.figment.io/tutorials/creating-a-secret-contract-from-scratch)
{% endhint %}

{% hint style="danger" %}
You can experience some issue with the availbility of the network. [**To check the current status**](https://secretnodes.com/secret/chains/holodeck-2)
{% endhint %}

Before, focusing on the deployement instuctions let's take a look fees payement object:

```typescript
  const customFees = {
    exec: {
      amount: [{ amount: '500000', denom: 'uscrt' }],
      gas: '500000',
    }
  };
```
* `customFees` object stored the predefined amount of fees to pay in order to **exec** a write-access method of the smart contract.  

----------------------------------

# The challenge

{% hint style="tip" %}
In`pages/api/secret/setter.ts`, complete the code of the default function. 
{% endhint %}

**Take a few minutes to figure this out.**

```tsx
//...
  // Increment the counter
  const handleMsg = { increment: {} };
  const response = undefined;
//...
```

**Need some help?** Check out this link!
* [**Contract example**](https://github.com/enigmampc/SecretJS-Templates/tree/master/5_contracts)  
* [**Documentation of `secrectjs`**](https://github.com/enigmampc/SecretNetwork/tree/master/cosmwasm-js/packages/sdk)  

{% hint style="info" %}
[You can **join us on Discord**, if you have questions](https://discord.gg/fszyM7K)
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

----------------------------------

# The solution

```tsx
//...
  // Increment the counter
  const handleMsg = { increment: {} };
  const response = await client.execute(contract, handleMsg);
//...
```

**What happened in the code above?**
* We're calling the `queryContractSmart()` method of the client, passing to it:
  * The `contract`, which is the contract address. 
  * The `{ increment: {} }` object which represent the name of the method we are calling and the parameters we're passing to it.

----------------------------------

# Make sure it works

Once you have the code above saved, click the button and watch the magic happen:

![](../../../.gitbook/assets/pathways/near/near-setter.png)

----------------------------------

# Conclusion

Congratulations! You have successfully created, deployed, and interacted with your smart contract on the Secret Network Testnet using DataHub.

While we have only covered a very small area of contract development you are more than welcome to continue exploration and experiments on your own, feel free to check out the [**Secret Network Developers**](https://scrt.network/developers) site for more examples and tutorials.

If you had any difficulties following this tutorial or simply want to discuss Secret Network and DataHub tech with us you can join [**our community**](https://discord.gg/fszyM7K) today!

We also invite you to join the Secret community on their [Discord](http://chat.scrt.network) channel and on the [Secret Forum](http://forum.scrt.network) to go deeper into Secret development.  
