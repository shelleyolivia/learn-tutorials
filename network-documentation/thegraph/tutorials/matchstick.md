# Introduction
In this tutorial we'll take a look at how to setup matchstick and write unit tests for the MasterChefV2 subgraph.  
This tutorial will typically show a call to action in the form of a squarebox with a $.  
Please note that if you need help with or would like to explore the usage of any command referenced in this tutorial, add the --help flag after the command.  
If you have any difficulty following this tutorial or simply want to discuss Solana tech with us you can join our community today!  

# Prerequisites
- Basic familiarity with a command-line interface.
- Basic familiarity with Git & GitHub.
- Basic familiarity with how TheGraph works.

# Requirements
## Tools Used

- [git](https://git-scm.com/)
- [nodejsv14+](https://nodejs.org/en/)
- [mustache](https://mustache.github.io/)
- [yarn](https://yarnpkg.com/)
- [curl](https://curl.se/)

## Getting the SushiSwap Subgraph
```sh
$ git clone https://github.com/sushiswap/sushiswap-subgraph
```  
Sushiswap has a couple of different subgraphs. For this tutorial, we wil go through the MasterChefV2 subgraph.  
```sh
$ cd sushiswap-subgraph/subgraphs/masterchefV2
```

# Tutorial
## Installing and Building the MasterChefV2 SushiSwap Subgraph
```sh
$ yarn
```
```sh
yarn install v1.22.11
[1/4] Resolving packages...
[2/4] Fetching packages...
info fsevents@2.3.2: The platform "linux" is incompatible with this module.
info "fsevents@2.3.2" is an optional dependency and failed compatibility check. Excluding it from installation.
[3/4] Linking dependencies...
[4/4] Building fresh packages...
Done in 51.76s.
```
```sh
$ yarn prepare:mainnet
```
```sh
yarn run v1.22.11
$ mustache config/mainnet.json template.yaml > subgraph.yaml
Done in 0.09s.
```
```sh
$ yarn codegen
```
```sh
.
.
.
✔ Generate types for contract ABIs
  Generate types for data source template CloneRewarderTime
  Generate types for data source template StakingRewardsSushi
  Write types for templates to generated/templates.ts
✔ Generate types for data source templates
  Load data source template ABI from packages/abis/CloneRewarderTime.json
  Load data source template ABI from packages/abis/StakingRewardsSushi.json
✔ Load data source template ABIs
  Generate types for data source template ABI: CloneRewarderTime > CloneRewarderTime (packages/abis/CloneRewarderTime.json)
  Write types to generated/templates/CloneRewarderTime/CloneRewarderTime.ts
  Generate types for data source template ABI: StakingRewardsSushi > StakingRewardsSushi (packages/abis/StakingRewardsSushi.json)
  Write types to generated/templates/StakingRewardsSushi/StakingRewardsSushi.ts
✔ Generate types for data source template ABIs
✔ Load GraphQL schema from schema.graphql
  Write types to generated/schema.ts
✔ Generate types for GraphQL schema

Types generated successfully

Done in 2.04s.
```

## There's an issue (Capital Letter) in this file.  
```sushiswap-subgraph/subgraphs/masterchefV2/src/entities/user.ts```  
Change line 4 to ```import { getMasterChef } from './masterchef'```

## Continue Building the Subgraph
```sh
$ yarn build
```
```sh
Build completed: /home/user/sushiswap-subgraph/subgraphs/masterchefV2/build/subgraph.yaml
```


## Installing the Testing Framework - matchstick
Follow the instructions at https://github.com/LimeChain/matchstick  
I'm using Ubuntu 20.04 so this is the command I will use. Please use the command that suits your operating system.
```curl
curl -OL https://github.com/LimeChain/matchstick/releases/download/0.1.2/binary-linux-20 && mv binary-linux-20 matchstick && chmod a+x matchstick
```

### Install postgressql  
```sh 
$ sudo apt install postgresql
```

### Installing Helpers  
```
$ yarn add matchstick-as
```

### To check if matchstick is working correctly run
```sh
$ ./matchstick --help
```

```sh
Matchstick 🔥 0.1.2
Limechain <https://limechain.tech>
Unit testing framework for Subgraph development on The Graph protocol.

USAGE:
    matchstick <DATASOURCE>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

ARGS:
    <DATASOURCE>    Sets the name of the datasource to use.
```

Creating Tests
Create a folder in the current directory called tests  
```sh
$ mkdir tests && cd tests && touch masterchefV2.test.ts
```  
Your testfile(masterchefV2.test.ts) should be the same name as the mappingfile  
```
dataSources:
  - kind: ethereum/contract
    name: MasterChefV2
    network: {{ network }}
    source:
      address: '{{ address }}'
      abi: MasterChefV2
      startBlock: {{ startBlock }}
    mapping:
      kind: ethereum/events
      apiVersion: 0.0.4
      language: wasm/assemblyscript
      file: ./src/mappings/masterchefV2.ts <-- This line right here!
```

Create a simple testcase to test if empty tests works.
Write the following code in ```masterchefV2.test.ts```
```js
import { clearStore, test, assert, newMockEvent } from "matchstick-as/assembly/index";
export function runTests(): void {
  test("EmptyTest", () => {
  }
}
```

Add the export to ```src/masterchefV2.ts``` at the bottom of the file
```js
export { runTests } from '../../tests/masterchefV2.test'
```
### Known issues
When runTests() is imported in the mappings file the deployment to the hosted service will break.  
For now, it's required to remove/comment out the import.


Run the Unit-Testing Framework
```sh
$ yarn build && ./matchstick MasterChefV2
```

You should see this appear on your screen.
```sh
Build completed: /home/user/sushiswap-subgraph/subgraphs/masterchefV2/build/subgraph.yaml

Done in 4.16s.

___  ___      _       _         _   _      _
|  \/  |     | |     | |       | | (_)    | |
| .  . | __ _| |_ ___| |__  ___| |_ _  ___| | __
| |\/| |/ _` | __/ __| '_ \/ __| __| |/ __| |/ /
| |  | | (_| | || (__| | | \__ \ |_| | (__|   <
\_|  |_/\__,_|\__\___|_| |_|___/\__|_|\___|_|\_\
                                                
Igniting tests 🔥

✅ EmptyTest

All tests passed! 😎
1 tests executed in 18.472016ms.
```

## Short Understanding of this Uniswap subgraph
- MasterChefV2 is a contract at ```0xEF0881eC094552b2e128Cf945EF17a6752B4Ec5d```.
- Refer to template.yaml for the ```eventHandlers``` that are hooked.
```yaml
eventHandlers:
    - event: Deposit(indexed address,indexed uint256,uint256,indexed address)
        handler: deposit
    - event: Withdraw(indexed address,indexed uint256,uint256,indexed address)
        handler: withdraw
    - event: EmergencyWithdraw(indexed address,indexed uint256,uint256,indexed address)
        handler: emergencyWithdraw
    - event: Harvest(indexed address,indexed uint256,uint256)
        handler: harvest
    - event: LogPoolAddition(indexed uint256,uint256,indexed address,indexed address)
        handler: logPoolAddition
    - event: LogSetPool(indexed uint256,uint256,indexed address,bool)
        handler: logSetPool
    - event: LogUpdatePool(indexed uint256,uint64,uint256,uint256)
        handler: logUpdatePool
```
These are the list of functions that should be tested.  
For this tutorial, We will go through the deposit event handler function.  

## Writing Tests for the Deposit Function

### Sushiswap's Deposit's ABI
From the handler function in SushiSwap, we know that the handler function looks like this.
```js
export function deposit(event: Deposit): void { ... }
```

We need to figure out how does the ```(event: Deposit)``` part of the code works...

From the EventHandler, we know that the signature of the function looks like this.
```yaml
eventHandlers:
- event: Deposit(indexed address,indexed uint256,uint256,indexed address)
    handler: deposit
```

We then look at the abi that is in the package to figure out the function signature.  
```
build/MasterChefV2/packages/abis/MasterChefV2.json
```
```json
  {
    "anonymous": false,
    "inputs": [
      {
        "indexed": true,
        "internalType": "address",
        "name": "user",
        "type": "address"
      },
      {
        "indexed": true,
        "internalType": "uint256",
        "name": "pid",
        "type": "uint256"
      },
      {
        "indexed": false,
        "internalType": "uint256",
        "name": "amount",
        "type": "uint256"
      },
      {
        "indexed": true,
        "internalType": "address",
        "name": "to",
        "type": "address"
      }
    ],
    "name": "Deposit",
    "type": "event"
  }
```
From the above, we know the signature of the DepositEvent looks like this...
```js
DepositEvent(userAddress:string, pid:BigInt, amount:BigInt, toAddress:string)
```

So... To create a deposit event, we need 4 parameters.
```js
function createDepositEvent(userAddress:string, pid:BigInt, amount:BigInt, toAddress:string): Deposit {
  // Create the Deposit Event -- Remember to Import
  let depositEvent = new Deposit();

  // Create the array of parameters...
  depositEvent.parameters = new Array();

  // Fill up the parameters in the correct order...
  let userAddressParam = new ethereum.EventParam();
  userAddressParam.value = ethereum.Value.fromAddress(Address.fromString(userAddress));
  let pidParam = new ethereum.EventParam();
  pidParam.value = ethereum.Value.fromUnsignedBigInt(pid);
  let amountParam = new ethereum.EventParam();
  amountParam.value = ethereum.Value.fromUnsignedBigInt(amount);
  let toAddressParam = new ethereum.EventParam();
  toAddressParam.value = ethereum.Value.fromAddress(Address.fromString(toAddress));

  // Add the parameters to the array
  depositEvent.parameters.push(userAddressParam);
  depositEvent.parameters.push(pidParam);
  depositEvent.parameters.push(amountParam);
  depositEvent.parameters.push(toAddressParam);
  return depositEvent;
}
```

Then, we write a test...
```js
test("Deposit Test", () => {
    // DepositEvent Signature
    // Deposit(indexed address,indexed uint256,uint256,indexed address)

    // Create a mockEvent
    let depositEvent = newMockEvent(createDepositEvent("0x89205A3A3b2A69De6Dbf7f01ED13B2108B2c43e7", BigInt.fromString("1000"), BigInt.fromString("2000"), "0x89205A3A3b2A69De6Dbf7f01ED13B2108B2c43e7")) as Deposit;

    // Let the Handler Run the Event
    deposit(depositEvent);

    // Check for Logic Correctness
    assert.fieldEquals("Pool", "1000", "id", "1000");
  });
```

Lastly, we check if the logic works correctly...
```js
export function deposit(event: Deposit): void {
  log.info('[MasterChefV2] Log Deposit {} {} {} {}', [
    event.params.user.toHex(),
    event.params.pid.toString(),
    event.params.amount.toString(),
    event.params.to.toHex()
  ])

  const masterChef = getMasterChef(event.block)
  const pool = getPool(event.params.pid, event.block)
  const user = getUser(event.params.to, event.params.pid, event.block)

  pool.slpBalance = pool.slpBalance.plus(event.params.amount)
  pool.save()

  user.amount = user.amount.plus(event.params.amount)
  user.rewardDebt = user.rewardDebt.plus(event.params.amount.times(pool.accSushiPerShare).div(ACC_SUSHI_PRECISION))
  user.save()
}
```
In the ```deposit``` handler function, we can see that there are calls to ```getPool```.  
In the getPool function, we see that a Pool entity is created.
```
export function getPool(pid: BigInt, block: ethereum.Block): Pool {
  const masterChef = getMasterChef(block)

  let pool = Pool.load(pid.toString())

  if (pool === null) {
    pool = new Pool(pid.toString())
    pool.masterChef = masterChef.id
    pool.pair = ADDRESS_ZERO
    pool.allocPoint = BIG_INT_ZERO
    pool.lastRewardBlock = BIG_INT_ZERO
    pool.accSushiPerShare = BIG_INT_ZERO
    pool.slpBalance = BIG_INT_ZERO
    pool.userCount = BIG_INT_ZERO
  }

  pool.timestamp = block.timestamp
  pool.block = block.number
  pool.save()

  return pool as Pool
}
```
Since the Pool object saves its id as pid, we can check if the pool created during the deposit function matches our variable.
```
assert.fieldEquals("Pool", "1000", "id", "1000");
```

Run the tests again and you can see that our newly created test passes!
```sh
$ yarn build && ./matchstick MasterChefV2
```

```sh
Build completed: /home/user/sushiswap-subgraph/subgraphs/masterchefV2/build/subgraph.yaml

Done in 4.16s.

___  ___      _       _         _   _      _
|  \/  |     | |     | |       | | (_)    | |
| .  . | __ _| |_ ___| |__  ___| |_ _  ___| | __
| |\/| |/ _` | __/ __| '_ \/ __| __| |/ __| |/ /
| |  | | (_| | || (__| | | \__ \ |_| | (__|   <
\_|  |_/\__,_|\__\___|_| |_|___/\__|_|\___|_|\_\
                                                
Igniting tests 🔥

✅ EmptyTest
✅ Deposit Test
INFO [MasterChefV2] Log Deposit 0x89205a3a3b2a69de6dbf7f01ed13b2108b2c43e7 1000 2000 0x89205a3a3b2a69de6dbf7f01ed13b2108b2c43e7

All tests passed! 😎
2 tests executed in 21.93132ms.
```

Here's the finalized ```masterchefV2.test.ts``` if you couldn't get it to work
```
import { Address, ethereum, BigInt } from "@graphprotocol/graph-ts";
import { clearStore, test, assert, newMockEvent } from "matchstick-as/assembly/index";

import {
  Deposit,
} from '../generated/MasterChefV2/MasterChefV2'
import { deposit } from "../src/mappings/masterchefV2";

function createDepositEvent(userAddress:string, pid:BigInt, amount:BigInt, toAddress:string): Deposit {
  // Create the Deposit Event -- Remember to Import
  let depositEvent = new Deposit();

  // Create the array of parameters...
  depositEvent.parameters = new Array();

  // Fill up the parameters in the correct order...
  let userAddressParam = new ethereum.EventParam();
  userAddressParam.value = ethereum.Value.fromAddress(Address.fromString(userAddress));
  let pidParam = new ethereum.EventParam();
  pidParam.value = ethereum.Value.fromUnsignedBigInt(pid);
  let amountParam = new ethereum.EventParam();
  amountParam.value = ethereum.Value.fromUnsignedBigInt(amount);
  let toAddressParam = new ethereum.EventParam();
  toAddressParam.value = ethereum.Value.fromAddress(Address.fromString(toAddress));

  // Add the parameters to the array
  depositEvent.parameters.push(userAddressParam);
  depositEvent.parameters.push(pidParam);
  depositEvent.parameters.push(amountParam);
  depositEvent.parameters.push(toAddressParam);
  return depositEvent;
}

export function runTests(): void {
  test("EmptyTest", () => {
    });
  
  test("Deposit Test", () => {
    // DepositEvent Signature
    // Deposit(indexed address,indexed uint256,uint256,indexed address)

    // Create a mockEvent
    let depositEvent = newMockEvent(createDepositEvent("0x89205A3A3b2A69De6Dbf7f01ED13B2108B2c43e7", BigInt.fromString("1000"), BigInt.fromString("2000"), "0x89205A3A3b2A69De6Dbf7f01ED13B2108B2c43e7")) as Deposit;

    // Let the Handler Run the Event
    deposit(depositEvent);

    // Check for Logic Correctness
    assert.fieldEquals("Pool", "1000", "id", "1000");
  });
}
```

# Conclusion
1. We went through how to setup, prepare and build the MasterChefV2 Subgraph.
1. We went through how to install matchstick for unit testing.
1. We went through how to figure out the parameters for the ethereum events.
1. We went through how to write a simple unit test for the deposit function.

# Next Steps
- Try writing more unit tests

# About the author
- This tutorial was created by antonyip. He can be found on [Github](https://github.com/antonyip).

# References
- https://github.com/sushiswap/sushiswap-subgraph
- https://github.com/LimeChain/matchstick
