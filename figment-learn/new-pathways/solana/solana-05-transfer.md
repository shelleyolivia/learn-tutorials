In order to transfer some value to another account, we need to create and send a signed transaction to the **cluster**. Once you understand how to do this, you will have a solid foundation on which to interact with other portions of the Solana API.

When a transaction is submitted to the **cluster**, the Solana runtime will execute a program to process each of the instructions contained in the transaction, in order, and atomically. This means that if any of the instructions fail for any reason, the entire transaction will revert. 

----------------------------------

# The challenge

{% hint style="warning" %}
In `pages/api/solana/transfer.ts` finish implementing the `transfer()` function.
{% endhint %}

**Take a few minutes to figure this out.**

```typescript
//..
  //... let's skip the beginning as it's should be familiar for you now.
  // The secret key is stored in our state as a stingified array
  const secretKey = Uint8Array.from(JSON.parse(secret as string));

  // Find the parameter to pass  
  const instructions = SystemProgram.transfer

  // How could you construct a signer array's
  const signers = 

  // Maybe adding someting to a Transaction could be interesting ?
  const transaction = new Transaction()

  const hash =// You should now what is expected here.
  res.status(200).json(hash);
//..
```

**Need some help?** Here are a few hints
* [Read about `sendAndConfirmTransaction`](https://solana-labs.github.io/solana-web3.js/modules.html#sendAndConfirmTransaction)  
* [Read about adding instructions to `Transaction`](https://solana-labs.github.io/solana-web3.js/classes/Transaction.html#add)  
* [Anatomy of a `Transaction`](https://docs.solana.com/developing/programming-model/transactions)

{% hint style="info" %}
You can also [**join us on Discord**](https://discord.gg/fszyM7K) if you have questions.
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

----------------------------------

# The solution

```typescript
//..
  //...
  //... let's skip the beginning as it's should be familiar for you now.
  const instructions = SystemProgram.transfer({
    fromPubkey,
    toPubkey,
    lamports,
  });
  
  const signers = [
    {
      publicKey: fromPubkey,
      secretKey
    }
  ];
  
  const transaction = new Transaction().add(instructions);
  
  const hash = await sendAndConfirmTransaction(
    connection,
    transaction,
    signers,
  )

  res.status(200).json(hash);
//..
```

**What happened in the code above:**

* We created a `signers` array with only one signer: the sender. It contains both the signer's `publicKey` and `secretKey`. 
* The `secretKey` is used to sign the transaction. 
* We create `instructions` for the transactions. From who, to who and what amount.
* We create a `Transaction` object and add the instructions
* Finaly, we send and await the confirmation of the signed transatcion using `sendAndConfirmTransaction`.

----------------------------------

# Make sure it works

Once you've filled in the form: 
* Press "Submit"
* Let's the magic happen

![](../../../.gitbook/assets/solana-transfer-v3.gif)

**About the explorer**, it's very good practice to look over all the fields one by one to familiarize yourself with the structure of a transaction. This page features the transaction result (`SUCCESS`), status (`FINALIZED`), the amount sent, the `from` and `to` addresses, the block that included this transaction, the fee that was paid, etc.

----------------------------------

# Next

Next, we will look at how to deploy a program written in the Rust language to the Solana cluster.
