# Introduction

In this tutorial we will be building a complete Dapp for the Near Registry. Near Registry will allow you to track popular 
content just like Hacker News. Anyone can post an entry to the Near Registry. Upvoting the content requires you to 
attach NEAR. The registry is sorted based on the NEAR attached to the entries. Same logic can be extended for any 
other type of registry.


# Prerequisites:

Install Nodejs (≥ 14) and yarn 
```bash
npm install --global yarn
```

# Requirements

NEAR testnet account - https://wallet.testnet.near.org/

# Setup

```bash
git clone https://github.com/viraja1/near-registry.git
cd near-registry
yarn install
yarn test
```

# Exploring The Code

1. The backend code is present in the `/assembly` folder and gets deployed to
   the NEAR blockchain when you run `yarn deploy`. This sort of
   code that runs on a blockchain is called a "smart contract". [Learn more
   about NEAR smart contracts][smart contract docs].
2. The backend test code is present in the `/assembly/__tests__/` folder. 
    We can test the backend code using `yarn test` command.
3. The frontend code is present in the `/src` folder. It uses near-api-js which is a JavaScript/TypeScript library for 
   development of decentralized applications on the NEAR platform. It works in conjunction with NEAR RPC endpoints 
   to help you connect your application to the NEAR blockchain. [Learn more
    about NEAR Frontend][NEAR Frontend]. 

We can write NEAR smart contracts in Rust or AssemblyScript. For Near Registry we will be using AssemblyScript. 
AssemblyScript is a dialect of TypeScript that compiles to Wasm. 

Contracts are a named collection of (exported) functions that have access (via near-sdk-as) to their execution 
context (sender, receiver, block height, etc) as well as storage services (key-value pair and convenience collections 
like Map, Vector and Deque), logging services and some utility functions.

To keep things organized, contracts can use one or more data objects which are commonly added to the model.ts file.
All contracts and models must explicitly import features of the NEAR they intend to use. 

#### assembly/model.ts

```typescript
import {context, u128, PersistentVector} from "near-sdk-as";

/**
 * Exporting a new class Entry so it can be used outside of this file.
 */
@nearBindgen
export class Entry {
  sender: string;

  constructor(public title: string, public description: string, public url: string, public id: i32, public votes: u128) {
    this.sender = context.sender;
  }
}

/**
 * collections.vector is a persistent collection. Any changes to it will
 * be automatically saved in the storage.
 * The parameter to the constructor needs to be unique across a single contract.
 * It will be used as a prefix to all keys required to store data in the storage.
 */
export const entries = new PersistentVector<Entry>("entries");
```

In the above models.ts file, we define a new custom type named Entry (primitive types like integers, strings and 
bool are always available by default). Since models are just AssemblyScript classes, they support custom constructors. 
Each entry has sender, title, description, url, id and votes. Each of these is associated with a specific type 
and is declared as public. In the constructor, we define the sender as  context.sender. The context object provides 
context for contract execution including information about the transaction sender, blockchain height, and attached 
deposit available for use during contract execution, etc.
                                                              
At the end, we define that entries is a PersistentVector (collection) of type Entry. The PersistentVector writes and reads 
from storage abstracting away a lot of what you might want to add to the storage object. They wrap 
the Storage class with convenience methods so you must always use a unique storage prefix for different collections 
to avoid data collision. PersistentVector acts like an array. To create entries, we use the synthax 
`new PersistentVector<Entry>("entries")`. The vector supports the methods like push, pop and length.


#### assembly/main.ts

```typescript
import {Entry, entries} from './model';
import {context, u128} from "near-sdk-as";

// --- contract code goes below

/**
 * Adds a new entry under the name of the sender's account id.\
 * NOTE: This is a change method. Which means it will modify the state.\
 * But right now we don't distinguish them with annotations yet.
 */
export function addEntry(title: string, description: string, url: string): void {
  // Creating a new entry and populating fields with our data
  const entry = new Entry(title, description, url, entries.length, u128.fromU64(0));
  // Adding the entry to end of the the persistent collection
  entries.push(entry);
}


/**
 * Up vote an entry using attachedDeposit\
 * NOTE: This is a change method. Which means it will modify the state.\
 * But right now we don't distinguish them with annotations yet.
 */
export function upVoteEntry(index: i32): void {
  const entry = entries[i32(index)];
  entry.votes = u128.add(entry.votes, context.attachedDeposit);
  entries[i32(index)] = entry;
}

/**
 * Returns an array of entries.\
 * NOTE: This is a view method. Which means it should NOT modify the state.
 */
export function getEntries(): Entry[] {
  const result = new Array<Entry>(entries.length);
  for (let i = 0; i < entries.length; i++) {
    result[i] = entries[i];
  }
  return result;
}
```

Contract function calls are stateless. Any state that you want to save to the blockchain needs to be explicitly 
saved by interacting with the storage object e.g. entries. In the main.ts file, the contract functions like 
`addEntry`, `upVoteEntry` and `getEntries` are defined. Function declarations follow standard AssemblyScript conventions, 
including the parameters they take, optional arguments and return values. 

There are two types of functions that can interact with the blockchain -- "view" functions and "change" functions. 
The difference, however, does not exist on the contract level. 
View functions like `getEntries` do not actually change the state of the blockchain. Change functions 
like `addEntry` and `upVoteEntry` modify the state.

In the addEntry function, we create a new Entry and add it to the entries PersistentVector using the push method. 
In the upVoteEntry function, we first fetch the entry from the entries PersistentVector using the index. Then we 
increment the entry votes using context.attachedDeposit. Then we update the entries PersistentVector using the index.

#### assembly/__tests__/registry.spec.ts

```typescript
import {addEntry, getEntries, upVoteEntry} from '../main';
import {Entry, entries} from '../model';
import {VMContext, Context, u128} from 'near-sdk-as';

function createEntry(title: string, description: string, url: string): Entry {
  return new Entry(title, description, url, 0, u128.fromU64(0));
}

const entry = createEntry('Near Protocol - Infrastructure for Innovation',
  'NEAR is an open source platform that accelerates the development of decentralized applications.',
  'https://near.org/');

describe('entries tests', () => {
  afterEach(() => {
    while (entries.length > 0) {
      entries.pop();
    }
  });

  it('adds a entry', () => {
    addEntry('Near Protocol - Infrastructure for Innovation',
      'NEAR is an open source platform that accelerates the development of decentralized applications.',
      'https://near.org/');
    expect(entries.length).toBe(
      1,
      'should only contain one entry'
    );
    expect(entries[0].url).toStrictEqual(
      'https://near.org/',
      'url matches'
    );
  });

  it('check up vote', () => {
    addEntry('Near Protocol - Infrastructure for Innovation',
      'NEAR is an open source platform that accelerates the development of decentralized applications.',
      'https://near.org/');
    expect(entries[0].votes.toString()).toStrictEqual(u128.fromU32(0).toString(),
      'entry should have 0 vote'
    );
    VMContext.setAttached_deposit(u128.from('10000000000000000000000'));
    upVoteEntry(i32(0));
    expect(entries[0].votes.toString()).toStrictEqual(u128.from('10000000000000000000000').toString(),
      'entry should have a vote'
    );
  });

  it('retrieves entries', () => {
    addEntry('Near Protocol - Infrastructure for Innovation',
      'NEAR is an open source platform that accelerates the development of decentralized applications.',
      'https://near.org/');
    const entriesList = getEntries();
    expect(entriesList.length).toBe(
      1,
      'should be one entry'
    );
    expect(entriesList).toIncludeEqual(
      entry,
      'entries should include:\n' + entry.toJSON()
    );
  });
});

describe('attached deposit tests', () => {
  beforeEach(() => {
    VMContext.setAttached_deposit(u128.fromString('0'));
    VMContext.setAccount_balance(u128.fromString('0'));
  });

  it('attaches a deposit to a contract call', () => {
    log('Initial account balance: ' + Context.accountBalance.toString());

    addEntry('Near Protocol - Infrastructure for Innovation',
      'NEAR is an open source platform that accelerates the development of decentralized applications.',
      'https://near.org/');
    VMContext.setAttached_deposit(u128.from('10'));

    log('Attached deposit: 10');
    log('Account balance after deposit: ' + Context.accountBalance.toString());

    expect(Context.accountBalance.toString()).toStrictEqual(
      '10',
      'balance should be 10'
    );
  });
});
```

To make sure our smart contract works as expected, we have written the above JavaScript tests. It tests various cases
like add an entry, upvote an entry, retrieve entries and attach a deposit to a contract call. To run the tests, we use
`yarn test` command. It then prints the results to the console.

# Deploy Smart Contract

Every smart contract in NEAR has its [own associated account][NEAR accounts]. 
When you run `yarn dev`, your smart contracts get deployed to the live NEAR testnet with a throwaway account. 
To make it permanent, use the following steps.


#### 1) Install near-cli

Install near-cli globally

```bash
npm install --global near-cli
```

Then ensure that it is installed properly using the following command

```bash
near --version
```
    


#### 2) Create an account for the contract

Visit [NEAR Wallet] and make a new account. You'll be deploying these smart contracts to this new account.

Now authorize NEAR CLI for this new account, and follow the instructions it gives you:

```bash
near login
```

#### 3) Deploy the smart contracts

```bash
yarn deploy
```

As you can see in `package.json`, this builds & deploys smart contracts to NEAR testnet.

Note down the smart contract account id from the console and update it in src/config.js:1

# Run the Frontend locally

Run the following commands from the near-registry folder

```bash
cd src
yarn install
yarn start
```

Then visit http://localhost:1234 from your browser to test the Near Registry

# Deploy the Frontend using netlify

Run the following commands from the near-registry folder

```bash
cd src
yarn build
yarn global add netlify-cli
netlify login
netlify deploy --prod
```

Then follow the instructions given by the netlify cli and specify `./dist` as the publish directory


# Screenshots
![](../../../.gitbook/assets/near_registry_1.png)

![](../../../.gitbook/assets/near_registry_2.png)

![](../../../.gitbook/assets/near_registry_3.png)

![](../../../.gitbook/assets/near_registry_4.png)

# App Link (NEAR Testnet)
https://near-registry.netlify.app/

# Conclusion
Congratulations! We have successfully deployed the NEAR Registry smart contract on the NEAR testnet. We have also tested the smart 
contracts by writing the test cases. We have then interacted with the smart contracts using the React frontend. 

# References
* NEAR Guest Book - https://github.com/near-examples/guest-book/
* NEAR Smart Contract Docs - https://docs.nearprotocol.com/docs/roles/developer/contracts/assemblyscript
* NEAR Frontend Docs - https://docs.near.org/docs/api/naj-quick-reference

# About the Author
This tutorial was created by [Viraj](https://github.com/viraja1/).


  [NEAR]: https://nearprotocol.com/
  [yarn]: https://yarnpkg.com/
  [AssemblyScript]: https://docs.assemblyscript.org/
  [smart contract docs]: https://docs.nearprotocol.com/docs/roles/developer/contracts/assemblyscript
  [asp]: https://www.npmjs.com/package/@as-pect/cli
  [NEAR accounts]: https://docs.nearprotocol.com/docs/concepts/account
  [NEAR Wallet]: https://wallet.testnet.near.org/
  [near-cli]: https://github.com/nearprotocol/near-cli
  [CLI]: https://www.w3schools.com/whatis/whatis_cli.asp
  [create-near-app]: https://github.com/nearprotocol/create-near-app
  [NEAR Frontend]: https://docs.near.org/docs/api/naj-quick-reference
