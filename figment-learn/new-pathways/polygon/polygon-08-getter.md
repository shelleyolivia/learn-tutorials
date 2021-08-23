# 

At this point we have deployed a smart contract on the Polygon testnet we have set the value of its storage and we have a client side application that's ready to fetch some date from it. We just need to wire up that last part.

-------------------------------------

## The Challenge

{% hint style="warning" %}
**On the file `components/protocols/polygon/GeStorage.tsx` Implement the `getValue`**.    
{% endhint %}

**Take a few minutes to figure this out.**

```typescript
    const getValue = async () => {
        try {
            setFetchingGet(true)        
            const provider = new ethers.providers.Web3Provider(window.ethereum)
		    // try to figure out the expected parameters
            const contract = new ethers.Contract(undefined)
			// try to figure out the expected method 
            const storage = undefined
            setContractNumber(storage.toString())
            setFetchingGet(false)
        } catch(error) {
            console.log(error)
            setFetchingGet(false)
        }
    }
```

Need some help? Check out these two tips/links  
* [**Create a Contract using ethers**](https://docs.ethers.io/v5/api/contract/contract/#Contract--creating) 
	* You can **console.log `SimpleStorageJson`** to find the contract's `abi` and `address` (through the property `networks`)  
* [**How to call a contract's methods on a ethers Contract object**](https://docs.ethers.io/v5/api/contract/contract/#Contract-functionsCall)  
* To read from the blockchain you don't need to spend any tokens so you can just use a provider to create a Contract instance. But to write you will need to create and sign a transaction through Metamask. Use a `signer` to create the Contract object!

{% hint style="info" %}
[Still not sure how to do this? **Join us on Discord** and someone will help!](https://discord.gg/fszyM7K)
{% endhint %}

-------------------------------------

## The solution

```typescript
    const getValue = async () => {
        try {
            setFetchingGet(true)        
            const provider = new ethers.providers.Web3Provider(window.ethereum)
            const contract = new ethers.Contract(
                SimpleStorageJson.networks['80001'].address,
                SimpleStorageJson.abi,
                provider
            )
            const storage = await contract.get();
            setContractNumber(storage.toString())
            setFetchingGet(false)
        } catch(error) {
            console.log(error)
            setFetchingGet(false)
        }
    }
```

What's happening in the code above?

* We create Contract objects using
  * The contract json's address
  * The contract json's abi
  * A web3 provider
* We then call the functions `get()` on this Contract object to operate our decentralized code. The names of the functions must match the ones we defined in our Solidity smart contract, otherwise how would we know which code to execute? 

-----------------------------

## Make sure it works

Once the code above save you can click and this is what the UI should look like!

![](../../../.gitbook/assets/polygon-getter-v2.gif)

-------------------------------------

## Conclusion

Congratulations! We have gone from zero to **Polygon**, covering all the most fundamental concepts needed for developers to succeed in using **Polygon**. From connecting to the network to interacting with smart contracts, you have completed coding challenges and created a functional yet basic dApp.   

From here, there are many ways to increase your skills with web3 development. We recommend following some of the other Pathways on Learn, to learn about what makes the other netwrk protocols unique.

If you are an experienced developer, you are welcome to contribute tutorials for **Polygon** and earn some **MATIC** tokens! Check out our tutorial [contribution guidelines](../../../other/tutorial-guidelines/) to get started.

{% hint style="info" %}
[**Please tell us what you think about this tutorial!**](https://docs.google.com/forms/d/e/1FAIpQLSc9taxobvDSdXprMEFhCXgfcwS_oA-lu-nbQdYEW6c57Ie6qg/viewform?usp=sf_link)
{% endhint %}
