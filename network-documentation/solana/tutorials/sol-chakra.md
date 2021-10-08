# Introduction

In the existing tutorials about Solana, we understood how we make smart contracts (or _programs_, as we say in the Solana ecosystem) and mint tokens and NFTs. However, a Dapp is incomplete without a frontend, and we cannot reach wider audience unless we host it on web. So, in this tutorial, we will understand how to develop frontend for Solana programs and how to make beautiful web apps effortlessly with the help of Chakra UI.

The code for this tutorial is available in the repository [https://github.com/kanav99/solana-boilerplate](https://github.com/kanav99/solana-boilerplate). The tutorial is split into parts and each part has a commit specific to it on the repository. Do a `git checkout <commit hash>` to refer the code at that checkpoint.

# Create an empty Chakra App

- Create an empty Chakra UI using the Create-React-App template.

```
npx create-react-app solana-boilerplate --template @chakra-ui
cd solana-boilerplate
```

- This creates a scaffolding for simple web app, with `src/App.js` file containing a rotating Logo. We don't need that, so we reduce the file to -

```jsx
import React from "react";
import {
  ChakraProvider,
  Box,
  Text,
  VStack,
  Grid,
  theme,
} from "@chakra-ui/react";
import { ColorModeSwitcher } from "./ColorModeSwitcher";

function App() {
  return (
    <ChakraProvider theme={theme}>
      <Box textAlign="center" fontSize="xl">
        <Grid minH="100vh" p={3}>
          <ColorModeSwitcher justifySelf="flex-end" />
          <VStack spacing={8}>
            <Text>Hello world!</Text>
          </VStack>
        </Grid>
      </Box>
    </ChakraProvider>
  );
}

export default App;
```

You can try running the app by running `npm start` and you'll see an empty page with a convenient color mode switching button with a "Hello world!" floating in the middle.

If this is your first time seeing a Chakra UI app, here's a refresher. All chakra components must be wrapped between `ChakraProvider` which controls the theming of all children components. `Box` is equivalent to `div` tag of HTML. `VStack` is a vertical stack of elements spaced evenly.

# Basic interaction with the Solana Network

To interact with the Solana Networks (mainnet, devnet, local etc), we use the Solana [JSON RPC API](https://docs.solana.com/developing/clients/jsonrpc-api). But the people at Solana Labs have made an NPM package `@solana/web3.js` which interacts with the API.

- We begin by installing the package to the app

```
npm install --save @solana/web3.js
```

- Import the package in App.js

```jsx
import * as web3 from "@solana/web3.js";
```

- We are not using wallets as of now (more on that in subsequent sections), so we will make a makeshift wallet, that is, if there private key doesn't exist in localstorage, generate a new one. So, we add this code to the global scope

```jsx
// Making a connection with Solana devnet
const connection = new web3.Connection(
  web3.clusterApiUrl("devnet"),
  "confirmed"
);

// Access a localstorage item pvkey
const pvkey = localStorage.getItem("pvkey");
var wallet;
if (pvkey === null) {
  // if nothing is found
  // generate a new wallet keypair and store the private key in localstorage
  wallet = web3.Keypair.generate();
  localStorage.setItem("pvkey", wallet.secretKey);
} else {
  // if existing wallet is found, parse the wallet
  let arr = new Uint8Array(pvkey.replace(/, +/g, ",").split(",").map(Number));
  // and create a wallet object
  wallet = web3.Keypair.fromSecretKey(arr);
}
```

Keep in mind - this makeshift wallet is not to be used in production. We will replace this with actual wallets in the coming sections, current code is just for understanding the structure. Now we have setup a connection with the network and have a wallet ready as well. Let's display some basic properties of the wallet - namely, public key and balance. Public key can be simply retrieved by `wallet.publicKey.toBase58()`, but the balance needs to be fetched from the network.

In the App.js file, to retrieve balance on-load, we use `useEffect` to interact with the network. We maintain a state variable to store the account info. Inside the App function,

```jsx
const [account, setAccount] = useState(null);

useEffect(() => {
  async function init() {
    // get account info from the network
    let acc = await connection.getAccountInfo(wallet.publicKey);
    setAccount(acc);
  }
  init();
}, []);
```

We have fetched the account details. This object contains the balance in lamport units, which is equivalent to 1/1000000000 of a SOL. Now to display the public key and balance, we change the rendering code

```jsx
<Text>Wallet Public Key: {wallet.publicKey.toBase58()}</Text>
<Text>
  Balance:{' '}
  {account ? (account.lamports / web3.LAMPORTS_PER_SOL) + ' SOL' : 'Loading..'}
</Text>
```

You can also use `connection.getBalance` to get only balance as well.

All the code till now is present in the `d7ecab7` commit of the final repository.

# Getting Airdrops

To work with the solana network, we need some SOLs. To get them on the mainnet, we need to buy them from any exchange and then transfer to the public key displayed on the page. But we are on devnet, so we can just get them for free through airdrops. We can do it from the CLI, but lets make an easy button to get them on click.

Import `useCallback` from react, `Button` and `toast` (we will use [toasts](https://chakra-ui.com/docs/feedback/toast) for beautiful erroring) from chakra. First, define the callback that gets an airdrop and updates account object.

```jsx
const toast = useToast();
const [airdropProcessing, setAirdropProcessing] = useState(false);

const getAirdrop = useCallback(async () => {
  setAirdropProcessing(true);
  try {
    var airdropSignature = await connection.requestAirdrop(
      wallet.publicKey,
      web3.LAMPORTS_PER_SOL
    );
    await connection.confirmTransaction(airdropSignature);
  } catch (error) {
    toast({ title: "Airdrop failed", description: error });
  }
  let acc = await connection.getAccountInfo(wallet.publicKey);
  setAccount(acc);
  setAirdropProcessing(false);
}, [toast]);
```

And add a button in the rendering part

```jsx
<Button onClick={getAirdrop} isLoading={airdropProcessing}>
  Get Airdrop of 1 SOL
</Button>
```

Get code on commit `83fa3e5`.

# Get Transaction history

Suppose we want to get the 10 most recent transactions whenever we load the page. We can create a new react state `transactions` and update the code inside the `init` function.

```jsx
const [transactions, setTransactions] = useState(null);
...
  async function init() {
    let acc = await connection.getAccountInfo(wallet.publicKey);
    setAccount(acc);
    let transactions = await connection.getConfirmedSignaturesForAddress2(
      wallet.publicKey,
      {
        limit: 10,
      }
    );
    setTransactions(transactions);
  }
```

and add this loop inside the rendering part.

```jsx
<Heading>Transactions</Heading>;
{
  transactions && (
    <VStack>
      {transactions.map((v, i, arr) => (
        <HStack key={"transaction-" + i}>
          <Text>Signature: </Text>
          <Code>{v.signature}</Code>
        </HStack>
      ))}
    </VStack>
  );
}
```

But we also want to call this `init` function after we airdrop, so that the transaction that added the SOLs also gets logged. So we can move this init function out of useEffect, but inside the `App` functional component. Hence, just before the function `getAirdrop` ends, we can call `init` and update the account balance and transactions.

You might be seeing a problem a problem over here - the transaction, balance, or account status in general, is not realtime. Which is what we will fix in the next section. For the code till now, refer to the commit `f50143c`.

# Realtime account updates using polling and custom React Hooks

Ever wondered how functions like `useToast` (in chakra), `useWallet` (in most web3 frameworks) work? These are all custom hooks. Right now, in our code, we have all of the code for frontend, wallet info, getting airdrop is all in single function `App`. The account info is not realtime. The code in the `App` functional component should not have access to `setAccounnt` or `setTransactions`, it should just recieve account info and list of transactions and show it as a list. We will fix all of it using React custom hooks. Using hooks, we can localize a part of the application's state inside a separate function, that internally manages and updates the value of those state, returning only the value of the realtime state. If you don't understand any part of what's coming, just bear with me until the code for the hook is complete.

All custom hooks should start with `use`, so let's call our hook `useSolanaAccount`. We know for sure that the hook should have the state
of `account` and `transactions` and should return both.

```jsx
function useSolanaAccount() {
  const [account, setAccount] = useState(null);
  const [transactions, setTransactions] = useState(null);
  // updating logic here
  return { account, transactions };
}
```

Now empty state is not that useful, let's update it. We want the state to be updated every 1 second. So, it would suffice to run the same old `init` function every 1 second. So, let's move that function in here.

```jsx
function useSolanaAccount() {
  const [account, setAccount] = useState(null);
  const [transactions, setTransactions] = useState(null);

  async function init() {
    let acc = await connection.getAccountInfo(wallet.publicKey);
    setAccount(acc);
    let transactions = await connection.getConfirmedSignaturesForAddress2(
      wallet.publicKey,
      {
        limit: 10,
      }
    );
    setTransactions(transactions);
  }

  // more code here

  return { account, transactions };
}
```

Now to run `init` every 1 second, we will use the `useEffect` hook to set an interval using `setInterval`. You may ask why inside `useEffect`? That's because we only want to set the interval once - writing it directly inside the function body will call it every time the state updates, causing it to run the `init` function multiple times (i.e, the number of times state changed) every second (too much math? you can remember it as golden rule - don't run any `fetch`y code directly inside the react functional component or hook). Final hook looks like this

```jsx
function useSolanaAccount() {
  const [account, setAccount] = useState(null);
  const [transactions, setTransactions] = useState(null);

  async function init() {
    let acc = await connection.getAccountInfo(wallet.publicKey);
    setAccount(acc);
    let transactions = await connection.getConfirmedSignaturesForAddress2(
      wallet.publicKey,
      {
        limit: 10,
      }
    );
    setTransactions(transactions);
  }

  useEffect(() => {
    setInterval(init, 1000);
  }, []);

  return { account, transactions };
}
```

Now, we can remove the definition and all calls of `init` from `App` and change the state inside it to

```jsx
...
const { account, transactions } = useSolanaAccount();
const toast = useToast();
const [airdropProcessing, setAirdropProcessing] = useState(false);

...
```

Voila! No more `setAccount` and `setTransactions` inside the App component. No more manually updating state after every change. Try sending some SOLs from the CLI to this account and see it update in real time! As an exercise to the reader, try using [`connection.onAccountChange`](https://solana-labs.github.io/solana-web3.js/classes/Connection.html#onAccountChange) instead of setInterval for getting updates :). Code until now is present in the commit `c598b22`.

# Using an actual wallet

Now before spending any of our precious SOLs, we want our users to feel safe about how we handle wallets. And it is always a good idea to leave the wallet logic and how the private key is stored to existing well-known projects. Solana has many wallet options like Solflare, Sollet, Phantom etc. We will make our application compatible with all of these using [`solana/wallet-adapter`](https://github.com/solana-labs/wallet-adapter) package(s). The goal of this section would be to remove this particular piece of code -

```jsx
const connection = new web3.Connection(
  web3.clusterApiUrl("devnet"),
  "confirmed"
);

const pvkey = localStorage.getItem("pvkey");
var wallet;
if (pvkey === null) {
  wallet = web3.Keypair.generate();
  localStorage.setItem("pvkey", wallet.secretKey);
} else {
  let arr = new Uint8Array(pvkey.replace(/, +/g, ",").split(",").map(Number));
  wallet = web3.Keypair.fromSecretKey(arr);
}
```

and replace it with something safer. The current implementation enables the person who hosts this application access to every user's wallet, which isn't right.

- Start by installing the required packages

```bash
npm i @solana/wallet-adapter-wallets \
      @solana/wallet-adapter-base \
      @solana/wallet-adapter-react \
      @solana/wallet-adapter-react-ui
```
