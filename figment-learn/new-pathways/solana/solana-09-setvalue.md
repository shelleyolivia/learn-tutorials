# 

We must check the account balance to make sure we have sufficient **SOL** to perform a transfer. The `getBalance()` function takes a `publicKey` as input and will return the balance associated with that `publicKey`, if there is any.

----------------------------------

## The challenge

{% hint style="warning" %}
In `pages/api/solana/balance.ts`, implement `balance`.
{% endhint %}

**Take a few minutes to figure this out**

```typescript
//...
  try {
    const { greeter, secret, programId } = req.body;
    const url = getSafeUrl();
    const connection = new Connection(url, "confirmed");

    const greeterPublicKey = new PublicKey(greeter);
    const programKey = new PublicKey(programId);

    const payerSecretKey = new Uint8Array(JSON.parse(secret));
    const payerKeypair = Keypair.fromSecretKey(payerSecretKey);

    const instruction = new TransactionInstruction({
      keys: [{pubkey: greeterPublicKey as PublicKey, isSigner: false, isWritable: true}],
      programId: programKey,
      data: Buffer.alloc(0), 
    });

    const hash = await sendAndConfirmTransaction(
      connection,
      new Transaction().add(instruction),
      [payerKeypair]
    );

    res.status(200).json(hash);
  } 
//...
```

**Need some help?** Here are a few hints
* [Read about getBalance](https://solana-labs.github.io/solana-web3.js/classes/Connection.html#getbalance)
* [Create a publicKey from a string](https://solana-labs.github.io/solana-web3.js/classes/PublicKey.html#constructor)  

{% hint style="info" %}
You can also [**join us on Discord**](https://discord.gg/fszyM7K) if you have questions.
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

----------------------------------

## The solution

```typescript
//...
  try {
    const { greeter, secret, programId } = req.body;
    const url = getSafeUrl();
    const connection = new Connection(url, "confirmed");

    const greeterPublicKey = new PublicKey(greeter);
    const programKey = new PublicKey(programId);

    const payerSecretKey = new Uint8Array(JSON.parse(secret));
    const payerKeypair = Keypair.fromSecretKey(payerSecretKey);

    const instruction = new TransactionInstruction({
      keys: [{pubkey: greeterPublicKey as PublicKey, isSigner: false, isWritable: true}],
      programId: programKey,
      data: Buffer.alloc(0), // All instructions are hellos
    });


    const hash = await sendAndConfirmTransaction(
      connection,
      new Transaction().add(instruction),
      [payerKeypair]
    );

    res.status(200).json(hash);
  } 
//...
```

**What happened in the code above?**

* We create a `PublicKey` using the string formated address
* We call `connection.getBalance` with that `publicKey`
* Be aware than the balance is on `LAMPORTS` (`console.log` is your friend ^^) 

----------------------------------

## Make sure it works

Enter the address just funded and click on **Check Balance**. You should see:

![](../../../.gitbook/assets/solana-balance.gif)

----------------------------------

## Conclusion

Congratulations on completing the Solana Pathway! We hope you had a fun time and learned a lot. Here are a few things you can check out to go further:

* Keep exploring the [Solana JS API](https://solana-labs.github.io/solana-web3.js/modules.html#sendandconfirmtransaction)
* Solana's [Hello World dApp](https://github.com/solana-labs/example-helloworld)
* Read some [programs written by Solana devs](https://github.com/solana-labs/solana-program-library/tree/master/examples)
* Look at the Solana's [Token Swap program](https://github.com/solana-labs/solana-program-library/tree/master/token-swap) (~ Uniswap clone!)

If you had any difficulty with this tutorial or simply want to discuss Solana with other developers, [join our community](https://community.figment.io) today, or [join us on Discord](https://discord.gg/EBveT5xs9D)!