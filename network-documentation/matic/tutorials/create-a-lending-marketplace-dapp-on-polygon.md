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

Add the smart contract code below:

LoanContract.sol

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

LoanCreator.sol

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

Web3Service.js
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

Now, setup `token.js`:

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

Create Loan Contract methods inside `loanContract.js`:

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

//Create `FinalizeCollateralTransfer()` method for final transfer of collateral to selected loan
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

Setup Loan Book methods inside `loanbook.js`:
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

Setup `App.js` in the following manner and divide it into components:

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

Create `components/` directory and setup separate UI components:

## LoanRequest component
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

Run the app and view it in the browser:
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
