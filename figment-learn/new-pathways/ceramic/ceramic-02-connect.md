# Metamask

Metamask is a crypto wallet which is available as browser extension. In order to go through this tutorial you will need to have Metamask installed no your computer as an extension to your favourite browser. For more information on how to do that please go to: [MetaMask - A crypto wallet & gateway to blockchain apps](https://metamask.io/).
We will be using Ethereum as a provider for Metamask.

# Challenge

If you’ve been in the blockchain ecosystem you are probably familiar with those “Connect” buttons which allow you to connect your wallet like MetaMask to dApp that you currently are view. In this tutorial we will implement this functionality in order for you to see the different between this simple “authentication” method with authentication using Ceramic/IDX.

{% hint style="tip" %}
**Imagine this scenario:** You're a fresh Web3 developer who just landed a sweet role at a promising new startup, eager to show off your skills. You've been asked to show users of our dApp which network they are connected to (to avoid any confusion) and store the account currently selected address in Metamask (so that we can reference it later). In **`components/protocols/ceramic/components/steps/Connect.tsx`**, implement the`checkConnection` function.
{% endhint %}

**Take a few minutes to figure this out.**

```typescript
const checkConnection = async () => {
  try {
    const provider = await detectEthereumProvider();

    if (provider) {
      // Connect to Ethereum using Web3Provider and Metamask
      // Define address and network
      const addresses = undefined;
      const address = undefined;

      setAddress(address);
    } else {
      alert('Please install Metamask at https://metamask.io');
    }
  } catch (error) {
    alert('Something went wrong');
  }
};
```

{% hint style="info" %}
You can [**join us on Discord**](https://discord.gg/fszyM7K), if you have questions or want help completing the tutorial.
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

----------------------------------

# Solution

```typescript
// solution
const checkConnection = async () => {
  try {
    const provider = await detectEthereumProvider();

    if (provider) {
      // Connect to Polygon using Web3Provider and Metamask
      // Define address and network
      const addresses = await provider.request({method: 'eth_requestAccounts'});
      const address = addresses[0];

      setAddress(address);
    } else {
      alert('Please install Metamask at https://metamask.io');
    }
  } catch (error) {
    alert('Something went wrong');
  }
};
//...
```

**What happened in the code above?**

* By using `window.ethereum.enable()`, Metamask is opened and you can connect your Ethereum account with a dApp. This way dApp can use your account to perform transactions and query the network.

-------------------------------------

# Conclusion

Now that we performed basic "authentication" of your wallet, we can move on and implement decentralized authentication with Ceramic/IDX.  

