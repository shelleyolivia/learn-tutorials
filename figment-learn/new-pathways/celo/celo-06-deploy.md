

----------------------------------

# The challenge

{% hint style="tip" %}
In `pages/api/celo/deploy.ts`, complete the code of the default function. Upload your first smart contract on the **Celo** network.
{% endhint %}

**Take a few minutes to figure this out.**

```tsx
//...
  try {
    const { secret } = req.body
    const url = getSafeUrl();
    const kit = newKit(url);

    const account = kit.web3.eth.accounts.privateKeyToAccount(secret)
    kit.addAccount(account.privateKey);

    let tx = await kit.sendTransaction({
        from: account.address,
        data: HelloWorld.bytecode
    })

    const receipt = await tx.waitReceipt()
    console.log(receipt)

    res.status(200).json({
        address: receipt?.contractAddress as string,
        hash: receipt.transactionHash
    })
  }
//...
```

**Need some help?**
* [**Contract example**](https://github.com/enigmampc/SecretJS-Templates/tree/master/5_contracts)  
* [**The `upload` function**](https://github.com/enigmampc/SecretNetwork/blob/7adccb9a09579a564fc90173cc9509d88c46d114/cosmwasm-js/packages/sdk/src/signingcosmwasmclient.ts#L208)  

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
* First, we upload the contract using `upload` method of the `SigningCosmWasmClient`.

----------------------------------

# Make sure it works

Once you have the code above saved, click on **Deploy Contract**

![](../../../.gitbook/assets/pathways/secret/secret-deploy.png)

----------------------------------

# Next

Now that we have deployed a smart contract, let's interact with it! In the following tutorials, we will look at how to use both view and change functions.
