---
description: Learn how to build a scavenger game (continued)
---

# App

## App <a id="app"></a>

Our `scaffold` utility has already created a pretty complete `app.go` file for us inside of `./app/app.go`. This version of the `app.go` file is meant to be as simple of an app as possible. It contains only the necessary modules needed for using an app that knows about coins \(`bank`\), user accounts \(`auth`\) and securing the application with proof-of-stake \(`staking`\).

One module which is missing but is part of the Cosmos SDK core set of features is `gov`, which allows for a simple form of governance to take place. This process includes making text proposals which can be voted on with coins or with delegation via [liquid democracy](https://en.wikipedia.org/wiki/Liquid_democracy). This module can also be used to update the logic of the application itself, via parameter changes which can affect specific parts of different modules.

Mostly we can just follow the `TODO`s that are marked in the file and add the necessary information about our new module. Afterwards it should look like:

```javascript
package app

import (
    "encoding/json"
    "io"
    "os"

    abci "github.com/tendermint/tendermint/abci/types"
    "github.com/tendermint/tendermint/libs/log"
    tmos "github.com/tendermint/tendermint/libs/os"
    dbm "github.com/tendermint/tm-db"

    bam "github.com/cosmos/cosmos-sdk/baseapp"
    "github.com/cosmos/cosmos-sdk/codec"
    "github.com/cosmos/cosmos-sdk/simapp"
    sdk "github.com/cosmos/cosmos-sdk/types"
    "github.com/cosmos/cosmos-sdk/types/module"
    "github.com/cosmos/cosmos-sdk/version"
    "github.com/cosmos/cosmos-sdk/x/auth"
    "github.com/cosmos/cosmos-sdk/x/auth/vesting"
    "github.com/cosmos/cosmos-sdk/x/bank"
    distr "github.com/cosmos/cosmos-sdk/x/distribution"
    "github.com/cosmos/cosmos-sdk/x/genutil"
    "github.com/cosmos/cosmos-sdk/x/params"
    "github.com/cosmos/cosmos-sdk/x/slashing"
    "github.com/cosmos/cosmos-sdk/x/staking"
    "github.com/cosmos/cosmos-sdk/x/supply"
    "github.com/cosmos/sdk-tutorials/scavenge/x/scavenge"
)

const appName = "app"

var (
    // default home directories for the application CLI
    DefaultCLIHome = os.ExpandEnv("$HOME/.scavengeCLI")

    // DefaultNodeHome sets the folder where the applcation data and configuration will be stored
    DefaultNodeHome = os.ExpandEnv("$HOME/.scavengeD")

    // NewBasicManager is in charge of setting up basic module elemnets
    ModuleBasics = module.NewBasicManager(
        genutil.AppModuleBasic{},
        auth.AppModuleBasic{},
        bank.AppModuleBasic{},
        staking.AppModuleBasic{},
        distr.AppModuleBasic{},
        params.AppModuleBasic{},
        slashing.AppModuleBasic{},
        supply.AppModuleBasic{},

        scavenge.AppModuleBasic{},
    )
    // account permissions
    maccPerms = map[string][]string{
        auth.FeeCollectorName:     nil,
        distr.ModuleName:          nil,
        staking.BondedPoolName:    {supply.Burner, supply.Staking},
        staking.NotBondedPoolName: {supply.Burner, supply.Staking},
    }
)

// MakeCodec generates the necessary codecs for Amino
func MakeCodec() *codec.Codec {
    var cdc = codec.New()

    ModuleBasics.RegisterCodec(cdc)
    vesting.RegisterCodec(cdc)
    sdk.RegisterCodec(cdc)
    codec.RegisterCrypto(cdc)

    return cdc.Seal()
}

type NewApp struct {
    *bam.BaseApp
    cdc *codec.Codec

    // keys to access the substores
    keys  map[string]*sdk.KVStoreKey
    tkeys map[string]*sdk.TransientStoreKey

    // subspaces
    subspaces map[string]params.Subspace

    // Keepers
    accountKeeper  auth.AccountKeeper
    bankKeeper     bank.Keeper
    stakingKeeper  staking.Keeper
    slashingKeeper slashing.Keeper
    distrKeeper    distr.Keeper
    supplyKeeper   supply.Keeper
    paramsKeeper   params.Keeper
    scavengeKeeper scavenge.Keeper

    // Module Manager
    mm *module.Manager

    // simulation manager
    sm *module.SimulationManager
}

// verify app interface at compile time
var _ simapp.App = (*NewApp)(nil)

// NewInitApp is a constructor function for scavengeApp
func NewInitApp(logger log.Logger, db dbm.DB, traceStore io.Writer, loadLatest bool,
    invCheckPeriod uint, baseAppOptions ...func(*bam.BaseApp)) *NewApp {

    // First define the top level codec that will be shared by the different modules
    cdc := MakeCodec()

    // BaseApp handles interactions with Tendermint through the ABCI protocol
    bApp := bam.NewBaseApp(appName, logger, db, auth.DefaultTxDecoder(cdc), baseAppOptions...)

    bApp.SetAppVersion(version.Version)

    keys := sdk.NewKVStoreKeys(bam.MainStoreKey, auth.StoreKey, staking.StoreKey,
        supply.StoreKey, distr.StoreKey, slashing.StoreKey, params.StoreKey, scavenge.StoreKey)

    tkeys := sdk.NewTransientStoreKeys(staking.TStoreKey, params.TStoreKey)

    // Here you initialize your application with the store keys it requires
    var app = &NewApp{
        BaseApp:   bApp,
        cdc:       cdc,
        keys:      keys,
        tkeys:     tkeys,
        subspaces: make(map[string]params.Subspace),
    }

    // The ParamsKeeper handles parameter storage for the application
    app.paramsKeeper = params.NewKeeper(app.cdc, keys[params.StoreKey], tkeys[params.TStoreKey])
    // Set specific supspaces
    app.subspaces[auth.ModuleName] = app.paramsKeeper.Subspace(auth.DefaultParamspace)
    app.subspaces[bank.ModuleName] = app.paramsKeeper.Subspace(bank.DefaultParamspace)
    app.subspaces[staking.ModuleName] = app.paramsKeeper.Subspace(staking.DefaultParamspace)
    app.subspaces[distr.ModuleName] = app.paramsKeeper.Subspace(distr.DefaultParamspace)
    app.subspaces[slashing.ModuleName] = app.paramsKeeper.Subspace(slashing.DefaultParamspace)

    // The AccountKeeper handles address -> account lookups
    app.accountKeeper = auth.NewAccountKeeper(
        app.cdc,
        keys[auth.StoreKey],
        app.subspaces[auth.ModuleName],
        auth.ProtoBaseAccount,
    )

    // The BankKeeper allows you perform sdk.Coins interactions
    app.bankKeeper = bank.NewBaseKeeper(
        app.accountKeeper,
        app.subspaces[bank.ModuleName],
        app.ModuleAccountAddrs(),
    )

    // The SupplyKeeper collects transaction fees and renders them to the fee distribution module
    app.supplyKeeper = supply.NewKeeper(
        app.cdc,
        keys[supply.StoreKey],
        app.accountKeeper,
        app.bankKeeper,
        maccPerms,
    )

    // The staking keeper
    stakingKeeper := staking.NewKeeper(
        app.cdc,
        keys[staking.StoreKey],
        app.supplyKeeper,
        app.subspaces[staking.ModuleName],
    )

    app.distrKeeper = distr.NewKeeper(
        app.cdc,
        keys[distr.StoreKey],
        app.subspaces[distr.ModuleName],
        &stakingKeeper,
        app.supplyKeeper,
        auth.FeeCollectorName,
        app.ModuleAccountAddrs(),
    )

    app.slashingKeeper = slashing.NewKeeper(
        app.cdc,
        keys[slashing.StoreKey],
        &stakingKeeper,
        app.subspaces[slashing.ModuleName],
    )

    // register the staking hooks
    // NOTE: stakingKeeper above is passed by reference, so that it will contain these hooks
    app.stakingKeeper = *stakingKeeper.SetHooks(
        staking.NewMultiStakingHooks(
            app.distrKeeper.Hooks(),
            app.slashingKeeper.Hooks()),
    )

    app.scavengeKeeper = scavenge.NewKeeper(
        app.bankKeeper,
        app.cdc,
        keys[scavenge.StoreKey],
    )

    app.mm = module.NewManager(
        genutil.NewAppModule(app.accountKeeper, app.stakingKeeper, app.BaseApp.DeliverTx),
        auth.NewAppModule(app.accountKeeper),
        bank.NewAppModule(app.bankKeeper, app.accountKeeper),
        scavenge.NewAppModule(app.scavengeKeeper, app.bankKeeper),
        supply.NewAppModule(app.supplyKeeper, app.accountKeeper),
        distr.NewAppModule(app.distrKeeper, app.accountKeeper, app.supplyKeeper, app.stakingKeeper),
        slashing.NewAppModule(app.slashingKeeper, app.accountKeeper, app.stakingKeeper),
        staking.NewAppModule(app.stakingKeeper, app.accountKeeper, app.supplyKeeper),
    )

    app.mm.SetOrderBeginBlockers(distr.ModuleName, slashing.ModuleName)
    app.mm.SetOrderEndBlockers(staking.ModuleName)

    // Sets the order of Genesis - Order matters, genutil is to always come last
    // NOTE: The genutils moodule must occur after staking so that pools are
    // properly initialized with tokens from genesis accounts.
    app.mm.SetOrderInitGenesis(
        distr.ModuleName,
        staking.ModuleName,
        auth.ModuleName,
        bank.ModuleName,
        slashing.ModuleName,
        scavenge.ModuleName,
        supply.ModuleName,
        genutil.ModuleName,
    )

    // register all module routes and module queriers
    app.mm.RegisterRoutes(app.Router(), app.QueryRouter())

    // The initChainer handles translating the genesis.json file into initial state for the network
    app.SetInitChainer(app.InitChainer)
    app.SetBeginBlocker(app.BeginBlocker)
    app.SetEndBlocker(app.EndBlocker)

    // The AnteHandler handles signature verification and transaction pre-processing
    app.SetAnteHandler(
        auth.NewAnteHandler(
            app.accountKeeper,
            app.supplyKeeper,
            auth.DefaultSigVerificationGasConsumer,
        ),
    )

    // initialize stores
    app.MountKVStores(keys)
    app.MountTransientStores(tkeys)

    err := app.LoadLatestVersion(app.keys[bam.MainStoreKey])
    if err != nil {
        tmos.Exit(err.Error())
    }

    return app
}

// GenesisState represents chain state at the start of the chain. Any initial state (account balances) are stored here.
type GenesisState map[string]json.RawMessage

func NewDefaultGenesisState() GenesisState {
    return ModuleBasics.DefaultGenesis()
}

func (app *NewApp) InitChainer(ctx sdk.Context, req abci.RequestInitChain) abci.ResponseInitChain {
    var genesisState GenesisState

    err := app.cdc.UnmarshalJSON(req.AppStateBytes, &genesisState)
    if err != nil {
        panic(err)
    }

    return app.mm.InitGenesis(ctx, genesisState)
}

func (app *NewApp) BeginBlocker(ctx sdk.Context, req abci.RequestBeginBlock) abci.ResponseBeginBlock {
    return app.mm.BeginBlock(ctx, req)
}

func (app *NewApp) EndBlocker(ctx sdk.Context, req abci.RequestEndBlock) abci.ResponseEndBlock {
    return app.mm.EndBlock(ctx, req)
}

func (app *NewApp) LoadHeight(height int64) error {
    return app.LoadVersion(height, app.keys[bam.MainStoreKey])
}

// Codec returns simapp's codec
func (app *NewApp) Codec() *codec.Codec {
    return app.cdc
}

// SimulationManager implements the SimulationApp interface
func (app *NewApp) SimulationManager() *module.SimulationManager {
    return app.sm
}

// ModuleAccountAddrs returns all the app's module account addresses.
func (app *NewApp) ModuleAccountAddrs() map[string]bool {
    modAccAddrs := make(map[string]bool)
    for acc := range maccPerms {
        modAccAddrs[supply.NewModuleAddress(acc).String()] = true
    }

    return modAccAddrs
}
```

Something you might notice near the beginning of the file is that I have renamed the `DefaultCLIHome` and the `DefaultNodeHome`. These are the directories located on your machine where the history of your application and the configuration of your CLI are stored, as well as the encrypted information for the keys you generate on your machine. I renamed them to `scavengeCLI` and `scavengeD` to better reflect our application.

Since we don't want to use the generic commands for our CLI and our application given to us by the `scaffold` command, let's rename the files within `cmd` as well. We will rename `./cmd/appcli/` to `./cmd/scavengeCLI` as well as `./cmd/appd` to `./cmd/scavengeD`. Inside our `./cmd/scavengeCLI/main.go` file we will also update to the following format:

```javascript
package main

import (
    "fmt"
    "os"
    "path"

    "github.com/cosmos/cosmos-sdk/client"
    "github.com/cosmos/cosmos-sdk/client/flags"
    "github.com/cosmos/cosmos-sdk/client/keys"
    "github.com/cosmos/cosmos-sdk/client/lcd"
    "github.com/cosmos/cosmos-sdk/client/rpc"
    sdk "github.com/cosmos/cosmos-sdk/types"
    "github.com/cosmos/cosmos-sdk/version"
    "github.com/cosmos/cosmos-sdk/x/auth"
    authcmd "github.com/cosmos/cosmos-sdk/x/auth/client/cli"
    "github.com/cosmos/cosmos-sdk/x/bank"
    bankcmd "github.com/cosmos/cosmos-sdk/x/bank/client/cli"
    "github.com/cosmos/sdk-tutorials/scavenge/app"
    "github.com/spf13/cobra"
    "github.com/spf13/viper"
    amino "github.com/tendermint/go-amino"
    "github.com/tendermint/tendermint/libs/cli"
)

func main() {
    // Configure cobra to sort commands
    cobra.EnableCommandSorting = false

    // Instantiate the codec for the command line application
    cdc := app.MakeCodec()

    // Read in the configuration file for the sdk
    config := sdk.GetConfig()
    config.SetBech32PrefixForAccount(sdk.Bech32PrefixAccAddr, sdk.Bech32PrefixAccPub)
    config.SetBech32PrefixForValidator(sdk.Bech32PrefixValAddr, sdk.Bech32PrefixValPub)
    config.SetBech32PrefixForConsensusNode(sdk.Bech32PrefixConsAddr, sdk.Bech32PrefixConsPub)
    config.Seal()

    rootCmd := &cobra.Command{
        Use:   "scavengeCLI",
        Short: "Scavenge Client",
    }

    // Add --chain-id to persistent flags and mark it required
    rootCmd.PersistentFlags().String(flags.FlagChainID, "", "Chain ID of tendermint node")
    rootCmd.PersistentPreRunE = func(_ *cobra.Command, _ []string) error {
        return initConfig(rootCmd)
    }

    // Construct Root Command
    rootCmd.AddCommand(
        rpc.StatusCommand(),
        client.ConfigCmd(app.DefaultCLIHome),
        queryCmd(cdc),
        txCmd(cdc),
        flags.LineBreak,
        lcd.ServeCommand(cdc, registerRoutes),
        flags.LineBreak,
        keys.Commands(),
        flags.LineBreak,
        version.Cmd,
        flags.NewCompletionCmd(rootCmd, true),
    )

    // Add flags and prefix all env exposed with AA
    executor := cli.PrepareMainCmd(rootCmd, "AA", app.DefaultCLIHome)

    err := executor.Execute()
    if err != nil {
        fmt.Printf("Failed executing CLI command: %s, exiting...\n", err)
        os.Exit(1)
    }
}

func queryCmd(cdc *amino.Codec) *cobra.Command {
    queryCmd := &cobra.Command{
        Use:     "query",
        Aliases: []string{"q"},
        Short:   "Querying subcommands",
    }

    queryCmd.AddCommand(
        authcmd.GetAccountCmd(cdc),
        flags.LineBreak,
        rpc.ValidatorCommand(cdc),
        rpc.BlockCommand(),
        authcmd.QueryTxsByEventsCmd(cdc),
        authcmd.QueryTxCmd(cdc),
        flags.LineBreak,
    )

    // add modules' query commands
    app.ModuleBasics.AddQueryCommands(queryCmd, cdc)

    return queryCmd
}

func txCmd(cdc *amino.Codec) *cobra.Command {
    txCmd := &cobra.Command{
        Use:   "tx",
        Short: "Transactions subcommands",
    }

    txCmd.AddCommand(
        bankcmd.SendTxCmd(cdc),
        flags.LineBreak,
        authcmd.GetSignCommand(cdc),
        authcmd.GetMultiSignCommand(cdc),
        flags.LineBreak,
        authcmd.GetBroadcastCommand(cdc),
        authcmd.GetEncodeCommand(cdc),
        authcmd.GetDecodeCommand(cdc),
        flags.LineBreak,
    )

    // add modules' tx commands
    app.ModuleBasics.AddTxCommands(txCmd, cdc)

    // remove auth and bank commands as they're mounted under the root tx command
    var cmdsToRemove []*cobra.Command

    for _, cmd := range txCmd.Commands() {
        if cmd.Use == auth.ModuleName || cmd.Use == bank.ModuleName {
            cmdsToRemove = append(cmdsToRemove, cmd)
        }
    }

    txCmd.RemoveCommand(cmdsToRemove...)

    return txCmd
}

func registerRoutes(rs *lcd.RestServer) {
    client.RegisterRoutes(rs.CliCtx, rs.Mux)
    app.ModuleBasics.RegisterRESTRoutes(rs.CliCtx, rs.Mux)
}

func initConfig(cmd *cobra.Command) error {
    home, err := cmd.PersistentFlags().GetString(cli.HomeFlag)
    if err != nil {
        return err
    }

    cfgFile := path.Join(home, "config", "config.toml")
    if _, err := os.Stat(cfgFile); err == nil {
        viper.SetConfigFile(cfgFile)

        if err := viper.ReadInConfig(); err != nil {
            return err
        }
    }
    if err := viper.BindPFlag(flags.FlagChainID, cmd.PersistentFlags().Lookup(flags.FlagChainID)); err != nil {
        return err
    }
    if err := viper.BindPFlag(cli.EncodingFlag, cmd.PersistentFlags().Lookup(cli.EncodingFlag)); err != nil {
        return err
    }
    return viper.BindPFlag(cli.OutputFlag, cmd.PersistentFlags().Lookup(cli.OutputFlag))
}
```

And within our `./cmd/scavengeD/main.go` we will update to the following format:

```javascript
package main

import (
    "encoding/json"
    "io"

    "github.com/spf13/cobra"
    "github.com/spf13/viper"

    abci "github.com/tendermint/tendermint/abci/types"
    "github.com/tendermint/tendermint/libs/cli"
    "github.com/tendermint/tendermint/libs/log"
    tmtypes "github.com/tendermint/tendermint/types"
    dbm "github.com/tendermint/tm-db"

    "github.com/cosmos/cosmos-sdk/baseapp"
    "github.com/cosmos/cosmos-sdk/client/debug"
    "github.com/cosmos/cosmos-sdk/client/flags"
    "github.com/cosmos/cosmos-sdk/server"
    "github.com/cosmos/cosmos-sdk/store"
    storetypes "github.com/cosmos/cosmos-sdk/store/types"
    sdk "github.com/cosmos/cosmos-sdk/types"
    "github.com/cosmos/cosmos-sdk/x/auth"
    genutilcli "github.com/cosmos/cosmos-sdk/x/genutil/client/cli"
    "github.com/cosmos/cosmos-sdk/x/staking"

    app "github.com/cosmos/sdk-tutorials/scavenge/app"
)

const flagInvCheckPeriod = "inv-check-period"

var invCheckPeriod uint

func main() {
    cdc := app.MakeCodec()

    config := sdk.GetConfig()
    config.SetBech32PrefixForAccount(sdk.Bech32PrefixAccAddr, sdk.Bech32PrefixAccPub)
    config.SetBech32PrefixForValidator(sdk.Bech32PrefixValAddr, sdk.Bech32PrefixValPub)
    config.SetBech32PrefixForConsensusNode(sdk.Bech32PrefixConsAddr, sdk.Bech32PrefixConsPub)
    config.Seal()

    ctx := server.NewDefaultContext()
    cobra.EnableCommandSorting = false
    rootCmd := &cobra.Command{
        Use:               "scavengeD",
        Short:             "Scavenge Daemon (server)",
        PersistentPreRunE: server.PersistentPreRunEFn(ctx),
    }

    rootCmd.AddCommand(genutilcli.InitCmd(ctx, cdc, app.ModuleBasics, app.DefaultNodeHome))
    rootCmd.AddCommand(genutilcli.CollectGenTxsCmd(ctx, cdc, auth.GenesisAccountIterator{}, app.DefaultNodeHome))
    rootCmd.AddCommand(genutilcli.MigrateGenesisCmd(ctx, cdc))
    rootCmd.AddCommand(
        genutilcli.GenTxCmd(
            ctx, cdc, app.ModuleBasics, staking.AppModuleBasic{},
            auth.GenesisAccountIterator{}, app.DefaultNodeHome, app.DefaultCLIHome,
        ),
    )
    rootCmd.AddCommand(genutilcli.ValidateGenesisCmd(ctx, cdc, app.ModuleBasics))
    rootCmd.AddCommand(AddGenesisAccountCmd(ctx, cdc, app.DefaultNodeHome, app.DefaultCLIHome))
    rootCmd.AddCommand(flags.NewCompletionCmd(rootCmd, true))
    rootCmd.AddCommand(debug.Cmd(cdc))

    server.AddCommands(ctx, cdc, rootCmd, newApp, exportAppStateAndTMValidators)

    // prepare and add flags
    executor := cli.PrepareBaseCmd(rootCmd, "AU", app.DefaultNodeHome)
    rootCmd.PersistentFlags().UintVar(&invCheckPeriod, flagInvCheckPeriod,
        0, "Assert registered invariants every N blocks")
    err := executor.Execute()
    if err != nil {
        panic(err)
    }
}

func newApp(logger log.Logger, db dbm.DB, traceStore io.Writer) abci.Application {
    var cache sdk.MultiStorePersistentCache

    if viper.GetBool(server.FlagInterBlockCache) {
        cache = store.NewCommitKVStoreCacheManager()
    }

    return app.NewInitApp(
        logger, db, traceStore, true, invCheckPeriod,
        baseapp.SetPruning(storetypes.NewPruningOptionsFromString(viper.GetString("pruning"))),
        baseapp.SetMinGasPrices(viper.GetString(server.FlagMinGasPrices)),
        baseapp.SetHaltHeight(viper.GetUint64(server.FlagHaltHeight)),
        baseapp.SetHaltTime(viper.GetUint64(server.FlagHaltTime)),
        baseapp.SetInterBlockCache(cache),
    )
}

func exportAppStateAndTMValidators(
    logger log.Logger, db dbm.DB, traceStore io.Writer, height int64, forZeroHeight bool, jailWhiteList []string,
) (json.RawMessage, []tmtypes.GenesisValidator, error) {

    if height != -1 {
        aApp := app.NewInitApp(logger, db, traceStore, false, uint(1))
        err := aApp.LoadHeight(height)
        if err != nil {
            return nil, nil, err
        }
        return aApp.ExportAppStateAndValidators(forZeroHeight, jailWhiteList)
    }

    aApp := app.NewInitApp(logger, db, traceStore, true, uint(1))

    return aApp.ExportAppStateAndValidators(forZeroHeight, jailWhiteList)
}
```

Finally we need to update our new `cmd` names within our `Makefile`. It should be updated to look like:

```javascript
PACKAGES=$(shell go list ./... | grep -v '/simulation')

VERSION := $(shell echo $(shell git describe --tags) | sed 's/^v//')
COMMIT := $(shell git log -1 --format='%H')

ldflags = -X github.com/cosmos/cosmos-sdk/version.Name=NewApp \
    -X github.com/cosmos/cosmos-sdk/version.ServerName=scavengeD \
    -X github.com/cosmos/cosmos-sdk/version.ClientName=scavengeCLI \
    -X github.com/cosmos/cosmos-sdk/version.Version=$(VERSION) \
    -X github.com/cosmos/cosmos-sdk/version.Commit=$(COMMIT) 

BUILD_FLAGS := -ldflags '$(ldflags)'

.PHONY: all
all: install

.PHONY: install
install: go.sum
        go install -mod=readonly $(BUILD_FLAGS) ./cmd/scavengeD
        go install -mod=readonly $(BUILD_FLAGS) ./cmd/scavengeCLI

go.sum: go.mod
        @echo "--> Ensure dependencies have not been modified"
        GO111MODULE=on go mod verify

# Uncomment when you have some tests
# test:
#     @go test -mod=readonly $(PACKAGES)
.PHONY: lint
# look into .golangci.yml for enabling / disabling linters
lint:
    @echo "--> Running linter"
    @golangci-lint run
    @go mod verify
```

Now our app is configured and ready to go!

Let's fire it up 🔥

If you had any difficulties following this tutorial or simply want to discuss Cosmos tech with us you can [**join our community today**](https://discord.gg/fszyM7K)!

