# 

Transfering some token is one of the major feature of web3. In this challenge, we're going to learn how to transfer a know amount of **MATIC** to a choosen recipient. Each time, a transfert occurs, we'are going to re-query the new balance of our account.

-------------------------------------

## The challenge

{% hint style="warning" %}
**Imagine this scenario:** You know you have a big balance and you want to eat some pizza. Then, you need to transfer **0.1** MATIC to buy one! In `components/protocols/polygon/steps/Transfer.tsx`, implement the `transfer` function :
{% endhint %}

**Take a few minutes to figure this out.**

```tsx
    const transfer = async () => {
        setFetching(true)
        try {
            const provider = new ethers.providers.Web3Provider(window.ethereum)
            const send_account = provider.getSigner().getAddress()
    
            const currentGasPrice = await provider.getGasPrice();
            const gas_price = ethers.utils.hexlify(parseInt(currentGasPrice.toString()));
    
            const transaction = {
                from: send_account,
                to: recipient,
                value: ethers.utils.parseEther('0.1'),
                nonce: provider.getTransactionCount(send_account, "latest"),
                gasLimit: ethers.utils.hexlify(100000),
                gasPrice: gas_price 
            }
            const hash = await provider.getSigner().sendTransaction(transaction)
            const receipt = await hash.wait()
            setHash(receipt.transactionHash)
            setFetching(false)
        } catch (error) {
            setError(error)
            setFetching(false)
        }
	}
```

**Need some help?** Check out these two links  
**→ Get an** [**account balance**](https://docs.ethers.io/v5/api/providers/provider/#Provider-getBalance) **using ethers  
**→ Format the balance using** [**ethers.utils.formatEther**](https://docs.ethers.io/v5/api/utils/display-logic/#unit-conversion)

{% hint style="info" %}
[Still not sure how to do this? **Join us on Discord** and someone will help!](https://discord.gg/fszyM7K)
{% endhint %}

-------------------------------------

## The solution

```javascript
    const transfer = async () => {
        setFetching(true)
        try {
            const provider = new ethers.providers.Web3Provider(window.ethereum)
            const send_account = provider.getSigner().getAddress()
    
            const currentGasPrice = await provider.getGasPrice();
            const gas_price = ethers.utils.hexlify(parseInt(currentGasPrice.toString()));
    
            const transaction = {
                from: send_account,
                to: recipient,
                value: ethers.utils.parseEther('0.1'),
                nonce: provider.getTransactionCount(send_account, "latest"),
                gasLimit: ethers.utils.hexlify(100000),
                gasPrice: gas_price 
            }
            const hash = await provider.getSigner().sendTransaction(transaction)
            const receipt = await hash.wait()
            setHash(receipt.transactionHash)
            setFetching(false)
        } catch (error) {
            setError(error)
            setFetching(false)
        }
	}
```

What happened in the code above? Let's take a closer look!

* For the duration of this function, `fetching` will be true so that we can conditionally render our UI. 
* We await `provider.getBalance()` because it returns a Promise. That Promise returns a BigNumber, which is a specific data type for handling numbers which fall [outside the range of safe values](https://docs.ethers.io/v5/api/utils/bignumber/#BigNumber--notes-safenumbers) in JavaScript. A BigNumber cannot be displayed in the same way as a normal number. We must therefore format the balance to transform it into a string for display, using `ethers.utils.formatEther()`.
* Before we exit the function, set `fetching` to false, which effects the conditional rendering happening in the return function of `3_Balance.tsx`.

-------------------------------------

## Make sure it works

When you have completed the code, the 'Check Balance' button will function. Click it to view the balance of the connected account:

![](../../../.gitbook/assets/matic_balance.png)

-------------------------------------

## Next Steps

Now that we have a funded Polygon account, we can use our MATIC tokens to deploy a smart contract.  
In the next tutorial, we will cover writing, testing and deploying the Solidity code using Truffle, a smart contract development suite.
