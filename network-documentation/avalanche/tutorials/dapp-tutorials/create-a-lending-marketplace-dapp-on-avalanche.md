# Create a Lending Marketplace dapp on Avalanche with Truffle Suite.

## Introduction

A Lending Marketplace provides a secure, flexible, open-source foundation for a decentralized loan marketplace on the Avalanche blockchain. It provides the pieces necessary to create a decentralized lending exchange, including the requisite lending assets, clearing, and collateral pool infrastructure, enabling third parties to build applications for lending.

## Prerequisites

[MetaMask](https://metamask.io/) is a browser-based blockchain wallet that can be used to store any kind of digital assets and cryptocurrency.
 
[Avalanche](https://www.avax.network/) is a blockchain that is EVM compatible.

[Create a Local Test Network](https://learn.figment.io/tutorials/create-a-local-test-network), [Using Truffle with the Avalanche C-Chain](https://learn.figment.io/network-documentation/tutorials/using-truffle-with-the-avalanche-c-chain)

## Requirement

[Node.js](https://nodejs.org/en/) enables the development of fast web servers in JavaScript by bringing event-driven programming to web servers.

[Truffle Suite](https://www.trufflesuite.com/) is a development environment and testing framework for EVM-based blockchains.

[React.js](https://reactjs.org/) is an open-source JavaScript library that is used to create single-page applications' user interfaces.

## Create a truffle project 

Install Truffle:
```
npm i -g truffle
```

Create a sample boilerplate:
```
truffle init
```

Add the smart contract code below:

LoanContract.sol

```
pragma solidity ^0.5.0;

import "openzeppelin-solidity/contracts/ownership/Ownable.sol";
import "openzeppelin-solidity/contracts/lifecycle/Pausable.sol";
import "openzeppelin-solidity/contracts/token/ERC20/IERC20.sol";
import "./libs/LoanMath.sol";
import "./External/PriceFeeder.sol";
import "./libs/String.sol";

/*import "github.com/OpenZeppelin/openzeppelin-solidity/contracts/token/ERC20/IERC20.sol";
import "github.com/OpenZeppelin/openzeppelin-solidity/contracts/math/SafeMath.sol";
import "./LoanMath.sol";
import "./PriceFeeder.sol";*/

contract LoanContract {

    using SafeMath for uint256;

    uint256 constant PLATFORM_FEE_RATE = 100;
    address constant WALLET_1 = 0x88347aeeF7b66b743C46Cb9d08459784FA1f6908;
    uint256 constant SOME_THINGS = 105;
    address admin = 0x95FfeBC06Bb4b7DeDfF961769055C335542E1dBF;

    enum LoanStatus {
        OFFER,
        REQUEST,
        ACTIVE,
        FUNDED,
        REPAID,
        DEFAULT
    }

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

    function enrichLoan(uint256 _interestRate, address _collateralAddress, uint256 _collateralAmount, uint256 _collateralPriceInETH, uint256 _ltv) public {
        loan.interestRate = _interestRate;
        loan.collateral.collateralAddress = _collateralAddress;
        loan.collateral.collateralPrice = _collateralPriceInETH;
        loan.collateral.collateralAmount = _collateralAmount;
        loan.collateral.collateralStatus = CollateralStatus.WAITING;
        loan.collateral.ltv = _ltv;
        emit LoanContractUpdated(_interestRate, _collateralAddress, _collateralPriceInETH, _collateralAmount, _ltv);
    }

    LoanData loan;

    //PriceFeeder price;

    IERC20 public ERC20;

    uint256 public remainingCollateralAmount = 0;

    /* struct Repayment {
        bytes32 id;
        uint256 repaidOn;
        uint256 amount;
        uint256 repaymentNumber;
    } */

    //mapping (uint256 => bool) internal repayments;

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

    modifier OnlyBorrower {
        require(msg.sender == loan.borrower, "Not Authorised");
        _;
    }
    
     modifier OnlyAdmin {
        require(msg.sender == admin, "Only Admin");
        _;
    }
    
    modifier OnlyLender {
        require(msg.sender == loan.lender, "Not Authorised");
        _;
    }
    
    
    
    // watch for this event  LoanStartedOn during two transactions approveLoanRequest & transferCollateralToLoan

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
        loan.loanStatus = _loanstatus;
        // loan.repayments = ;
        //loan.outstandingAmount = LoanMath.calculateTotalLoanRepaymentAmount(_loanAmount, _interestRate, PLATFORM_FEE_RATE, _duration);
        remainingCollateralAmount = _collateralAmount;
        loan.collateral = CollateralData(_collateralAddress, _collateralAmount, _collateralPriceInETH, _ltv, CollateralStatus.WAITING);
        // later this will be filled when borrower accepts the loan
    }

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
        
        // We check the latest price of the collateral using the oracle
        // Here we need to change CollateralAddress to String
        /**
        *    We need to use the string
        */
        // Before we send address for price we need to convert it into the string

        //string memory contractAddress = toString(loan.collateral.collateralAddress);
        // We make the price call and then we check the price using .price () method
        //price.update.value(msg.value)(contractAddress);
        // what is msg.value?
        
        // this would need to be called after price is fed!
        //loan.collateral.collateralPrice = price.price();
        
        ERC20.transferFrom(msg.sender, address(this), loan.collateral.collateralAmount);

        emit CollateralTransferToLoanSuccessful(msg.sender, loan.collateral.collateralAmount, loan.collateral.collateralPrice);

        // contract will also be transferring funds to borrower (only in case of loan offer)
         if(prevStatus == LoanStatus.FUNDED)
        {
        address(uint160(loan.borrower)).transfer(loan.loanAmount);
        emit FundTransferToBorrowerSuccessful(loan.borrower, loan.loanAmount);
        loan.startedOn = now;
        loan.loanStatus = LoanStatus.ACTIVE;
        emit LoanStarted(loan.startedOn);
        // We monitor this event and block time it was fired. every duration interval apart, we call function to make a call for potentially failed repayments
        }
    }

    function acceptLoanOffer(uint256 _interestRate, address _collateralAddress, uint256 _collateralAmount, uint256 _collateralPriceInETH, uint256 _ltv) public {

        require(loan.loanStatus == LoanStatus.FUNDED, "Incorrect loan status");
        loan.borrower = msg.sender;
        /* This will call setters and enrich loan data */
        enrichLoan(_interestRate,_collateralAddress,_collateralAmount, _collateralPriceInETH,_ltv);

        // borrower should transfer collateral after this. use same above method? YES (validation done)
        // to be done in UI
    }

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


    function getLoanData() view public returns (
        uint256 _loanAmount, uint128 _duration, uint256 _interest, string memory _acceptedCollateralsMetadata, uint256 startedOn, LoanStatus _loanStatus,
        address _collateralAddress, uint256 _collateralAmount, uint256 _collateralPrice, uint256 _ltv, CollateralStatus _collateralStatus,
        uint256 _remainingCollateralAmount,
        address _borrower, address _lender) {

        return (loan.loanAmount, loan.duration, loan.interestRate, loan.acceptedCollateralsMetadata, loan.startedOn, loan.loanStatus, loan.collateral.collateralAddress, loan.collateral.collateralAmount, loan.collateral.collateralPrice, loan.collateral.ltv, loan.collateral.collateralStatus, remainingCollateralAmount, loan.borrower, loan.lender);
    }

    /* function getPaidRepaymentsCount() view public returns (uint256) {
      return loan.repayments.length;
    } */

    /* function getAllPaidRepayments() view public returns(uint256[] memory){
      return loan.repayments;
    } */

    function getCurrentRepaymentNumber() view public returns(uint256) {
      return LoanMath.getRepaymentNumber(loan.startedOn, loan.duration);
    }

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

    // this func to be called when any repayment due date is passed
    
    
    // based on nth duration it is triggered we pass repayment number from UI
    function makeFailedRepayments(uint256 _repaymentNumberMissed) public OnlyAdmin {
    
    // UI checks if anytime now > due date of repayment n
    //uint256 totalLoanRepayments = LoanMath.getTotalNumberOfRepayments(loan.duration);
    // can be done in UI
    //this is not handled properly
    
    uint256 repaymentNumber = _repaymentNumberMissed;
    
   // cheks if repayment n was added in paid repayments array
        require(loan.repayments[repaymentNumber] == false,"repayment was already paid");
    
        // initates transfer according to repayment amount and current value of collateral1
        (uint256 _repayAmount,uint256 interest,uint256 fees) = getRepaymentAmount(repaymentNumber);
         uint256 collateralAmountToTrasnfer = LoanMath.calculateCollateralAmountToDeduct((_repayAmount.sub(fees)).mul(SOME_THINGS.div(100)), loan.collateral.collateralPrice);
         ERC20 = IERC20(loan.collateral.collateralAddress);
         ERC20.transfer(loan.lender, collateralAmountToTrasnfer);
         emit CollateralSentToLenderForDefaultedRepayment(repaymentNumber,loan.lender,collateralAmountToTrasnfer);
  }


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

    function transferToWallet1(uint256 fees) private {
        address(uint160(WALLET_1)).transfer(fees);
    }

    function transferCollateralToWallet1 (uint256 fees) private {
        uint256 feesInCollateralAmount = LoanMath.calculateCollateralAmountToDeduct(fees, loan.collateral.collateralPrice);
        ERC20 = IERC20(loan.collateral.collateralAddress);
        ERC20.transfer(WALLET_1, feesInCollateralAmount);
    }


  /// I will update this one after take on outstanding amount and discussion with lloyd
 /*   function returnCollateralToBorrower() public OnlyBorrower {

        require(now > loan.startedOn + loan.duration * 1 minutes, "Loan Still Active");
        require(loan.collateral.collateralStatus != CollateralStatus.RETURNED, "Collateral Already Returned");

        ERC20 = IERC20(loan.collateral.collateralAddress);
    /// I will update this one after take on outstanding amount and discussion with lloyd
        uint256 collateralAmountToDeduct = LoanMath.calculateCollateralAmountToDeduct(loan.outstandingAmount, loan.collateral.collateralPrice);

        loan.collateral.collateralStatus = CollateralStatus.RETURNED;

        remainingCollateralAmount = collateralAmountToDeduct;

        ERC20.transfer(msg.sender, loan.collateral.collateralAmount.sub(collateralAmountToDeduct));

        emit CollateralTransferReturnedToBorrower(msg.sender, loan.collateral.collateralAmount.sub(collateralAmountToDeduct));
  /// I will update this one after take on outstanding amount and discussion with lloyd
    } */

/*    function claimCollateralByLender() public OnlyLender {

        require(now > loan.startedOn + loan.duration * 1 minutes, "Loan Still Active");
        require(loan.loanStatus != LoanStatus.DEFAULT, "Collateral Claimed Already");

        if(loan.outstandingAmount > 0) {

            uint256 collateralAmountToTransfer = LoanMath.calculateCollateralAmountToDeduct(loan.outstandingAmount, loan.collateral.collateralPrice);
            // at any of this point price has to be fed by oracle

            remainingCollateralAmount = remainingCollateralAmount.sub(collateralAmountToTransfer);

            loan.loanStatus = LoanStatus.DEFAULT;

            ERC20 = IERC20(loan.collateral.collateralAddress);

            ERC20.transfer(msg.sender, collateralAmountToTransfer);

            emit CollateralClaimedByLender(msg.sender, collateralAmountToTransfer);
        }
    } */

}
```

LoanCreator.sol

```
pragma solidity ^0.5.0;

import "openzeppelin-solidity/contracts/ownership/Ownable.sol";
import "openzeppelin-solidity/contracts/lifecycle/Pausable.sol";
import "./LoanContract.sol";

contract LoanCreator is Ownable, Pausable {

  address[] public loans;


 constructor() public {

 }

 event LoanOfferCreated(address, address);
 event LoanRequestCreated(address, address);

 // parameters and attributes will change depending on changes in loancontract
 function createNewLoanOffer(uint256 _loanAmount, uint128 _duration, string memory _acceptedCollateralsMetadata) public returns(address _loanContractAddress) {

         _loanContractAddress = address (new LoanContract(_loanAmount, _duration, _acceptedCollateralsMetadata, 0, address(0), 0, 0, 0, address(0), msg.sender, LoanContract.LoanStatus.OFFER));

         loans.push(_loanContractAddress);

         emit LoanOfferCreated(msg.sender, _loanContractAddress);

         return _loanContractAddress;
 }

 function createNewLoanRequest(uint256 _loanAmount, uint128 _duration, uint256 _interest, address _collateralAddress, uint256 _collateralAmount, uint256 _collateralPriceInETH)
 public returns(address _loanContractAddress) {

         _loanContractAddress = address (new LoanContract(_loanAmount, _duration, "", _interest, _collateralAddress, _collateralAmount, _collateralPriceInETH, 50, msg.sender, address(0), LoanContract.LoanStatus.REQUEST));

         loans.push(_loanContractAddress);

         emit LoanRequestCreated(msg.sender, _loanContractAddress);

         return _loanContractAddress;
 }

 function getAllLoans() public view returns(address[] memory){
     return loans;
 }

}
```

## Compile Contracts with Truffle

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

## Run Migrations

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

## Smart contract interaction

Loan request
```
const LoanBook = web3.eth.contract(LoanBookABI).at(LoanBookAddress);
LoanBook.createNewLoanRequest(params);
```

Loan Offer
```
const LoanBook = web3.eth.contract(LoanBookABI).at(LoanBookAddress);
LoanBook.createNewLoanOffer(params);
```

Get All loan data
```
const LoanBook = web3.eth.contract(LoanBookABI).at(LoanBookAddress);
LoanBook.getAllLoans(params);
})
``` 

Get collateral price
```
const LoanBook = web3.eth.contract(LoanBookABI).at(LoanBookAddress);
LoanBook.getCollateralPrice(params.collateralAddress);
```

Standard Token
```
const ERC20 = web3.eth.contract(StandardTokenABI).at(params.ERC20Token);
ERC20.approve(params.loanContractAddress, web3.toWei(params.tokenAmount));
```

# Conclusion

Now you know about creating a Lending Marketplace with Trufflesuite and ReactJS on the Avalanche network.

If you had any difficulties following this tutorial or simply want to discuss Avalanche tech with us you can [**join our community today**](https://community.figment.io/) or [**Join our discord channel**](https://discord.gg/fszyM7K)!

# About the author

[Devendra Yadav](https://community.figment.io/u/dev.koold)

# References
- https://learn.figment.io/tutorials/create-a-local-test-network
- https://learn.figment.io/tutorials/using-truffle-with-the-avalanche-c-chain
- https://github.com/crypto-lend
