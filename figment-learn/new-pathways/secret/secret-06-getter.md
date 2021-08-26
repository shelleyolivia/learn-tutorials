Our Contract is on-chain, and we're going to learn how to fetch the count stored on the contract. 

{% hint style="working" %}
If you want to learn more about Secret smart contracts, look at [**Developing your first secret contract**](https://learn.figment.io/tutorials/creating-a-secret-contract-from-scratch)
{% endhint %}


{% hint style="danger" %}
You can experience some issue with the availbility of the network [**To check the current status**](https://secretnodes.com/secret/chains/holodeck-2)
{% endhint %}

Before, focusing on the deployement instuctions let's take a look fees payement object:

```typescript
const customFees = {
  send: {
    amount: [{ amount: '80000', denom: 'uscrt' }],
    gas: '80000',
  },
}
```
* `customFees` object stored the predefined amount of fees to pay in order to **send** a query to the smart contract.  

----------------------------------

# The challenge

{% hint style="tip" %}
In`pages/api/secret/getter.ts`, complete the code of the default function. 
{% endhint %}

**Take a few minutes to figure this out.**

```tsx
//...
  // Get the stored value
  console.log('Querying contract for current count');
  let response = undefined.
  let count = response.count as number
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
  // Get the stored value
  console.log('Querying contract for current count');
  let response = await client.queryContractSmart(contract, { get_count: {} })
  let count = response.count as number
//...
```

**What happened in the code above?**
* We're calling the `queryContractSmart()` method of the client, passing to it:
  * The `contract`, which is the contract address. 
  * The `{ get_count: {} }` object which represent the name of the method we are calling and the parameters we're passing to it.

----------------------------------

# Make sure it works

Once you have the code above saved, click the button and watch the magic happen:

![](../../../.gitbook/assets/pathways/near/near-getter.png)

----------------------------------

# Next

Now, time for the last challenge! Time to modify the state of the contract and thus the state of the blockchain. Let's go!
