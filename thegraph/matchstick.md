# Introduction
In this tutorial we'll take a look at how to setup matchstick and write unit tests for the MasterChefV2 subgraph.  
This tutorial will typically show a call to action in the form of a codebox.  
Please note that if you need help with or would like to explore the usage of any command referenced in this tutorial, add the `--help` flag after the command.  
 
# Prerequisites
- Basic familiarity with a command-line interface.
- Basic familiarity with Git & GitHub.
- Basic familiarity with how TheGraph works.
- Basic understanding of why [unit-tests](https://en.wikipedia.org/wiki/Unit_testing) are important.

# Requirements
The following software must be installed:
- [nodejs](https://nodejs.org/en/) v14+ (a minimum of 14.17.6 LTS is recommended)
- [curl](https://curl.se/)
- [git](https://git-scm.com/)
- [yarn](https://yarnpkg.com/)
- [mustache](https://mustache.github.io/)

Getting the SushiSwap Subgraph 

```text
git clone https://github.com/sushiswap/sushiswap-subgraph
``` 

SushiSwap has a couple of different subgraphs. For this tutorial, we will go through the MasterChefV2 subgraph. 

```text
cd sushiswap-subgraph/subgraphs/masterchefV2
```  
# Getting Started

We will begin by installing and building the MasterChefV2 SushiSwap subgraph. From the root directory of the cloned repository, run yarn to install the dependencies:  

```text
yarn
``` 

This will output something similar:  

```text
yarn install v1.22.11
[1/4] Resolving packages...
[2/4] Fetching packages...
info fsevents@2.3.2: The platform "linux" is incompatible with this module.
info "fsevents@2.3.2" is an optional dependency and failed compatibility check. Excluding it from installation.
[3/4] Linking dependencies...
[4/4] Building fresh packages...
Done in 51.76s.
```  

Next, we will use a script from `package.json` to load the configurations into subgraph.yaml. Run the command:  

```text
yarn prepare:mainnet
```  

Your output should be:  

```text
yarn run v1.22.11
mustache config/mainnet.json template.yaml > subgraph.yaml
Done in 0.09s.
```  

Then, we will generate the classes needed for the subgraph to work correctly. 

```text
yarn codegen
```  

Your output should look like:  

```text
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

**There's an issue with capitalization in this file: 
`sushiswap-subgraph/subgraphs/masterchefV2/src/entities/user.ts`**  
You will need to change line 4 of this file to the following:  

```typescript
import { getMasterChef } from './masterchef'
```  

Lastly, we will build the subgraph.

```text
yarn build
```  
The final line of your output should be:  

```text
Build completed: /home/user/sushiswap-subgraph/subgraphs/masterchefV2/build/subgraph.yaml
```  

# Installing the matchstick testing framework

Follow the [installation instructions](https://github.com/LimeChain/matchstick) for matchstick.  
I'm using Ubuntu 20.04 so this is the command I will use.  

```text
curl -OL https://github.com/LimeChain/matchstick/releases/download/0.1.2/binary-linux-20 && mv binary-linux-20 matchstick && chmod a+x matchstick
```  

**Install postgressql**  

Install postgressql, it is required for our testing framework.  

```text 
sudo apt install postgresql
```  

**Install matchstick helpers**

Now we can install the matchstick helpers, with the command: 

```
yarn add matchstick-as
```  

To check that matchstick is working correctly, use the command:  
```text
./matchstick --help
```  

This should output: 

```text
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

# Creating Tests

Create a folder in the current directory called `tests`:

```text
mkdir tests && cd tests && touch masterchefV2.test.ts
``` 

> If you're using Microsoft Windows, you can use this command instead 
> ```text
> mkdir tests && cd tests && type nul > masterchefV2.test.ts
> ```  

Your test file (`masterchefV2.test.ts`) should have the same name as the mapping file:

```yaml
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
      file: ./src/mappings/masterchefV2.ts # This line right here!
```  

Now we can create a simple test case to see if empty tests work. Write the following code in `masterchefV2.test.ts`:

```typescript
import { clearStore, test, assert, newMockEvent } from "matchstick-as/assembly/index";
export function runTests(): void {
  test("EmptyTest", () => {
  }
}
```  

Add the export to the file `src/masterchefV2.ts` at the end of the file:  

```typescript
export { runTests } from '../../tests/masterchefV2.test'
```  

> **Known issue**  
> When runTests() is imported in the mappings file, the deployment to the hosted service will break. For now, it's required to remove/comment out the import.

Now it is time to run the unit testing framework:

```text
yarn build && ./matchstick MasterChefV2
```  

You should see similar output:  

```text
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

# Understanding the SushiSwap subgraph

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

This is the list of functions that should be tested. For this tutorial, we will go through the deposit event handler function.  

# Testing the Deposit Function

**SushiSwap's Deposit ABI**

From the handler function in SushiSwap, we know that the handler function looks like this: 

```js
export function deposit(event: Deposit): void { ... }
```  

How does the `(event: Deposit)` part of the code work? Let's take a closer look:  

From the EventHandler, we know that the signature of the function looks like this:

```yaml
eventHandlers:
- event: Deposit(indexed address,indexed uint256,uint256,indexed address)
    handler: deposit
```  

We then look at the ABI that is in the package (`build/MasterChefV2/packages/abis/MasterChefV2.json
`) to figure out the function signature.

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

From the above, we know the signature of the DepositEvent looks like this:

```js
DepositEvent(userAddress:string, pid:BigInt, amount:BigInt, toAddress:string)
```  

So, to create a deposit event we need 4 parameters:

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

Then, we write a test case:

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

Lastly, we check if the logic works correctly:

```js
export function deposit(event: Deposit): void {
  log.info('[MasterChefV2] Log Deposit {} {} {} {}', [
    event.params.user.toHex(),
    event.params.pid.toString(),
    event.params.amount.toString(),
    event.params.to.toHex()
  ])

  const masterChef = getMasterChef(event.block)
  const pool = getPool(event.params.pid, event.block) // getPool is called here!
  const user = getUser(event.params.to, event.params.pid, event.block)

  pool.slpBalance = pool.slpBalance.plus(event.params.amount)
  pool.save()

  user.amount = user.amount.plus(event.params.amount)
  user.rewardDebt = user.rewardDebt.plus(event.params.amount.times(pool.accSushiPerShare).div(ACC_SUSHI_PRECISION))
  user.save()
}
```

In the deposit handler function, we can see that there are calls to getPool. In the getPool function, we see that a Pool entity is created:

```js
export function getPool(pid: BigInt, block: ethereum.Block): Pool {
  const masterChef = getMasterChef(block)

  let pool = Pool.load(pid.toString()) // pid is saved here!

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

Since the Pool object saves the input of pid into its id, we can check if the id of the pool created during the deposit function matches the pid:

```js
// This function takes in 4 variables: (GraphObjectName, GraphObjectID, GraphObjectParamName, GraphObjectParamValue)
assert.fieldEquals("Pool", "1000", "id", "1000");
```  

Run the tests again and you can see that our newly created test passes! 

```text
yarn build && ./matchstick MasterChefV2
```  

Your output should look like this:  

```text
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

Here's the finalized `masterchefV2.test.ts`, in case you couldn't get it to work:

```js
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
2. We went through how to install matchstick for unit testing.
3. We went through how to figure out the parameters for the ethereum events.
4. We went through how to write a simple unit test for the deposit function.

# Next Steps

1. Try changing `assert.fieldEquals("Pool", "1000", "id", "1000");` to make the test fail! What happens?
2. What other tests cases can you think of? Try writing more unit tests.

# About the author

- This tutorial was created by Anton Yip. He can be found on [Github](https://github.com/antonyip).

# References

- https://github.com/sushiswap/sushiswap-subgraph
- https://github.com/LimeChain/matchstick
