At the beginning of your this journey on Polygon's world, we have generated a mnemonic. Now we're going to learn how to restore an wallet from a mnemonic and how derive the address and the private key when the wallet have been restored

Ready ?

-------------------------------------

# The Challenge

{% hint style="tip" %}
**On the file `components/protocols/steps/polygon/Restore.tsx` Implement the `restore`**. First using `ethers` look for `Wallet`, next when the wallet have been regenerated try do deduce which property we're going to call in order to display the address and the private key, finally verify than the generated key match the existing one.   
{% endhint %}

**Take a few minutes to figure this out.**

```tsx
    const restore = () => {
        console.log(value)
        try {
            const wallet = undefined
            const selectedAddress = window.ethereum.selectedAddress;
            if (undefined === selectedAddress) {
                setAddress(undefined)
                setSecret(undefined)
            } else {
                setError('Unable to restore account')
            }
        } catch (error) {
            setAddress(null)
            setSecret(null)
            setError('Invalid mnemonic')
        }
    }
```

Need some help? Check out these two tips/links  
* [**Create Wallet using ethers**](https://docs.ethers.io/v5/api/signer/#Wallet) 
* [**Properties of a Wallet**](https://docs.ethers.io/v5/api/signer/#Wallet--properties) 

{% hint style="info" %}
You can [**join us on Discord**](https://discord.gg/fszyM7K), if you have questions or want help completing the tutorial.
{% endhint %}

Still not sure how to do this? No problem! The solution is below so you don't get stuck.

-------------------------------------

# The solution

```javascript
    const restore = () => {
        console.log(value)
        try {
            const wallet = ethers.Wallet.fromMnemonic(value.trim())
            const selectedAddress = window.ethereum.selectedAddress;
            if (wallet.address.toLocaleLowerCase() === selectedAddress) {
                setAddress(wallet.address.toLocaleLowerCase())
                setSecret(wallet.privateKey.toLocaleLowerCase())
            } else {
                setError('Unable to restore account')
            }
        } catch (error) {
            setAddress(null)
            setSecret(null)
            setError('Invalid mnemonic')
        }
    }
```

What happened in the code above? Let's take a closer look!
* First, we need to call `fromMnemonic` method of `Wallet` class.
* Next, we compare if the restored address match the existing one.
* Next, we store the address to display it in the UI.
* Last, we do the same for the private key.

-------------------------------------

# Make sure it works

When you have completed the code:
* Copy-Paste your **mnemonic**
* Click on **Restore Account**

![](../../../.gitbook/assets/polygon-restore-v2.gif)

-------------------------------------

# Next Steps

The ability to restore an account whitout the need to depend on tier-party is a great feature from web3. Now, we're ready to fund our account with some **MATIC** the native token of the Polygon blockchain.
