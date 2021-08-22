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
    const programId = req.body.programId as PublicKey;
    const url = getSafeUrl();
    const connection = new Connection(url, "confirmed");
    const publicKey = new PublicKey(programId);
    const programInfo = await connection.getAccountInfo(publicKey);

    if (programInfo === null) {
        if (fs.existsSync(PROGRAM_SO_PATH)) {
            throw new Error(
              'Program needs to be deployed with `solana program deploy`',
            );
        } else {
          throw new Error('Program needs to be built and deployed');
        }
    } else if (!programInfo.executable) {
      throw new Error(`Program is not executable`);
    }

    res.status(200).json(true);
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
    const programId = req.body.programId as PublicKey;
    const url = getSafeUrl();
    const connection = new Connection(url, "confirmed");
    const publicKey = new PublicKey(programId);
    const programInfo = await connection.getAccountInfo(publicKey);

    if (programInfo === null) {
        if (fs.existsSync(PROGRAM_SO_PATH)) {
            throw new Error(
              'Program needs to be deployed with `solana program deploy`',
            );
        } else {
          throw new Error('Program needs to be built and deployed');
        }
    } else if (!programInfo.executable) {
      throw new Error(`Program is not executable`);
    }

    res.status(200).json(true);
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

## Next

Now that we have an account and that this account has been funded with **SOL** tokens, we are ready to make a transfer!
