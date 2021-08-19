# 3. Fund the account

## Devnet, Testnet, Mainnet

With some protocols, different networks (testnet, mainnet, etc) have different tokens names. For example with Polkadot, the mainnet token is DOT and the testnet token is WND. But in the Solana world, the token is always called *SOL*, no matter what network (or *cluster*) you are on. But don't get too excited: the tokens you get for free on the devnet cannot be used on Solana's mainnet. Nice try though

Speaking of clusters, make sure you always know which one you're on. In your code, we already saw this when we created a `Connection` object. We passed the URL of a node and that node is connected to one of the clusters. When you're looking at the [Solana Explorer](https://explorer.solana.com/?cluster=devnet) you can select which cluster you want to look at by selecting it at the top right of the page:

![](https://github.com/figment-networks/datahub-learn/upload/new-pathways/.gitbook/assets//solana-fund-00.png)

## Airdropping

To fund an account, we will do what is called an *airdrop* some tokens will magically fall from the sky onto our wallets! This will provide us with some SOL so that we can test making transfers as well as view the transaction details on a block explorer.

To do this we will make use of the JS API's `requestAirdrop()` method. It takes an address and an amount designated in... *lamport*.

> 1 SOL is equal to 1,000,000,000 lamports.The name of lamports is in honour of Solana's biggest technical influence, [Leslie Lamport](https://en.wikipedia.org/wiki/Leslie_Lamport).

## The challenge

> In `pages/api/solana/fund.ts`, implement `fund()`. Convert the text input to an address and use `requestAirdrop`to get 1 SOL.

**Need some help?** Here are a few hints.
* [Create a publicKey from a string](https://solana-labs.github.io/solana-web3.js/classes/PublicKey.html#constructor)  
* [`requestAirdrop()`documentation](https://solana-labs.github.io/solana-web3.js/classes/Connection.html#requestairdrop)

Take a few minutes to figure this out.

```typescript
//..
    // Create a PublicKey address from string formated address
    // Call requestAirdrop
    // On success, retrieve the transaction hash
    const { address } = req.body.address as PublicKey;
    const url = getSafeUrl();
    const connection = new Connection(url)
    const address = undefined  
    const hash = undefined
    await undefined
    res.status(200).json(hash)
  
//..
}
```

> You can also [**join us on Discord**](https://discord.gg/fszyM7K) if you have questions.

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

## The solution

```typescript
//..
    const { address } = req.body.address as PublicKey;
    const url = getSafeUrl();
    const connection = new Connection(url)
    const address = new PublicKey(req.body.address as PublicKey)  
    const hash = await connection.requestAirdrop(address, LAMPORTS_PER_SOL)
    await connection.confirmTransaction(hash);
    res.status(200).json(hash)
//..
}
```

**What happened in the code above?**

* We created a `PublicKey` from the string formated address
* We passed it to `requestAirdrop` together with a constant wich repesent one `SOL`
* We verify than the transaction is confirmed
* Then we return the hash of the transaction for the UI.

Once you have the code above saved:
* Copy and paste the your genrated address in the text input.   
* Click on **Fund this Address** 

And the magic happen

![](https://github.com/figment-networks/datahub-learn/blob/master/.gitbook/assets/solana-fund.png)

## Next

Before we make our first transfer, let's first check that that the account is correctly funded.
