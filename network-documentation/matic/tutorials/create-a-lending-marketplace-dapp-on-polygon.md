# Create a Lending Marketplace dapp on Polygon with Truffle Suite.

# Introduction

A Lending Marketplace provides a secure, flexible, open-source foundation for a decentralized loan marketplace on the Polygon blockchain. It provides the pieces necessary to create a decentralized lending exchange, including the requisite lending assets, repayments, and collateral infrastructure, enabling third parties to build applications for lending.

# Prerequisites

[MetaMask](https://metamask.io/) is a browser-based blockchain wallet that can be used to store any kind of digital assets and cryptocurrency.

[Polygon](https://docs.polygon.technology/) is a blockchain that is EVM compatible.

# Requirement

[Node.js](https://nodejs.org/en/) enables the development of fast web servers in JavaScript by bringing event-driven programming to web servers.

[Truffle Suite](https://www.trufflesuite.com/) is a development environment and testing framework for EVM-based blockchains.

[React.js](https://reactjs.org/) is an open-source JavaScript library that is used to create single-page applications' user interfaces.

# Create a truffle project

Install Truffle:
```
npm i -g truffle
```

Clone this [Git Repository](https://github.com/devilla/cryptolend.eth) and read the [Deploying and Debugging Smart Contracts on Polygon](https://learn.figment.io/tutorials/deploying-and-debugging-smart-contracts-on-polygon) tutorial to setup network config inside Truffle and learn the deployment on the Polygon network.

```
git clone https://github.com/Devilla/cryptolend.eth.git
```
Go to the repository:
```
cd cryptolend.eth
```

Install the required depencencies:
```
npm i
```

# Introduction to smart contracts

There are two main smart contracts which are regarding creating the loan offer and request, and second the contract details of the loan with repayment methods

## Loan creator
A smart contract for both the lender & the borrower to create their loan offer and request respectively. A person might be willing to offer a loan or requesting for a loan.

## Importing the dependencies

OpenZeppelin Contracts helps you minimize risk by using battle-tested libraries of smart contracts over several blockchains.

```
pragma solidity ^0.5.0;

import "openzeppelin-solidity/contracts/ownership/Ownable.sol";
import "openzeppelin-solidity/contracts/lifecycle/Pausable.sol";
import "./LoanContract.sol";

```
Here two events have been build as part of the Loan creator :
•	createNewLoanOffer
•	createNewLoanRequest

### createNewLoanOffer:

Taking input from the person who’s willing to give a loan in form of a cryptocurrency, this will include the Loan Amount, duration, data about the Collateral accepted in case the loan isn’t paid, referencing the same to the loan contract address.
     
```

function createNewLoanOffer(uint256 _loanAmount, uint128 _duration, string memory _acceptedCollateralsMetadata) public returns(address _loanContractAddress) {

         _loanContractAddress = address (new LoanContract(_loanAmount, _duration, _acceptedCollateralsMetadata, 0, address(0), 0, 0, 0, address(0), msg.sender, LoanContract.LoanStatus.OFFER));

         loans.push(_loanContractAddress);

         emit LoanOfferCreated(msg.sender, _loanContractAddress);

         return _loanContractAddress;
         
  ```
  
  Variables from the lenders end are accepted & appended into the array named loans. The loan offer created then sent as a message to that loan contract address


### createNewLoanRequest: 
This is a request from the borrower, the person who needs a loan.
Includes the Loan Amount, duration, interest the person willing to pay, data about the Collateral (collateral address, collateral amount- the cryptocurrency being requested as loan, price of the collateral in the specific cryptocurrency)& finally the loan contract address.

Variables from the requester’s end are accepted & appended into the array named loans.
The loan offer created then sent as a message to that loan contract address

```
function createNewLoanRequest(uint256 _loanAmount, uint128 _duration, uint256 _interest, address _collateralAddress, uint256 _collateralAmount, uint256 _collateralPriceInETH)
 public returns(address _loanContractAddress) {

         _loanContractAddress = address (new LoanContract(_loanAmount, _duration, "", _interest, _collateralAddress, _collateralAmount, _collateralPriceInETH, 50, msg.sender, address(0), LoanContract.LoanStatus.REQUEST));

         loans.push(_loanContractAddress);

         emit LoanRequestCreated(msg.sender, _loanContractAddress);

         return _loanContractAddress;
 }

 function getAllLoans() public view returns(address[] memory){
     return loans;

```
The second contract that we're prominently using is the `LoanContract.sol`

Importing the dependencies for our contract.

**LoanMath** is a library created for our mathematical functions(can be found in the libs folder), it consists of all the financial-related functions being used in our smart contract
**String** is a library for our string functions, it converts any type bytes32 into a string


```
pragma solidity ^0.5.0;

import "openzeppelin-solidity/contracts/ownership/Ownable.sol";
import "openzeppelin-solidity/contracts/lifecycle/Pausable.sol";
import "openzeppelin-solidity/contracts/token/ERC20/IERC20.sol";
import "./libs/LoanMath.sol";
import "./libs/String.sol";
```

**Contracts** in **Solidity** are similar to classes in _object-oriented languages_. Calling a function on a different contract (instance) will perform an _EVM function call_ and thus switch the context such that state variables in the calling contract are inaccessible // [Solidity](https://docs.soliditylang.org/en/v0.8.9/contracts.html)


Then we start creating our contract & also declare the data types used in our contract. The wallet address & admin address corresponding to our contract are also pre-mentioned.

```
contract LoanContract {

    using SafeMath for uint256;

    uint256 constant PLATFORM_FEE_RATE = 100;
    address constant WALLET_1 = 0x88347aeeF7b66b743C46Cb9d08459784FA1f6908;
    uint256 constant SOME_THINGS = 105;
    address admin = 0x95FfeBC06Bb4b7DeDfF961769055C335542E1dBF;
```

_Enumerated lists_**LoanStatus** & **CollateralStatus** are created thereafter which limit user options to select from the given options for both the lender & the loan requester.


```
 enum CollateralStatus {
        WAITING,
        ARRIVED,
        RETURNED,
        DEFAULT
    }

    struct CollateralData {

        address collateralAddress;
        uint256 collateralAmount;
        uint256 collateralPrice; // will have to subscribe to oracle
        uint256 ltv;
        CollateralStatus collateralStatus;
    }

```
Generating records for the **CollateralData** & **LoanData** with struct types
_Struct_ types are used to represent a record, a data type with more than one member of different data types. The _struct_ types we’re creating here have data about the Collateral being provided by the loan requester & the details of the loan being sanctioned.


```
 struct CollateralData {

        address collateralAddress;
        uint256 collateralAmount;
        uint256 collateralPrice; // will have to subscribe to oracle
        uint256 ltv;
        CollateralStatus collateralStatus;
    }

    struct LoanData {

        uint256 loanAmount;
        uint256 loanCurrency;
        uint256 interestRate; // will be updated on acceptance in case of loan offer
        string acceptedCollateralsMetadata; //json string
        uint128 duration;
        uint256 createdOn;
        uint256 startedOn;
       // uint256 outstandingAmount;
        mapping (uint256 => bool) repayments;
        address borrower;
        address lender;
        LoanStatus loanStatus;
        CollateralData collateral; // will be updated on accepance in case of loan offer
    }
```

A function **enrich loan** is created to  provide the details inside, once our loan is sanctioned.

```

function enrichLoan(uint256 _interestRate, address _collateralAddress, uint256 _collateralAmount, uint256 _collateralPriceInETH, uint256 _ltv) public {
        loan.interestRate = _interestRate;
        loan.collateral.collateralAddress = _collateralAddress;
        loan.collateral.collateralPrice = _collateralPriceInETH;
        loan.collateral.collateralAmount = _collateralAmount;
        loan.collateral.collateralStatus = CollateralStatus.WAITING;
        loan.collateral.ltv = _ltv;
        emit LoanContractUpdated(_interestRate, _collateralAddress, _collateralPriceInETH, _collateralAmount, _ltv);
    }
    
```
    
Below we are declaring our events to store the arguments passed in transaction logs, these logs further are stored on the blockchain & can be accessed using the address of the contract till the contract is present on the blockchain. Various events in ref the collateral transfer, funds transfer, collateral return on complete loan repayment, collateral seizure incase of loan non-payment of loan & in case any update is done further in the loan contract.

At all these events a log will be created in the blockchain which remains there till eternity & can be accessed for legal or non-legal claims & allegations by either party.



```
event CollateralTransferToLoanFailed(address, uint256);
    event CollateralTransferToLoanSuccessful(address, uint256, uint256);
    event FundTransferToLoanSuccessful(address, uint256);
    event FundTransferToBorrowerSuccessful(address, uint256);
    event LoanRepaid(address, uint256);
    event LoanStarted(uint256 _value); // watch for this event 
    event CollateralTransferReturnedToBorrower(address, uint256);
    event CollateralClaimedByLender(address, uint256);
    event CollateralSentToLenderForDefaultedRepayment(uint256,address,uint256);
    event LoanContractUpdated(uint256, address, uint256, uint256, uint256);
```


Here we’re declaring the _constructor_ function to be executed for our contract. Post execution, the final code of the contract is stored on the blockchain.

```
constructor(uint256 _loanAmount, uint128 _duration, string memory _acceptedCollateralsMetadata,
        uint256 _interestRate, address _collateralAddress,
        uint256 _collateralAmount, uint256 _collateralPriceInETH, uint256 _ltv, address _borrower, address _lender, LoanStatus _loanstatus) public {
        loan.loanAmount = _loanAmount;
        loan.duration = _duration;
        loan.acceptedCollateralsMetadata = _acceptedCollateralsMetadata;
        loan.interestRate = _interestRate;
        loan.createdOn = now;
        loan.borrower = _borrower;
        loan.lender = _lender;
        loan.loanStatus = _loanstatus;remainingCollateralAmount = _collateralAmount;
        loan.collateral = CollateralData(_collateralAddress, _collateralAmount, _collateralPriceInETH, _ltv, CollateralStatus.WAITING);
        
```
Later this will be filled when borrower accepts the loan.

functions used:
- **transferFundsToLoan** – to transfer funds to loan address after loan is sanctioned
- **toString** – converts the address into a string to which loan is being sent
- **transferCollateralToLoan** – transfers the collateral after the loan request is created

```
    // after loan offer created
    function transferFundsToLoan() public payable OnlyLender {
         require(msg.value >= loan.loanAmount, "Sufficient funds not transferred");
          loan.loanStatus = LoanStatus.FUNDED;
          //status changed OFFER -> FUNDED
         emit FundTransferToLoanSuccessful(msg.sender, msg.value);
    }
    
    function toString(address x) public returns (string memory) {
        bytes memory b = new bytes(20);
        for (uint i = 0; i < 20; i++)
            b[i] = byte(uint8(uint(x) / (2**(8*(19 - i)))));
        return string(b);
    }

    // after loan request created
    function transferCollateralToLoan() payable public OnlyBorrower  {

        ERC20 = IERC20(loan.collateral.collateralAddress);
        LoanStatus prevStatus = loan.loanStatus;

        if(loan.collateral.collateralAmount > ERC20.allowance(msg.sender, address(this))) {
            emit CollateralTransferToLoanFailed(msg.sender, loan.collateral.collateralAmount);
            revert();
        }

        loan.collateral.collateralStatus = CollateralStatus.ARRIVED;

```

We instantiate the collateral status arrived once conditions for the loan sanctioned as per the contract are met contract then transfer funds to borrower(only in case of loan offer)
And instantiate the collateral status to arrive once our conditions for loan sanctioned as per the contract are met contract will also be transferring funds to borrower (only in case of loan offer)

```
emit CollateralTransferToLoanSuccessful(msg.sender, loan.collateral.collateralAmount, loan.collateral.collateralPrice)

```


Some more functions created being used in our loan contract:

**acceptLoanOffer**: calls an event that keeps track of the acceptance of the loan offered by the requester

```
function acceptLoanOffer(uint256 _interestRate, address _collateralAddress, uint256 _collateralAmount, uint256 _collateralPriceInETH, uint256 _ltv) public {

        require(loan.loanStatus == LoanStatus.FUNDED, "Incorrect loan status");
        loan.borrower = msg.sender;
        /* This will call setters and enrich loan data */
        enrichLoan(_interestRate,_collateralAddress,_collateralAmount, _collateralPriceInETH,_ltv);

        // borrower should transfer collateral after this. use same above method? YES (validation done)
        // to be done in UI
    }
```

**approveLoanRequest**:  this calls an event to show that the loan has been approved.the date-time for a loan started will be stored used for keeping a track of repayments
```
   function approveLoanRequest() public payable {

        require(msg.value >= loan.loanAmount, "Sufficient funds not transferred");
        require(loan.loanStatus == LoanStatus.REQUEST, "Incorrect loan status");

        loan.lender = msg.sender;
        loan.loanStatus = LoanStatus.FUNDED;
        emit LoanStarted(loan.startedOn);
        // We monitor this event and block time it was fired. every duration interval apart, we call function to make a call for potentially failed repayments

        emit FundTransferToLoanSuccessful(msg.sender, msg.value);
        loan.startedOn = now;
        
        address(uint160(loan.borrower)).transfer(loan.loanAmount);
        //loan.loanStatus = LoanStatus.ACTIVE;
        emit FundTransferToBorrowerSuccessful(loan.borrower, loan.loanAmount);
    }
```

**getLoanData** : will publicly view the loan details- amount left, collateral status,loan status, addresses of the borrower & lender .at each repayment of the loan installment, this function feeds  the values in blockchain publicly viewable
 
 ```
  function getLoanData() view public returns (
        uint256 _loanAmount, uint128 _duration, uint256 _interest, string memory _acceptedCollateralsMetadata, uint256 startedOn, LoanStatus _loanStatus,
        address _collateralAddress, uint256 _collateralAmount, uint256 _collateralPrice, uint256 _ltv, CollateralStatus _collateralStatus,
        uint256 _remainingCollateralAmount,
        address _borrower, address _lender) {

        return (loan.loanAmount, loan.duration, loan.interestRate, loan.acceptedCollateralsMetadata, loan.startedOn, loan.loanStatus, loan.collateral.collateralAddress, loan.collateral.collateralAmount, loan.collateral.collateralPrice, loan.collateral.ltv, loan.collateral.collateralStatus, remainingCollateralAmount, loan.borrower, loan.lender);
    }
 ```
 
**getCurrentRepaymentNumber**: The number of the current instalment is returned as well as kept track on throughout the repayment process.
 
```
function getCurrentRepaymentNumber() view public returns(uint256) {
      return LoanMath.getRepaymentNumber(loan.startedOn, loan.duration);
    }
```



**getRepaymentAmount**: The amount for each instalment for repayment is calculated based on the instalment number & the interest being levied on.
```
function getRepaymentAmount(uint256 repaymentNumber) view public returns(uint256 amount, uint256 monthlyInterest, uint256 fees){

        uint256 totalLoanRepayments = LoanMath.getTotalNumberOfRepayments(loan.duration);

        monthlyInterest = LoanMath.getAverageMonthlyInterest(loan.loanAmount, loan.interestRate, totalLoanRepayments);

        if(repaymentNumber == 1)
            fees = LoanMath.getPlatformFeeAmount(loan.loanAmount, PLATFORM_FEE_RATE);
        else
            fees = 0;

        amount = LoanMath.calculateRepaymentAmount(loan.loanAmount, monthlyInterest, fees, totalLoanRepayments);

        return (amount, monthlyInterest, fees);
    }

```

**makeFailedRepayments**: based on the nth duration it is triggered we pass repayment number from UI.
 ```
 function makeFailedRepayments(uint256 _repaymentNumberMissed) public OnlyAdmin {
 uint256 repaymentNumber = _repaymentNumberMissed;
 require(loan.repayments[repaymentNumber] == false,"repayment was already paid");
 (uint256 _repayAmount,uint256 interest,uint256 fees) = getRepaymentAmount(repaymentNumber);
         uint256 collateralAmountToTrasnfer = LoanMath.calculateCollateralAmountToDeduct((_repayAmount.sub(fees)).mul(SOME_THINGS.div(100)), loan.collateral.collateralPrice);
         ERC20 = IERC20(loan.collateral.collateralAddress);
         ERC20.transfer(loan.lender, collateralAmountToTrasnfer);
         emit CollateralSentToLenderForDefaultedRepayment(repaymentNumber,loan.lender,collateralAmountToTrasnfer);
  }
 
 ```
 

**repayLoan**: Keeps track of if the loan has been completely paid & emits the event to be stored on the blockchain. The instalment number of the repayment id also logged in at the same.
 
 ```
     function repayLoan() public payable {

        require(now <= loan.startedOn + loan.duration * 1 minutes, "Loan Duration Expired");

        uint256 repaymentNumber = LoanMath.getRepaymentNumber(loan.startedOn, loan.duration);

        (uint256 amount, , uint256 fees) = getRepaymentAmount(repaymentNumber);

        require(msg.value >= amount, "Required amount not transferred");

        if(fees != 0){
            transferToWallet1(fees);
        }
        uint256 toTransfer = amount.sub(fees);

      //  loan.outstandingAmount = loan.outstandingAmount.sub(msg.value);

      //  if(loan.outstandingAmount <= 0)
      //      loan.loanStatus = LoanStatus.REPAID;

        loan.repayments[repaymentNumber] = true;

        address(uint160(loan.lender)).transfer(toTransfer);

       // should log particular repaymentNumber paid instead
        emit LoanRepaid(msg.sender, amount);
    }
 ```

**transferToWallet1**: fees for contract use will be taken by the contract service provider. will be private only viewable to the owner of the contract
 
 ```
 function transferToWallet1(uint256 fees) private {
        address(uint160(WALLET_1)).transfer(fees);
    }
 
 ```
 
 
**transferCollateralToWallet1**: The collateral will be transferred in the wallet provided by the contract owner, for a fair use policy of the contract.
```
    function transferCollateralToWallet1 (uint256 fees) private {
        uint256 feesInCollateralAmount = LoanMath.calculateCollateralAmountToDeduct(fees, loan.collateral.collateralPrice);
        ERC20 = IERC20(loan.collateral.collateralAddress);
        ERC20.transfer(WALLET_1, feesInCollateralAmount);
    }
```

## Compile and migrate

Loan contract & loancreator is being used 

Open `truffle console` to run a local blockchain in your terminal at `http://127.0.0.1:9545/`:
```
truffle develop
```

This will and display `Account addresses` along with their `Private Keys` and `Mnemonic` required for deploying the smart contracts.

In the `truffle console` compile the smart contracts:

```
truffle(develop)> compile

Compiling your contracts...
===========================
> Compiling .\contracts\LoanContract.sol
> Compiling .\contracts\LoanCreator.sol
> Compiling .\contracts\LoanProduct.sol
> Compiling .\contracts\Migrations.sol
> Compiling .\contracts\StandardToken.sol
> Compiling .\contracts\libs\DateTime\DateTime.sol
> Compiling .\contracts\libs\DateTime\api.sol
> Compiling .\contracts\libs\LoanMath.sol
> Compiling .\contracts\libs\LoanMethods.sol
> Compiling .\contracts\libs\String.sol
> Compiling openzeppelin-solidity\contracts\GSN\Context.sol
> Compiling openzeppelin-solidity\contracts\access\Roles.sol
> Compiling openzeppelin-solidity\contracts\access\roles\PauserRole.sol
> Compiling openzeppelin-solidity\contracts\lifecycle\Pausable.sol
> Compiling openzeppelin-solidity\contracts\math\SafeMath.sol
> Compiling openzeppelin-solidity\contracts\ownership\Ownable.sol
> Compiling openzeppelin-solidity\contracts\token\ERC20\IERC20.sol
> Compilation warnings encountered:

> Artifacts written to C:\Users\hp\cryptolend\build\contracts
> Compiled successfully using:
   - solc: 0.5.0+commit.1d4f565a.Emscripten.clang
```

Now, `migrate` the compiled smart contracts:

```
truffle(develop)> migrate

Starting migrations...
======================
> Network name:    'develop'
> Network id:      5777
> Block gas limit: 6721975 (0x6691b7)


2_deploy_contracts.js
=====================

   Deploying 'LoanCreator'
   -----------------------
   > transaction hash:    0x232be40e9171c62f74585c52e15492a8a8653b8a65eb9f97f6e57ccdcb0eec66
   > Blocks: 0            Seconds: 0
   > contract address:    0xc7Eb239cA1e53093B645A50d70B4a895AAD94cb0
   > block number:        4
   > block timestamp:     1629372448
   > account:             0x2F3CeD6f849630301feC1dD613869E8cc3857665
   > balance:             99.990053476
   > gas used:            2460473 (0x258b39)
   > gas price:           2 gwei
   > value sent:          0 ETH
   > total cost:          0.004920946 ETH


   Deploying 'StandardToken'
   -------------------------
   > transaction hash:    0xa74cf285dc8e80b73b91a9334304a408c973a67ecd9d8e700559c7c7d8e321d8
   > Blocks: 0            Seconds: 0
   > contract address:    0xBD613f04E9Fd211b95A608776620B6C49f11A421
   > block number:        5
   > block timestamp:     1629372449
   > account:             0x2F3CeD6f849630301feC1dD613869E8cc3857665
   > balance:             99.988584512
   > gas used:            734482 (0xb3512)
   > gas price:           2 gwei
   > value sent:          0 ETH
   > total cost:          0.001468964 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:         0.010922144 ETH


Summary
=======
> Total deployments:   2
> Final cost:          0.010922144 ETH
- Saving migration to chain.
```

# Using UI in the browser

Clone this [Git Repository](https://github.com/Devilla/cryptolend.ui)
```
git clone https://github.com/Devilla/cryptolend.ui.git
```
Browse the project directory:
```
cd cryptolend.ui
```

Install dependencies required for the project:
```
npm i
```

Run the app in the browser:
```
npm start
```
In the chrome/firefox browser create a [Custom RPC](https://medium.com/stakingbits/setting-up-metamask-for-polygon-matic-network-838058f6d844) in the Networks as follows:

```
Network Name: Polygon
New RPC URL: https://rpc-mainnet.matic.network or
ChainID: 137
Symbol: MATIC
Block Explorer URL: https://polygonscan.com/
```

Open the lending dapp in browser and go to the `/myloans` path and connect to Polygon Network in the Metamask extention. Now feel free to go throgh the various features of thhe lending marketplace like creating a loan request on `/request` path and loan offer on `/offer`, which can also be browsed from the Navigation bar.

# Conclusion

Now you know about creating a Lending Marketplace with Truffle Suite and ReactJS on the Polygon network.

If you had any difficulties following this tutorial or simply want to discuss Polygon tech with us you can [**join our community today**](https://community.figment.io/) or [**Join our discord channel**](https://discord.gg/fszyM7K)!

# About the author

[Devendra Yadav](https://community.figment.io/u/dev.koold) [Prince Rana]

# References
- https://github.com/crypto-lend
- https://learn.figment.io/tutorials/deploying-and-debugging-smart-contracts-on-polygon
- https://docs.soliditylang.org/en/v0.8.7/
