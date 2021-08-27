Of course, everyone likes to eat pizza, but among all available pizzas you prefer the pizza you've made for yourself. So to simplify the transfer process we're going to make a transfer *from* our own account *to* our own account again.

To do so, you'll need to make an encrypted transaction - in the **Celo** world everything is done with privacy in mind! Let's take a look at how to do this.

----------------------------------

# The challenge

{% hint style="warning" %}
In `pages/api/celo/transfer.ts`, complete the code of the function. There is a lot to fill here, so be careful!
{% endhint %}

**Take a few minutes to figure this out.**

```tsx
//..

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

 //..
```

**What happened in the code above?**
* First, we create a secure connection using `SigningCosmWasmClient`, passing:

----------------------------------

# Make sure it works

Once you have the code above saved:
* Fill in the amount of **CELO** you want to send to your favorite pizza maker (and as you realize, it was yourself).
* Click on **Submit Transfer**.

![](../../../.gitbook/assets/pathways/celo/celo-transfer.png)

----------------------------------

# Next

Now that we have funded our account and made a transfer, let's move on to deploying some code (known as a "smart contract") to the **Secret** blockchain! Ready to take the plunge? Let's go... 
