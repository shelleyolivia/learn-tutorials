# Connect

## Requirements
Metamask


## Description
If you’ve been in the blockchain ecosystem you are probably familiar with those “Connect” buttons which allow you to connect your wallet like MetaMask to dApp that you currently are view. In this tutorial we will implement this functionality in order for you to see the different between this simple “authentication” method with authentication using Ceramic/IDX.

## Steps
In order to go through this tutorial you will need to have Metamask installed no your computer as an extension to your favourite browser. For more information on how to do that please go to: [MetaMask - A crypto wallet & gateway to blockchain apps](https://metamask.io/)

The Challenge
Go to Connect.tsx and add a way to open Metamask wallet:

```js
const getConnection = async () => {
  try {
    const addresses = await window.ethereum.enable();
    const address = addresses[0];

    setAddress(address);

    dispatch({
      type: 'SetAddress',
      address: address,
    });
  } catch (error) {
    setError(error);
  }
};
```

## What happened here
By using window.ethereum.enable(), Metamask is opened and you can connect your Ethereum account with a dApp. This way dApp can use your account to perform transactions and query the network


## Next

