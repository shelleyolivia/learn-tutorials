---
description: Learn how to build a nameservice application (continued)
---

# Key

## Key <a id="key"></a>

Start by navigating to the `key.go` file in the types folder. Within the `key.go` file, you will see that the keys which will be used throughout the creation of the module have already been created.

Defining keys that will be used throughout the application helps with writing [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) code.

```javascript
package types

const (
    // ModuleName is the name of the module
    ModuleName = "nameservice"

    // StoreKey to be used when creating the KVStore
    StoreKey = ModuleName

    // RouterKey is the module name router key
    RouterKey = ModuleName

    // QuerierRoute to be used for querierer msgs
    QuerierRoute = ModuleName
)
```

If you had any difficulties following this tutorial or simply want to discuss Cosmos tech with us you can [**join our community today**](https://discord.gg/fszyM7K)!

