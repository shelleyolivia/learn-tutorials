# 

Last but not least, we'll need to modify the stored data into **greeter**. Doing so, will change the state of the blockchain, then we'll have to create a transaction for this. In the challenge below we're going to show you how to achieve this.

----------------------------------

## The challenge

{% hint style="warning" %}
In `pages/api/solana/balance.ts`, implement `balance`.
{% endhint %}

**Take a few minutes to figure this out**

```typescript 
//... 
  const instruction = new TransactionInstruction({ 
    keys: [{pubkey: greeterPublicKey as PublicKey, isSigner: false, isWritable: true}], 
    programId: programKey, 
    data: Buffer.alloc(0), // All instructions are hellos 
  }); 

  // this your turn to figure out 
  // how to create this transaction 
  const hash = await sendAndConfirmTransaction(undefined);

  res.status(200).json(undefined);
//...
```

**Need some help?** Here are a few hints
* [Read about TransactionInstruction](https://solana-labs.github.io/solana-web3.js/classes/Connection.html#getbalance)
* [Read about sendAndConfirmTransaction](https://solana-labs.github.io/solana-web3.js/classes/PublicKey.html#constructor)  

{% hint style="info" %}
[You can **join us on Discord**, if you have questions](https://discord.gg/fszyM7K)
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

----------------------------------

## The solution

```typescript
//...
  const instruction = new TransactionInstruction({
    keys: [{pubkey: greeterPublicKey, isSigner: false, isWritable: true}],
    programId: programKey,
    data: Buffer.alloc(0), // All instructions are hellos
  });

  const hash = await sendAndConfirmTransaction(
    connection,
    new Transaction().add(instruction),
    [payerKeypair]
  );

  res.status(200).json(hash);
//...
```

**What happened in the code above?**

* First, we create a instruction for a transaction:
  * With the greeter's keys, seeting **isWritable** flag to `true`
  * With the address of the program we want to call
  * With the data we want to pass to the call
* Finally we send and await that the transaction confirm; payer being the account created during firsts' steps.

----------------------------------

## Make sure it works

Once you have the code above saved:
* Click on **Send A Greeting** 
* Let's the magic happen

![](../../../.gitbook/assets/solana-set-v3.gif)

----------------------------------

## Conclusion

Congratulations on completing the Solana Pathway! We hope you had a fun time and learned a lot. Here are a few things you can check out to go further:

* [Keep exploring the Solana JS API](https://solana-labs.github.io/solana-web3.js/modules.html#sendandconfirmtransaction)
* [Solana's Hello World dApp](https://github.com/solana-labs/example-helloworld)
* [Read some programs written by Solana devs](https://github.com/solana-labs/solana-program-library/tree/master/examples)
* [Look at the Solana's Token Swap program](https://github.com/solana-labs/solana-program-library/tree/master/token-swap)

{% hint style="info" %}
If you had any difficulty with this tutorial or simply want to discuss Solana with other developers, [join our community](https://community.figment.io) today, or [join us on Discord](https://discord.gg/EBveT5xs9D)!
{% endhint %}
