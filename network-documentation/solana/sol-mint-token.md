# How to create a token on Solana

## Minting a fungible token on the Solana blockchain

## Introduction

In this tutorial we will be creating a **token** using the **Solana** blockchain. Tokens have many functionalties, such as a [social token](https://www.nasdaq.com/articles/social-tokens%3A-get-ready-for-the-next-massive-crypto-trend-2021-04-29), a [utility token](https://invao.org/token-classes-explained-coin-vs-utility-token-vs-security-token/), or a [coin](https://invao.org/token-classes-explained-coin-vs-utility-token-vs-security-token/).

Solana has a Token Program, written in Rust, that will allow us to create our own token. 

Note: If any command confuses you, it is normally possible to concatenate the command with **--help**.

* For example:
```bash
solana --help
```

## Prerequisites

* Basic familiarity with a [command-line interface](https://en.wikipedia.org/wiki/Command-line_interface)
* Basic familiarity with Git & Github
* Around $1 USD of [SOL](https://coinmarketcap.com/currencies/solana/) accessible

## Requirements

* [Rust](https://rustup.rs/) installed
* [Solana Tool Suite](https://docs.solana.com/cli/install-solana-cli-tools) installed
* [Github Account](https://github.com/) created
* [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) installed

### **1) Create an SOL wallet**

First, we are going to create an SOL wallet, through the command line, to handle our transactions.

* Install the **spl-token** CLI

```bash
cargo install spl-token-cli
```

* Create a new wallet, which will return the **pubkey**
```bash
solana-keygen new
```
	* You can enter a passphrase or just skip ahead and leave it empty. 
	* Write the seed phrase down somewhere safe. 

![walletCreationPhoto](../../../.gitbook/assets/createSOLWallet.png)

### **2) Fund the wallet**

We will now use the *pubkey* that was returned above to fund our wallet with SOL. The SOL is needed to pay for any changes we make to the blockchain, for example, the transaction that creates our token. 

* I will be sending 0.1 SOL to this wallet from a [Sollet](https://chrome.google.com/webstore/detail/sollet/fhmfendgdocmcbmfikdcogofphimnkno?hl=en) browser wallet, where I have SOL already. You can get some SOL through an exchange, such as [FTX](https://ftx.com/en).

Note: FTX and Solana are closely linked; FTX's sister company Alameda Research is an investor in Solana, and FTX also operates its own decentralized exchange called Serum on the Solana blockchain.

* Check the balance of the wallet

```bash
solana balance pubKey
```

### **3) Create the token**

* Create the token, which will return the **tokenAddress**

```bash
spl-token create-token
```

![tokenCreationPhoto](../../../.gitbook/assets/createSOLToken.png)

After running this command, congrats! You have created your own token on Solana. But we can't stop here; there is so much more we can do with this.

* Check the balance of the wallet 
```bash
solana balance pubKey
```

Your balance should be lower, indicating something happened. We paid for our token to be officially created 👍

### **4) Create an account to handle the tokens**

Now we need to create an [account](https://docs.solana.com/developing/programming-model/accounts) that can work with the tokens. 

* Create an account, which will return the **accountAddress**

```bash
spl-token create-account tokenAddress
```

* Check the balance of the wallet 

```bash
solana balance pubKey
```

### **5) Mint the tokens**

It's time to mint some tokens and send them around. For the mintAmount, I chose 1,000,000 for fun. This would be based off how big you expect your market cap to be.

* Mint some tokens

```bash
spl-token mint tokenAddress mintAmount recipientAddress
```

The recipientAddress would be the account you created to handle the tokens. 

Note: If you are lost on what any of these terms mean, refer to the note at the top of the page, which reminds it's always possible to use the **--help** command for more information.

* For example 

```bash
spl-token mint --help
```

![mintingTokensPhoto](../../../.gitbook/assets/mintSOLToken.png)

Go ahead and check the wallet balance after this transaction if you'd like. Once your tokens have finished minting, it's time to think: What's stopping me from minting an infinite amount?

### **6) Limit supply to prevent unlimited minting**

It is crucial to prevent infinite minting of our token, once you have minted as much as needed.

* Disable minting

```bash
spl-token authorize tokenAddress mint --disable
```

This will show the mint authority, which should match the pubkey of your wallet. The new mint authority should now be disabled. 

* Check token balance of existing accounts

```bash
spl-token accounts
```

You should now see your token with the balance matching how much you minted. Check the wallet balance too if you'd like, which will show the difference after the transaction. 

### **7) Transfer token to browser wallet**

This step is optional, but now we are going to send all of the token from our terminal-created wallet to our browser wallet. You must have some SOL in your browser wallet as well so it can automatically add the token. 

* Send token to browser wallet

```bash
spl-token transfer --fund-recipient tokenAddress transferAmount recipientAddress
```

This time, the recipientAddress should be your browser wallet's address. For example, I'm using my Sollet wallet (as mentioned in section 2 above), so I will need the SOL address of that wallet. 


After the transaction goes through, you should see your new token in your browser wallet! But, there's an obvious problem. It has no name...

### **8) Submit pull request to Solana to register token**

Our token is created and live on the blockchain, but Solana is not yet officially recognizing it. We need to get all the required information for the token ready for submission. 

* Clone the [Solana token list](https://github.com/solana-labs/token-list) 

```bash
git clone https://github.com/solana-labs/token-list
```

I cloned the repository to my desktop. 

* The logo should be added in the token-list folder under 

```bash
assets/mainnet/<mint address>/*.<png/svg>
```

The mint address is the tokenAddress we used before. Go ahead and name the file **logo.png** or **logo.svg**. It looks like they prefer be either of those file types. 

* Go to the token list file 

```bash
token-list/src/tokens/solana.tokenlist.json
```

* Add your token to the list like so:

![addingTokenToListPhoto](../../../.gitbook/assets/SOLTokenList.png)

Note: My example token will be a social token, so make sure to not copy that unless yours is one too.

Note: The link for the token image must be that exact format; just change the token address to your token address, and the logo filename to your logo filename and type. I named it **logo.png** to keep it simple. 

* Save the file 

* Fork the [token-list](https://github.com/solana-labs/token-list) repository to your Github

* While still inside the token-list folder in the command line, set the url of your locally cloned token-list folder to the forked repository, for example: 

```bash
git remote set-url origin https://github.com/jacobmakarsky/token-list
```

* Add all the files, commit and push the changes

You should now see the changes in your forked repository. 

* Go to the [pull requests page](https://github.com/solana-labs/token-list/pulls)

* Select the **New pull request** button

* Select the **compare across forks** option

* Select your forked repository from the **head repository** dropdown list

You should see 2 changed files; your token changes and the logo image. 

* Ensure these details are correct, for example, the folder's name holding the logo image should exactly match your token address in the token list. 

![tokenPullRequest](../../../.gitbook/assets/SOLPullRequest.png)

We are now ready to create the pull request! 

* Go ahead and click the green **Create pull request** button.

* Once finished adding a title and filling out the details, press the green **Create pull request** button again. 

Congratulations! Your token is now on the way to being official. Only if we could actually trade the token though...

### **9) BONUS: Add a market for your token on Serum**

You've made it! Your token is live and has a name. It can now be sent around and used for whatever its utility may be. But, there is no market created for it to be traded. 

Note: It costs roughly **10-15 SOL** to create a market, which right now is around **$350 USD**. 

* Head over to [Serum](https://dex.projectserum.com/#/list-new-market) and connect your chosen wallet in the top right. 

* Fill out the form

The **Base Token Mint Address** will be your tokenAddress, and the **Quote Token Mint Address** will be what token you want your token paired to. I would use USDT for my example. 

Once submitted, you should have a live market that can trade the tokens. I did not do this for my social token as I do not want it to be traded. 

## Conclusion

Woohoo! If you made it through all the steps, you have succesfully created your own token on the Solana blockchain. If you completed the bonus, you now have a tradeable token 👍


## What's next?

We have built our own token on the Solana blockchain. Feel free to continue experimenting by attempting to mint more of the token, sending the token around to other wallets, creating a market for it, making a [liquidity pool](https://raydium.io/liquidity/) etc. Check back in the future for a tutorial on how to make a [non-fungible token](https://www.blockchain-council.org/blockchain/a-quick-guide-to-fungible-vs-non-fungible-tokens/#:~:text=Non%2DFungible%20are%20Non%2DInterchangeable,token%20of%20the%20same%20type.) on Solana as well. 

## About the Author

This tutorial was created by Jacob Makarsky. You can find him on [Github](https://github.com/jacobmakarsky) or the [Figment Forum](https://community.figment.io/u/jacobmakarsky/summary) for any further help.

If you had any difficulties following this tutorial or simply want to discuss Solana's tech with us you can [join our community today](https://discord.gg/fszyM7K)!


## References

1) [Solana Token Program](https://spl.solana.com/token)
2) [Loop Creative Andy](https://www.youtube.com/watch?v=1cn-HnG_yns)











