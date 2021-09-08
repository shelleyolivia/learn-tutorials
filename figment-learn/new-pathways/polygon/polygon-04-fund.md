In this tutorial, we will acquire some free **MATIC** on the testnet so that we can deploy a smart contract and interact with it. Before dealing with actual value on the Polygon mainnet, it is wise to practice using the Polygon testnet (called Mumbai).

Click on the icon in the upper-right corner of the page to copy the hexadecimal address of the Metamask account to the clipboard:

![](../../../.gitbook/assets/click_to_copy.png)

Visit [https://faucet.matic.network/](https://faucet.matic.network/) and paste the address from your selected account in Metamask into the textinput. It is OK to leave the default options for **MATIC** tokens and the Mumbai network selected. Click the Submit button, then click again below to confirm the transaction.

{% hint style="warning" %}
Sometimes the transaction response on the faucet website returns an `[Object object]` - when this happens, simply refresh the page and try again.
{% endhint %}

Once this transaction is confirmed, you will have 1 **MATIC** on the Mumbai testnet!  

-------------------------------------

# The challenge

{% hint style="tip" %}
**Imagine this scenario:** You know you have a big balance. You need to show that balance so you can brag about it to all your awesome Web 3 developer friends! In `components/protocols/polygon/steps/Balance.tsx`, implement the`checkBalance` function :
{% endhint %}

**Take a few minutes to figure this out.**

```typescript
const checkBalance = async () => {
  setFetching(true)
  const provider = new ethers.providers.Web3Provider(window.ethereum);
  const selectedAddress = window.ethereum.selectedAddress;

  // TODO
  // Define those two variables
  const selectedAddressBalance = undefined
  const balanceToDisplay = undefined

  setBalance(balanceToDisplay);
  setFetching(false)
}
```

**Need some help?** Check out these two links  
Get an [**account balance**](https://docs.ethers.io/v5/api/providers/provider/#Provider-getBalance) using ethers  
Format the balance using [**ethers.utils.formatEther**](https://docs.ethers.io/v5/api/utils/display-logic/#unit-conversion)

{% hint style="info" %}
You can [**join us on Discord**](https://discord.gg/fszyM7K), if you have questions or want help completing the tutorial.
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

-------------------------------------

# The solution

```typescript
  const checkBalance = async () => {
    setFetching(true);
    const provider = new ethers.providers.Web3Provider(window.ethereum);
    const selectedAddress = window.ethereum.selectedAddress;
    const selectedAddressBalance = await provider.getBalance(selectedAddress);
    const balanceToDisplay = ethers.utils.formatEther(selectedAddressBalance.toString());
    setBalance(balanceToDisplay);
    setFetching(false);
  }
```

**What happened in the code above?**

* For the duration of this function, `fetching` will be true so that we can conditionally render our UI. 
* We await `provider.getBalance` because it returns a Promise. That Promise returns a BigNumber, which is a specific data type for handling numbers which fall [outside the range of safe values](https://docs.ethers.io/v5/api/utils/bignumber/#BigNumber--notes-safenumbers) in JavaScript. A BigNumber cannot be displayed in the same way as a normal number. We must therefore format the balance to transform it into a string for display, using `ethers.utils.formatEther`.
* Before we exit the function, set `fetching` to false, which effects the conditional rendering happening in the return function of `Balance.tsx`.

-------------------------------------

# Make sure it works

When you have completed the code, the **Check Balance** button will function. Click it to view the balance of the connected account:

![](../../../.gitbook/assets/pathways/polygon/polygon-fund.gif)

-------------------------------------

# Conclusion

Now that we have a funded Polygon account, we can use our MATIC tokens to deploy a smart contract.  
In the next tutorial, we will cover writing, testing and deploying the Solidity code using Truffle - a popular smart contract development suite.
