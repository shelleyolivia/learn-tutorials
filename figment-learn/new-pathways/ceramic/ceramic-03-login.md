# Log In

## Requirements

“@3id/connect”: “^0.2.5”,
“@ceramicnetwork/3id-did-resolver”: “^1.4.3”,
“@ceramicnetwork/http-client”: “^1.3.0”,
“@ceramicstudio/idx”: “^0.12.2”,
“dids”: “^2.4.0”,

## Description
Connecting your Metamask wallet to dApp allows for easy interaction with smart contracts. You can make transaction, query the network, all from the context of your account. What if you could associate data like name, avatar, social accounts and also application-specific data to your account? In this tutorial we will use IDX to log user in using IDX. But first let’s explain few things. You already know about DIDs which is a W3C standard for decentralized identities. In order to get a DID and then use it for authentication you have to use DID provider that conforms to a particular DID method.  In this tutorial we will use 3ID Connect  DID provider which is a wallet that handles hey management for you.

```js
const logIn = async (address: string): Promise<string> => {
  const threeIdConnect = new ThreeIdConnect();

  const provider = new EthereumAuthProvider(window.ethereum, address);

  await threeIdConnect.connect(provider);

  const didInstance = new DID({
    provider: threeIdConnect.getDidProvider(),
    resolver: {
      ...ThreeIdResolver.getResolver(ceramicRef.current),
    },
  });

  await ceramicRef.current.setDID(didInstance);

  const did = ceramicRef.current.did as DID;
  const userDID = await did.authenticate();

  if (setCurrentUserDID) {
    setCurrentUserDID(userDID);
  }

  idxRef.current = new IDX({
    ceramic: ceramicRef.current,
    aliases,
  });

  return userDID;
};
```

## What happened here?
First we create an instance of an Ethereum provider  and tie it to a specific address that we receive after connecting our wallet to our website.

Then we make treeIdConnect to connect to this provider and once this is done we create a new instance of DID for which we provide provider and a resolver. Resolver is responsible for return DID document give the DID string and provider is responsible for providing a json-roc interface which allows for the creation and usage of a DID.
Then we set did instance to ceramic client.
Then we call authenticate which will start the authentication process. A popup will appear on the top right corrner which will prompt you for either create or update your existing identity. You will need to sign tx using Metamask and your DID should be associated with your Ethereum account now.
As a last step we create an IDX instance with authenticated ceramic client instance and any aliases that we want to use.




