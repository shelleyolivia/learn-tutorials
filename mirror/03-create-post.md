# Step 3: Writing a Post

Traditional blogs use databases to persist a post. An author might write some content and publish it, which triggers a create action on the server that writes the data to the database. The database would have schema that define the data structure. For instance, we might have a **posts** table with columns for post id, content, and author id.

The blockchain however is a bit different because it's decentralized. Rather than writing to a specific database, we'll be leveraging a distributed system that can't be controlled by a central authority.
  
## Arweave - Sustainable Onchain Storage

Arweave is a decentralized network that allows users with extra storage space to provide that space as a storage to the network in exchange for economic incentives. Users looking to store data in a decentralized, secure, censorship-resistant way, can leverage the network to do so. Effectively, Arweave is the storage part of the stack for dApps.

Arwaeve uses a technology called "blockweaves" to achieve this. Similar to a blockchain, which connects transactions in a main chain across blocks, blockweaves are connected blocks that contain data. One of the main differences is that rather than connect blocks as a singly linked list, blockweaves are more like graphs.

This system requires a robust and sustainable economic structure to ensure storage providers are incentivized to replicate information forever at reasonable costs to the user. Basically, the user pays a small fee upfront to permanently store data. That fee is treated as principal that generates interest over time for storage providers.

The architecture to achieve all this is beyond the scope of this tutorial, but if you're interested in going deeper, we recommend checking out the [Arweave yellow paper](https://www.arweave.org/yellow-paper.pdf).

# Implementation 🧩

There are several ways to interact with Arweave. You might expect that we would build something on a testnet-like set up first. But we're actually going to interact with the Arweave mainnet directly. This is enabled by a faucet program from Arweave that allows us to create an Arweave wallet and fund it with just enough tokens to get us going.

## Arweave Wallet

Go to the [Arweave mainnet faucet](https://faucet.arweave.net/) and follow the instructions to create a wallet. Make sure you download the JSON file at the end of the process. Re-name that file to `arweave-wallet.json` and save it at the root of the project. You'll notice we have included this in the `.gitignore`.

Now initialize the `ARWEAVE_WALLET` environment variable in the command line:

```text
$ ARWEAVE_WALLET=$(cat arweave-wallet.json)
```

## Posting to Arweave

With the wallet set up, we can now develop an endpoint to create posts. In `/api/arweave/post.ts` we have a skeleton endpoint for us to add the Arweave write logic. You'll notice that we're passing in `req.body` which includes data and address fields. The data in this case will be the post's text and the address will be the public address of the author.

First we need to initialize the wallet by using `JSON.parse`:

```javascript
const wallet = JSON.parse(process.env.ARWEAVE_WALLET as string);
```

We should also instantiate a transaction by leveraging [Arweave's JavaScript library](https://github.com/ArweaveTeam/arweave-js) and passing in the data and the wallet.

```javascript
const transaction = await arweave.createTransaction({data: data}, wallet);
```

We also want to add tags as the metadata that will associate the data with our app and with the author. We should also include a tag denoting the data "application/json".

```javascript
transaction.addTag('App-Name', process.env.APP_NAME as string);
transaction.addTag('Content-Type', 'application/json');
transaction.addTag('Address', address);
```

Then we want to sign the transaction and post it to Arweave:

```javascript
await arweave.transactions.sign(transaction, wallet);
await arweave.transactions.post(transaction);
```

You'll notice that Arweave's syntax is pretty straightforward. The only new aspect of this workflow is the signing piece, which is required to authenticate the transaction as valid.

To finish the endpoint, we'll want to return the transaction id as part of the response:

```javascript
res.status(200).json(transaction.id);
```

## Finishing the Form

Now we need to make use of our post endpoint. To do that, we need to go to `CreatePostForm.tsx` and finish the `handleSubmit` function.

In order to submit the Arweave transaction that creates the post, we can leverage axios and call the `post` endpoint we wrote above. We should pass in the data and address as part of the request body. And we should store the return value, the transaction id, in a variable as well since we'll need it later when we incorporate the NFT functionality.

```javascript
const response = await axios.post(routes.api.arweave.post, {
  data,
  address,
});
const transactionId = response.data;
```

With that our users are able to create entries that will live forever on the Arweave blockchain! There's one issue though - even though we can create an entry, we can't see it in our dApp yet. We'll address that shortly in Step 4.

##### _Listing 3.1: Code for creating a post endpoint_

```javascript
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

##### _Listing 3.2: Code for handling post submission_

```javascript
if (provider && contract) {
  const data = createJsonMetaData(values);

  // Submit Arweave transaction
  // Use axios to post data and address to api/arweave/post endpoint.
  // This request should return transactionId

  // Mint NFT
  // Get signer and connect it to smart contract
  // More information can be found here: https://docs.ethers.io/v5/getting-started/#getting-started--writing

  // Call `createToken` method passing in transactionId
  const response = await axios.post(routes.api.arweave.post, {
    data,
    address,
  });
  const transactionId = response.data;
```

# Challenge 🏋️

Navigate to `[ ]` in your editor and follow the steps included as comments to finish writing the `[ ]` function. We include a description along with a link to the documentation you need to review in order to implement each line. The relevant code block is also included in [Listing 3.2](#listing-32-instructions-for-creating-a-post) below.

##### _Listing 3.3: Instructions for creating a post endpoint_

```javascript
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
    //Documentation can be found here: https://github.com/ArweaveTeam/arweave-js

    // Sign Arweave transaction with your wallet. Documentation can be found here: https://github.com/ArweaveTeam/arweave-js

    // Post Arweave transaction. Documentation can be found here: https://github.com/ArweaveTeam/arweave-js

    // Return transaction id
  }
  ...
}
```

##### _Listing 3.4: Instructions for writing handling post submission_

```javascript
if (provider && contract) {
  const data = createJsonMetaData(values);
  // Submit Arweave transaction
  // Use axios to post data and address to api/arweave/post endpoint.
  // This request should return transactionId
  alert('Entry created successfully');
}
```
