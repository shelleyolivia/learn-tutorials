# Set your Datahub API key

{% hint style="info" %}
If you have previously completed a Pathway, you may already have `.env.local`! Just add your Tezos API key in the appropriate place and remember to save the file _before_ starting the development server.
{% endhint %}

Create an `.env.local` file at the root of the directory. Copy and paste the contents of the existing `.env.example` into the new file and save it to disk (you could also rename `.env.example` to `.env.local`).

The value for `DATAHUB_TEZOS_API_KEY` can be found on the [**DataHub Services Dashboard**](https://datahub.figment.io/services/secret). Click on the **Tezos** icon in the list of available protocols and then copy your key as shown below. You can now paste your personal API key into `.env.local`. This will authenticate you and enable you to make JSON-RPC requests to Secret via DataHub.

![](../../../.gitbook/assets/pathways/tezos/tezos-setup.gif)

{% hint style="info" %}
[**Join us on Discord**](https://discord.gg/fszyM7K), if you encounter any issues with the tutorial or have any questions!**
{% endhint %}

---------------------------

# Next

You can now move ahead to the next step by clicking on the "Next" button below on the right. There are also links to the instructions for each step on the UI.
