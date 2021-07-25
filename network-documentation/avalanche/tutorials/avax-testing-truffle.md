---
description: >-
  Write unit test for smart contract on the avalanche network using Solidity and Truffle
---

# How to Write Unit-Tests for Smart Contracts in Avalanche

## Introduction

Unit testing is a software testing method where individual units or components of the software are tested. The purpose of unit tests is to validate that each unit of the software performs as designed.

In smart contract development, good Unit tests are very critical. Given that smart contracts are immutable once deployed and are often responsible for managing huge sums of money, writing good unit tests cannot be over-emphasized.

A thorough understanding of the smart contract being tested is very important to the effectiveness of the test. Steps must be taken to document what the smart contract does and how it does it. The next step is to map out each test to correspond with the contract’s functionality. In this tutorial, we will follow these outlined steps to write unit tests that cover the functionality of our contract.

## **Prerequisites**

This tutorial builds on a previously written tutorial on avalanche, so before we proceed any further make sure to complete &mdash; <cite>[Making an advanced e-Voting dApp on Avalanche Fuji network using Trufflesuite][1]</cite>.

## Breaking It Down

As pointed out above, a thorough understanding of the contract is required to write effective unit tests. Here, we'll break down our smart contract into separate units and map out each of our tests to correspond with the contract's functionality. If you followed the previous tutorial, you have a basic understanding of what the contract does and how. Let's go ahead and break down the contract to figure out what we need to test for.

In the `Election.sol` file, we have one constructor and two functions:

```javascript
constructor(string[] memory _nda, string[] memory _candidates) {
    require(_candidates.length > 0, "There should be atleast 1 candidate.");
    name = _nda[0];
    description = _nda[1];
    for (uint256 i = 0; i < _candidates.length; i++) {
        addCandidate(_candidates[i]);
    }
}

function addCandidate(string memory _name) private {
    candidates[candidatesCount] = Candidate(candidatesCount, _name, 0);
    candidatesCount++;
}

function vote(uint256 _candidate) public {
    require(!voters[msg.sender], "Voter has already Voted!");
    require(
        _candidate < candidatesCount && _candidate >= 0,
        "Invalid candidate to Vote!"
    );
    voters[msg.sender] = true;
    candidates[_candidate].voteCount++;
}
```

The `constructor(_nda, _candidates)` handles the instantiation of a new Election contract, it declares two `array of string` parameters named `_nda` and `_candidates` respectively. The constructor is responsible for setting values for an Election such as `name`, `description` and `candidates`. It is apparent from this that we need a unit test that validates that when instantiated, the contract variables are set to the values passed to the constructor through the arguments.

The `addCandidate(_name)` function takes a `string` argument named `_name` - the name of the candidate - its function is to add a candidate to the `candidates mapping` variable. The function is called in the constructor so we can validate that it works as expected from the constructor test case.

The `vote(_candidate)` function takes a uint256 argument named `_candidate`. This function is used to increment the total numbers of votes a candidate has and also adds the address of the voter to the `voters` mapping. Testing this will require a unit test that calls this function and validates that the necessary variables are set to the arguments passed to the function. One more thing we need to do here is to find a way to test for the two `require()` statements to make sure the function fails according to the rule.

We can map out the above to the following test cases:

`Contructor()` test case

```
when the contract is instantiated  with (["US Election", "Presidential Election"], ["Satoshi", "Musk"])
  it should set name, description and candidates to the same values respectively

when the contract is instantiated  with (["US Election", "Presidential Election"], ["Satoshi", "Musk"])
  it should add the candidates to the candidates mapping

```

`Vote()` test cases

```
when the contract is instantiated  with (["US Election", "Presidential Election"], ["Satoshi", "Musk"])
  when vote() is called with (0)
    it should increment the voteCount of Satoshi

when the contract is instantiated  with (["US Election", "Presidential Election"], ["Satoshi", "Musk"])
  when vote() is called with (3)
    it should revert

when the contract is instantiated  with (["US Election", "Presidential Election"], ["Satoshi", "Musk"])
  when vote is called with address 0xsampleaddress
    when vote is called with address 0xsampleaddress
      it should revert
```

We have five test cases in total, two to validate that our `constructor()` and `addCandidate()` function work as expected, the other three to validate the functionality of the `vote()` function.

## Writing the unit-tests in Solidity.

Running `truffle init` in the previous article created the basic directories necessary for smart contract development. One of these directories is the `test` directory. We'll need to navigate to this directory as it is the specialized directory for test files:

```bash
cd test
```

Create a new file:

```bash
touch TestElection.sol
```

Open the TestElection.sol file with your favourite code editor and add the following lines:

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "truffle/Assert.sol";
import "../contracts/Election.sol";

contract TestElection {

}
```

Truffle provides us with a default assertion library in `truffle/Assert.sol` which we imported in the third line of the code. You can find all available assertion functions in [Assert.sol][2]. We also added an import for the contract we are going to test against in the fourth line. Assertion libraries are packages that contain functions we can use to verify that things are correct.

In the fifth line, we declared the name of the test contract as `TestElection`. Preceding the name of the contract with `Test` lets the test runner know this is going to be a test suite.

Now we are ready to start writing our test cases. The first test case we are going to write is the constructor test case:

```javascript
function testConstructor() public {
    string[] memory nda = new string[](2);
    nda[0] = "US Election";
    nda[1] = "US presidential election";

    string[] memory candidates = new string[](2);
    candidates[0] = "Satoshi";
    candidates[1] = "Musk";

    Election election = new Election(nda, candidates);

    Assert.equal(election.name(), "US Election", "Name should be US Election");
    Assert.equal(election.description(), "US presidential election", "Name should be US presidential election");
}
```

Test cases in Solidity are Solidity functions with one or more assert statements. As the case mapping in the [Breaking It Down](#breaking-it-down) section suggested, We instantiate a new Election contract with the necessary values and then use the Truffle Assertion library to validate that the `name` and `description` variables of the contract has the values as those used to instantiate the contract. The signature of the `Assert.equal()` function is `equal(string memory a, string memory b, string memory message) internal returns (bool result)`. The function compares `a` and `b` returns true/false depending on the result of the comparison.

Running the test with the `truffle test` command yields the following result:

```bash
Using network 'development'.


Compiling your contracts...
===========================
✔ Fetching solc version list from solc-bin. Attempt #1
> Compiling ./contracts/Election.sol
> Compiling ./contracts/Migrations.sol
> Compiling ./test/TestElection.sol
✔ Fetching solc version list from solc-bin. Attempt #1
> Artifacts written to /var/folders/2z/5tb906js0c9_4y8q_y2hj4l40000gn/T/test--39459-k8UW13jOkERA
> Compiled successfully using:
   - solc: 0.8.6+commit.11564f7e.Emscripten.clang



  TestElection
✔ Fetching solc version list from solc-bin. Attempt #1
    ✓ testConstructor (134ms)

1 passing (34s)

```

This shows that our test passed as expected.

{% hint style="info" %}

### Heres a sample of what the output could be if the test case failed

```bash
Using network 'development'.


Compiling your contracts...
===========================
✔ Fetching solc version list from solc-bin. Attempt #1
> Compiling ./contracts/Election.sol
> Compiling ./contracts/Migrations.sol
> Compiling ./test/TestElection.sol
✔ Fetching solc version list from solc-bin. Attempt #1
> Artifacts written to /var/folders/2z/5tb906js0c9_4y8q_y2hj4l40000gn/T/test--48586-m42AhmyIu6IK
> Compiled successfully using:
   - solc: 0.8.6+commit.11564f7e.Emscripten.clang



  TestElection
✔ Fetching solc version list from solc-bin. Attempt #1
✔ Fetching solc version list from solc-bin. Attempt #1
    1) testConstructor
    > No events were emitted


  0 passing (26s)
  1 failing

  1) TestElection
       testConstructor:
     Error: Name should be US Election (Tested: US Election, Against: US Electio)
      at checkResultForFailure (/Users/mac/.nvm/versions/node/v14.15.3/lib/node_modules/truffle/build/webpack:/packages/core/lib/testing/SolidityTest.js:66:1)
      at runMicrotasks (<anonymous>)
      at processTicksAndRejections (internal/process/task_queues.js:93:5)
      at Context.<anonymous> (/Users/mac/.nvm/versions/node/v14.15.3/lib/node_modules/truffle/build/webpack:/packages/core/lib/testing/SolidityTest.js:92:1)
```

This informs us that the comparison failed as the two values being compared are not the same.

{% endhint %}

Our second test case is the for the private `addCandidate()` function:

```javascript
function testAddCandidate() public {
    string[] memory nda = new string[](2);
    nda[0] = "US Election";
    nda[1] = "US presidential election";

    string[] memory candidates = new string[](2);
    candidates[0] = "Satoshi";
    candidates[1] = "Musk";

    Election election = new Election(nda, candidates);
    uint256 firstCandidateId;
    string memory firstCandidateName;
    uint256 firstCandidateVoteCount;

    uint256 secondCandidateId;
    string memory secondCandidateName;
    uint256 secondCandidateVoteCount;

    (firstCandidateId, firstCandidateName, firstCandidateVoteCount) = election.candidates(0);
    (secondCandidateId, secondCandidateName, secondCandidateVoteCount) = election.candidates(1);

    Assert.equal(firstCandidateId, 0, "Candidate id should be '0'");
    Assert.equal(firstCandidateName, "Satoshi", "Candidate name should be 'Satoshi'");
    Assert.equal(firstCandidateVoteCount, 0, "Candidate voteCount should be '0'");
    Assert.equal(secondCandidateId, 1, "Candidate id should be '1'");
    Assert.equal(secondCandidateName, "Musk", "Candidate name should be 'Musk'");
    Assert.equal(secondCandidateVoteCount, 0, "Candidate voteCount should be '0'");
    Assert.equal(election.candidatesCount(), 2, "Candidate count be 2");
}
```

Here, we instantiate a new Election contract, get the details of the candidate from the contract and then test that they equal the values passed to the constructor.

Next, we'll look into how to test the `require()` call in a function. Our next test case is going to be for the second `require()` call in the `vote()` function. Since Solidity v0.4.17, a function type member was added to allow access to a [function selector][3]. This made exception testing in solidity so much easier than before:

```javascript
function testVoteFailIfVoted() public {
    string[] memory nda = new string[](2);
    nda[0] = "US Election";
    nda[1] = "US presidential election";

    string[] memory candidates = new string[](2);
    candidates[0] = "Satoshi";
    candidates[1] = "Musk";

    Election election = new Election(nda, candidates);
    election.vote(0);

    (, , uint256 voteCount) = election.candidates(0);

    Assert.equal(voteCount, 1, "'voteCount' should be 1");
    Assert.isTrue(election.voters(address(this)), "Should be true");

    bytes4 selector = election.vote.selector;
    bytes memory data = abi.encodeWithSelector(selector, uint256(0));

    (bool success, ) = address(election).call(data);

    Assert.isFalse(success, "Should be false");
}
```

What we are testing here is whether the `vote()` function fails as necessary when an address is used to vote twice. To do that, we instantiate the contract, vote for a candidate twice. the second call to vote should fail for this test to pass. To test the exception, we first get access to the function selector with `bytes4 selector = election.vote.selector;`, encode the selector with `bytes memory data = abi.encodeWithSelector(selector, uint256(0));` then make an external call with the returned data from the second step with `(bool success, ) = address(election).call(data);`, lastly we validate that the transaction failed with `Assert.isFalse(success, "Should be false");`

## **Conclusion**

There you go, I believe congratulations are in order. You made it to the end of this short but interesting article on writing unit tests for smart contracts in Avalanche!.

## What's Next?

Though this article does its best to outline the steps involved in writing unit tests for smart contracts, nothing solidifies your knowledge in this new world of testing more than picking up some contracts and coming up with test cases for them. I also suggest you head on to the project repo on [Github][4]. The repo contains Javascript test files in case you're interested in writing test cases in javascript.

## About The **Author**

This tutorial was created by [Segun Ogundipe](https://www.linkedin.com/in/segun-ogundipe), You can get in touch with the author on [Figment Forum](https://community.figment.io/u/segun-ogundipe) and on [GitHub](https://github.com/segun-ogundipe)

If you had any difficulties following this tutorial or simply want to discuss Avalanche tech with us you can [**join our community today**](https://discord.gg/fszyM7K)!

[1]: https://learn.figment.io/network-documentation/avalanche/tutorials/making-advanced-e-voting-dapp-avalanche-fuji-using-trufle
[2]: https://github.com/trufflesuite/truffle/blob/develop/packages/resolver/solidity/Assert.sol
[3]: https://docs.soliditylang.org/en/v0.5.3/abi-spec.html#function-selector
[4]: https://github.com/Segun-Ogundipe/advance-voting-test
