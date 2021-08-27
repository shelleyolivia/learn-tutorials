---
description: Learn how to build a nameservice application (continued)
---

# Types

## Types <a id="types"></a>

First thing we're going to do is create a module in the `/x/` folder with the scaffold tool using the below command:

In the case of this tutorial we will be naming the module `nameservice`

```javascript
cd x/

scaffold module [user] [repo] nameservice
```

### `types.go` <a id="types-go"></a>

Now we can continue with creating a module. Start by creating the file `types.go`in `./x/nameservice/types` folder which will hold customs types for the module.

## Whois

Each name will have three pieces of data associated with it.

* Value - The value that a name resolves to. This is just an arbitrary string, but in the future you can modify this to require it fitting a specific format, such as an IP address, DNS Zone file, or blockchain address.
* Owner - The address of the current owner of the name
* Price - The price you will need to pay in order to buy the name

To start your SDK module, define your `nameservice.Whois` struct in the `./x/nameservice/types/types.go` file:

```javascript
package types

import (
    "fmt"
    "strings"

    sdk "github.com/cosmos/cosmos-sdk/types"
)

// MinNamePrice is Initial Starting Price for a name that was never previously owned
var MinNamePrice = sdk.Coins{sdk.NewInt64Coin("nametoken", 1)}

// Whois is a struct that contains all the metadata of a name
type Whois struct {
    Value string         `json:"value"`
    Owner sdk.AccAddress `json:"owner"`
    Price sdk.Coins      `json:"price"`
}

// NewWhois returns a new Whois with the minprice as the price
func NewWhois() Whois {
    return Whois{
        Price: MinNamePrice,
    }
}

// implement fmt.Stringer
func (w Whois) String() string {
    return strings.TrimSpace(fmt.Sprintf(`Owner: %s
Value: %s
Price: %s`, w.Owner, w.Value, w.Price))
}
```

As mentioned in the [Design doc](https://tutorials.cosmos.network/nameservice/tutorial/app-design.html), if a name does not already have an owner, we want to initialize it with some MinPrice.

If you had any difficulties following this tutorial or simply want to discuss Cosmos tech with us you can [**join our community today**](https://discord.gg/fszyM7K)!

