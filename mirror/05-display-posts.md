# Step 5: Fetching a List of Posts

If you've developed full stack apps on Web 2, you're probably familiar with the concept of a query builder. These libraries facilitate interactions with databases by creating a language-native API for database interactions that abstract the complexity of writing direct SQL.

We're about to use a similar tool in Web 3. This step will require us to leverage [ArDB](https://www.npmjs.com/package/ardb) - a way to interact with Arweave with TypeScript without having to write GraphQL directly.

# Implementation 🧩

We'll start by opening the posts query endpoint file `api/arweave/search/[[query]].ts`. In this file we've started a function that will receive a query and return a list of posts. The goal is for the client to receive the list and display it so users can select a post to read.

This endpoint should be flexible enough to handle requests for the most recent posts published on the application as well as the most recent posts published by a specific user.

First we should instantiate an ardb instance, which we can do by confirming we've installed the relevant package and imported **ArDB** into this file. We should also create a `tags` variable with the app name.

```javascript
const ardb = new ArDB(arweave);
const tags = [{name: 'App-Name', values: [process.env.APP_NAME as string]}];
```

Next we should retrieve the author's address from the `query` param if there is one and create another tag in the tags list:

```javascript
const searchAddress = query && query[0];

if (searchAddress) {
  tags.push({name: 'Address', values: [searchAddress]});
}
```

With everything we need in place, we can leverage `ardb` to issue a search query for a transaction matching our `tags`. Similar to other query builders, we can chain methods to increase the specificity of the query. In this case, we'll want to find the latest 10 posts.

```javascript
const txs = await ardb.search('transactions').tags(tags).limit(10).find();
```

Now everything is in place. If you refresh your dApp, you'll see the posts you created in the previous steps. Magic!

![Figure 7: And all the posts simply... appear!](./assets/magic.jpeg)

{% label %}
Figure 7: And all the posts simply... appear!

At this point, you'll be tempted to create an entry to test the functionality. And you absolutely should do that! But don't be alarmed when you click on the entry you just created and instead of getting the post, you get a message telling you it hasn't been confirmed yet.

Recall from Box 4.1 that distributed systems take a few minutes to come to consensus and confirm transactions. Keep making progress, and before you're done with Step 6, your entry will be ready to go.

You'll also notice that if you click on a post, there's a warning in the NFT box with an error message. That's because we currently have a zero address as the default smart contract address. We'll fix that in Step 6.

##### _Listing 5.1: Code for fetching a list of posts_
```javascript
try {
  const {query} = _req.query;

  const ardb = new ArDB(arweave);

  const searchAddress = query && query[0];

  const tags = [{name: 'App-Name', values: [process.env.APP_NAME as string]}];
  if (searchAddress) {
    tags.push({name: 'Address', values: [searchAddress]});
  }

  const txs = await ardb.search('transactions').tags(tags).limit(10).find();

  const promises = txs.map((tx: any) => getData(tx._id));
  const data = await Promise.all(promises);

  res.status(200).json(data);
}
```

# Challenge 🏋️

Navigate to `api/arweave/search/[[query]].ts` in your editor and follow the steps included as comments to finish writing the query endpoint. We include a description along with a link to the documentation you need to review in order to implement each line. The relevant code block is also included in [Listing 5.2](#listing-52-instructions-for-fetching-a-list-of-posts) below.

##### _Listing 5.2: Instructions for fetching a list of posts_
```javascript
try {
  const {query} = _req.query;

  // Initialize ArDB
  // More information about ArDB can be found here: https://www.npmjs.com/package/ardb

  // Retrieve searchAddress

  // Build tags

  // Search for transaction withs App-Name and Address (optional) tags
  // More information can be found here: https://www.npmjs.com/package/ardb

  const promises = txs.map((tx: any) => getData(tx._id));
  const data = await Promise.all(promises);

  res.status(200).json(data);
}
```
