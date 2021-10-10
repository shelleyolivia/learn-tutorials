# Create a Lending Marketplace dapp on Polygon with Truffle Suite.

## Introduction

A Lending Marketplace provides a secure, flexible, open-source foundation for a decentralized loan marketplace on the Polygon blockchain. It provides the pieces necessary to create a decentralized lending exchange, including the requisite lending assets, repayments, and collateral infrastructure, enabling third parties to build applications for lending.

## Prerequisites

[MetaMask](https://metamask.io/) is a browser-based blockchain wallet that can be used to store any kind of digital assets and cryptocurrency.

[Polygon](https://docs.polygon.technology/) is a blockchain that is EVM compatible.

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

***Add the smart contract code below:***

***LoanContract.sol***

```
// Define the solidity compiler version
pragma solidity ^0.5.0;

// Import the required smart contracts and libraries
import "openzeppelin-solidity/contracts/ownership/Ownable.sol";
import "openzeppelin-solidity/contracts/lifecycle/Pausable.sol";
import "openzeppelin-solidity/contracts/token/ERC20/IERC20.sol";
import "./libs/LoanMath.sol";
import "./External/PriceFeeder.sol";
import "./libs/String.sol";

// Create a contract named LoanContract which defines the loan methods
contract LoanContract {

// Define the variables and contants

    using SafeMath for uint256;

// Exchange rate between two selected tokens
    uint256 constant PLATFORM_FEE_RATE;

// First account address for loan creation
    address constant WALLET_1 = "YOUR FIRST ACCOUNT ADDRESS";

// Address of owner for smart contract deployment
    address admin = "OWNER ADDRESS";
    
    uint256 constant SOME_THINGS = 105;
    
// Predefined values for Loan Status
    enum LoanStatus {
        OFFER,
        REQUEST,
        ACTIVE,
        FUNDED,
        REPAID,
        DEFAULT
    }
    
// Predefined values for Collateral Status
    enum CollateralStatus {
        WAITING,
        ARRIVED,
        RETURNED,
        DEFAULT
    }

// Structure for Collateral Data
    struct CollateralData {
        address collateralAddress;
        uint256 collateralAmount;
        uint256 collateralPrice; // will have to subscribe to oracle
        uint256 ltv;
        CollateralStatus collateralStatus;
    }

// Structure for Loan Data
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
    
// Method for loan enrichment
    function enrichLoan(uint256 _interestRate, address _collateralAddress, uint256 _collateralAmount, uint256 _collateralPriceInETH, uint256 _ltv) public {
        loan.interestRate = _interestRate;
        loan.collateral.collateralAddress = _collateralAddress;
        loan.collateral.collateralPrice = _collateralPriceInETH;
        loan.collateral.collateralAmount = _collateralAmount;
        loan.collateral.collateralStatus = CollateralStatus.WAITING;
        loan.collateral.ltv = _ltv;
        emit LoanContractUpdated(_interestRate, _collateralAddress, _collateralPriceInETH, _collateralAmount, _ltv);
    }

// Loan Data object
    LoanData loan;

// Declare a public ERC20 token 
    IERC20 public ERC20;

// Initialize the remaining collateral ammount variable
    uint256 public remainingCollateralAmount = 0;

// Events for different loan and collateral status
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
    
    // Modifiers for Borrower, Admin and Lender

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



    // Watch for this event LoanStartedOn during two transactions approveLoanRequest & transferCollateralToLoan

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

    // After loan offer created transfer Funds to Loan
    function transferFundsToLoan() public payable OnlyLender {
         require(msg.value >= loan.loanAmount, "Sufficient funds not transferred");
    // Status changed OFFER -> FUNDED
          loan.loanStatus = LoanStatus.FUNDED;

         emit FundTransferToLoanSuccessful(msg.sender, msg.value);
    }
    
    // Method to convert address to string
    function toString(address x) public returns (string memory) {
        bytes memory b = new bytes(20);
        for (uint i = 0; i < 20; i++)
            b[i] = byte(uint8(uint(x) / (2**(8*(19 - i)))));
        return string(b);
    }

    // After loan request created transfer Collateral to Loan
    function transferCollateralToLoan() payable public OnlyBorrower  {
        
        // Define the ERC20 token
        ERC20 = IERC20(loan.collateral.collateralAddress);
        
        // Loan status updated
        LoanStatus prevStatus = loan.loanStatus;

        // Emit event if collateral transfer is failed
        if(loan.collateral.collateralAmount > ERC20.allowance(msg.sender, address(this))) {
            emit CollateralTransferToLoanFailed(msg.sender, loan.collateral.collateralAmount);
            revert();
        }

        // Loan Collateral Status updated
        loan.collateral.collateralStatus = CollateralStatus.ARRIVED;

        // Transfer the ERC20 token
        ERC20.transferFrom(msg.sender, address(this), loan.collateral.collateralAmount);

        // Event CollateralTransferToLoanSuccessful is emitted
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

    // Method for accepting Loan Offer
    function acceptLoanOffer(uint256 _interestRate, address _collateralAddress, uint256 _collateralAmount, uint256 _collateralPriceInETH, uint256 _ltv) public {

        require(loan.loanStatus == LoanStatus.FUNDED, "Incorrect loan status");
        loan.borrower = msg.sender;
        /* This will call setters and enrich loan data */
        enrichLoan(_interestRate,_collateralAddress,_collateralAmount, _collateralPriceInETH,_ltv);

        // borrower should transfer collateral after this. use same above method? YES (validation done)
        // to be done in UI
    }

    // Method for approving Loan Request
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

    // Method for getting Loan Data
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

    // Method for getting Current Repayment Number
    function getCurrentRepaymentNumber() view public returns(uint256) {
      return LoanMath.getRepaymentNumber(loan.startedOn, loan.duration);
    }

    // Method for getting Repayment amount
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

    // this method to be called when any repayment due date is passed


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

    // Method for Repayment of loan
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

    // Method for transferring fees to Wallet1 address
    function transferToWallet1(uint256 fees) private {
        address(uint160(WALLET_1)).transfer(fees);
    }

    // Method for transferring Collateral to Wallet1
    function transferCollateralToWallet1 (uint256 fees) private {
        uint256 feesInCollateralAmount = LoanMath.calculateCollateralAmountToDeduct(fees, loan.collateral.collateralPrice);
        ERC20 = IERC20(loan.collateral.collateralAddress);
        ERC20.transfer(WALLET_1, feesInCollateralAmount);
    }

    // Method for returning Collateral to Borrower
    function returnCollateralToBorrower() public OnlyBorrower {

        require(now > loan.startedOn + loan.duration * 1 minutes, "Loan Still Active");
        require(loan.collateral.collateralStatus != CollateralStatus.RETURNED, "Collateral Already Returned");

        ERC20 = IERC20(loan.collateral.collateralAddress);
        
        uint256 collateralAmountToDeduct = LoanMath.calculateCollateralAmountToDeduct(loan.outstandingAmount, loan.collateral.collateralPrice);

        loan.collateral.collateralStatus = CollateralStatus.RETURNED;

        remainingCollateralAmount = collateralAmountToDeduct;

        ERC20.transfer(msg.sender, loan.collateral.collateralAmount.sub(collateralAmountToDeduct));

        emit CollateralTransferReturnedToBorrower(msg.sender, loan.collateral.collateralAmount.sub(collateralAmountToDeduct));
    }

    // Method for claiming Collateral by Lender 
    function claimCollateralByLender() public OnlyLender {

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
    }

}
```

***LoanCreator.sol***

```
// Define solidity compiler version
pragma solidity ^0.5.0;

// Import the required smart contracts
import "openzeppelin-solidity/contracts/ownership/Ownable.sol";
import "openzeppelin-solidity/contracts/lifecycle/Pausable.sol";
import "./LoanContract.sol";

// Create a contract named LoanCreator
contract LoanCreator is Ownable, Pausable {

  address[] public loans;


 constructor() public {

 }

// Emit the Loan offer/request on creation
 event LoanOfferCreated(address, address);
 event LoanRequestCreated(address, address);

 // parameters and attributes will change depending on changes in loancontract
 function createNewLoanOffer(uint256 _loanAmount, uint128 _duration, string memory _acceptedCollateralsMetadata) public returns(address _loanContractAddress) {

         _loanContractAddress = address (new LoanContract(_loanAmount, _duration, _acceptedCollateralsMetadata, 0, address(0), 0, 0, 0, address(0), msg.sender, LoanContract.LoanStatus.OFFER));

         loans.push(_loanContractAddress);

         emit LoanOfferCreated(msg.sender, _loanContractAddress);

         return _loanContractAddress;
 }

// Method for creating a new Loan Request
 function createNewLoanRequest(uint256 _loanAmount, uint128 _duration, uint256 _interest, address _collateralAddress, uint256 _collateralAmount, uint256 _collateralPriceInETH)
 public returns(address _loanContractAddress) {

         _loanContractAddress = address (new LoanContract(_loanAmount, _duration, "", _interest, _collateralAddress, _collateralAmount, _collateralPriceInETH, 50, msg.sender, address(0), LoanContract.LoanStatus.REQUEST));

         loans.push(_loanContractAddress);

         emit LoanRequestCreated(msg.sender, _loanContractAddress);

         return _loanContractAddress;
 }

// Method for getting all the loan addresses
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

# Building UI with ReactJS

Create a react app project:
```
npx create-react-app lending-ui
```
Browse the project directory:
```
cd lending-ui
```

Install Web3 and setup methods inside services directory:
```
npm i web3
```

***Web3Service.js***
```
// Import the lodash 'isEmpty'
import isEmpty from 'lodash/isEmpty';

// Create fetchNetwork() method to get the current network
export const fetchNetwork = () => {
  return new Promise((resolve, reject) => {
  
    // Initialize web3 from `window.web3`
    const { web3 } = window;

    web3 && web3.version && web3.version.getNetwork((err, netId) => {
      if (err) {
        reject(err);
      } else {
        resolve(netId);
      }
    });
  });
};

// Create fetchAccounts() to get the account addresses
export const fetchAccounts = () => {
  return new Promise((resolve, reject) => {
  
    // Initialize web3 from `window.web3`
    const { web3 } = window;
    
    // Initialize constant ethAccounts
    const ethAccounts = getAccounts();

    // Check if ethAccounts is empty
    if (isEmpty(ethAccounts)) {
      web3 && web3.eth && web3.eth.getAccounts((err, accounts) => {
        if (err) {
          reject(err);
        } else {
          resolve(accounts);
        }
      });
    } else {
      resolve(ethAccounts);
    }
  });
};

// Create getAccounts() method to retreive the selected accouunt
export function getAccounts() {
  try {
  
    // Initialize web3 from `window.web3`
    const { web3 } = window;
    
    // throws if no account selected
    return web3.eth.accounts;
  } catch (e) {
    return [];
  }
}

// Create fetchMinedTransactionReceipt method getting the transaction receipt after the is successfull
export const fetchMinedTransactionReceipt = (transactionHash) => {

  return new Promise((resolve, reject) => {

    // Initialize web3 from `window.web3`
    const { web3 } = window;

    // Timer to check for transaction receipt every 2 seconds
    var timer = setInterval(()=> {
      web3.eth.getTransactionReceipt(transactionHash, (err, receipt)=> {
        if(!err && receipt){
          clearInterval(timer);
          resolve(receipt);
        }
      });
    }, 2000)

  })
}
```

***Now, setup `token.js`:***

```
// Import the required services and ABIs
import { fetchMinedTransactionReceipt } from './Web3Service';
import { StandardTokenABI } from '../components/Web3/abi';

// Create an ERC20 token approval method for collateral transfer and loan repayments
export const ExecuteTokenApproval = (params) => {

    return new Promise((resolve, reject) => {

        // Initialize web3 from `window.web3`
        const { web3 } = window;
        
        // Initialize the ERC20 constant by passing token ABI and address.
        const ERC20 = web3.eth.contract(StandardTokenABI).at(params.ERC20Token);

        // Call the `approve()` method of the selected ERC20 token
        ERC20.approve(params.loanContractAddress, params.tokenAmount,{
            from: web3.eth.accounts[0]
            }, async (err, transactionHash) => {
                if(!err){
                    console.log(transactionHash);
                    const receipt = await fetchMinedTransactionReceipt(transactionHash);
                    resolve(receipt);
                } else {
                    reject(err);
                }
          });
    });
}
```

***Create Loan Contract methods inside `loanContract.js`:***

```
// Import the required services and ABIs
import { fetchMinedTransactionReceipt } from './Web3Service';
import { LoanContractABI } from '../components/Web3/abi';

// Create a `GetLoanDetails()` method to fetch the selected loan details by passing the `loanContractAddress`
export const GetLoanDetails = (loanContractAddress) => {

    return new Promise((resolve, reject) => {

        // Initialize web3 from `window.web3`
        const { web3 } = window;
        
        // Initialize the LoanContract constant by passing loan ABI and address.
        const LoanContract = web3.eth.contract(LoanContractABI).at(loanContractAddress);

        LoanContract.getLoanData((err, loan) => {
            if(!err){
                resolve(loan);
            } else {
                reject(err);
            }
        });
    })
}

// Create `FinalizeCollateralTransfer()` method for final transfer of collateral to selected loan
export const FinalizeCollateralTransfer = (loanContractAddress, collateralAddress) => {

    return new Promise((resolve, reject) => {

        // Initialize web3 from `window.web3`
        const { web3 } = window;

        // Initialize the LoanContract constant by passing loan ABI and address.
        const LoanContract = web3.eth.contract(LoanContractABI).at(loanContractAddress);

        // Call `transferCollateralToLoan()` method from LoanContract and return Transaction Receipt
        LoanContract.transferCollateralToLoan(collateralAddress, {
            from: web3.eth.accounts[0]
            }, async (err, transactionHash) => {
                if(!err){
                    console.log(transactionHash);
                    const receipt = await fetchMinedTransactionReceipt(transactionHash);
                    resolve(receipt);
                } else {
                    reject(err);
                }
            });
    })
}

// Create `AcceptLoanOffer()` for accepting the loan offer by borrower
export const AcceptLoanOffer = (loanContractAddress) => {

    return new Promise((resolve, reject) => {

        // Initialize web3 from `window.web3`
        const { web3 } = window;
        
        // Initialize the LoanContract constant by passing loan ABI and address.
        const LoanContract = web3.eth.contract(LoanContractABI).at(loanContractAddress);

        // Call `acceptLoanOffer()` from LoanContract and return Transaction Receipt
        LoanContract.acceptLoanOffer({
            from: web3.eth.accounts[0]
            }, async (err, transactionHash) => {
                if(!err){
                    console.log(transactionHash);
                    const receipt = await fetchMinedTransactionReceipt(transactionHash);
                    resolve(receipt);
                } else {
                    reject(err);
                }
            });
    })
}

// Create `TransferFundsToLoanContract()` method so the lender can transfer funds to a loan
export const TransferFundsToLoanContract = (loanContractAddress, loanAmount) => {

    return new Promise((resolve, reject) => {
    
        // Initialize web3 from `window.web3`
        const { web3 } = window;

        // Initialize the LoanContract constant by passing loan ABI and address.
        const LoanContract = web3.eth.contract(LoanContractABI).at(loanContractAddress);

        // Call `transferFundsToLoanContract()` method from LoanContract and return the transaction receipt
        LoanContract.transferFundsToLoanContract({
            from: web3.eth.accounts[0],
            value: web3.toWei(loanAmount)
            }, async (err, transactionHash) => {
                if(!err){
                    console.log(transactionHash);
                    const receipt = await fetchMinedTransactionReceipt(transactionHash);
                    resolve(receipt);
                } else {
                    reject(err);
                }
            });
    })
}

// Create `ApproveAndFundLoanRequest()` method so Lender can approve and fund the loan request
export const ApproveAndFundLoanRequest = (loanContractAddress, loanAmount) => {

    return new Promise((resolve, reject) => {

        // Initialize web3 from `window.web3`
        const { web3 } = window;

        // Initialize the LoanContract constant by passing loan ABI and address.
        const LoanContract = web3.eth.contract(LoanContractABI).at(loanContractAddress);

        // Call `approveLoanRequest()` from LoanContract
        LoanContract.approveLoanRequest({
            from: web3.eth.accounts[0],
            value: web3.toWei(loanAmount)
            }, async (err, transactionHash) => {
                if(!err){
                    console.log(transactionHash);
                    const receipt = await fetchMinedTransactionReceipt(transactionHash);
                    resolve(receipt);
                } else {
                    reject(err);
                }
            });
    })
}

  // Create `GetRepaymentData()` method to retreive the selected repayment with repaymentNumber and loanContractAddress
  export const GetRepaymentData = (loanContractAddress, repaymentNumber) => {

    return new Promise((resolve, reject) => {

        // Initialize web3 from `window.web3`
        const { web3 } = window;

        // Initialize the LoanContract constant by passing loan ABI and address.
        const LoanContract = web3.eth.contract(LoanContractABI).at(loanContractAddress);
        
        // Call `getRepaymentAmount()` method from LoanContract to get the repayment amount
        LoanContract.getRepaymentAmount((err, repaymentData) => {
            if(!err){
            
                // Call `checkRepaymentStatus()` from LoanContract to check repayment status and return loan repayment details
                LoanContract.checkRepaymentStatus(repaymentNumber, (err, repayee) => {
                    if(!err){
                        resolve({
                            loanContractAddress: loanContractAddress,
                            repaymentNumber: repaymentNumber,
                            repayee: repayee,
                            totalRepaymentAmount: web3.fromWei(repaymentData[0].toNumber()),
                            monthlyInterest: web3.fromWei(repaymentData[1].toNumber())
                        });
                    } else {
                        reject(err);
                    }
                });
            } else {
                reject(err);
            }
        });

    })
}

// Create `RepayLoan()` method for selected repayment of a loan
export const RepayLoan = (loanContractAddress, repaymentAmount) => {

    return new Promise((resolve, reject) => {

        // Initialize web3 from `window.web3`
        const { web3 } = window;

        // Initialize the LoanContract constant by passing loan ABI and address.
        const LoanContract = web3.eth.contract(LoanContractABI).at(loanContractAddress);

        LoanContract.repayLoan({
            from: web3.eth.accounts[0],
            value: web3.toWei(repaymentAmount)
            }, async (err, transactionHash) => {
                if(!err){
                    console.log(transactionHash);
                    const receipt = await fetchMinedTransactionReceipt(transactionHash);
                    resolve(receipt);
                } else {
                    reject(err);
                }
            });
    })
}

// Create `LiquidateLoanCollateral()` method
export const LiquidateLoanCollateral = (loanContractAddress) => {

    return new Promise((resolve, reject) => {

        // Initialize web3 from `window.web3`
        const { web3 } = window;

        // Initialize the LoanContract constant by passing loan ABI and address.
        const LoanContract = web3.eth.contract(LoanContractABI).at(loanContractAddress);

        LoanContract.liquidateCollateral({
            from: web3.eth.accounts[0]
            }, async (err, transactionHash) => {
                if(!err){
                    console.log(transactionHash);
                    const receipt = await fetchMinedTransactionReceipt(transactionHash);
                    resolve(receipt);
                } else {
                    reject(err);
                }
            });
    })
}

// Create `ClaimCollateralByBorrower()` method
export const ClaimCollateralByBorrower = (loanContractAddress) => {

    return new Promise((resolve, reject) => {

        // Initialize web3 from `window.web3`
        const { web3 } = window;

        const LoanContract = web3.eth.contract(LoanContractABI).at(loanContractAddress);

        // Initialize the LoanContract constant by passing loan ABI and address.
        LoanContract.returnCollateralToBorrower({
            from: web3.eth.accounts[0]
            }, async (err, transactionHash) => {
                if(!err){
                    console.log(transactionHash);
                    const receipt = await fetchMinedTransactionReceipt(transactionHash);
                    resolve(receipt);
                } else {
                    reject(err);
                }
            });
    })
}

// Create `ClaimCollateralByLender()` method
export const ClaimCollateralByLender = (loanContractAddress, repaymentNumber) => {

    return new Promise((resolve, reject) => {

        // Initialize web3 from `window.web3`
        const { web3 } = window;

        const LoanContract = web3.eth.contract(LoanContractABI).at(loanContractAddress);
        
        // Initialize the LoanContract constant by passing loan ABI and address.
        LoanContract.claimCollateralOnDefault(repaymentNumber, {
            from: web3.eth.accounts[0]
            }, async (err, transactionHash) => {
                if(!err){
                    console.log(transactionHash);
                    const receipt = await fetchMinedTransactionReceipt(transactionHash);
                    resolve(receipt);
                } else {
                    reject(err);
                }
            });
    })
}
```

***Setup Loan Book methods inside `loanbook.js`:***
```
// Import the utils, services and ABIs
import { padLeft } from 'web3-utils';
import { fetchMinedTransactionReceipt } from './Web3Service';
import { LoanBookABI, LoanBookAddress } from '../components/Web3/abi';

// Create `GetLoans()` method
export const GetLoans = () => {
    return new Promise((resolve, reject) => {

        // Initialize web3 from `window.web3`
        const { web3 } = window;

        const LoanBook = web3.eth.contract(LoanBookABI).at(LoanBookAddress);

        LoanBook.getAllLoans((err, loans) => {
            if(!err){
                console.log(loans);
                resolve(loans);
            } else {
                reject(err);
            }
        });
    })
}

// Create `CreateNewLoanRequest()` method
export const CreateNewLoanRequest = (params) => {

    return new Promise((resolve, reject) => {

        // Initialize web3 from `window.web3`
        const { web3 } = window;

        const LoanBook = web3.eth.contract(LoanBookABI).at(LoanBookAddress);

        LoanBook.createNewLoanRequest(web3.toWei(params.principal), params.duration,
            [[web3.toHex(params.collateralAddress), web3.toHex(0), padLeft(web3.toHex(params.interest),64)]],{
            from: web3.eth.accounts[0]
            },async(err, transactionHash) => {
                if(!err){
                    console.log(transactionHash);
                    const receipt = await fetchMinedTransactionReceipt(transactionHash);
                    resolve(padLeft(web3.toHex(web3.toBigNumber(receipt.logs[0].topics[2])), 32));
                } else {
                    reject(err);
                }
          });
    });
}

// Create `CreateNewLoanOffer()` method
export const CreateNewLoanOffer = (params) => {

    return new Promise((resolve, reject) => {

        // Initialize web3 from `window.web3`
        const { web3 } = window;

        const LoanBook = web3.eth.contract(LoanBookABI).at(LoanBookAddress);

        let collateralsMetadata = new Array();

        for(var i in params.collaterals){
            collateralsMetadata[i] =
                [ web3.toHex(params.collaterals[i].collateral),
                    padLeft(web3.toHex(params.collaterals[i].ltv),64),
                    padLeft(web3.toHex(params.collaterals[i].mpr),64)];
        }

        // Call `createNewLoanOffer()` method from LoanBook
        LoanBook.createNewLoanOffer(web3.toWei(params.principal), params.duration, collateralsMetadata, {
            from: web3.eth.accounts[0]
          }, async(err, transactionHash) => {
            if(!err){
                console.log(transactionHash);
                const receipt = await fetchMinedTransactionReceipt(transactionHash);
                // console.log(receipt.logs[0].topics[2])
                resolve(padLeft(web3.toHex(web3.toBigNumber(receipt.logs[0].topics[2])), 32));
            } else {
                reject(err);
            }
          });
    });
}

// Create `FetchCollateralPrice()` method to retreive the latest price
export const FetchCollateralPrice = (params) => {

    return new Promise((resolve, reject) => {
        
        // Initialize web3 from `window.web3`
        const { web3 } = window;

        // Initialize the LoanBook constant by passing loanbook ABI and address.
        const LoanBook = web3.eth.contract(LoanBookABI).at(LoanBookAddress);
         
        // Call `getCollateralPrice()` from LoanBook
        LoanBook.getCollateralPrice(params.collateralAddress, (err, price) => {
            if(!err){
                resolve(web3.fromWei(price.toNumber()));
            } else {
                reject(err);
            }
        });
    })

}
```

***Setup `App.js` in the following manner and divide it into components:***

```
import React from 'react';
import './assets/vendor/font-awesome/css/font-awesome.css';
import './assets/vendor/nucleo/css/nucleo.css';
import './assets/css/argon.min.css';
import './App.css';

import { Provider } from 'react-redux';

import { createAppStore } from './components/state/stores/AppStore';

import { AppRouter } from './components/routers/AppRouter';

import { Web3Provider } from './components/Web3/Web3Provider';

const App = () => (
  <Provider store={createAppStore()}>
    <div style={{ backgroundColor: 'white' }}>
      <Web3Provider />
      <AppRouter />
    </div>
  </Provider>
);
export default App;
```

***Create `components/` directory and setup separate UI components:***

***LoanRequest component***
```
// Import the required libraries and components
import React, { Component } from 'react';
import { Link } from 'react-router';
import Loader from 'react-loader';
import Header from '../pages/Header';
import { CreateNewLoanRequest, FetchCollateralPrice } from '../../services/loanbook';
import { ExecuteTokenApproval } from '../../services/token';
import { FinalizeCollateralTransfer } from '../../services/loanContract';
import { supported_erc20_token, getTokenBySymbol, getTokenByAddress } from '../Web3/erc20';


// Create a loan request class component extending React.Component
class LoanRequest extends Component {

  // Create a contructor and call `super()` method of parent class
  constructor() {
    super();

    // Initialize the state variables inside the constructor
    this.state = {
      collateral: true,
      loan: false,
      currency: false,
      borrow: false,
      durationView: false,
      mprView: false,
      monthlyInterest: false,
      borrowLess: false,
      loaded: true,
      alertLoanAmount: false,
      createRequestAlert: false,
      allowCreateRequest: true,
      approveRequestAlert: false,
      transferCollateralAlert: false,
      transferCollateralSuccessAlert: false,
      transferCollateralFailAlert: false,
      loanContractAddress: '',
      ropstenTransactionhash: '',
      collateralValue: 0,
      loanAmount: null,
      duration: null,
      monthlyInt: 0,
      durationArr: [30, 60, 90, 120, 150, 180, 210, 240, 270, 300, 330, 360],
      durationStart: 0,
      durationEnd: 360,
      totalPremium: null,
      monthlyInstallment: null,
      loanAmountInput: '',
      apr: 0,
      originationFee: 1,
      collateralCurrency: 'ETH',
      erc20_tokens: supported_erc20_token
    };
  }

  // Define `createLoanRequest()` method
  createLoanRequest = async (principal, duration, interest, collateralAddress, collateralAmount, priceInEth) => {
    try {
      // Pass the required params as an object in thhe `CreateNewLoanRequest()` method
      const loanContractAddress = await CreateNewLoanRequest({
        principal: principal,
        duration: duration,
        interest: interest,
        collateralAddress: collateralAddress,
        collateralAmount: collateralAmount,
        priceInEth: priceInEth
      });

      // Set the below state variables
      this.setState({
        allowCreateRequest: false,
        createRequestAlert: true,
        approveRequestAlert: true,
        loanContractAddress: loanContractAddress,
        collateralAddress: collateralAddress
        //getTokenBySymbol[collateralCurrency] && getTokenBySymbol[collateralCurrency].address
      })

    } catch (e) {
      // Log the error if any occured
      console.log(e);
    } finally {

    }
  }

  handleERC20TokenApproval = async (collateralAddress, loanContractAddress, collateralValue) => {

    try {

      await ExecuteTokenApproval({
        ERC20Token: collateralAddress,
        loanContractAddress: loanContractAddress,
        tokenAmount: collateralValue
      });

      this.setState({
        approveRequestAlert: false,
        createRequestAlert: false,
        transferCollateralAlert: true
      });
    } catch (error) {
      console.log(error);
    }
  }

  // Define `handleTransferCollateral()` method
  handleTransferCollateral = async (loanContractAddress, collateralAddress) => {

    // try-catch block to handle errors
    try {
      // call `FinalizeCollateralTransfer()` method
      await FinalizeCollateralTransfer(loanContractAddress, collateralAddress);

      // set the Collateral transfer variables responsible for UI change
      this.setState({
        transferCollateralAlert: false,
        transferCollateralSuccessAlert: true
      });

    } catch (error) {

      // set the Collateral transfer variables responsible for UI change
      this.setState({
        transferCollateralAlert: false,
        transferCollateralFailAlert: true
      });
    }
  }

  // Define `handleMonthlyInterest()` method 
  handleMonthlyInterest = (e) => {

    // Assign the state variables to local variables
    const { loanAmount, monthlyInt, duration, totalPremium, monthlyInstallment, apr, originationFee } = this.state;

    // conditions for increment of monthly interest value
    if (e.target.value == 'plus' && monthlyInt < 5) {
      this.setState({ monthlyInt: monthlyInt + 0.25 });
      let totalRepayment = ((loanAmount * (monthlyInt + 0.25) * ((duration / 30) + 1)) / (2 * 100))
      this.setState({ monthlyInstallment: (((loanAmount * (monthlyInt + 0.25) * ((duration / 30) + 1)) / (2 * duration / 30 * 100)) + loanAmount / (duration / 30)) })
      this.setState({ totalPremium: totalRepayment });
      this.setState({ apr: (totalRepayment / loanAmount) * 100 })
      console.log("apr : ", apr, totalRepayment);
    }

    // conditions for decrement of monthly interest value
    else if (e.target.value == 'minus' && monthlyInt > 0) {
      this.setState({ monthlyInt: monthlyInt - 0.25 });
      let totalRepayment = ((loanAmount * (monthlyInt - 0.25) * ((duration / 30) + 1)) / (2 * 100))
      this.setState({ monthlyInstallment: (((loanAmount * (monthlyInt - 0.25) * ((duration / 30) + 1)) / (2 * duration / 30 * 100)) + loanAmount / (duration / 30)) })
      this.setState({ totalPremium: totalRepayment });
      this.setState({ apr: (totalPremium / loanAmount) * 100 })
    }
  }

  // Define `handleCollateralConversion()` method
  handleCollateralConversion = async (collateralAddress) => {

    // Assign the state variable to local variable
    let { collateralValue } = this.state;

    try {

      const collateralPrice = await FetchCollateralPrice({
        collateralAddress: collateralAddress
      });

      /* collateralPrice here is in ETH which is divided by 2 is used assuming fix LTV of 50%
       and collateralValue is number of tokens which can be decimal value */
      this.setState({
        loanAmount: (collateralValue * collateralPrice / 2),
        currency: false,
        borrow: true
      })

    } catch (error) {
      //Log error
      console.log(error);
    }
  }

  // Call the `render()` method to display the frontend
  render() {

    // Assign the state variables to the local
    const { loanAmount, duration, monthlyInt, collateralAddress, collateralValue, collateralCurrency, collateral, erc20_tokens, loan, currency, borrow,
      durationView, durationArr, monthlyInterest, borrowLess, totalPremium, monthlyInstallment, originationFee, apr, alertLoanAmount,
      loanAmountInput, createRequestAlert, allowCreateRequest, loanContractAddress, ropstenTransactionhash, approveRequestAlert, transferCollateralAlert, transferCollateralSuccessAlert, transferCollateralFailAlert } = this.state;

    // Call the `return()` method and display the frontend code
    return (

      // Create a parent `<div>`
      <div className="LoanRequest text-center">

        {/* Call the `<Header/>` and <Loader/> components*/}
        <Header />
        <Loader loaded={this.state.loaded} />

        {/** Create New Loan Request UI */}
        <div className="position-relative">
          <section className="section-hero section-shaped my-0">
            <div className="">
              <span className="span-150"></span>
              <span className="span-50"></span>
              <span className="span-50"></span>
              <span className="span-75"></span>
              <span className="span-100"></span>
              <span className="span-75"></span>
              <span className="span-50"></span>
              <span className="span-100"></span>
              <span className="span-50"></span>
              <span className="span-100"></span>
            </div>
            <div className="container shape- d-flex align-items-center">
              <div className="col-lg-7">
                <div className="card">
                  <div className="card-header text-center">
                    <h5> New Loan Request</h5>
                  </div>

                  {/** Chose your collateral curency display only when `collateral is returned true`*/}
                  <div className="card-body" style={{ display: collateral ? 'block' : 'none' }}>
                    <div className="alert alert-primary alert-dismissible fade show" role="alert">
                      <span className="alert-text">Choose your collateral currency.</span>
                    </div>
                    <div className="row mt-5">
                      <div className="col-md-6" style={{ marginTop: '25px', marginBottom: '85px', cursor: 'pointer' }}
                        onClick={() => { this.setState({ collateral: false, loan: true }); }}>
                        <span className="btn-inner--text"><img style={{ width: '25px' }} src="/assets/img/eth.png" /></span>
                        <br />
                        <p>Ethereum</p>
                      </div>
                      <div className="col-md-4 form-group mt-3">
                        <label id="exampleFormControlSelect1">ERC20 TOKENS</label>
                        <select className="form-control" id="exampleFormControlSelect1" onClick={(e) => {
                          this.setState({ collateralCurrency: e.target.value });
                        }}>
                          {
                            erc20_tokens.map((item, i) => {
                              return <option>{item.symbol}</option>;
                            })
                          }
                        </select>
                      </div>
                    </div>
                    <div className="btn-wrapper" style={{ marginTop: '20px', cursor: 'pointer' }} onClick={() => { this.setState({ collateral: false, loan: true }) }}>
                      <a href="#" className="btn btn-primary btn-icon mb-3 mb-sm-0 m-5">
                        <span className="btn-inner--text">Next</span>
                      </a>
                    </div>
                  </div>

                  {/** Choose collateral amount UI */}
                  <div className="card-body" style={{ display: loan ? 'block' : 'none' }}>
                    <div className="alert alert-primary alert-dismissible fade show" role="alert">
                      <span className="alert-text">Insert the collateral amount.</span>
                    </div>
                    <input className="form-control form-control-lg" type="text" placeholder={collateralCurrency} onChange={(e) => { this.setState({ collateralValue: e.target.value }); }} />
                    <div className="row">
                      <div className="col-md-6" style={{ marginTop: '153px', cursor: 'pointer' }} onClick={() => { this.setState({ collateral: true, loan: false }) }}>
                        <a href="#" className="btn btn-primary btn-icon mb-3 mb-sm-0 m-5">
                          <span className="btn-inner--text">Back</span>
                        </a>
                      </div>
                      <div className="col-md-6" style={{ marginTop: '153px', cursor: 'pointer' }} onClick={() => {
                        this.setState({ loan: false, currency: true });
                      }}>
                        <a href="#" className="btn btn-primary btn-icon mb-3 mb-sm-0 m-5">
                          <span className="btn-inner--text">Next</span>
                        </a>
                      </div>
                    </div>
                  </div>

                  {/** Choose loan currency UI */}
                  <div className="card-body" style={{ display: currency ? 'block' : 'none' }}>
                    <div className="alert alert-primary alert-dismissible fade show" role="alert">
                      <span className="alert-text">Choose your loan currency.</span>
                    </div>
                    <div className="btn-wrapper" style={{ marginTop: '25px', marginBottom: '247px', cursor: 'pointer' }} onClick={() => this.handleCollateralConversion(getTokenBySymbol[collateralCurrency] && getTokenBySymbol[collateralCurrency].address)}>
                      <span className="btn-inner--text"><img style={{ width: '25px' }} src="/assets/img/eth.png" /></span>
                      <br />
                      <p>Ethereum</p>
                    </div>
                    <div className="btn-wrapper" style={{ marginTop: '20px', cursor: 'pointer' }} onClick={() => { this.setState({ currency: false, loan: true }); }}>
                      <a href="#" className="btn btn-primary btn-icon mb-3 mb-sm-0 m-5">
                        <span className="btn-inner--text">Back</span>
                      </a>
                    </div>
                  </div>
                  <div className="card-body" style={{ display: borrow ? 'block' : 'none' }}>
                    <div className="alert alert-primary alert-dismissible fade show" role="alert">
                      <span className="alert-text">Great! you can borrow :</span>
                    </div>

                    {/** Select the borrow less feature*/}
                    {borrowLess ?
                      <div>
                        <input className="form-control form-control-lg" type="number" min="1" max={loanAmount} placeholder='Enter loan amount' Value={loanAmount} onChange={(e) => {
                          if (e.target.value > loanAmount)
                            this.setState({ alertLoanAmount: true, loanAmountInput: e.target.value });
                          else
                            this.setState({ alertLoanAmount: false, loanAmountInput: e.target.value });
                        }} />
                        {alertLoanAmount && <div className="alert alert-danger alert-dismissible fade show" role="alert">
                          <span className="alert-text">Please select less than or equal to {loanAmount}</span>
                        </div>}
                      </div>
                      :
                      <div>
                        <p style={{ border: 'solid grey 1px' }}>{loanAmount} ETH</p>
                        <a className="text-right" style={{ cursor: 'pointer' }} onClick={() => { this.setState({ borrowLess: true }) }}>want to borrow less ?</a>
                      </div>
                    }
                    <div className="row">
                      <div className="col-md-6" style={{ marginTop: '153px', cursor: 'pointer' }} onClick={() => { this.setState({ currency: true, borrow: false }); }}>
                        <a href="#" className="btn btn-primary btn-icon mb-3 mb-sm-0 m-5">
                          <span className="btn-inner--text">Back</span>
                        </a>
                      </div>
                      <div className="col-md-6" style={{ marginTop: '153px', cursor: 'pointer' }}
                        onClick={() => {
                          this.setState({ durationView: true, borrow: false, loanAmount: borrowLess ? loanAmountInput : loanAmount });
                          console.log('LOANAMOUNT : ', loanAmount);

                        }}>
                        <a href="#" className="btn btn-primary btn-icon mb-3 mb-sm-0 m-5">
                          <span className="btn-inner--text">Next</span>
                        </a>
                      </div>
                    </div>
                  </div>

                  {/** Define the loan duration */}
                  <div className="card-body" style={{ display: durationView ? 'block' : 'none' }}>
                    <div className="alert alert-primary alert-dismissible fade show" role="alert">
                      <span className="alert-text">Define loan duration.</span>
                    </div>
                    <br />

                    <div className="btn-wrapper" style={{ marginTop: '85px', cursor: 'pointer' }}>
                      {
                        durationArr.map((item, i) => {
                          return <button id={i} type="button" className="btn btn-outline-primary"
                            onClick={() => { this.setState({ duration: item }) }}>{item}</button>;
                        })
                      }
                    </div>
                    <div className="row">
                      <div className="col-md-6" style={{ marginTop: '20px', cursor: 'pointer' }}
                        onClick={() => { this.setState({ durationView: false, borrow: true }); }}>
                        <a href="#" className="btn btn-primary btn-icon mb-3 mb-sm-0 m-5">
                          <span className="btn-inner--text">Back</span>
                        </a>
                      </div>
                      <div className="col-md-6" style={{ marginTop: '20px', cursor: 'pointer' }}
                        onClick={() => { this.setState({ durationView: false, monthlyInterest: true }) }}>
                        <a href="#" className="btn btn-primary btn-icon mb-3 mb-sm-0 m-5">
                          <span className="btn-inner--text">Next</span>
                        </a>
                      </div>
                    </div>
                  </div>

                  {/** Choose the monthly interest percentage */}
                  <div className="card-body" style={{ display: monthlyInterest ? 'block' : 'none', marginBottom: monthlyInt ? '123px' : '260px' }}>
                    <div className="alert alert-primary alert-dismissible fade show" role="alert">
                      <span className="alert-text">Choose the monthly interest percentage for this loan.</span>
                    </div>

                    <div className="text-left">
                      <button className="btn btn-icon btn-primary" type="button" value="minus" onClick={this.handleMonthlyInterest}>
                        -
                      </button>
                    </div>
                    <div className="text-right">
                      <input className="form-control" type="text" value={monthlyInt} style={{ width: '60px', marginTop: '-43px', marginLeft: '373px' }} id="example-time-input" />
                    </div>
                    <div className="text-right" style={{ marginTop: '-44px' }}>
                      <button className="btn btn-icon btn-primary" type="button" value="plus" onClick={this.handleMonthlyInterest}>
                        +
                      </button>
                    </div>

                    <div className="btn-wrapper" style={{ marginTop: '20px', cursor: 'pointer' }} onClick={() => { this.setState({ durationView: true, monthlyInterest: false }) }}>
                      <a href="#" className="btn btn-primary btn-icon mb-3 mb-sm-0 m-5">
                        <span className="btn-inner--text">Back</span>
                      </a>
                    </div>


                  </div>

                  {/** Loan Description */}
                  {monthlyInt ? <div>
                    <div className="alert alert-primary alert-dismissible fade show text-left pl-3 " role="alert">
                      <span className="alert-text">Total premium for this loan : {totalPremium} ETH ({apr.toFixed(2)}% APR)</span>
                    </div>
                    <h6 className="text-left pl-3" style={{ fontSize: "12px" }}>Monthly installment : {monthlyInstallment} ETH</h6>
                    <p className="text-left pl-3" style={{ fontSize: "12px" }}>Origination fee will be taken once the loan has been funded.</p>
                    <h6 className="text-left pl-3" style={{ fontSize: "13px" }}><span>Origination fee : {originationFee}%</span></h6>
                  </div>
                    : ''
                  }
                </div>
              </div>

              {/** Overview of the loan */}
              <div className="col-md-5">
                <div className="card">
                  <div className="card-header text-center">
                    Overview
                  </div>
                  <div className="card-body text-left mt-5" style={{ marginBottom: monthlyInt ? '176px' : '253px' }}>
                    {collateralValue ?
                      <div><p>Collateral : {collateralValue} {collateralCurrency}</p></div>
                      : <div><p>Collateral : (not set)</p></div>
                    }
                    {
                      loanAmount ? <div><p>Loan amount : {loanAmount} ETH </p></div>
                        :
                        <div><p>Loan amount : (not set)</p></div>
                    }
                    {
                      duration ? <p>Duration : {duration} days</p>
                        :
                        <p>Duration : (not set)</p>
                    }
                    {
                      monthlyInt ?
                        <p>Monthly interest (MPR) : {monthlyInt} %</p>
                        :
                        <p>Monthly interest (MPR) : (not set)</p>
                    }

                  </div>

                  {/** Create a loan request */}
                  {allowCreateRequest && monthlyInt ?
                    <div className="btn-wrapper text-center mb-5 mt-5" onClick={() => {

                      this.createLoanRequest(loanAmount, duration, monthlyInt * 100, "0x70Bb12F4A179D816767aB4e8d24A914D573A2839", collateralValue, 100);
                    }}>
                      <br />
                      <a className="btn btn-primary btn-icon " style={{ color: 'white' }}>
                        <span className="btn-inner--text">Create</span>
                      </a>
                    </div>
                    : ''
                  }
                  {{/** Approve Loan Request UI*/ }}
                  {approveRequestAlert &&
                    <div className="alert alert-primary" role="alert">
                      <strong>Approve Transfer collateral of {collateralValue} {collateralCurrency} tokens</strong>
                    </div>
                  }
                  {approveRequestAlert &&

                    <button className="btn btn-primary" type="button" onClick={() => {
                      this.handleERC20TokenApproval(collateralAddress, loanContractAddress, collateralValue)
                    }}>
                      Approve
                    </button>}

                  {/** Transfer Collateral UI */}
                  {transferCollateralAlert &&
                    <div className="alert alert-primary" role="alert">
                      <strong>Transfer collateral of {collateralValue} {collateralCurrency} tokens</strong>
                    </div>
                  }
                  {transferCollateralAlert &&
                    <button className="btn btn-primary" type="button" onClick={() => {
                      this.handleTransferCollateral(loanContractAddress, collateralAddress)
                    }}>
                      Transfer
                    </button>}
                  {/** Colateral Transfer status update UI */}
                  {transferCollateralFailAlert && <div className="alert alert-warning mt-2" style={{ marginLeft: '-1.5%', width: '104.5%' }} role="alert">
                    Collateral transfer has failed.
                  </div>}
                  {transferCollateralSuccessAlert && <div className="alert alert-success mt-2" style={{ marginLeft: '-1.5%', width: '104.5%' }} role="alert">
                    Collateral has been transferred successfully. Your loan request is waiting to be funded now!
                  </div>}
                </div>
              </div>
            </div>
          </section>
        </div>
        {createRequestAlert && <div className="alert alert-success" style={{ marginLeft: '9.5%', width: '46.5%', marginTop: '-7%' }} role="alert">
          <strong>Congratulations! Loan Request is Created successfully!</strong>
        </div>}
        {createRequestAlert && <Link href={"https://ropsten.etherscan.io/tx/" + ropstenTransactionhash} style={{ color: '#fff' }} target='_blank'> Check transation on Ropsten </Link>}
      </div>
    );
  }
}

// Export the `LoanRequest` class based component
export default LoanRequest;
```
***Loan offer UI component***
```
// Import the required libraries, styles and components
import React, { Component } from 'react';
import { Link } from 'react-router-dom';
import '../../assets/vendor/font-awesome/css/font-awesome.css';
import '../../assets/vendor/nucleo/css/nucleo.css';
import './LoanOffer.css';
import Header from './Header';
import { CreateNewLoanOffer, FetchCollateralPrice } from '../../services/loanbook';
import { TransferFundsToLoanContract } from '../../services/loanContract';
import { supported_erc20_token, getTokenBySymbol, getTokenByAddress } from '../Web3/erc20';

// Create a LoanOffer Class based component imported from 'react'
class LoanOffer extends Component {

  // Define the constructor and call `super()` method of parent class
  constructor() {
    super();

    // Initialize the state variables
    this.state = {
      loanCurrency: true,
      collateral: false,
      loan: false,
      currency: false,
      borrow: false,
      durationView: false,
      mprView: false,
      collateralValue: '(not set)',
      loanAmount: '(not set)',
      duration: null,
      monthlyInt: '(not set)',
      collateralSafe: '(not set)',
      ltv: 50,
      mpr: 1,
      collateralCurrency: null,
      createOfferAlert: true,
      approveOfferAlert: false,
      acceptLoanAlert: false,
      loanContractAddress: '',
      ropstenTransactionhash: '',
      durationArr: [30, 60, 90, 120, 150, 180, 210, 240, 270, 300, 330, 360],
      durationStart: 1,
      durationEnd: 360,
      stableCoins: ['STABLE COINS', 'DAI', 'PAX', 'TUSD'],
      erc20_tokens: supported_erc20_token,
      collateralMetadata: [],
    };
  }

  // Array removal method
  arrayRemove = (arr, value) => {

    return arr.filter(function (ele) {
      return ele != value;
    });

  }


  // Create Loan offer
  createLoanOffer = async (principal, duration, collateralMetadata) => {
    try {
      console.log("collateralMetadata", collateralMetadata);
      let collaterals = [];
      let collateralAddress = [];
      collateralMetadata.map((collateral) => {
        collateralAddress = getTokenBySymbol[collateral.collateral] && getTokenBySymbol[collateral.collateral].address;
        collaterals.push({
          collateral: collateralAddress,
          ltv: collateral.ltv,
          mpr: collateral.mpr
        });
      })

      const loanContractAddress = await CreateNewLoanOffer({
        principal: principal,
        duration: duration,
        collaterals: collaterals
      });

      this.setState({
        createOfferAlert: false,
        approveOfferAlert: true,
        loanContractAddress: loanContractAddress
      });

    } catch (error) {

    }
  }

  // Fund loan offer
  fundLoanOffer = async (loanAmount, loanContractAddress) => {

    try {
      await TransferFundsToLoanContract(loanContractAddress, loanAmount);

      //window.location="/myloans";
      console.log("fundLoanOffer => loanContractAddress line 87", loanContractAddress);

      this.setState({
        approveOfferAlert: false,
        acceptLoanAlert: true
      });

    } catch (error) {
      console.log(error);
    }
  }

  // Accept loan offer
  acceptLoanOffer = async (interest, collateralAddress, loanContractAddress, collateralAmount, collateralPrice, ltv) => {
    console.log("acceptLoanOffer => loanContractAddress line 102", loanContractAddress);
    // var self = this;
    // const LoanContract = window.web3.eth.contract(LoanContractABI).at(loanContractAddress);
    // const acceptLoan = await LoanContract.acceptLoanOffer(interest, collateralAddress, collateralAmount, collateralPrice, ltv,{
    //   from: window.web3.eth.accounts[0],
    //   gas: 300000
    // }, (err, transactionHash) => {
    //   if (!err)
    //     console.log(transactionHash);
    // })
    // console.log("ACCEPT LOAN :", acceptLoan);

  }

  // Handle addition of new collaterals
  handleAddCollateral = (collateralCurrency, ltv, mpr) => {
    let { collateralMetadata, erc20_tokens } = this.state;
    collateralMetadata.push({
      collateral: collateralCurrency,
      ltv: ltv,
      mpr: mpr * 100
    });
    let array = [...erc20_tokens]; // make a separate copy of the array
    this.setState({
      erc20_tokens: array.filter((item) => {
        console.log(item.symbol, collateralCurrency);
        return item.symbol !== collateralCurrency
      })
    });
    this.setState({ collateralMetadata: collateralMetadata })
  }

  // handle event after collaterals are selected
  handleCollateralNext = () => {
    this.setState({ durationView: true, collateral: false });
  }

  // Call the render method for UI components
  render() {

    // Initialize the state variables to the local variables
    const { loanAmount, duration, monthlyInt, loan, currency, borrow, durationView, durationArr, monthlyInterest, borrowLess, erc20_tokens, collateralMetadata, collateralValue,
      collateralCurrency, ltv, mpr, createOfferAlert, approveOfferAlert, acceptLoanAlert, loanContractAddress, ropstenTransactionhash } = this.state;

    // Call the return() for UI components
    return (
      // Create a parent `<div>` and must to keep every UI component inside it
      <div className="LoanOffer text-center">

        {/** Display the Header component */}
        <Header />
        <div className="position-relative">
          <section className="section-hero section-shaped my-0">
            <div className="">
              <span className="span-150"></span>
              <span className="span-50"></span>
              <span className="span-50"></span>
              <span className="span-75"></span>
              <span className="span-100"></span>
              <span className="span-75"></span>
              <span className="span-50"></span>
              <span className="span-100"></span>
              <span className="span-50"></span>
              <span className="span-100"></span>
            </div>

            {/** Display Loan offer UI component */}
            <div className="container shape- d-flex align-items-center">
              <div className="col-lg-7">
                <div className="card">
                  <div className="card-header text-center">
                    <h5> New Loan Offer</h5>
                  </div>

                  {/** Choose your loan currency UI*/}
                  <div className="card-body" style={{ display: this.state.loanCurrency ? 'block' : 'none' }}>
                    <div className="alert alert-primary alert-dismissible fade show" role="alert">
                      <span className="alert-text"><strong>Choose your loan offer currency.</strong></span>
                    </div>
                    <div className="row mt-5">
                      <div className="col-md-6" style={{ marginTop: '25px', marginBottom: '230px', cursor: 'pointer' }} onClick={() => { this.setState({ loanCurrency: false, loan: true }); }}>
                        <span className="btn-inner--text"><img style={{ width: '25px' }} src="/assets/img/eth.png" /></span>
                        <br />
                        <p>Ethereum</p>
                      </div>
                      <div className="col-md-4 form-group mt-3">
                        <select className="form-control" id="exampleFormControlSelect1">
                          {
                            this.state.stableCoins.map((item, i) => {
                              return <option>{item}</option>;
                            })
                          }
                        </select>
                      </div>
                    </div>
                  </div>

                  {/** Insert Loan abbount UI */}
                  <div className="card-body" style={{ display: this.state.loan ? 'block' : 'none' }}>
                    <div className="alert alert-primary alert-dismissible fade show" role="alert">
                      <span className="alert-text"><strong>Insert the loan offer amount.</strong></span>
                    </div>
                    <input className="form-control form-control-lg" type="text" placeholder="ETH" onChange={(e) => { this.setState({ loanAmount: e.target.value }); }} />
                    <div className="btn-wrapper" style={{ marginTop: '200px', cursor: 'pointer' }} onClick={() => { this.setState({ loanCurrency: false, loan: false, borrow: true }); }}>
                      <a href="#" className="btn btn-primary btn-icon mb-3 mb-sm-0 m-5">
                        <span className="btn-inner--text">Next</span>
                      </a>
                    </div>
                  </div>

                  {/** Choose loan currency UI */}
                  <div className="card-body" style={{ display: this.state.currency ? 'block' : 'none' }}>
                    <div className="alert alert-primary alert-dismissible fade show" role="alert">
                      <span className="alert-text"><strong>Choose your loan currency.</strong></span>
                    </div>
                    <div className="btn-wrapper" style={{ marginTop: '200px', cursor: 'pointer' }} onClick={() => { this.setState({ currency: false }); }}>
                      <span className="btn-inner--text"><img src="/assets/img/eth.png" /></span>
                      <br />
                      <p>Ethereum</p>
                    </div>
                  </div>

                  <div className="card-body" style={{ display: this.state.borrow ? 'block' : 'none' }}>
                    <div className="alert alert-success alert-dismissible fade show" role="alert">
                      <span className="alert-text"><strong>Are you willing to lend less?</strong></span>
                    </div>
                    <input className="form-control form-control-lg" type="text" value={loanAmount} placeholder={loanAmount} onChange={(e) => { this.setState({ loanAmount: e.target.value }); }} />
                    <div className="btn-wrapper" style={{ marginTop: '200px', cursor: 'pointer' }} onClick={() => { this.setState({ collateral: true, borrow: false }); }}>
                      <a href="#" className="btn btn-primary btn-icon mb-3 mb-sm-0 m-5">
                        <span className="btn-inner--text">Next</span>
                      </a>
                    </div>
                  </div>

                  {/** Add collateral UI */}
                  <div className="card-body" style={{ display: this.state.collateral ? 'block' : 'none' }}>
                    <div className="alert alert-primary alert-dismissible fade show" role="alert">
                      <span className="alert-text">Add your collateral.</span>
                    </div>
                    {<div className="row ml-2">
                      {<div className="card bg-gradient-success col-md-5" style={{ height: '300px', marginLeft: '30%' }}>
                        <div className="col-md-12 form-group mt-5">
                          <select className="form-control" id="exampleFormControlSelect1" style={{ width: '80px', display: 'inline' }} onClick={(e) => {
                            this.setState({ collateralCurrency: e.target.value });
                          }}>
                            {
                              erc20_tokens.map((item) => {
                                return <option>{item.symbol}</option>;
                              })
                            }
                          </select>
                          <h6 className="mt-4">LTV</h6>
                          <input className="font-weight-bold mb-0" type="number" value={ltv} style={{ width: 'inherit', textAlign: 'center' }} onChange={(e) => this.setState({ ltv: (e.target.value > 0 && e.target.value <= 50) ? e.target.value : 1 })} />
                          <h6 className="mt-4">Interest</h6>
                          <input className="font-weight-bold mb-0" type="number" value={mpr} style={{ width: 'inherit', textAlign: 'center' }} onChange={(e) => this.setState({ mpr: (e.target.value > 0 && e.target.value <= 5) ? e.target.value : 1 })} />
                        </div>
                      </div>
                      }
                    </div>
                    }

                    <div className="text-center mt-3">
                      <button className="btn btn-icon btn-primary" type="button" value="plus" onClick={() => this.handleAddCollateral(collateralCurrency, ltv, mpr)}>
                        <span><i className="fa fa-plus"></i></span>
                      </button>
                    </div>

                    <div className="btn-wrapper" style={{ marginTop: '-30px', cursor: 'pointer' }} onClick={this.handleCollateralNext}>
                      <a href="#" className="btn btn-primary btn-icon mb-3 mb-sm-0 m-5">
                        <span className="btn-inner--text">Next</span>
                      </a>
                    </div>
                  </div>

                  {/** Define loan duration UI */}
                  <div className="card-body" style={{ display: this.state.durationView ? 'block' : 'none' }}>
                    <div className="alert alert-primary alert-dismissible fade show" role="alert">
                      <span className="alert-text">Define loan duration.</span>
                    </div>
                    <br />
                    <div className="btn-wrapper" style={{ marginTop: '245px', cursor: 'pointer' }}>
                      {
                        this.state.durationArr.map((item, i) => {
                          return <button id={i} type="button" className="btn btn-outline-primary" onClick={() => { this.setState({ duration: item }) }}>{item}</button>;
                        })
                      }
                    </div>
                  </div>
                </div>
              </div>

              {/** Loan Overview UI */}
              <div className="col-md-5">
                <div className="card">
                  <div className="card-header text-center">
                    Overview
                  </div>
                  <div className="card-header text-left">
                    <p>Loan amount {loanAmount} ETH</p>
                  </div>

                  {/** Selected collateral UI */}
                  <div className="card-body text-left" style={{ marginBottom: !duration ? '45%' : '21%' }}>
                    <p>Collateral</p>
                    {collateralMetadata.map((collateral, i) => {
                      console.log('collateralMetadata', collateral);
                      return <div className="col" key={i}>
                        <img id="img1 " alt="img1" src={`/assets/img/32/color/${collateral.collateral}.png`} alt={collateral.collateral} />
                        <p className="small">{collateral.collateral}</p>
                        <p className="small">LTV : {collateral.ltv}</p>
                        <p className="small">MPR: {collateral.mpr / 100}</p>
                      </div>;
                    })

                    }
                    {/** Fund loan UI */}
                    {duration ? <div className="mt-4"><p>Duration {duration} days</p></div>
                      : <div className="mt-4"><p>Duration  (not set) </p></div>}

                    {!!duration &&
                      createOfferAlert &&
                      <div className="btn-wrapper text-center" onClick={() => {
                        this.createLoanOffer(loanAmount, duration, collateralMetadata);
                      }}>
                        <br />
                        <a className="btn btn-primary btn-icon mb-3 mb-sm-0 m-5" style={{ color: 'white' }}>
                          <span className="btn-inner--text">Create</span>
                        </a>
                      </div>}
                    {approveOfferAlert &&
                      <div className="btn-wrapper text-center mt-3">
                        <button className="btn btn-primary" type="button" onClick={() => {
                          this.fundLoanOffer(loanAmount, loanContractAddress);
                        }}>
                          Fund Loan
                        </button>
                      </div>}
                    {/*acceptLoanAlert &&
                  <div className="btn-wrapper text-center mt-3">
                    <button className="btn btn-primary" type="button" onClick={()=>{
                      this.acceptLoanOffer(mpr1, CollateralAddress.toString(), loanContractAddress, window.web3.toWei(loanAmount), window.web3.toWei(0.1), ltv1);
                      }}>
                      Accept Loan
                    </button>
                  </div>*/}

                    {/** Funded loan status update UI */}
                    {acceptLoanAlert && <div className="alert alert-success mt-2" style={{ marginLeft: '-1.5%', width: '104.5%', marginTop: '-7%' }} role="alert">
                      Your loan is funded successfully!
                    </div>}
                  </div>
                </div>
              </div>
            </div>
          </section>
        </div>

        {/** Loan offer creation status update UI */}
        {!createOfferAlert && <div className="alert alert-success" style={{ marginLeft: '9.5%', width: '46.5%', marginTop: '-7%' }} role="alert">
          <strong>Congratulations! Loan Offer is Created successfully!</strong>
        </div>}
        {createOfferAlert && <Link to={"https://ropsten.etherscan.io/tx/" + ropstenTransactionhash} style={{ color: '#fff' }} target='_blank'> Check transation on Ropsten </Link>}
      </div>
    );
  }
}

// Export the LoanOffer component
export default LoanOffer;
```

***My Loans UI component***

```
// Import the libraries, utils, styles and components
import React, { Component } from "react";
import {
  LoanContractABI,
  StandardTokenABI,
} from "../Web3/abi";
import Header from "../pages/Header";
import { getTokenByAddress } from '../Web3/erc20';
import { enableWeb3 } from '../Web3/enableWeb3';
import { GetLoans } from "../../services/loanbook";
import {
  GetLoanDetails,
  GetRepaymentData,
  RepayLoan,
  ClaimCollateralByBorrower,
  ClaimCollateralByLender,
  LiquidateLoanCollateral
} from "../../services/loanContract";
import "../../assets/vendor/font-awesome/css/font-awesome.css";
import "../../assets/vendor/nucleo/css/nucleo.css";
import "./MyLoans.css";

// Create a class based component imported from react Component
class MyLoans extends Component {

  // declare `Constructor()` and call super method of parent class
  constructor() {
    super();

    // call `enableWeb3()` method
    enableWeb3();

    // call `viewMyLoans()` method
    this.viewMyLoans();

    // initialize state variables
    this.state = {
      myBorrowedLoans: [],
      myFundedLoans: [],
      repayments: [],
      activeLoan: [],
      loanStatuses: [
        "INACTIVE",
        "OFFER",
        "REQUEST",
        "ACTIVE",
        "FUNDED",
        "REPAID",
        "DEFAULT"
      ],
      currentDate: null,
      currentDueDate: null,
      repaymentDuration: 0,
      activeRepayment: 0,
      currentLoanAddress: "",
      currentLoanNumber: 0,
      repaymentIndex: 0,
      currentCollateralValue: "",
      loaded: true,
      approveRequestAlert: false,
      transferCollateralAlert: false,
      borrowedLoans: true,
      fundedLoans: false,
      showDropDown: null,
      loanRepaid: 0
    };
  }

  //Get All Loans
  viewMyLoans = async () => {
    try {
      const loans = await GetLoans();

      let currentDate = new Date();
      this.setState({ currentDate: currentDate });

      let { myBorrowedLoans, myFundedLoans } = this.state;
      myBorrowedLoans = [];
      myFundedLoans = [];
      loans.map(async loanAddress => {
        const loan = await GetLoanDetails(loanAddress);
        const user = window.web3.eth.accounts[0];

        if (loan[10] === user && loan[5].toNumber() > 2) {
          myBorrowedLoans.push({
            loanAddress: loanAddress,
            loanAmount: window.web3.fromWei(loan[0].toNumber()),
            duration: loan[1].toNumber(),
            interest: loan[2].toNumber(),
            createOn: loan[3].toNumber(),
            startedOn: loan[4].toNumber(),
            status: loan[5].toNumber(),
            collateralAddress: loan[6],
            collateralAmount: loan[7].toNumber(),
            collateralPrice: loan[8].toNumber(),
            collateralStatus: loan[9].toNumber(),
            borrower: loan[10],
            lender: loan[11],
            liquidatedAmount: window.web3.fromWei(loan[12].toNumber()),
            collaterals: {
              address: loan[13][0][0],
              ltv: loan[13][0][1],
              mpr: loan[13][0][2]
            }
          });
          this.setState({
            myBorrowedLoans: myBorrowedLoans
          });
        }
        if (loan[11] === user && loan[5].toNumber() > 2) {
          myFundedLoans.push({
            loanAddress: loanAddress,
            loanAmount: window.web3.fromWei(loan[0].toNumber()),
            duration: loan[1].toNumber(),
            interest: loan[2].toNumber(),
            createOn: loan[3].toNumber(),
            startedOn: loan[4].toNumber(),
            status: loan[5].toNumber(),
            collateralAddress: loan[6],
            collateralAmount: loan[7].toNumber(),
            collateralPrice: loan[8].toNumber(),
            collateralStatus: loan[9].toNumber(),
            borrower: loan[10],
            lender: loan[11],
            liquidatedAmount: window.web3.fromWei(loan[12].toNumber()),
            collaterals: {
              address: loan[13][0][0],
              ltv: loan[13][0][1],
              mpr: loan[13][0][2]
            }
          });

          console.log(myFundedLoans);
          this.setState({
            myFundedLoans: myFundedLoans
          });
        }
      });
    } catch (e) {
      console.log(e);
    } finally {
    }
  };

  // create `getActiveLoanRepayments()`
  getActiveLoanRepayments = async (loanAddress, duration) => {
    let { repayments } = this.state;

    repayments = new Array();
    try {
      let totalNumberOfRepayments = duration / 30;

      for (var i = 1; i <= totalNumberOfRepayments; i++) {
        const repaymentData = await GetRepaymentData(loanAddress, i);
        repayments.push(repaymentData);
      }

      return repayments;
    } catch (error) {
      console.log(error);
    }
  };

  // create `getInActiveLoanRepayments()`
  getInActiveLoanRepayments = loanAddress => { };

  handleLoanRepayment = async (loanContractAddress, repaymentAmount) => {
    // Repay Loan
    try {
      await RepayLoan(loanContractAddress, repaymentAmount);
    } catch (error) {
      console.log(error);
    }
  };

  // create `handleLoanRepaid()`
  handleLoanRepaid = async (repayments, activeLoan, currentDate) => {
    try {
      let { loanRepaid } = this.state;
      let totalRepayment = 0;
      let totalRepaid = 0;
      repayments.map((repayment) => {
        totalRepayment = totalRepayment + parseFloat(repayment.totalRepaymentAmount)
        if (activeLoan.borrower === repayment.repayee) {
          totalRepaid = totalRepaid + parseFloat(repayment.totalRepaymentAmount)
        }
      })

      this.setState({ loanRepaid: ((totalRepaid / totalRepayment) * 100) })
    } catch (error) {
      console.log(error);
    }
  }

  // create `handleRepaymentRows()`
  handleRepaymentRows = (
    currentLoanAddress,
    duration,
    currentDueDate,
    currentCollateralValue
  ) => {
    let {
      repaymentRows,
      repaymentAmount,
      repaymentNumber,
      loanAddresses,
      dueDate,
      currentDate,
      repaymentIndex,
      activeRepayment
    } = this.state;
    repaymentRows = [];

    for (var i = 0; i < duration / 30; i++) {
      repaymentRows.push(
        <tr id="repay" key={i}>
          <td>
            <div className="media-body">
              <span className="mb-0 text-sm">Repayment No.</span>
            </div>
            <span className="badge-dot">
              <i className="bg-info"> </i>
              {i + 1}
            </span>
          </td>
          <td>
            <div className="media-body">
              <span className="mb-0 text-sm">Amount</span>
            </div>
            <span>{repaymentAmount[i]} ETH</span>
          </td>
          <td>
            <div className="media-body">
              <span className="mb-0 text-sm">Due Date</span>
            </div>
            <span>
              {
                this.convertDate(currentDueDate, i).split(
                  " GMT+0530 (India Standard Time)"
                )[0]
              }
            </span>
          </td>
          <td>
            <div className="media-body">
              <span className="mb-0 text-sm">Status</span>
            </div>
            <span>{currentDate > dueDate[i] ? "Due" : "Defaulted"}</span>
          </td>

          <td>
            <div className="media-body">
              <span className="mb-0 text-sm">Action</span>
            </div>
            <span>
              <button
                className={
                  repaymentIndex == activeRepayment
                    ? "btn btn-success"
                    : "btn btn-primary"
                }
                style={{ fontSize: "9px", padding: "2px", fontStyle: "bold" }}
                type="button"
                onClick={() => {
                  console.log(
                    "currentLoanAddress",
                    currentLoanAddress,
                    "repaymentAmount",
                    repaymentAmount[repaymentIndex]
                  );
                  this.handleRepayment(
                    currentLoanAddress,
                    repaymentAmount[repaymentIndex]
                  );
                  this.setState({ repaymentIndex: repaymentIndex + 1 });
                }}
              >
                Repay
              </button>
            </span>
          </td>

          <td>
            <div className="media-body">
              <span className="mb-0 text-sm">Comments</span>
            </div>
            <span>--</span>
          </td>

          <td>
            <div className="media-body">
              <span className="mb-0 text-sm">Collateral</span>
            </div>
            <span>Value : {currentCollateralValue} {/*getTokenByAddress[loan.collateralAddress] && getTokenByAddress[loan.collateralAddress].symbol*/}</span>
          </td>
          {/*repaymentIndex==duration/30 && <td>
               <button className="btn btn-primary" type="button" onClick={()=>{
                 this.handleReturnCollateralToBorrower(loanAddresses[i])
                 }}>
                 Claim
               </button>
               </td>*/}
        </tr>
      );
    }
    return repaymentRows;
  };

  // create `approveRequest()` method
  approveRequest = (
    collateralAddress,
    loanContractAddress,
    collateralAmount
  ) => {
    // Transfer Collateral to Loan Contract
    // this will be two transaction, first transaction will be to Token Contract and Second will be to Loan Contract

    // Transaction 1 Approval

    this.setState({ approveRequestAlert: true });

    const tokenContractInstance = window.web3.eth
      .contract(StandardTokenABI)
      .at(collateralAddress);
    tokenContractInstance.approve(
      loanContractAddress,
      collateralAmount,
      {
        from: window.web3.eth.accounts[0]
      },
      function (err, res) {
        if (!err) {
          console.log(res);
          window.location = "/myloans";
        } else {
        }
      }
    );
  };

  handleTransferCollateral = loanContractAddress => {
    // Transfer Collateral to Loan Contract

    // Transaction 2 Transfer to Loan Contract
    this.setState({ transferCollateralAlert: true });
    const LoanInstance = window.web3.eth
      .contract(LoanContractABI)
      .at(loanContractAddress);
    LoanInstance.transferCollateralToLoan(
      {
        from: window.web3.eth.accounts[0]
      },
      function (err, res) {
        if (!err) console.log(res);
        window.location = "/myloans";
      }
    );
  };

  // create `handleReturnCollateralToBorrower()` method
  handleReturnCollateralToBorrower = loanAddress => {
    const LoanInstance = window.web3.eth
      .contract(LoanContractABI)
      .at(loanAddress);
    LoanInstance.returnCollateralToBorrower((err, res) => {
      if (!err) {
        console.log(res);
      }
    });
  };

  // create `convertDate()` method
  convertDate = (currentDueDate, i) => {
    let date = new Date(currentDueDate * 1000);
    date.setMinutes(date.getMinutes() + (i + 1) * 30);
    date = date.toString();
    return date;
  };

  // create `convertDateEpoc()` method
  convertDateEpoc = (currentDueDate, i) => {
    let date = new Date(currentDueDate * 1000);
    date.setMinutes(date.getMinutes() + (i + 1) * 30);
    return date;
  };

  // call `render()` method to display UI components
  render() {
    const {
      myBorrowedLoans,
      myFundedLoans,
      repayments,
      activeLoan,
      loanRepaid,
      currentDate,
      borrowedLoans,
      fundedLoans,
      loanStatuses,
      approveRequestAlert,
      transferCollateralAlert,
      showDropDown
    } = this.state;
    return (
      <div className="MyLoans text-center">
        {/*<Loader loaded={loaded}/>*/}
        <Header />
        <div className="position-relative">
          <section className="section-hero section-shaped my-0">
            <div className="">
              <span className="span-150"></span>
              <span className="span-50"></span>
              <span className="span-50"></span>
              <span className="span-75"></span>
              <span className="span-100"></span>
              <span className="span-75"></span>
              <span className="span-50"></span>
              <span className="span-100"></span>
              <span className="span-50"></span>
              <span className="span-100"></span>
            </div>

            {/** My Borrowed loans and funded loans */}
            <div className="d-flex align-items-center">
              <div className="col px-0">
                <div className="card">
                  <div className="card-header text-left">
                    <a
                      href="#"
                      className={
                        borrowedLoans
                          ? " btn btn-primary"
                          : " btn btn-secondary"
                      }
                      onClick={() => {
                        this.setState({
                          borrowedLoans: true,
                          fundedLoans: false
                        });
                      }}
                    >
                      My Borrowed Loans
                    </a>
                    <a
                      href="#"
                      className={
                        fundedLoans ? "btn btn-primary" : "btn btn-secondary"
                      }
                      onClick={() => {
                        this.setState({
                          borrowedLoans: false,
                          fundedLoans: true
                        });
                      }}
                    >
                      My Funded Loans
                    </a>
                  </div>

                  {/** My Borrowed loans table */}
                  <div className="card-body">
                    <div>
                      {borrowedLoans ? (
                        <div
                          className="table-responsive"
                          style={{ marginTop: "-25px" }}
                        >
                          <table className="table align-items-center table-flush">
                            <thead className="thead">
                              <tr>
                                <th scope="col">Loan Address</th>
                                <th scope="col">Loan Amount</th>
                                <th scope="col">Collateral Currency</th>
                                <th scope="col">Duration</th>
                                <th scope="col">MPR</th>
                                <th scope="col">Status</th>
                                <th scope="col">Expires on</th>
                                <th scope="col">
                                  <i className="fa fa-arrrow-down"></i>
                                </th>
                              </tr>
                            </thead>
                            <tbody>
                              {myBorrowedLoans.map((loan, i) => {
                                return (
                                  <>
                                    <tr key={i}>
                                      <th scope="row mt-3">
                                        <div className="media align-items-center">
                                          <div className="media-body">
                                            <span className="mb-0 text-sm">
                                              {loan.loanAddress}
                                            </span>
                                          </div>
                                        </div>
                                      </th>
                                      <td>
                                        <span className="badge-dot">
                                          <i className="bg-info"></i>{" "}
                                          {loan.loanAmount} ETH
                                        </span>
                                      </td>
                                      <td>
                                        <div className="text-center">
                                          <span className="">{getTokenByAddress[loan.collateralAddress] && getTokenByAddress[loan.collateralAddress].symbol}</span>
                                          <div></div>
                                        </div>
                                      </td>
                                      <td>
                                        <div className="text-center">
                                          <span className="">
                                            {loan.duration}
                                          </span>
                                          <div></div>
                                        </div>
                                      </td>

                                      <td>
                                        <div className="text-center">
                                          <span className="">
                                            {loan.interest / 100}%
                                          </span>
                                          <div></div>
                                        </div>
                                      </td>
                                      <td>
                                        <div className="text-center">
                                          <span className="">
                                            {" "}
                                            {loanStatuses[loan.status]}
                                          </span>
                                          <div></div>
                                        </div>
                                      </td>
                                      <td>
                                        <span className="">
                                          {
                                            this.convertDate(
                                              loan.startedOn,
                                              -1
                                            ).split(
                                              " GMT+0530 (India Standard Time)"
                                            )[0]
                                          }
                                        </span>
                                      </td>
                                      {loan.status > 0 && (
                                        <td>
                                          <DropDown
                                            id={loan.loanAddress}
                                            show={
                                              showDropDown === loan.loanAddress && loan.loanAddress === (repayments && repayments.loanContractAddress)
                                            }
                                            activeLoan={loan}
                                            loanAddress={loan.loanAddress}
                                            duration={loan.duration}
                                            currentDate={currentDate}
                                            self={this}
                                          />
                                        </td>
                                      )}
                                    </tr>
                                    {showDropDown === loan.loanAddress &&
                                      repayments.map((repayment, i) => {
                                        return (
                                          <tr id="repay" key={i}>
                                            <td>
                                              <div className="media-body">
                                                <span className="mb-0 text-sm">
                                                  Repayments
                                                </span>
                                              </div>

                                              <span className="badge-dot">
                                                <i className="bg-info"></i>{" "}
                                                {repayment &&
                                                  repayment.repaymentNumber}
                                              </span>
                                            </td>
                                            <td>
                                              <div className="media-body">
                                                <span className="mb-0 text-sm">
                                                  Amount
                                                </span>
                                              </div>
                                              <span>
                                                {repayment &&
                                                  repayment.totalRepaymentAmount}{" "}
                                                ETH
                                              </span>
                                            </td>
                                            <td>
                                              <div className="media-body">
                                                <span className="mb-0 text-sm">
                                                  Due Date
                                                </span>
                                              </div>
                                              <span>
                                                {
                                                  this.convertDate(
                                                    activeLoan.startedOn,
                                                    repayment.repaymentNumber - 1
                                                  ).split(
                                                    " GMT+0530 (India Standard Time)"
                                                  )[0]
                                                }
                                              </span>
                                            </td>
                                            <td>
                                              <div className="media-body">
                                                <span className="mb-0 text-sm">
                                                  Status
                                                </span>
                                              </div>
                                              <span>
                                                {activeLoan.borrower === repayment.repayee
                                                  ? "Paid"
                                                  : currentDate >
                                                    this.convertDateEpoc(
                                                      activeLoan.startedOn,
                                                      repayment.repaymentNumber - 1
                                                    )
                                                    ? "Defaulted"
                                                    : "Due"}
                                              </span>
                                            </td>
                                            {(currentDate <
                                              this.convertDateEpoc(
                                                activeLoan.startedOn,
                                                repayment.repaymentNumber - 1
                                              )) &&
                                              activeLoan.borrower != repayment.repayee &&
                                              <td>
                                                <button
                                                  className="btn btn-info"
                                                  onClick={() => {
                                                    this.handleLoanRepayment(
                                                      repayment &&
                                                      repayment.loanContractAddress,
                                                      repayment &&
                                                      repayment.totalRepaymentAmount
                                                    );

                                                  }}
                                                >
                                                  Repay
                                                </button>
                                              </td>}
                                          </tr>
                                        );
                                      })}
                                    {showDropDown === loan.loanAddress && <tr>
                                      <td>
                                        <div className="media-body">
                                          <span className="mb-0 text-sm">
                                            {" "}
                                            Collateral Amount
                                          </span>
                                        </div>
                                        <span>
                                          {parseFloat(activeLoan.collateralAmount)}
                                          {activeLoan && getTokenByAddress[activeLoan.collateralAddress] && getTokenByAddress[activeLoan.collateralAddress].symbol}
                                        </span>
                                      </td>
                                      <td>
                                        <span>
                                          Loan Repaid{" "}
                                        </span>
                                        <span>
                                          {loanRepaid.toFixed(2) + "% "}
                                        </span>
                                      </td>
                                      <td>
                                        <span>
                                          Collateral Left {" "}
                                        </span>
                                        <span>
                                          {activeLoan && activeLoan.collateralAmount}
                                          {activeLoan && getTokenByAddress[activeLoan.collateralAddress] && getTokenByAddress[activeLoan.collateralAddress].symbol}
                                        </span>
                                      </td>
                                      {currentDate >
                                        this.convertDateEpoc(
                                          activeLoan.startedOn,
                                          repayments[(activeLoan.duration / 30) - 1].repaymentNumber - 1
                                        )
                                        && activeLoan.collateralStatus < 2
                                        &&
                                        loanRepaid.toFixed(2) == 100.00 &&
                                        <td>
                                          <button
                                            className="btn btn-primary"
                                            type="button"
                                            onClick={() => {
                                              ClaimCollateralByBorrower(
                                                repayments[(activeLoan.duration / 30) - 1].loanContractAddress
                                              );
                                            }}
                                          >
                                            Claim
                                          </button>
                                        </td>}
                                    </tr>}
                                  </>
                                );
                              })}
                            </tbody>
                          </table>
                        </div>
                      ) : (
                        <div
                          className="table-responsive"
                          style={{ marginTop: "-25px" }}
                        >
                          <table className="table align-items-center table-flush">
                            <thead className="thead">
                              <tr>
                                <th scope="col">Loan Id</th>
                                <th scope="col">Loan Amount</th>
                                <th scope="col">Collateral Currency</th>
                                <th scope="col">Duration</th>
                                <th scope="col">MPR</th>
                                <th scope="col">Status</th>
                                <th scope="col">Expires on</th>
                                <th scope="col">
                                  <i className="fa fa-arrrow-down"></i>
                                </th>
                              </tr>
                            </thead>
                            <tbody>
                              <tr
                                style={{ cursor: "pointer" }}
                                onClick={() => {
                                  this.setState({ display1: false });
                                }}
                              />
                              {myFundedLoans.map(loan => {
                                return (
                                  <>
                                    <tr key={loan.loanAddress}>
                                      <th scope="row mt-3">
                                        <div className="media align-items-center">
                                          <div className="media-body">
                                            <span className="mb-0 text-sm">
                                              {loan.loanAddress}
                                            </span>
                                          </div>
                                        </div>
                                      </th>
                                      <td>
                                        <span className="badge-dot">
                                          <i className="bg-info"></i>{" "}
                                          {loan.loanAmount} ETH
                                        </span>
                                      </td>
                                      <td>
                                        <div className="text-center">
                                          <span className="">{getTokenByAddress[loan.collateralAddress] && getTokenByAddress[loan.collateralAddress].symbol}</span>
                                          <div></div>
                                        </div>
                                      </td>
                                      <td>
                                        <div className="text-center">
                                          <span className="">
                                            {loan.duration}
                                          </span>
                                          <div></div>
                                        </div>
                                      </td>

                                      <td>
                                        <div className="text-center">
                                          <span className="">
                                            {loan.interest / 100}%
                                          </span>
                                          <div></div>
                                        </div>
                                      </td>
                                      <td>
                                        <div className="text-center">
                                          <span className="">
                                            {" "}
                                            {loanStatuses[loan.status]}
                                          </span>
                                          <div></div>
                                        </div>
                                      </td>
                                      <td>
                                        <span className="">
                                          {
                                            this.convertDate(
                                              loan.startedOn,
                                              -1
                                            ).split(
                                              " GMT+0530 (India Standard Time)"
                                            )[0]
                                          }
                                        </span>
                                      </td>

                                      {loan.status > 0 && (
                                        <td>
                                          <DropDown
                                            id={loan.loanAddress}
                                            show={
                                              showDropDown === loan.loanAddress
                                            }
                                            activeLoan={loan}
                                            loanAddress={loan.loanAddress}
                                            duration={loan.duration}
                                            self={this}
                                          />
                                        </td>
                                      )}
                                    </tr>
                                    {showDropDown === loan.loanAddress &&
                                      repayments.map((repayment, i) => {
                                        return (
                                          <tr id="repay" key={i}>
                                            <td>
                                              <div className="media-body">
                                                <span className="mb-0 text-sm">
                                                  Repayments
                                                </span>
                                              </div>

                                              <span className="badge-dot">
                                                <i className="bg-info"></i>{" "}
                                                {repayment &&
                                                  repayment.repaymentNumber}
                                              </span>
                                            </td>
                                            <td>
                                              <div className="media-body">
                                                <span className="mb-0 text-sm">
                                                  Amount
                                                </span>
                                              </div>
                                              <span>
                                                {repayment &&
                                                  repayment.totalRepaymentAmount}{" "}
                                                ETH
                                              </span>
                                            </td>
                                            <td>
                                              <div className="media-body">
                                                <span className="mb-0 text-sm">
                                                  Due Date
                                                </span>
                                              </div>
                                              <span>
                                                {
                                                  this.convertDate(
                                                    activeLoan.startedOn,
                                                    repayment.repaymentNumber - 1
                                                  ).split(
                                                    " GMT+0530 (India Standard Time)"
                                                  )[0]
                                                }
                                              </span>
                                            </td>
                                            <td>
                                              <div className="media-body">
                                                <span className="mb-0 text-sm">
                                                  Status
                                                </span>
                                              </div>
                                              <span>
                                                {activeLoan.borrower === repayment.repayee
                                                  ? "Paid"
                                                  : currentDate >
                                                    this.convertDateEpoc(
                                                      activeLoan.startedOn,
                                                      repayment.repaymentNumber - 1
                                                    )
                                                    ? "Defaulted"
                                                    : "Due"}
                                              </span>
                                            </td>
                                            {currentDate >
                                              this.convertDateEpoc(
                                                activeLoan.startedOn,
                                                repayment.repaymentNumber - 1
                                              )
                                              && activeLoan.borrower !== repayment.repayee
                                              && activeLoan.lender !== repayment.repayee
                                              && <td>
                                                <button
                                                  className="btn btn-primary"
                                                  type="button"
                                                  onClick={() => {
                                                    ClaimCollateralByLender(
                                                      repayments[0].loanContractAddress, repayment.repaymentNumber
                                                    );
                                                  }}
                                                >
                                                  Claim
                                                </button>
                                              </td>}
                                          </tr>
                                        );
                                      })}
                                    {showDropDown === loan.loanAddress && <tr>
                                      <td>
                                        <div className="media-body">
                                          <span className="mb-0 text-sm">
                                            {" "}
                                            Collateral Amount
                                          </span>
                                        </div>
                                        <span>
                                          {activeLoan && activeLoan.collateralAmount}
                                          {activeLoan && getTokenByAddress[activeLoan.collateralAddress] && getTokenByAddress[activeLoan.collateralAddress].symbol}
                                        </span>
                                      </td>
                                      <td>
                                        <span>
                                          Loan Repaid {" "}
                                        </span>
                                        <span>
                                          {loanRepaid.toFixed(2) + "% "}
                                        </span>
                                      </td>
                                      <td>
                                        <span>
                                          Collateral Left {"   "}
                                        </span>
                                        <span>
                                          {activeLoan && activeLoan.collateralAmount}
                                          {activeLoan && getTokenByAddress[activeLoan.collateralAddress] && getTokenByAddress[activeLoan.collateralAddress].symbol}
                                        </span>
                                      </td>
                                      <td>
                                        <button
                                          className="btn btn-danger"
                                          type="button"
                                          dataToggle="tooltip"
                                          title="You can liquidate a portion of the loan’s collateral if the LTV exceeds 75%!"
                                          onClick={() => {
                                            LiquidateLoanCollateral(
                                              repayments[0].loanContractAddress
                                            );
                                          }}
                                        >
                                          Liquidate
                                        </button>
                                      </td>
                                    </tr>}
                                  </>
                                );
                              })}
                            </tbody>
                          </table>
                        </div>
                      )}
                      {/*<span className="alert-text">You haven't lent yet. Check the available loan request below!</span>*/}
                    </div>
                    <div className="btn-wrapper">
                      <a
                        href="/view-offers"
                        className="btn btn-primary btn-icon"
                        data-toggle="scroll"
                      >
                        <span className="btn-inner--text">View All Offers</span>
                      </a>
                      <a
                        href="/view-requests"
                        className="btn btn-primary btn-icon"
                      >
                        <span className="btn-inner--text">
                          View All Requests
                        </span>
                      </a>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </section>

          {approveRequestAlert && (
            <div className="alert alert-success" role="alert">
              <strong>Loan request approved successfully!</strong>
            </div>
          )}

          {transferCollateralAlert && (
            <div className="alert alert-success" role="alert">
              <strong>Transfer collateral successfully.</strong>
            </div>
          )}
        </div>
      </div>
    );
  }
}

// create `dropDown()` method
function DropDown(props) {
  const { id, show, loanAddress, duration, self, activeLoan, currentDate } = props;
  let { repayments } = self.state;
  return (
    <div className="dropdown">
      <button
        className="btn btn-info"
        onClick={async () => {
          self.setState({ repayments: [], showDropDown: null });
          repayments = await self.getActiveLoanRepayments(
            loanAddress,
            duration
          );
          self.setState({
            repayments: repayments, activeLoan: activeLoan,
            showDropDown: id
          });
          console.log("repayments - ", repayments);
          console.log("activeLoan - ", activeLoan);

          self.handleLoanRepaid(
            repayments, activeLoan, currentDate
          );
          self.viewMyLoans();
        }}
        aria-haspopup="true"
        aria-expanded="true"
      >
        Details
      </button>
    </div>
  );
}

// export class component Myloans
export default MyLoans;
```

***View All Requests component***

```
// Import libraries, services, styles and components 
import React, { Component } from 'react';
import Nouislider from "nouislider-react";
import Header from '../pages/Header';
import { GetLoans } from '../../services/loanbook';
import { supported_erc20_token, getTokenByAddress } from '../Web3/erc20';
import { GetLoanDetails, ApproveAndFundLoanRequest } from '../../services/loanContract';
import '../../assets/vendor/font-awesome/css/font-awesome.css';
import '../../assets/vendor/nucleo/css/nucleo.css';
import './ViewAllOffers.css';

// create a `ViewAllRequests` class Component imported from react
class ViewAllRequests extends Component {

  // create `constructor()` and call `super()` method of parent class
  constructor() {
    super();

    // call `viewAllRequest()` method
    this.viewAllRequest();

    // initialize the state variables
    this.state = {
      loanRequests: [],
      collateralMetadata: true,
      safeness: 'SAFE',
      expireIn: '5D 15H 30M',
      loanCurrency: 'ETH',
      collateralCurrency: 'ALL',
      waitingForLender: true,
      waitingForCollateral: false,
      waitingForPayback: false,
      finished: false,
      defaulted: false,
      minMonthlyInt: 0,
      maxMonthlyInt: 5,
      minDuration: 0,
      maxDuration: 12,
      erc20_tokens: [{ symbol: 'ALL' }, ...supported_erc20_token]
    };
  }

  //create `viewAllRequest()` method to get all Loans
  viewAllRequest = async () => {

    // try-catch block
    try {

      // assign `GetLoans()` method returned values to `loans`
      const loans = await GetLoans();

      // assign the `loanRequests` state variable to local variable
      let { loanRequests } = this.state;

      // traverse the loans and match the loan by address value
      loans.map(async (loanAddress) => {
        const loan = await GetLoanDetails(loanAddress);

        /*    if(loan[5].toNumber() === 2){
              for request/active i have changed it to >= 2
        */

        if (loan[4].toNumber() >= 2) {

          loanRequests.push({
            loanAddress: loanAddress,
            loanAmount: window.web3.fromWei(loan[0].toNumber()),
            duration: loan[1].toNumber(),
            interest: (loan[2].toNumber() / 100),
            collateral: {
              address: loan[5][0][0],
              amount: loan[7].toNumber()
            },
            status: loan[4].toNumber(),

          });

          console.log("loan reqests", loanRequests);
          this.setState({
            loanRequests: loanRequests
          });
        }
      });
    } catch (e) {
      console.log(e);
    } finally {

    }
  }

  // create a `convertDate()` method 
  convertDate = (currentDueDate, i) => {
    let date = new Date(currentDueDate * 1000)
    date.setMinutes(date.getMinutes() + ((i + 1) * 30));
    date = date.toString()
    return date;
  }

  // create `approveAndFundLoanRequest()` method
  approveAndFundLoanRequest = async (loanAmount, loanContractAddress) => {

    try {
      await ApproveAndFundLoanRequest(loanContractAddress, loanAmount);
    } catch (error) {
      console.log(error);
    }
  }


  // call `render()` method to display the frontend
  render() {

    // assign the state variables to their respective local variables
    const { erc20_tokens, minDuration, maxDuration, minMonthlyInt, maxMonthlyInt, collateralCurrency, loanRequests, waitingForLender, waitingForPayback, finished } = this.state;

    // call the `return()` method and create a parent `<div>`
    return (
      <div className="ViewAllRequests text-center">

        {/** call the `<Header>` component */}
        <Header />

        <div className="position-relative">
          <section className="section-hero section-shaped my-0">
            <div className="shape shape-style-1 shape-primary">
              <span className="span-150"></span>
              <span className="span-50"></span>
              <span className="span-50"></span>
              <span className="span-75"></span>
              <span className="span-100"></span>
              <span className="span-75"></span>
              <span className="span-50"></span>
              <span className="span-100"></span>
              <span className="span-50"></span>
              <span className="span-100"></span>
            </div>

            {/** create ViewAll requests components UI */}
            <div className="container d-flex align-items-left" style={{ marginLeft: '-15px' }}>
              <div className="card card-pricing border-0 col-md-4">
                <div className="card-header bg-transparent">
                  <i className="fa fa-filter" aria-hidden="true"></i>
                  <a className="ls-1 text-primary py-3 mb-0 ml-2">View All Requests</a>
                </div>
                <div className="card-body">
                  <ul className="list-unstyled my-4">
                    {/** choose Loan Currency filter */}
                    <li>
                      <div className="form-group">
                        <label for="exampleFormControlSelect1">Loan Currency</label>
                        <select className="form-control" id="exampleFormControlSelect1"
                          onClick={(e) => {
                            this.setState({ loanCurrency: e.target.value });
                          }}>
                          <option>ETH</option>;
                        </select>
                      </div>
                    </li>
                    {/** choose Collateral Currency filter*/}
                    <li>
                      <div className="form-group">
                        <label for="exampleFormControlSelect1">Collateral Currency</label>
                        <select className="form-control" id="exampleFormControlSelect1" onClick={(e) => {
                          this.setState({ collateralCurrency: e.target.value });
                        }}>
                          {
                            erc20_tokens.map((item, i) => {
                              return <option>{item.symbol}</option>;
                            })
                          }
                        </select>
                      </div>
                    </li>
                    {/** choose Loan State filter  */}
                    <li>
                      <div className="card">
                        <label for="">Loan state</label>
                        <div className="card-body">
                          <form>
                            <div className="row">
                              <div className="col text-left">
                                <div className="custom-control custom-checkbox mb-3">
                                  <input className="custom-control-input" id="customCheck1" type="checkbox" checked={this.state.waitingForLender}
                                    onClick={() => { this.setState({ waitingForLender: !this.state.waitingForLender }) }} />
                                  <label className="custom-control-label" for="customCheck1">Waiting for Lenders</label>
                                </div>
                                <div className="custom-control custom-checkbox mb-3">
                                  <input className="custom-control-input" id="customCheck3" type="checkbox" checked={this.state.waitingForPayback}
                                    onClick={() => { this.setState({ waitingForPayback: !this.state.waitingForPayback }) }} />
                                  <label className="custom-control-label" for="customCheck3">Waiting for Payback</label>
                                </div>
                                <div className="custom-control custom-checkbox mb-3">
                                  <input className="custom-control-input" id="customCheck4" type="checkbox" checked={this.state.finished}
                                    onClick={() => { this.setState({ finished: !this.state.finished }) }} />
                                  <label className="custom-control-label" for="customCheck4">Finished</label>
                                </div>
                              </div>
                            </div>
                          </form>
                        </div>
                      </div>
                    </li>
                    {/** select range of Monthly Interest */}
                    <li>
                      <div className="mt-3">
                        <label for="">Monthly Interest</label>
                        <div className="">
                          <label style={{ marginLeft: '-180px' }}> ({minMonthlyInt} %) </label>
                        </div>
                        <div className="" style={{ marginRight: '-180px', marginTop: '-30px' }}>
                          <label> ({maxMonthlyInt} %)</label>
                        </div>
                        {/** `Nouislider` component for Monthly Interest*/}
                        <Nouislider range={{ min: 0, max: 5 }} start={[0, 5]} connect
                          onChange={(e) => { this.setState({ minMonthlyInt: e[0], maxMonthlyInt: e[1] }); }} />
                      </div>
                    </li>
                    <li>
                      <div className="mt-3">
                        <label for="">Duration</label>
                        <div className="">
                          <label style={{ marginLeft: '-180px' }}> ({minDuration} Month) </label>
                        </div>
                        <div className="" style={{ marginRight: '-180px', marginTop: '-30px' }}>
                          <label> ({maxDuration} Month)</label>
                        </div>
                        {/** `Nouislider` component for Monthly Interest*/}
                        <Nouislider range={{ min: 0, max: 12 }} start={[0, 12]} connect
                          onChange={(e) => { this.setState({ minDuration: e[0], maxDuration: e[1] }); }} />
                      </div>
                    </li>
                  </ul>
                </div>
                {/** reset filters to default */}
                <div className="card-footer" style={{ marginTop: '-40px' }} onClick={() => {
                  this.setState({
                    loanCurrency: 'ETH',
                    collateralCurrency: 'ALL',
                    waitingForLender: true,
                    waitingForCollateral: false,
                    waitingForPayback: false,
                    finished: false,
                    defaulted: false,
                    minMonthlyInt: 0,
                    maxMonthlyInt: 5,
                    minDuration: 0,
                    maxDuration: 12,
                  })
                }}>
                  <a href="#!" className=" text-muted">Reset Filters</a>
                </div>
              </div>
              {/** map Loan Requests as per the filters */}
              <div className="ml-4 row">
                {
                  loanRequests.map((loanRequest) => {
                    return ((waitingForLender && loanRequest.status == 2) ||
                      (waitingForPayback && loanRequest.status == 3) ||
                      (finished && loanRequest.status == 4)) &&
                      (collateralCurrency == 'ALL' || getTokenByAddress[loanRequest.collateral.address].symbol == collateralCurrency) &&
                      loanRequest.duration / 30 > minDuration && loanRequest.duration / 30 < maxDuration &&
                      loanRequest.interest > minMonthlyInt && loanRequest.interest < maxMonthlyInt &&
                      <div className="col">
                        <div className="card">
                          <div className="card-header">
                            <div className="row row-example" style={{ fontSize: '60%' }}>
                              <div className="col-sm">
                                <span><p>Amount  </p></span>
                                <span className="btn-inner--text"><img style={{ width: '20px' }} src="/assets/img/eth.png" /> {loanRequest.loanAmount} ETH </span>
                              </div>
                              <div className="col-sm">
                                <span><p>Collateral </p></span>
                                <span className="btn-inner--text"> {loanRequest.collateral.amount} {getTokenByAddress[loanRequest.collateral.address] && getTokenByAddress[loanRequest.collateral.address].symbol}</span>
                              </div>
                            </div>
                          </div>
                          <div className="card-body text-left">
                            <p>Earnings : {loanRequest.interest} %</p>
                            <p>Duration  : {loanRequest.duration} days</p>
                          </div>
                          {/** if loanRequest status is active the approve and fund Loan Request */}
                          {loanRequest.status == 2 &&
                            <div className="btn-wrapper text-center"
                              onClick={() => this.approveAndFundLoanRequest(loanRequest.loanAmount, loanRequest.loanAddress)}>
                              <a href="#" className="btn btn-primary btn-icon m-1">
                                <span className="btn-inner--text">Fund Now</span>
                              </a>
                            </div>}
                        </div>
                        {/** Loan Request status Alert*/}
                        <div
                          className="alert alert-primary alert-dismissible fade show text-center"
                          role="alert"
                        >
                          <span className="alert-text">{loanRequest.status == 2 ? 'Waiting for lender' : loanRequest.status == 3 ? 'Waiting for payback' : 'Finished'}</span>
                        </div>
                      </div>;
                  })
                }
              </div>
            </div>
          </section>
        </div>
      </div>
    );
  }
}

// export the `ViewAllRequests` component
export default ViewAllRequests;
```

***View All Offer Component***
```
// import libraries, services, styles and components 
import React, { Component } from "react";
import Nouislider from "nouislider-react";
import { GetLoans, FetchCollateralPrice } from "../../services/loanbook";
import {
  GetLoanDetails,
  GetRepaymentData,
  AcceptLoanOffer,
  FinalizeCollateralTransfer
} from "../../services/loanContract";
import { ExecuteTokenApproval } from '../../services/token';
import { supported_erc20_token, getTokenBySymbol, getTokenByAddress } from '../Web3/erc20';
import SweetAlert from "react-bootstrap-sweetalert";
import "./ViewAllOffers.css";
import Header from "./Header";

// create a class Component imported from react
class ViewAllOffers extends Component {

  //create `constructor()` and call `super()` method of parent class
  constructor() {
    super();

    // call `viewAllOffers()` method
    this.viewAllOffers();

    // initialize the state variables
    this.state = {
      loanOffers: [],
      collateralMetadataAlert: false,
      transferCollateralAlert: false,
      acceptCollateralAlert: false,
      approveCollateralAlert: false,
      safeness: 'SAFE',
      expireIn: '5D 15H 30M',
      waitingForBorrower: true,
      waitingForCollateral: false,
      waitingForPayback: false,
      finished: false,
      minMonthlyInt: '0',
      maxMonthlyInt: '5',
      minDuration: '0',
      maxDuration: '12',
      defaulted: false,
      collateralCurrencyToken: '',
      collateralAddress: "",
      loanAddress: '',
      activeLoanOffer: [],
      activeCollateralValue: 0,
      loanCurrency: '',
      collateralCurrency: 'ALL',
      erc20_tokens: [{ symbol: 'ALL' }, ...supported_erc20_token],
      loanOfferCount: 1
    };
  }

  // create `viewAllOffers()` method to get all Loans
  viewAllOffers = async () => {

    // try-catch block
    try {

      // assign `GetLoans()` method returned values to `loans`
      const loans = await GetLoans();

      // assign the `loanOffers` state variable to local variable
      let { loanOffers } = this.state;

      // traverse the loans and match the loan by address value
      for (const loanAddress of loans) {
        const loan = await GetLoanDetails(loanAddress);

        if (loan[5].toNumber() > 0) {
          let collaterals = [];
          let finished = 'waiting';

          for (var i in loan[13]) {
            collaterals.push({
              address: loan[13][i][0].split('000000000000000000000000')[0],
              ltv: window.web3.toBigNumber(loan[13][i][1]).toNumber(),
              mpr: window.web3.toBigNumber(loan[13][i][2]).toNumber() / 100,
              collateralCurrency: loan[13][i][3],
            });
          }
          if (loan[5].toNumber() > 2) {
            finished = true;
            let repayments = [];
            let borrower = loan[10];
            let currentDate = new Date();
            let startedOn = loan[4].toNumber();
            let duration = loan[1].toNumber();


            repayments = await this.getActiveLoanRepayments(loanAddress, duration);

            let dueDate = this.convertDateEpoc(startedOn,
              repayments[(duration / 30) - 1].repaymentNumber - 1
            );


            repayments.map((repay) => {
              if (repay.repayee !== borrower &&
                currentDate > dueDate) {
                finished = false;
              }
            })
          }

          loanOffers.push({
            loanAddress: loanAddress,
            loanAmount: window.web3.fromWei(loan[0].toNumber()),
            duration: loan[1].toNumber(),
            interest: loan[2].toNumber() / 100,
            collaterals: collaterals,
            status: loan[5].toNumber(),
            finished: finished
          });
        }
      }

      console.log('loanOffers >>>>', loanOffers);


      this.setState({ loanOffers });
    } catch (e) {
      console.error(e);
    }
  };

  // create `getActiveLoanRepayments()` method
  getActiveLoanRepayments = async (loanAddress, duration) => {

    // assign the `repayments` state variable to local variable
    let { repayments } = this.state;

    // create a new repayments array
    repayments = new Array();

    // try-catch block
    try {

      // calculate number of repayments
      let totalNumberOfRepayments = duration / 30;

      for (var i = 1; i <= totalNumberOfRepayments; i++) {
        const repaymentData = await GetRepaymentData(loanAddress, i);
        repayments.push(repaymentData);
      }

      return repayments;
    } catch (error) {
      console.log(error);
    }
  };

  // create `handleAcceptLoanOffer()` method
  handleAcceptLoanOffer = async loanContractAddress => {

    // try-catch block
    try {

      // call the `AcceptLoanOffer()` method with selected collateral address
      await AcceptLoanOffer(loanContractAddress);
      return true;
    } catch (e) {
      console.log(e);
      return false;
    } finally {
    }
  };

  // create `handleCollateralTransfer()` method
  handleCollateralTransfer = async (loanContractAddress, collateralCurrencyToken) => {

    // assign collateralAddress value
    let collateralAddress = getTokenBySymbol[collateralCurrencyToken] && getTokenBySymbol[collateralCurrencyToken].address;

    //try-catch block
    try {

      // call `FinalizeCollateralTransfer()` method
      await FinalizeCollateralTransfer(loanContractAddress, collateralAddress);
    } catch (e) {
      console.log(e);
    } finally {
    }
  };

  // create `handleCollateralConversion()` method
  handleCollateralConversion = async (activeLoanOffer, collateralCurrencyToken) => {

    // add LTV ratio of collateral as third argument to this function or use the default value 200
    let ltv = 200;

    // assign collateral address value
    let collateralAddress = getTokenBySymbol[collateralCurrencyToken] && getTokenBySymbol[collateralCurrencyToken].address;

    // traverse `activeLoanOffer` array and set state variables
    activeLoanOffer.collaterals.map((item, i) => {
      if (getTokenByAddress[item.address] && getTokenByAddress[item.address].symbol === collateralCurrencyToken) {
        ltv = item.ltv;
        this.setState({ collateralAddress: collateralAddress, collateralCurrencyToken: collateralCurrencyToken });
      }
    })

    // try-catch block
    try {

      // assign `FetchCollateralPrice()` value to collateralPrice
      const collateralPrice = await FetchCollateralPrice({
        collateralAddress: collateralAddress
      });


      /* while assigning state variable `activeCollateralValue` and calculating collateralValue, this formula is used
      ((window.web3.toWei(loanAmount) / window.web3.toWei(collateralPrice))* (ltv/100))*/
      this.setState({
        activeCollateralValue: ((window.web3.toWei(activeLoanOffer.loanAmount) / (window.web3.toWei(collateralPrice)) * (ltv / 100)))
      })

    } catch (error) {

    }
  }

  // create `hideAlertCancel()` method to hide the cancel option
  hideAlertCancel = () => {
    this.setState({ collateralMetadataAlert: false });
  };

  // create `hideAlertConfirm()` method to hide the confirm option
  hideAlertConfirm = () => {
    this.setState({
      collateralMetadataAlert: false,
      acceptCollateralAlert: true
    });
  };

  // create `hideAlertAcceptCollateralCancel()` method to hide the Accept Collateral's Cancel option
  hideAlertAcceptCollateralCancel = () => {
    this.setState({ acceptCollateralAlert: false });
  };

  // create `hideAlertTransferCollateralCancel()` method to hide the Transfer Collateral's Cancel option
  hideAlertTransferCollateralCancel = () => {
    this.setState({ transferCollateralAlert: false });
  };

  // create `hideAlertAcceptCollateralConfirm()` method to hide the Accept Collateral's Cancel option
  hideAlertAcceptCollateralConfirm = async () => {
    const { loanAddress } = this.state;
    const transferCollateralAlert = await this.handleAcceptLoanOffer(loanAddress);
    this.setState({ acceptCollateralAlert: false, approveCollateralAlert: true });
  }

  // create `hideAlertTransferCollateralConfirm()` method to hide the Transfer Collateral's Confirm option
  hideAlertTransferCollateralConfirm = async () => {
    let { loanAddress, collateralCurrencyToken } = this.state;
    this.setState({ transferCollateralAlert: false });
    this.handleCollateralTransfer(loanAddress, collateralCurrencyToken);
  }

  // create `hideAlertAppproveCollateralConfirm()` method to hide the Approve Collateral's Confirm option
  hideAlertAppproveCollateralConfirm = async (collateralCurrencyToken, loanContractAddress, collateralValue) => {
    let collateralAddress = getTokenBySymbol[collateralCurrencyToken] && getTokenBySymbol[collateralCurrencyToken].address;
    try {
      await ExecuteTokenApproval({
        ERC20Token: collateralAddress,
        loanContractAddress: loanContractAddress,
        tokenAmount: collateralValue
      });

      this.setState({
        approveCollateralAlert: false,
        transferCollateralAlert: true
      });
    } catch (error) {
      console.log(error);
    }
  }

  // create `hideAlertApproveCollateralCancel()` method to hide the Approve Collateral's Cancel option
  hideAlertApproveCollateralCancel = async (collateralAddress, loanContractAddress, collateralValue) => {
    this.setState({ approveRequestAlert: false, acceptCollateralAlert: false });
    console.log("collateralAddress : ", collateralAddress);
  }

  // create `handleTakeThisLoan()` method to handle the take loan
  handleTakeThisLoan = (loanOffer) => {
    this.setState({ collateralMetadataAlert: true, loanAddress: loanOffer.loanAddress, activeLoanOffer: loanOffer });
  }

  /** create `convertDateEpoc()` method */
  convertDateEpoc = (currentDueDate, i) => {
    let date = new Date(currentDueDate * 1000);
    date.setMinutes(date.getMinutes() + (i + 1) * 30);
    return date;
  };

  // call `render()` method to display the frontend
  render() {

    // assign the state variables to their respective local variables
    let {
      erc20_tokens,
      minDuration,
      maxDuration,
      collateralMetadataAlert,
      transferCollateralAlert,
      acceptCollateralAlert,
      collateralCurrencyToken,
      activeLoanOffer,
      activeCollateralValue,
      loanAddress,
      approveCollateralAlert,
      loanOffers,
      maxMonthlyInt,
      minMonthlyInt,
      waitingForPayback,
      finished,
      defaulted,
      waitingForBorrower,
      collateralCurrency,
      loanOfferCount
    } = this.state;

    // call `render()` method to display the frontend
    return (
      <div className="ViewAllOffers text-center">

        {/** call the `<Header>` component */}
        <Header />

        <div className="position-relative">
          <section className="section-hero section-shaped my-0">
            <div className="shape shape-style-1 shape-primary">
              <span className="span-150"></span>
              <span className="span-50"></span>
              <span className="span-50"></span>
              <span className="span-75"></span>
              <span className="span-100"></span>
              <span className="span-75"></span>
              <span className="span-50"></span>
              <span className="span-100"></span>
              <span className="span-50"></span>
              <span className="span-100"></span>
            </div>

            {/** create ViewAll Offers components UI */}
            <div
              className="container d-flex align-items-left"
              style={{ marginLeft: "-15px" }}
            >
              <div className="card card-pricing border-0 col-md-4">
                <div className="card-header bg-transparent">
                  <i className="fa fa-filter" aria-hidden="true"></i>
                  <a className="ls-1 text-primary py-3 mb-0 ml-2">
                    View All Offers
                  </a>
                </div>
                <div className="card-body">
                  <ul className="list-unstyled my-4">
                    {/** Select Loan crrency */}
                    <li>
                      <div className="form-group">
                        <label for="exampleFormControlSelect1">
                          Loan Currency
                        </label>
                        <select
                          className="form-control"
                          id="exampleFormControlSelect1"
                          onClick={e => {
                            this.setState({
                              loanCurrency: e.target.value
                            });
                          }}
                        >
                          <option>ETH</option>;
                        </select>
                      </div>
                    </li>
                    {/** Select Collateral Currency */}
                    <li>
                      <div className="form-group">
                        <label for="exampleFormControlSelect1">
                          Collateral Currency
                        </label>
                        <select
                          className="form-control"
                          id="exampleFormControlSelect1"
                          onClick={e => {
                            this.setState({
                              collateralCurrency: e.target.value
                            });
                          }}
                        >
                          {erc20_tokens.map((item, i) => {
                            return <option>{item.symbol}</option>;
                          })}
                        </select>
                      </div>
                    </li>
                    {/** set Loan state */}
                    <li>
                      <div className="card">
                        <label for="">Loan state</label>
                        <div className="card-body">
                          <form>
                            <div className="row">
                              <div className="col text-left">
                                <div className="custom-control custom-checkbox mb-3 ">
                                  <input
                                    className="custom-control-input"
                                    id="customCheck1"
                                    type="checkbox"
                                    checked={this.state.waitingForBorrower}
                                    onClick={() => {
                                      this.setState({
                                        waitingForBorrower: !this.state
                                          .waitingForBorrower
                                      });
                                    }}
                                  />
                                  {/** Loan state: Waiting for Borrowers UI */}
                                  <label
                                    className="custom-control-label"
                                    for="customCheck1"
                                  >
                                    Waiting for Borrowers
                                  </label>
                                </div>

                                {/** Loan state: Waiting for Payback UI */}
                                <div className="custom-control custom-checkbox mb-3 ">
                                  <input
                                    className="custom-control-input"
                                    id="customCheck3"
                                    type="checkbox"
                                    checked={this.state.waitingForPayback}
                                    onClick={() => {
                                      this.setState({
                                        waitingForPayback: !this.state
                                          .waitingForPayback
                                      });
                                    }}
                                  />
                                  <label
                                    className="custom-control-label"
                                    for="customCheck3"
                                  >
                                    Waiting for Payback
                                  </label>
                                </div>
                                {/** Loan state: Finished UI */}
                                <div className="custom-control custom-checkbox mb-3 ">
                                  <input
                                    className="custom-control-input"
                                    id="customCheck4"
                                    type="checkbox"
                                    checked={this.state.finished}
                                    onClick={() => {
                                      this.setState({
                                        finished: !this.state.finished
                                      });
                                    }}
                                  />
                                  <label
                                    className="custom-control-label"
                                    for="customCheck4"
                                  >
                                    Finished
                                  </label>
                                </div>{" "}
                                {/** Loan state: Finished UI */}
                                <div className="custom-control custom-checkbox mb-3 ">
                                  <input
                                    className="custom-control-input"
                                    id="customCheck5"
                                    type="checkbox"
                                    checked={this.state.defaulted}
                                    onClick={() => {
                                      this.setState({
                                        defaulted: !this.state.defaulted
                                      });
                                    }}
                                  />
                                  <label
                                    className="custom-control-label"
                                    for="customCheck5"
                                  >
                                    Defaulted
                                  </label>
                                </div>
                              </div>
                            </div>
                          </form>
                        </div>
                      </div>
                    </li>
                    {/** Select minimum and maximum Montly Interest */}
                    <li>
                      <div className="mt-3">
                        <label for="">Monthly Interest</label>
                        <div className="">
                          <label style={{ marginLeft: "-180px" }}>
                            {" "}
                            ({minMonthlyInt} %){" "}
                          </label>
                        </div>
                        <div
                          className=""
                          style={{ marginRight: "-180px", marginTop: "-30px" }}
                        >
                          <label> ({maxMonthlyInt} %)</label>
                        </div>
                        {/** Nouislider component */}
                        <Nouislider
                          range={{ min: 0, max: 5 }}
                          start={[0, 5]}
                          connect
                          onChange={e => {
                            this.setState({
                              minMonthlyInt: e[0],
                              maxMonthlyInt: e[1]
                            });
                          }}
                        />
                      </div>
                    </li>
                    {/** Select minimum and maximum loan duration */}
                    <li>
                      <div className="mt-3">
                        <label for="">Duration</label>
                        <div className="">
                          <label style={{ marginLeft: "-180px" }}>
                            {" "}
                            ({this.state.minDuration} Month){" "}
                          </label>
                        </div>
                        <div
                          className=""
                          style={{ marginRight: "-180px", marginTop: "-30px" }}
                        >
                          <label> ({this.state.maxDuration} Month)</label>
                        </div>
                        {/** Nouislider component */}
                        <Nouislider
                          range={{ min: 0, max: 12 }}
                          start={[0, 12]}
                          connect
                          onChange={e => {
                            this.setState({
                              minDuration: e[0],
                              maxDuration: e[1]
                            });
                            console.log(this.state.maxDuration);
                          }}
                        />
                      </div>
                    </li>
                  </ul>
                </div>
                {/**Reset UI filters */}
                <div className="card-footer">
                  <a href="#!" className=" text-muted">
                    Reset Filters
                  </a>
                </div>
              </div>
              {/** map Loan Offers as per the filters */}
              <div className="ml-4 row">
                {loanOffers.map((loanOffer, index) => (
                  ((waitingForBorrower && loanOffer.status == 1) || (waitingForPayback && loanOffer.status == 3 && loanOffer.finished === 'waiting') || (finished && loanOffer.status == 3 && loanOffer.finished) || (defaulted && loanOffer.status == 3 && !loanOffer.finished)) &&
                    (loanOffer.duration / 30 > minDuration && loanOffer.duration / 30 < maxDuration) &&
                    ((loanOffer.collaterals[0] && loanOffer.collaterals[0].mpr > minMonthlyInt && loanOffer.collaterals[0].mpr < maxMonthlyInt) ||
                      (loanOffer.collaterals[1] && loanOffer.collaterals[1].mpr > minMonthlyInt && loanOffer.collaterals[1].mpr < maxMonthlyInt) ||
                      (loanOffer.collaterals[2] && loanOffer.collaterals[2].mpr > minMonthlyInt && loanOffer.collaterals[2].mpr < maxMonthlyInt) ||
                      (loanOffer.collaterals[3] && loanOffer.collaterals[3].mpr > minMonthlyInt && loanOffer.collaterals[3].mpr < maxMonthlyInt) ||
                      (loanOffer.collaterals[4] && loanOffer.collaterals[4].mpr > minMonthlyInt && loanOffer.collaterals[4].mpr < maxMonthlyInt) ||
                      (loanOffer.collaterals[5] && loanOffer.collaterals[5].mpr > minMonthlyInt && loanOffer.collaterals[5].mpr < maxMonthlyInt)) &&
                    ((collateralCurrency == 'ALL') || ((loanOffer.collaterals[0] && getTokenByAddress[loanOffer.collaterals[0].address] && getTokenByAddress[loanOffer.collaterals[0].address].symbol == collateralCurrency) ||
                      (loanOffer.collaterals[1] && getTokenByAddress[loanOffer.collaterals[1].address] && getTokenByAddress[loanOffer.collaterals[1].address].symbol == collateralCurrency) ||
                      (loanOffer.collaterals[2] && getTokenByAddress[loanOffer.collaterals[2].address] && getTokenByAddress[loanOffer.collaterals[2].address].symbol == collateralCurrency) ||
                      (loanOffer.collaterals[3] && getTokenByAddress[loanOffer.collaterals[3].address] && getTokenByAddress[loanOffer.collaterals[3].address].symbol == collateralCurrency) ||
                      (loanOffer.collaterals[4] && getTokenByAddress[loanOffer.collaterals[4].address] && getTokenByAddress[loanOffer.collaterals[4].address].symbol == collateralCurrency) ||
                      (loanOffer.collaterals[5] && getTokenByAddress[loanOffer.collaterals[5].address] && getTokenByAddress[loanOffer.collaterals[5].address].symbol == collateralCurrency))) && (loanOfferCount++) ?
                    <div key={index} className={loanOfferCount > 3 ? "col-md-4" : "col"}>
                      <div className="card">
                        <div className="card-header">
                          <div className="row row-example">
                            <div className="mx-auto mb-2">
                              {loanOffer.collaterals.map((collateral) => {
                                return getTokenByAddress[collateral.address] && <img src={`/assets/img/32/color/` + (getTokenByAddress[collateral.address] && getTokenByAddress[collateral.address].symbol) + `.png`} style={{ width: '30px' }} />;
                              })
                              }
                            </div>
                          </div>
                          <div
                            className="ml-3"
                            style={{ fontSize: "x-small" }}
                          >
                            LTV {loanOffer.collaterals.map((item) => {
                              return item.ltv + "% ";
                            })}

                          </div>
                          <div
                            className="ml-3"
                            style={{ fontSize: "x-small" }}
                          >
                            MPR {loanOffer.collaterals.map((item) => {
                              return item.mpr + "% ";
                            })}

                          </div>
                        </div>
                        <div className="card-body ">
                          <p style={{ fontSize: "small" }}>Duration  : {loanOffer.duration} days</p>
                          <p style={{ fontSize: "small" }}>Amount  : {loanOffer.loanAmount} ETH</p>

                          {/** if loanOffer status is active, then take this loan*/}
                          {loanOffer.status === 1 && <div className="btn-wrapper text-center" onClick={() => {
                            this.handleTakeThisLoan(loanOffer)
                          }}>
                            <a href="#" className="btn btn-primary btn-icon mt-2">
                              <span className="btn-inner--text">
                                Take this loan
                              </span>
                            </a>
                          </div>}
                        </div>
                      </div>
                      <div
                        className="alert alert-primary alert-dismissible fade show text-center"
                        role="alert"
                      >
                        <span className="alert-text">{(loanOffer.status == 1 && waitingForBorrower) ? 'Waiting for borrower'
                          : (loanOffer.status == 3 && waitingForPayback && loanOffer.finished == 'waiting') ? 'Waiting for payback'
                            : (loanOffer.status == 3 && finished && loanOffer.finished) ? 'Finished'
                              : (loanOffer.status == 3 && defaulted && !loanOffer.finished) ? 'Defaulted'
                                : 'Waiting for payback'}</span>
                      </div>
                    </div>
                    : ''
                ))}
                {/** Loan offer status SweetAlert components*/}
                {collateralMetadataAlert && (
                  <SweetAlert
                    info
                    showCancel
                    confirmBtnText="Confirm"
                    cancelBtnText="Cancel"
                    confirmBtnBsStyle="info"
                    cancelBtnBsStyle="default"
                    customIcon="thumbs-up.jpg"
                    title="Take this loan?"
                    onConfirm={this.hideAlertConfirm}
                    onCancel={this.hideAlertCancel}
                  >
                    <div
                      className="col-md-5 form-group mt-5"
                      style={{ marginLeft: "27%" }}
                    >
                      <label for="exampleFormControlSelect1">
                        Select collateral
                      </label>
                      {/** select collateral and handle its conversion in native currency say Matic/ETH */}
                      <select
                        className="form-control"
                        style={{ width: "80px", display: "inline" }}
                        onClick={e => {
                          this.handleCollateralConversion(activeLoanOffer, e.target.value);
                        }}
                      >
                        {activeLoanOffer.collaterals.map((item, i) => {
                          return <option>{getTokenByAddress[item.address] && getTokenByAddress[item.address].symbol}</option>;
                        })}
                      </select>

                      {activeLoanOffer.collaterals.map((item, i) => {
                        return (getTokenByAddress[item.address] && getTokenByAddress[item.address].symbol === collateralCurrencyToken) && <label for="exampleFormControlSelect1" key={i}>
                          MPR : {item.mpr} &nbsp; LTV : {item.ltv}%
                        </label>;
                      })}
                    </div>
                  </SweetAlert>
                )}

                {/** acceptCollateral status SweetAlert component*/}
                {acceptCollateralAlert && (
                  <SweetAlert
                    warning
                    showCancel
                    confirmBtnText="Accept"
                    confirmBtnBsStyle="info"
                    cancelBtnBsStyle="default"
                    title="Accept Loan Offer"
                    onConfirm={this.hideAlertAcceptCollateralConfirm}
                    onCancel={this.hideAlertAcceptCollateralCancel}
                  >
                    Accept Transfer collateral of {activeCollateralValue}{" "}
                    {collateralCurrencyToken} tokens
                  </SweetAlert>
                )}
                {/** approveCollateral status SweetAlert component*/}
                {approveCollateralAlert && (
                  <SweetAlert
                    warning
                    showCancel
                    confirmBtnText="Approve"
                    confirmBtnBsStyle="info"
                    cancelBtnBsStyle="default"
                    title="Approve Loan Offer"
                    onConfirm={() => { this.hideAlertAppproveCollateralConfirm(collateralCurrencyToken, loanAddress, activeCollateralValue) }}
                    onCancel={this.hideAlertApproveCollateralCancel}
                  >
                    Approve Transfer collateral of {activeCollateralValue}{" "}
                    {collateralCurrencyToken} tokens
                  </SweetAlert>
                )}
                {/** transferCollateral status SweetAlert component*/}
                {transferCollateralAlert && (
                  <SweetAlert
                    info
                    showCancel
                    confirmBtnText="Transfer"
                    confirmBtnBsStyle="info"
                    cancelBtnBsStyle="default"
                    title="Transfer Collateral"
                    onConfirm={this.hideAlertTransferCollateralConfirm}
                    onCancel={this.hideAlertTransferCollateralCancel}
                  >
                    Transfer collateral of {activeCollateralValue} {collateralCurrencyToken} tokens
                  </SweetAlert>
                )}
              </div>
            </div>
          </section>
        </div>
      </div>
    );
  }
}

// export class component `ViewAllOffers`
export default ViewAllOffers;
```

***Save the respected code and run the app to interact with it in the browser:***
```
npm start
```

# Conclusion

Now you know about creating a Lending Marketplace with Truffle Suite and ReactJS on the Polygon network.

If you had any difficulties following this tutorial or simply want to discuss Polygon tech with us you can [**join our community today**](https://community.figment.io/) or [**Join our discord channel**](https://discord.gg/fszyM7K)!

# About the author

[Devendra Yadav](https://community.figment.io/u/dev.koold)

# References
- https://github.com/crypto-lend
