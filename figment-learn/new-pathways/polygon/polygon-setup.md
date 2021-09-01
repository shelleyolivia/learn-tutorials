# Requirements

* [Node.js](https://nodejs.org) v14+ installed (we recommend using [nvm](https://github.com/nvm-sh/nvm))
* [yarn](https://yarnpkg.com/) 
* [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) installed
* [Metamask](https://metamask.io/) browser extension installed. 

Start by [cloning](https://git-scm.com/docs/git-clone) the repository and installing the [dependencies](https://classic.yarnpkg.com/en/docs/managing-dependencies/) with `yarn` :

```text
git clone https://github.com/figment-networks/learn-web3-dapp.git
cd learn-web3-dapp
yarn
```

# Safety disclaimers

{% hint style="info" %}
If you **ALREADY** have Metamask installed and are using it for a hot wallet, we _**strongly recommend**_ creating an entirely new wallet in Metamask for the purposes of these tutorials. Figment Learn wants nothing to do with your personal keys. We do not want any accidents involving anybody's cryptocurrency! Again, you _must not continue_ until you take care of this.  
{% endhint %}

{% hint style="danger" %}
If you **DO NOT** already have Metamask installed, welcome to the wonderful world of web3!   
The first piece of advice we will give you is to make _absolutely sure that you write down the_ [**Secret Recovery Phrase**](https://community.metamask.io/t/what-is-a-secret-recovery-phrase-and-how-to-keep-your-crypto-wallet-secure/3440) that is generated during Metamask's initial setup. Do not store it in a digital format. Do not share it with anybody. Please do keep it accessible to yourself, because we will be using it during the pathway. 
{% endhint %}

-------------------------------------

# Add the Mumbai testnet to Metamask

The first task is to connect to the Polygon Mumbai testnet by adding it to the list of RPC endpoints in Metamask. Click on the Fox head icon in your web browser to open the popup, and then follow this workflow to complete the process :

* Click on the current network at the top of the Metamask popup (by default is says "Ethereum Mainnet")
* Scroll down and click on "Custom RPC"
* Fill in the form:
  * Network Name: `Polygon Mumbai`
  * New RPC URL: `https://rpc-mumbai.maticvigil.com/`
  * Chain ID: `80001`
  * Currency Symbol: `MATIC`
  * Block Explorer URL : `https://mumbai.polygonscan.com`
* Double check the information, then click on the Save button.

![](../../../.gitbook/assets/add_mumbai.png)

We use the testnet for development before moving into production on the main network or "mainnet".

-------------------------------------

# Set your API Key

The value for `DATAHUB_POLYGON_API_KEY` can be found on the [**DataHub Services Dashboard**](https://datahub.figment.io/services/Polygon). Click on the **Polygon** icon in the list of available protocols and then copy your key as shown below. You can now paste your personal API key into `.env.local`. This will authenticate you and enable you to make JSON-RPC requests to Polygon via DataHub.

![](../../../.gitbook/assets/pathways/polygon/polygon-setup.gif)

{% hint style="info" %}
[**Join us on Discord**](https://discord.gg/fszyM7K), if you encounter any issues with the tutorial or have any questions!**
{% endhint %}


# Start the server

With the API key in place, save the `.env.local` file and start the Next.js development server with:

```text
yarn dev
```

Once the development server loads and compiles the application, open your default browser and navigate to `http://localhost:3000`:

![](../../../.gitbook/assets/pathway-home.gif)

{% hint style="warning" %}
Did you know you can change the port? By default Next.js uses port 3000, but if you have another service already running on that port, use the `--port` flag, like `yarn dev --port 1122`.
{% endhint %}

-------------------------------------

# Next

You can now move ahead to the next step by clicking on the "Next" button below on the right. There are also links to the instructions for each step on the UI.
