# Introduction

In this tutorial, you will learn how to successfully connect your React Native App to use the Celo Wallet and return a Wallet Address from the Alfajores Wallet.

_To carry out transactions on the Celo Network, you have to connect your Wallet to be able to carry out transactions. When you start out building a dAPP using React Native, you will need this guide to demonstrate how you can install the required libraries to get your dApp up and running._

# Prerequisites

This article assumes that you have basic knowledge of JavaScript \(TypeScript\) and how to start a React Native App using expo. It is also assumed that you have read the expo documentation and have basic knowledge of the Celo Wallet.

1. [Celo Wallet](https://docs.celo.org/getting-started/alfajores-testnet/using-the-mobile-wallet)
2. [React Native using expo](https://docs.expo.io/)
3. [DappKit](https://docs.celo.org/developer-guide/dappkit/setup)

# Project Setup

You will need node version `^10.13.0`  
Open the Celo documentation and follow the setup instructions:

`expo init $YOUR_APP_NAME`  
We will use the `TypeScript Template >> Tabs` 

![](https://github.com/figment-networks/learn-tutorials/raw/master/assets/terminalimage.png)

To use the Celo DappKit, install using: `yarn add @celo/dappkit`  
DAppKit's dependencies require a bit of adjustment to use vanilla Expo. The first are a lot of the Node.js modules that are expected. You can get those mostly by using the following modules:`yarn add node-libs-react-native vm-browserify`

**Important Note!**

The default React Native library that is installed with the expo package will cause your app to break. To fix this, go to the `package.json` file and replace the `react-native` version value with this: `^0.63.4` at the time of writing this tutorial. You can always use the most recent version of React Native.

**A Note on the installed Packages**

_node-libs-react-native_: This package provides React Native compatible implementations of Node core modules like stream and http. This is a fork of node-libs-browser with a few packages swapped to be compatible in React Native.

_vm-browserify_ is used to emulate node's vm module for the browser.

**A couple of points to note:**

_The metro.config.js file_

```javascript
const crypto = require.resolve('crypto-browserify');
const url = require.resolve('url/');
module.exports = {
  resolver: {
    extraNodeModules: {
      crypto,
      url,
      fs: require.resolve('expo-file-system'),
      http: require.resolve('stream-http'),
      https: require.resolve('https-browserify'),
      net: require.resolve('react-native-tcp'),
      os: require.resolve('os-browserify/browser.js'),
      path: require.resolve('path-browserify'),
      stream: require.resolve('readable-stream'),
      vm: require.resolve('vm-browserify'),
    },
  },
};
```

This should allow you to build the project, however some dependencies might expect certain invariants on the global environment. For that you should create a file `global.ts` with the content below:

```typescript
export interface Global {
  btoa: any
  self: any
  Buffer: any
  process: any
  location: any
}

declare var global: Global
if (typeof global.self === 'undefined') {
  global.self = global
}

if (typeof btoa === 'undefined') {
  global.btoa = function (str: string) {
    return new Buffer(str, 'binary').toString('base64')
  }
}

global.Buffer = require('buffer').Buffer
global.process = require('process')
global.location = {
  protocol: 'https',
}
```

And then add `import './global'` at the top of your `App.js/tsx` file

**Set up Redux**

Install the Redux libraries using the following command:

```text
yarn add redux redux-thunk redux-logger react-redux
```

For TypeScript to run redux logger locally without any errors, you have to add the library `@types/redux-logger` to the devDependencies using the command:

```text
yarn add @types/redux-logger --dev
```

Run the app with the command `expo start`.

# Code

After installing the required libraries, we can then build 2 simple screens in the React Native app. The first screen will have one button. The button will be used to sign a User into the dApp. The second screen will be the screen that proves that the user has successfully authenticated. 

The second screen will return the user's wallet address.

**Let's build the Screens**:

```javascript
export default function LoginScreen() {
  return (
    <View style={styles.container}>
      <Button>Connect Wallet</Button>
    </View>
  );
}
```

![Login Screen](https://github.com/figment-networks/learn-tutorials/raw/master/assets/loginscreen.png)

```javascript
export default function HomeScreen() {
  return (
    <View>
      <Text>wallet address</Text>
      <Text>wallet phone</Text>
    </View>
  );
}
```

**Let's implement the Logic to connect to the Wallet**:

We will use Redux to manage the app state, we have to set up Redux actions to make a call to the Celo Wallet and return the result which is then saved in the app global state. To keep your directory devoid of clutter, open a directory to hold the files for the Redux logic:

![](https://github.com/figment-networks/learn-tutorials/raw/master/assets/reduxdirectory.png)

You can add a file called `constants.js` to the Redux Store Directory. The constants will be used to track the wallet connection process by having the constants as the type of action that was dispatched and then update the state accordingly using a reducer.

```javascript
export const walletConstants = {
  CONNECT_REQUEST: 'WALLET_CONNECT_REQUEST',
  CONNECT_SUCCESS: 'WALLET_CONNECT_SUCCESS',
  CONNECT_FAILURE: 'WALLET_CONNECT_FAILURE',
};
```

Create a `walletsAction.ts` file in the actions folder

```javascript
/*
 The export statement is used to export the only function in the file so that the function can be called using `walletsActions.connect()`
 */
export const walletActions = {
  connect,
};

/*
This function is a simple method provided by Celo to connect to the Valora or Alfajores (for testing) wallet. The `dispatch()` is a redux function which is used to emit actions which we can then listen for in the reducer and update the state accordingly.
*/
function connect() {
  return (dispatch: any) => {
    // This dispatch calls a function that is declared later on in the code.
    dispatch(request('Connecting to wallet'));

    // These variables are needed to connect to the wallet
    // requestId is used to identify the request so we can listen for the same request using the waitForAccoutAuth() function
    // dappName holds the name of the App the wallet will expose as requesting for the detaila
    // callback is the screen we want to send the user to after a successfull connection is made
    const requestId = 'dapplogin';
    const dappName = 'celodapp';
    const callback = Linking.makeUrl('two');

    // This is from the Celo DappKit library, it fires up the wallet and gets the neccessary information
    requestAccountAddress({
      requestId,
      dappName,
      callback,
    });

    //This function listens for the request above and fire up an action to be handled by a reducer.
    waitForAccountAuth(requestId)
      .then((res) => {
        dispatch(success(res));
      })
      .catch((err) => {
        dispatch(failure(err));
        dispatch(alertActions.error(err.toString()));
      });
  };

  // These are the function calls which are dispatched when the user makes a request. The state of the app changes with the status of the request response.
  function request(message: string) {
    return { type: walletConstants.CONNECT_REQUEST, message };
  }
  function success(res: object) {
    return { type: walletConstants.CONNECT_SUCCESS, res };
  }
  function failure(error: any) {
    return { type: walletConstants.CONNECT_FAILURE, error };
  }
}
```

The above file is the `walletAction.ts` It contains the logic to connect to the Wallet and saves the response in the global redux state which is accessible to any part of the codebase. The next thing will be to write the reducer logic to handle the app state modification.

Create a `walletReducer.ts` file in the reducers folder and add the following code:

```javascript
// The initial state of the wallet
const initialState = {
  failed: true,
  connecting: false,
  message: '',
  address: '',
  phone: '',
};

// This is the wallet reducer which takes the state and action as parameters.
// It modifies the state accordingly based on the type of action that it receives from the dispatch calls in the `walletAction.ts` file
export function wallet(state = initialState, action: any) {
  switch (action.type) {
    case walletConstants.CONNECT_REQUEST:
      return {
        connecting: true,
        message: action.message,
      };
    case walletConstants.CONNECT_SUCCESS:
      return {
        failed: false,
        address: action.res.address,
        phone: action.res.phoneNumber,
      };
    case walletConstants.CONNECT_FAILURE:
      return {
        error: action.error,
      };
    default:
      return state;
  }
}
```

Create a `store.ts` file in the store folder:

```javascript
// This is where the store is setup. This is where redux updates the state of the store based on the user actions.
import { createStore, applyMiddleware } from 'redux';
import thunkMiddleware from 'redux-thunk';
import { createLogger } from 'redux-logger';

import { rootReducer } from './reducers';

const loggerMiddleware = createLogger();

export const store = createStore(rootReducer, applyMiddleware(thunkMiddleware, loggerMiddleware));
```

Let's update the `App.tsx` file with the Redux Provider:

```javascript
// The Redux Provider wraps around the entire application
export default function App() {
  return (
    <Provider store={store}>
    ...
    </Provider>
  );
}
```

Time to update the LoginScreen and the HomeScreen with the Redux Actions:

```javascript
// The useDispatch is a React Redux Hook to dispatch an action creator.
export default function HomeScreen() {
  const dispatch = useDispatch();

  // This function calls up the connect function from the walletAction Action
  const login = () => {
    dispatch(walletActions.connect());
  };

  return (
    <View style={styles.container}>
      <Button onPress={login}>Connect Wallet</Button>
    </View>
  );
}
```

Also update the HomeScreen and the HomeScreen with the Redux Actions:

```javascript
// Here, we connect to the global state with useSelector hook
export default function HomeScreen() {
  const wallet = useSelector((state: any) => state.wallet);

  // We make sure to handle instances where a user tries to navigate to this page without connecting the app to their wallet by making sure to navigate back to the loginscreen if a connection to the wallet hasn't been made yet
  React.useEffect(() => {
    if (wallet.failed) {
      navigation.navigate('Root');
    }
  }, []);

  return (
    <View>
      <Text>{wallet.address}</Text>
      <Text>{wallet.phone}</Text>
    </View>
  );
}
```

Run your app and Login to see the Wallet and Phone number returned to the Home Screen.

![Wallet Screen](https://github.com/figment-networks/learn-tutorials/raw/master/assets/walletscreen.png)

![Home Screen](https://github.com/figment-networks/learn-tutorials/raw/master/assets/homescreen.png)

# WalletConnect v1 alternative

Instead of using celo DappKit api, nowadays WalletConnect is recommended:

[Valora + WalletConnect v1](https://docs.celo.org/developer-resources/walkthroughs/valora-wc-v1)

This chapter explains how to upgrade the app using node version `^v17.3.0` WalletConnect instead of DappKit
In addition, disconnect functionality and balances information are retrieved and displayed (new sample pictures added). Also, we will explain how to run
the application in web browser mode.

1. Make sure you have nvm installed. This helps you switch between node versions very easy, depending on project needs

```text
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.37.2/install.sh | bash
```

2. Install and use node v17.3.0 with `nvm` the [Node.js version manager](https://github.com/nvm-sh/nvm#intro)

```text
nvm install v17.3.0
nvm use 17.3.0
```

3. To use the WalletConnect v1, install using: 

```text
yarn add @celo/contractkit
yarn add @walletconnect/web3-provider
yarn add react-native-tcp
yarn add web3
```

Prior to running the application, you may need to execute `export NODE_OPTIONS=--openssl-legacy-provider` needed for some Linux systems (CentOS 7)

Run the app using: `expo start` You should see the following:

![Start](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/assets/START_APP.png?raw=true)

Point your web browser under: http://localhost:19002 according to above picture. 
In the following screen, choose "Run in web browser" and your app will be launched in a new web page.

![Run in web browser](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/assets/RUN_WEB_BROWSER.png?raw=true)

## Code changes
1. In the `constants.js` add new constant:
```javascript
DISCONNECT_REQUEST: 'WALLET_DISCONNECT_REQUEST'
```
  
2. Remove DappKit logic from `walletsAction.ts`, change the logic of connect function with WalletConnect support. Add `disconnect` function

```javascript

  export const walletActions = {
    connect, disconnect,
  };

  /*
   Provider configuration to enable wallet connection
  */
  const provider = new WalletConnectProvider({
    rpc: {
      44787: "https://alfajores-forno.celo-testnet.org",
      42220: "https://forno.celo.org",
    },
  });

function connect(navigation) {
  return (dispatch: any) => {
    // This dispatch calls a function that is declared later on in the code.
    let asyncConnect = async() => {
      await provider.enable();
      const web3 = new Web3(provider);
      let kit = newKitFromWeb3(web3)
      kit.defaultAccount = provider.accounts[0]
      let asyncGetData = async() => {
        const stableToken = await kit.contracts.getStableToken();
        const goldToken = await kit.contracts.getGoldToken();
        const cUsdBalanceObj = await stableToken.balanceOf(kit.defaultAccount);
        const goldBalanceObj = await goldToken.balanceOf(kit.defaultAccount);
        const res = {address:kit.defaultAccount, cUsd:cUsdBalanceObj/10**18, celo:goldBalanceObj/10**18};
        dispatch(success(res));
        navigation.navigate('TabTwo');
      }
      provider.on("accountsChanged", (accounts) => {
        asyncGetData();
      });
    }
    dispatch(request('Connecting to wallet'));
    asyncConnect();
  };
  ...
 }
 /* Disconnect from wallet function
 */
 function disconnect() {
  return (dispatch: any) => {
    // This dispatch calls a function that is declared later on in the code.
    let asyncDisconnect = async() => {
        await provider.disconnect();
    }
    dispatch(requestDisconnect('Disconnecting from wallet'));
    asyncDisconnect();
    };
    function requestDisconnect(message: string) {
      return { type: walletConstants.DISCONNECT_REQUEST, message };

    }
  }
```
  
3. Add `cUsd` and `celo` fields to app state in `walletReducer.ts`

```javascript
  const initialState = {
    failed: true,
    connecting: false,
    message: '',
    address: '',
    cUsd: '',
    celo:''
  };

  export function wallet(state = initialState, action: any) {
    switch (action.type) {
      ...
      case walletConstants.CONNECT_SUCCESS:
        return {
          failed: false,
          address: action.res.address,
          cUsd: action.res.cUsd,
          celo: action.res.celo
        };
     ...
    }
  }
```

4. Add logout function to LoginScreen, and inject tab two navigation object in `connect` (as you noticed, we removed the Linking.makeUrl used by DappKit callback, with react native navigation support, which does not refresh entire page to show wallet address/balances. This is not needed, since react uses `state` to show updated page in a responsive async manner. 

```javascript
export default function LoginScreen({ navigation }) {
  const dispatch = useDispatch();

  // This function calls up the connect function from the walletAction Action
  const login = () => {
    dispatch(walletActions.connect(navigation));
  };

  const logout = () => {
    dispatch(walletActions.disconnect());
  };

  return (
    <View style={styles.container}>

        <Button onPress={login} title="Connect Wallet"></Button>
        <Button onPress={logout} title="Disconnect Wallet"></Button>

    </View>
  );
}
```

Here is how the login page should look like now, the new disconnect button is added
![Login Screen](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/assets/TAB_ONE.png?raw=true)

5. Change HomeScreen to display balances in addition to address

```javascript
export default function HomeScreen({ navigation }) {
  ...
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Address: {wallet.address}</Text>
      <Text style={styles.title1}>cUSD: {wallet.cUsd}</Text>
      <Text style={styles.title1}>Celo: {wallet.celo}</Text>
    </View>
  );
}
```

## Application workflow

Run your app and Login to see the Wallet address and balances returned to the Home Screen.

1. First, scan your valora mobile app with the WalletConnect screen shown after the Connect button is pressed (first picture from below)

2. You need to allow access from your Valora Mobile app in order to see the data from Home Screen (second picture from below)

3. When permission granted, the second tab (home screen) is automatically displayed with address and balances information (third picture from below)

![Wallet connect Screen](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/assets/VALORA_QR.png?raw=true)

![Wallet Screen](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/assets/VALORA_CONNECT_F.jpeg?raw=true)

![Home Screen](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/assets/VALORA_TAB2_F.png?raw=true)

# Conclusion

In this tutorial, we learned: How to successfully connect your Redux based React Native App to use the Celo Wallet and return a Wallet Address from the Valora/Alfajores Wallet. The last chapter (WalletConnect v1 alternative) explains how to use the new connection api supported by valora with your mobile installed valora app (https://valoraapp.com/).

# About the Authors

This tutorial was created by [Segun Ogundipe](https://www.linkedin.com/in/segun-ogundipe) and [Emmanuel Oaikhenan](https://github.com/emmaodia).

The WalletConnect v1 alternative chapter was created by [Mircea Carasel](https://ro.linkedin.com/in/mirceac).
Also, the full code with the WalletConnect approach can be found here: https://github.com/mirceac/valora2
