Traditional blogs use databases to persist an entry. An author might write some content and publish it, which triggers a create action on the server that writes the data to the database. The database would have schema that define the data structure. For instance, we might have an **entries** table with columns for entry id, content, and author id.

The blockchain, however, is a bit different because it's decentralized. Rather than writing to a specific database, we'll be leveraging a distributed system that cannot be controlled by a central authority.
  
# Sustainable, Decentralized Storage 💾

[Arweave](https://www.arweave.org/) is a decentralized network that allows users with extra storage space to provide that space as storage to the network in exchange for economic incentives. Users looking to store data in a decentralized, secure, censorship-resistant way, can leverage the network to do so. Effectively, Arweave is the storage part of the stack for dApps.

Arweave uses a technology called "blockweaves" to achieve this. Similar to a blockchain, which connects transactions in a main chain across blocks, blockweaves are connected blocks that contain data. One of the main differences is that rather than connect blocks as a singly linked list, blockweaves are more like graphs.

This system requires a robust and sustainable economic structure to ensure storage providers are incentivized to replicate information forever at reasonable costs to the user. Basically, the user pays a small fee upfront to permanently store data. That fee is treated as principal that generates interest over time for storage providers.

The architecture to achieve all this is beyond the scope of this tutorial, but if you're interested in going deeper, we recommend checking out the [Arweave yellow paper](https://www.arweave.org/yellow-paper.pdf).

![Like this but decentralized...](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/mirror/assets/storage.jpeg)

{% label %}
Like this but decentralized...

# Implementation 🧩

There are several ways to interact with Arweave. In our case we're going to use a faucet program from Arweave that allows us to create an Arweave wallet and fund it with just enough tokens to get us going.

{% sidenote title="Box 3.1: What's a faucet?" %}
Faucets are smart contracts that transfer a limited amount of tokens to users for testing purposes. Most protocols implement faucets to facilitate developers' testing and building efforts.

# Arweave Wallet 👜

Go to the [Arweave mainnet faucet](https://faucet.arweave.net/) and follow the instructions to create a wallet. Make sure you download the JSON file at the end of the process. Re-name that file to `arweave-wallet.json` and save it at the root of the project. You'll notice we have included this in the `.gitignore` to prevent exposing any private information.

Once you've saved the JSON file to the root, we should stop the local server in our terminal (press CTRL+C in the terminal window where the server is running). Then we can initialize the `ARWEAVE_WALLET` environment variable in the command line and run the local server again:

```text
$ ARWEAVE_WALLET=$(cat arweave-wallet.json) yarn dev
```

# Posting to Arweave 📨

With the Arweave wallet in place, we can now develop an endpoint to create entries. In `pages/api/arweave/entry.ts`, we have a skeleton endpoint for us to add the Arweave write logic. You'll notice that we're passing in `req.body` which includes data and address fields. The data in this case will be the entry's text and the address will be the author's public address. In the file `routes.ts` you can see how the routes are defined. 

First we need to initialize the wallet by using `JSON.parse`:

```typescript
const wallet = JSON.parse(process.env.ARWEAVE_WALLET as string);
```

We should also instantiate a transaction by leveraging [Arweave's JavaScript library](https://github.com/ArweaveTeam/arweave-js) and passing in the data and the wallet.

```typescript
const transaction = await arweave.createTransaction({data: data}, wallet);
```

Next, we'll want to add tags as the metadata that will associate the data with our dApp and with the author. We should also include a tag denoting the data as "application/json".

```typescript
transaction.addTag('App-Name', process.env.APP_NAME as string);
transaction.addTag('Content-Type', 'application/json');
transaction.addTag('Address', address);
```

Then, we want to sign the transaction and post it to Arweave:

```typescript
await arweave.transactions.sign(transaction, wallet);
await arweave.transactions.post(transaction);
```

You'll notice that Arweave's syntax is pretty straightforward. The only new aspect of this workflow is the signing piece, which is required to authenticate the transaction as valid.

To finish the endpoint, we'll want to return the transaction id as part of the response:

```typescript
res.status(200).json(transaction.id);
```

# Finishing the Form 📝

Now we need to make use of our entry endpoint. To do that, we need to go to `components/CreateEntryForm/CreateEntryForm.tsx` and finish the `handleSubmit` function.

In order to submit the Arweave transaction that creates the entry, we can use [axios](https://axios-http.com/docs/intro) to call the `entry` endpoint we wrote above. We should pass in the data and address as part of the request body, and we should store the return value - the transaction id - in a variable as well. We'll need the transaction id later when we incorporate the NFT functionality into our dApp.

```typescript
const response = await axios.post(routes.api.arweave.post, {
  data,
  address,
});
const transactionId = response.data;
console.log('transactionId: ', transactionId);
```

With that complete, our users are able to create entries that will live forever on the Arweave blockchain! You can create an entry yourself and confirm that the `transactionId` is printed in the console.

There's one issue though. Even though we can create an entry, we can't see it in our dApp yet. We'll address that shortly, but first we need an endpoint to fetch entries. We'll tackle that next in Step 4.

##### _Listing 3.1: Code for creating a entry endpoint_

```typescript
export default async function (
  req: NextApiRequest,
  res: NextApiResponse<string>,
): Promise<any> {
  try {
    const {data, address} = req.body;

    const wallet = JSON.parse(process.env.ARWEAVE_WALLET as string);
    const transaction = await arweave.createTransaction({data: data}, wallet);

    transaction.addTag('App-Name', process.env.APP_NAME as string);
    transaction.addTag('Content-Type', 'application/json');
    transaction.addTag('Address', address);

    await arweave.transactions.sign(transaction, wallet);
    await arweave.transactions.post(transaction);
    res.status(200).json(transaction.id);
  }
  ...
}
```

##### _Listing 3.2: Code for handling entry submission_

```typescript
if (provider && contract) {
  const data = createJsonMetaData(values);

  const response = await axios.post(routes.api.arweave.post, {
    data,
    address,
  });
  const transactionId = response.data;
  console.log('transactionId: ', transactionId);
}
```

# Challenge 🏋️

Navigate to `pages/api/arweave/entry.ts` and `components/CreateEntryForm/CreateEntryForm.tsx` in your code editor and follow the steps included as comments to finish writing the create entry functionality. We include a description along with a link to the documentation you need to review in order to implement each line. The relevant code block is also included in **Listing 3.3** and **Listing 3.4** below.

##### _Listing 3.3: Instructions for creating a entry endpoint_

```typescript
export default async function (
  req: NextApiRequest,
  res: NextApiResponse<string>,
): Promise<any> {
  try {
    const {data, address} = req.body;

    // Initialize wallet using ARWEAVE_WALLET environmental variable (Tip: Use JSON.parse)

    // Create Arweave transaction passing in data. Documentation can be found here: https://github.com/ArweaveTeam/arweave-js

    // Add tags:
    // - App-Name - APP_NAME environmental variable
    // - Content-Type - Should be application/json
    // - Address - Address of a user
    // Documentation can be found here: https://github.com/ArweaveTeam/arweave-js

    // Sign Arweave transaction with your wallet. Documentation can be found here: https://github.com/ArweaveTeam/arweave-js

    // Post Arweave transaction. Documentation can be found here: https://github.com/ArweaveTeam/arweave-js

    // Return transaction id
  }
  ...
}
```

##### _Listing 3.4: Instructions for handling entry submission_

```typescript
if (provider && contract) {
  const data = createJsonMetaData(values);
  // Submit Arweave transaction
  // Use axios to post data and address to api/arweave/entry endpoint.
  // This request should return transactionId

  // Stop here when you complete Step 3 ^^^^
  ...
}
```

Once you have completed the code, you will want to try out creating an entry by clicking on the **Create Entry** button on the Dashboard, then entering a title and some text.

![Screenshot displaying a list of entries](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/mirror/assets/entries.jpg)