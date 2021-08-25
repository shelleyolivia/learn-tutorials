# 

The following software is required to set up and complete the Solana Pathway

* [Node.js v14 or higher installed](https://nodejs.org/)
* [yarn installed](https://yarnpkg.com/getting-started/install)
* [git installed](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

Start by cloning the repository and installing the dependencies with yarn :

```bash
git clone https://github.com/figment-networks/learn-web3-dapp.git
cd learn-web3-dapp
yarn
```

---------------------------

## Key in the ignition 

Create an `.env.local` file at the root of the directory. Copy and paste the contents of the existing `.env.example` into the new file and save it to disk (alternatively, you can rename `.env.example`).

The value for `DATAHUB_SOLANA_API_KEY` can be found on the [DataHub Services Dashboard](https://datahub.figment.io/services/solana). Click on the Solana icon in the list of available protocols and then copy your key as shown below. You can now paste your personal API key into `.env.local` . This will authenticate you and enable you to make JSON-RPC requests to the Solana cluster via DataHub.

![](../../../.gitbook/assets/solana-setup-00.gif)

{% hint style="info" %}
[**Join us on Discord**, if you encounter any issues with the tutorial or have any questions!](https://discord.gg/fszyM7K)
{% endhint %}

---------------------------

## Start the browser

With the API key in place, save the `.env.local` file and start the React interface with :

```bash
yarn dev
```

Once the development server loads and compiles the application, open your default browser and go to `http://localhost:3000` :

![](../../../.gitbook/assets/pathway-home.gif)

{% hint style="info" %}
You can change the port of the Next.js development server, if needed: `yarn dev --port 1234`.
{% endhint %}

---------------------------

## You're in the drivers seat!

You can now move ahead to the next step by clicking on the "Next" button below on the right.
