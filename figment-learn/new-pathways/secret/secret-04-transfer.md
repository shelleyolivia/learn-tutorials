Of course, like everyone you like to eat pizza, but among pizza you do prefer the one made by yourself. Then, as you have already understand to simplify the process we're going to make a transfer from our account to our account again.

To realize such task, you'll need to make an encrypted transaction as in the **Secret** world everything is done with privacy in mind.

Let's take a look at it.

----------------------------------

# The challenge

{% hint style="warning" %}
In`pages/api/secret/transfer.ts`, complete the code of the function. There is a lot to fill here, so be careful!
{% endhint %}

**Take a few minutes to figure this out.**

```tsx
//..
  // 0. A very specific Secret features (allowing to made the transaction encrypted)
  const txEncryptionSeed = EnigmaUtils.GenerateNewSeed();

  // 1. The fees you'll will to pay to realize the transaction
  const fees = {
    send: {
      amount: [{ amount: '80000', denom: 'uscrt' }],
      gas: '80000',
    },
  };

  // 2. Initialise a secure's secret client TODO
  const client = new SigningCosmWasmClient(undefined);

  // 2. Send tokens
  const memo = 'sendTokens example'; // optional memo
  const sent = await client.sendTokens(undefined);
 
  // 3. Query the tx result
  const query = { id: sent.transactionHash };
  const transaction = await client.searchTx(query);
//..
```

**Need some help?** Check out these links
* [**Transaction example**](https://github.com/enigmampc/SecretJS-Templates/blob/master/4_transactions/send.js)  
* [**Documentation of `secrectjs`**](https://github.com/enigmampc/SecretNetwork/tree/master/cosmwasm-js/packages/sdk)  

{% hint style="info" %}
[**You can join us on Discord, if you have questions**](https://discord.gg/fszyM7K)
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

----------------------------------

# The solution

```tsx
//...
  // 1. Initialise a secure's secret client
  const client = new SigningCosmWasmClient(
    url,
    address,
    (signBytes) => signingPen.sign(signBytes),
    txEncryptionSeed, 
    fees
  );

  // 2. Send tokens
  const memo = 'sendTokens example'; // optional memo
  const sent = await client.sendTokens(address, [{ 
    amount: txAmount,
    denom: 'uscrt'
  }], memo);
 //..
```

**What happened in the code above?**
* First, we create a secure connection using `SigningCosmWasmClient`, passing:
  * `url` of the network
  * `address` of our wallet
  * A closure capturing our `sigingPen`, to sign transaction.
  * A seed for privacy
  * Some fees to reward the validator which will process our transaction
* Next, we send the expected amount of token using `sendTokens`, passing: 
  * The recipient `address`
  * The amount, and token
  * an optional `memo` working like a notice.

----------------------------------

# Make sure it works

Once you have the code above saved:
* Fill in the amount of **Secret** you want to send to favorite pizza maker (and as you reallize it was yourself).
* Click on **Submit Transfer**.

![](../../../.gitbook/assets/pathways/secret/secret-transfer.gif)

----------------------------------

# Next

Now that we have funded our account and made a transfer, let's move on to deploying some code (known as a "smart contract") to the **Secret** blockchain! Ready to take the plunge? Let's go... 
