#

In the following tutorials, we're going to interact with the Solana blockchain (and in particular its Devnet network) using the `@solana/web3.js` library. It's a convenient way to interface with the RPC API when building a Javascript application (under the hood it implements Solana's RPC methods and exposes them as Javascript objects). We will explore it together as we add features to our app.

{% hint style="info" %}
[The documentation for `@solana/web3.js`](https://solana-labs.github.io/solana-web3.js/)
{% endhint %}

----------------------------------

## The challenge

{% hint style="warning" %}
In `pages/api/solana/connect.tsx`, implement `connect` by creating a `Connection` instance and getting the API's version. Render it on the webpage.
{% endhint %}

**Take a few minutes to figure this out**

```typescript
//...
  try {
    const url = getSafeUrl();
    const connection = undefined;
    const version = undefined;
    res.status(200).json(version?.["solana-core"]);
  }
 //...
```

**Need some help?** Check out those two links
* [Creating a `Connection` instance](https://solana-labs.github.io/solana-web3.js/classes/Connection.html#constructor)  
* [Getting the API's `version`](https://solana-labs.github.io/solana-web3.js/classes/Connection.html#getversion)

{% hint style="info" %}
[You can **join us on Discord**, if you have questions](https://discord.gg/fszyM7K)
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

----------------------------------

## The solution

```typescript
//...
  try {
    const url = getSafeUrl();
    const connection = new Connection(url, "confirmed");
    const version = await connection.getVersion();
    res.status(200).json(version?.["solana-core"]);
  } 
//...
```

**What happened in the code above?**
* We created a `connection` instance of the `Connection` class using the `new` constructor
* We then call `getVersion` on that `connection` instance. The docs state that `connection.getVersion()` returns a Promise so we chain `.then()` and a `.catch()` to respectively handle the case where the promise is fulfilled and rejected.
* If the promise is fulfilled, we get a `version` object back, that we can store in our functional component's local state, using the `setVersion` exposed by the React hook `useState`.

----------------------------------

## Visual's test

Once the code above save you can refresh the page to see the update.

![](../../../.gitbook/assets/solana-connect.gif)

----------------------------------

## Next

We're going to use this `connection` instance every time we need to connect to Solana. We're going to want to move some tokens around but first we need an account to hold tokens. That's what we'll do next!
