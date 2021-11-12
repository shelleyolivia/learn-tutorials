---
description: Learn How to build a nameservice application (continued)
---

# Genesis

## Genesis <a id="genesis"></a>

The AppModule interface includes a number of functions for use in initializing and exporting GenesisState for the chain. The `ModuleBasicManager` calls these functions on each module when starting, stopping or exporting the chain. Here is a very basic implementation that you can expand upon.

Go to `x/nameservice/genesis.go` and you will see that a few things are missing. We have to fill them in according to the needs of the module. Below you will see what is missing:

```javascript
package nameservice

import (
    "fmt"

    sdk "github.com/cosmos/cosmos-sdk/types"
)

type GenesisState struct {
    WhoisRecords []Whois `json:"whois_records"`
}

func NewGenesisState(whoIsRecords []Whois) GenesisState {
    return GenesisState{WhoisRecords: nil}
}

func ValidateGenesis(data GenesisState) error {
    for _, record := range data.WhoisRecords {
        if record.Owner == nil {
            return fmt.Errorf("invalid WhoisRecord: Value: %s. Error: Missing Owner", record.Value)
        }
        if record.Value == "" {
            return fmt.Errorf("invalid WhoisRecord: Owner: %s. Error: Missing Value", record.Owner)
        }
        if record.Price == nil {
            return fmt.Errorf("invalid WhoisRecord: Value: %s. Error: Missing Price", record.Value)
        }
    }
    return nil
}

func DefaultGenesisState() GenesisState {
    return GenesisState{
        WhoisRecords: []Whois{},
    }
}

func InitGenesis(ctx sdk.Context, keeper Keeper, data GenesisState) {
    for _, record := range data.WhoisRecords {
        keeper.SetWhois(ctx, record.Value, record)
    }
}

func ExportGenesis(ctx sdk.Context, k Keeper) GenesisState {
    var records []Whois
    iterator := k.GetNamesIterator(ctx)
    for ; iterator.Valid(); iterator.Next() {

        name := string(iterator.Key())
        whois := k.GetWhois(ctx, name)
        records = append(records, whois)

    }
    return GenesisState{WhoisRecords: records}
}
```

Next we will define what the genesis state will be, the default genesis and a way to validate it so we don't run into any errors when we start the chain with preexisting state.

```javascript
package types

import (
    "fmt"
)

type GenesisState struct {
    WhoisRecords []Whois `json:"whois_records"`
}

func NewGenesisState(whoIsRecords []Whois) GenesisState {
    return GenesisState{WhoisRecords: nil}
}

func ValidateGenesis(data GenesisState) error {
    for _, record := range data.WhoisRecords {
        if record.Owner == nil {
            return fmt.Errorf("invalid WhoisRecord: Value: %s. Error: Missing Owner", record.Value)
        }
        if record.Value == "" {
            return fmt.Errorf("invalid WhoisRecord: Owner: %s. Error: Missing Value", record.Owner)
        }
        if record.Price == nil {
            return fmt.Errorf("invalid WhoisRecord: Value: %s. Error: Missing Price", record.Value)
        }
    }
    return nil
}

func DefaultGenesisState() GenesisState {
    return GenesisState{
        WhoisRecords: []Whois{},
    }
}
```

A few notes about the above code:

* `ValidateGenesis()` validates the provided genesis state to ensure that expected invariants hold
* `DefaultGenesisState()` is used mostly for testing. This provides a minimal GenesisState.
* `InitGenesis()` is called on chain start, this function imports genesis state into the keeper.
* `ExportGenesis()` is called after stopping the chain, this function loads application state into a GenesisState struct to later be exported to `genesis.json` alongside data from the other modules.

### Now your module has everything it needs to be incorporated into your Cosmos SDK application. <a id="now-your-module-has-everything-it-needs-to-be-incorporated-into-your-cosmos-sdk-application"></a>

If you had any difficulties following this tutorial or simply want to discuss Cosmos tech with us you can [**join our community today**](https://discord.gg/fszyM7K)!

