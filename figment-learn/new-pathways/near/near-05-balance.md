After the creation of your account on the **NEAR** an automatic **airdrop** will fund it with 200 **NEAR**. Here we'are going to check the balance our account to check if everything if alright.

------------------------

# Challenge

{% hint style="tip" %}
In `pages/api/near/balance.ts`, complete the code of the function.
{% endhint %}

**Take a few minutes to figure this out**

```typescript
  try {
      const { network, accountId } = req.body;
      const config = configFromNetwork(network);       
      const client = await connect(config);
      const account = undefined;
      const balance = undefined;
      console.log(balance)
      return res.status(200).json(balance)
  }
```

**Need some help?** Check out those two links
* [The `Account` class](https://near.github.io/near-api-js/classes/account.account-1.html)  
* [Basic NEAR economic](https://docs.near.org/docs/concepts/gas)

{% hint style="info" %}
[You can **join us on Discord**, if you have questions](https://discord.gg/fszyM7K)
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

------------------------

# Solution

```typescript
  try {
      const { network, accountId } = req.body;
      const config = configFromNetwork(network);       
      const client = await connect(config);
      const account = await client.account(accountId);
      const balance = await account.getAccountBalance();
      console.log(balance)
      return res.status(200).json(balance)
  }
```

**What happened in the code above?**
* First, we create an `Account` object representing our account.
* Next, we call the `getAccountBalance` method` 

{% hint style="tip" %}
The amount retrieve by the `getAccountBalance` method is in **yoctoNEAR**, then to convert it to **NEAR** you'll need to divide it by 10**24 
{% endhint %}


------------------------

# Make sure it works

Once the code is complete and the file is saved, Next.js will rebuild the API route: 
* Click on **Check Balance** 
You should see:

![](../../../.gitbook/assets/pathways/near/near-balance.gif)

-----------------------------

# Next

200 **NEAR** available, hmmm ... I guess it's more than enough to do our first transfer. In the next step we're going to buy a pizza.
