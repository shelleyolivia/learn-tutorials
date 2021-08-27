The following software is required to set up and complete the **Celo** Pathway

* [**Node.js v14 or higher installed**](https://nodejs.org/)
* [**`yarn` installed**](https://yarnpkg.com/getting-started/install)
* [**`git` installed**](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

Start by [**cloning**](https://git-scm.com/docs/git-clone) the repository and installing the [**dependencies**](https://classic.yarnpkg.com/en/docs/managing-dependencies/) with `yarn` :

```bash
git clone https://github.com/figment-networks/learn-web3-dapp.git
cd learn-web3-dapp
yarn
```

---------------------------

# Set your key

Create an `.env.local` file at the root of the directory. Copy and paste the contents of the existing `.env.example` into the new file and save it to disk (you could also rename `.env.example` to `.env.local`).

The value for `DATAHUB_CELO_API_KEY` can be found on the [**DataHub Services Dashboard**](https://datahub.figment.io/services/secret). Click on the **Celo** icon in the list of available protocols and then copy your key as shown below. You can now paste your personal API key into `.env.local`. This will authenticate you and enable you to make JSON-RPC requests to Secret via DataHub.

![](../../../.gitbook/assets/pathways/celo/celo-setup.gif)

{% hint style="info" %}
[**Join us on Discord, if you encounter any issues with the tutorial or have any questions!**](https://discord.gg/fszyM7K)
{% endhint %}

---------------------------

# Start the server

With the API key in place, save the `.env.local` file and start the Next.js development server with:

```bash
yarn dev
```

Once the development server loads and compiles the application, open your default browser and navigate to `http://localhost:3000`:

![](../../../.gitbook/assets/pathway-home.gif)

{% hint style="warning" %}
You can change the default port if needed `yarn dev --port 1234`
{% endhint %}

---------------------------

# Next

You can now move ahead to the next step by clicking on the "Next" button below on the right. There are also links to the instructions for each step on the UI.
