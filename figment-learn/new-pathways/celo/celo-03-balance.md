Now that you have created an account on the **celo** `holodeck-2` network, and funded it using the faucet - We're going to check the balance of our account to make sure everything went alright.

{% hint style="info" %}
The native token on the **Celo Network** is **SCRT**
{% endhint %}

------------------------

# Challenge

{% hint style="tip" %}
In `pages/api/celo/balance.ts`, complete the code of the default function.
{% endhint %}

**Take a few minutes to figure this out**

```typescript
//...

//...
```

**Need some help?** Check out these links
* [**Query example**](https://github.com/enigmampc/SecretJS-Templates/blob/master/3_query_node/query.js)
* [**Check out the CosmWasmClient source to understand the `Account` interface**](https://github.com/enigmampc/SecretNetwork/blob/master/cosmwasm-js/packages/sdk/src/cosmwasmclient.ts) 
* [**Also, look at `getAccount()` and what it returns**](https://github.com/enigmampc/SecretNetwork/blob/7adccb9a09579a564fc90173cc9509d88c46d114/cosmwasm-js/packages/sdk/src/cosmwasmclient.ts#L231)

{% hint style="info" %}
[**You can join us on Discord, if you have questions**](https://discord.gg/fszyM7K)
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

------------------------

# Solution

```typescript
//...

//...
```

**What happened in the code above?**
* First, we return an instance of the `Account` class from the `getAccount()` method.
* Next, we check the balance by accessing the `amount` property of the `Account.balance[0]`. The array attached to `balance` here is because the TypeScript definitions specify a `balance` as being a `ReadOnlyArray<Coin>`. The zero-index refers to the SCRT Coin.
  * Take note of the use of the [optional chaining operator](https://www.codeisbae.com/typescript-optional-chaining-nullish-coalescing/): `?.` This effectively prevents passing an incorrect value back to the client-side, because if there is no balance property present the expression will not evaluate.

{% hint style="info" %}
If you want to see more info, why not inspect the `Account` Object directly in the terminal using `console.log(account)`?
{% endhint %}

{% hint style="tip" %}
The amount returned by is denominated in **μSCRT**, so to convert it to **SCRT** you'll need to divide it by 10**6 
{% endhint %}

------------------------

# Make sure it works

Once the code is complete and the file is saved, Next.js will rebuild the API route. Click on **Check Balance** and you should see the balance displayed on the page:

![](../../../.gitbook/assets/pathways/celo/celo-balance.png)

-----------------------------

# Next

100 **CELO** available, hmmm ... seems it's more than enough to do our first transfer. In the next step, we're going to buy a pizza which means making a transfer of tokens!
