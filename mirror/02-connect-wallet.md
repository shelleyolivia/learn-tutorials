# Step 2: Connecting a Wallet

The first step for any dApp is setting up a way to connect to a user's wallet. This allows the dApp to authenticate the user without using traditional passwords. More importantly, this allows the dApp to pull the user's onchain data so we can build user-specific dApp experiences.

{% sidenote title="Box 2.1: A note on crypto wallets" %}
Crypto wallets are the gateway through which users interact with blockchain protocols. The wallet locally stores a user's private keys encrypted by a password and allows the user to sign transactions that prove control of an account's keys in a trustless way. Developers can leverage this technology to authenticate users and build applications where users control their own data in a portable way.

In this case, we'll be using a JavaScript library called [ethers.js](https://docs.ethers.io/) to interact with a user's browser extension wallet like [MetaMask](https://metamask.io/). This will allow us to detect whether the wallet is installed, and access the user account's API for getting its data and leveraging its functionality.

# Implementation 🧩
The first step in performing any blockchain operation is typically instantiating a connection. This can be done by leveraging the **Providers** class in ethers.js. A Web 3 provider object includes functionality to read from a protocol. There are several ways to implement this, but we'll leverage a simple implementation with **Web3Provider**.

```javascript
const provider = new ethers.providers.Web3Provider(window.ethereum);
```

The `window.ethereum` object comes courtesy of MetaMask, so we can pass it in as an argument to instantiate our Web 3 provider object.

If you're familiar with React development, you can probably guess that we'll need a way to preserve application state after we connect to the wallet. That way we don't have to make the user re-connect their wallet again. For this we'll use a React's [Context](https://reactjs.org/docs/context.html).

Furthermore, we should implement flow control for the logic above and consider abstracting it into a hook. There are several ways to implement this so feel free to try different implementations to learn more.

In our case, we decided to write a custom hook in `context/web3Context.tsx` that returns a state provider (not to be confused with the Web 3 provider!) with a `connect` function. This function will be responsible for requesting the wallet accounts, getting the signer and setting our user's public address.

Once we have a Web 3 provider, we can leverage the send method to request the accounts associated with the wallet we want to connect. Then, we can instantiate a **Signer** object that will give us access to functions that allow us to change the state of the blockchain.

```javascript
await provider.send('eth_requestAccounts', []);
const signer = provider.getSigner();
const address = await signer.getAddress();
```

{% sidenote title="Box 2.2: Without centralized servers, what are we connecting to?" %}
Going deep into something like the Ethereum Virtual Machine is beyond the scope of this tutorial, but this question is critical if you want to better understand the backend you're building on. Instead of connecting to a specific server and reading from a database, we're reading from a distributed state machine. It's distributed because there lots of nodes all over the world holding the same state. It's a state machine because it's in one state at any given time. We interact with it and compete with others using it to update a slice of its state that will get recorded before the next cycle of state updates.

Putting these three lines together with our `connect` function, and understanding that our custom hook is already in use, our users should now be able to connect by clicking on the **Connect** button at the top right.

>>>> TBD Insert gif
![Figure 3: Forget logins, now our users can connect to our app with their wallets!](./assets/ladder.jpeg)

{% label %}
Figure 3: Forget logins, now our users can connect to our app with their wallets!

##### _Listing 2.1: Code for connecting a wallet_
```javascript
const connect = useCallback(async () => {
  if (provider) {
    // Request accounts, get signer and get signer's address
    // More information can be found: https://docs.ethers.io/v5/getting-started/#getting-started--connecting
    await provider.send('eth_requestAccounts', []);
    const signer = provider.getSigner();
    const address = await signer.getAddress();

    setAddress(address);
  } else {
    alert('Please install MetaMask at https://metamask.io');
  }
}, [provider]);
```

# Challenge 🏋️

Navigate to `context/web3Context.tsx` in your editor and follow the steps included as comments to finish writing the `Web3Provider` function. We include a description along with a link to the documentation you need to review in order to implement each line. The relevant code block is also included in [Listing 2.2](#listing-22-instructions-for-connecting-a-wallet) below.

##### _Listing 2.2: Instructions for connecting a wallet_
```javascript
const connect = useCallback(async () => {
  if (provider) {
    // Request accounts, get signer and get signer's address
    // More information can be found: https://docs.ethers.io/v5/getting-started/#getting-started--connecting

    const address = '';

    setAddress(address);
  } else {
    alert('Please install MetaMask at https://metamask.io');
  }
}, [provider]);
```
