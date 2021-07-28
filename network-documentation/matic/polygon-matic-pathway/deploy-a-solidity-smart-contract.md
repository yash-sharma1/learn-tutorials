---
description: 'Learn how to write, test and deploy Solidity on Polygon using Truffle'
---

# Deploy a Solidity smart contract

Solidity is a high level language. It is partly designed after ECMAScript and therefore it is said to be **similar to JavaScript**. But the similarity ends there. It gets compiled \(not interpreted\) and usually deployed on Blockchains that understand the Ethereum Virtual Machine \(EVM\), like Polygon! When a smart contract is deployed, it becomes immutable. This has both benefits and drawbacks, which we will discuss below.

We can use [HardHat](https://hardhat.org) or [Truffle](https://trufflesuite.com) to ease development and deployment of our Solidity code. The [Remix IDE](https://remix.ethereum.org) is also a popular choice for contract development and deployment. There are several existing guides for other EVM compatible networks available on Figment Learn \(check out the tutorials for [Celo and Truffle](https://learn.figment.io/network-documentation/celo/tutorial/deploying-smart-contracts-on-celo-with-truffle), or [Celo and HardHat](https://learn.figment.io/network-documentation/celo/tutorial/celo-hardhat-deploy-and-nft-app) for example\). Both HardHat and Truffle are capable of running local development blockchains. We will focus on using Truffle in this tutorial.

Run the following commands to get started

```text
npm install -g truffle
cd contracts/polygon/SimpleStorage
yarn
```

## The SimpleStorage Solidity contract

One of the most basic, non-trivial, types of smart contract is a **simple storage contract**.   
This example was adapted from the [Solidity documentation](https://solidity.readthedocs.io/en/develop/introduction-to-smart-contracts.html).

{% code title="contracts/polygon/SimpleStorage/contracts/SimpleStorage.sol" %}
```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract SimpleStorage {
    uint storedData;

    constructor() {
  		  storedData = 0;
  	}

    function set(uint x) public {
        storedData = x;
    }

    function get() public view returns (uint) {
        return storedData;
    }
}
```
{% endcode %}

The first line of a Solidity file should contain a comment which describes the type of license governing the source code. The `SPDX-License-Identifier` will most commonly be the MIT licence, although a comprehensive list can be found at [https://spdx.org/licenses/](https://spdx.org/licenses/). The Solidity compiler will issue a warning if this line is not present at compilation time.

The next line specifies the version of the Solidity compiler to be used when compiling this contract. Using [semantic versioning](https://semver.org/), it is possible to prevent a Solidity file from being compiled by incompatible versions - most often in the case of breaking changes between major versions.

Next we define our contract name, `SimpleStorage` - The contract name can be anything, but should be descriptive of the functionality. The naming convention for Solidity is that the filename should match the UpperCamelCase contract name, hence `SimpleStorage.sol`.

The `constructor()` is called only once, when the contract is deployed. The constructor is the place to assign default values to any variables, and performs an initial configuration of the app state. We will set the initial value of the `storedData` variable to `0`.

Next we declare a function signature for the `set()` function, which has a [visibility](https://docs.soliditylang.org/en/v0.8.6/contracts.html#visibility-and-getters) of public. `set` takes a single argument `x`, an unsigned integer \([uint](https://docs.soliditylang.org/en/v0.8.6/types.html#integers)\). The function body is a single line, which assigns the value passed via `x` to the `storedData` state variable.

The `get()` function signature is slightly different, in that there is no argument being passed. It also has a visibility of public, is a [view](https://docs.soliditylang.org/en/v0.8.6/types.html?highlight=view#function-types) type of function, and specifies a return type of `uint`. Its function body will simply return the current value of `storedData`.

## Test the smart contract

This test uses the Truffle-provided `Assert` and `DeployedAddresses` contracts. These are built in to Truffle and the development blockchain. It is also important that we import the Solidity file we want to test! 

Because deployed bytecode is immutable, it is best to work with security and best practices in mind. Prevent accidentally deploying code with errors by always _testing prior to deployment_. We will test our SimpleStorage contract with Truffle. 

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "truffle/Assert.sol";
import "truffle/DeployedAddresses.sol";
import "../contracts/SimpleStorage.sol";

contract TestSimpleStorage {

  function testInitialStoredDataUsingDeployedContract() public {
    SimpleStorage store = SimpleStorage(DeployedAddresses.SimpleStorage());
    Assert.equal(store.get(), 0, "storedData should be 0 initially");
  }

  function testSettingStoredData() public {
    SimpleStorage store = new SimpleStorage();
    uint expected = 10000;
    store.set(expected);
    Assert.equal(store.get(), expected, "storedData should equal 10000");
  }

}
```

The first test will be run against a frehly deployed version of the SimpleStorage code every time, which is why the initial test for `storedData` should always equal zero.

The second test sets the value of `storedData` to `10000` and then queries it from the blockchain in the same manner as the first test, asserting that it will be equal to `10000`. 

Simple functionality, simple tests!  
Before running the tests, we must ensure we are running a local blockchain with Truffle. Open a separate terminal window and run the command:

```text
truffle develop
```

With this local blockchain running, we can test the SimpleStorage contract with the command:

```text
truffle test
```

This will compile the contract before deploying it to the Truffle development chain and performing the tests. You should see similar output in your terminal:

```text
Using network 'development'.


Compiling your contracts...
===========================
> Compiling ./test/TestSimpleStorage.sol
> Artifacts written to /var/folders/xc/jxvcxfc901nd0jmqv594w8sw0000gn/T/test--89605-BscbjjTFIYq2
> Compiled successfully using:
   - solc: 0.8.0+commit.c7dfd78e.Emscripten.clang



  TestSimpleStorage
    ✓ testInitialStoredDataUsingDeployedContract (60ms)
    ✓ testSettingStoredData (56ms)


  2 passing (6s)
```

## Deploy the smart contract

Before we deploy, we must do one last thing to prepare. We need to put a secret recovery phrase \(or mnemonic if you prefer\) of our MATIC funded Metamask account into the file `contracts/polygon/SimpleStorage/.secret.example` and rename the file to `.secret`.   
  
If you recall, at the beginning of the pathway we reminded users to keep their secret recovery phrase at hand and to generate a new Metamask wallet ONLY for the purposes of the tutorial. This is why. We would never ask anybody to put an actively used private key or secret recovery phrase into an unsecure location.

Because we are operating on a testnet, it is _less_ of a concern but still very much a concern.

Compiling Solidity with Truffle is a straightforward process, just make sure that your preferred configuration is set in `truffle-config.js` \(paths, compilers, networks, etc.\) and then run the command:

```text
truffle compile
```

Deploying Migrations with Truffle is quite similar to deploying, but provides more flexibility for custom workflows. A full explanation of migrations is beyond the scope of this tutorial, but please do read the Truffle [documentation](https://www.trufflesuite.com/docs/truffle/getting-started/running-migrations) on the subject. To deploy the SimpleStorage contract to Polygon, run this command :

```text
truffle migrate --network matic
```

The flag `--network matic` lets Truffle know which network we want to deploy our migrations to. The configuration for each network is set inside of `truffle-config.js`. 

## Using the Application Binary Interface \(ABI\):

[The Solidity Contract ABI Specification](https://docs.soliditylang.org/en/v0.8.6/abi-spec.html) explains that an ABI is a standard way to interact with contracts in the Ethereum ecosystem, both from outside the blockchain and for contract-to-contract interaction. Data is encoded according to its type, as described in the specification. The encoding is not self describing and thus requires a schema in order to decode.

The ABI is considered an "[artifact](https://trufflesuite.github.io/artifact-updates/background.html#what-are-artifacts)" in relation to a compiled Solidity contract. Most commonly, developers will interact with an ABI in JSON format. Read more about [what this means](https://docs.soliditylang.org/en/v0.8.6/abi-spec.html#json).

## Next Steps

Now that we have a deployed and functioning smart contract, let's interact with it!
