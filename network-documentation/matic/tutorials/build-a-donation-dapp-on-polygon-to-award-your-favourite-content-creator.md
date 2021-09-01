# Introduction

In this tutorial, you will learn how to create Donation Dapp on Polygon to award your favorite content creator and how to deploy your smart contract on the Polygon network.

This is the app we will be creating at the end of this tutorial.

![Dapp Demo](../../../.gitbook/assets/dapp-demo.gif)

# Prerequisites

To follow along with this tutorial you will need basic knowledge and understanding of Blockchain, Smart contracts and React(NextJS) framework for frontend.

## Requirements

- We will need Metamask installed in our browser from [HERE](https://metamask.io/).
- Make sure to have NodeJS 12.0.1+ version installed.

#### Techstack used in this tutorial

- [Solidity](https://docs.soliditylang.org/en/v0.8.7/) - For writing smart contracts
- [Truffle](https://github.com/trufflesuite/truffle) - Truffle provides development environment for developing blockchain applications locally
- [NextJS](https://nextjs.org/) - To Create frontend for our Dapp
- [Web3.js](https://web3js.readthedocs.io/en/v1.4.0/) - Javascript librabry to connect our frontend to smart contract
- [Polygon Network](https://polygon.technology/) - For deploying our smart contract
- [IPFS](https://ipfs.io/) - To store image and videos uploaded by content creators

# Things we will be covering in this article

- Setting up development for Solidity and NextJs
- Creating smart contracts using Solidity
- Compiling and migrating smart contracts using truffle
- Writing tests for our smart contract
- Linking smart contract to frontend using web3js
- Creating UI using TailwindCSS in NextJs
- Using IPFS to upload images'
- Publishing the smart contract to Polygon Testnet

## Setting up development for Solidity and NextJs

To set up solidity and nextjs we will have to install few packages using `npm`. Make sure you have `npm` installed in your system and then run the following commands to install the packages and create project directories.

```bash
npm install -g truffle # Install truffle globally so that you can use truffle from any directory

npx create-next-app --typescript # Install nextjs and setup typescript for your project
# When asked put project-name as polygon-dapp so that it would be easy for you to follow along with this tutorial

cd polygon-dapp
truffle init # Create truffle-config.js test/ and contracts/ directory
```

This is how folder structure will be after the setup.

![Folder Structure](../../../.gitbook/assets/folder-structure.png)

`truffle init` command will create the following directories,

- `contracts/` : All the smart contracts are created in `contracts/` directory.
- `migrations/`: All migrations scripts used by truffle to deploy smart contracts are stored in the `migrations/` directory.
- `test/`: All the test scripts for smart contracts are places in the `test` directory.
- `truffle-config.js`: Contains configuration settings for truffle.

## Creating smart contract using Solidity

Create the `DonationContract.sol` file in the `contracts/` directory and add the following code to the same file.

```sol
// SPDX-License-Identifier: GPL-3.0
pragma  solidity  ^0.5.16;

contract  DonationContract {
  // Keep track of total number of images in contract
  uint256  public imageCount =  0;

  // Data structure to store images data
  struct  Image {
    uint256 id;
    string  hash;
    string description;
    uint256 donationAmount;
    address  payable author;
  }

  mapping(uint256  => Image) public images;

  // Event emitted when image is created
  event  ImageCreated(
    uint256  id,
    string  hash,
    string  description,
    uint256  donationAmount,
    address  payable  author
  );

  // Event emitted when an there is a donation
  event  DonateImage(
    uint256  id,
    string  hash,
    string  description,
    uint256  donationAmount,
    address  payable  author
  );
}
```

- `imageCount` is an unsigned public integer that stores the total number of images in a smart contract.
- `struct Image` is a data structure to store image hash and metadata related to that image. `Image` struct contains the id of the image, IPFS hash for that image, description added by the content creator, total donation amount received on that image, and the author address where the donation will be sent to.
- `mapping(uint256 => Image) public images` is a map that stores all the images, where the key is the `id` and value is the `Image struct`.
- `ImageCreated` and `DonateImage` are the events emitted by blockchain so that our Dapps can listen to it and function accordingly.

```
// Create an Image
function  uploadImage(string  memory  _imgHash, string  memory  _description) public {
  require(bytes(_imgHash).length >  0);
  require(bytes(_description).length >  0);
  require(msg.sender !=  address(0x0));

  imageCount++;
  images[imageCount] =  Image(
    imageCount,
    _imgHash,
    _description,
    0,
    msg.sender
  );

  emit  ImageCreated(imageCount, _imgHash, _description, 0, msg.sender);
}
```

- `uploadImage` function accepts image hash and description as parameters. In the function first, we make sure the `imageHash` and `description` are not empty and the sender address is not empty. Then we increment the `imageCount` by one and create a new Image struct object and store it in the `images` map with `imageCount` as key.
- `msg` is a global variable that contains the address of the person who called the function and in our case the author of the content. So to store the address of `author` we can directly use `msg.sender`
- Once the object is stored in the map we can emit the `ImageCreated` event with relevant data.

```
function  donateImageOwner(uint256  _id) public  payable {
  require(_id >  0  && _id <= imageCount);

  Image memory _image = images[_id];
  address  payable _author = _image.author;
  address(_author).transfer(msg.value);
  _image.donationAmount = _image.donationAmount +  msg.value;
  images[_id] = _image;

  emit  DonateImage(
    _id,
    _image.hash,
    _image.description,
    _image.donationAmount,
    _author
  );
}
```

- `donateImageOwner` is a public payable function that accepts the id of the image. Since the function is `payable`, the global variable `msg` contains a field called `value` that has the number of coins which is sent for the donation.
- In the function first, we check `id` parameter is valid. Then we extract the image from the `images` map and store it in the `_image` variable, and from the `_image` variable we extract the address of the author where we will send the donation amount.
- Then we call the `transfer()` function on the `_author` address and pass in the `msg.value` which will transfer the amount of donation to the author's account.
- `images[_id] = _image` - After the donation is transfered to author's account, we can increment the `donationAmount` in `_image` struct and save the `_image` back to `images` map to update the data.
- Once everything is done we have to emit the `DonateImage` event.

That's all for the smart contract, now we can move forward with the compilation and migration process.

## Compiling and migrating smart contracts using truffle

To compile, open the terminal and run the following command,

```bash
truffle compile
```

- After running the command, you should the output similar to the image below,

![Truffle Compile](../../../.gitbook/assets/truffle-compile.png)

- For the migration of our contact to the blockchain, go to the `migrations/` directory, create a file `2_donation_contract_migration.js` and add the following code.

```
const DonationContract = artifacts.require("DonationContract");

module.exports = function (deployer) {
  deployer.deploy(DonationContract);
};
```

- If you notice in the `migrations/` directory, there is a file called `1_initial_migration.js` which handles the migration for the `Migrations.sol` contract which is created by default when we initialize truffle. Each contract needs a migration file to migrate it to blockchains.
- Before running the migration there are a couple of things that you have to set up. Firstly you need to install [Ganache](https://www.trufflesuite.com/ganache) on your system. Ganache is a GUI app that provides a local blockchain for you to work on. If you don't want a GUI app then you can install [Ganache CLI](https://www.npmjs.com/package/ganache-cli) instead. After installing start the Ganache on your system. Secondly, you have to modify the `truffle-config.js` file in the root directory of your project. You can delete all the content of `truffle-config.js` and replace it with the code below.

```js
module.exports = {
  networks: {
    development: {
      host: "localhost",
      port: 7545,
      network_id: "*",
    },
  },
  contracts_directory: "./contracts",
  contracts_build_directory: "./abis",
  compilers: {
    solc: {
      optimizer: {
        enabled: true,
        runs: 200,
      },
    },
  },

  db: {
    enabled: false,
  },
};
```

- Here we are defining the basic configuration which truffle will use to deploy/migrate our smart contract. Currently, we will be deploying to `localhost:7545` where our ganache blockchain is running.
- To Deploy to the development network, run the following command from the root directory of your project.

```bash
truffle migrate
```

This command will deploy your contract ganache, and will give you output that looks similar to this -

![Output Image](../../../.gitbook/assets/truffle-migrate.png)

## Writing tests for our smart contract

Writing a test for your smart contract is a very crucial and critical task. Once a block is added in blockchain it cannot be modified, if anything goes wrong then all the subsequent blocks will have issues, this makes it very important to test your code before deploying it to the mainnet.

When we ran `truffle init`, truffle had created `test/` directory in the root directory of our project. In the `test/` directory you can create a file with whatever name you prefer, let's go with `donation-contract-tests.js`.

> Note that truffle uses [chai](https://www.npmjs.com/package/chai) library to write tests.

Before starting with the test, you will need to install a couple of packages.

```bash
npm install chai chai-as-promised
```

Now in `test/donation-contract-tests.js` file add the following code,

```js
const { assert } = require("chai");
const DonationContract = artifacts.require("./DonationContract.sol");
require("chai").use(require("chai-as-promised")).should();

contract("DonationContract", ([deployer, author, donator]) => {
  let donationContract;
  before(async () => {
    donationContract = await DonationContract.deployed();
  });

  describe("deployment", () => {
    it("should be an instance of DonationContract", async () => {
      const address = await donationContract.address;
      assert.notEqual(address, null);
      assert.notEqual(address, 0x0);
      assert.notEqual(address, "");
      assert.notEqual(address, undefined);
    });
  });

  describe("Images", () => {
    let result;
    const hash = "abcd1234";
    const description = "This is a test image";
    let imageCount;
    before(async () => {
      result = await donationContract.uploadImage(hash, description, {
        from: author,
      });
      imageCount = await donationContract.imageCount();
    });

    it("Check Image", async () => {
      let image = await donationContract.images(1);
      assert.equal(imageCount, 1);
      const event = result.logs[0].args;
      assert.equal(event.hash, hash);
      assert.equal(event.description, description);
    });

    it("Allow users to donate", async () => {
      let oldAuthorBalance;
      oldAuthorBalance = await web3.eth.getBalance(author);
      oldAuthorBalance = new web3.utils.BN(oldAuthorBalance);
      result = await donationContract.donateImageOwner(imageCount, {
        from: donator,
        value: web3.utils.toWei("1", "Ether"),
      });

      const event = result.logs[0].args;
      let newAuthorBalance;
      newAuthorBalance = await web3.eth.getBalance(author);
      newAuthorBalance = new web3.utils.BN(newAuthorBalance);

      let donateImageOwner;
      donateImageOwner = web3.utils.toWei("1", "Ether");
      donateImageOwner = new web3.utils.BN(donateImageOwner);

      const expactedBalance = oldAuthorBalance.add(donateImageOwner);
      assert.equal(newAuthorBalance.toString(), expactedBalance.toSting());
    });
  });
});
```

- These tests are pretty much self-explanatory. Here we are describing two different tests, `deployment` and `Images`.
- In `deployment`, we are checking whether the contract is deployed properly and the address of the contract is valid.
- In the `Images` test, first, we are creating a sample image with an author, and then in the `Check Image` test case, we are fetching the image by passing `id` as 1 in `donationContract.images(1)` function. Then we can assert the values received by the event to check if every value is valid and correct.
- In the `Allow users to donate` test case, first, we are fetching the account balance of a user and then donating to the author of an image. After the donation is sent, we again fetch the updated balance of that user. Now we can check if the old balance and new balance should only the defer by the amount donated to the author.
- If you have noticed at the top of the test script, we have added an artifact code - `const DonationContract = artifacts.require("DonationContract.sol");`. These artifacts are added at the runtime by truffle, hence it is important to run our test using truffle. To run tests, execute the following command,

```bash
truffle test
```

The output of the above command should be similar to this -

![Output image](../../../.gitbook/assets/truffle-test.png)

## Linking smart contract to frontend using web3js

Now we are done with Smart contracts and tests, it's time to integrate our smart contracts with the frontend application.

For contract calls, we will create a context and provide all the data and functions through it. Create a folder `contexts/` in the root directory of your project and create a file called `DataContext.tsx` in that folder.

`context/DataContext.tsx`

```typescript
declare let window: any;
import { createContext, useContext, useEffect, useState } from "react";
import Web3 from "web3";
import DonationContract from "../abis/DonationContract.json";

interface DataContextProps {
  account: string;
  contract: any;
  loading: boolean;
  images: any[];
  imageCount: number;
  updateImages: () => Promise<void>;
  donateImageOwner: (id: string, donateAmout: any) => Promise<void>;
}

const DataContext = createContext<DataContextProps | null>(null);

export const DataProvider: React.FC = ({ children }) => {
  const data = useProviderData();
  return <DataContext.Provider value={data}>{children}</DataContext.Provider>;
};

export const useData = () => useContext<DataContextProps | null>(DataContext);
```

- At the top we are setting the type of `window` object to null, so that typescript would allow us to use functions like `window.web3` and `window.ethereum`.
- After that, we make necessary imports. If you don't have the `web3` package installed, you can install it by running - `npm install web3`.
- In the 4th line, we are fetching `DonationContract`, which is a JSON object that contains the ABI of our contract. This ABI and JSON file is created by truffle when we run the `truffle migrate` command. The ABI contains all the information like, address of your contract in blockchain, what are the functions present in the contract, and the parameters and returns types of these functions.
- Next, we have declared an interface for our DataContext, which contains all the variables and functions type which will be provided by DataContext.
- Then we have to create `DataContext` and `DataProvider`. We can also create a custom hook called `use data which we can later use in our components to easily access values from context.

- > If you want to learn about React Context and how it works, you can follow [this](https://reactjs.org/docs/context.html) from the official documentation of React.

```typescript
export const useProviderData = () => {
  const [loading, setLoading] = useState(true);
  const [images, setImages] = useState<any[]>([]);
  const [imageCount, setImageCount] = useState(0);
  const [account, setAccount] = useState("0x0");
  const [contract, setContract] = useState<any>();

  useEffect(() => {
    loadWeb3();
    loadBlockchainData();
  }, []);

  const loadWeb3 = async () => {
    if (window.ethereum) {
      window.web3 = new Web3(window.ethereum);
      await window.ethereum.enable();
    } else if (window.web3) {
      window.web3 = new Web3(window.web3.currentProvider);
    } else {
      window.alert("Non-Eth browser detected. Please consider using MetaMask.");
    }
  };

  const loadBlockchainData = async () => {
    const web3 = window.web3;
    var allAccounts = await web3.eth.getAccounts();
    setAccount(allAccounts[0]);
    const networkData = DonationContract.networks["5777"];
    if (networkData) {
      var tempContract = new web3.eth.Contract(
        DonationContract.abi,
        networkData.address
      );
      setContract(tempContract);
      var count = await tempContract.methods.imageCount().call();
      setImageCount(count);
      var tempImageList = [];
      for (var i = 1; i <= count; i++) {
        const image = await tempContract.methods.images(i).call();
        tempImageList.push(image);
      }
      tempImageList.reverse();
      setImages(tempImageList);
    } else {
      window.alert("TestNet not found");
    }
    setLoading(false);
  };
};
```

- Create a function called `useProviderData` in the same `DataContext.tsx` file. `useProviderData` is being called inside `DataProvider`, hence everytime page loads `useProviderData` is called.
- In this function, first, we are declaring all the state variables that we required and then in the `useEffect` we are making two function call, `loadWeb3` and `loadBlockchainData`.
- In `loadWeb3`, we are first checking if `window.ethereum` is present which is usually injected by the wallet extension you are using. If window.ethereum is present then we set `window.web3` as the object of `Web3(window.ethereum)`, where `window.ethereum` is the provider for web3.
- If `window.ethereum` is not present then we check if `window.web3` is present, if yes then set `window.web3` as an object of `Web3(window.web3)`, where `window.web3` is the provider for web3.
- If none of these is present then it means the user does not have a wallet installed in their browser and in such case, we can show an alert.
- In the `loadBlockchainData` function, we are fetching all the necessary data we require like, the user's account number, contract object using ABI from the `DonationContract.json` file. Once we have the contract object, we can make necessary contract calls. In this function, we are fetching the total image count and all the images from our contract.

In the same `useProviderData` function add following functions.

```typescript
const updateImages = async () => {
  setLoading(true);
  if (contract !== undefined) {
    var count = await contract.methods.imageCount().call();
    setImageCount(count);
    var tempImageList = [];
    for (var i = 1; i <= count; i++) {
      const image = await contract.methods.images(i).call();
      tempImageList.push(image);
    }
    tempImageList.reverse();
    setImages(tempImageList);
  }
  setLoading(false);
};

const donateImageOwner = async (id: string, donateAmout: any) => {
  await contract.methods
    .donateImageOwner(id)
    .send({ from: account, value: donateAmout });
};

return {
  account,
  contract,
  loading,
  images,
  imageCount,
  updateImages,
  donateImageOwner,
};
```

- `updateImages` function just fetches the latest image count and images and sets the variables.
- `donateImageOwner` accepts the image id and donates an amount, then call the `donateImageOwner` function of the smart contract and pass in then parameters along with the `from` variable which will be the account number of the current user.
- In the end, return all the variables and functions from the `useProviderData` function.

Now we have add `DataProvider` in `_app.tsx` file. Head over to `pages/_app.tsx` and wrap `<Component {...pageProps} />` component with `<DataProvider>` component.

```tyepscript
import  "tailwindcss/tailwind.css";
import { DataProvider } from  "../contexts/DataContext";

function  MyApp({ Component, pageProps }) {
  return (
    <>
      <DataProvider>
        <Component {...pageProps} />
      </DataProvider>
    </>
  );
}

export  default  MyApp;
```

Now we can access all context variables in any of our components.

## Creating UI using TailwindCSS in NextJs

For UI we will be using [`tailwindcss`](https://tailwindcss.com/) and [`headlessui`](https://headlessui.dev/). To install these packages run the following command,

```bash
npm install tailwindcss @headlessui/react
```

Now in `pages/index.tsx` delete all the code and replace it with -

```typescript
import Head from "next/head";
import { useState } from "react";
import Body from "../components/Body";
import Header from "../components/Layout/Header";
import { UploadImage } from "../components/UploadImage";
import { useData } from "../contexts/DataContext";

export default function Home() {
  let [isOpen, setIsOpen] = useState(false);
  const { loading } = useData();

  function closeModal() {
    setIsOpen(false);
  }

  function openModal() {
    setIsOpen(true);
  }

  return (
    <div className="flex flex-col items-center justify-start min-h-screen py-2">
      <Head>
        <title>Decentragram</title>
        <link rel="icon" href="/favicon.ico" />
      </Head>
      <UploadImage isOpen={isOpen} closeModal={closeModal} />
      <Header />
      <div
        className="max-w-2xl w-full bg-blue-100 rounded-xl flex justify-center items-center py-2 mt-3 hover:bg-blue-200 cursor-pointer"
        onClick={openModal}
      >
        <span className="text-blue-500 font-bold text-lg">Upload Image</span>
      </div>
      {loading ? (
        <div className="font-bold text-gray-400 mt-36 text-4xl">Loading...</div>
      ) : (
        <Body />
      )}
    </div>
  );
}
```

- Here we are fetching the `loading` variable from the `useData()` hook and checking if it's loading then show `Loading...` text or else show `<Body />` component.
- Now we will have to create three components, so create folder `components/` and create three files `Body.tsx`, `UploadImage.tsx`, and `Header.tsx`.

`Body.tsx`

```typescript
declare let window: any;
import Identicon from "identicon.js";
import React from "react";
import { useData } from "../contexts/DataContext";

const Body = () => {
  const { images } = useData();
  return (
    <>
      {images.length > 0 &&
        images.map((image, index) => (
          <BodyItem
            key={index}
            totalDonationss={image.donationAmount}
            address={image.author}
            description={image.description}
            hash={image.hash}
            id={image.id}
          />
        ))}
    </>
  );
};

export default Body;

const BodyItem = ({ address, description, totalDonationss, hash, id }) => {
  const { donateImageOwner, updateImages } = useData();
  var data = new Identicon(address, 200).toString();
  return (
    <div className="w-full md:mx-0 md:max-w-2xl mt-5 p-3 border rounded-xl flex flex-col">
      <div className="flex flex-row space-x-5 bg-gray-100 rounded-t-xl py-3 px-4 border-t border-l border-r font-mono items-center">
        <img width={35} height={35} src={`data:image/png;base64, ${data}`} />
        <div className="overflow-ellipsis w-52 overflow-hidden">{address}</div>
      </div>
      <img src={`https://ipfs.infura.io/ipfs/${hash}`} />
      <div className="py-3 px-4 flex flex-col border-l border-r">
        <span className="font-sans font-bold">Description</span>
        <span className="font-sans pt-2">{description}</span>
      </div>
      <div className="bg-gray-100 rounded-b-xl py-3 px-4 border-b border-l border-r font-mono flex flex-row justify-between">
        <span>
          Total DONATIONS: {window.web3.utils.fromWei(totalDonations, "Ether")}{" "}
          MATIC
        </span>
        <div
          onClick={async () => {
            let donationAmount = window.web3.utils.toWei("0.1", "Ether");
            await donationImageOwner(id, donationAmount);
            await updateImages();
          }}
        >
          <span className="cursor-pointer font-bold text-blue-400">
            DONATE: 0.1 MATIC
          </span>
        </div>
      </div>
    </div>
  );
};
```

- Here we are getting all the images from the `useData()` hook, looping over each image and displaying them.
- Each image has an option to donate 0.1 MATIC to the author or the content. Clicking on `DONATE: 0.1 MATIC` , we are calling `donationImageOwner()` fuction from `useData()` hook, that will evantually call `donationImageOwner` function of out smart contract.

`Header.tsx`

```typescipt
import Identicon from  "identicon.js";
import React, { useEffect } from  "react";
import { useData } from  "../../contexts/DataContext";

function  Header() {
  const { account } =  useData();
  const [data, setData] =  React.useState();
  useEffect(() => {
    if (account  !==  "0x0") {
      setData(new  Identicon(account, 200).toString());
    }
  }, [account]);
  return (
    <div  className="container items-center">
      <div  className="flex flex-col md:flex-row items-center md:justify-between border py-3 px-5 rounded-xl">
        <span  className="font-mono">Polygon MATIC</span>
        <div  className="flex flex-row space-x-2 items-center">
          <div  className="h-5 w-5 rounded-full bg-blue-500"></div>
          <span  className="font-mono text-xl font-bold">Decentagram</span>
        </div>
        <div  className="flex flex-row space-x-2 items-center">
          <span  className="font-mono overflow-ellipsis w-52 overflow-hidden">
            {account}
          </span>
          {account  &&  data  && (
            <img
              width={35}
              height={35}
              src={`data:image/png;base64, ${data}`}
            />
          )}
        </div>
      </div>
    </div>
  );
}

export  default  Header;
```

- In the header, we are showing the account number of the current user which we can get from `useData()`.
- Just to make UI a bit better, we are also showing a [identicon](https://github.com/stewartlord/identicon.js/tree/master) based on account number.

`UploadImage.tsx`

For uploading an image, we have a popup model, where the author can choose the image from their system and upload it to IPFS.

```typescript
import { Dialog, Transition } from "@headlessui/react";
import { create } from "ipfs-http-client";
import { Fragment, useState } from "react";
import { useData } from "../contexts/DataContext";

interface Props {
  isOpen: boolean;
  closeModal: () => void;
}

export const UploadImage: React.FC<Props> = ({ isOpen, closeModal }) => {
  const [buttonTxt, setButtonTxt] = useState<string>("Upload");
  const [file, setFile] = useState<File | null>(null);
  const { contract, account, updateImages } = useData();
  const client = create({ url: "https://ipfs.infura.io:5001/api/v0" });
  const [description, setDescription] = useState<string>("");

  const uploadImage = async () => {
    setButtonTxt("Uploading to IPFS...");
    const added = await client.add(file);
    setButtonTxt("Creating smart contract...");
    contract.methods
      .uploadImage(added.path, description)
      .send({ from: account })
      .then(async () => {
        await updateImages();
        setFile(null);
        setDescription("");
        setButtonTxt("Upload");
        closeModal();
      })
      .catch(() => {
        closeModal();
      });
  };

  return (
    <>
      <Transition appear show={isOpen} as={Fragment}>
        <Dialog
          as="div"
          className="fixed inset-0 z-10 overflow-y-auto"
          onClose={closeModal}
        >
          <Dialog.Overlay className="fixed inset-0 bg-black opacity-40" />

          <div className="min-h-screen px-4 text-center ">
            <Transition.Child
              as={Fragment}
              enter="ease-out duration-300"
              enterFrom="opacity-0"
              enterTo="opacity-100"
              leave="ease-in duration-200"
              leaveFrom="opacity-100"
              leaveTo="opacity-0"
            >
              <Dialog.Overlay className="fixed inset-0" />
            </Transition.Child>

            {/* This element is to trick the browser into centering the modal contents. */}
            <span
              className="inline-block h-screen align-middle"
              aria-hidden="true"
            >
              &#8203;
            </span>
            <Transition.Child
              as={Fragment}
              enter="ease-out duration-300"
              enterFrom="opacity-0 scale-95"
              enterTo="opacity-100 scale-100"
              leave="ease-in duration-200"
              leaveFrom="opacity-100 scale-100"
              leaveTo="opacity-0 scale-95"
            >
              <div className="inline-block w-full p-6 my-8 overflow-hidden text-left align-middle transition-all transform bg-white shadow-xl rounded-2xl max-w-xl">
                <Dialog.Title
                  as="h3"
                  className="text-lg font-medium leading-6 text-gray-900"
                >
                  Upload Image to IPFS
                </Dialog.Title>
                <div className="mt-2">
                  <input
                    onChange={(e) => setFile(e.target.files[0])}
                    className="my-3"
                    type="file"
                  />
                </div>

                {file && (
                  <div className="mt-2">
                    <img src={URL.createObjectURL(file)} />
                  </div>
                )}

                <div className="mt-4">
                  <textarea
                    onChange={(e) => {
                      setDescription(e.target.value);
                    }}
                    value={description}
                    placeholder="Description"
                    className="px-3 py-1 font-sourceSansPro text-lg bg-gray-100 hover:bg-white focus:bg-white rounded-lg border-4 hover:border-4 border-transparent hover:border-green-200 focus:border-green-200 outline-none focus:outline-none focus:ring w-full pr-10 transition-all duration-500 ring-transparent"
                  />
                </div>
                <div className="mt-4">
                  <button
                    type="button"
                    disabled={buttonTxt !== "Upload"}
                    className="inline-flex justify-center px-4 py-2 text-sm font-medium text-blue-900 bg-blue-100 border border-transparent rounded-md hover:bg-blue-200 focus:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 focus-visible:ring-blue-500"
                    onClick={() => {
                      if (file) uploadImage();
                    }}
                  >
                    {buttonTxt}
                  </button>
                </div>
              </div>
            </Transition.Child>
          </div>
        </Dialog>
      </Transition>
    </>
  );
};
```

## Using IPFS to upload images

In the `UploadImage.tsx` we are using IPFS to upload the image. For this, we have to install a package.

```bash
npm install ipfs-http-client
```

Before using IPFS to upload images, we have to create an IPFS client.

```typescript
const client = create({ url: "https://ipfs.infura.io:5001/api/v0" });
```

> Here we are using a URL provided by [Infura](https://infura.io/product/ipfs).

```javascript
const added = await client.add(file);
```

To upload a file, call the `add` function on the client object. Add function will return an object that contains the hash of the image uploaded. Once we have the `hash`, we can store the hash in our smart contract.

This hash can later be used to access the image from IPFS by appending the hash to the URL provided by infura.

```txt
https://ipfs.infura.io/ipfs/${hash}
```

## Publishing the smart contract to Polygon Testnet

Publishing your smart contract is relatively simple. You will have to add provider details for Matic testnet in your `truffle-config.js`.

- First, you have to connect your metamask wallet to Matic mumbai-testnet. You can follow [this](https://docs.matic.network/docs/develop/metamask/config-polygon-on-metamask/) instructions by polygon docs to connect. Once you have connected metamask to Matic mumbai-testnet, you have to get the `mnemonic` of your metamask that you would have received when you created the metamask account. If you don't have those `mnemonics` then follow [these](https://metamask.zendesk.com/hc/en-us/articles/360015290032-How-to-reveal-your-Secret-Recovery-Phrase) steps to get the `mnemonics`.
- Now create a file `.secret` at the root of your project directory and paste your `mnemonic` in that file. This is the `mnemonic` of the account that will have to pay gas fees for contract deployment.
- **NOTE - Since we are deploying to testnet we don't have to pay actual MATIC token, we can use [Matic Faucet](https://faucet.matic.network/) to receive 1 MATIC in your account in the mumbai-testnet network.**
- **Make sure you have added .secret file in your .gitignore and never share your mnemonics with anyone.**
- Now we will have to install `hdwallet-provider` to pay gas fees while deploying.

```bash
npm install @truffle/hdwallet-provider
```

Once you have all these ready, modify the `truffle-config`. Your `truffle-config.js` should look something like this.

```javascript
const HDWalletProvider = require("@truffle/hdwallet-provider");
const fs = require("fs");
const mnemonic = fs.readFileSync(".secret").toString().trim();

module.exports = {
  networks: {
    development: {
      host: "localhost",
      port: 7545,
      network_id: "*",
    },
    matic: {
      provider: () =>
        new HDWalletProvider({
          mnemonic: {
            phrase: mnemonic,
          },
          providerOrUrl: `https://matic-mumbai.chainstacklabs.com`,
          chainId: 80001,
        }),
      network_id: 80001,
      confirmations: 2,
      timeoutBlocks: 200,
      skipDryRun: true,
      chainId: 80001,
    },
  },
  contracts_directory: "./contracts",
  contracts_build_directory: "./abis",
  compilers: {
    solc: {
      optimizer: {
        enabled: true,
        runs: 200,
      },
    },
  },
  db: {
    enabled: false,
  },
};
```

Here, we are adding another network `matic`, and adding `providerOrUrl`, `chainId`, and `network_id` of Polygon mumbai-testnet. You can find this information on [Polygon docs](https://docs.matic.network/docs/develop/network-details/network/).

After editing `truffle-config.js`, you just have to run the following command to deploy your contract to polygon testnet.

```bash
truffle migrate --network matic
```

After running the above code, you should get output similar to this -

```
Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.



Starting migrations...
======================
> Network name:    'matic'
> Network id:      80001
> Block gas limit: 20000000 (0x1312d00)


1_initial_migration.js
======================

   Replacing 'Migrations'
   ----------------------
   > transaction hash:    0x8cbc2616e3bf70965c32afded6ae6ece70038df1e39ee2ae4b832383964c7de1
   > Blocks: 2            Seconds: 4
   > contract address:    0x2F082Dd688b1B3c9a8A5310fda680A03873e7fA0
   > block number:        18368623
   > block timestamp:     1630486877
   > account:             0x2a65871CCbC1Ce534A388Bdf83aA28127c5d7c8a
   > balance:             1.851879295
   > gas used:            193243 (0x2f2db)
   > gas price:           1 gwei
   > value sent:          0 ETH
   > total cost:          0.000193243 ETH

   Pausing for 2 confirmations...
   ------------------------------
   > confirmation number: 2 (block: 18368627)

   > Saving migration to the chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:         0.000193243 ETH


2_deploy_contracts.js
=====================

   Replacing 'DonationContract'
   ------------------------
   > transaction hash:    0x24169d6f55478fbae26dc4a89d2b51da75af51ecddb7011ca0d3cd61f06225bc
   > Blocks: 2            Seconds: 4
   > contract address:    0x42c27d00217a7F5AE1cDAd9e71C2342373e13ADE
   > block number:        18368634
   > block timestamp:     1630486899
   > account:             0x2a65871CCbC1Ce534A388Bdf83aA28127c5d7c8a
   > balance:             1.850963627
   > gas used:            869930 (0xd462a)
   > gas price:           1 gwei
   > value sent:          0 ETH
   > total cost:          0.00086993 ETH

   Pausing for 2 confirmations...
   ------------------------------
   > confirmation number: 3 (block: 18368638)

   > Saving migration to the chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:          0.00086993 ETH


Summary
=======
> Total deployments:   2
> Final cost:          0.001063173 ETH
```

# Conclusion

In this tutorial, we have seen how to create a smart contract in solidity, have to use IPFS to upload images, how to interact with smart contracts in frontend applications using web3js and how to publish your smart contract to polygon testnet. Thank you so much for reading this far in this tutorial. This donation dapp is one of the endless possibilities of creating a dapp. If you end up creating something cool do let me know [@viral*sangani*](https://twitter.com/viral_sangani_), I would love to hear from you.

That's all for this tutorial. 👋
