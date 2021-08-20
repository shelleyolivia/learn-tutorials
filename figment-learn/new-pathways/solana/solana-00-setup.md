# Setup the project

## Prerequisites

The following software is required to set up and complete the Solana Pathway

* Node.js v14 or higher installed : [https://nodejs.org/](https://nodejs.org/)
* yarn installed : [https://yarnpkg.com/getting-started/install](https://yarnpkg.com/getting-started/install)

Start by cloning the repository and install the npm dependencies :

```bash
git clone https://github.com/figment-networks/learn-web3-dapp.git
cd learn-web3-dapp
yarn
```

Create an `.env.local` file at the root of the directory. Copy and paste the contents of the existing `.env.example` into the new file and save it to disk (alternatively, you can rename `.env.example` ).

The value for `DATAHUB_SOLANA_API_KEY` can be found on the [DataHub Services Dashboard](https://datahub.figment.io/services/solana). Click on the Solana icon in the list of available protocols and then copy your key as shown below. You can now paste your personal API key into `.env.local` . This will authenticate you and enable you to make JSON-RPC requests to the Solana cluster via DataHub.

![](../../../.gitbook/assets/solana-setup-00.gif)

With the API key in place, save the `.env.local` file and start the React interface with :

```bash
yarn dev
```

Once the development server loads and compiles the application, your default browser should be redirected to `http://localhost:3000` :

{% hint style="warning" %}
If the local development server has started, but a new browser window or tab has not opened, you can still do so manually by pasting (or typing) `http://localhost:3000` into the address bar of your browser. Once the app is running and you can view it in your browser, click on  "View step instructions" to see the challenge for step one, Connect to the Solana Devnet! 
{% endhint %}

![](../../../.gitbook/assets/solana-setup-01.gif)

{% hint style="info" %}
[**Join us on Discord**](https://discord.gg/fszyM7K) if you encounter any issues with the tutorial or have any questions!
{% endhint %}

You can now move ahead to the next step by clicking on the "Next" button below on the right.
