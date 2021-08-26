After the creation of your account on the **Secret** `holodeck-2` network, and funding it using the faucet. We're going to check the balance of our account to make sure everything went alright.

{% hint style="info" %}
The native token in **Secret** is **SCRT**
{% endhint %}

------------------------

# Challenge

{% hint style="tip" %}
In `pages/api/secret/balance.ts`, complete the code of the default function.
{% endhint %}

**Take a few minutes to figure this out**

```typescript
  try {
    const url = await getSafeUrl()
    const { address }= req.body
    const client = new CosmWasmClient(url)

    // Query the Account object 
    const account = undefined;
    // Return the balance
    const balance = undefined;

    res.status(200).json(balance)
  }
```

**Need some help?** Check out these links
* [**Query example**](https://github.com/enigmampc/SecretJS-Templates/blob/master/3_query_node/query.js)
* [**Documentation of `secrectjs`**](https://github.com/enigmampc/SecretNetwork/tree/master/cosmwasm-js/packages/sdk)  

{% hint style="info" %}
[**You can join us on Discord, if you have questions**](https://discord.gg/fszyM7K)
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

------------------------

# Solution

```typescript
  try {
    const url = await getSafeUrl()
    const { address }= req.body
    const client = new CosmWasmClient(url);

    const account = await client.getAccount(address);
    const balance = account?.balance[0].amount as string;

    res.status(200).json(balance)
  }
```

**What happened in the code above?**
* First, we return an `Account` from `getAccount` method.
* Next, we return 

{% hint style="info" %}
If you want to know why not inspecting the `Account` Object directly in the terminal using `console.log(account)`
{% endhint %}

{% hint style="tip" %}
The amount returned by is denominated in **μSCRT**, so to convert it to **SCRT** you'll need to divide it by 10**6 
{% endhint %}

------------------------

# Make sure it works

Once the code is complete and the file is saved, Next.js will rebuild the API route. Click on **Check Balance** and you should see the balance displayed on the page:

![](../../../.gitbook/assets/pathways/secret/secret-balance.gif)

-----------------------------

# Next

100 **SCRT** available, hmmm ... I guess it's more than enough to do our first transfer. In the next step, we're going to buy a pizza!
