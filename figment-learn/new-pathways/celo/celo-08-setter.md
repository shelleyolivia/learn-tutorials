Our contract is on-chain, and we're going to learn how to modify the value stored in the state of the contract. 

{% hint style="working" %}
If you want to learn more about Secret smart contracts, follow the [**Developing your first secret contract**](https://learn.figment.io/tutorials/creating-a-secret-contract-from-scratch) tutorial.
{% endhint %}

----------------------------------

# The challenge

{% hint style="tip" %}
In `pages/api/celo/setter.ts`, complete the code of the default function. 
{% endhint %}

**Take a few minutes to figure this out.**

```tsx
//...
  try {
    const { secret, newMessage, contract } = req.body;
    const url = getSafeUrl();
    const kit = newKit(url);

    const account = kit.web3.eth.accounts.privateKeyToAccount(secret);
    kit.addAccount(account.privateKey);

    // Create a new contract instance with the HelloWorld contract info
    const instance = new kit.web3.eth.Contract(
        // @ts-ignore
        HelloWorld.abi, 
        contract
    )

    // Add your account to ContractKit to sign transactions
    // This account must have a CELO balance to pay tx fees, get some https://celo.org/build/faucet
    kit.connection.addAccount(account.privateKey);
    const txObject = await instance.methods.setName(newMessage);
    let tx = await kit.sendTransactionObject(txObject, { from: account.address });

    let receipt = await tx.waitReceipt();

    res.status(200).json(receipt.transactionHash);
  }
//...
```

**Need some help?** Check out this link!
* [**Contract example**](https://github.com/enigmampc/SecretJS-Templates/tree/master/5_contracts)  

{% hint style="info" %}
[You can **join us on Discord**, if you have questions](https://discord.gg/fszyM7K)
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

----------------------------------

# The solution

```tsx
//...

//...
```

**What happened in the code above?**
* We're calling the `execute` method of the `SigningCosmWasmClient`, passing to it:

----------------------------------

# Make sure it works

Once you have the code above saved, click the button and watch the magic happen:

![](../../../.gitbook/assets/pathways/celo/celo-setter.png)

----------------------------------

# Conclusion

In this tutorial, we learned quite a lot! We took a quick look at one of Ethereum’s most powerful development tool - Truffle. We used it to compile our smart contract. Then we deployed our smart contract with few lines of Javascript code and called two methods on that smart contract.

We have only covered a very small area of contract development. We invite you to keep experimenting on your own, and we will be providing more advanced Celo tutorials shortly to help you get to the next level.

If you had any difficulties following this tutorial or simply want to discuss Celo and DataHub tech with us you can[ join our community](https://discord.gg/Chhuv5zHy3) today!